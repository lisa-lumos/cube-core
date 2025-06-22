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

Dimensions can be organized into hierarchies. Common Cube dimension types: time/string/number/boolean. 
```yml
cubes:
  - name: orders
    # ...
 
    dimensions:
      - name: created_at
        sql: created_at
        type: time
        # You can use this time dimension with all default granularities:
        # year, quarter, month, week, day, hour, minute, second
 
      - name: completed_at
        sql: completed_at
        type: time
        # You can use this time dimension with all default granularities
        # and an additional custom granularity defined below
        granularities:
          - name: fiscal_year_starting_on_february_01
            interval: 1 year
            offset: 1 month
```

Time dimensions are essential to have performance boosts, such as partitioned pre-aggs and incremental refreshes. 

Measures. Measure types in cube: avg/boolean/count/count_distinct/count_distinct_approx/max/min/number/string/sum/time. 

Additivity determines whether measure values can be further aggregated. 

Measures that do not ref other measures are leaf measures. Whether a query contains only additive leaf measures affect pre-agg matching. 

## Syntax
## Dynamic data models
## refs
## Recipes
## dbt














