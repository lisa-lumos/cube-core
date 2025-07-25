# 5. Caching
Cube provides 2-level caching:
1. In-memory cache. Available by default. Act as a buffer for the database, when concurrent users query the same data. 
2. Pre-aggregations. Need explicit configurations to activate. A layer of aggregated data built/refreshed by Cube. 

For in-mem cache, the cache key is a generated SQL statement, with any existing query-dependent pre-aggs. Upon receiving an in-coming request, Cube first checks the cache using this key, if an existing value is present in the cache, and the refresh_key val for the query hasn't changed, the cached value will be returned. 

Depends on the datasource, Cube might first builds pre-aggs as tables in the source db, then exports this data into the pre-agg storage, so Cube may require write access to the pre-agg schema in the database. 

Cube defines a refresh_key for each cube, to assess if the data needs to be refreshed. By default, Cube checks this refresh key every 10 secs. 

If background refresh is disabled, Cube will refresh the cache during query execution, which may lead to delays. So recommend to always enable background refresh. 

Cache types:
1. Pre-agg in Cube Store. Did not reach database.
2. Pre-agg in Cube Store with a suboptimal query plan. Did not reach database, might use index to enhance perf.
3. Pre-agg in the data source. Consider use Cube Store as pre-agg storage. 
4. Im-memory cache. The fastest query retrieval method, require the exact same query ran very recently. 
5. No cache. 

## Getting started with Pre-aggregations
A pre-agg is a condensed version of the source data. It can reduce the size of the dataset significantly, and can be used by qualified subsequent queries. A cube can have many pre-aggs. 

Pre-aggs are stored in Cube Store, which is a dedicated pre-agg storage layer. 

Example:
```yml
cubes:
  - name: orders
    sql_table: orders
 
    measures:
      - name: count
        type: count
 
    dimensions:
      - name: id
        sql: id
        type: number
        primary_key: true
 
      - name: status
        sql: status
        type: string
 
      - name: completed_at
        sql: completed_at
        type: time
 
    pre_aggregations:
      - name: order_statuses # used to populate a drop-down, showing all status choices: 
        dimensions:
          - status
      - name: orders_by_completed_at # 1 row for each mo
        measures:
          - count
        time_dimension: completed_at
        granularity: month
        refresh_key:
          every: 12 hour
          sql: select max(created_at) from orders

```

## Using pre-aggs
When executing a query, Cube will try to match and fulfill it with a pre-agg first, if no match found, it will then query the upstream database. 

`refresh_key`: default value is 1hr. When `every` and `sql` are used together, Cube will run the sql based on the interval, if the query returns new results, the pre-agg will be refreshed. 

Partitioning. Can config `update_window` to incrementally refresh only the last set of partitions. Any rollup pre-agg can be partitioned by time using `partition_granularity`. e.g.:
```yml
cubes:
  - name: orders
    # ...
 
    pre_aggregations:
      - name: category_and_date
        measures:
          - count
          - revenue
        dimensions:
          - category
        time_dimension: created_at
        granularity: day
        partition_granularity: month
        refresh_key:
          every: 1 day
          incremental: true
          update_window: 3 months
```

Every physical partition is stored as a parquet file. The partition key is same as index sorting key. So when time dimension partitioning is used together with an index, the data will first be partitioned by time, then sorted by index. 

In general, use less partitions, as long as you are happy with the query speed. 



## Matching pre-aggs



## Refreshing pre-aggs



## Lambda pre-aggs



## Running in prod



## Recipes




