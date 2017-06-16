---
title: InfluxDB
tags: technology, databases
---

**InfluxDB** is an open-source database written in Go to handle time series data and uses a similar query language to SQL.

### Time Series Data
A sequence of data point, typically consisting of successive measurements made from the same source over a time interval.

Regular data occurs at a fixed interval, while irregular data occurs sporadically.

### Examples of Time Series Data
1. Number of times a hashtag was used per hour.
2. Weather data collected over time: Temperature, atmosphere, air pressure.
3. Disk-usage, short-term load, or other devops data.

### Other Time Series Databases
- OpenTSDB
- KDB+
- Graphite
- Riak TS

### InfluxDB's Data Model
- **measurement**: containers for other parts of the data model
- **tags**: key/value pairs that are indexed, so they're a bit more performant than fields; values may only be strings
- **tag set**: comma-delineated list of all tags on an entry
- **fields**: key/value pairs where values can be stored as floats, ints, strings, or bools
- **field set**: comma-delineated list of fields on an entry
- **timestamp**: unix epoch timestamp marking x-axis data

### The Line Protocol
InfluxDB uses its own protocol to accept new entries into the database. The format is as follows: `measurement,tag_set field_set timestamp`. For an entry into the measurement `stock_price` with the tag set of `ticker=A,market=NASDAQ`, you would use the following line:

```
stock_price,ticker=A,market=NASDAQ price=177.03 1445299200000000000
```

A series in InfluxDB is represented by the measurement and tag set (ie. `stock_price,ticker=A,market=NASDAQ`). If you have too many series, it can become a scaling issue, so structuring data appropriately is key.

The timestamp is optional to pass in line protocol and will default to the server's local time.

### Example Metaqueries
- `SHOW DATABASES`
- `USE dbname`
- `SHOW SERIES`
- `SHOW FIELD KEYS`

### Example Queries
- `SELECT * FROM ...`
- `SELECT MEAN(value) FROM ...`
- `SELECT ... GROUP BY time(10m)`
- `DELETE FROM ...`
- `SELECT <field_key> FROM ...`
- `SELECT x + y FROM ...`
- `SELECT * FROM ... WHERE field > 10`
- `SELECT * FROM ... WHERE tag = 'something'`
- `SELECT * FROM ... WHERE field =~ /.*/`
- `SELECT * FROM ... WHERE time > now() - 1h`
- `SELECT * FROM ... WHERE time > now() - 10s`
- `SELECT ... ORDER BY field DESC`


### Functions
Functions in InfluxDB are grouped into 3 major types: aggregators, selectors, and transformers.

### Aggregators
- `count()`
- `distinct()`
- `integral()`
- `mean()`
- `median()`
- `spread()`
- `sum()`
- `stddev()`

### Selectors
- `bottom()`
- `first()`
- `last()`
- `max()`
- `min()`
- `percentile()`
- `top()`

### Transformers
- `derivative()`
- `difference()`
- `elapsed()`
- `moving_average()`
- `non_negative_derivative()`

### Continuous Queries
Continuous Queries (CQ) are queries that run automatically and periodically on realtime data and store query results in a specified measurement.

CQs operate on realtime data. They use the local server’s timestamp, the GROUP BY time() interval, and InfluxDB’s preset time boundaries to determine when to execute and what time range to cover in the query.

CQs execute at the same interval as the cq_query’s GROUP BY time() interval, and they run at the start of InfluxDB’s preset time boundaries. If the GROUP BY time() interval is one hour, the CQ executes at the start of every hour.

Using CQs is necessary to roll up high-resolution data into historic aggregations.

### Using Continuous Queries to Downsample
When using CQs to downsample, tags are converted to fields on the downsampled data. However, if you supply the query `GROUP BY *` tags are preserved.

### Retention Policy
A **Retention Policy** is part of InfluxDB's data structure that describes for how long the database keeps data. InfluxDB compares your local server's timestamp to the timestamps on your data and deletes data that are older than the RP's `DURATION`. A single database can have several RPs and RPs are unique per database.

InfluxDB writes to a retention policy called `DEFAULT` when not explicitly stated. There is an automatically generated RP on all created databases called `autogen` with an infinite retention period. It is set as `DEFAULT` if not replaced.

Examples:
- `CREATE RETENTION POLICY "two_hours" ON ... DURATION 2h REPLICATION 1 DEFAULT`
- `CREATE RETENTION POLICY "a_year" ON ... DURATION 52w REPLICATION 1`

### Using Continuous Queries with RPs
```
CREATE RETENTION POLICY "two_hours" ON "metrics" DURATION 2h REPLICATION 1

CREATE RETENTION POLICY "a_year" ON "metrics" DURATION 52w REPLICATION 1

DROP CONTINUOUS QUERY "public_per_5m" ON "metrics"
CREATE CONTINUOUS QUERY "public_per_5m" ON "metrics" BEGIN
  SELECT count("value") AS "pageviews"
  INTO "two_hours"."public_5m"
  FROM "public"
  GROUP BY time(5m), *
END

DROP CONTINUOUS QUERY "public_per_1h" ON "metrics"
CREATE CONTINUOUS QUERY "public_per_1h" ON "metrics"
RESAMPLE EVERY 5m
BEGIN
  SELECT sum("pageviews") AS "pageviews"
  INTO "a_year"."public_1h"
  FROM "two_hours"."public_5m"
  GROUP BY time(1h), *
END
```
