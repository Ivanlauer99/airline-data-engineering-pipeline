# ✈️ Airline Data Engineering Pipeline

[![Databricks](https://img.shields.io/badge/Databricks-Lakehouse-red?logo=databricks)](https://databricks.com)
[![Delta Lake](https://img.shields.io/badge/Delta%20Lake-Enabled-blue)](https://delta.io/)
[![SQL](https://img.shields.io/badge/SQL-Spark%20SQL-orange)](https://spark.apache.org/sql/)
[![Architecture](https://img.shields.io/badge/Architecture-Medallion-green)](https://www.databricks.com/glossary/medallion-architecture)

## 📋 Tabla de contenidos

* [Descripción del proyecto](#-descripción-del-proyecto)
* [Problema que resuelve](#-problema-que-resuelve)
* [Fuente del dataset](#-fuente-del-dataset)
* [Arquitectura Medallion](#-arquitectura-medallion)
* [Modelo de datos (Star Schema)](#-modelo-de-datos-star-schema)
* [Decisiones técnicas](#-decisiones-técnicas)
* [Estructura del proyecto](#-estructura-del-proyecto)
* [Requisitos previos](#-requisitos-previos)
* [Cómo ejecutar el pipeline](#-cómo-ejecutar-el-pipeline)
* [Casos de uso](#-casos-de-uso)
* [Próximos pasos](#-próximos-pasos)

---

## 🚀 Descripción del proyecto

✨ **Proyecto completado y funcional** ✨

Pipeline de Data Engineering end-to-end que implementa la **arquitectura Medallion (Bronze → Silver → Gold)** sobre Databricks Lakehouse para analizar datos de vuelos comerciales.

El proyecto transforma datos crudos en un **modelo dimensional tipo estrella (Star Schema)** optimizado para consultas analíticas, permitiendo responder preguntas de negocio sobre puntualidad, eficiencia operativa y desempeño de aerolíneas.

---

## 🎯 Problema que resuelve

### Desafío de negocio

Las aerolíneas y autoridades aeroportuarias necesitan:

* **Identificar patrones de demora** por aerolínea, aeropuerto y fecha
* **Optimizar operaciones** reduciendo tiempos muertos y cancelaciones
* **Mejorar la experiencia del pasajero** anticipando retrasos
* **Analizar tendencias históricas** para planificación estratégica

### Solución técnica

Este pipeline:

1. **Ingesta** datos crudos de vuelos en formato raw
2. **Limpia y normaliza** eliminando duplicados, corrigiendo tipos de datos
3. **Enriquece** con lookup tables (nombres de aerolíneas, información geográfica)
4. **Modela dimensionalmente** para consultas de alto rendimiento
5. **Habilita cargas incrementales** mediante operaciones idempotentes (MERGE)

---

## 📦 Fuente del dataset

**Dataset:** Airline On-Time Performance Data  
**Fuente:** [U.S. Department of Transportation - Bureau of Transportation Statistics](https://www.transtats.bts.gov/DL_SelectFields.aspx?gnoyr_VQ=FGJ&QO_fu146_anzr=b0-gvzr)  
**Cobertura:** Vuelos comerciales de Estados Unidos  
**Campos incluidos:**

* Información de vuelo (número, fecha, aerolínea)
* Aeropuertos de origen y destino
* Horarios programados vs. reales
* Demoras por categoría (meteorológica, seguridad, carrier, late aircraft)
* Distancia y duración del vuelo
* Estado de cancelación

**Nota:** Los datos utilizados son públicos y agregados. No contienen información personal de pasajeros.

---

## 🏗️ Arquitectura Medallion

Este proyecto sigue el patrón **Medallion Architecture** recomendado por Databricks:

```
┌─────────────────────────────────────────────────────────────┐
│                    🟤 BRONZE LAYER                          │
│                                                             │
│  • Datos crudos sin transformar                             │
│  • Carga completa desde fuente original                     │
│  • Mantiene estructura original (CSV → Delta)               │
│  • Tabla: bronze_flights                                    │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                    ⚪ SILVER LAYER                           │
│                                                             │
│  • Datos limpios y validados                                │
│  • Tipos de datos corregidos (DATE, INT, DECIMAL)          │
│  • Columnas normalizadas (snake_case)                       │
│  • Duplicados eliminados                                    │
│  • Tabla: silver_flights                                    │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                    🟡 GOLD LAYER                            │
│                                                             │
│  • Modelo dimensional (Star Schema)                         │
│  • Enriquecido con lookup tables                            │
│  • Optimizado para consultas analíticas                     │
│                                                             │
│  Dimensiones:                                               │
│    ✓ dim_airline                                            │
│    ✓ dim_airport                                            │
│    ✓ dim_date                                               │
│                                                             │
│  Hechos:                                                    │
│    ✓ fact_flights                                           │
└─────────────────────────────────────────────────────────────┘
```

### Ventajas de esta arquitectura

* **Trazabilidad:** Datos originales siempre disponibles en Bronze
* **Reusabilidad:** Silver puede alimentar múltiples modelos Gold
* **Calidad:** Cada capa agrega validaciones y transformaciones incrementales
* **Performance:** Gold optimizada para queries de negocio

---

## 🧠 Modelo de datos (Star Schema)

El modelo dimensional sigue el diseño **Star Schema** de Ralph Kimball:

```
                         ┌────────────────────┐
                         │     dim_date       │
                         │--------------------│
                         │ date_id (PK)       │
                         │ year               │
                         │ month              │
                         │ day                │
                         │ day_of_week        │
                         │ month_name         │
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
                           │ diverted             │
                           └──────────────────────┘
```

### Granularidad de la tabla de hechos

**Grano:** Un registro por vuelo individual  
**Clave natural:** `flight_date + airline_code + flight_number + origin + destination`  
**Surrogate key:** `flight_id` (generada con `monotonically_increasing_id()`)

### Dimensiones conformadas

* **dim_date:** Reutilizable en otros proyectos de análisis temporal
* **dim_airline:** Enriquecida con lookup de nombres completos
* **dim_airport:** Incluye información geográfica (ciudad, estado) para análisis regional

---

## 🛠️ Decisiones técnicas

### 1. ¿Por qué Star Schema y no Snowflake?

* **Simplicidad de queries:** JOINs más sencillos para analistas de negocio
* **Performance:** Menos JOINs = menor latencia en consultas
* **Desnormalización controlada:** Las dimensiones son pequeñas, la redundancia no es un problema

### 2. ¿Por qué granularidad a nivel de vuelo individual?

* **Flexibilidad analítica:** Permite agregaciones en cualquier dimensión (día, mes, año, aerolínea, ruta)
* **Drill-down:** Posibilidad de analizar vuelos específicos problemáticos
* **Balance storage/performance:** El volumen de datos no justifica pre-agregación

### 3. ¿Por qué operaciones MERGE idempotentes?

* **Cargas repetibles:** Re-ejecutar el pipeline no duplica datos
* **Actualizaciones incrementales:** Permite corregir datos históricos sin reconstruir todo
* **Confiabilidad:** El pipeline puede reiniciarse en cualquier punto sin riesgo

### 4. ¿Por qué surrogate keys en dimensiones?

* **Estabilidad:** Las claves de negocio pueden cambiar, las surrogate keys no
* **Performance:** Enteros son más eficientes para JOINs que strings
* **Slowly Changing Dimensions:** Facilita implementar SCD Type 2 en el futuro

### 5. ¿Por qué Delta Lake?

* **ACID transactions:** Garantiza consistencia en escrituras concurrentes
* **Time travel:** Permite auditar cambios históricos
* **MERGE support:** Operaciones UPSERT nativas
* **Optimizaciones:** ZORDER, file compaction automáticas

---

## 📂 Estructura del proyecto

```
airline-data-engineering-pipeline/
│
├── 01_bronze_layer.sql              # Ingesta de datos crudos
├── 02_silver_layer.sql              # Limpieza y normalización
├── 03_gold_dim_airline.sql          # Dimensión aerolíneas
├── 04_gold_dim_airport.sql          # Dimensión aeropuertos
├── 05_gold_dim_date.sql             # Dimensión calendario
├── 06_gold_fact_flights.sql         # Tabla de hechos principal
│
├── lookup_tables/
│   ├── airlines_lookup.csv          # Mapeo código → nombre aerolínea
│   └── airports_lookup.csv          # Información geográfica aeropuertos
│
├── README.md                        # Este archivo
└── .gitignore                       # Archivos excluidos del control de versiones
```

---

## 📋 Requisitos previos

* **Databricks Workspace** (Community Edition o superior)
* **Cluster de Spark** con:
  * Databricks Runtime 12.x o superior
  * Delta Lake habilitado (incluido por defecto)
* **Catálogo Unity Catalog** (recomendado) o Hive Metastore
* **Permisos:**
  * `CREATE SCHEMA` en el catálogo objetivo
  * `CREATE TABLE` en el schema
  * `READ FILES` desde la ubicación de datos

---

## 🚀 Cómo ejecutar el pipeline

### Paso 1: Configuración inicial

1. **Clona este repositorio** en tu workspace de Databricks:
   ```bash
   # Desde Repos en Databricks
   Git → Add Repo → URL: <tu-repo-url>
   ```

2. **Sube el dataset** a un volumen o DBFS:
   ```sql
   -- Opción A: Unity Catalog Volume
   CREATE VOLUME IF NOT EXISTS main.default.airline_data;
   -- Sube el CSV a /Volumes/main/default/airline_data/flights.csv
   
   -- Opción B: DBFS (alternativa)
   -- Sube a dbfs:/FileStore/airline_data/flights.csv
   ```

3. **Sube las lookup tables**:
   * `lookup_tables/airlines_lookup.csv` → mismo volumen/DBFS
   * `lookup_tables/airports_lookup.csv` → mismo volumen/DBFS

### Paso 2: Crear el catálogo y schema

```sql
-- Ejecuta este código una sola vez
CREATE CATALOG IF NOT EXISTS airline_pipeline;
USE CATALOG airline_pipeline;

CREATE SCHEMA IF NOT EXISTS bronze;
CREATE SCHEMA IF NOT EXISTS silver;
CREATE SCHEMA IF NOT EXISTS gold;
```

### Paso 3: Ejecutar el pipeline en orden

**Ejecuta los notebooks/scripts en este orden:**

```bash
# 1. Bronze Layer - Carga datos crudos
01_bronze_layer.sql

# 2. Silver Layer - Limpia y normaliza
02_silver_layer.sql

# 3. Gold Dimensions - Construye dimensiones
03_gold_dim_airline.sql
04_gold_dim_airport.sql
05_gold_dim_date.sql

# 4. Gold Facts - Construye tabla de hechos
06_gold_fact_flights.sql
```

### Paso 4: Validar los datos

```sql
-- Verificar conteo de registros en cada capa
SELECT 'bronze' AS layer, COUNT(*) AS records FROM airline_pipeline.bronze.bronze_flights
UNION ALL
SELECT 'silver', COUNT(*) FROM airline_pipeline.silver.silver_flights
UNION ALL
SELECT 'gold', COUNT(*) FROM airline_pipeline.gold.fact_flights;

-- Query de ejemplo: Top 5 aerolíneas con mayor demora promedio
SELECT 
    a.airline_name,
    ROUND(AVG(f.arrival_delay), 2) AS avg_delay_minutes,
    COUNT(*) AS total_flights
FROM airline_pipeline.gold.fact_flights f
INNER JOIN airline_pipeline.gold.dim_airline a ON f.airline_id = a.airline_id
WHERE f.cancelled = 0
GROUP BY a.airline_name
ORDER BY avg_delay_minutes DESC
LIMIT 5;
```

### Paso 5: (Opcional) Optimizar tablas

```sql
-- Optimizar tablas Gold para mejor performance
OPTIMIZE airline_pipeline.gold.fact_flights ZORDER BY (date_id, airline_id);
OPTIMIZE airline_pipeline.gold.dim_airline;
OPTIMIZE airline_pipeline.gold.dim_airport;
OPTIMIZE airline_pipeline.gold.dim_date;
```

### Automatización (opcional)

Para ejecutar el pipeline automáticamente:

1. **Databricks Workflows:**
   * Crea un Job con tasks encadenados
   * Configura schedule (diario/semanal)
   * Añade alertas en caso de fallo

2. **Delta Live Tables:**
   * Convierte los scripts a sintaxis DLT
   * Obtén lineage automático y data quality checks

---

## 📊 Casos de uso

Una vez completado el pipeline, puedes responder preguntas como:

### Análisis de puntualidad

* ¿Qué aerolínea tiene mejor promedio de puntualidad?
* ¿En qué mes del año hay más demoras?
* ¿Los vuelos del viernes tienen más retrasos que otros días?

### Análisis de aeropuertos

* ¿Qué aeropuertos de origen generan más cancelaciones?
* ¿Qué ciudad tiene la peor conectividad por demoras?
* ¿Hay patrones geográficos en las demoras (costa este vs. oeste)?

### Análisis de rutas

* ¿Qué ruta (origen-destino) tiene mayor demora promedio?
* ¿Las rutas largas (>2000 millas) son más propensas a demoras?
* ¿Qué combinación aerolínea-ruta es la más confiable?

### Análisis temporal

* ¿Hay estacionalidad en las cancelaciones?
* ¿Los fines de semana tienen mejor puntualidad?
* ¿Cómo evolucionó la puntualidad año tras año?

---

## 🧩 Tecnologías y herramientas

* **Databricks Lakehouse** - Plataforma unificada de datos
* **Apache Spark** - Motor de procesamiento distribuido
* **Delta Lake** - Storage layer con ACID transactions
* **SQL (Spark SQL)** - Lenguaje de transformación
* **Medallion Architecture** - Patrón de diseño de datos
* **Star Schema (Kimball)** - Modelado dimensional
* **MERGE (UPSERT)** - Operaciones idempotentes

---

## 📌 Estado del proyecto

### ✅ Proyecto 100% Completado

Todas las capas del pipeline están implementadas y funcionando:

#### Bronze Layer ✅
* Ingesta completa de datos crudos desde CSV
* Tabla `bronze_flights` creada con formato Delta
* Preservación de datos originales para auditoría

#### Silver Layer ✅
* Limpieza y normalización de datos completada
* Tipos de datos corregidos (DATE, INT, DECIMAL)
* Eliminación de duplicados implementada
* Columnas en formato snake_case
* Tabla `silver_flights` lista para consumo

#### Gold Layer ✅
* **Dimensiones creadas y populadas:**
  * ✅ `dim_airline` - Enriquecida con lookup table
  * ✅ `dim_airport` - Con información geográfica completa
  * ✅ `dim_date` - Dimensión calendario conformada
* **Tabla de hechos implementada:**
  * ✅ `fact_flights` - Star Schema completo con MERGE idempotente
  * ✅ Todas las foreign keys configuradas
  * ✅ Métricas de delay y distancia disponibles

#### Características adicionales ✅
* ✅ Lookup tables integradas (airlines y airports)
* ✅ Operaciones MERGE para cargas repetibles
* ✅ Surrogate keys en todas las dimensiones
* ✅ Pipeline completamente funcional y probado

**Estado:** 🎉 **PRODUCCIÓN - LISTO PARA USAR** 🎉

---

## 🔮 Posibles mejoras futuras

> **Nota:** El proyecto actual está **completo y funcional**. Las siguientes son mejoras opcionales para escalar o extender el pipeline en el futuro.

### Optimizaciones de rendimiento

* Implementar particionamiento en fact table por año/mes para queries más rápidas
* Añadir ZORDER adicionales en columnas de filtrado frecuente
* Implementar agregaciones pre-calculadas (cubos OLAP) para dashboards
* Habilitar Photon para acelerar queries SQL

### Calidad de datos

* Añadir data quality tests (Great Expectations / Delta Live Tables expectations)
* Implementar alertas automáticas para anomalías en los datos
* Validaciones de integridad referencial automatizadas
* Monitoreo de freshness de datos

### Escalabilidad arquitectónica

* Convertir a Delta Live Tables (DLT) para lineage automático y UI visual
* Implementar Slowly Changing Dimensions (SCD Type 2) para tracking histórico
* Añadir incremental loads con Change Data Capture (CDC)
* Particionar por aerolínea para queries paralelas más eficientes

### Nuevas métricas y análisis

* KPI: On-Time Performance (OTP) score por aerolínea
* KPI: Airport Delay Index (índice de retraso aeroportuario)
* KPI: Route Reliability Score (confiabilidad de rutas)
* Análisis de causas raíz de delays (weather, carrier, security, late aircraft)
* Predicción de delays con Machine Learning (regresión/clasificación)

### Visualización y reportes

* Dashboard interactivo en Databricks SQL
* Integración con Power BI / Tableau / Looker
* Reportes ejecutivos automatizados (PDF diario/semanal)
* Alertas en tiempo real para cancelaciones masivas
* API REST para consultas externas

### DevOps y automatización

* CI/CD con GitHub Actions / Azure DevOps
* Tests unitarios para transformaciones SQL
* Orquestación con Databricks Workflows
* Terraform para infraestructura como código
* Versionado de schema con Delta Lake time travel

---

## 📝 Notas importantes

* Este proyecto usa **datos públicos agregados** sin información personal
* Los scripts son **idempotentes**: puedes re-ejecutarlos sin duplicar datos
* Cada capa (Bronze/Silver/Gold) mantiene **full history** para auditoría
* El modelo está diseñado para **escalabilidad** (terabytes de datos)

---

## 📄 Licencia

Este proyecto es de código abierto y está disponible bajo la licencia MIT.

---

## 👤 Autor

**Ivan Lauer**  
📧 ivanlauer99@gmail.com  
🔗 [LinkedIn](https://www.linkedin.com/in/ivan-lauer-data/) | [GitHub](https://github.com/Ivanlauer99)

---

## 🙏 Agradecimientos

* Dataset proporcionado por **U.S. Department of Transportation**
* Arquitectura basada en mejores prácticas de **Databricks**
* Modelado dimensional siguiendo principios de **Ralph Kimball**

---

⭐ Si este proyecto te resultó útil, ¡dale una estrella en GitHub!