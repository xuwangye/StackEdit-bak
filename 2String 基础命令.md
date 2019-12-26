### **String数据类型常用命令**
1 setnx:用于给指定的key设置value，如果key已经存在则返回0。nx代表:not exist

2 setex:用于给指定的key设置value，并且需要指定该key的有效时间。 10秒后则返回为空

3 setrange:给指定的key的值重新覆盖内容，从4指定位置，替换内容为ming，最终结果为xiaoming
```
111.231.51.81:6379> get name
"xiaowang"
111.231.51.81:6379> setrange name 4 ming
(integer) 8
111.231.51.81:6379> get name
"xiaoming"
```
4 mset:批量设置key对应的value值，以下设置username、age、sex 分别对应 wangwu、10、1
```
111.231.51.81:6379> mset username wangwu age 10 sex 1  
OK  
111.231.51.81:6379>
```
5 msetnx:批量设置key对应的value值，如果key已存在则返回0。由于以上设置过，则结果返回0

6 getset:给指定key设置新值，并且返回之前原始数据。

7 getrange:返回一个字符串的子字符串，相当于字符串截取，下标0是起始位置，下标3是结尾位置

8 mget:批量获取key对应的value，按顺序展示
```
111.231.51.81:6379> mget username age sex  
1) "wangwu"  
2) "10"  
3) "1"  
111.231.51.81:6379>
```
10 incr:将指定key中存储的数字值增一，必须是数字类型，否则会返回错误信息

11 incrby:将指定key中存储的数字指定增加多少，以下指定增长10
```
111.231.51.81:6379> get age
"12"
111.231.51.81:6379> incrby age 10
(integer) 22
111.231.51.81:6379>
```

12 decr:将指定key中存储的数字值减一，必须是数字类型，否则会返回错误信息

13 decrby:将指定key中存储的数字指定减少多少，以下指定减少10

14 append:给指定的key中的值追加字符串

15 strlen:返回指定key中value的长度
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MDAzMzk4NTVdfQ==
-->