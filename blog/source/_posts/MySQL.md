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
* 记录语句逻辑，如“给id = 2的行的c字段加1”
* append
* sync_binlog = 1 to make sure binlog is not lost

InnoDB prepares redo log. Executor updates binlog. InnoDB commits redo log.

### How to restore
* 按找最近的全量备份恢复。然后，从最近备份时间点开始，根据binlog恢复。

## Transaction
* high isolation will lead to low efficiency
### Isolation levels
* read uncommitted: no view.
* read committed: a view is created when a SQL query executes
* repeatable read: data seen in this transaction are the same as what they were when this transaction started (view created at that time).
* serializable: read write will add lock. After a transaction finishes, another transaction can execute.
