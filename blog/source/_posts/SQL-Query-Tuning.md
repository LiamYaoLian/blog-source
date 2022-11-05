---
title: SQL Query Tuning
date: 2022-11-05 11:47:08
tags:
---


# what happens when you send MySQL a query:
The client sends the SQL statement to the server.
The server checks the query cache. If there’s a hit, it returns the stored result from the cache; otherwise, it passes the SQL statement to the next step.
The server parses, preprocesses, and optimizes the SQL into a query execution plan.
The query execution engine executes the plan by making calls to the storage engine API.
The server sends the result to the client.

The protocol is half-duplex, which means that at any given time the MySQL server can be either sending or receiving messages, but not both. It also means there is no way to cut a message short.

the temporary table created by the subquery has no indexes.

# Metrics
query cost metrics include:
- Execution time
- Number of rows examined
- Number of rows returned
Note: access types: a full table scan > index scans > range scans > unique index lookups > constants

# Best Practices
* Try changing the schema such as using summary tables
* Try covering indexes
* Use as few queries as possible, but sometimes executing a few simple queries instead of one complex one.
* Avoid retrieving unnecessary data such as `select *`
* union
  - filter out unnecessary rows within union
  - use union all instead of union if possible
* Sometimes avoid correlated subqueries.
  - use join
  - or manually generate the IN() list by executing the subquery as a separate query with GROUP_CONCAT(). Sometimes this can be faster than a JOIN.
* join
  - decompose a join by running multiple single-table queries instead of a multitable join, and then performing the join in the application. For example, instead of this single query:
  ```
  mysql> SELECT * FROM tag
      ->    JOIN tag_post ON tag_post.tag_id=tag.id
      ->    JOIN post ON tag_post.post_id=post.id
      -> WHERE tag.tag='mysql';
  ```
  You might run these queries:
  ```
  mysql> SELECT * FROM  tag WHERE tag='mysql';
  mysql> SELECT * FROM  tag_post WHERE tag_id=1234;
  mysql> SELECT * FROM  post WHERE  post.id in (123,456,567,9098,8904);

  ```
  - join in the order A, B. Ideally A should be a smaller table.
  - Have indexes on the columns in the ON clauses. Consider the join order when adding indexes. If you’re joining tables A and B on column c and the query optimizer decides to join the tables in the order B, A, you don’t need to index the column on table B. In general, you need to add indexes only on the second table in the join order, unless they’re needed for some other reason.
* group by, order by
  - Try to ensure that any GROUP BY or ORDER BY expression refers only to columns from a single table, so MySQL can try to use an index for that operation.
  - If you need to group a join by a value that comes from a lookup table, it’s usually more efficient to group by the lookup table’s identifier than by the value. For example, the following query isn’t as efficient as it could be:
  ```
  mysql> SELECT actor.first_name, actor.last_name, COUNT(*)
      -> FROM sakila.film_actor
      ->    INNER JOIN sakila.actor USING(actor_id)
      -> GROUP BY actor.first_name, actor.last_name;
  ```
  Rewrite:
  ```
  mysql> SELECT actor.first_name, actor.last_name, COUNT(*)
      -> FROM sakila.film_actor
      ->    INNER JOIN sakila.actor USING(actor_id)
      -> GROUP BY film_actor.actor_id;
  ```    
* limit
- do the offset on a covering index, rather than the full rows. You can then join the result to the full row and retrieve the additional columns you need. Consider the following query:
```
mysql> SELECT film_id, description FROM sakila.film ORDER BY title LIMIT 50, 5;
```
Rewrite:
```
mysql> SELECT film.film_id, film.description
    -> FROM sakila.film
    ->    INNER JOIN (
    ->       SELECT film_id FROM sakila.film
    ->       ORDER BY title LIMIT 50, 5
    ->    ) AS lim USING(film_id);
```
Sometimes you can also convert the limit to a positional query, which the server can execute as an index range scan. For example, if you precalculate and index a position column, you can rewrite the query as follows:
```
mysql> SELECT film_id, description FROM sakila.film
    -> WHERE position BETWEEN 50 AND 54 ORDER BY position;
```

Another example:
```
select * from t limit 900000, 10;
```
Write:
```
select * from t
where id > 900000
limit 10;
```
* replace "or" with "in" or "union"
* in
  - if "in" list is too large, it can be slow. The optimizer will “share” the list by copying it to the corresponding columns in all related tables.  
* batch insert
* min and max
  ```
  mysql> SELECT MIN(actor_id) FROM sakila.actor WHERE first_name = 'PENELOPE';
  ```
  Because there’s no index on first_name, this query performs a table scan. If MySQL scans the primary key, it can theoretically stop after reading the first matching row, because the primary key is strictly ascending and any subsequent row will have a greater actor_id.
  Rewrite:
  ```
  mysql> SELECT actor_id FROM sakila.actor USE INDEX(PRIMARY)
      -> WHERE first_name = 'PENELOPE' LIMIT 1;
  ```   
  
Reference: https://www.oreilly.com/library/view/high-performance-mysql/9780596101718/ch04.html
