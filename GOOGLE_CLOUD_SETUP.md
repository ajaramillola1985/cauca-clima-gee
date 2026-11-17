# Guía de configuración: Google Cloud + Earth Engine para `cauca-clima-gee`

Pasos para ejecutar `06_aptitud_riesgo_cafe.ipynb` en Google Colab con el proyecto de Google Cloud **`sig-unal-2026`**. Si ya tienes el proyecto creado y registrado, empieza en la sección 4.

> Las consolas de Google cambian de diseño con frecuencia. Aquí se indica *qué* hacer y *dónde* (con enlace directo). Si algo no coincide, la documentación oficial enlazada es la fuente de verdad.

---

## Índice

1. [Conceptos clave](#1-conceptos-clave)
2. [Crear el proyecto de Google Cloud](#2-crear-el-proyecto-de-google-cloud)
3. [Habilitar la API y registrar el proyecto para Earth Engine](#3-habilitar-la-api-y-registrar-el-proyecto-para-earth-engine)
4. [Abrir el notebook en Colab](#4-abrir-el-notebook-en-colab)
5. [Autorizar tu cuenta](#5-autorizar-tu-cuenta)
6. [Verificar la conexión](#6-verificar-la-conexion)
7. [Exportar a Google Drive](#7-exportar-a-google-drive)
8. [Usar el GeoJSON de municipios del repositorio](#8-usar-el-geojson-de-municipios-del-repositorio)
9. [Cuotas de uso no comercial](#9-cuotas-de-uso-no-comercial)
10. [Errores frecuentes](#10-errores-frecuentes)
11. [Enlaces oficiales](#11-enlaces-oficiales)

---

## 1. Conceptos clave

| Concepto | Qué es |
|---|---|
| **Proyecto de Google Cloud** | Contenedor donde se habilitan APIs, se asignan permisos y se cuenta el uso. Earth Engine siempre trabaja a través de un proyecto. |
| **Nombre vs. ID del proyecto** | El *nombre* es solo una etiqueta (p. ej. `SIG-UNAL-2026`). El **ID** es el identificador único, siempre en minúsculas (p. ej. `sig-unal-2026`), y es el que va en el notebook. |
| **Registro en Earth Engine** | Declarar si el uso es comercial o no comercial. Sin registro, el proyecto no puede usar Earth Engine. |
| **Autorización (scopes)** | Los permisos que le das a Earth Engine sobre tu cuenta de Google: leer datos, crear exportaciones, escribir en Drive, etc. |
| **EECU-hora** | Unidad con la que Earth Engine mide el cómputo consumido. |

---

## 2. Crear el proyecto de Google Cloud

1. Abre <https://console.cloud.google.com/projectcreate> con tu cuenta de Google.
2. Escribe el **nombre** del proyecto.
3. Revisa el **ID** que aparece debajo y edítalo si quieres. Reglas: de 6 a 30 caracteres, solo minúsculas, números y guiones, debe empezar con letra y **no se puede cambiar después**.
4. Haz clic en **Crear**.

Quien crea el proyecto con una cuenta personal queda como **propietario** (*Owner*) y no necesita asignarse permisos adicionales.

Para ver el ID de un proyecto que ya existe: abre <https://console.cloud.google.com/>, haz clic en el selector de proyectos (arriba a la izquierda) y copia la columna **ID**.

---

## 3. Habilitar la API y registrar el proyecto para Earth Engine

Con el proyecto seleccionado en la parte superior de la consola:

1. **Habilitar la API:** abre <https://console.cloud.google.com/apis/library/earthengine.googleapis.com> y haz clic en **Habilitar**. Si ya dice *API habilitada* o muestra el botón *Administrar*, está lista.
2. **Registrar el proyecto:** abre <https://console.cloud.google.com/earth-engine>.
   - Elige **uso no comercial** (investigación, docencia y tesis de posgrado entran aquí).
   - Completa el cuestionario: tipo de organización académica, Universidad Nacional de Colombia y una descripción breve del uso.
   - Elige el nivel de cuota **Community** (ver sección 9).

El estado no comercial debe **re-verificarse cada año**. Cambiar un proyecto de no comercial a comercial es **permanente**.

---

## 4. Abrir el notebook en Colab

1. En <https://github.com/ajaramillola1985/cauca-clima-gee>, haz clic en el botón **Open in Colab** del README.
2. En Colab: **Archivo → Guardar una copia en Drive**. Así trabajas sobre tu copia y no pierdes los cambios.
3. Revisa que la celda de la sección 1 tenga tu ID:
   ```python
   PROYECTO_GEE = 'sig-unal-2026'
   ```
4. La próxima vez, abre tu copia desde Google Drive (carpeta *Colab Notebooks*), no desde el botón de GitHub.

---

## 5. Autorizar tu cuenta

La celda de la sección 1 ejecuta:

```python
ee.Authenticate(auth_mode='notebook')
ee.Initialize(project=PROYECTO_GEE)
```

1. Abre el enlace que aparece debajo de la celda.
2. Elige la **misma cuenta de Google que es propietaria** de `sig-unal-2026`.
3. En la pantalla de permisos, **marca "Seleccionar todo"** o todas las casillas una por una. Si dejas alguna sin marcar, podrás ver mapas y descargar, pero **la exportación a Drive fallará** (sección 10).
4. Copia el código de verificación, pégalo en el cuadro de Colab y presiona Enter.

La autorización se guarda en la sesión de Colab. Si la sesión se reinicia o el permiso vence, vuelve a ejecutar la celda. Para forzar una autorización nueva:

```python
ee.Authenticate(auth_mode='notebook', force=True)
ee.Initialize(project=PROYECTO_GEE)
```

> **Nunca subas al repositorio** códigos de verificación ni salidas de la celda de autorización. Antes de guardar el notebook en GitHub usa **Editar → Borrar todas las salidas**.

Documentación: <https://developers.google.com/earth-engine/guides/auth>

---

## 6. Verificar la conexión

En una celda nueva:

```python
print(ee.Number(1).add(1).getInfo())                        # → 2
print(ee.Image('USGS/SRTMGL1_003').bandNames().getInfo())   # → ['elevation']
```

Si ambas líneas responden, la conexión funciona y puedes usar **Entorno de ejecución → Ejecutar todas**.

---

## 7. Exportar a Google Drive

La sección 11 del notebook lanza una tarea que guarda un GeoTIFF a 100 m en la carpeta **`cauca_gee`** de tu Drive.

1. Ejecuta la celda. Debe imprimir `Exportación iniciada`.
2. Sigue el avance en <https://code.earthengine.google.com/tasks> o con `print(tarea.status()['state'])`: `READY` → `RUNNING` → `COMPLETED`.
3. Para todo el Cauca a 100 m puede tardar entre 5 y 20 minutos.

Si la exportación falla pero los mapas y la descarga de la sección 10 funcionan, el problema es casi siempre la **autorización**. Revisa la sección 10 de esta guía, fila *insufficient authentication scopes*.

**Alternativa sin Drive:** la sección 10 del notebook descarga los mismos mapas directamente desde Colab a 250 m (GeoTIFF para QGIS, PNG con límites y municipios en `.gpkg`).

---

## 8. Usar el GeoJSON de municipios del repositorio

El archivo `data/cauca_municipios.geojson` contiene los **42 municipios del Cauca** (polígonos en EPSG:4326, atributos `municipio`, `departamento` y `pais`).

- **Fuente:** Marco Geoestadístico Nacional del DANE, en la versión simplificada distribuida por geoBoundaries (gbOpen, COL ADM2).
- **Licencia:** CC BY 4.0. Cita: DANE (Departamento Administrativo Nacional de Estadística), vía geoBoundaries — Runfola, D. et al. (2020). *geoBoundaries: A global database of political administrative boundaries.* PLoS ONE 15(4): e0231866. doi:[10.1371/journal.pone.0231866](https://doi.org/10.1371/journal.pone.0231866)
- **Precisión:** es una versión **simplificada**, adecuada para mapas a escala departamental y para recortar el área de estudio. Para cálculos de área oficiales o cartografía de detalle, descarga el Marco Geoestadístico Nacional completo en el geoportal del DANE.

### Reemplazar FAO GAUL por este archivo en el notebook

En la sección 3 (*Área de estudio*), reemplaza la definición de `cauca`, `region` y `municipios` por:

```python
import json, urllib.request

URL_MUNICIPIOS = ('https://raw.githubusercontent.com/ajaramillola1985/'
                  'cauca-clima-gee/main/data/cauca_municipios.geojson')
with urllib.request.urlopen(URL_MUNICIPIOS) as r:
    geojson = json.load(r)

# ADM2_NAME se agrega para que las celdas de descarga sigan funcionando sin cambios
municipios = (ee.FeatureCollection(geojson)
              .map(lambda f: f.set('ADM2_NAME', f.get('municipio'))))
region = municipios.geometry()
cauca = ee.FeatureCollection([ee.Feature(region, {'nombre': 'Cauca'})])

print('Municipios:', municipios.size().getInfo())   # debe ser 42
```

La URL *raw* funciona porque Colab abre solo el notebook, no el resto del repositorio. Si el repositorio es privado, Colab no podrá leerla: sube el archivo a la sesión (icono 📁) y usa `json.load(open('cauca_municipios.geojson'))`.

---

## 9. Cuotas de uso no comercial

Desde el **27 de abril de 2026**, cada proyecto no comercial tiene una cuota mensual de cómputo que se reinicia cada mes:

| Nivel | Cuota mensual | Requisitos |
|---|---|---|
| **Community** | 150 EECU-hora | Proyecto no comercial verificado. Nivel por defecto. |
| **Contributor** | 1.000 EECU-hora | Cuenta de facturación activa, pero el uso no comercial de Earth Engine **no se cobra**. Otros servicios de Google Cloud que uses en el proyecto sí pueden generar cobros. |
| **Partner** | 100.000 EECU-hora | Solicitud aparte con revisión manual de varias semanas. |

- Si agotas la cuota, puedes seguir usando Earth Engine con menos capacidad hasta el mes siguiente.
- La cuota cuenta tanto las consultas interactivas como las exportaciones.
- Usar varios proyectos para evadir los límites (*quota farming*) viola los términos de servicio.

Documentación: <https://developers.google.com/earth-engine/guides/noncommercial_tiers>

---

## 10. Errores frecuentes

| Mensaje (fragmento) | Causa | Solución |
|---|---|---|
| `Project ... is not registered to use Earth Engine` | Falta el registro. | Sección 3, paso 2. |
| `Earth Engine API has not been used in project ...` / `SERVICE_DISABLED` | API no habilitada. | Sección 3, paso 1. Espera 1–2 minutos. |
| `Caller does not have required permission to use project ...` | La cuenta autorizada no tiene acceso al proyecto. | Autoriza con la cuenta propietaria, o pide los roles *Service Usage Consumer* y *Earth Engine Resource Viewer*. |
| `Please authorize access...` / `invalid_grant` | Autorización vencida o de otra cuenta. | `ee.Authenticate(auth_mode='notebook', force=True)` y `ee.Initialize(...)`. |
| `Request had insufficient authentication scopes` / `ACCESS_TOKEN_SCOPE_INSUFFICIENT` al exportar | En la autorización quedó desmarcado algún permiso, o alguno quedó de solo lectura. | Ve a <https://myaccount.google.com/connections>, quita el acceso de la aplicación de Earth Engine y vuelve a autorizar marcando **todas** las casillas (sección 5). Luego ejecuta otra vez `ee.Initialize(project=PROYECTO_GEE)` antes de exportar. |
| `User memory limit exceeded` / `Computation timed out` | Consulta interactiva demasiado pesada. | Aumenta la escala (p. ej. de 100 a 250 m) o usa la exportación a Drive. |
| `Too many concurrent aggregations` / HTTP 429 | Límite de concurrencia o cuota agotada. | Espera y reintenta; revisa el uso del mes (sección 9). |
| La descarga directa falla por tamaño | Límite de las descargas desde Colab. | Sube `ESCALA` en la sección 10 del notebook (p. ej. 500) o exporta a Drive. |

Guía de depuración: <https://developers.google.com/earth-engine/guides/debugging>

---

## 11. Enlaces oficiales

- Earth Engine: <https://earthengine.google.com/>
- Configuración de proyectos Cloud para Earth Engine: <https://developers.google.com/earth-engine/cloud/earthengine_cloud_project_setup>
- Autenticación e inicialización: <https://developers.google.com/earth-engine/guides/auth>
- Control de acceso y roles: <https://developers.google.com/earth-engine/guides/access_control>
- Niveles de cuota no comercial: <https://developers.google.com/earth-engine/guides/noncommercial_tiers>
- Exportación de datos: <https://developers.google.com/earth-engine/guides/exporting_images>
- Catálogo de datos: <https://developers.google.com/earth-engine/datasets>
- Marco Geoestadístico Nacional (DANE): <https://geoportal.dane.gov.co/>
- geoBoundaries: <https://www.geoboundaries.org/>
