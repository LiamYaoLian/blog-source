---
title: MySQL
date: 2022-04-07 20:25:28
tags: DB
---

## Overview
* server layer + storage engine layer
  - server layer
    - connector: connects client to server; wait_timeout can decide how long it will wait before disconnecting if the client has not sent a query.
    - cache: After MySQL 8.0, not have this.
    - analyzer: lexical and syntax analysis
    - optimizer: which index to use, which table to be joined first
    - executor: execute; update binlog in disk
    the server layer has functions, stored procedures, trigger, view, etc

  - storage engine: store, update log
    - engines: InnoDb, MyISAM, Memory, etc

## Syntax
### Insert
```

INSERT INTO 表名 [(字段名 [,字段名] ...)] VALUES (值的列表);


INSERT INTO 表名 （字段名）
SELECT 字段名或值
FROM 表名
WHERE 条件;

INSERT INTO demo.goodsmaster
(
  itemnumber,
  barcode,
  goodsname,
  specification,
  unit,
  price
)
VALUES
(
  4,
  '0003',
  '尺子',
  '三角型',
  '把',
  5
);
```

* can skip inserting some columns if null allowed or have default values
* ON DUPLICATE: 假设我们要把门店 B 的商品数据，插入到门店 A 的商品表中去，如果有重复的商品编号，就用门店 B 的条码，替换门店 A 的条码，用门店 B 的商品名称，替换门店 A 的商品名称；如果没有重复的编号，就直接把门店 B 的商品数据插入到门店 A 的商品表中。

```

mysql> SELECT *
    -> FROM demo.goodsmaster;
+------------+---------+-----------+---------------+------+------------+
| itemnumber | barcode | goodsname | specification | unit | salesprice |
+------------+---------+-----------+---------------+------+------------+
|          1 | 0001    | 书        | 16开          | 本   |      89.00 |
|          2 | 0002    | 笔        | 10支装        | 包   |       5.00 |
|          3 | 0003    | 橡皮      | NULL          | 个   |       3.00 |
+------------+---------+-----------+---------------+------+------------+
3 rows in set (0.00 sec)



mysql> SELECT *
    -> FROM demo.goodsmaster1;
+------------+---------+-----------+---------------+------+------------+
| itemnumber | barcode | goodsname | specification | unit | salesprice |
+------------+---------+-----------+---------------+------+------------+
|          1 | 0001    | 教科书    | NULL          | NULL |      89.00 |
|          4 | 0004    | 馒头      |               |      |       1.50 |
+------------+---------+-----------+---------------+------+------------+
2 rows in set (0.00 sec)



INSERT INTO demo.goodsmaster
SELECT *
FROM demo.goodsmaster1 as a
ON DUPLICATE KEY UPDATE barcode = a.barcode,goodsname=a.goodsname;
-- 运行结果如下
mysql> SELECT *
    -> FROM demo.goodsmaster;
+------------+---------+-----------+---------------+------+------------+
| itemnumber | barcode | goodsname | specification | unit | salesprice |
+------------+---------+-----------+---------------+------+------------+
|          1 | 0001    | 教科书    | 16开          | 本   |      89.00 |
|          2 | 0002    | 笔        | 10支装        | 包   |       5.00 |
|          3 | 0003    | 橡皮      | NULL          | 个   |       3.00 |
|          4 | 0004    | 馒头      |               |      |       1.50 |
+------------+---------+-----------+---------------+------+------------+
4 rows in set (0.00 sec)
```

### Delete
```

DELETE FROM 表名
WHERE 条件

// 删除全部数据
DELETE FROM demo.goodsmaster;
```

### Update
```

UPDATE 表名
SET 字段名=值
WHERE 条件
```
* Don't update primary key

### Read
```

SELECT *|字段列表
FROM 数据源
WHERE 条件
GROUP BY 字段
HAVING 条件
ORDER BY 字段
LIMIT 起始点，行数
```

```
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
    [HIGH_PRIORITY]
    [STRAIGHT_JOIN]
    [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
    [SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    select_expr [, select_expr] ...
    [into_option]
    [FROM table_references
      [PARTITION partition_list]]
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}, ... [WITH ROLLUP]]
    [HAVING where_condition]
    [WINDOW window_name AS (window_spec)
        [, window_name AS (window_spec)] ...]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ... [WITH ROLLUP]]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [into_option]
    [FOR {UPDATE | SHARE}
        [OF tbl_name [, tbl_name] ...]
        [NOWAIT | SKIP LOCKED]
      | LOCK IN SHARE MODE]
    [into_option]

into_option: {
    INTO OUTFILE 'file_name'
        [CHARACTER SET charset_name]
        export_options
  | INTO DUMPFILE 'file_name'
  | INTO var_name [, var_name] ...
}

```
* Use of an unqualified * with other items in the select list may produce a parse error. To avoid this problem, use a qualified tbl_name.* reference:
```
SELECT AVG(score), t1.* FROM t1 ...
```
* You need not specify a tbl_name or db_name.tbl_name prefix for a column reference unless the reference would be ambiguous.

* The INTO clause, if present, can appear in any position indicated by the syntax description, but within a given statement can appear only once, not in multiple positions.

* column alias:
 - it is good practice to be in the habit of using AS explicitly when specifying column aliases.
 - can be used in GROUP BY, ORDER BY, or HAVING clauses
 - If the column alias contains spaces, you need to place it inside quotes
 - It is not permissible to refer to a column alias in a WHERE clause, because the column value might not yet be determined when the WHERE clause is executed.
```
SELECT CONCAT('Jane',' ','Doe') AS 'Full name';
```

* from: table, derived table
```
tbl_name [[AS] alias] [index_hint]
```
 - devrived table must have an alias
 - If you name more than one table, you are performing a join.
 - The use of index hints provides the optimizer with information about how to choose indexes during query processing. For a description of the syntax for specifying these hints, see Section 8.9.4, “Index Hints”.You can use SET max_seeks_for_key=value as an alternative way to force MySQL to prefer key scans instead of table scans. See Section 5.1.8, “Server System Variables”.

* GROUP BY
  - permits a WITH ROLLUP modifier. See Section 12.20.2, “GROUP BY Modifiers”.



* order by
  - asc, desc
  - If ORDER BY occurs within a parenthesized query expression and also is applied in the outer query, the results are undefined and may change in a future version of MySQL.
  - When you use ORDER BY or GROUP BY to sort a column in a SELECT, the server sorts values using only the initial number of bytes indicated by the max_sort_length system variable.

* limit:
 - start index is 0
 - `limit 1, 2`: 1 means starting from index 1 (included); 2 means 2 records
 - With one argument, the value specifies the number of rows to return from the beginning of the result set
 - For prepared statements, you can use placeholders.
 ```
 SET @skip=1; SET @numrows=5;
 PREPARE STMT FROM 'SELECT * FROM tbl LIMIT ?, ?';
 EXECUTE STMT USING @skip, @numrows;
 ```
 - If LIMIT occurs within a parenthesized query expression and also is applied in the outer query, the results are undefined and may change in a future version of MySQL


* HAVING
   - The query result includes only groups satisfying the HAVING conditions. (If no GROUP BY is present, all rows implicitly form a single aggregate group.).
   - The WHERE clause specifies conditions on columns in the select list, but cannot refer to aggregate functions.
   - LIMIT is applied after HAVING
   - SQL standard: HAVING must reference only columns in the GROUP BY clause or columns used in aggregate functions. Extension: permits HAVING to refer to columns in the SELECT list and columns in outer subqueries. If a HAVING column name is used both in GROUP BY and as an aliased column in the select column list, preference is given to the column in the GROUP BY column
   -

 * For update: ...
 * DISTINCT
 * STRAIGHT_JOIN forces the optimizer to join the tables in the order in which they are listed in the FROM clause.
 * STRAIGHT_JOIN does not apply to any table that the optimizer treats as a const or system table.
 * SQL_BIG_RESULT or SQL_SMALL_RESULT can be used with GROUP BY or DISTINCT to tell the optimizer that the result set has many rows or is small, respectively. For SQL_BIG_RESULT, MySQL directly uses disk-based temporary tables if they are created, and prefers sorting to using a temporary table with a key on the GROUP BY elements. For SQL_SMALL_RESULT, MySQL uses in-memory temporary tables to store the resulting table instead of using sorting. This should not normally be needed.
 * SQL_BUFFER_RESULT forces the result to be put into a temporary table. This helps MySQL free the table locks early and helps in cases where it takes a long time to send the result set to the client. This modifier can be used only for top-level SELECT statements, not for subqueries or following UNION.
 * Named Windows: https://dev.mysql.com/doc/refman/8.0/en/window-functions-named-windows.html
 * Partition: https://dev.mysql.com/doc/refman/8.0/en/partitioning-selection.html



## Log
### Redo Log in InnoDB
* Write-ahead logging
  - write to redo log, update memory, then disk
  - when the redo log is full, move to disk to clean some space
* Crash-safe: innodb_flush_log_at_trx_commit = 1
### binlog in the server layer
* can be used by all engines
* Has statement-based logging: Events contain SQL statements that produce data changes (inserts, updates, deletes)
* append
* sync_binlog = 1 to make sure binlog is not lost

InnoDB prepares redo log. Executor updates binlog. InnoDB commits redo log.
### Undo log
* When there are no read-views earlier than this undo log record, it will be deleted.
![undo log and read-view](../image/mysql/2.webp)
Source: https://time.geekbang.org/column/article/68963
* Long transaction will cause too many undo log records to stay

* To avoid long transaction, set autocommit=1
* To find long transaction (e.g. 60s):

```MySql

select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
```
* Analyze general log to avoid long transaction issues
* SET_MAX_EXECUTION_TIME


### How to restore
* Find the most recent full backup. Then based on binlog records later than the most recent full backup, restore data.
* back up every day or every week? More frequent, more storage, less RTO(Recovery Time Objective).

## Transaction
* high isolation will lead to low efficiency
### Isolation Levels
* read uncommitted: no read-view.
* read committed: a read-view is created when a SQL query executes
* repeatable read: a read-view is created when the transaction starts , so data seen in this transaction are the same as what they were when this transaction started.
* serializable: read and write will add lock. After a transaction finishes, another transaction can execute.
![isolation levels](../image/mysql/1.webp)
Source: https://time.geekbang.org/column/article/68963

## index
* different storage engines may have different index implementation.
### InnoDB index model
* B+ tree
![InnoDB index model](../image/mysql/3.webp)
* primary key index (called clustered index in InnoDB): leaves store records
* non primary key index (called secondary index in InnoDB): leaves store primary key values. If search based on non primary key index, will search the B+ tree of the non primary key index first, find the primary key value, and then search the B+ tree of the primary key index.
* index maintenance
  - inserting may cause some records to be moved
  - if a data page is full, inserting will create a new data page
* Usually using auto increment as the primary key leads to better performance and less storage. However, for the k-v situation, may use a business column as the primary key
