# 5. Caching
Cube provides 2-level caching:
1. In-memory cache. Available by default. Act as a buffer for the database, when concurrent users query the same data. 
2. Pre-aggregations. Need explicit configurations to activate. A layer of aggregated data built/refreshed by Cube. 

For in-mem cache, the cache key is a generated SQL statement, with any existing query-dependent pre-aggs. Upon receiving an in-coming request, Cube first checks the cache using this key, if an existing value is present in the cache, and the refresh_key val for the query hasn't changed, the cached value will be returned. 

## Getting started with Pre-aggregations



## Using pre-aggs



## Matching pre-aggs



## Refreshing pre-aggs



## Lambda pre-aggs



## Running in prod



## Recipes




