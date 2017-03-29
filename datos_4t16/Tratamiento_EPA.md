---
title: "EPA3T2016"
author: "Benito"
date: "15 de marzo de 2017"
output: html_document
---




### Descarga y lectura del fichero EPA

Leemos el fichero con microdatos de la EPA del trimestre de 2017

#####readr, plyr,dplyr,highr,rmarkdown,reshape2,stringi,formatR







###Dividimos por 100 la variable FACTOREL para reducirla a unidades y calculamos el total poblacional. 



```r
epa176$FACTOREL<-epa176$FACTOREL/100
Poblacion<-sum(epa176$FACTOREL)
summary(epa176$FACTOREL)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##   11.41  152.10  226.60  289.60  359.30 2851.00
```

```r
Poblacion
```

```
## [1] 46019618
```


###Filtramos el fichero por las variables que nos interesan les damos nuevos nombres:

CCAA,EDAD5,SEXO1,NAC1,NFORMA,AOI,FACTOREL.


```r
library(plyr); library(dplyr)

epa_sel<-select(epa176,CCAA,EDAD5,SEXO1,NAC1,NFORMA,AOI,FACTOREL)


glimpse(epa_sel)
```

```
## Observations: 158,915
## Variables: 7
## $ CCAA     <chr> "16", "16", "16", "16", "16", "16", "16", "16", "16",...
## $ EDAD5    <dbl> 40, 35, 35, 40, 40, 0, 40, 40, 45, 16, 45, 40, 40, 45...
## $ SEXO1    <chr> "1", "6", "1", "1", "6", "6", "1", "1", "6", "1", "6"...
## $ NAC1     <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1"...
## $ NFORMA   <chr> "S1", "SU", "SP", "SP", "SU", NA, "SU", "S1", "S1", "...
## $ AOI      <chr> "03", "04", "04", "04", "03", NA, "04", "04", "03", "...
## $ FACTOREL <dbl> 274.46, 213.05, 213.05, 235.52, 235.52, 209.10, 274.4...
```



###Creamos una nueva variable "ACTIVIDAD" que identifica la situación respecto del empleo


```r
epa_sel<-epa_sel%>%mutate(ACTIVIDAD=cut(as.numeric(AOI),breaks=c(0,4,6,9),labels=c("Ocupados","Parados","Inactivos")))
```


###Creamos una nueva variable "ESTUDIOS" que agrupa e identifica los niveles de estudio

```r
epa_sel$NFORMA <- mapvalues(epa_sel$NFORMA, from = c('AN','P1','P2','S1','SG','SP','SU'), to = c('01','02','03','04','05','06','07'))

epa_sel<-epa_sel%>%mutate(ESTUDIOS=cut(as.numeric(NFORMA),breaks=c(0,1,3,6,7),labels=c("Analfabetos","E. Primarios","E. Secundarios","E. Universitarios")))
```

###Creamos una nueva variable "GRUPO_EDAD" que reduce y explicita los grupos de edad



```r
epa_sel<-epa_sel%>%mutate(GRUPO_EDAD=cut(as.numeric(EDAD5),breaks=c(-1,10,16,25,35,45,55,65),labels=c("Menores de 16 años","De 16 a 20 años","De 20 a 29 años","De 30 a 39 años","De 40 a 49 años","De 50 a 59 años","Mayores de 60 años")))

muestra_edad<-as.data.frame(table(epa_sel$GRUPO_EDAD))

library(knitr)

kable(muestra_edad, align=c('l','r'),caption = "Distribución de la muestra por edades",format.args = list(decimal.mark = ",", big.mark = "."))
```



|Var1               |   Freq|
|:------------------|------:|
|Menores de 16 años | 24.503|
|De 16 a 20 años    |  6.441|
|De 20 a 29 años    | 14.832|
|De 30 a 39 años    | 18.683|
|De 40 a 49 años    | 24.419|
|De 50 a 59 años    | 24.505|
|Mayores de 60 años | 45.532|

####Eliminamos variables redundantes

```r
epa_sel<-epa_sel%>%select(-EDAD5,-NFORMA,-AOI)
glimpse(epa_sel)
```

```
## Observations: 158,915
## Variables: 7
## $ CCAA       <chr> "16", "16", "16", "16", "16", "16", "16", "16", "16...
## $ SEXO1      <chr> "1", "6", "1", "1", "6", "6", "1", "1", "6", "1", "...
## $ NAC1       <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "...
## $ FACTOREL   <dbl> 274.46, 213.05, 213.05, 235.52, 235.52, 209.10, 274...
## $ ACTIVIDAD  <fctr> Ocupados, Ocupados, Ocupados, Ocupados, Ocupados, ...
## $ ESTUDIOS   <fctr> E. Secundarios, E. Universitarios, E. Secundarios,...
## $ GRUPO_EDAD <fctr> De 40 a 49 años, De 30 a 39 años, De 30 a 39 años,...
```

####Convertimos en factores las variables CCAA, SEXO1 y NAC1, y renombramos estas dos últimas



```r
colnames(epa_sel)[2:3]<-c("SEXO", "NACIONALIDAD")


epa_sel$CCAA<-as.factor(epa_sel$CCAA)
epa_sel$SEXO<-as.factor(epa_sel$SEXO)
epa_sel$NACIONALIDAD<-as.factor(epa_sel$NACIONALIDAD)
  
glimpse(epa_sel)
```

```
## Observations: 158,915
## Variables: 7
## $ CCAA         <fctr> 16, 16, 16, 16, 16, 16, 16, 16, 16, 16, 16, 16, ...
## $ SEXO         <fctr> 1, 6, 1, 1, 6, 6, 1, 1, 6, 1, 6, 6, 1, 1, 1, 6, ...
## $ NACIONALIDAD <fctr> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
## $ FACTOREL     <dbl> 274.46, 213.05, 213.05, 235.52, 235.52, 209.10, 2...
## $ ACTIVIDAD    <fctr> Ocupados, Ocupados, Ocupados, Ocupados, Ocupados...
## $ ESTUDIOS     <fctr> E. Secundarios, E. Universitarios, E. Secundario...
## $ GRUPO_EDAD   <fctr> De 40 a 49 años, De 30 a 39 años, De 30 a 39 año...
```




####Agrupamos las observaciones por todas las variables de nuestro interés (CCAA, SEXO, ACTIVIDAD, ESTUDIOS,GRUPO_EDAD)y obtenemos los totales poblacionales para cada combinación posible


```r
tablas_EPA<-epa_sel%>%group_by(CCAA,SEXO,NACIONALIDAD,ACTIVIDAD,ESTUDIOS,GRUPO_EDAD)%>%summarise(Total=sum(FACTOREL))

#Reordenamos las columnas
tablas_EPA<-select(tablas_EPA,CCAA,SEXO,GRUPO_EDAD,ESTUDIOS,NACIONALIDAD,ACTIVIDAD,TOTAL=Total)

glimpse(tablas_EPA)
```

```
## Observations: 4,360
## Variables: 7
## $ CCAA         <fctr> 01, 01, 01, 01, 01, 01, 01, 01, 01, 01, 01, 01, ...
## $ SEXO         <fctr> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
## $ GRUPO_EDAD   <fctr> De 40 a 49 años, De 50 a 59 años, Mayores de 60 ...
## $ ESTUDIOS     <fctr> Analfabetos, Analfabetos, Analfabetos, E. Primar...
## $ NACIONALIDAD <fctr> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
## $ ACTIVIDAD    <fctr> Ocupados, Ocupados, Ocupados, Ocupados, Ocupados...
## $ TOTAL        <dbl> 220.23, 260.98, 1061.38, 469.66, 14839.38, 14499....
```



####Sustituimos los valores de CCAA, SEXO y NACIONALIDAD por sus literales



```r
tablas_EPA$CCAA <- mapvalues(tablas_EPA$CCAA, from = c('01','02','03','04','05','06','07','08','09','10', '11','12','13','14','15','16','17','51','52'), to = c('01  Andalucía', '02  Aragón', '03  Asturias', '04  Baleares', '05  Canarias', '06  Cantabria', '07  Castilla-León', '08  Castilla-La Mancha', '09  Cataluña', '10  Comunidad Valenciana', '11  Extremadura', '12  Galicia', '13  Madrid', '14  Murcia', '15  Navarra', '16  País Vasco', '17  Rioja, La','51  Ceuta','52  Melilla '))

tablas_EPA$SEXO <- mapvalues(tablas_EPA$SEXO, from = c('1','6'), to = c('Varones', 'Mujeres'))

tablas_EPA$NACIONALIDAD <-mapvalues(tablas_EPA$NACIONALIDAD, from = c('1','2','3'), to = c('Española','Española y doble nacionalidad','Extranjera'))

kable(head(tablas_EPA), caption = "Distribución de la población por todas las variables de interés", format.args = list(decimal.mark = ",", big.mark = "."))
```



|CCAA          |SEXO    |GRUPO_EDAD         |ESTUDIOS     |NACIONALIDAD |ACTIVIDAD |     TOTAL|
|:-------------|:-------|:------------------|:------------|:------------|:---------|---------:|
|01  Andalucía |Varones |De 40 a 49 años    |Analfabetos  |Española     |Ocupados  |    220,23|
|01  Andalucía |Varones |De 50 a 59 años    |Analfabetos  |Española     |Ocupados  |    260,98|
|01  Andalucía |Varones |Mayores de 60 años |Analfabetos  |Española     |Ocupados  |  1.061,38|
|01  Andalucía |Varones |De 16 a 20 años    |E. Primarios |Española     |Ocupados  |    469,66|
|01  Andalucía |Varones |De 20 a 29 años    |E. Primarios |Española     |Ocupados  | 14.839,38|
|01  Andalucía |Varones |De 30 a 39 años    |E. Primarios |Española     |Ocupados  | 14.499,41|


####Ordenamos las filas de datos por los valores de las sucesivas columnas


```r
tablas_EPA<-arrange(tablas_EPA,CCAA,SEXO,GRUPO_EDAD,ESTUDIOS,NACIONALIDAD,ACTIVIDAD)

kable(head(tablas_EPA), caption = "Distribución de la población por todas las variables de interés", format.args = list(decimal.mark = ",", big.mark = "."))
```



|CCAA          |SEXO    |GRUPO_EDAD         |ESTUDIOS     |NACIONALIDAD                  |ACTIVIDAD |      TOTAL|
|:-------------|:-------|:------------------|:------------|:-----------------------------|:---------|----------:|
|01  Andalucía |Varones |Menores de 16 años |NA           |Española                      |NA        | 720.928,09|
|01  Andalucía |Varones |Menores de 16 años |NA           |Española y doble nacionalidad |NA        |  10.080,77|
|01  Andalucía |Varones |Menores de 16 años |NA           |Extranjera                    |NA        |  14.913,55|
|01  Andalucía |Varones |De 16 a 20 años    |Analfabetos  |Española                      |Inactivos |     384,10|
|01  Andalucía |Varones |De 16 a 20 años    |E. Primarios |Española                      |Ocupados  |     469,66|
|01  Andalucía |Varones |De 16 a 20 años    |E. Primarios |Española                      |Parados   |   2.049,88|


####Eliminamos los menores de 16 años




```r
tablas_EPA<-tablas_EPA%>%filter(GRUPO_EDAD!="Menores de 16 años")

kable(head(tablas_EPA), caption = "Distribución de los mayores de 16 años por todas las variables de interés", format.args = list(decimal.mark = ",", big.mark = "."))
```



|CCAA          |SEXO    |GRUPO_EDAD      |ESTUDIOS       |NACIONALIDAD |ACTIVIDAD |    TOTAL|
|:-------------|:-------|:---------------|:--------------|:------------|:---------|--------:|
|01  Andalucía |Varones |De 16 a 20 años |Analfabetos    |Española     |Inactivos |   384,10|
|01  Andalucía |Varones |De 16 a 20 años |E. Primarios   |Española     |Ocupados  |   469,66|
|01  Andalucía |Varones |De 16 a 20 años |E. Primarios   |Española     |Parados   | 2.049,88|
|01  Andalucía |Varones |De 16 a 20 años |E. Primarios   |Española     |Inactivos | 9.391,29|
|01  Andalucía |Varones |De 16 a 20 años |E. Primarios   |Extranjera   |Inactivos | 1.485,19|
|01  Andalucía |Varones |De 16 a 20 años |E. Secundarios |Española     |Ocupados  | 4.995,43|




```r
library(reshape2)
```

```
## Warning: package 'reshape2' was built under R version 3.3.2
```

```r
Valores<-dcast(tablas_EPA,CCAA+SEXO+GRUPO_EDAD+ESTUDIOS+NACIONALIDAD ~ ACTIVIDAD,value.var = "TOTAL")
```




####Calculamos valores absolutos y relativos de empleo , paro e inactividad para el Conjunto Nacional



```r
TotalNac_EPA<-Valores%>%summarise(
  Total_Ocupados=round(sum(Ocupados,na.rm = TRUE),1),
  Total_Parados=round(sum(Parados,na.rm = TRUE),1),
  Total_Activos=Total_Ocupados+Total_Parados,
  Tasa_desempleo=round(100*(Total_Parados/Total_Activos),2),
  Total_Inactivos=sum(Inactivos,na.rm = TRUE),
  Tasa_actividad=round(100*Total_Activos/(Total_Activos+Total_Inactivos),2))

kable(TotalNac_EPA, caption = "Población mayor de 16 años según su relación con la Actividad. Total Nacional", format.args = list(decimal.mark = ",", big.mark = "."))
```



| Total_Ocupados| Total_Parados| Total_Activos| Tasa_desempleo| Total_Inactivos| Tasa_actividad|
|--------------:|-------------:|-------------:|--------------:|---------------:|--------------:|
|     18.508.095|     4.237.799|    22.745.894|          18,63|      15.839.027|          58,95|


####Calculamos valores absolutos y relativos de empleo , paro e inactividad por CCAA



```r
TablaCCAA_EPA<-Valores%>%group_by(CCAA)%>%summarise(
  Total_Ocupados=round(sum(Ocupados,na.rm = TRUE),1),
  Total_Parados=round(sum(Parados,na.rm = TRUE),1),
  Total_Activos=Total_Ocupados+Total_Parados,
  Tasa_desempleo=round(100*(Total_Parados/Total_Activos),2),
  Total_Inactivos=sum(Inactivos,na.rm = TRUE),
  Tasa_actividad=round(100*Total_Activos/(Total_Activos+Total_Inactivos),2))

kable(TablaCCAA_EPA, caption = "Población mayor de 16 años según su relación con la Actividad. Comunidades Autónomas", format.args = list(decimal.mark = ",", big.mark = "."))
```



|CCAA                     | Total_Ocupados| Total_Parados| Total_Activos| Tasa_desempleo| Total_Inactivos| Tasa_actividad|
|:------------------------|--------------:|-------------:|-------------:|--------------:|---------------:|--------------:|
|01  Andalucía            |    2.845.256,2|   1.120.305,7|   3.965.561,9|          28,25|    2.935.235,79|          57,47|
|02  Aragón               |      560.840,8|      87.782,1|     648.622,9|          13,53|      446.944,82|          59,20|
|03  Asturias             |      393.757,9|      67.254,4|     461.012,3|          14,59|      444.669,45|          50,90|
|04  Baleares             |      511.640,3|      81.875,9|     593.516,2|          13,80|      356.076,68|          62,50|
|05  Canarias             |      826.280,0|     273.986,0|   1.100.266,0|          24,90|      705.553,67|          60,93|
|06  Cantabria            |      240.819,9|      35.650,3|     276.470,2|          12,89|      215.479,16|          56,20|
|07  Castilla-León        |      972.866,3|     169.132,5|   1.141.998,8|          14,81|      934.997,56|          54,98|
|08  Castilla-La Mancha   |      770.146,4|     219.021,6|     989.168,0|          22,14|      692.017,59|          58,84|
|09  Cataluña             |    3.202.613,2|     558.457,1|   3.761.070,3|          14,85|    2.340.719,02|          61,64|
|10  Comunidad Valenciana |    1.973.843,8|     467.371,5|   2.441.215,3|          19,15|    1.661.873,04|          59,50|
|11  Extremadura          |      357.966,9|     141.349,4|     499.316,3|          28,31|      409.637,55|          54,93|
|12  Galicia              |    1.049.575,2|     204.192,7|   1.253.767,9|          16,29|    1.095.520,89|          53,37|
|13  Madrid               |    2.860.834,6|     489.109,9|   3.349.944,5|          14,60|    1.951.893,07|          63,18|
|14  Murcia               |      571.596,3|     130.453,6|     702.049,9|          18,58|      486.191,61|          59,08|
|15  Navarra              |      277.331,8|      30.836,3|     308.168,1|          10,01|      214.962,15|          58,91|
|16  País Vasco           |      902.104,9|     126.189,2|   1.028.294,1|          12,27|      786.500,49|          56,66|
|17  Rioja, La            |      135.765,9|      16.617,0|     152.382,9|          10,90|      106.785,13|          58,80|
|51  Ceuta                |       27.867,9|       8.040,4|      35.908,3|          22,39|       29.036,49|          55,29|
|52  Melilla              |       26.986,2|      10.173,1|      37.159,3|          27,38|       24.932,61|          59,85|


####Unimos ambas tablas


```r
TablaNACIONAL_EPA<-TotalNac_EPA%>%mutate(CCAA="Conjunto Nacional")
glimpse(TablaCCAA_EPA)
```

```
## Observations: 19
## Variables: 7
## $ CCAA            <fctr> 01  Andalucía, 02  Aragón, 03  Asturias, 04  ...
## $ Total_Ocupados  <dbl> 2845256.2, 560840.8, 393757.9, 511640.3, 82628...
## $ Total_Parados   <dbl> 1120305.7, 87782.1, 67254.4, 81875.9, 273986.0...
## $ Total_Activos   <dbl> 3965561.9, 648622.9, 461012.3, 593516.2, 11002...
## $ Tasa_desempleo  <dbl> 28.25, 13.53, 14.59, 13.80, 24.90, 12.89, 14.8...
## $ Total_Inactivos <dbl> 2935235.79, 446944.82, 444669.45, 356076.68, 7...
## $ Tasa_actividad  <dbl> 57.47, 59.20, 50.90, 62.50, 60.93, 56.20, 54.9...
```

```r
TablaNACIONAL_EPA<-TablaNACIONAL_EPA%>%select(CCAA,Total_Ocupados,Total_Parados,Total_Activos,Tasa_desempleo,Total_Inactivos,Tasa_actividad)


TablaCCAA_EPA<-rbind(TablaNACIONAL_EPA,TablaCCAA_EPA)


kable(TablaCCAA_EPA, caption = "Población mayor de 16 años según su relación con la Actividad por Comunidades Autónomas y Conjunto Nacional", format.args = list(decimal.mark = ",", big.mark = "."))
```



|CCAA                     | Total_Ocupados| Total_Parados| Total_Activos| Tasa_desempleo| Total_Inactivos| Tasa_actividad|
|:------------------------|--------------:|-------------:|-------------:|--------------:|---------------:|--------------:|
|Conjunto Nacional        |   18.508.094,5|   4.237.799,1|  22.745.893,6|          18,63|   15.839.026,77|          58,95|
|01  Andalucía            |    2.845.256,2|   1.120.305,7|   3.965.561,9|          28,25|    2.935.235,79|          57,47|
|02  Aragón               |      560.840,8|      87.782,1|     648.622,9|          13,53|      446.944,82|          59,20|
|03  Asturias             |      393.757,9|      67.254,4|     461.012,3|          14,59|      444.669,45|          50,90|
|04  Baleares             |      511.640,3|      81.875,9|     593.516,2|          13,80|      356.076,68|          62,50|
|05  Canarias             |      826.280,0|     273.986,0|   1.100.266,0|          24,90|      705.553,67|          60,93|
|06  Cantabria            |      240.819,9|      35.650,3|     276.470,2|          12,89|      215.479,16|          56,20|
|07  Castilla-León        |      972.866,3|     169.132,5|   1.141.998,8|          14,81|      934.997,56|          54,98|
|08  Castilla-La Mancha   |      770.146,4|     219.021,6|     989.168,0|          22,14|      692.017,59|          58,84|
|09  Cataluña             |    3.202.613,2|     558.457,1|   3.761.070,3|          14,85|    2.340.719,02|          61,64|
|10  Comunidad Valenciana |    1.973.843,8|     467.371,5|   2.441.215,3|          19,15|    1.661.873,04|          59,50|
|11  Extremadura          |      357.966,9|     141.349,4|     499.316,3|          28,31|      409.637,55|          54,93|
|12  Galicia              |    1.049.575,2|     204.192,7|   1.253.767,9|          16,29|    1.095.520,89|          53,37|
|13  Madrid               |    2.860.834,6|     489.109,9|   3.349.944,5|          14,60|    1.951.893,07|          63,18|
|14  Murcia               |      571.596,3|     130.453,6|     702.049,9|          18,58|      486.191,61|          59,08|
|15  Navarra              |      277.331,8|      30.836,3|     308.168,1|          10,01|      214.962,15|          58,91|
|16  País Vasco           |      902.104,9|     126.189,2|   1.028.294,1|          12,27|      786.500,49|          56,66|
|17  Rioja, La            |      135.765,9|      16.617,0|     152.382,9|          10,90|      106.785,13|          58,80|
|51  Ceuta                |       27.867,9|       8.040,4|      35.908,3|          22,39|       29.036,49|          55,29|
|52  Melilla              |       26.986,2|      10.173,1|      37.159,3|          27,38|       24.932,61|          59,85|

####Calculamos valores absolutos y relativos de empleo , paro e inactividad por SEXO


```r
TablaSEXO_EPA<-Valores%>%group_by(SEXO)%>%summarise(
  Total_Ocupados=round(sum(Ocupados,na.rm = TRUE),1),
  Total_Parados=round(sum(Parados,na.rm = TRUE),1),
  Total_Activos=Total_Ocupados+Total_Parados,
  Tasa_desempleo=round(100*(Total_Parados/Total_Activos),2),
  Total_Inactivos=sum(Inactivos,na.rm = TRUE),
  Tasa_actividad=round(100*Total_Activos/(Total_Activos+Total_Inactivos),2))

kable(TablaSEXO_EPA, caption = "Población mayor de 16 años según su relación con la Actividad. Distribución por sexos", format.args = list(decimal.mark = ",", big.mark = "."))
```



|SEXO    | Total_Ocupados| Total_Parados| Total_Activos| Tasa_desempleo| Total_Inactivos| Tasa_actividad|
|:-------|--------------:|-------------:|-------------:|--------------:|---------------:|--------------:|
|Varones |     10.071.867|     2.095.082|    12.166.949|          17,22|       6.610.365|          64,80|
|Mujeres |      8.436.228|     2.142.717|    10.578.945|          20,25|       9.228.662|          53,41|

####Calculamos valores absolutos y relativos de empleo , paro e inactividad por GRUPOS DE EDAD

```r
TablaGRUPO_EDAD_EPA<-Valores%>%group_by(GRUPO_EDAD)%>%summarise(
  Total_Ocupados=round(sum(Ocupados,na.rm = TRUE),1),
  Total_Parados=round(sum(Parados,na.rm = TRUE),1),
  Total_Activos=Total_Ocupados+Total_Parados,
  Tasa_desempleo=round(100*(Total_Parados/Total_Activos),2),
  Total_Inactivos=sum(Inactivos,na.rm = TRUE),
  Tasa_actividad=round(100*Total_Activos/(Total_Activos+Total_Inactivos),2))

kable(TablaGRUPO_EDAD_EPA, caption = "Población mayor de 16 años según su relación con la Actividad. Distribución por GRUPOS DE EDAD", format.args = list(decimal.mark = ",", big.mark = "."))
```



|GRUPO_EDAD         | Total_Ocupados| Total_Parados| Total_Activos| Tasa_desempleo| Total_Inactivos| Tasa_actividad|
|:------------------|--------------:|-------------:|-------------:|--------------:|---------------:|--------------:|
|De 16 a 20 años    |       96.870,3|     137.522,2|     234.392,5|          58,67|     1.522.271,0|          13,34|
|De 20 a 29 años    |    2.332.103,5|     987.128,7|   3.319.232,2|          29,74|     1.420.053,0|          70,04|
|De 30 a 39 años    |    4.991.395,5|   1.022.470,9|   6.013.866,4|          17,00|       647.449,1|          90,28|
|De 40 a 49 años    |    5.696.273,6|   1.046.358,0|   6.742.631,6|          15,52|       905.196,2|          88,16|
|De 50 a 59 años    |    4.272.236,5|     852.847,7|   5.125.084,2|          16,64|     1.499.028,4|          77,37|
|Mayores de 60 años |    1.119.215,1|     191.471,5|   1.310.686,6|          14,61|     9.845.029,0|          11,75|

####Calculamos valores absolutos y relativos de empleo , paro e inactividad por NACIONALIDAD

```r
TablaNACIONALIDAD_EPA<-Valores%>%group_by(NACIONALIDAD)%>%summarise(
  Total_Ocupados=round(sum(Ocupados,na.rm = TRUE),1),
  Total_Parados=round(sum(Parados,na.rm = TRUE),1),
  Total_Activos=Total_Ocupados+Total_Parados,
  Tasa_desempleo=round(100*(Total_Parados/Total_Activos),2),
  Total_Inactivos=sum(Inactivos,na.rm = TRUE),
  Tasa_actividad=round(100*Total_Activos/(Total_Activos+Total_Inactivos),2))

kable(TablaNACIONALIDAD_EPA, caption = "Población mayor de 16 años según su relación con la Actividad. Distribución por NACIONALIDAD", format.args = list(decimal.mark = ",", big.mark = "."))
```



|NACIONALIDAD                  | Total_Ocupados| Total_Parados| Total_Activos| Tasa_desempleo| Total_Inactivos| Tasa_actividad|
|:-----------------------------|--------------:|-------------:|-------------:|--------------:|---------------:|--------------:|
|Española                      |   15.976.706,4|   3.409.222,1|  19.385.928,5|          17,59|    14.570.916,0|          57,09|
|Española y doble nacionalidad |      523.115,4|     171.183,8|     694.299,2|          24,66|       228.432,3|          75,24|
|Extranjera                    |    2.008.272,7|     657.393,2|   2.665.665,9|          24,66|     1.039.678,5|          71,94|


####Calculamos valores absolutos y relativos de empleo , paro e inactividad por ESTUDIOS

```r
TablaESTUDIOS_EPA<-Valores%>%group_by(ESTUDIOS)%>%summarise(
  Total_Ocupados=round(sum(Ocupados,na.rm = TRUE),1),
  Total_Parados=round(sum(Parados,na.rm = TRUE),1),
  Total_Activos=Total_Ocupados+Total_Parados,
  Tasa_desempleo=round(100*(Total_Parados/Total_Activos),2),
  Total_Inactivos=sum(Inactivos,na.rm = TRUE),
  Tasa_actividad=round(100*Total_Activos/(Total_Activos+Total_Inactivos),2))

kable(TablaESTUDIOS_EPA, caption = "Población mayor de 16 años según su relación con la Actividad. Distribución por NIVEL DE ESTUDIOS", format.args = list(decimal.mark = ",", big.mark = "."))
```



|ESTUDIOS          | Total_Ocupados| Total_Parados| Total_Activos| Tasa_desempleo| Total_Inactivos| Tasa_actividad|
|:-----------------|--------------:|-------------:|-------------:|--------------:|---------------:|--------------:|
|Analfabetos       |       39.990,1|      31.131,0|      71.121,1|          43,77|       593.013,1|          10,71|
|E. Primarios      |    1.141.464,5|     556.825,7|   1.698.290,2|          32,79|     6.088.570,2|          21,81|
|E. Secundarios    |    9.544.041,6|   2.652.424,4|  12.196.466,0|          21,75|     7.035.023,3|          63,42|
|E. Universitarios |    7.782.598,3|     997.418,0|   8.780.016,3|          11,36|     2.122.420,2|          80,53|









