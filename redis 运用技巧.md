1 **使用过Redis分布式锁么，它是怎么实现的？**
先拿setnx来争抢锁，抢到之后，再用expire给锁加一个过期时间防止锁忘记了释放。

**如果在setnx之后执行expire之前进程意外crash或者要重启维护了，那会怎么样？**
set指令有非常复杂的参数，这个应该是可以同时把setnx和expire合成一条指令来用的！
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk0MTk3MDQzXX0=
-->