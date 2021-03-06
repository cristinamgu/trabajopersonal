# trabajopersonal


```{r}
install.packages("nycflights13")
library(nycflights13)
library(tidyverse)
vuelos <- nycflights13::flights
```

#### 1. Encuentra todos los vuelos que llegaron más de una hora tarde de lo previsto.

```{r}
a <- filter(vuelos, arr_delay >60)
dim(a)
```

Los vuelos que llegaron por lo menos 1 hora después de su hora de llega estimada fueron 27789

#### 2. Encuentra todos los vuelos que volaron hacia San Francisco (aeropuertos SFO y OAK).

```{r}
b <- filter(vuelos, dest == "SFO" | dest == "OAK")
dim(b)
```

Encontramos 13643 vuelos que tenian como destino San Francisco.

#### 3.  Encuentra todos los vuelos operados por United American (UA) o por American Airlines (AA).

```{r}
c <- filter(vuelos, carrier == "UA" | carrier == "AA")
dim(c)
```

Hay 91394 vuelos operados por United American (UA) o por American Airlines (AA)

#### 4. Encuentra todos los vuelos que salieron en los meses de primavera (Abril, Mayo y Junio)

```{r}
d <- filter(vuelos, month == 4 | month == 5 | month == 6)
dim(d)
```

Hay 85369 vuelos entre los meses de primavera.

#### 5. Encuentra todos los vuelos que llegaron más de una hora tarde pero salieron con menos de una hora de retraso.

```{r}
e <- filter(vuelos, arr_delay > 60 & dep_delay < 60)
dim(e)
```

4956 vuelos llegaron más de una hora tarde pero salieron con menos de una hora de retraso

#### 6. Encuentra todos los vuelos que salieron con más de una hora de retraso pero consiguieron llegar con menos de 30 minutos de retraso (el avión aceleró en el aire)

```{r}
f <- filter(vuelos, dep_delay > 60 & arr_delay < 30)
dim(f)
```

181 vuelos salieron con más de una hora de retraso pero consiguieron llegar con menos de 30 minutos de retraso.

#### 7. Encuentra todos los vuelos que salen entre medianoche y las 7 de la mañana (vuelos nocturnos).

```{r}
g <- filter(vuelos, dep_time <=700) 
dim(g)
```

Los vuelos que salen entre la medianoche y las 7 de la mañana son 31958.

#### 8. ¿Cuántos vuelos tienen un valor desconocido de dep_time?

```{r}
a <- which(is.na(vuelos$dep_time))
length(a)
```

#### 9. ¿Qué variables del dataset contienen valores desconocidos?

```{r}
apply(X = is.na(vuelos), MARGIN = 2, FUN = sum)
```
Las variables que contienen valores desconocidos son: dep_time, dep_delay, arr_time, arr_delay, tailnum, air_time.

#### 10. Ordena los vuelos de flights para encontrar los vuelos más retrasados en la salida. ¿Qué vuelos fueron los que salieron los primeros antes de lo previsto?

```{r}
retrasados <- arrange(vuelos,desc(dep_delay))
adelantados <- arrange(vuelos,dep_delay)
```

Los vuelos que salieron los primeros antes de los previsto fueron aquellos que tienen los siguientes tailnum: N592JB (salió 43 mins antes) y N612DL (salió 33 mins antes).

#### 11. Ordena los vuelos de flights para encontrar los vuelos más rápidos. Usa el concepto de rapidez que consideres.

```{r}
vuelos$velocidad <- (vuelos$distance/vuelos$air_time)
arrange(vuelos,desc(velocidad))
```
#### 12.  ¿Qué vuelos tienen los trayectos más largos?

```{r}
arrange(vuelos,desc(distance))
```
El trayecto más largo es siempre el que parte del aeropuerto JFK y aterriza en el aeropuerto HNL con una distancia total de 4983 millas.

#### 13. ¿Qué vuelos tienen los trayectos más cortos?

```{r}
arrange(vuelos,distance)
```

El trayecto más corto es siempre aquel que sale del aeropuerto EWR y llega al aeropuerto PHL con una distancia de 80 millas. El primer vuelo no lo tengo en cuenta porque supongo que fue cancelado ya que no tiene tiempo de vuelo (aunque si hubiese salido sería el que tiene menor distancia).

#### 14.  El dataset de vuelos tiene dos variables, dep_time y sched_dep_time muy útiles pero difíciles de usar por cómo vienen dadas al no ser variables continuas. Fíjate que cuando pone 559, se refiere a que el vuelo salió a las 5:59... Convierte este dato en otro más útil que represente el número de minutos que pasan desde media noche.

```{r}
vuelos_nueva_min <- mutate(vuelos, salida_programada_min = (sched_dep_time %/% 100 * 60 + sched_dep_time %% 100) %% 1440, horario_salida_min = (dep_time %/% 100 * 60 + dep_time %% 100) %% 1440)
vuelos_nueva_min
```

#### 15.Compara los valores de dep_time, sched_dep_time y dep_delay. ¿Cómo deberían relacionarse estos tres números? Compruébalo y haz las correcciones numéricas que necesitas.

```{r}
vuelos_nueva_min$comprobacion1 <- (vuelos_nueva_min$sched_dep_time + vuelos_nueva_min$dep_delay)

vuelos_nueva_min$comprobacion2 <- (vuelos_nueva_min$salida_programada_min + vuelos_nueva_min$dep_delay)
```

La relación existente entre las tres variables tendría que corresponderse con que dep_time = sched_dep_time + dep_delay, pero al realizar la nueva columna (comprobacion1) comprobamos que no coinciden porque están en unidades distintas. Es por ello, que utilizariamos la nueva variable para la salida programada (en minutos) creadas en el ejercicio anterior.

#### 16. Investiga si existe algún patrón del número de vuelos que se cancelan cada día.

```{r}
vueloscancelados <-  vuelos %>%
  mutate(cancelado = (is.na(arr_delay) | is.na(arr_delay))) %>%
  group_by(year, month, day) %>%
  summarise(num_cancelado = sum(cancelado), num_vuelo = n(),)
ggplot(vueloscancelados) +
  geom_line(aes(x = num_vuelo, y = num_cancelado, col=num_vuelo,)) 
```

Con el gráfico anterior podemos deducir que el número de vuelos cancelados aumenta cuando aumenta el número de vuelos, es por ello que podemos afirmar que son dos variables directamente proporcionales.

#### 17. Investiga si la proporción de vuelos cancelados está relacionada con el retraso promedio por día en los vuelos.

```{r}
propretraso_cancel <- 
  vuelos %>%
  mutate(cancelados = (is.na(tailnum))) %>%
  group_by(year, month, day) %>%
  summarise(proporcion.cancelados = mean(cancelados),media_dep_delay = mean(dep_delay, na.rm = TRUE),media_arr_delay = mean(arr_delay, na.rm = TRUE)) %>% ungroup()

ggplot(propretraso_cancel) + geom_line(aes(x = media_dep_delay, y = proporcion.cancelados, col=proporcion.cancelados))
```

Con el gráfico obtenido no podemos afirmar que exista una relacion directa entre la proporción de vuelos cancelados y la media del tiempo de retraso de los mismos.

#### 18. Investiga si la proporción de vuelos cancelados está relacionada con el retraso promedio por aeropuerto en los vuelos.

```{r}
LGA <- filter(vuelos, origin=="LGA")

Prop_retraso_cancel <- 
  LGA %>%
  mutate(cancelados = (is.na(tailnum))) %>%
  group_by(origin, dest) %>%
  summarise(prop_cancelados = mean(cancelados),med_dep_delay = mean(dep_delay, na.rm = TRUE),med_arr_delay = mean(arr_delay, na.rm = TRUE)) %>% ungroup()

ggplot(Prop_retraso_cancel) +
  geom_line(aes(x = med_dep_delay, y = prop_cancelados, col= prop_cancelados))

JFK <- filter(vuelos, origin=="JFK")

Prop_retraso_cancel <- 
  JFK %>%
  mutate(cancelados = (is.na(tailnum))) %>%
  group_by(origin, dest) %>%
  summarise(prop_cancelados = mean(cancelados),med_dep_delay = mean(dep_delay, na.rm = TRUE),med_arr_delay = mean(arr_delay, na.rm = TRUE)) %>% ungroup()

ggplot(Prop_retraso_cancel) +
  geom_line(aes(x = med_dep_delay, y = prop_cancelados, col= prop_cancelados))

EWR <- filter(vuelos, origin=="EWR")

Prop_retraso_cancel <- 
  EWR %>%
  mutate(cancelados = (is.na(tailnum))) %>%
  group_by(origin, dest) %>%
  summarise(prop_cancelados = mean(cancelados),med_dep_delay = mean(dep_delay, na.rm = TRUE),med_arr_delay = mean(arr_delay, na.rm = TRUE)) %>% ungroup()

ggplot(Prop_retraso_cancel) +
  geom_line(aes(x = med_dep_delay, y = prop_cancelados, col= prop_cancelados))
```

Se observa una relación entre el número de vuelos cancelados y la media del tiempo de retraso en el aeropuerto JFK, pero para los otros dos aeropuertos no podemos afirmar lo mismo con las graficas obtenidas.

#### 19. ¿Qué compañía aérea sufre los peores retrasos?

```{r}
vuelos %>%
   group_by(carrier) %>%
   summarise(dep_delay = mean(dep_delay, na.rm = TRUE)) %>%
   arrange(desc(dep_delay))
vuelos %>%
   group_by(carrier) %>%
   summarise(arr_delay = mean(arr_delay, na.rm = TRUE)) %>%
   arrange(desc(arr_delay))
```

La compañia F9 es la que sufre los peores retrasos tanto en la hora de partida de los vuelos como a la llegada de los mismos al aeropuerto.

#### 20.  Queremos saber qué hora del día nos conviene volar si queremos evitar los retrasos en la salida.

```{r}
vuelos %>%
  group_by(hour) %>%
  summarise(dep_delay = mean(dep_delay, na.rm = TRUE)) %>%
  arrange(dep_delay)
```

La mejor hora para coger un avión si lo que queremos es evitar retrasos en el vuelo es las 5 de la mañana.

#### 21. Queremos saber qué día de la semana nos conviene volar si queremos evitar los retrasos en la salida.

```{r}
make_dtime <- function(year, month, day, time) {
  make_datetime(year, month, day, time %/% 100, time %% 100)
}
vuelos_dt <- vuelos %>% 
  filter(!is.na(dep_time), !is.na(arr_time)) %>% 
  mutate(
    dep_time = make_dtime(year, month, day, dep_time),
    arr_time = make_dtime(year, month, day, arr_time),
    sched_dep_time = make_dtime(year, month, day, sched_dep_time),
    sched_arr_time = make_dtime(year, month, day, sched_arr_time)
  ) %>% 
  select(origin, dest, ends_with("delay"), ends_with("time"))
vuelos_dt %>%
  mutate(dow = wday(sched_dep_time)) %>%
  group_by(dow) %>%
  summarise(
    dep_delay = mean(dep_delay),
    arr_delay = mean(arr_delay, na.rm = TRUE)
  ) %>%
  print(n = Inf)

vuelos_dt %>%
   mutate(wday = wday(dep_time, label = TRUE)) %>% 
   group_by(wday) %>% 
   summarize(ave_dep_delay = mean(dep_delay, na.rm = TRUE)) %>% 
   ggplot(aes(x = wday, y = ave_dep_delay)) + 
   geom_bar(stat = "identity", col = "red")
```

Si queremos evitar los retrasos en la salida del vuelo el día que más nos conviene viajar es en Sábado.

#### 22. Para cada destino, calcula el total de minutos de retraso acumulado.

```{r}
retraso_total_vuelos <- vuelos %>%
  filter(arr_delay > 0) %>%
  group_by(dest) %>%
  summarise(arr_delay = sum(arr_delay))
retraso_total_vuelos
```

#### 23. Para cada uno de ellos, calcula la proporción del total de retraso para dicho destino.

```{r}
vuelos %>%
   filter(arr_delay > 0) %>%
   group_by(dest, origin, carrier, flight) %>%
   summarise(arr_delay = sum(arr_delay)) %>%
   group_by(dest) %>%
   mutate(
     arr_delay_prop = arr_delay / sum(arr_delay)
   ) %>%
   arrange(dest, desc(arr_delay_prop)) %>%
   select(carrier, flight, origin, dest, arr_delay_prop) 
```

#### 24. Es hora de aplicar todo lo que hemos aprendido para visualizar mejor los tiempos de salida para vuelos cancelados vs los no cancelados. Recuerda bien qué tipo de dato tenemos en cada caso. ¿Qué deduces acerca de los retrasos según la hora del día a la que está programada el vuelo de salida?

```{r}
vuelos_dt %>%
  mutate(sched_dep_hour = hour(sched_dep_time)) %>%
  group_by(sched_dep_hour) %>%
  summarise(dep_delay = mean(dep_delay)) %>%
  ggplot(aes(y = dep_delay, x = sched_dep_hour)) +
  geom_point() +
  geom_smooth()
```

#### 25.Subir la carpeta a github y facilitar la url.

```{r}

```


#### 26. Al finalizar el documento agrega el comando sessionInfo()

```{r}
sessionInfo()
```



