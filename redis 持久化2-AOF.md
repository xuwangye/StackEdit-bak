<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>redis 持久化2-AOF</title>
  <link rel="stylesheet" href="https://stackedit.io/style.css" />
</head>

<body class="stackedit">
  <div class="stackedit__html"><h2 id="一、aof持久化">一、AOF持久化</h2>
<p>RDB持久化是将进程数据写入文件，而AOF持久化(即Append Only File持久化)，则是将Redis执行的每次写命令记录到单独的日志文件中（有点像MySQL的binlog）；当Redis重启时再次执行AOF文件中的命令来恢复数据。</p>
<p>与RDB相比，AOF的实时性更好，因此已成为主流的持久化方案。</p>
<h3 id="、开启aof">1、开启AOF</h3>
<p>Redis服务器默认开启RDB，关闭AOF；要开启AOF，需要在配置文件中配置：</p>
<blockquote>
<p>appendonly yes</p>
</blockquote>
<h3 id="、执行流程">2、执行流程</h3>
<p>由于需要记录Redis的每条写命令，因此AOF不需要触发，下面介绍AOF的执行流程。</p>
<p>AOF的执行流程包括：</p>
<ul>
<li>
<p>命令追加(append)：将Redis的写命令追加到缓冲区aof_buf；</p>
</li>
<li>
<p>文件写入(write)和文件同步(sync)：根据不同的同步策略将aof_buf中的内容同步到硬盘；</p>
</li>
<li>
<p>文件重写(rewrite)：定期重写AOF文件，达到压缩的目的。</p>
</li>
</ul>
<p><strong>1) 命令追加(append)</strong><br>
Redis先将写命令追加到缓冲区，而不是直接写入文件，主要是为了避免每次有写命令都直接写入硬盘，导致硬盘IO成为Redis负载的瓶颈。</p>
<p>命令追加的格式是Redis命令请求的协议格式，它是一种纯文本格式，具有兼容性好、可读性强、容易处理、操作简单避免二次开销等优点；具体格式略。在AOF文件中，除了用于指定数据库的select命令（如select 0 为选中0号数据库）是由Redis添加的，其他都是客户端发送来的写命令。</p>
<p><strong>2) 文件写入(write)和文件同步(sync)</strong><br>
Redis提供了多种AOF缓存区的同步文件策略，策略涉及到操作系统的write函数和fsync函数，说明如下：</p>
<p>为了提高文件写入效率，在现代操作系统中，当用户调用write函数将数据写入文件时，操作系统通常会将数据暂存到一个内存缓冲区里，当缓冲区被填满或超过了指定时限后，才真正将缓冲区的数据写入到硬盘里。这样的操作虽然提高了效率，但也带来了安全问题：如果计算机停机，内存缓冲区中的数据会丢失；因此系统同时提供了fsync、fdatasync等同步函数，可以强制操作系统立刻将缓冲区中的数据写入到硬盘里，从而确保数据的安全性。</p>
<p>AOF缓存区的同步文件策略由参数<strong>appendfsync</strong>控制，各个值的含义如下：</p>
<p><strong>always</strong>：命令写入aof_buf后立即调用系统fsync操作同步到AOF文件，fsync完成后线程返回。这种情况下，每次有写命令都要同步到AOF文件，硬盘IO成为性能瓶颈，Redis只能支持大约几百TPS写入，严重降低了Redis的性能；即便是使用固态硬盘（SSD），每秒大约也只能处理几万个命令，而且会大大降低SSD的寿命。</p>
<p><strong>no</strong>：命令写入aof_buf后调用系统write操作，不对AOF文件做fsync同步；同步由操作系统负责，通常同步周期为30秒。这种情况下，文件同步的时间不可控，且缓冲区中堆积的数据会很多，数据安全性无法保证。</p>
<p><strong>everysec</strong>：命令写入aof_buf后调用系统write操作，write完成后线程返回；fsync同步文件操作由专门的线程每秒调用一次。everysec是前述两种策略的折中，是性能和数据安全性的平衡，因此是Redis的默认配置，也是我们推荐的配置。</p>
<p><strong>3) 文件重写(rewrite)</strong></p>
<p>随着时间流逝，Redis服务器执行的写命令越来越多，AOF文件也会越来越大；过大的AOF文件不仅会影响服务器的正常运行，也会导致数据恢复需要的时间过长。</p>
<p>文件重写是指定期重写AOF文件，减小AOF文件的体积。需要注意的是，AOF重写是把Redis进程内的数据转化为写命令，同步到新的AOF文件，不会对旧的AOF文件进行任何读取、写入操作。</p>
<p>关于文件重写需要注意的另一点是：对于AOF持久化来说，文件重写虽然是强烈推荐的，但并不是必须的。即使没有文件重写，数据也可以被持久化并在Redis启动的时候导入。因此在一些实现中，会关闭自动的文件重写，然后通过定时任务在每天的某一时刻定时执行。</p>
<p><strong>文件重写之所以能够压缩AOF文件</strong>，原因在于：</p>
<ol>
<li><strong>过期的数据不再写入文件</strong></li>
<li><strong>无效的命令不再写入文件</strong>：如有些数据被重复设值(set mykey v1, set mykey v2)、有些数据被删除了(sadd myset v1, del myset)等等</li>
<li><strong>多条命令可以合并为一个</strong>：如sadd myset v1, sadd myset v2, sadd myset v3可以合并为sadd myset v1 v2 v3。不过为了防止单条命令过大造成客户端缓冲区溢出，对于list、set、hash、zset类型的key，并不一定只使用一条命令；而是以某个常量为界将命令拆分为多条。这个常量在redis.h/REDIS_AOF_REWRITE_ITEMS_PER_CMD中定义，不可更改，3.0版本中值是64。<br>
<img src="https://upload-images.jianshu.io/upload_images/11450903-c9d1a5df7d311328?imageMogr2/auto-orient/strip%7CimageView2/2/w/354/format/webp" alt=""></li>
</ol>
<p>通过上述内容可以看出，由于重写后AOF执行的命令减少了，文件重写既可以减少文件占用的空间，也可以加快恢复速度。</p>
<p><strong>文件重写的触发</strong><br>
文件重写的触发，分为手动触发和自动触发：</p>
<ul>
<li>手动触发：直接调用bgrewriteaof命令，该命令的执行与bgsave有些类似：都是fork子进程进行具体的工作，且都只有在fork时阻塞。<br>
<img src="https://upload-images.jianshu.io/upload_images/11450903-03b9d9dfbccdd47b?imageMogr2/auto-orient/strip%7CimageView2/2/w/364/format/webp" alt=""></li>
</ul>
<p>自动触发：根据auto-aof-rewrite-min-size和auto-aof-rewrite-percentage参数，以及aof_current_size和aof_base_size状态确定触发时机。</p>
<p>auto-aof-rewrite-min-size：执行AOF重写时，文件的最小体积，默认值为64MB。</p>
<p>auto-aof-rewrite-percentage：执行AOF重写时，当前AOF大小(即aof_current_size)和上一次重写时AOF大小(aof_base_size)的比值。</p>
<p>其中，参数可以通过config get命令查看：<br>
<img src="https://upload-images.jianshu.io/upload_images/11450903-d5a473e8e3ca714f?imageMogr2/auto-orient/strip%7CimageView2/2/w/438/format/webp" alt=""></p>
<p>状态可以通过info persistence查看：<br>
<img src="https://upload-images.jianshu.io/upload_images/11450903-220376e42551543a?imageMogr2/auto-orient/strip%7CimageView2/2/w/239/format/webp" alt=""></p>
<p>只有当auto-aof-rewrite-min-size和auto-aof-rewrite-percentage两个参数同时满足时，才会自动触发AOF重写，即bgrewriteaof操作。</p>
<p><strong>文件重写的流程</strong><br>
<img src="https://upload-images.jianshu.io/upload_images/11450903-c3f8e583f858e995?imageMogr2/auto-orient/strip%7CimageView2/2/w/379/format/webp" alt=""></p>
<p>关于文件重写的流程，有两点需要特别注意：</p>
<p>重写由父进程fork子进程进行；</p>
<p>重写期间Redis执行的写命令，需要追加到新的AOF文件中，为此Redis引入了aof_rewrite_buf缓存。</p>
<p>对照上图，文件重写的流程如下：</p>
<p>Redis父进程首先判断当前是否存在正在执行 bgsave/bgrewriteaof的子进程，如果存在则bgrewriteaof命令直接返回，如果存在bgsave命令则等bgsave执行完成后再执行。前面曾介绍过，这个主要是基于性能方面的考虑。</p>
<p>父进程执行fork操作创建子进程，这个过程中父进程是阻塞的。</p>
<p>父进程fork后，bgrewriteaof命令返回”Background append only file rewrite started”信息并不再阻塞父进程，并可以响应其他命令。Redis的所有写命令依然写入AOF缓冲区，并根据appendfsync策略同步到硬盘，保证原有AOF机制的正确。</p>
<p>由于fork操作使用写时复制技术，子进程只能共享fork操作时的内存数据。由于父进程依然在响应命令，因此Redis使用AOF重写缓冲区(图中的aof_rewrite_buf)保存这部分数据，防止新AOF文件生成期间丢失这部分数据。也就是说，bgrewriteaof执行期间，Redis的写命令同时追加到aof_buf和aof_rewirte_buf两个缓冲区。</p>
<p>子进程根据内存快照，按照命令合并规则写入到新的AOF文件。</p>
<p>子进程写完新的AOF文件后，向父进程发信号，父进程更新统计信息，具体可以通过info persistence查看。</p>
<p>父进程把AOF重写缓冲区的数据写入到新的AOF文件，这样就保证了新AOF文件所保存的数据库状态和服务器当前状态一致。</p>
<p>使用新的AOF文件替换老文件，完成AOF重写。</p>
<h3 id="、启动时加载">3、启动时加载</h3>
<p>前面提到过，当AOF开启时，Redis启动时会优先载入AOF文件来恢复数据；只有当AOF关闭时，才会载入RDB文件恢复数据。</p>
<p>当AOF开启，但AOF文件不存在时，即使RDB文件存在也不会加载(更早的一些版本可能会加载，但3.0不会)</p>
<p><strong>文件校验</strong><br>
与载入RDB文件类似，Redis载入AOF文件时，会对AOF文件进行校验，如果文件损坏，则日志中会打印错误，Redis启动失败。但如果是AOF文件结尾不完整(机器突然宕机等容易导致文件尾部不完整)，且aof-load-truncated参数开启，则日志中会输出警告，Redis忽略掉AOF文件的尾部，启动成功。aof-load-truncated参数默认是开启的：</p>
<p><img src="https://upload-images.jianshu.io/upload_images/11450903-048dca57ed553aa3?imageMogr2/auto-orient/strip%7CimageView2/2/w/369/format/webp" alt=""></p>
<p><strong>伪客户端</strong><br>
因为Redis的命令只能在客户端上下文中执行，而载入AOF文件时命令是直接从文件中读取的，并不是由客户端发送；因此Redis服务器在载入AOF文件之前，会创建一个没有网络连接的客户端，之后用它来执行AOF文件中的命令，命令执行的效果与带网络连接的客户端完全一样。</p>
<h3 id="、aof常用配置总结">4、AOF常用配置总结</h3>
<p>下面是AOF常用的配置项，以及默认值；前面介绍过的这里不再详细介绍。</p>
<p>appendonly no：是否开启AOF</p>
<p>appendfilename “appendonly.aof”：AOF文件名</p>
<p>dir ./：RDB文件和AOF文件所在目录</p>
<p>appendfsync everysec：fsync持久化策略</p>
<p>no-appendfsync-on-rewrite no：AOF重写期间是否禁止fsync；如果开启该选项，可以减轻文件重写时CPU和硬盘的负载（尤其是硬盘），但是可能会丢失AOF重写期间的数据；需要在负载和安全性之间进行平衡</p>
<p>auto-aof-rewrite-percentage 100：文件重写触发条件之一</p>
<p>auto-aof-rewrite-min-size 64mb：文件重写触发提交之一</p>
<p>aof-load-truncated yes：如果AOF文件结尾损坏，Redis启动时是否仍载入AOF文件</p>
<h3 id="五、方案选择与常见问题"><strong>五、方案选择与常见问题</strong></h3>
</div>
</body>

</html>
