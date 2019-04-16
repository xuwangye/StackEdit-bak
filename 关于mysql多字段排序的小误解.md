---


---

<p>select * from user order by 字段1 ，字段2 desc</p>
<p>这时候字段1其实默认的时asc排序的，字段2是desc。<br>
字段二的排序方法并不会使的字段一也desc排序。</p>
<p>下面才是真正的字段1desc，字段2desc<br>
select * from user order by 字段1 <font color="red">desc</font>，字段2 <font color="red">desc</font></p>

