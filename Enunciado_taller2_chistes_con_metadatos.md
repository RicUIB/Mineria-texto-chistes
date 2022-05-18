---
title: "Introducción y enunciado de la práctica análisis de chsites"
author: ''
date: "16/05/2022"
output:
  html_document: 
    toc: yes
    number_sections: yes
    keep_md: yes
  word_document:
    toc: yes
  pdf_document:
    toc: yes
    number_sections: yes
linkcolor: red
header-includes: \renewcommand{\contentsname}{Contenidos}
citecolor: blue
toccolor: blue
urlcolor: blue
editor_options: 
  chunk_output_type: console
---



# Análisis de un conjunto de chistes con metadatos 


Algunas ayudas y ejemplos en "Data_model_chistes2.Rmd", se ha cambiado a la libreria "word2vec" más reciente pero menos comentada.

El fichero "data/chistes_con_metadatos_curado.csv"  contiene  unos 7170 chistes de la web [100chistes.com](https://www.1000chistes.com/) y de [pintamania](https://www.pintamania.com/es/chistes).



##  Carga de datos


Los datos están en un fichero separado por ";"  contiene 5 variables 


* origen: la web de origen del chiste; 100 chistes o pintamanía `factor`
* titulo: EL título del chiste `character`.
* categoria: cortos|malos|Jaimito; son  una variable `character`  de categorías separadas por "|"
* palabra_clave: políticos|argentinos; son una una variable `character` de palabras clave separadas por "|"
tags; 
* votos: Número de votos `integer`; solo para pintamania
* texto: tipo character;  es el texto del chiste en UTF-8 separado por "" `character`. 





```r
data_raw=read_csv("data/chistes_con_metadatos_curado.csv",col_names=TRUE)
```

```
## Rows: 7169 Columns: 6
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (5): origen, titulo, categorias, palabra_clave, texto
## dbl (1): votos
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```r
glimpse(data_raw)
```

```
## Rows: 7,169
## Columns: 6
## $ origen        <chr> "1000 chistes", "1000 chistes", "1000 chistes", "1000 ch…
## $ titulo        <chr> "Dime con quién andas...", "Luz automática", "Política a…
## $ categorias    <chr> "cortos|malos", "cortos|malos|borrachos|matrimonios", "c…
## $ palabra_clave <chr> "feos", "neveras", "políticos|argentinos", "sangre", "fu…
## $ votos         <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
## $ texto         <chr> "- Dime con quién andas y te diré quién eres.  - No ando…
```



```r
knitr::kable(head(data_raw,20))
```



|origen       |titulo                           |categorias                                        |palabra_clave                      | votos|texto                                                                                                                                                                                                                                                                                |
|:------------|:--------------------------------|:-------------------------------------------------|:----------------------------------|-----:|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|1000 chistes |Dime con quién andas...          |cortos&#124;malos                                 |feos                               |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |
|1000 chistes |Luz automática                   |cortos&#124;malos&#124;borrachos&#124;matrimonios |neveras                            |    NA|Va el marido completamente borracho y le dice a su mujer al irse para cama:  - Me ha pasado algo increíble. He ido al baño y al abrir la puerta se ha encendido la luz automáticamente, sin hacer nada.  - ¡La madre que te parió!, ¡Te mato!, ya te has vuelto a mear en la nevera. |
|1000 chistes |Política argentina               |cortos&#124;malos                                 |políticos&#124;argentinos          |    NA|Un diputado argentino se encuentra en la calle con un amigo de la infancia y éste le pregunta: - ¿Cómo estás llevando esta crisis? - ¡La verdad que duermo como un bebé! - ¡Dormís como un bebé! ¿Pero cómo hacés? - ¡Me despierto cada 3 horas llorando!                            |
|1000 chistes |0 positivo                       |cortos&#124;malos                                 |sangre                             |    NA|- ¡Rápido, necesitamos sangre! - Yo soy 0 positivo. - Pues muy mal, necesitamos una mentalidad optimista.                                                                                                                                                                            |
|1000 chistes |Mejor portero                    |cortos&#124;malos                                 |futbol&#124;porteros               |    NA|- ¿Cuál es el mejor portero del mundial?  - Evidente ¡el de Para-guay!                                                                                                                                                                                                               |
|1000 chistes |Donación para la piscina         |cortos&#124;malos                                 |dinero&#124;agua                   |    NA|El otro día unas chicas llamarón a mi puerta y me pidieron una pequeña donación para una piscina local.  Les di un garrafa de agua.                                                                                                                                                  |
|1000 chistes |Clase de astrología              |cortos&#124;malos&#124;profesores                 |planetas                           |    NA|- Andresito, ¿qué planeta va después de Marte?  - Miércole, señorita.                                                                                                                                                                                                                |
|1000 chistes |Bob Esponja                      |cortos&#124;malos                                 |esponja&#124;gimnasios             |    NA|- ¿Por qué Bob Esponja no va al gimnasio?  - Porque ya está cuadrado.                                                                                                                                                                                                                |
|1000 chistes |Ojalá lloviera                   |cortos&#124;malos                                 |ciegos                             |    NA|Van dos ciegos y le dice uno al otro:  - Ojalá lloviera...  - Ojalá yo también...                                                                                                                                                                                                    |
|1000 chistes |En Canarias                      |cortos&#124;suegras                               |canarias&#124;coches&#124;noticias |    NA|Noticia de última hora!!   Muere una suegra atropellada en Canarias.   Y esto es todo, las 8 en España y UNA menos en Canarias...                                                                                                                                                    |
|1000 chistes |Dicen que estoy loco             |cortos&#124;malos&#124;Jaimito                    |locos&#124;sillas                  |    NA|– Mamá, mamá, en el colegio dicen que estoy loco. – ¿Y quién dice eso de ti? – ...Me lo dicen las sillas...                                                                                                                                                                          |
|1000 chistes |Bocadillo de jamón               |cortos&#124;malos                                 |madres&#124;jamón                  |    NA|– Mamá, mamá, ¿me haces un bocata de jamón? – ¿York? – Sí, túrk.                                                                                                                                                                                                                     |
|1000 chistes |Te echan de varias universidades |malos&#124;cortos                                 |universitarios&#124;universidades  |    NA|- Qué pasa si te expulsan de cuatro univerdades? - .... - Que estás perdiendo facultades                                                                                                                                                                                             |
|1000 chistes |Un pelo en la cama               |cortos&#124;malos                                 |cuentos&#124;pelos                 |    NA|- Qué es un pelo en una cama? - ... - El bello durmiente                                                                                                                                                                                                                             |
|1000 chistes |Entre techos                     |cortos&#124;malos                                 |casas                              |    NA|- Qué le dice el techo del comedor al techo de la cocina? - .... - Te hecho de menos!                                                                                                                                                                                                |
|1000 chistes |Se va la luz                     |cortos&#124;malos                                 |pijos&#124;escuelas                |    NA|- Qué pasa si se va la luz en una escuela privada? - .... - No se ve ni un pijo!                                                                                                                                                                                                     |
|1000 chistes |País sin tacos                   |cortos&#124;malos                                 |país                               |    NA|- En qué se convierte un país en el que se prohíben los tacos? - ....  - En un país destacado!                                                                                                                                                                                       |
|1000 chistes |Messi de aquí a 45 días          |cortos&#124;malos                                 |deportistas&#124;futbol&#124;Messi |    NA|- Qué es Messi en 45 días? - ........ - Mes y medio!                                                                                                                                                                                                                                 |
|1000 chistes |Mundo con forma cubica           |cortos&#124;malos                                 |cubanos&#124;planetas              |    NA|- Qué pasaría si el mundo en lugar de ser una esfera fuera un cubo? - .... - Pues que todos seríamos cubanos                                                                                                                                                                         |
|1000 chistes |Saludable                        |cortos&#124;malos&#124;amigos                     |comidas&#124;deportes              |    NA|- Soy una persona muy saludable. - ¿Haces mucho deporte y comes sano? - No. Es que la gente me saluda por la calle y yo... pues les devuelvo el saludo.                                                                                                                              |


## Extracción del diccionario raw empírico desde los chistes

Extraemos al dic_raw_1 todas las palabras que aparecen  con separación   espacio. 

Criterios iniciales:

* Decidimos enconding a UTF-8  columna  text_utf8 si hay que depurar por enconding habrá que ver cómo.
* Hay que decidir qué se hace con los CARACTERES SPECIALES:{,:; () ¿?!!}. De momento los voy a eliminar 
* Todas las MAYÚSCULAS a MINÚSCULAS
* De momento NO SE ELIMINAN DIGITOS: se quedan tal cual, hay que distinguir los de los dígitos de años.
* No catalogamos idiomas....  se supone que todo está en castellano o términos técnicos que añadiremos
* Castellano es toda palabra o   derivado de palabra que se encuentre en un spelling estándar de castellano que podemos ir adaptando.





```r
library(tidytext)
library(stringr)
texto_df=data_raw
glimpse(texto_df)
```

```
## Rows: 7,169
## Columns: 6
## $ origen        <chr> "1000 chistes", "1000 chistes", "1000 chistes", "1000 ch…
## $ titulo        <chr> "Dime con quién andas...", "Luz automática", "Política a…
## $ categorias    <chr> "cortos|malos", "cortos|malos|borrachos|matrimonios", "c…
## $ palabra_clave <chr> "feos", "neveras", "políticos|argentinos", "sangre", "fu…
## $ votos         <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
## $ texto         <chr> "- Dime con quién andas y te diré quién eres.  - No ando…
```

```r
#arreglo categorias a columnas distintas se podrían pasar a arrays.
texto_df = texto_df %>% separate(col=c("categorias"),sep="\\|",into=paste0("C",1:5),fill="right")
```

```
## Warning: Expected 5 pieces. Additional pieces discarded in 4 rows [1015, 1039,
## 1529, 1669].
```

```r
texto_df = texto_df %>% separate(col=c("palabra_clave"),sep="\\|",into=paste0("palabra",1:5),fill="right")
```

```
## Warning: Expected 5 pieces. Additional pieces discarded in 10 rows [167, 1587,
## 1589, 1657, 1988, 2072, 2190, 2233, 2363, 2376].
```

```r
texto_df =texto_df %>% mutate(texto_curado=str_squish(str_replace_all(texto, "\\:|-|#|_", " ")))
glimpse(texto_df)
```

```
## Rows: 7,169
## Columns: 15
## $ origen       <chr> "1000 chistes", "1000 chistes", "1000 chistes", "1000 chi…
## $ titulo       <chr> "Dime con quién andas...", "Luz automática", "Política ar…
## $ C1           <chr> "cortos", "cortos", "cortos", "cortos", "cortos", "cortos…
## $ C2           <chr> "malos", "malos", "malos", "malos", "malos", "malos", "ma…
## $ C3           <chr> NA, "borrachos", NA, NA, NA, NA, "profesores", NA, NA, NA…
## $ C4           <chr> NA, "matrimonios", NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
## $ C5           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ palabra1     <chr> "feos", "neveras", "políticos", "sangre", "futbol", "dine…
## $ palabra2     <chr> NA, NA, "argentinos", NA, "porteros", "agua", NA, "gimnas…
## $ palabra3     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, "noticias", NA, NA, N…
## $ palabra4     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ palabra5     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ votos        <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ texto        <chr> "- Dime con quién andas y te diré quién eres.  - No ando …
## $ texto_curado <chr> "Dime con quién andas y te diré quién eres. No ando con n…
```

```r
## str_replace_all(text, "\\:|-|#", " ") reemplazo ":" o "-" o "#" por espacio
# esto es necesario para arreglar "hola:Pepe" que quedaría cómo una palabra si elimino:
## str_squish quita espacios  repetidos

texto_tokens=texto_df %>%  unnest_tokens(word, texto_curado)
glimpse(texto_tokens)
```

```
## Rows: 295,503
## Columns: 15
## $ origen   <chr> "1000 chistes", "1000 chistes", "1000 chistes", "1000 chistes…
## $ titulo   <chr> "Dime con quién andas...", "Dime con quién andas...", "Dime c…
## $ C1       <chr> "cortos", "cortos", "cortos", "cortos", "cortos", "cortos", "…
## $ C2       <chr> "malos", "malos", "malos", "malos", "malos", "malos", "malos"…
## $ C3       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "…
## $ C4       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "…
## $ C5       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ palabra1 <chr> "feos", "feos", "feos", "feos", "feos", "feos", "feos", "feos…
## $ palabra2 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ palabra3 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ palabra4 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ palabra5 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ votos    <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ texto    <chr> "- Dime con quién andas y te diré quién eres.  - No ando con …
## $ word     <chr> "dime", "con", "quién", "andas", "y", "te", "diré", "quién", …
```

```r
knitr::kable(head(texto_tokens,20))
```



|origen       |titulo                  |C1     |C2    |C3        |C4          |C5 |palabra1 |palabra2 |palabra3 |palabra4 |palabra5 | votos|texto                                                                                                                                                                                                                                                                                |word          |
|:------------|:-----------------------|:------|:-----|:---------|:-----------|:--|:--------|:--------|:--------|:--------|:--------|-----:|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------|
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |dime          |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |con           |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |quién         |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |andas         |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |y             |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |te            |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |diré          |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |quién         |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |eres          |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |no            |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |ando          |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |con           |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |nadie         |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |eres          |
|1000 chistes |Dime con quién andas... |cortos |malos |NA        |NA          |NA |feos     |NA       |NA       |NA       |NA       |    NA|- Dime con quién andas y te diré quién eres.  - No ando con nadie...  - Eres feo.                                                                                                                                                                                                    |feo           |
|1000 chistes |Luz automática          |cortos |malos |borrachos |matrimonios |NA |neveras  |NA       |NA       |NA       |NA       |    NA|Va el marido completamente borracho y le dice a su mujer al irse para cama:  - Me ha pasado algo increíble. He ido al baño y al abrir la puerta se ha encendido la luz automáticamente, sin hacer nada.  - ¡La madre que te parió!, ¡Te mato!, ya te has vuelto a mear en la nevera. |va            |
|1000 chistes |Luz automática          |cortos |malos |borrachos |matrimonios |NA |neveras  |NA       |NA       |NA       |NA       |    NA|Va el marido completamente borracho y le dice a su mujer al irse para cama:  - Me ha pasado algo increíble. He ido al baño y al abrir la puerta se ha encendido la luz automáticamente, sin hacer nada.  - ¡La madre que te parió!, ¡Te mato!, ya te has vuelto a mear en la nevera. |el            |
|1000 chistes |Luz automática          |cortos |malos |borrachos |matrimonios |NA |neveras  |NA       |NA       |NA       |NA       |    NA|Va el marido completamente borracho y le dice a su mujer al irse para cama:  - Me ha pasado algo increíble. He ido al baño y al abrir la puerta se ha encendido la luz automáticamente, sin hacer nada.  - ¡La madre que te parió!, ¡Te mato!, ya te has vuelto a mear en la nevera. |marido        |
|1000 chistes |Luz automática          |cortos |malos |borrachos |matrimonios |NA |neveras  |NA       |NA       |NA       |NA       |    NA|Va el marido completamente borracho y le dice a su mujer al irse para cama:  - Me ha pasado algo increíble. He ido al baño y al abrir la puerta se ha encendido la luz automáticamente, sin hacer nada.  - ¡La madre que te parió!, ¡Te mato!, ya te has vuelto a mear en la nevera. |completamente |
|1000 chistes |Luz automática          |cortos |malos |borrachos |matrimonios |NA |neveras  |NA       |NA       |NA       |NA       |    NA|Va el marido completamente borracho y le dice a su mujer al irse para cama:  - Me ha pasado algo increíble. He ido al baño y al abrir la puerta se ha encendido la luz automáticamente, sin hacer nada.  - ¡La madre que te parió!, ¡Te mato!, ya te has vuelto a mear en la nevera. |borracho      |

```r
dic_raw_1=sort(unique(texto_tokens$word))
nw=length(dic_raw_1)# 
nw # número de poalbaras distintas
```

```
## [1] 23456
```

## Construcción del modelo de diccionario

Construiremos una tabla de modelado del corpus de palabras de los chistes:

* Como primary key la word ( las `nw` words) (desde el text_raw en utf8)
* Su frecuencia: número de veces que  aparece en los  chistes
* Si es correcta  según un spelling de español  de España (hay que buscar... qué hay mejor)


```r
count_freq=texto_tokens %>% group_by(word) %>% summarise(N=n())
dic_raw_1 = tibble(word=dic_raw_1) %>% left_join(count_freq,by="word")
```


Ahora vemos claramente cómo podemos mejorar las words para UNIFICARLAS en un único "léxico" que nos permita un tratamiento unificado, auqnue las variantes escritas podrían tener significado humorístico.

Ejemplos


Palabras que contienen "zq"


```r
dic_raw_1[grep("zq",dic_raw_1$word),]
```

```
## # A tibble: 4 × 2
##   word          N
##   <chr>     <int>
## 1 izquierda    20
## 2 izquierdo     7
## 3 vazquezy      1
## 4 vezque        1
```


Palabras que  contienen "ch"


```r
dic_raw_1[grep("(ch)",dic_raw_1$word),]
```

```
## # A tibble: 832 × 2
##    word                  N
##    <chr>             <int>
##  1 2ºchiste              1
##  2 abolladuras.dicho     1
##  3 abrochados            1
##  4 acha                  1
##  5 achedo                1
##  6 achica                1
##  7 achííííííííííííís     1
##  8 achillar              1
##  9 achina                1
## 10 achiqué               1
## # … with 822 more rows
```


Palabras (dos palabras) con :


```r
dic_raw_1[grep(":",dic_raw_1$word),]
```

```
## # A tibble: 0 × 2
## # … with 2 variables: word <chr>, N <int>
```
### Añadimos columna  de spelling al diccionario


Primero veamos algunos ejemplos de las sugerencias: ver manual en de  [hunspell](https://docs.ropensci.org/hunspell/articles/intro.html).
[Github diccionarios open office](https://github.com/LibreOffice/dictionaries)


```r
library("spelling")
library("hunspell")
#https://github.com/titoBouzout/Dictionaries # do
#es=dictionary(lang = "diccionarios/es_ES.dic", affix = "diccionarios/es_ES.dic", add_words = NULL,  cache = FALSE)
es_ES<- dictionary("diccionarios/es_ES.dic")
#print(es_ES)
list_dictionaries()# estos son los que  vienen por defecto
```

```
##  [1] "bg_BG"     "ca_ES"     "cs_CZ"     "da_DK"     "de_DE_neu" "de_DE"    
##  [7] "el_GR"     "en_AU"     "en_CA"     "en_GB"     "en_US"     "es_AR"    
## [13] "es_BO"     "es_CL"     "es_CO"     "es_CR"     "es_CU"     "es_DO"    
## [19] "es_EC"     "es_ES"     "es_GT"     "es_HN"     "es_MX"     "es_NI"    
## [25] "es_PA"     "es_PE"     "es_PR"     "es_PY"     "es_SV"     "es_US"    
## [31] "es_UY"     "es_VE"     "fr_FR"     "hr_HR"     "hu-HU"     "id_ID"    
## [37] "it_IT"     "lt_LT"     "lv_LV"     "nb_NO"     "nl_NL"     "pl_PL"    
## [43] "pt_BR"     "pt_PT"     "ro_RO"     "ru_RU"     "sh"        "sk_SK"    
## [49] "sl_SI"     "sr"        "sv_SE"     "uk_UA"     "vi_VN"
```

```r
hunspell_check(c("bieja","colon","colón"),dic= es_ES)
```

```
## [1] FALSE  TRUE FALSE
```

```r
hunspell_suggest(c("bieja","colon","colón"),dic=es_ES)
```

```
## [[1]]
## [1] "vieja" "biela"
## 
## [[2]]
## [1] "colon"  "clono"  "colo"   "colona" "colono" "colan"  "colen"  "color" 
## 
## [[3]]
## [1] "colon" "clonó" "coló"  "colan" "colen"
```

```r
palabras=c("amor", "amoroso", "amorosamente", "amado", "amante", "amador")
hunspell_analyze(palabras,dic=es_ES)
```

```
## [[1]]
## [1] " st:amor"      "a st:mor fl:a"
## 
## [[2]]
## [1] "a st:moroso fl:a"
## 
## [[3]]
## [1] "a st:morosamente fl:a"
## 
## [[4]]
## [1] " st:amar fl:D"
## 
## [[5]]
## [1] " st:amante"       " st:amantar fl:E"
## 
## [[6]]
## [1] " st:amador"      "a st:mador fl:a"
```

Eliminaremos las palabras que aparezcan menos de $K_{min}=3$ o $K_{max}=500$ veces y números y  tomaremos la primera sugerencia para las palabras que den incorrectas y solo la primera sugerencia.




```r
K_min=3
K_max=500
dic_raw_1 = dic_raw_1 %>% filter(N>K_min & N<K_max )
dim(dic_raw_1)
```

```
## [1] 5332    2
```

```r
dic_raw_1= dic_raw_1[-grep("\\w*[0-9]+\\w*\\s*",dic_raw_1$word),]
dim(dic_raw_1)
```

```
## [1] 5259    2
```

```r
palabras_incorrectas= sapply(dic_raw_1$word, FUN=function(x) hunspell_check(x,dic=es_ES))
table(palabras_incorrectas)
```

```
## palabras_incorrectas
## FALSE  TRUE 
##  1485  3774
```

```r
lista_sugerencias= sapply(dic_raw_1$word, FUN=function(x) hunspell_suggest(x,dic=es_ES))

# nos quedamos con la primera tanto para correctas como para incorrectas

dic_raw_1$word_curada=sapply(lista_sugerencias, FUN=function(x) x[1])
dic_raw_1$lista_sugerencias=sapply(lista_sugerencias,
                                          FUN=function(x){
                                            if(length(x)>=1) {return(paste(x,collapse=","))}
                                            if(length(x)==0){return(NA)}
                                            })
# eliminamos NA

dic_raw_1 = dic_raw_1[!is.na(dic_raw_1$word_curada),]
dim(dic_raw_1)
```

```
## [1] 5238    4
```


# Primer modelo de curado de los chistes




```r
knitr::kable(head(dic_raw_1,20))
```



|word       |  N|word_curada |lista_sugerencias                                                                    |
|:----------|--:|:-----------|:------------------------------------------------------------------------------------|
|â          |  5|a           |a,e,o,d,u,y                                                                          |
|aa         |  5|as          |as,a,ara,asa,ata,ala,ama,aja,aya,ea,ar,na,ca,ta,al                                   |
|aaa        | 10|asa         |asa,ara,ata,ala,ama,aja,aya,a                                                        |
|aaaa       |  7|bezaar      |bezaar                                                                               |
|abajo      | 76|abajo       |abajo,abajó,bajo,abaja,abaje,abano,abato,atajo,abalo,ahajo,abajá,abajé,a bajo        |
|abanico    |  4|abanico     |abanico,abanicó,abanicos,abanica,abanicá                                             |
|abecedario |  8|abecedario  |abecedario,abecedarios                                                               |
|abeja      |  7|abeja       |abeja,abaje,aneja,abejar,abejas,abaja,aleja                                          |
|aber       | 12|abre        |abre,saber,caber,haber,abey,ayer,aberra,rabera                                       |
|abeto      |  4|abeto       |abeto,aneto,abetos,beato,abato,abete,abito,ateto,aleto                               |
|abia       | 66|abiar       |abiar,rabia,abina,sabia,abita,labia,abra,aria,amia,babi                              |
|abian      | 10|abina       |abina,rabian,abinan,abitan,abiar,abran,babiano                                       |
|abienta    |  8|avienta     |avienta,ablienta,ambienta,abierta,asienta,alienta,habiente,enrabieta,tienta,entablen |
|abierta    |  8|abierta     |abierta,abiertas,rabieta,abierto,acierta                                             |
|abiertas   |  5|abiertas    |abiertas,abierta,rabietas,abiertos,aciertas                                          |
|abierto    |  5|abierto     |abierto,abiertos,abierta,acierto                                                     |
|abiertos   |  4|abiertos    |abiertos,abierto,abiertas,aciertos                                                   |
|abion      |  5|abino       |abino,sabiondo                                                                       |
|abitacion  |  6|habitaciÃ³n |habitaciÃ³n                                                                          |
|abla       |  7|bala        |bala,alba,ala,abala,nabla,tabla,ambla,fabla,habla,abra,arla,aula,aballa              |

```r
texto_tokens= texto_tokens %>% right_join(dic_raw_1,word_curada,by="word")
```



## Siguiente paso tratamiento de los datos curados y generación de las Document Term Matrix


Primera aproximación generación dela DTM del corpus de peticiones curadas. Cruzar estos datos con los tópicos/key words de losa chistes.
Podéis hacerlo con tidytext o con tm (o con quanteda).



```r
library(tm)
library(tidytext)
texto_tokens$N=1
DTM=cast_dtm(texto_tokens,document="titulo",term="word_curada",value=N)
MM=as.matrix(DTM)
titulos=row.names(MM)
MM=as_tibble(MM)
MM$titulo=titulos
```


## Generación de tópicos 4 tópicos



```r
library(topicmodels)
set.seed(22)
chistes_2=LDA(DTM, k=2, method = "Gibbs", control = NULL, model = NULL)

chistes_documentos <- tidy(chistes_2, matrix = "gamma")
chistes_documentos%>% arrange(document) 
```

```
## # A tibble: 11,960 × 3
##    document                       topic gamma
##    <chr>                          <int> <dbl>
##  1 --DAA---NI YO SE                   1 0.475
##  2 --DAA---NI YO SE                   2 0.525
##  3 -¿A TI QUÉ ES LO QUE MÁS TE MO     1 0.468
##  4 -¿A TI QUÉ ES LO QUE MÁS TE MO     2 0.532
##  5 -NO ME CORRIJAS                    1 0.524
##  6 -NO ME CORRIJAS                    2 0.476
##  7 ,METAS                             1 0.5  
##  8 ,METAS                             2 0.5  
##  9 !!QUE LOCO!!                       1 0.483
## 10 !!QUE LOCO!!                       2 0.517
## # … with 11,950 more rows
```

```r
tabla_topicos =chistes_documentos %>% pivot_wider(id_cols=document, names_from=topic,values_from= gamma) 
names(tabla_topicos)[2:3]=paste0("Topico_",names(tabla_topicos)[2:3])
names(tabla_topicos)
```

```
## [1] "document" "Topico_1" "Topico_2"
```

```r
Topico =apply(tabla_topicos[,2:3],1,
              FUN=function(x) {
                if(x[1]>x[2]){topico=1}
                if(x[1]<x[2]){topico=2}
if(x[1]==x[2]){topico=0}
return(topico)
                })


tabla_topicos = tabla_topicos %>% mutate(Clase=Topico)
tabla_topicos
```

```
## # A tibble: 5,980 × 4
##    document                 Topico_1 Topico_2 Clase
##    <chr>                       <dbl>    <dbl> <dbl>
##  1 Dime con quién andas...     0.561    0.439     1
##  2 Luz automática              0.416    0.584     2
##  3 Política argentina          0.463    0.537     2
##  4 0 positivo                  0.464    0.536     2
##  5 Mejor portero               0.491    0.509     2
##  6 Donación para la piscina    0.475    0.525     2
##  7 Clase de astrología         0.519    0.481     1
##  8 Bob Esponja                 0.509    0.491     1
##  9 Ojalá lloviera              0.491    0.509     2
## 10 En Canarias                 0.467    0.533     2
## # … with 5,970 more rows
```


Podemos extraer también las categoría o palabras clave pero son demasiadas.


```r
C1=texto_df %>% select(titulo, C1)

df= C1 %>% right_join(MM,by="titulo")
names(df)[1:10]
```

```
##  [1] "titulo" "C1"     "dime"   "quien"  "andas"  "eres"   "ando"   "nadie" 
##  [9] "feo"    "marido"
```

```r
library(naivebayes)

set.seed(1)
nrow(df)
```

```
## [1] 7133
```

```r
Ntraining=floor(0.8*nrow(df))
Ntraining
```

```
## [1] 5706
```

```r
Ntesting=nrow(df)-Ntraining
Ntesting
```

```
## [1] 1427
```

```r
training=sample(1:nrow(df),size=Ntraining,replace = FALSE)
testing=setdiff(1:row(df),training)
```

```
## Warning in 1:row(df): numerical expression has 34074341 elements: only the first
## used
```

```r
train_data=df[training,-1]
testing_data=df[testing,-c(1:2)]
```



Quizá demasiadas categorías mejor topic models a 2 , 3 o 4 ,categorías.



# Word to vect NUEVA librería word2vec


https://github.com/bnosac/word2vec



```r
#install.packages("devtools","Rtools")
#install.packages("word2vec")

library(word2vec)
txt_clean=txt_clean_word2vec(x=data_raw$texto, ascii = FALSE, alpha = TRUE, tolower = TRUE, trim = TRUE)
str(txt_clean)
```

```
##  chr [1:7169] "dime con quién andas y te diré quién eres no ando con nadie eres feo" ...
```

```r
model=word2vec(x=txt_clean,
  type = "skip-gram",
  dim = 50,
  window = 10,
  iter = 5L,
  lr = 0.05,
  hs = FALSE,
  negative = 5L,
  sample = 0.001,
  min_count = 5L,
  split = c(" \n,.-!?:;/\"#$%&'()*+<=>@[]\\^_`{|}~\t\v\f\r", ".\n?!"),
  stopwords = character(),
  threads = 1L,
  encoding = "UTF-8"
)
```



```r
embeding=as.matrix(model)
emb <- predict(model, c("autobus", "jaimito", "mujer"), type = "embedding")
emb
```

```
##               [,1]      [,2]       [,3]        [,4]      [,5]       [,6]
## autobus  0.4215567 0.9428163 -0.8552876 -0.36391082 0.9671350 -0.3949787
## jaimito -0.8588676 0.3935143 -0.2069047 -0.01337613 0.6109695  1.4240206
## mujer   -0.1493722 0.3694185 -0.8188819 -0.87417185 1.4728718  0.4671481
##              [,7]       [,8]       [,9]      [,10]    [,11]       [,12]
## autobus  1.384173 -0.3434443 -1.5555979 -1.4749378 1.047273 -0.02751573
## jaimito  1.464965 -0.4736260 -0.8631358 -0.9238483 1.107051 -0.68335766
## mujer   -1.670666 -0.7917522 -0.9374381 -1.1259614 0.985620 -0.84107113
##              [,13]      [,14]     [,15]      [,16]      [,17]      [,18]
## autobus -0.4845360 -0.1534294 -2.456188 -0.7817395 -0.1810317 0.45490038
## jaimito -1.3978566  0.8420411 -1.139609  0.1097986  0.6339280 1.63298965
## mujer   -0.3512531  0.4928747 -1.263952  2.1460378 -0.9479676 0.03398087
##              [,19]      [,20]      [,21]     [,22]      [,23]     [,24]
## autobus -1.4324708  0.2366222 -0.5974758 -1.774317 -0.4350906 -1.449672
## jaimito  0.4728744  0.6823083 -0.1804533 -1.095239 -1.8646376 -2.555794
## mujer   -2.4116209 -0.4968911 -1.2039707  0.442573 -1.6134865 -1.407564
##              [,25]     [,26]      [,27]      [,28]      [,29]       [,30]
## autobus -3.0353773 0.7860203 -0.1958140 -1.2103856  0.1130121 -0.03037001
## jaimito -1.0825338 1.5375080  0.4149951  0.3583186  0.4648841 -0.65435940
## mujer   -0.9180669 0.6169215  0.7923238  0.6126564 -0.6979165  0.47860941
##                [,31]      [,32]      [,33]      [,34]      [,35]     [,36]
## autobus  0.001338045  0.7792166 -0.3514432 -1.1959176  1.2939211 1.5450839
## jaimito -0.794681489  0.9110416  0.1453255  0.1826286 -0.1927757 0.5099583
## mujer    0.553770185 -0.5586681 -0.3998969 -0.5771878  0.8973290 0.1705930
##              [,37]      [,38]     [,39]      [,40]      [,41]      [,42]
## autobus -1.0953430  0.1801254 0.7035779  1.3122360  0.1324582 -0.3055362
## jaimito -0.3567404 -0.6199719 1.5269845  1.5194751  0.9064705 -0.3915070
## mujer   -2.6402757 -0.8543287 0.5429524 -0.7725535 -0.3306820  1.5400352
##              [,43]      [,44]       [,45]      [,46]        [,47]       [,48]
## autobus -1.0885913  0.3016092 -0.10671484  0.4528818 -0.110895596  0.03319363
## jaimito -0.4973685 -2.4686146 -0.05825014  1.2034585 -0.003891494 -0.30288032
## mujer   -1.1476076  0.5469596  0.29674396 -0.1658486  0.268733770 -0.51948953
##               [,49]      [,50]
## autobus -0.02819879 1.25208414
## jaimito  1.06921577 0.76172853
## mujer    0.98436630 0.09067208
```

```r
nn  <- predict(model, c("jaimito", "profesor"), type = "nearest", top_n = 5)
nn
```

```
## $jaimito
##     term1     term2 similarity rank
## 1 jaimito   aleluya  0.8630080    1
## 2 jaimito    decime  0.8621444    2
## 3 jaimito     jaimi  0.8546897    3
## 4 jaimito oraciones  0.8486884    4
## 5 jaimito  cuaderno  0.8481737    5
## 
## $profesor
##      term1     term2 similarity rank
## 1 profesor    alumno  0.9058183    1
## 2 profesor     memin  0.9003770    2
## 3 profesor profesora  0.8879084    3
## 4 profesor    examen  0.8865024    4
## 5 profesor     clase  0.8861179    5
```




```r
doc2vec(model,c("padre","madre","hijo"))
```

```
##            [,1]       [,2]      [,3]        [,4]      [,5]       [,6]     [,7]
## [1,] -0.6631149 -2.2293512 0.3133301 -0.35540514 0.8621576  0.6294288 1.161323
## [2,] -0.5199794 -0.5765639 0.9870838  0.80108881 0.3825774  0.7537708 1.242650
## [3,] -1.1226842 -0.6394330 0.5774237 -0.07083955 0.9901022 -0.8176799 1.682462
##           [,8]       [,9]      [,10]     [,11]      [,12]      [,13]     [,14]
## [1,] -1.592528 -1.3256892 -0.7012890 1.9553201 -1.3534984 -0.1578257 0.7430517
## [2,] -1.646416 -0.2065819 -0.9072923 1.4374909 -0.4760452 -1.1333734 0.3901017
## [3,] -1.633710 -1.2067848 -0.4446349 0.1423274 -0.5367879 -1.0371214 0.2540149
##           [,15]      [,16]      [,17]     [,18]        [,19]       [,20]
## [1,] -1.3470183 -0.1874375  0.1867044 0.5176286  0.002289056  1.06650361
## [2,] -2.0002533  1.6205320 -0.6272374 1.0063061 -1.031888285 -0.20972472
## [3,] -0.1558065  0.5921854 -0.6483002 1.0503434 -0.830729965 -0.05788169
##            [,21]      [,22]      [,23]     [,24]     [,25]      [,26]    [,27]
## [1,] -0.31522285 -0.6847959 -0.6356397 -1.403348 -2.620684 -0.1446983 1.083465
## [2,]  0.09312627 -0.9272729 -0.4030613 -1.972689 -1.418926  1.6557231 1.023034
## [3,]  0.15263030 -1.7268218  0.4781103 -1.570649 -2.435584  0.8328188 1.043498
##           [,28]     [,29]     [,30]      [,31]     [,32]      [,33]      [,34]
## [1,]  1.0768599 0.5175305 1.2982325  0.3573153 0.6158582 -0.1521813 -1.0618535
## [2,] -0.2019940 1.1908026 0.8923329 -0.1949284 1.1543632  0.8592242  1.1768807
## [3,]  0.8397368 1.6164408 1.0406737  0.2923286 1.4632506  1.0512569 -0.4467555
##            [,35]      [,36]       [,37]      [,38]    [,39]      [,40]
## [1,]  0.03624482 -0.8051954  0.54750346 -0.9699155 1.380236 -0.2165192
## [2,] -0.63608326  0.0628906 -0.28252912  0.2568058 2.361701 -0.3077727
## [3,]  0.32350761 -0.1472194  0.04058348  0.1349823 1.700560 -0.7527174
##          [,41]       [,42]      [,43]      [,44]      [,45]     [,46]
## [1,] 2.0281925 -0.32543423 -0.8934513  0.2576817 -0.4598967 0.8825789
## [2,] 0.3774199  0.94201397 -0.8922680 -0.3181949  0.3301518 0.1853569
## [3,] 1.4918513  0.02173101 -0.9286671 -0.3283266  0.5677399 1.6345751
##           [,47]      [,48]       [,49]     [,50]
## [1,] -0.1388690 -1.2007649 0.214398870 0.6093407
## [2,]  0.4646024 -0.6577190 1.852263068 0.4741192
## [3,] -0.5871766 -0.8520733 0.002205131 1.7133539
```



```r
M=as.matrix(model)
dim(M)
```

```
## [1] 4375   50
```

```r
#Simi=word2vec_similarity(M,M,top_n=+Inf, type="cosine")
cosine <- function(x,y) sum(x * y)/sqrt(sum(x^2)*sum(y^2))
# install.packages("proxy")
library(proxy)
SS=as.matrix(simil(M,method=cosine))
diag(SS)=1
D=sqrt(1-SS)
dimnames(D)=list(dimnames(M)[[1]],dimnames(M)[[1]])
sol_MDS=cmdscale(D,k = 3,list=TRUE)
str(sol_MDS)
```

```
## List of 5
##  $ points: num [1:4375, 1:3] -0.3544 -0.0336 0.0986 0.1058 0.0639 ...
##   ..- attr(*, "dimnames")=List of 2
##   .. ..$ : chr [1:4375] "usd" "tocas" "ria" "caducado" ...
##   .. ..$ : NULL
##  $ eig   : NULL
##  $ x     : NULL
##  $ ac    : num 0
##  $ GOF   : num [1:2] 0.241 0.241
```

```r
par(mfrow=c(1,3))
plot(sol_MDS$points[,c(1,2)])
text(sol_MDS$points[,c(1,2)],dimnames(M)[[1]])
plot(sol_MDS$points[,c(1,3)])
text(sol_MDS$points[,c(1,3)],dimnames(M)[[1]])
plot(sol_MDS$points[,c(2,3)])
text(sol_MDS$points[,c(2,3)],dimnames(M)[[1]])
```

![](Enunciado_taller2_chistes_con_metadatos_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

```r
par(mfrow=c(1,1))
```






# Naive bayes
Podéis utilizar algún algoritmo  de naivebayes con los metadatos  de los chistes (fichero que se explica abajo) o con topic models.


## Más chistes con metadatos

En el fichero de este git "chistes_con_metadatos.csv" hay más chistes con dos columnas de metadatos para practicar.


# Enunciado

Basándonos en  las ayudas de Enunciado_taller2_chistes_con_metadatos.Rmd" lo anterior generar un modelo de datos con 4 tópicos (de topic models o combinado con categorías o palabras clave. Asignar cada tópico a su $\alpha$ más alto) y un diccionario de palabras curadas por chistes.


## Cuestión 1

Naive Bayes para predecir las 4 categorias de chistes a partir de las variables  de presencia ausencia de las palabras. Evaluar el modelo.

## Cuestión 2 

A partir de la librería `word2vec` generar una proyección estudiar  si las palbaras (gammas)
