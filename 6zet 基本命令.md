再往Zset集合中添加数据，需要加上该元素的顺序，每一次赋值Zset会重新按照顺序属性进行调整顺序。

1 zadd:往集合中添加元素

2 zrange:查看集合中的元素，后面加上withscores即可显示当前元素所对应的顺序

3 zrem:删除集合中指定的元素

4 zincrby:指定增加元素所对应的顺序，之前noe对应的顺序是1，然后使用zincrby新增了3
```
127.0.0.1:6379> zrange myzset 0 -1 withscores
1) "noe"
2) "1"
127.0.0.1:6379> zincrby myzset 3 noe
"4"
127.0.0.1:6379> zrange myzset 0 -1 withscores
1) "noe"
2) "4"
127.0.0.1:6379> 
```

5 zrank:返回指定元素在集合中的所对应的索引。 索引是从0开始的

6 zrevrange:按集合顺序，从大到小进行显示

7 zrangebyscore:显示指定顺序范围内的元素，按元素顺序
```
127.0.0.1:6379> zrange myzset 0 -1 withscores
1) "noe"
2) "1"
3) "tow"
4) "2"
5) "three"
6) "3"
7) "four"
8) "4"
127.0.0.1:6379> zrangebyscore myzset 2 3
1) "tow"
2) "three"
127.0.0.1:6379> 
```
8 zcount：返回指定顺序范围内元素的个数，按元素顺序
```
127.0.0.1:6379> zrange myzset 0 -1 withscores
1) "noe"
2) "1"
3) "tow"
4) "2"
5) "three"
6) "3"
7) "four"
8) "4"
127.0.0.1:6379> zcount myzset 2 3
(integer) 2
127.0.0.1:6379> 
```
9 zcrad:返回集合中元素数量

10 zremrangebyrank:删除指定索引范围内的元素，索引从0开始，0到1，相对应的元素是noe、tow。按索引
```
127.0.0.1:6379> zrange myzset 0 -1
1) "noe"
2) "tow"
3) "three"
4) "four"
127.0.0.1:6379> zremrangebyrank myzset 0 1
(integer) 2
127.0.0.1:6379> zrange myzset 0 -1
1) "three"
2) "four"
127.0.0.1:6379>
```

11 zremrangebyscore:删除指定顺序范围内的元素，按元素所对应的顺序

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA3ODE5NTEyM119
-->