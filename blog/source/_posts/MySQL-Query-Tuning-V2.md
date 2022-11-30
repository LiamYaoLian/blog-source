---
title: MySQL Query Tuning V2
date: 2022-11-30 10:05:46
tags:
---
# Transaction and Lock
* Use InnoDB (default setting) reduces great hassle. Transaction is implemented by storage engine. If your transaction uses Table A and Table B and Table B's storage engine does not support transaction, MySQL does not throw an error and data on Table B will not be rolled back.
Note: Lock is release on commit or roll back.
* Use lock with smaller granularity to allow more concurrency. But more locks consumes more resources. It is a trade off.
* Retry after deadlock is detected. InnoDB storage engine will notice circular dependencies and return an error instantly. InnoDB rolls back the transaction that has the fewest exclusive row locks.
```
# Transaction 1
START TRANSACTION;
UPDATE StockPrice SET close = 45.50 WHERE stock_id = 4 and date = ‘2020-05-01’;
UPDATE StockPrice SET close = 19.80 WHERE stock_id = 3 and date = ‘2020-05-02’;
COMMIT;
```
```
# Transaction 2
START TRANSACTION;
UPDATE StockPrice SET high = 20.12 WHERE stock_id = 3 and date = '2020-05-02';
UPDATE StockPrice SET high = 47.20 WHERE stock_id = 4 and date = '2020-05-01';
COMMIT;
```
* If you set AUTOCOMMIT = 0, you are always in a transaction until you issue a COMMIT or ROLLBACK. MySQL then starts a new transaction immediately.
* If You set AUTOCOMMIT = 1, use `begin` and `start transaction` if one transaction has many queries
Note: Certain commands (typically DDL such as ALTER TABLE, but may be others such as LOCK TABLES) cause MySQL to commit the transaction before they execute.
* Choose the suitable isolation level: READ COMMITTED, READ UNCOMMITTED, REPEATABLE READ, (usually not choose) SERIALIZABLE
TODO
* Understand how MVCC allows most read queries not needing to acquire locks. `READ COMMITTED` and `REPEATABLE READ` use MVCC, while `READ UNCOMMITTED` and `SERIALIZABLE` do not need it.
InnoDB implements MVCC by assigning a transaction ID for each transaction at the first time the transaction reads any data. When a record is modified within that transaction, an undo record that explains how to revert that change is written to the undo log, and the rollback pointer of the transaction is pointed at that undo log record.

* May need to use explicit locking:
```
SELECT ... FOR SHARE          // After MySQL 8.0
SELECT ... FOR UPDATE
```
TODO




# Schema
## Data Type
1. Smaller is usually better than bigger. For example, if `TINYINT` is sufficient, it is more efficient than `INT`.
2. Estimate conservatively. Realizing the data type is not big enough after the code has been in production is more painful.
3. Simple is good. For example, integers are cheaper than characters.
4. Avoid `null` if possible. Set a default value to leverage index.
5. `DECIMAL` is precise but more expensive. In some high-volume cases, use a BIGINT instead and store the data as some multiple of the smallest fraction of currency. For example, multiply all dollar amounts by a million and store the result in a BIGINT.
6. `VARCHAR` require less storage space than fixed-length types because it uses only as much space as it needs. However, because the rows are variable length, they can grow when you update them, which can cause extra work. If a row grows and no longer fits in its original location, the behavior is storage engine dependent. For example, InnoDB may need to split the page to fit the row into it. It’s usually worth using VARCHAR when the maximum column length is much larger than the average length; when updates to the field are rare, so fragmentation is not a problem; and when you’re using a complex character set such as UTF-8, where each character uses a variable number of bytes of storage.
7. Using ENUM instead of a string type. ENUM is 1 or 2 bytes. ENUM is stored as integers as the sequence specified in the definition. Avoid `ENUM('1', '2', '3')`. ENUM will be converted to string in a string context. If you want to sort them alphabetically:
```
SELECT e FROM enum_test ORDER BY FIELD(e, 'apple', 'dog', 'fish');
```
8. DATETIME is independent of TIMEZONE, 8 bytes, millisecond-precision, from the year 1000 to the year 9999. TIMESTAMP depends on TIMEZONE, 4 bytes, second-precision, from 1970-1-1 midnight GMT to 2038-1-19. MySQL server, operating system, and client connections all have time zone settings.

## Primary KEY
1. Once you choose a type for PK, make sure you use the same type in all related tables
2. The Smaller, the better
3. Integer is usually better than string
4. Avoid random number, because values will be distributed over a large space. If you still want to use UUID, convert it into 16-byte numbers with UNHEX() and store them in a BINARY(16) column. You can retrieve the values in hexadecimal format with the HEX() function.

## Other
* If a table have hundreds of columns, even if we only use a few columns, InnoDB will parse all columns, which is expensive.

## Summary Table
* To improve performance

# Index
## B Tree
* Avoid duplicate, redundant, unused indices, because MySQL has to maintain each index separately.
* Use cluster index if possible. Cluster index leaf stores the row. Non-cluster index leaf stores PK.
* The leaf pages can perform better if they are physically sequential, otherwise it's called "fragmented" and range scans or full index scans are slower. Avoid that with random values such as UUID. To defragment data, you can either run OPTIMIZE TABLE or dump and reload the data.
* Use multicolumn index if possible instead of many indices on single columns (because MySQL needs to do "index merge" for the latter one), and consider:
  - using covering index, which is an index that contains (or “covers”) all the data needed to satisfy a query.
  - the left-most rule when deciding the order of columns.
  - what sorting is needed in queries. B-tree indexes stores the data in sorted order, and MySQL can exploit that for ORDER BY and GROUP BY. Ordering the results by the index does not work if a column in ORDER BY is not in the index or the order in ORDER BY is not the order of the index. If you need to sort in different directions, a trick is to store a reversed or negated value.
  ```
  CREATE TABLE rental (
   ...
   PRIMARY KEY (rental_id),
   UNIQUE KEY rental_date (rental_date,inventory_id,customer_id),
   KEY idx_fk_inventory_id (inventory_id),
   KEY idx_fk_customer_id (customer_id),
   KEY idx_fk_staff_id (staff_id),
   ...
  );
  ```
  ```
  ... WHERE rental_date = '2005-05-25' ORDER BY inventory_id, staff_id;
  # does not use index because the ORDER BY refers to a column that isn’t in the index
  ```
  The ORDER BY clause needs to form a leftmost prefix of the index unless leading columns are constants.
  ```
  CREATE TABLE rental (
   ...
   PRIMARY KEY (rental_id),
   UNIQUE KEY rental_date (rental_date,inventory_id,customer_id),
   KEY idx_fk_inventory_id (inventory_id),
   KEY idx_fk_customer_id (customer_id),
   KEY idx_fk_staff_id (staff_id),
   ...
  );
  ```
  ```
  mysql> EXPLAIN SELECT rental_id, staff_id FROM sakila.rental
      -> WHERE rental_date = '2005-05-25'
      -> ORDER BY inventory_id, customer_id\G
  *************************** 1. row ***************************
   type: ref
   possible_keys: rental_date
   key: rental_date
   rows: 1
   Extra: Using where
  ```
  If the query joins multiple tables, ordering the results by the index works only when all columns in the ORDER BY clause refer to the first table.
  - Assuming there is no sorting and group by, the column that can filter out the most rows should be the leftest.
* Consider using prefix to save storage. You must define prefix if you want to index BLOB, TEXT, or very long VARCHAR columns, because MySQL disallows indexing their full length.
```
ALTER TABLE cities ADD KEY (city(7));
```

# Query Tuning
## Slower to Fast
constants
unique index lookups
range scans
index scans
full table scans

## Tips
1. Avoid `SELECT *`. Only include columns needed.
2. Choose the suitable query size. It is a trade-off between:
  - reduce the number of connections established.
  - avoid locking for too long if there is a lock.
3. Do not perform calculation on index column while filtering. An example that violates this:
  ```
  SELECT actor_id FROM actor WHERE actor_id + 1 = 5;
  ```
4. Join
  - Avoid join too many tables. Join them in application.
  - The table has fewer rows that meet the ON criteria should be the first table.
  - Use the column with an index as the join criteria if possible.
5. Avoid correlated subqueries:
  - Method 1: Replace IN with EXISTS
  ```
  mysql> SELECT * FROM film
    -> WHERE EXISTS(
    ->    SELECT * FROM film_actor WHERE actor_id = 1
    ->       AND film_actor.film_id = film.film_id);
  ```
  - Method 2: Join. But this may not be more efficient because it will join all rows that can be joined first before filtering.
  - Method 3: Generate the IN list with GROUP_CONCAT in another query


6. Use columns in the same table in GROUP BY and ORDER BY to leverage index. Move the WITH ROLLUP functionality into your application code.
7. LIMIT with large offet
```
SELECT film_id, description FROM sakila.film ORDER BY title LIMIT 50, 5;
```
- Method 1:
```
mysql> SELECT film.film_id, film.description
    -> FROM sakila.film
    ->    INNER JOIN (
    ->       SELECT film_id FROM sakila.film
    ->       ORDER BY title LIMIT 50, 5
    ->    ) AS lim USING(film_id);
```
- Method 2: position is indexed
```
SELECT film_id, description FROM sakila.film
WHERE position BETWEEN 50 AND 54 ORDER BY position;
```
- Method 3
```
mysql> SELECT * FROM sakila.rental
-> WHERE rental_id < 16030
-> ORDER BY rental_id DESC LIMIT 20;
```
8. Replace `OR` with `IN`. MySQL uses binary search on the `IN` list. However, `IN` list that is too long may hurt performance,
because the optimizer may copy the list for all related tables (due to a WHERE, ON, or USING clause that sets the column equals to that on another table).
9. Union:
  - add filtering
  ```
  (SELECT first_name, last_name
   FROM sakila.actor
   ORDER BY last_name)
  UNION ALL
  (SELECT first_name, last_name
   FROM sakila.customer
   ORDER BY last_name)
  LIMIT 20;
  ```
  Should be:
  ```
  (SELECT first_name, last_name
   FROM sakila.actor
   ORDER BY last_name
   LIMIT 20)
  UNION ALL
  (SELECT first_name, last_name
   FROM sakila.customer
   ORDER BY last_name
   LIMIT 20)
  LIMIT 20;
  ```
  - If sorting is not needed, use UNION ALL, because UNION will sort.
10. MAX() and MIN()
```
mysql> SELECT MIN(actor_id) FROM sakila.actor WHERE first_name = 'PENELOPE';
```
Can be:
```
mysql> SELECT actor_id FROM sakila.actor USE INDEX(PRIMARY)
    -> WHERE first_name = 'PENELOPE' LIMIT 1;
```

11. Count()
```
SELECT SUM(IF(color = 'blue', 1, 0)) AS blue,SUM(IF(color = 'red', 1, 0))
AS red FROM items;
```
