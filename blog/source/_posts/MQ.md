---
title: MQ
date: 2022-11-07 11:27:06
tags:
---
# Why
* asynchronous operation
* decouple
* rate limit
TODO

# Challenge
* lost message
- After the message has been persisted in at least two servers, ack to producer
- After consumer finishes business logic, ack
* duplicate messages
- version
- id
- unique key
* order
- globally: one producer, one queue, one thread consumes
- locally: one thread consumes one queue
* message accumulation
- bug?
- consuming slowly: optimize consumption, such as batch insert
- scale-out: add more queues
