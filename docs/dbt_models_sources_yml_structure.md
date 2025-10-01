# Sample structure

````text
version: 2

sources:
  - name: ___
    database: ___    # explicitly points to ____ database
    tables:
      - name: dataset___tablename1
      - name: dataset___tablename2
      - name: dataset___tablename3
      - name: dataset___tablename4
      - name: dataset___tablename5
````
*aaand* so on and so forth~

- **note:** this is just a sample template, wherein we connected to a ClickHouse server
- **use when:** before running dbt, ensure that table names in sources.yml are updated! *or else*
