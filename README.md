# DBT TPCH Snowflake Projects

This project is for building an ELT process to load data from Snowflake TPCH (Simple Data) and transform the data using DBT to make a staging and marts layers. The technologies that use within this project are Snowflake and DBT.

# Project Content

1. Setup Snowflake (Warehouse, Database, Schema and Role)
2. Initialize DBT Project
3. Transform Data (Staging, Marts)

# Setup Snowflake

Follow my sql command below to setup Warehouse, Database, Schema and Role within Snowflake

```
USE ROLE ACCOUNTADMIN;

CREATE WAREHOUSE DBT_WH WITH
WAREHOUSE_SIZE = 'X-SMALL'
AUTO_SUSPEND = 300
AUTO_RESUME = TRUE;

CREATE OR REPLACE DATABASE DBT_DB;
CREATE ROLE DBT_ROLE;

SHOW GRANTS ON WAREHOUSE DBT_WH;

GRANT USAGE ON WAREHOUSE DBT_WH TO ROLE DBT_ROLE;
GRANT ROLE DBT_ROLE TO USER <your user>;
GRANT ALL ON DATABASE DBT_DB TO ROLE DBT_ROLE;

USE ROLE DBT_ROLE;

CREATE OR REPLACE SCHEMA DBT_DB.DBT_SCHEMA;
```

# Initilize DBT Project

To initialize the project, kinldy you can make and activate your virtual env python adn then create dbt directory project.

- Create VE

```
python -m venv dbt_venv
```

- Instal the dbt core and dbt-snowflake

```
python -m pip install dbt-core;
python -m pip install dbt-snowflake
```

- Check your dbt version

```
dbt --version
```

- Initialize dbt project

```
dbt init
```

- Configure your dbt init project by filling the form that prompted
- Named the project 'dbt_snowflake' and then enter
- Chose Snowflake as adapter by fill 1 in the promp
- Enter your username
- Enter your password for username
- Role : DBT_ROLE
- Warehouse : DBT_WH
- Database : DBT_DB
- Schema : DBT_SCHEMA
- Threads : 10

Test your configuration above by running this command, but makesure that you are in the file of project.

```
cd dbt_snow
```

Run the command dbt config after you are in the dbt_snow directory

```
dbt config
```

- Configure your dbt_project.yml
  Let's configure the dbt_project.yml to make 2 models (staging and marts)

```
models:
  dbt_snowflake_pro:
    # Config indicated by + and applies to all files under models/example/
    staging:
      materialized: view
      snowflake_warehouse: dbt_wh
    marts:
      materialized: table
      snowflake_warehouse: dbt_wh
```

- Configure the source data
  Create a new file named tpch_sources.yml in your staging directory

```
version: 2

sources:
  - name: tpch
    database: snowflake_sample_data
    schema: tpch_sf1
    tables:
      - name: orders
        columns:
          - name: o_orderkey
            tests:
              - unique
              - not_null
      - name: lineitem
        columns:
          - name: l_orderkey
            tests:
              - relationships:
                  to: source('tpch', 'orders')
                  field: o_orderkey
```

# Testing Using Generic and SIngular test

- Generic test (is a parameterized query that accepts arguments)

```
models:
  - name: fct_orders
    columns:
      - name: order_key
        tests:
          - unique
          - not_null
          - relationships:
              to: ref('stg_tpch_orders')
              field: order_key
              severity: warn
      - name: status_code
        test:
          - accepted_values:
              values: ["P", "O", "F"]
```

- Singular test (testing in its simplest form)

```
SELECT
    *
FROM
    {{ ref('fct_orders') }}
WHERE
    date(order_date) > CURRENT_DATE()
    OR date(order_date) < date('1990-01-01')
```

After you finish building model within silver and gold, you can execute the command below

```
dbt run
```

# Create Project Documentation

Generating documentation in dbt is a crucial step in managing and maintaining a healthy data analytics workflow. It improves collaboration, understanding, and the overall quality of your data transformation processes.

To generate the project documentation, execute this command below

```
dbt docs generate
```

# Dbt Web Server (Lineage Graph)

Run the command below, to see the documentation within catalog and lineage graph of your model design befor

```
dbt docs serve --port 8001
```

# Thank you
