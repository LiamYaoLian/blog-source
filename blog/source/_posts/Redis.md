---
title: Redis
date: 2022-05-08 11:49:20
tags:
---

## Configuration
* daemonize yes
* port: change 6379 to other
* logfile: change to "redis.log"
* requirepass foobared // foobared is the password
* protected-mode no //allow remote access
* databases
* bind 0.0.0.0 // means allow a computer with any IP to access. Usually should use the IP of the server rather than 0.0.0.0


## redis-cli
* netstat - tulpn | grep redis
* redis-cli -p 6380
* auth foobared
* ping
* exit
* shutdown
* select 0 // select database No. 0
* set name lily // 设置key=name, value=lily
* get hello // 获得key=hello结果
* keys he* // 根据Pattern表达式查询符合条件的Key
* dbsize // 返回key的总数
* exists a // 检查key=a是否存在
* del a // 删除key=a的数据
* expire hello 20 // 设置key=hello 20秒后过期
* ttl hello // 查看key=a的过期剩余时间
* keys *

## Data Type
* String // String最大512mb, 建议单个kv不超过100kb
* List
* Hash
* Set
* Zset // 有序集合

### String
get: get hello 获得key=hello结果
set: set hello world 设置key=hello,value=hello
mset: mset hello world java best 一次性设置多个值
mget: mget hello java 一次性获取多个值
del: del hello 删除key=hello
incr: incr count key值自增1
decr: decr count key值自减1
incrby: incrby count 99 自增指定步长
decrby: decrby count 99 自减指定步长

### Hash
hget: `hget emp:1 age` 获取hash中key=age的值
hset: `hset emp:1 age 23` 设置hash中age=23
hmset: `hmset emp:1 age 30 name kaka` 设置hash多个值
hmget: `hmget emp:1 age name` 获取hash多个值
hgetall: `hgetall emp:1` 获取hash所有值
hdel: `hdel emp:1 age` 删除user:1的age
hexists: `hexists emp:1 name` 检查是否存在
hlen: `hlen emp:1` 获取指定长度

### List
* List列表最大长度为2的32次方-1，可以包含40亿个元素

rpush listkey c b a - 右侧插入
lpush listkey f e d - 左侧插入
rpop listkey - 右侧弹出
lpop listkey - 左侧弹出
llen listkey - 获取长度
lrange listkey 0 2 // start at 0, end at 2 (included)
lrange listkey 0 -1 // These offsets can also be negative numbers indicating offsets starting at the end of the list. For example, -1 is the last element of the list, -2 the penultimate, and so on.



defcba

### Set
smembers set1
sadd set1 a
sinter set1 set2 //交集元素
sunion set1 set2 // union
sdiff set1 set2 // in set1 but not in set2

### Zset

zadd zset1 100 a
zadd zset1 101 b
zrange zset1 0 -1 withscores// print a and b and their scores, -1 means all
zrangebyscore zset1 100 103

## Jedis
```java
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```
```java
package com.imooc.jedis;

import com.alibaba.fastjson.JSON;
import redis.clients.jedis.Jedis;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class CacheSample {
    public CacheSample(){
        Jedis jedis = new Jedis("192.168.132.144");
        try {
            List<Goods> goodsList = new ArrayList<Goods>();
            goodsList.add(new Goods(8818, "红富士苹果", "", 3.5f));
            goodsList.add(new Goods(8819, "进口脐橙", "", 5f));
            goodsList.add(new Goods(8820, "进口香蕉", "", 25f));
            jedis.auth("12345");
            jedis.select(3);
            for (Goods goods : goodsList) {
                String json = JSON.toJSONString(goods);
                System.out.println(json);
                String key = "goods:" + goods.getGoodsId();
                jedis.set(key , json);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            jedis.close();
        }
    }

    public static void main(String[] args) {
        new CacheSample();
        System.out.printf("请输入要查询的商品编号：");
        String goodsId = new Scanner(System.in).next();
        Jedis jedis = new Jedis("192.168.132.144");
        try{
            jedis.auth("12345");
            jedis.select(3);
            String key = "goods:" + goodsId;
            if(jedis.exists(key)){
                String json = jedis.get(key);
                System.out.println(json);
                Goods g = JSON.parseObject(json, Goods.class);
                System.out.println(g.getGoodsName());
                System.out.println(g.getPrice());
            }else{
                System.out.println("您输入的商品编号不存在，请重新输入！");
            }
        }catch(Exception e){
            e.printStackTrace();
        }finally {
            jedis.close();
        }
    }
}

```

```java
package com.imooc.jedis;

import redis.clients.jedis.Jedis;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class JedisTestor {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("192.168.132.144" , 6379);
        try {
            jedis.auth("12345");
            jedis.select(2);
            System.out.println("Redis连接成功");
            //字符串
            jedis.set("sn" , "7781-9938");
            String sn = jedis.get("sn");
            System.out.println(sn);
            jedis.mset(new String[]{"title" , "婴幼儿奶粉" , "num" , "20"});
            List<String> goods =  jedis.mget(new String[]{"sn" , "title" , "num"});
            System.out.println(goods);
            Long num = jedis.incr("num");
            System.out.println(num);
            //Hash
            jedis.hset("student:3312" , "name" , "张晓明");
            String name = jedis.hget("student:3312" , "name");
            System.out.println(name);
            Map<String,String> studentMap = new HashMap();
            studentMap.put("name", "李兰");
            studentMap.put("age", "18");
            studentMap.put("id", "3313");
            jedis.hmset("student:3313", studentMap);
            Map<String,String> smap =  jedis.hgetAll("student:3313");
            System.out.println(smap);
            //List
            jedis.del("letter");
            jedis.rpush("letter" , new String[]{"d" , "e" , "f"});
            jedis.lpush("letter" ,  new String[]{"c" , "b" , "a"});
            List<String> letter =  jedis.lrange("letter" , 0 , -1);
            jedis.lpop("letter");
            jedis.rpop("letter");
            letter = jedis.lrange("letter", 0, -1);
            System.out.println(letter);
        }catch(Exception e){
            e.printStackTrace();
        }finally {
            jedis.close();
        }

    }
}

```
