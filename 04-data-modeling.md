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

Views. Views do not define their own members - they ref cubes by join paths, and include cubes' members. Optionally, you can group members of a view into folders. 

Views do not define pre-aggs - they reuse pre-aggs from underlying cubes. 
```yml
views:
  - name: orders
 
    cubes:
      - join_path: base_orders
        includes:
          - status
          - created_date
          - total_amount
          - total_amount_shipped
          - count
          - average_order_value
 
      - join_path: base_orders.line_items.products
        includes:
          - name: name
            alias: product
 
      - join_path: base_orders.users
        prefix: true
        includes: "*"
        excludes:
          - company
```

Dimensions. A cube can have more than 1 PK dimension, so they are all parts of a composite key. 
```yml
cubes:
  - name: orders
    # ...
 
    dimensions:
      - name: id
        sql: id
        type: number
        # Here we explicitly let Cube know this field is the primary key
        # This is required for de-duplicating results when using joins
        primary_key: true
 
      - name: status
        sql: status
        type: string
```



## Syntax
## Dynamic data models
## refs
## Recipes
## dbt














