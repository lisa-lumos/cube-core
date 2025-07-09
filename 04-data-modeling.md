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

Joins. In Cube, all joins are left joins. 
```yml
cubes:
  - name: orders
    # ...
 
    joins:
      - name: line_items
        sql: "{CUBE}.id = {line_items.order_id}"
        relationship: many_to_one
```

Segments. Named filters that live inside the data model. 
```yml
cubes:
  - name: orders
    # ...
 
    segments:
      - name: only_completed
        sql: "{CUBE}.status = 'completed'"
```

Pre-aggregations. Accelerate frequently used queries, and keep the cache up-to-date. 
```yml
cubes:
  - name: orders
    # ...
 
    pre_aggregations:
      - name: main
        measures:
          - count
        dimensions:
          - status
        time_dimension: created_at
        granularity: day
```

Proxy dimensions: calculated dims from existing dims. 
```yml
cubes:
  - name: users
    sql_table: users
 
    dimensions:
      - name: initials
        sql: "SUBSTR(first_name, 1, 1)"
        type: string
 
      - name: last_name
        sql: "UPPER(last_name)"
        type: string
 
      - name: full_name
        sql: "{initials} || '. ' || {last_name}" # calculated dim
        type: string
```

```yml
cubes:
  - name: orders
    sql: >
      SELECT 1 AS id, 1 AS user_id UNION ALL
      SELECT 2 AS id, 1 AS user_id UNION ALL
      SELECT 3 AS id, 2 AS user_id
 
    dimensions:
      - name: id
        sql: id
        type: number
        primary_key: true
 
      - name: user_name
        sql: "{users.name}" # using dim from a diff cube
        type: string
 
    measures:
      - name: count
        type: count
 
    joins:
      - name: users
        sql: "{users}.id = {orders}.user_id"
        relationship: one_to_many
 
  - name: users
    sql: >
      SELECT 1 AS id, 'Alice' AS name UNION ALL
      SELECT 2 AS id, 'Bob'   AS name
 
    dimensions:
      - name: name
        sql: name
        type: string
```

Time dimension granularity. 
```yml
cubes:
  - name: users
    sql: >
      SELECT '2025-01-01T00:00:00Z' AS created_at UNION ALL
      SELECT '2025-02-01T00:00:00Z' AS created_at UNION ALL
      SELECT '2025-03-01T00:00:00Z' AS created_at
 
    dimensions:
      - name: created_at
        sql: created_at
        type: time
 
        granularities: # prepare a granularity
          - name: sunday_week
            interval: 1 week
            offset: -1 day
 
      - name: created_at__year
        sql: "{created_at.year}" # an cube existing granularity
        type: time
 
      - name: created_at__sunday_week
        sql: "{created_at.sunday_week}" # assign it to a dim
        type: time
```

Subquery dimensions. If you have 2 cubes joined, you can use a subquery dimension to bring a measure from 2nd cube to 1st cube, as a dimension. 
```yml
cubes:
  - name: orders
    sql: >
      SELECT 1 AS id, 1 AS user_id UNION ALL
      SELECT 2 AS id, 1 AS user_id UNION ALL
      SELECT 3 AS id, 2 AS user_id
 
    dimensions:
      - name: id
        sql: id
        type: number
        primary_key: true
 
    measures:
      - name: count
        type: count
 
    joins:
      - name: users
        sql: "{users}.id = {orders}.user_id"
        relationship: one_to_many
 
  - name: users
    sql: >
      SELECT 1 AS id, 'Alice' AS name UNION ALL
      SELECT 2 AS id, 'Bob'   AS name
 
    dimensions:
      - name: id
        sql: id
        type: number
        primary_key: true
 
      - name: name
        sql: name
        type: string
 
      - name: order_count # subquery dimension
        sql: "{orders.count}"
        type: number
        sub_query: true # note this
 
    measures:
      - name: avg_order_count
        sql: "{order_count}"
        type: avg
```

Multi-stage calculations. Calculated in 2 or more stages, and often involve manipulations, or already aggregated data. Each stage results in one/more CTEs in the generated SQL query. They are currently not accelerated by pre-aggs. 

rolling window, period-to-date, prior date, fixed dimension, ranking, etc. 

Extension. Can create a child cube/view that reuses all declared members of a parent cube/view. Reusable. The usual pattern is to extract commom measures/dims/joins from the parent, and extend from it. 

Polymorphic cubes. 

Data blending. Literally a union. 

Joins. Only need to be defined from one direction. 3 types of joins:
- one_to_one
- one_to_many
- many_to_one

"model/cubes/customers.yml":
```yml
cubes:
  - name: customers
    # ...
 
    joins:
      - name: orders
        relationship: one_to_many
        sql: "{CUBE}.id = {orders.customer_id}"
```

You can swap join tables as needed. 

When generating SQL queries, Cube uses PKs (defined as a dimension in the cube) to avoid fanouts. 

## Syntax
Recommend: cubes live in the "models/cubes" folder, and views live in the "models/views" folder. 

You can use yml/js to define models, or mixed. yml and snake case is recommended. 

dimensions/measures can use sql expressions, such as `sql: "upper(status)"`. 

Use `{col_name}` to refer to dimensions/measures defined in the cube itself. 

Use `{time_dimension_name.granularity}` to change granularity. I think can use sql too. 

Use `{cube_name.col_name}` to avoid ambiguity when joining. 

Use `{CUBE}.col_name` to refer to itself. 

Referencing another cube in the dim definition instructs Cube to make an implicit join to that cube. 

Curly braces are optional outside of sql/sql_table keys. 

## Dynamic data models
Cube data models support jinja macros. 

Dependencies can be listed in "requirements.txt" file. 

## Recipes
## dbt














