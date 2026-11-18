# Monitoreo climático y riesgo fitosanitario del café en el Cauca con Google Earth Engine

Este proyecto usa Google Earth Engine (GEE) y Google Colab para el departamento del Cauca (Colombia). Con él se obtienen:

* el **mapa de zonas aptas para café**;
* los **mapas de riesgo de roya** (*Hemileia vastatrix*) y de **muerte descendente** (*Phoma* spp.) sobre el área sembrada real;
* un **análisis de sensibilidad** de los resultados a los pesos de los criterios;
* el **riesgo mensual** con **alertas tempranas por municipio**.

[!\[Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/TU_USUARIO/cauca-clima-gee/blob/main/notebooks/06_aptitud_riesgo_cafe.ipynb)

## Notebooks

|Notebook|Qué hace|
|-|-|
|`06\_aptitud\_riesgo\_cafe.ipynb`|Calcula la aptitud, el riesgo anual de roya y Phoma sobre el área sembrada y la sensibilidad a los pesos. Exporta mapas en PNG y GeoTIFF|
|`07\_riesgo\_mensual\_alertas.ipynb`|Calcula el riesgo mes a mes con CHIRPS y ERA5-Land, arma la serie de los últimos 24 meses, hace los mapas del último mes y emite alertas por municipio en CSV|

Los dos notebooks funcionan por separado. En cada uno hay que poner el ID del proyecto de Earth Engine en la primera celda y ejecutar las celdas en orden. Para conectarse se usa `auth\_mode='notebook'`, es decir, un enlace y un código. Al generar el token no hay que marcar la opción de solo lectura.

## Fuentes de datos

|Tema|Variable|Dataset|ID en GEE|Resolución|
|-|-|-|-|-|
|Límites|Departamento y municipios|FAO GAUL 2015 niveles 1 y 2|`FAO/GAUL/2015/level1`, `level2`|vector|
|Relieve|Altitud y pendiente|SRTM|`USGS/SRTMGL1\_003`|30 m|
|Clima|Temperatura media y amplitud térmica, 1995–2024|TerraClimate|`IDAHO\_EPSCOR/TERRACLIMATE`|\~4,6 km (corregida a 30 m por altitud)|
|Clima|Precipitación anual y mensual|CHIRPS diario|`UCSB-CHG/CHIRPS/DAILY`|\~5,5 km|
|Clima|Humedad relativa y anomalía mensual de temperatura|ERA5-Land mensual|`ECMWF/ERA5\_LAND/MONTHLY\_AGGR`|\~11 km|
|Suelo|pH|OpenLandMap|`OpenLandMap/SOL/SOL\_PH-H2O\_USDA-4C1A2A\_M/v02`|250 m|
|Cobertura|Land Cover 2021|ESA WorldCover|`ESA/WorldCover/v200`|10 m|
|Cultivo|Probabilidad de presencia de café, 2018–2024|Forest Data Partnership|`projects/forestdatapartnership/assets/coffee/model\_2026a`|10 m|
|Restricción|Áreas protegidas|WDPA|`WCMC/WDPA/current/polygons`|vector|

## Método

### Aptitud para café

Es un análisis multicriterio por suma ponderada. Cada variable se convierte en un puntaje de 0 a 1 con funciones lineales, que pueden ser trapezoidales, crecientes o decrecientes.

|Variable|Rango óptimo|Límites|Peso|
|-|-|-|-|
|Temperatura media anual|18–22 °C|16–24 °C|0,30|
|Lluvia anual|1.500–2.800 mm|1.200–3.500 mm|0,20|
|pH del suelo|5,0–5,5|4,5–6,5|0,20|
|Altitud|1.300–1.900 m|1.000–2.200 m|0,15|
|Pendiente|hasta 50 %|100 %|0,15|

Quedan excluidas las zonas construidas, el agua, los humedales, los manglares y las áreas protegidas.

### Climatología reciente

La temperatura viene de TerraClimate, promediada entre 1995 y 2024. Para pasarla a 30 m se corrige con un gradiente térmico de −0,006 °C/m, que se puede ajustar, aplicado a la diferencia entre la altitud de SRTM y la altitud media de cada celda de TerraClimate. La amplitud térmica es el promedio de la diferencia mensual entre máxima y mínima (tmmx − tmmn), que equivale a la variable BIO2 de WorldClim.

### Área sembrada

El riesgo se calcula solo donde hay café. Para delimitarlo hay tres opciones, que se eligen en `FUENTE\_AREA\_CAFE`:

* `'fdp'`: usa el mapa de probabilidad de presencia de café de Forest Data Partnership, con un umbral de 0,5 que se puede ajustar.
* `'asset'`: usa polígonos propios (por ejemplo, de lotes o veredas cafeteras) subidos como asset de Earth Engine.
* `'modelo'`: usa la zona cafetera potencial de la versión anterior.

Conviene comparar la superficie que da `'fdp'` con la cifra oficial de la Federación Nacional de Cafeteros para el Cauca.

### Riesgo de roya y de muerte descendente

Los índices miden la favorabilidad climática relativa para cada enfermedad y se basan en lo descrito por Cenicafé: la roya es más severa por debajo de 1.600 m y la muerte descendente es limitante por encima de 1.600 m, con temperaturas menores de 20 °C, alta amplitud térmica y lluvias prolongadas.

|Enfermedad|Variable|Mayor riesgo|Peso|
|-|-|-|-|
|Roya|Temperatura media|21–25 °C|0,45|
|Roya|Altitud|menos de 1.300 m|0,30|
|Roya|Lluvia|más de 2.500 mm/año (210 mm/mes)|0,15|
|Roya|Humedad relativa|más de 92 %|0,10|
|Phoma|Altitud|más de 1.900 m|0,35|
|Phoma|Temperatura media|menos de 17 °C|0,25|
|Phoma|Amplitud térmica|11,5 °C o más|0,20|
|Phoma|Lluvia|más de 3.200 mm/año (265 mm/mes)|0,20|

Los umbrales de lluvia, humedad y amplitud se ajustaron con la climatología y la zona anteriores. La sección 10 del notebook 06 muestra el rango de cada variable dentro del área sembrada, para recalibrarlos.

### Sensibilidad a los pesos

El notebook 06 evalúa la sensibilidad de dos formas:

* **Uno a la vez:** cada peso se sube y se baja un 20 % y los demás se reescalan. Se reporta el porcentaje del área que cambia de clase.
* **Monte Carlo:** se simulan 30 juegos de pesos con una distribución de Dirichlet centrada en los pesos base. Se reporta el mapa de acuerdo con la clase base y el porcentaje del área con acuerdo de 0,8 o más.

### Riesgo mensual y alertas

Cada mes se calcula con estas variables:

* **lluvia:** CHIRPS del mes;
* **temperatura:** la climatología de TerraClimate corregida por altitud, más la anomalía del mes en ERA5-Land;
* **humedad relativa:** ERA5-Land del mes;
* **amplitud térmica:** la climatología de TerraClimate para ese mes del año.

Para cada municipio con al menos 100 ha de café se calcula el porcentaje del área en riesgo alto o muy alto (índice de 0,6 o más) y la anomalía frente al mismo mes del periodo 2015–2024. El nivel de alerta se asigna así:

|Nivel|% del área de café en riesgo alto o muy alto|
|-|-|
|Normal|menos de 30 %|
|Atención|30 % a 49,9 %|
|Alerta|50 % o más|

ERA5-Land se publica con unos 3 meses de rezago, así que el último mes disponible suele ser de hace 2 o 3 meses.

## Estructura del repositorio

```
cauca-clima-gee/
├── README.md
├── requirements.txt
├── .gitignore
└── notebooks/
    ├── 06\_aptitud\_riesgo\_cafe.ipynb
    └── 07\_riesgo\_mensual\_alertas.ipynb
```

## Requisitos

* Una cuenta de Google con un proyecto de Google Cloud registrado en [Earth Engine](https://earthengine.google.com/).
* Google Colab, que ya trae las librerías necesarias. Para trabajar en local están en `requirements.txt`.

## Limitaciones

* Los índices miden favorabilidad climática relativa. No predicen la incidencia de las enfermedades y todavía no se han validado con datos de campo.
* No tienen en cuenta la variedad sembrada, el sombrío, el brillo solar ni el manejo del cultivo.
* La humedad relativa de ERA5-Land (\~11 km) suaviza el relieve de montaña. El viento del reanálisis se descartó porque subestima los valores en la zona cafetera.
* El mapa de café de Forest Data Partnership todavía está en evolución y debe contrastarse con cifras oficiales.
* Los límites municipales de GAUL 2015 son aproximados. Para mayor precisión se puede usar el Marco Geoestadístico del DANE subido como asset.

## Hoja de ruta

* \[x] Aptitud para café
* \[x] Riesgo de roya y Phoma
* \[x] Restringir el riesgo al área sembrada real
* \[x] Climatología reciente (TerraClimate 1995–2024) en lugar de WorldClim v1
* \[x] Análisis de sensibilidad a los pesos
* \[x] Riesgo mensual y alertas por municipio
* \[ ] Recalibrar umbrales con la nueva climatología y el área sembrada
* \[ ] Validar con registros de incidencia (Comité de Cafeteros, Cenicafé El Tambo)
* \[ ] Automatizar las alertas mensuales

## Referencias principales

* Cenicafé (2012). *La roya del cafeto*. Avance Técnico 425.
* Gil-Vallejo, L. F. y Leguizamón-Caycedo, J. (2003). Muerte descendente. En *Enfermedades del cafeto en Colombia*, Cenicafé.
* Menza, H. D. y Peláez, M. J. (2016). Epidemiología de la muerte descendente del cafeto (*Phoma* spp.) en tres sistemas de producción de café. *Revista Cenicafé*, 67(1), 66–77.
* Forest Data Partnership: los datos de café son "Produced by Google for the Forest Data Partnership" (CC-BY 4.0).

## Licencia

MIT. Los datos conservan las licencias de sus fuentes originales.

