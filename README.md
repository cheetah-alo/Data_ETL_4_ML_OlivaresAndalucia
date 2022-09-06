# Información General

Este proyecto corresponde al proceso de extracción, transformación y carga (ETL) de el trabajo final del Master de Big Data y  Data Analytics realizado el IMF Madrid, basado en la creacion de un modelo de machine learning para el olivar andaluz. 



Este proceso se realizo usando python en notebooks.



# Extracción de Datos y creación del dataset 

Los conjuntos de datos o datasets con los que se va a trabajar en este TFM provienen de la Junta de Andalucía. Estos dataset son datos abiertos y se pueden descargar de la web descrita a continuación:  https://www.juntadeandalucia.es/datosabiertos/portal/catalogo.html

Para este proceso se ejecutó un notebook con 5 líneas de sentencia. Concretamente, se descargaron dos tipos de datos:

* El primero, los datos climáticos obtenidos por Red de Estaciones Agroclimáticas [2002-2020]. Red de Alerta e Información Fitosanitaria (RAIF). Estos datos se recogen desde el año 2002 hasta la actualidad, se encuentran en el siguiente enlace:

https://www.juntadeandalucia.es/datosabiertos/portal/dataset/raif-clima

* El segundo conjunto de datos a descargar es el de Datos obtenidos del seguimiento de plagas y enfermedades en las estaciones de control biológico [2006-2020]. Red de Alerta e Información Fitosanitaria (RAIF). Estos datos se recogen desde el año 2006 hasta la actualidad y se pueden encontrar en el siguiente enlace:

https://www.juntadeandalucia.es/datosabiertos/portal/dataset/raif



# Preparación del dataset

Tras descargar ambos conjuntos de datos, se observa que vienen en formato .rar con el siguiente nombre: raif_climatico_2002_2020.rar y raif_olivar_andalucia_2006_2019.rar. Una vez descomprimidos, el raif_climatico_2002_2020.rar incluye 21 ficheros, 19 de ellos separados de forma anual contienen los datos climáticos diarios recogidos por cada estación agroclimática y los otros, uno contiene la descripción de los campos y el otro la identificación y descripción geográfica de las estaciones agroclimáticas.



El raif_olivar_andalucia_2006_2019.rar incluye 11 ficheros, 8 de ellos contienen los muestreos de las parcelas separados por provincias desde el año 2006 hasta el año 2016, otro contiene los muestreos de las parcelas de todas las provincias desde el año 2017 hasta el año 2019 y los dos últimos ficheros contienen información sobre las parcelas, uno comprende las parcelas de los años 2006 al 2016 y el otro las parcelas desde el año 2017 hasta 2019.



Todos los ficheros incluidos en ambos .rar tienen formato XML, formato que se propone convertir a CSV para que el trato de estos ficheros sea más cómodo. El proceso de conversión a CSV y el cruzado de los dataset con el fin de unificar todos los datos en único dataset para la creación de los modelos de aprendizaje automático se describen en los siguientes apartados.



## Extracción de archivos XML

Lo primero que se hace es descargar los .rar y se descomprimen en el directorio indicado. Tras ello se coge cada fichero XML, se convierte a diccionario con la librería xmltodict, se trata como json, con la librería json, y este diccionario se pasa, mediante la librería pandas, como dataframe. Finalmente, este dataframe se guarda en la carpeta indicada en formato CSV. Este mismo procedimiento se ha realizado con todos los ficheros XML encontrados en ambos .rar.



### RAIF climático

En este apartado se va a describir cómo se han unificado todos los datos extraídos del raif_climatico_2002_2020.rar a excepción del fichero Descripcion_Campos.csv. Este proceso se ha realizado en un nuevo notebook y los pasos seguidos son:

\1.   Se carga por separado cada CSV con los datos anuales climáticos recogidos por cada estación de forma diaria en un dataframe.

\2.   Se realiza una inspección de todos los dataframe y se seleccionan las variables coincidentes: 'COD_EST', 'FECHA', 'T_MAX', 'T_MED', 'T_MIN', 'H_R_MAX', 'H_R_MED', 'H_R_MIN', 'RADIACION', 'LLUVIA', 'V_MAX', 'V_MED', 'V_MIN', 'CV1', 'CV2', 'CV3', 'CV4', 'DIRECCION'

\3.   Tras esto, se juntan todos los dataframe en uno mediante la función append.

\4.   Por otro lado, se ha cargado el dataset de Identificacion_EstacionesClima.csv para añadir a cada fila la descripción de la estación agroclimática con el fin de cruzar este dataframe con el RAIF olivares a través del código de municipio asociado a cada estación. Finalmente, como se dirá más adelante, el cruce se hizo por código de estación y fecha.

\5.   Por último, el RAIF olivar contiene muestreos a partir de 2006 y hasta finales de 2019. Por ello se eliminan del dataframe todas aquellas fechas anteriores al 26 de diciembre de 2005 y posteriores al 11 de diciembre de 2019.

 

###      RAIF olivar y parcelas

En este apartado se va a describir cómo se han unificado todos los datos extraídos del raif_olivar_andalucia_2006_2019.rar. Este proceso se ha realizado en un nuevo notebook y a continuación se describen los pasos seguidos:

* Primero se ha realizado el tratamiento de las parcelas. Para ello se limpiaron por separado ambos dataset para poner en la columna 401 Estación climática asociada el código de estación en sustitución del nombre de la estación como aparecía en algunos casos. Tras ello, y mediante la función append se unificó el dataset de las parcelas.

* Tras las parcelas, se procedió a unificar en un único dataset el RAIF Olivar que venía separado por provincias desde el año 2006 al 2016 y unificado en otro dataset a partir del año 2017 hasta el 2019. Se limpiaron por separado y finalmente se unificaron con la función append.

* Por último, se han cruzado los dataset de parcelas y RAIF olivar en un único dataset a través de la columna CODPARCELA.

### Unión de RAIF climático, olivar y parcelas

En este apartado se va a describir cómo se han cruzado el dataset parcelas y RAIF olivar con estaciones agroclimáticas. Este proceso se ha realizado en el notebook de Python 4_clima_muestreos.ipynb y a continuación se describen los pasos seguidos:

* El cruce de ambos dataframe se va a hacer a través del código de estación y fecha.

* Como los datos de las estaciones agroclimáticas se recogen de forma diaria y los del RAIF Olivar se recogen de manera semanal, se procede a hacer la media semanal de las estaciones agroclimáticas para poder hacer el cruce de ambos dataset. Esto genera la pérdida de dos tercios del dataset ya que la fecha de la media semanal de las estaciones agroclimáticas no coincide con la fecha de la recogida de muestras en RAIF Olivar.

Debido a que se produce una pérdida importante de datos, se replicó la media semanal para cada día de esa semana y gracias a esta replicación no se pierde ningún dato.

