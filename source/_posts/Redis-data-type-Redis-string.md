---
title: Redis数据类型之字符串类型
date: 2019-06-28 20:50:28
categories:
- 笔记
tags: 
- Redis
---
## Redis数据类型之字符串类型

### 字符串类型

#### 介绍：


Redis String类型是可以与Redis键关联的最简单的值类型。: )

#### 常用命令：

##### SET key value (赋值)

以键名`key`保存一个字符串`value`。当`key`已经存在对应的值时会被覆盖，无论它原本保存的是什么类型的数据。[【文档】](https://redis.io/commands/set)

##### GET key (取值)

获取键名`key`对应的值。如果如果该键不存在则返回`nil`。当`key`中存储的数据不是`string`类型时会返回错误，因为GET指令只处理string类型的值[【文档】](https://redis.io/commands/get)

###### 示例：

```bash
redis> HSET hset1 name Joey # 存入一个散列类型的键值
(integer) 1
redis> TYPE hset1
hash
redis> GET hset1 # GET 命令不能用来获取字符串类型以外的value
(error) WRONGTYPE Operation against a key holding the wrong kind of value
redis> SET hset1 str # SET 命令使用相同的键存入字符串，覆盖了原来的散列类型数据
OK
redis> GET hset1 # GET 获取字符串类型数据
"str"
redis> TYPE hset1
string
```

##### INCR key (递增数字)

将`key`存储的数字+1并返回。如果`key`包含错误类型（`string`以外的类型）的`value`或包含不能表示为整数的字符串，则会提示错误。当要操作的`key`不存在时会默认`value`为0，所以第一次递增后的结果是1。[【文档】](https://redis.io/commands/incr)

###### 示例：

```bash
redis> SET myint 520
OK
redis> INCR myint
(integer) 521
redis> GET myint
"521"
```



##### INCRBY key increment (增加指定的整数)

除了能指定递增的number大小，就跟INCR命令一样了。[【文档】](https://redis.io/commands/incrby)

所以 INCR key 跟 INCRBY key 1 是一样的

###### 示例

```bash
redis> INCRBY myint1 1
(integer) 1
redis> GET myint1
"1"
```

##### DECR 递减数字

将`key`中存储的数字减1。如果`key`包含错误类型（`string`以外的类型）的`value`或包含不能表示为整数的字符串，则会提示错误。当要操作的`key`不存在时会默认`value`为0，所以第一次递增后的结果是-1。[【文档】](https://redis.io/commands/decr)

###### 示例

```bash
redis> SET number1 10
"OK"
redis> DECR number1
(integer) 9
redis> DECR number2
(integer) -1
```

##### APPEND key value 向尾部追加字符串

如果`key`已存在且为字符串，则此命令会在字符串末尾附加`value`。如果`key`不存在，则创建它并将其设置为空字符串，因此在这种特殊情况下`APPEND`将类似于`SET`。[【文档】](https://redis.io/commands/append)

###### 示例

```bash
redis> APPEND s1 abc
(integer) 3
redis> GET s1
"abc"
redis> APPEND s1 def
(integer) 6
redis> GET s1
"abcdef"
```
