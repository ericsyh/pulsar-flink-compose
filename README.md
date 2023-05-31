# pulsar-flink-compose

A sample of Pulsar Flink SQL connector in docker-compose.

## For Flink 1.13

### Run the environment

```
# Clone the repo
git clone git@github.com:ericsyh/pulsar-flink-compose.git && pulsar-flink-compose

# Run the components
docker-compose up -d
```

###  Run the demo

1. Run the Flink SQL client

```
docker-compose exec jobmanager ./bin/sql-client.sh embedded -e ./sql-conf/sql-client-defaults.yaml -l ./connector-lib
```

2. Create Database

```
CREATE DATABASE Flink;
```
```
USE Flink;
```

3. Create table

```
CREATE TABLE users (
    name STRING,
    age INT,
    income DOUBLE,
    single BOOLEAN,
    createTime BIGINT,
    row_time AS cast(TO_TIMESTAMP(FROM_UNIXTIME(createTime / 1000), 'yyyy-MM-dd HH:mm:ss') as timestamp(3)),
    WATERMARK FOR row_time AS row_time - INTERVAL '5' SECOND
) WITH (
  'connector' = 'pulsar',
  'topic' = 'persistent://public/default/user',
  'format' = 'json'
);
```

4. Insert data into table

```
INSERT INTO users VALUES 
  ('user 1', 31, 250000.0, false, 1656831003),
  ('user 2', 45, 250003.0, false, 1656833003),
  ('user 3', 73, 150300.0, false, 1656834003),
  ('user 4', 53, 35000.0, false, 1656836003),
  ('user 5', 74, 2500.0, false, 1656835003),
  ('user 6', 11, 75000.0, true, 1656862003),
  ('user 7', 25, 250230.0, true, 1659831003),
  ('user 8', 43, 135000.0, true, 1653831003),
  ('user 9', 14, 15000.0, true, 1656841003);
```

5. Query data

```
SELECT * 
FROM users /*+ OPTIONS('scan.startup.mode'='earliest') */
WHERE income >= 100000;
```

## For Flink 1.12

### Run the environment

```
# Clone the repo
git clone git@github.com:ericsyh/pulsar-flink-compose.git && pulsar-flink-compose

git checkout pulsar-flink-1.12

# Run the components
docker-compose up -d
```


###  Run the demo

1. Run the Flink SQL client

```
docker-compose exec jobmanager ./bin/sql-client.sh embedded -d ./sql-conf/sql-client-defaults.yaml -l ./connector-lib
```

2. Create table

```
CREATE TABLE orders (
    order_uid  BIGINT,
    product_id BIGINT,
    price      DECIMAL(32, 2),
    order_time TIMESTAMP(3),
    process_time AS PROCTIME(),
    WATERMARK FOR order_time AS order_time - INTERVAL '5' SECOND
)
```

3. Insert data into table

```
INSERT INTO orders VALUES
  (1, 100, 30.15, CURRENT_TIMESTAMP),
  (2, 200, 40, CURRENT_TIMESTAMP),
  (3, 300, 28000.56, CURRENT_TIMESTAMP),
  (4, 400, 42960.90, CURRENT_TIMESTAMP),
  (5, 500, 50000.1, CURRENT_TIMESTAMP),
  (6, 100, 688888888.7, CURRENT_TIMESTAMP),
  (7, 300, 20.99, CURRENT_TIMESTAMP),
  (8, 100, 6000, CURRENT_TIMESTAMP)
```

4. Query data

```
SELECT order_uid, product_id, order_time
FROM orders /*+ OPTIONS('scan.startup.mode'='earliest') */
WHERE price >= 1000
```