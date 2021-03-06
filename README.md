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
CREATE DATABASE Flink
```

3. Create table

```
CREATE TABLE orders (
    order_uid  BIGINT,
    product_id BIGINT,
    price      DECIMAL(32, 2),
    order_time TIMESTAMP(3),
    process_time AS PROCTIME(),
    WATERMARK FOR order_time AS order_time - INTERVAL '5' SECOND
) with (
   'connector' = 'pulsar',
   'topic' = 'persistent://public/default/orders'
)
```

4. Insert data into table

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

5. Query data

```
SELECT order_uid, product_id, order_time
FROM orders /*+ OPTIONS('scan.startup.mode'='earliest') */
WHERE price >= 1000
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