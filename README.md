# ✈️ Airline Data Pipeline (Bronze → Silver → Gold)

## 🚀 Descripción del proyecto

Este proyecto implementa un pipeline de **Data Engineering** utilizando arquitectura **Medallion (Bronze, Silver, Gold)** para el procesamiento y análisis de datos de vuelos comerciales.

El objetivo es construir una solución de datos basada en **Databricks Lakehouse**, utilizando procesos **ETL/ELT**, almacenamiento en **Delta Lake** y un modelo dimensional optimizado para análisis de negocio.

El modelo final permite analizar:

- Puntualidad de vuelos
- Rendimiento de aerolíneas
- Performance de aeropuertos
- Tendencias temporales
- Principales causas de demoras


---

# 🏗️ Arquitectura del pipeline

Flujo general:
Dataset Fuente
|
↓
Landing Layer
|
↓
Bronze Layer
(Datos crudos)
|
↓
Silver Layer
(Limpieza y transformación)
|
↓
Gold Layer
(Star Schema)
|
↓
Analytics / BI



---

# 🟤 Bronze Layer

## Objetivo

Mantener una copia inicial de los datos provenientes del origen conservando la estructura original.

Características:

- Datos crudos del dataset de vuelos
- Sin transformaciones complejas
- Conserva estructura original del origen
- Almacenamiento en formato Delta


---

# ⚪ Silver Layer

## Objetivo

Transformar los datos Bronze en información limpia y consistente preparada para consumo analítico.

Procesos realizados:

- Corrección de tipos de datos
- Normalización de columnas
- Limpieza de registros
- Tratamiento de valores nulos
- Preparación para modelado dimensional


---

# 🟡 Gold Layer (Modelo Estrella)

La capa Gold implementa un modelo dimensional siguiendo buenas prácticas de **Kimball Dimensional Modeling**.


## 📌 Dimensiones

### dim_airline

Aerolíneas enriquecidas mediante lookup tables.

Contiene:

- Código de aerolínea
- Nombre
- Identificadores dimensionales


### dim_airport

Información de aeropuertos origen y destino:

- Código
- Nombre
- Ciudad
- Estado
- Información geográfica


### dim_date

Dimensión calendario para análisis temporal:

- Día
- Mes
- Año
- Día de semana


---

## 📌 Tabla de hechos

### fact_flights

Tabla central del modelo que representa eventos de vuelos.

Incluye métricas operacionales:

- Distancia
- Demoras
- Cancelaciones
- Información temporal
- Relaciones con dimensiones


---

# 🧠 Modelo de datos

El modelo sigue un **Star Schema**:

- 1 tabla de hechos central (`fact_flights`)
- Dimensiones conformadas reutilizables


Relaciones:
fact_flights → dim_airline

fact_flights → dim_airport (origen)

fact_flights → dim_airport (destino)

fact_flights → dim_date



---

# 🔄 Pipeline de datos

El proceso completo incluye:

1. Ingesta de datos en Bronze
2. Limpieza y normalización en Silver
3. Enriquecimiento mediante lookup tables
4. Construcción de dimensiones
5. Construcción de tabla de hechos
6. Cargas incrementales mediante MERGE idempotente
7. Validaciones de calidad de datos


---

# ⚙️ Características técnicas

✔ Arquitectura Medallion (Bronze / Silver / Gold)

✔ Databricks Lakehouse

✔ Apache Spark

✔ PySpark

✔ Spark SQL

✔ Delta Lake

✔ Procesos ETL / ELT

✔ Modelo estrella (Star Schema)

✔ Modelado dimensional Kimball

✔ Surrogate Keys

✔ Lookup Tables

✔ MERGE para cargas idempotentes

✔ Data Quality Checks


---

# ✈️ Dataset

El dataset contiene información de vuelos comerciales:

- Fechas de vuelo
- Aeropuertos origen y destino
- Aerolíneas
- Horarios programados y reales
- Demoras por múltiples causas
- Distancia
- Duración del vuelo


---

# 📊 Business Questions

La capa Gold permite responder preguntas como:


## ✈️ Aerolíneas

- ¿Qué aerolíneas presentan mayor demora promedio?
- ¿Cuál tiene mejor porcentaje de puntualidad?
- ¿Qué compañías tienen mayor cantidad de vuelos?


## 🛫 Aeropuertos

- ¿Qué aeropuertos presentan mayor retraso?
- ¿Cuáles tienen peor desempeño operacional?


## 📅 Tiempo

- ¿Cómo evolucionan los retrasos por día, mes y año?
- ¿Qué períodos presentan mayor volumen de vuelos?


---

# 🔍 Data Quality

Se implementan controles para validar:

- Registros duplicados
- Valores nulos críticos
- Integridad referencial entre dimensiones y hechos
- Consistencia de métricas operacionales


---

# 🧩 Tecnologías utilizadas

- Python
- SQL
- Apache Spark
- PySpark
- Spark SQL
- Databricks
- Delta Lake
- Git / GitHub


---

# 📂 Estructura del proyecto
airline-data-pipeline/

│
├── README.md
│
├── docs/
│ └── star_schema.png
│
├── Bronze/
│
├── Silver/
│
├── Gold/
│
└── sql/
└── analytics/
├── airline_performance.sql
├── airport_delay_analysis.sql
└── flight_kpis.sql

---

# 📌 Estado del proyecto

✔ Bronze implementado

✔ Silver estructurado

✔ Dimensiones creadas

✔ Lookup integration completada

✔ Fact table implementada

✔ Star Schema construido

✔ Consultas analíticas desarrolladas


---

# 🚀 Próximos pasos

- Optimización de performance con partitioning y ZORDER
- Creación de métricas GOLD (KPIs de puntualidad)
- Dashboards en Power BI / Tableau
- Mayor cobertura de Data Quality
- Orquestación con Apache Airflow


---

# 👨‍💻 Autor

**Ivan Alejandro Lauer**

Junior Data Engineer en formación

GitHub:
https://github.com/Ivanlauer99/

LinkedIn:
https://www.linkedin.com/in/ivan-lauer-data/
