# 5. Caching
Cube provides 2-level caching:
1. In-memory cache. Available by default. Act as a buffer for the database, when concurrent users query the same data. 
2. Pre-aggregations. Need explicit configurations to activate. A layer of aggregated data built/refreshed by Cube. 

For in-mem cache, the cache key is a generated SQL statement, with any existing query-dependent pre-aggs. Upon receiving an in-coming request, Cube first checks the cache using this key, if an existing value is present in the cache, and the refresh_key val for the query hasn't changed, the cached value will be returned. 

## Getting started with Pre-aggregations
Depends on the datasource, Cube might first builds pre-aggs as tables in the source db, then exports this data into the pre-agg storage, so Cube may require write access to the pre-agg schema in the database. 

Cube defines a refresh_key for each cube, to assess if the data needs to be refreshed. By default, Cube checks this refresh key every 10 secs. 

If background refresh is disabled, Cube will refresh the cache during query execution, which may lead to delays. So recommend to always enable background refresh. 


## Using pre-aggs



## Matching pre-aggs



## Refreshing pre-aggs



## Lambda pre-aggs



## Running in prod



## Recipes




