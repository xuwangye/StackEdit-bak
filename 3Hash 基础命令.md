
1 hsetnx:用于给哈希表中的字段赋值，如果哈希中的字段已存在则返回0，不会覆盖。nx代表 not exits

2 hmset

3 hmget

4 hincrby

5 hexists:用于查看哈希表中是否存在该字段

6 hlen:查看哈希表中全部key的数量

7 hdel:删除指定哈希表中的字段

8 hkeys:查看哈希中全部的key，只返回key

9 hvals:查看哈希表中全部key对应的值，只返回值

10 hgetall:查看哈希表中key和对应值，返回key和对应的值
```
127.0.0.1:6379> hmset myhash username xiaoming age 10 sex 1
OK
127.0.0.1:6379> hgetall myhash
1) "username"
2) "xiaoming"
3) "age"
4) "10"
5) "sex"
6) "1"
127.0.0.1:6379>
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExMjM3NDg1MTVdfQ==
-->