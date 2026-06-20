# ✈️ Airline Data Engineering Project

## 📌 Descripción del proyecto

Este proyecto implementa un pipeline de datos utilizando una arquitectura Medallion (Bronze, Silver y Gold) sobre un dataset de vuelos comerciales.

El objetivo es realizar la ingesta, validación, limpieza y transformación de datos para generar información confiable para análisis de operaciones aéreas.

La fuente contiene información sobre:
- vuelos
- aerolíneas
- aeropuertos origen/destino
- horarios programados y reales
- retrasos
- cancelaciones
- distancia y tiempos de vuelo

---

# 🏗️ Arquitectura del proyecto

### Landing
Contiene los archivos CSV originales sin modificaciones.

### Bronze
Se almacenan los datos tal como fueron ingeridos desde la fuente en formato Delta.

Tabla: airline_catalog.bronze.flights_bronze


### Silver
Capa destinada a:
- limpieza
- normalización de tipos
- tratamiento de valores nulos
- generación de columnas derivadas

### Gold
Capa analítica con métricas para consumo final.

---

# 🔎 Exploración inicial del dataset (EDA)

## Información general

**Registros analizados:**

539.747 vuelos

**Período analizado:**

Enero 2025

El dataset contiene información operacional de vuelos incluyendo:

- aerolínea operadora
- número de vuelo
- aeropuerto origen y destino
- horarios programados
- horarios reales
- demoras
- cancelaciones
- causas de retraso

---

# 📊 Hallazgos de calidad de datos

## Valores nulos

Se analizaron valores faltantes por columna.

Principales campos afectados:

| Columna | % Nulos |
|---|---:|
| CANCELLATION_CODE | 96.98% |
| CARRIER_DELAY | 81.82% |
| WEATHER_DELAY | 81.82% |
| NAS_DELAY | 81.82% |
| SECURITY_DELAY | 81.82% |

### Interpretación

Los valores nulos encontrados corresponden principalmente a ausencia de eventos:

- `CANCELLATION_CODE` solo existe cuando el vuelo fue cancelado.
- Las causas de demora solo se completan cuando existe una demora asociada.

Por lo tanto, no representan necesariamente errores de calidad.

---

# 🚫 Valores inválidos

Se realizaron validaciones sobre campos críticos:

| Validación | Resultado |
|-|-|
| Demoras negativas | 0 registros |
| Distancias inválidas | 0 registros |
| Duplicados | 0 registros |

El dataset no presenta problemas críticos de integridad.

---

# 🔄 Problemas de transformación detectados

## Fechas

Campo: 
FL_DATE
Actualmente:
STRING
Acción en Silver: Convertir a: DATE


---

## Horarios

Campos:
CRS_DEP_TIME
DEP_TIME
CRS_ARR_TIME
ARR_TIME

Actualmente almacenados como: INT 
Formato:
1301 → 13:01

Acción en Silver:

Transformar a formato horario para facilitar análisis temporales.

---

## Campos booleanos

Campos:
CANCELLED
DIVERTED

Actualmente:
DOUBLE 
Acción:
Convertir a: BOOLEAN

---

# 🧹 Recomendaciones para Silver

Las transformaciones definidas son:

- Normalización de tipos de datos.
- Conversión de fechas y horarios.
- Tratamiento de valores nulos según lógica de negocio.
- Eliminación de columnas técnicas como `_rescued_data`.
- Creación de columnas derivadas:

  - estado del vuelo
  - duración del vuelo
  - diferencia entre horario programado y real
  - clasificación de demora

---

# 📈 Análisis futuros (Gold)

Se plantean métricas analíticas como:

## Performance de aerolíneas

- cantidad de vuelos
- demora promedio
- porcentaje de cancelaciones

## Análisis de aeropuertos

- aeropuertos con mayor tráfico
- rutas más frecuentes

## Análisis de puntualidad

- vuelos demorados
- distribución de retrasos
- principales causas

---

# 🛠️ Tecnologías utilizadas

- Databricks
- Unity Catalog
- Delta Lake
- SQL
- Arquitectura Medallion

---

# ✅ Estado del proyecto

✔ Dataset cargado en Landing  
✔ Tabla Bronze creada  
✔ Exploración inicial realizada  
✔ Validaciones de calidad completadas  
⬜ Construcción de Silver  
⬜ Desarrollo de métricas Gold
