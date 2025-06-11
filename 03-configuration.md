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























