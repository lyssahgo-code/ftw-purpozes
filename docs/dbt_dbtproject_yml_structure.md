# Sample structure

````text
name: "ex_dataset"
version: "1.0"
config-version: 2

profile: clickhouse_ftw
model-paths: ["models"]
macro-paths: ["macros"]  

models:
  ex_dataset:
    clean:
      +schema: clean
      +materialized: table
    mart:
      +schema: mart
      +materialized: view
