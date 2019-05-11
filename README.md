# Listas de frecuencia CREA
Listas de frecuencias de las palabras del Corpus de Referencia del Español Actual (CREA) de la Real Academia Española (RAE). Los datos fueron obtenidos de http://corpus.rae.es/creanet.html.


## 1000 formas más frecuentes

### Procesamiento de los datos

Los datos pueden descargarse directamente desde el sitio web de corpus de la RAE.

```r
crea_1000 <- read_delim("http://corpus.rae.es/frec/1000_formas.TXT", delim = "\n", locale = locale(encoding = 'Latin1'))
```

El archivo tiene varios problemas que hay que arreglar para poder utilizarlo: las columnas no están separadas todas por el mismo delimitador (la mayoría de las veces es un tabulador ("\t"), pero en otras hay uno o más espacios), en la última celda hay un caracter extra ("\u001a"), etc.

Primero arreglamos los nombres de las columnas para que sea más fácil trabajar con ellas:

```r
crea_1000 <- janitor::clean_names(crea_1000)
```

Eliminamos los espacios extra

```r
crea_1000 <- mutate(crea_1000, orden_frec_absoluta_frec_normalizada =
  stringr::str_replace_all(orden_frec_absoluta_frec_normalizada, " ", ""))
```

Ahora que desaparecieron los espacios extra, podemos separar las columnas:

```r
crea_1000 <- separate(crea_1000, orden_frec_absoluta_frec_normalizada, c("ranking", "palabra", "frecuencia_absoluta", "frecuencia_normalizada"), "\t")
```

Listo. Solo queda resolver que el tipo de dato de cada columna sea el adecuado. En el caso de `ranking` es sencillo:

```r
crea_1000 <- mutate(crea_1000, ranking = as.numeric(ranking))
```

En el caso de `frecuencia_absoluta`, tenemos que eliminar primero las comas que separan los dígitos:

```r
crea_1000 <- mutate(crea_1000, frecuencia_absoluta = stringr::str_replace_all(frecuencia_absoluta, ",", "")) %>%
  mutate(frecuencia_absoluta = as.numeric(frecuencia_absoluta))
```

La última observación de `frecuencia_normalizada` tiene un caracter extra (\u001a), por lo que tenemos que eliminarlo antes de poder convertir los datos a numéricos:

```r
crea_1000 <- mutate(crea_1000, frecuencia_normalizada = stringr::str_replace_all(frecuencia_normalizada, "\u001a", "")) %>%
  mutate(frecuencia_normalizada = as.numeric(frecuencia_normalizada))
```

¡Listo! Ya podemos guardar el archivo:

```r
write_csv(crea_1000, "crea_1000.csv")
```
