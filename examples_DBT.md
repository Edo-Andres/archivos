# 5 Proyectos Básicos para Aprender DBT

> 💡 **Stack base recomendado (Local Windows 10):**
> - **DBT Core** + **DuckDB** (sin servidor, perfecto para aprender)
> - **Docker** para PostgreSQL cuando quieras escalar
> - **Python** para generar/obtener datos
> - **GCP BigQuery** como destino opcional en la nube

---

## ⚙️ Setup Inicial (Común para todos los proyectos)

```bash
# 1. Crear entorno virtual
python -m venv dbt-env
dbt-env\Scripts\activate  # Windows

# 2. Instalar dbt con DuckDB (más fácil para empezar)
pip install dbt-duckdb

# O con PostgreSQL (más profesional)
pip install dbt-postgres

# 3. Iniciar proyecto
dbt init mi_proyecto
```

---

## 🛒 Proyecto 1: Pipeline de Ventas E-Commerce
**Nivel:** ⭐ Principiante puro  
**Concepto clave:** Estructura de capas (staging → marts)

### ¿Qué aprenderás?
- Estructura básica de un proyecto DBT
- Modelos staging y marts
- Tests básicos (`unique`, `not_null`)
- `dbt run`, `dbt test`, `dbt docs`

### Flujo
```
CSV (datos de ventas)
    → Python genera datos falsos
        → DuckDB/PostgreSQL
            → DBT transforma
                → Reporte final
```

### Paso 1: Generar datos con Python
```python
# generate_data.py
import pandas as pd
import numpy as np
from faker import Faker
import random

fake = Faker('es_MX')

# Generar órdenes
orders = []
for i in range(1000):
    orders.append({
        "order_id": i + 1,
        "customer_id": random.randint(1, 200),
        "product_id": random.randint(1, 50),
        "amount_cents": random.randint(500, 50000),
        "status": random.choice(["completed", "returned", "pending"]),
        "created_at": fake.date_between("-1y", "today")
    })

pd.DataFrame(orders).to_csv("data/raw_orders.csv", index=False)
print("✅ Datos generados!")
```

### Paso 2: Estructura DBT
```
models/
├── staging/
│   └── stg_orders.sql         # Limpia y renombra columnas
├── intermediate/
│   └── int_orders_completed.sql  # Solo órdenes completadas
└── marts/
    └── fct_ventas_mensuales.sql  # Agregación mensual
```

### Modelos SQL
```sql
-- models/staging/stg_orders.sql
SELECT
    order_id,
    customer_id,
    product_id,
    amount_cents / 100.0 AS amount_usd,  -- Convertir centavos
    status,
    created_at::date AS order_date
FROM {{ source('raw', 'orders') }}
WHERE order_id IS NOT NULL

-- models/marts/fct_ventas_mensuales.sql
SELECT
    DATE_TRUNC('month', order_date) AS mes,
    COUNT(order_id)                 AS total_ordenes,
    SUM(amount_usd)                 AS revenue_total,
    AVG(amount_usd)                 AS ticket_promedio
FROM {{ ref('int_orders_completed') }}
GROUP BY 1
ORDER BY 1
```

### Tests mínimos
```yaml
# models/staging/stg_orders.yml
version: 2
models:
  - name: stg_orders
    columns:
      - name: order_id
        tests:
          - unique
          - not_null
      - name: status
        tests:
          - accepted_values:
              values: ['completed', 'returned', 'pending']
```

### 🌐 Versión GCP
```yaml
# profiles.yml - BigQuery
mi_proyecto:
  outputs:
    dev:
      type: bigquery
      method: oauth
      project: tu-proyecto-gcp
      dataset: dbt_dev
      location: US
```

---

## 💰 Proyecto 2: Tracker de Gastos Personales
**Nivel:** ⭐⭐ Básico-Intermedio  
**Concepto clave:** Seeds, macros y documentación

### ¿Qué aprenderás?
- Usar **seeds** (archivos CSV dentro de DBT)
- Crear tu primera **macro**
- Generar **documentación** con `dbt docs`
- Usar `dbt_utils`

### Flujo
```
CSV de gastos (manual o exportado de banco)
    → DBT Seeds
        → Modelos de transformación
            → Dashboard de gastos
```

### Paso 1: Seed de categorías
```csv
-- seeds/categorias.csv
categoria_id,nombre,tipo
1,Supermercado,Necesidad
2,Netflix,Entretenimiento
3,Renta,Necesidad
4,Restaurante,Ocio
5,Gasolina,Transporte
```

### Paso 2: Tu primera Macro
```sql
-- macros/clasificar_gasto.sql
{% macro clasificar_gasto(monto) %}
    CASE
        WHEN {{ monto }} < 100  THEN 'Bajo'
        WHEN {{ monto }} < 500  THEN 'Medio'
        WHEN {{ monto }} < 2000 THEN 'Alto'
        ELSE 'Muy Alto'
    END
{% endmacro %}
```

### Paso 3: Modelo principal
```sql
-- models/marts/fct_gastos_mensuales.sql
SELECT
    DATE_TRUNC('month', fecha) AS mes,
    c.tipo                     AS tipo_gasto,
    SUM(g.monto)               AS total_gastado,
    COUNT(*)                   AS num_transacciones,
    {{ clasificar_gasto('AVG(g.monto)') }} AS nivel_promedio
FROM {{ ref('stg_gastos') }}       g
JOIN {{ ref('categorias') }}       c USING (categoria_id)
GROUP BY 1, 2
```

```bash
# Comandos clave de este proyecto
dbt seed           # Carga los CSV
dbt run            # Ejecuta modelos
dbt docs generate  # Genera documentación
dbt docs serve     # Abre documentación en browser
```

---

## 🐳 Proyecto 3: Pipeline con Docker + PostgreSQL
**Nivel:** ⭐⭐ Básico-Intermedio  
**Concepto clave:** Entorno profesional con Docker

### ¿Qué aprenderás?
- Configurar **PostgreSQL con Docker**
- Cargar datos con Python (**pandas + SQLAlchemy**)
- Usar **variables de entorno**
- Correr DBT contra una BD real

### docker-compose.yml
```yaml
version: '3.8'
services:

  postgres:
    image: postgres:15
    container_name: dbt_postgres
    environment:
      POSTGRES_USER: dbt_user
      POSTGRES_PASSWORD: dbt_pass
      POSTGRES_DB: dbt_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

```bash
# Levantar la BD
docker-compose up -d

# Verificar que está corriendo
docker ps
```

### Cargar datos con Python
```python
# load_data.py
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine(
    "postgresql://dbt_user:dbt_pass@localhost:5432/dbt_db"
)

# Cargar datasets públicos (ej: COVID, clima, etc.)
df = pd.read_csv("data/raw_data.csv")
df.to_sql("raw_data", engine, schema="public",
          if_exists="replace", index=False)

print("✅ Datos cargados en PostgreSQL!")
```

### profiles.yml con variables de entorno
```yaml
mi_proyecto:
  outputs:
    dev:
      type: postgres
      host: localhost
      port: 5432
      user: "{{ env_var('DBT_USER') }}"
      password: "{{ env_var('DBT_PASSWORD') }}"
      dbname: dbt_db
      schema: dbt_dev
```

```bash
# Windows - setear variables de entorno
set DBT_USER=dbt_user
set DBT_PASSWORD=dbt_pass
dbt run
```

---

## 📊 Proyecto 4: Análisis de Dataset Público (COVID / Clima)
**Nivel:** ⭐⭐⭐ Intermedio  
**Concepto clave:** Sources, Snapshots y tests avanzados

### ¿Qué aprenderás?
- Configurar **sources** con freshness checks
- Crear **snapshots** (historial de cambios)
- Instalar y usar **paquetes dbt** (`dbt_utils`, `dbt_expectations`)
- Tests personalizados

### Dataset sugerido
```python
# download_data.py - Datos COVID públicos
import pandas as pd

url = "https://covid.ourworldindata.org/data/owid-covid-data.csv"
df = pd.read_csv(url, usecols=[
    'iso_code', 'location', 'date',
    'new_cases', 'new_deaths',
    'total_vaccinations', 'population'
])
df.to_csv("data/covid_raw.csv", index=False)
```

### Sources con Freshness
```yaml
# models/sources.yml
version: 2
sources:
  - name: raw
    tables:
      - name: covid_data
        freshness:
          warn_after: {count: 7, period: day}
          error_after: {count: 30, period: day}
        loaded_at_field: date
```

### Snapshot (historial de cambios)
```sql
-- snapshots/covid_snapshot.sql
{% snapshot covid_snapshot %}
    {{
        config(
            target_schema='snapshots',
            unique_key='iso_code || date',
            strategy='check',
            check_cols=['total_vaccinations', 'new_cases']
        )
    }}
    SELECT * FROM {{ source('raw', 'covid_data') }}
{% endsnapshot %}
```

### packages.yml
```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.1.1
  - package: calogica/dbt_expectations
    version: 0.10.1
```

### Tests avanzados con dbt_expectations
```yaml
columns:
  - name: new_cases
    tests:
      - dbt_expectations.expect_column_values_to_be_between:
          min_value: 0
          max_value: 1000000
      - dbt_expectations.expect_column_to_exist
```

```bash
dbt deps              # Instalar paquetes
dbt snapshot          # Ejecutar snapshots
dbt source freshness  # Verificar frescura de datos
dbt build             # Todo junto
```

---

## 🔄 Proyecto 5: Pipeline E2E con CI/CD (El más completo)
**Nivel:** ⭐⭐⭐ Intermedio-Avanzado  
**Concepto clave:** Pipeline completo profesional

### ¿Qué aprenderás?
- **GitHub Actions** para CI/CD
- **Modelos incrementales** (`incremental`)
- Estructura profesional completa
- Despliegue a **GCP BigQuery**

### Modelo Incremental
```sql
-- models/marts/fct_eventos_incremental.sql
{{
    config(
        materialized='incremental',
        unique_key='event_id',
        on_schema_change='sync_all_columns'
    )
}}

SELECT
    event_id,
    user_id,
    event_type,
    event_timestamp,
    DATE(event_timestamp) AS event_date
FROM {{ ref('stg_eventos') }}

{% if is_incremental() %}
    -- Solo procesar datos nuevos
    WHERE event_timestamp > (
        SELECT MAX(event_timestamp) FROM {{ this }}
    )
{% endif %}
```

### GitHub Actions (CI/CD)
```yaml
# .github/workflows/dbt_ci.yml
name: DBT CI Pipeline

on:
  pull_request:
    branches: [main]

jobs:
  dbt-ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install DBT
        run: pip install dbt-bigquery  # o dbt-postgres

      - name: Install packages
        run: dbt deps

      - name: Run DBT Build
        run: dbt build --select state:modified+
        env:
          DBT_USER: ${{ secrets.DBT_USER }}
          DBT_PASSWORD: ${{ secrets.DBT_PASSWORD }}
```

---

## 🗺️ Ruta de Aprendizaje Sugerida

```
Semana 1-2     Semana 3-4       Semana 5-6      Semana 7-8
    │               │                │               │
Proyecto 1  →  Proyecto 2  →  Proyecto 3  →  Proyecto 4
(Capas DBT)   (Seeds/Macros)  (Docker+PG)   (Sources/Snap)
                                                    │
                                               Proyecto 5
                                             (CI/CD + GCP)
```

## 📦 Comparativa de Proyectos

| # | Proyecto | Warehouse | Concepto Clave | Tiempo Est. |
|---|----------|-----------|----------------|-------------|
| 1 | E-Commerce | DuckDB | Capas básicas | 1 semana |
| 2 | Gastos | DuckDB | Seeds + Macros | 1 semana |
| 3 | Docker + PG | PostgreSQL | Entorno real | 1-2 semanas |
| 4 | COVID/Público | PostgreSQL | Snapshots | 2 semanas |
| 5 | E2E CI/CD | PG o BigQuery | Pipeline completo | 2-3 semanas |

> 🚀 **Mi recomendación:** Empieza con el **Proyecto 1 con DuckDB** ya que no necesitas instalar ningún servidor de base de datos, es todo en archivos locales, y te permite enfocarte 100% en aprender DBT sin distracciones de infraestructura. ... https://docs.getdbt.com/docs/build/sources?version=1.12

