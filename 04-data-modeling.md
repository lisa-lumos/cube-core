# 4. Data modeling

```yml
cubes:
  - name: users
 
    measures:
      - name: count
        sql: id
        type: count
 
      - name: paying_count
        sql: id
        type: count
        filters: # an array, can apply any number of filters
          # it is best practice to use {CUBE}
          # to refer to itself, when ref its col
          - sql: "{CUBE}.paying = 'true'"
 
      - name: paying_percentage
        sql: "100.0 * {paying_count} / {count}" # ref existing measures
        type: number
        format: percent
 
    # ...
```
## Concepts
Cubes. Usually declared in separate files, with 1 cube per file. 
```yml
cubes:
  - name: orders
    # can use complex sql queries:
    sql: >
      SELECT *
      FROM orders, line_items
      WHERE orders.id = line_items.order_id
 
```

Can use joins to define relations between cubes. 

For large multi-tenancy configurations (100+ tenants), consider multi-cluster deployment. 

Views. 

## Syntax
## Dynamic data models
## refs
## Recipes
## dbt














