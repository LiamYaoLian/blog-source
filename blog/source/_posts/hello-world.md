---
title: Collection
tag: Framework
---

![Collection Framework](../image/collection/collection_framework.png)

## List
* ArrayList: Object[]
* Vector: Object[], thread-safe
* LinkedList: doubly-linked

## Set
* HashSet: based on HashMap
* LinkedHashSet: extends HashSet, use LinkedHashMap
* TreeSet: RedBlackTree

## Map
* HashMap: if array length < 64, array + linked list, resizes the array, otherwise converts the linked list to RedBlackTree
* LinkedHashMap: extends HashMap, adds a doubly-linked list to save the order of keys
* HashTable: array + linked list; thread-safe
* TreeMap: RedBlackTree

# Thread-safe
* CopyOnWriteArrayList

* ConcurrentLinkedQueue
* BlockingQueue
* ConcurrentHashMap
* ConcurrentSkipListMap




## Why not array?
* fixed length

## Iterator
Safe: when an element is modified, throws concurrentModificationException
