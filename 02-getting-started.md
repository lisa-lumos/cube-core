# 2. Getting started
You can start in one of 2 ways:
1. Use Cube Cloud (managed infra)
2. Deploy Cube to your own infrastructure with Docker

Example "model/cubes/orders.yml" file:
```yml
cubes:
  - name: orders
    sql_tables: ecom.orders
    public: false

    joins:
      - name: users
        sql: "{CUBE}.user_id = {users}.id"
        relationship: many_to_one
    
    dimensions:
      - name: status
        sql: status
        type: string

      - name: id
        sql: id
        type: number
        primary_key: true

      - name: created_at
        sql: created_at
        type: time

      - name: completed_at
        sql: completed_at
        type: time

    measures:
      - name: count
        type: count
    
      # calculates only completed orders
      - name: completed_count
        type: count
        filters:
          - sql: "{CUBE}.status = 'completed'"
      
      # a derived measure (based on existing measures)
      - name: completed_percentage
        type: number
        sql: "(100.0 * {CUBE}.completed_count / nullif({CUBE}.count, 0))"
        format: percent
```

Usually, cubes tend to be normalized entities, while views are denormalized entities, where you pick many measures/dimensions from multiple cubes as needed, to describe a business entity. 

Views are usually loaded in the "views" folder, and have a _view postfix. For example, "model/views/orders_view.yml":
```yml
views:
  - name: orders_view
    
    cubes: # include from other cubes
      - join_path: orders
        includes:
          - status
          - created_at
          - count
          - completed_count
          - completed_percentage

      - join_path: orders.users
        prefix: true
        includes:
          - city
          - age
          - state

```

"Semantic Layer Sync" programmatically connects to a BI tool to Cube, and creates/updates BI-specific entities that correspond to entities (cubes/views/measures/dimensions) within the data model in Cube. It is configured using the `semanticLayerSync` option in the "cube.js" config file. After running the sync, you should see the orders_view dataset that was created in BI. Alternatively, you can connect to Cube from BI tool, and create all the mappings manually. 

Recommend: Making cubes private, and only exposing views. 

























