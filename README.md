# Monitoreo climático del Cauca con Google Earth Engine

Monitoreo de variables climáticas, cobertura del suelo y pendiente en el departamento del Cauca (Colombia) con Google Earth Engine (GEE) y Google Colab, y generación de **mapas de zonas aptas para café** y **mapas de riesgo de roya y muerte descendente (Phoma)** a partir de características de clima y suelo.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/TU_USUARIO/cauca-clima-gee/blob/main/notebooks/01_area_estudio.ipynb)

## Objetivos

1. Delimitar el Cauca como área de estudio y extraer series de tiempo de variables climáticas.
2. Caracterizar la cobertura del suelo (Land Cover) y la pendiente del terreno.
3. Integrar características de clima y suelo en una base de capas común.
4. Producir el mapa de zonas aptas para café mediante análisis multicriterio.
5. Producir mapas de riesgo de roya y de muerte descendente (Phoma).

## Fuentes de datos (catálogo de GEE)

| Tema | Variable | Dataset | ID en GEE | Resolución |
|---|---|---|---|---|
| Límite | Departamento del Cauca | FAO GAUL nivel 1 | `FAO/GAUL/2015/level1` | vector |
| Clima | Precipitación | CHIRPS diario | `UCSB-CHG/CHIRPS/DAILY` | ~5,5 km |
| Clima | Temperatura del aire, humedad del suelo, evaporación | ERA5-Land mensual | `ECMWF/ERA5_LAND/MONTHLY_AGGR` | ~11 km |
| Clima | Temperatura superficial | MODIS LST | `MODIS/061/MOD11A2` | 1 km |
| Clima | Déficit hídrico, evapotranspiración, PDSI | TerraClimate | `IDAHO_EPSCOR/TERRACLIMATE` | ~4,6 km |
| Cobertura | Land Cover 2021 | ESA WorldCover | `ESA/WorldCover/v200` | 10 m |
| Cobertura | Land Cover casi en tiempo real | Dynamic World | `GOOGLE/DYNAMICWORLD/V1` | 10 m |
| Cobertura | Land Cover anual (histórico) | MODIS MCD12Q1 | `MODIS/061/MCD12Q1` | 500 m |
| Relieve | Elevación y pendiente | SRTM | `USGS/SRTMGL1_003` | 30 m |
| Suelo | Textura, pH, carbono orgánico | OpenLandMap | `OpenLandMap/SOL/...` | 250 m |
| Clima | Temperatura media y amplitud térmica (climatología) | WorldClim BIO | `WORLDCLIM/V1/BIO` | ~1 km |
| Restricción | Áreas protegidas | WDPA | `WCMC/WDPA/current/polygons` | vector |

> Verifica cada ID en el [catálogo de Earth Engine](https://developers.google.com/earth-engine/datasets) antes de usarlo: las versiones cambian. Para el límite oficial se puede subir como *asset* el shapefile del DANE/IGAC, y para suelos, la cartografía del IGAC.

## Estructura del repositorio

```
cauca-clima-gee/
├── README.md
├── requirements.txt
├── .gitignore
├── config/
│   ├── config.yaml            # proyecto GEE, fechas, escala, CRS
│   └── criterios_aptitud.yaml # rangos y pesos por cultivo o uso
├── notebooks/
│   ├── 01_area_estudio.ipynb
│   ├── 02_variables_climaticas.ipynb
│   ├── 03_cobertura_suelo.ipynb
│   ├── 04_pendiente_relieve.ipynb
│   ├── 05_caracteristicas_clima_suelo.ipynb
│   └── 06_aptitud_riesgo_cafe.ipynb
├── src/
│   ├── area.py        # carga del límite del Cauca
│   ├── clima.py       # precipitación, temperatura, anomalías
│   ├── cobertura.py   # land cover y cambios
│   ├── relieve.py     # elevación, pendiente, orientación
│   ├── suelo.py       # propiedades del suelo
│   ├── aptitud.py     # reclasificación y superposición ponderada
│   ├── riesgo.py      # índices de riesgo
│   └── exportar.py    # exportación a Drive / assets
├── data/              # tablas pequeñas (CSV); los ráster NO se suben
└── outputs/
    ├── mapas/
    └── tablas/
```

## Módulos del proyecto

### 1. Área de estudio
Carga el límite del Cauca. En GAUL hay que filtrar por nombre exacto para no incluir el Valle del Cauca.

### 2. Variables climáticas
- Precipitación mensual y anual, climatología y anomalías (CHIRPS).
- Temperatura media, máxima y mínima (ERA5-Land, MODIS LST).
- Balance hídrico y déficit (TerraClimate).
- Series de tiempo por municipio o subregión, exportadas a CSV.

### 3. Cobertura del suelo
- Mapa de clases (WorldCover) y área por clase.
- Cambios de cobertura entre años (Dynamic World o MCD12Q1).

### 4. Pendiente y relieve
- Elevación, pendiente (grados y %) y orientación a partir de SRTM.
- Clasificación de la pendiente en rangos (plano, ondulado, quebrado, escarpado).

### 5. Características de clima y suelo
Reúne todas las capas en una imagen multibanda con la misma escala y proyección: precipitación, temperatura, déficit hídrico, elevación, pendiente, cobertura, textura, pH y carbono orgánico. Es la entrada de los módulos 6 y 7.

### 6. Mapa de zonas aptas para café
Análisis multicriterio (notebook `06_aptitud_riesgo_cafe.ipynb`). Cada variable se convierte en un puntaje de 0 a 1 y se combina con suma ponderada:

| Variable | Rango óptimo | Límites | Peso |
|---|---|---|---|
| Temperatura media anual | 18–22 °C | 16–24 °C | 0,30 |
| Lluvia anual | 1.500–2.800 mm | 1.200–3.500 mm | 0,20 |
| pH del suelo | 5,0–5,5 | 4,5–6,5 | 0,20 |
| Altitud | 1.300–1.900 m | 1.000–2.200 m | 0,15 |
| Pendiente | hasta 50 % | 100 % | 0,15 |

Se excluyen zonas construidas, agua, humedales y áreas protegidas. El resultado se clasifica en: no apto, marginal, moderado, apto y muy apto.

### 7. Mapas de riesgo: roya y muerte descendente (Phoma)
Índices de favorabilidad climática para cada enfermedad, calculados solo dentro de la zona cafetera potencial.

**Roya (*Hemileia vastatrix*)**: más fuerte en zonas bajas y cálidas.

| Variable | Condición de mayor riesgo | Peso |
|---|---|---|
| Temperatura media | 21–25 °C | 0,40 |
| Altitud | menos de 1.600 m | 0,25 |
| Lluvia anual | más de 2.000 mm | 0,20 |
| Humedad relativa | más de 85 % | 0,15 |

**Muerte descendente (*Phoma* spp.)**: propia de zonas altas, frías y ventosas.

| Variable | Condición de mayor riesgo | Peso |
|---|---|---|
| Altitud | más de 1.600–1.800 m | 0,30 |
| Temperatura media | menos de 18–20 °C | 0,20 |
| Amplitud térmica día-noche | 11 °C o más | 0,20 |
| Viento | más de 6 km/h | 0,15 |
| Lluvia anual | lluvias prolongadas (más de 3.000 mm) | 0,15 |

Cada índice se clasifica en muy bajo, bajo, medio, alto y muy alto. Los umbrales parten de publicaciones de Cenicafé y los pesos son una propuesta inicial: ambos deben calibrarse con registros de incidencia en campo. El índice no incluye la variedad sembrada (resistente o susceptible a roya), el sombrío ni el brillo solar.

## Requisitos

- Cuenta de Google con acceso a [Earth Engine](https://earthengine.google.com/) y un proyecto de Google Cloud registrado.
- Google Colab.

`requirements.txt`:
```
earthengine-api
geemap
pandas
geopandas
matplotlib
pyyaml
```

## Inicio rápido en Colab

```python
!git clone https://github.com/TU_USUARIO/cauca-clima-gee.git
%cd cauca-clima-gee
!pip install -q -r requirements.txt

import ee, geemap
ee.Authenticate()
ee.Initialize(project='TU_PROYECTO_GEE')

# Límite del Cauca
cauca = (ee.FeatureCollection('FAO/GAUL/2015/level1')
         .filter(ee.Filter.eq('ADM0_NAME', 'Colombia'))
         .filter(ee.Filter.eq('ADM1_NAME', 'Cauca')))

# Precipitación anual 2024 (CHIRPS)
lluvia = (ee.ImageCollection('UCSB-CHG/CHIRPS/DAILY')
          .filterDate('2024-01-01', '2025-01-01')
          .sum().clip(cauca))

# Pendiente (SRTM)
pendiente = ee.Terrain.slope(ee.Image('USGS/SRTMGL1_003')).clip(cauca)

# Cobertura del suelo (ESA WorldCover)
cobertura = ee.ImageCollection('ESA/WorldCover/v200').first().clip(cauca)

m = geemap.Map()
m.centerObject(cauca, 8)
m.addLayer(lluvia, {'min': 1000, 'max': 6000, 'palette': ['white', 'blue', 'purple']}, 'Lluvia 2024 (mm)')
m.addLayer(pendiente, {'min': 0, 'max': 45, 'palette': ['green', 'yellow', 'red']}, 'Pendiente (°)')
m.addLayer(cobertura, {}, 'Land Cover')
m
```

## Hoja de ruta

- [ ] Área de estudio y configuración
- [ ] Series climáticas y anomalías
- [ ] Cobertura del suelo y cambios
- [ ] Pendiente y relieve
- [ ] Capa integrada de clima y suelo
- [ ] Mapa de zonas aptas para café
- [ ] Mapas de riesgo de roya y Phoma
- [ ] Riesgo mensual (monitoreo con CHIRPS y ERA5-Land)
- [ ] Validación con datos de campo (IDEAM, IGAC, UPRA)
- [ ] Exportación de mapas finales

## Licencia

MIT (o la que prefieras). Los datos conservan las licencias de sus fuentes originales.
