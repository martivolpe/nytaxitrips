# **🚕 Análisis de los viajes de taxi en Nueva York**

Los taxis de Nueva York son emblemas constitutivos de la cultura e identidad newyorkina, desde De Niro en Taxi Driver hasta Carrie Bradshaw en Sex & The City, los medios retratan a los famosos taxis amarillos como parte orgánica del bioma de la gran ciudad. En esta ocasión este trabajo pretende tomar este fenómeno dinámico newyorkino y transformarlo en un análisis de las distintas aristas que tocan los viajes de taxi en toda la ciudad: aspectos económicos, de demanda y de tendencias según geolocalización.

## Dataset
Se empleó el dataset público de GCP (bigquery-public-data) “new_york_taxi_trips” para trabajar con los datos de los taxis amarillos a partir del 2022 ("tlc_yellow_trips_2022"). Esta cuenta con los datos de todos los viajes realizados en el 2022 por vendor, localización (pickup y dropoff), fecha, metros recorridos y aspectos de facturación. 

## Plan de métricas

En línea con el enfoque de negocio planteado se elaboran 12 KPIs que obedecen a un análisis de demanda por un lado y un análisis financiero por el otro, con el objetivo de medir variables que permitan realizar proyecciones a futuro e identificar tendencias.

Las mismas se detallan a continuación junto con las aclaraciones correspondientes tanto en su fórmula como en su utilidad.
<img width="665" alt="image" src="https://github.com/user-attachments/assets/809aef54-dabb-477f-925c-e03b92a63a0f">

[Detalle](https://docs.google.com/spreadsheets/d/1Uvb99d4AfV_Cqxi_HMk6uombVXROjcfWchnstuc9f4U/edit?gid=0#gid=0)

## Objetivos

- Identificar las tendencias de demanda a lo largo del año y de la semana para optimizar la planificación de la flota de taxis
- Comprender la composición del ticket total de los viajes y la ganancia neta de los conductores
- Analizar la relación entre la cantidad de viajes y de facturación de acuerdo a zonas de la ciudad
- Entender en mayor profundidad los comportamientos de los pasajeros en cuanto a propinas y tarifas promedio

## Hipótesis

- A pesar de la cantidad de conceptos de descuentos la mayor parte de la composición de la facturación se compone de ganancia neta para el conductor
- Hay una correspondencia entre la tarifa promedio y la variación de la facturación total
- Hay un descenso de la demana en los días de fin de semana por la menor necesidad de movilidad
- La cantidad de viajes no necesariamente implica mayor ganancia para el conductor por zonas en donde aumentan los descuentos (ex: peajes, tarifas aeroportuarias)

## Transformación y carga de datos (ELT/ETL)

En búsqueda de incrementar la calidad de los datos y traducir la base “cruda” en insights de análisis se realizaron tres capas en el proceso de transformación:

**Bronze layer**: En el proyecto bronze-layer se copia el dataset público de GCP (bigquery-public-data) “new_york_taxi_trips” para trabajar con las tablas que contienen los datos de los viajes de los yellow taxis del período 2019 al 2022:
- tlc_yellow_trips_2021-> en el proyecto bronze-layer renombrada yellow_trips_2021
- tlc_yellow_trips_2022-> en el proyecto bronze-layer renombrada yellow_trips_2022
- taxi_zone_geom-> en el proyecto bronze-layer renombrada zone_geom
Finalmente se define avanzar con la tabla del 2021 y 2022.

**Silver layer**: Transformación del proyecto bronce en un modelo de datos estrella con las siguientes tablas

- Dim_vendor: consolidación de toda la data referida a la dimensión vendor
<img width="404" alt="image" src="https://github.com/user-attachments/assets/92ac4df4-ec06-4b6e-ad15-c664fba2fbae">

Luego se suman columnas para contener datos ficticios referidos al vendor como first_name, last_name y register_date.
Resultado final:
<img width="452" alt="image" src="https://github.com/user-attachments/assets/71d3096e-0c01-463e-9ddf-af472a244d79">



- Dim_location: consolidación de toda la data de zonas geográficas de los viajes referida a la dimensión location, tomando la tabla de locaciones que contenía el dataset original.
<img width="443" alt="image" src="https://github.com/user-attachments/assets/b7bca74f-ffa9-42b4-8a55-e9f2fb96401e">

- Dim_date: consolidación de toda la data referida a la dimensión date (fechas) y creación de un date_id-> [Detalle](https://docs.google.com/spreadsheets/d/1Uvb99d4AfV_Cqxi_HMk6uombVXROjcfWchnstuc9f4U/edit?gid=388777468#gid=388777468)

- Fact_trips: como tabla de hechos con la data pertinente referida a los viajes. En este caso se generó una [Query](https://docs.google.com/spreadsheets/d/1Uvb99d4AfV_Cqxi_HMk6uombVXROjcfWchnstuc9f4U/edit?gid=124986425#gid=124986425) con las siguientes transformaciones:
Creación de un trip_id (PK) para poder asignar un valor único de identificación a cada viaje realizado

Transformaciones referidas a dim_date:
- Integración del date_id (FK) de la tabla dim_date en los campos de pickup_date_id y dropoff_date_id, con el objetivo de optimizar la data, evitando el almacenamiento excesivo en la tabla como la granularidad horaria
- Extracción de la fecha en formato YYYY-MM-DD en los campos pickup_date y dropoff_date
- Extracción de la fecha en formato YYYY en el campo year
- Adición de dos funciones para cálculos financieros:
total_discount: Monto total de los conceptos que significan un descuento para la ganancia neta del conductor, tales como los impuestos, las tarifas aeroportuarias, los peajes.
total_profit: Monto total de ganancia que percibe el conductor (tip+rate).

Por último, se concreta el DER.
<img width="397" alt="image" src="https://github.com/user-attachments/assets/792d8bb9-a6b7-43b3-b486-10030be5374c">

**Gold layer**

Finalmente, se realiza la conexión con BigQuery y se importan las tablas del dataset de la silver-layer: silver-layer-435402.ny_trips. Debido al volumen del dataset, particularmente de la tabla fact (30MM rows), se realizan modificaciones como la segmentación de fechas solo para el 2022 y la exclusión de todos los registros de un vendor_id.

Transformaciones
- Cargado el dataset la primera transformación realizada es la eliminación de la columna de trip_id para optimizar recursos, dado que era un ID con muchos caracteres, y su reemplazo por un ID incremental que luego se reordena para llevar al principio.
- Creación de la tabla calendario



  
