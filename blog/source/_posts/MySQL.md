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
