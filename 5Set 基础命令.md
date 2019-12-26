    Set是集合，他是String类型的无序集合，set是通过hash table来实现新增、删除、查找的。set集合支持查找集合中的差集、交集、并集。

1 sadd:往set集合中添加元素，不能添加重复数据

2 smembers:查看set集合中的元素

3 srem:删除set集合中元素

4 spop:随机删除set集合中元素，返回删除元素值

5 sdiff:取两个set集合的差集，以第一个集合为准，取差集。
```
127.0.0.1:6379> smembers book
1) "englishbook"
2) "mathsbook"
127.0.0.1:6379> smembers booktwo
1) "mathsbook"
2) "storybook"
127.0.0.1:6379> sdiff book booktwo
1) "englishbook"
127.0.0.1:6379> sdiff booktwo book
1) "storybook"
127.0.0.1:6379> 
```

6 sdiffstore:取两个set集合的差集，并存储到另一个set集合中
```
127.0.0.1:6379> smembers book
1) "mathsbook"
2) "storybook"
127.0.0.1:6379> smembers booktwo
1) "chinsesbook"
2) "mathsbook"
127.0.0.1:6379> sdiffstore rebooks book booktwo
(integer) 1
127.0.0.1:6379> smembers rebooks
1) "storybook"
127.0.0.1:6379>
```

7 sinter:取两个set集合的交集，同理sinterstore则是把返回交集的结果，存储到另一个set集合中

8 sunion:取出两个set集合的并集，同理sunionstore则是把返回交集的结果，存储到另一个set集合中

9 smove:从第一个set集中的元素，移动到第二个set集合中
```
127.0.0.1:6379> smembers book
1) "mathsbook"
2) "storybook"
127.0.0.1:6379> smembers booktwo
1) "chinsesbook"
2) "mathsbook"
127.0.0.1:6379> smove book booktwo storybook
(integer) 1
127.0.0.1:6379> smembers booktwo
1) "chinsesbook"
2) "mathsbook"
3) "storybook"
127.0.0.1:6379>
```
10 scard:查看set集中的元素个数，返回个数

11 sismember:判断该元素，是否存在该set集合中，如果存在返回1，不存在返回0

12 srandmember:随机返回集合中的某个元素，但是不删除元素
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk0NzI4MDE2MF19
-->