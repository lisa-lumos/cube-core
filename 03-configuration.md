# 3. Configuration
## Environment variables
For static configs, which are not supposed to change while Cube deployment is running. 

Such as CUBEJS_DATASOURCES. 

For Cube Core, you can set them in any way supported by Docker, such as a ".env" file, or the `environment` option in the "docker-compose.yml" file. 

For example, for redshift cluster, required env vars are:
- CUBEJS_DB_HOSE
- CUBEJS_DB_NAME
- CUBEJS_DB_USER
- CUBEJS_DB_PASS

## Configuration options
For dynamic configs, that are applied dynamically while Cube deployment is running. Take precedence over environment vars. 

Such as query_rewrite. 

Can be defined either using "cube.py" file, or "cube.js" file, in the root folder of a Cube project. 

A minimal correct "cube.js" file:
```js
module.exports = {}
```

An example correct "cube.js" file:
```js
module.exports = {
  // Base path for the REST API
  basePath: '/cube-api',
 
  // Inspect, modify, or restrict every query
  queryRewrite: (query, { securityContext }) => {
    if (securityContext.order_id) {
      query.filters.push({
        member: 'orders_view.id',
        operator: 'equals',
        values: [securityContext.order_id]
      })
    }
    return query
  }
}
```

When in doubt, use Python.

When using Docker, mount the config file and the data model folder to "/cube/conf" in the container. 

Cube can be run in an insecure dev mode, by setting CUBEJS_DEV_MODE = true. 

## Pre-Aggregation support
By default, AWS Redshift uses batching to build pre-aggs. 

Require the Redshift user to have ownership of a schema. By default, the schema name is `prod_pre_aggregations`. It can be set using the `pre_aggregations_schema` configuration option. 

For improved pre-agg performance with large datasets, enable export bucket functionality, with these env vars:
- CUBEJS_DB_EXPORT_BUCKET_type=s3
- CUBEJS_DB_EXPORT_BUCKET=..
- CUBEJS_DB_EXPORT_BUCKET_AWS_KEY=...
- CUBEJS_DB_EXPORT_BUCKET_AWS_SECRET=...
- CUBEJS_DB_EXPORT_BUCKET_AWS_REGION=...

## Viz tools - Tableau
Not recommended: using Tableau extracts when connecting to Cube. Use pre-aggs instead. 

## Concurrency
Query queue:
- dedupes queries, insulate upstream data sources from query spikes
- allows to run queries against data sources concurrently, for performance. 

For Amazon Redshift, default concurrency is 5. For Snowflake, it is 8. 

## Multitenancy
For cases where 
- you need to serve different datasets for different users/tenants that are not related to each other. 
- or, users need to access the same data, but from different databases. 

`securityContext` can include all the necessary data to identify a user/org/app, etc. 

When you want to define row-level security within the same db for different users, use `queryRewrite`. e.g.:
```js
module.exports = {
  queryRewrite: (query, { securityContext }) => {
    if (securityContext.categoryId) {
      query.filters.push({
        member: "products.category_id",
        operator: "equals",
        values: [securityContext.categoryId]
      })
    }
    return query
  }
}
```
for cube:
```yml
cubes:
  - name: products
    sql_table: products
```















