---
title: Distributed Transaction
date: 2022-03-25 19:56:36
tags:
---

## Transaction
a programming execution unit to manipulate data in the databases
* Atomic: all succeed or all fail
* Consistency: does not change the data integrity of database. ?
* Isolation: not disturb other transactions
* Durability: permanently change

### Isolation
* Read uncommitted
* Read committed: ?
* Repeatable read: ?
* Serializable: ?

## How to ensure ACID
InnoDB:
* A: Undo log
* C: Undo log
* I: lock
* D: Redo log

### Undo log
record data before manipulation in undo log, rollback

### Redo log
record new data in redo log

### Issues
## Dirty Read
## Phantom Read
## Unrepeatable Read
##

## What
* Make sure data consistent in sub-system.
* A big operation consists of small operations on many nodes. Make sure all succeed or fail.
* Example: Create order and decrement stock count. All succeed or fail.

## Example
* Multiple databases
* Sharding
* Microservices

## CAP
* Consistency: all nodes see the same data at the same time.
* Availability: return normal result (success or failure) within reasonable time
* Partition tolerance: ?
CP or AP
Money: CP
Internet: AP


### Consistency
* Strong: The data in all nodes is the same at any time
* Weak: ?
* Eventual: ?

## BASE
For AP
Eventual instead of strong consistency
* Basically available: some available: longer response time; lose some functions
* Soft state: allow middle state, 即允许系统在不同节点的数据副本之间进⾏数据同步的过程存在延时
* Eventually consistent: ?

### BASE vs ACID
* ACID: strong consistency
* BASE: sacrifice consistency for availability

## How to solve the consistency problem
* avoid distribution
* distributed transaction

## Distributed Transaction Type
* Hard transaction: CP. Suitable for low concurrency and short transaction
* Soft transaction: BASE.

### Hard transaction
* XA: 2PC, JTA, JTS; 3PC; due to synchronous blocking, low efficiency. 
