1 lpush :将一个元素或者多个元素新增到列表的头部，返回当前list中元素数量，采用的是 栈。

2 lrange:返回list集合中指定区间中的元素，0下标代表第一个元素，-1代表最后一个元素。

3 rpush:将一个元素或者多个元素新增到列表的尾部，返回当前list中元素数量，采用的是 队列。

4 linsert:在指定列表的元素前或者后插入元素,以下是在bird元素之前插入tiger元素
```
127.0.0.1:6379> lrange mylist2 0 -1
1) "bird"
2) "elephant"
127.0.0.1:6379> linsert mylist2 before bird tiger
(integer) 3
127.0.0.1:6379> lrange mylist2 0 -1
1) "tiger"
2) "bird"
3) "elephant"
127.0.0.1:6379> 
```
5 lset:设置list列表中指定元素下标的值, 下标0代表第一个元素，替换成了bear。
```
127.0.0.1:6379> lrange mylist2 0 -1
1) "tiger"
2) "bird"
3) "elephant"
127.0.0.1:6379> lset mylist2 0 bear
OK
127.0.0.1:6379> lrange mylist2 0 -1
1) "bear"
2) "bird"
3) "elephant"
127.0.0.1:6379> 
```

6 lrem:从对应list列表中删除n和value相同的元素，"lrem list 1 noe"代表在list列表中删除1和noe相同的元素。
```
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "tow"
3) "noe"
4) "noe"
127.0.0.1:6379> lrem list 1 noe
(integer) 1
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "tow"
3) "noe"
127.0.0.1:6379>
```

7 ltrim:保留指定key值范围内的数据，已知list列表有三个元素，现在只想保留前两个，最后显示tow已经去掉了
```
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "noe"
3) "tow"
127.0.0.1:6379> ltrim list 0 1
OK
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "noe"
127.0.0.1:6379> 
```
8 lpop:从列表头部删除一个元素，返回删除的元素值。
```
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "noe"
127.0.0.1:6379> lpop list
"three"
127.0.0.1:6379> lrange list 0 -1
1) "noe"
127.0.0.1:6379>
```
9 rpop:从列表尾部删除一个元素，返回删除的元素值。

10 rpoplpush:从第一个列表尾部移除一个元素并且添加到第二个列表中头部

11 lindex: 返回列表中指定下标的元素值，元素下标0开始

12 llen:返回列表中元素的个数
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc4MDAzODMwXX0=
-->