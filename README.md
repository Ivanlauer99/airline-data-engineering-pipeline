# ✈️ Airline Data Pipeline (Bronze → Silver → Gold)

## 🚀 Descripción del proyecto

Este proyecto implementa un pipeline de Data Engineering con arquitectura **Medallion (Bronze, Silver, Gold)** para el análisis de vuelos comerciales.

El objetivo es construir un **modelo estrella (Star Schema)** optimizado para análisis de puntualidad, aerolíneas, aeropuertos y desempeño operativo.

---

## 🏗️ Arquitectura del pipeline

### 🟤 Bronze Layer
- Datos crudos del dataset de vuelos
- Sin transformaciones complejas
- Conserva estructura original del origen

---

### ⚪ Silver Layer
- Datos limpios y estandarizados
- Tipos de datos corregidos
- Columnas normalizadas
- Preparado para modelado dimensional

---

### 🟡 Gold Layer (Modelo Estrella)

#### 📌 Dimensiones
- `dim_airline` → aerolíneas enriquecidas con lookup
- `dim_airport` → aeropuertos origen/destino con información geográfica y nombre
- `dim_date` → dimensión calendario para análisis temporal

#### 📌 Tabla de hechos
- `fact_flights` → eventos de vuelos con métricas operacionales

---

## 🧠 Modelo de datos

El modelo sigue un **Star Schema**:

- 1 tabla de hechos central (`fact_flights`)
- Dimensiones conformadas reutilizables

Relaciones:

- fact_flights → dim_airline
- fact_flights → dim_airport (origen)
- fact_flights → dim_airport (destino)
- fact_flights → dim_date

                         ┌────────────────────┐
                         │     dim_date       │
                         │--------------------│
                         │ date_id (PK)       │
                         │ year               │
                         │ month              │
                         │ day                │
                         │ day_of_week        │
                         └─────────┬──────────┘
                                   │
                                   │
┌────────────────────┐     ┌───────▼──────────────┐     ┌────────────────────┐
│   dim_airline      │     │    fact_flights      │     │   dim_airport      │
│--------------------│     │----------------------│     │--------------------│
│ airline_id (PK)    │────▶│ flight_id (PK)       │◀────│ airport_id (PK)    │
│ airline_code       │     │ date_id (FK)         │     │ airport_code       │
│ airline_name       │     │ airline_id (FK)      │     │ city               │
└────────────────────┘     │ origin_airport_id    │     │ state              │
                           │ destination_airport_id│     │ airport_name       │
                           │ flight_number        │     └────────────────────┘
                           │ departure_delay      │
                           │ arrival_delay        │
                           │ distance             │
                           │ cancelled            │
                           └──────────────────────┘
---

## 🔄 Pipeline de datos

1. Ingesta de datos en Bronze
2. Limpieza y normalización en Silver
3. Enriquecimiento con lookup tables
4. Construcción de dimensiones
5. Construcción de fact table con MERGE idempotente

---

## ⚙️ Características técnicas

- ✔ Arquitectura Medallion
- ✔ Modelo estrella (Star Schema)
- ✔ Cargas idempotentes con MERGE
- ✔ Uso de surrogate keys en dimensiones
- ✔ Enriquecimiento con lookup tables
- ✔ Manejo de valores nulos en métricas de delay

---

## ✈️ Dataset

El dataset contiene información de vuelos comerciales:

- Fechas de vuelo
- Aeropuertos de origen y destino
- Aerolíneas
- Horarios programados y reales
- Demoras por múltiples causas
- Distancia y duración del vuelo

---

## 📊 Casos de uso

Este modelo permite analizar:

- Puntualidad de aerolíneas
- Aeropuertos con mayor retraso
- Tendencias por tiempo (día/mes/año)
- Impacto del clima en vuelos
- Rutas más frecuentes y problemáticas

---

## 🧩 Tecnologías utilizadas

- SQL (Spark SQL / Delta Lake)
- Databricks Lakehouse
- Modelado dimensional (Kimball)
- MERGE para cargas idempotentes

---

## 📌 Estado del proyecto

- ✔ Bronze implementado
- ✔ Silver estructurado
- ✔ Dimensiones creadas
- ✔ Lookup integration completada
- ✔ Fact table en Star Schema

---

## 🚀 Próximos pasos

- Optimización de performance (partitioning / ZORDER)
- Creación de métricas GOLD (KPIs de puntualidad)
- Dashboards en Power BI / Tableau
- Data Quality layer