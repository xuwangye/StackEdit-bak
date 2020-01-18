<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>redis 持久化1-RDB</title>
  <link rel="stylesheet" href="https://stackedit.io/style.css" />
</head>

<body class="stackedit">
  <div class="stackedit__html"><p><strong>Redis高可用概述</strong><br>
我们知道，在Web服务器中，高可用是指服务器可以正常访问的时间，衡量的标准是在多长时间内可以提供正常服务（99.9%、99.99%、99.999%等等）。但是在Redis语境中，高可用的含义似乎要宽泛一些，除了保证提供正常服务（如主从分离、快速容灾技术等），还需要考虑数据容量的扩展、数据安全不会丢失等。</p>
<p>在Redis中，实现高可用的技术主要包括持久化、复制、哨兵和集群，下面分别说明它们的作用，以及解决了什么样的问题：</p>
<p><strong>持久化</strong>：有时甚至不被归为高可用的手段，主要作用是数据备份，即将数据存储在硬盘，保证数据不会因进程退出而丢失。<br>
(最低级的高可用技术)</p>
<p><strong>复制</strong>：复制主要实现了数据的多机备份以及对于读操作的负载均衡和简单的故障恢复。缺陷是故障恢复无法自动化、写操作无法负载均衡、存储能力受到单机的限制。<br>
(复制是高可用Redis的基础，哨兵和集群都是在复制基础上实现高可用的。)</p>
<p><strong>哨兵</strong>：在复制的基础上，哨兵实现了自动化的故障恢复。缺陷是写操作无法负载均衡、存储能力受到单机的限制。</p>
<p><strong>集群</strong>：通过集群，Redis解决了写操作无法负载均衡以及存储能力受到单机限制的问题，实现了较为完善的高可用方案。</p>
<h2 id="一、rdb持久化">一、RDB持久化</h2>
<p>RDB持久化是将当前进程中的数据生成快照保存到硬盘（因此也称作快照持久化），保存的文件后缀是<strong>rdb</strong>；当Redis重新启动时，可以读取快照文件恢复数据。</p>
<h3 id="、触发条件">1、触发条件</h3>
<p>RDB持久化的触发分为手动触发和自动触发两种。</p>
<ul>
<li><strong>手动触发</strong><br>
save命令和bgsave命令都可以生成RDB文件。<br>
save命令：会阻塞Redis服务器进程，直到RDB文件创建完毕为止，在Redis服务器阻塞期间，服务器不能处理任何命令请求。<br>
bgsave命令：会创建一个子进程，由子进程来负责创建RDB文件，父进程(即Redis主进程)则继续处理请求。</li>
</ul>
<p><strong>bgsave命令执行过程中，只有fork子进程时会阻塞服务器，而对于save命令，整个过程都会阻塞服务器，因此save已基本被废弃，线上环境要杜绝save的使用</strong>，后文中也将只介绍bgsave命令。此外，在自动触发RDB持久化时，Redis也会选择bgsave而不是save来进行持久化。下面介绍自动触发RDB持久化的条件。</p>
<ul>
<li><strong>自动触发</strong><br>
save m n<br>
自动触发最常见的情况是在配置文件中通过save m n，指定当m秒内发生n次变化时，会触发<strong>bgsave</strong>。</li>
</ul>
<p>例如，查看Redis的默认配置文件(Linux下为Redis根目录下的redis.conf)，可以看到如下配置信息：<br>
<img src="https://upload-images.jianshu.io/upload_images/11450903-9288d7402834954e?imageMogr2/auto-orient/strip%7CimageView2/2/w/554/format/webp" alt=""><br>
其中save 900 1的含义是：当时间满900秒时，如果Redis数据发生了至少1次变化，则执行bgsave；save 300 10和save 60 10000同理。当三个save条件满足任意一个时，都会引起bgsave的调用。</p>
<p><strong>save m n的实现原理</strong><br>
Redis的save m n，是通过serverCron函数、dirty计数器、和lastsave时间戳来实现的。</p>
<p>serverCron是Redis服务器的周期性操作函数，默认每隔100ms执行一次；该函数对服务器的状态进行维护，其中一项工作就是检查 save m n 配置的条件是否满足，如果满足就执行bgsave。</p>
<p>dirty计数器是Redis服务器维持的一个状态，记录了上一次执行bgsave/save命令后，服务器状态进行了多少次修改(包括增删改)；而当save/bgsave执行完成后，会将dirty重新置为0。<br>
例如，如果Redis执行了set mykey helloworld，则dirty值会+1；如果执行了sadd myset v1 v2 v3，则dirty值会+3；注意dirty记录的是服务器进行了多少次修改，而不是客户端执行了多少修改数据的命令。</p>
<p>lastsave时间戳也是Redis服务器维持的一个状态，记录的是上一次成功执行save/bgsave的时间。</p>
<p><em><strong>一个是周期扫描(serverCron)，一个是时间记录(lastsave)，一个是次数记录(dirty)，合在一起就变成了save m n</strong></em></p>
<p>save m n的原理如下：每隔100ms，执行serverCron函数；在serverCron函数中，遍历save m n配置的保存条件，只要有一个条件满足，就进行bgsave。对于每一个save m n条件，只有m 和 n两条同时满足时才算满足如下：</p>
<ul>
<li>当前时间-lastsave &gt; m</li>
<li>dirty &gt;= n</li>
</ul>
<p>下图是save m n触发bgsave执行时，服务器打印日志的情况：<br>
<img src="https://upload-images.jianshu.io/upload_images/11450903-f1df9ee3d74c70e2?imageMogr2/auto-orient/strip%7CimageView2/2/w/554/format/webp" alt=""></p>
<p>除了save m n以外，还有一些其他情况会触发bgsave：</p>
<ol>
<li>在主从复制场景下，如果从节点执行全量复制操作，则主节点会执行bgsave命令，并将rdb文件发送给从节点；</li>
<li>执行shutdown命令时，自动执行rdb持久化</li>
</ol>
<h3 id="、执行流程">2、执行流程</h3>
<p>前面介绍了触发bgsave的条件，下面将说明bgsave命令的执行流程，如下图所示：<br>
<img src="https://upload-images.jianshu.io/upload_images/11450903-545686bb372f50ca?imageMogr2/auto-orient/strip%7CimageView2/2/w/554/format/webp" alt=""></p>
<p>图片中的5个步骤所进行的操作如下：<br>
Redis父进程首先判断：当前是否在执行save、bgsave、bgrewriteaof（后面会详细介绍该命令）的子进程，如果在执行则bgsave命令直接返回(bgsave/bgrewriteaof 的子进程不能同时执行）。主要是基于性能方面的考虑：两个并发的子进程同时执行大量的磁盘写操作，可能引起严重的性能问题。</p>
<p>父进程执行fork操作创建子进程，这个过程中父进程是阻塞的，Redis不能执行来自客户端的任何命令；</p>
<p>父进程fork后，bgsave命令返回”Background saving started”信息并不再阻塞父进程，并可以响应其他命令；</p>
<p>子进程创建RDB文件，根据父进程内存快照生成临时快照文件，完成后对原有文件进行原子替换；</p>
<p>子进程发送信号给父进程表示完成，父进程更新统计信息。</p>
<h3 id="、rdb文件">3、RDB文件</h3>
<p>RDB文件是经过压缩的二进制文件，下面介绍关于RDB文件的一些细节。</p>
<ul>
<li>
<p>存储路径<br>
RDB文件的存储路径既可以在启动前配置，也可以通过命令动态设定。<br>
配置：dir配置指定目录，dbfilename指定文件名。默认是Redis根目录下的dump.rdb文件。<br>
动态设定：Redis启动后也可以动态修改RDB存储路径，在磁盘损害或空间不足时非常有用；执行命令为config set dir {newdir}和config set dbfilename {newFileName}。如下所示(Windows环境)：<br>
<img src="https://upload-images.jianshu.io/upload_images/11450903-767d2a41f5ea05ad?imageMogr2/auto-orient/strip%7CimageView2/2/w/378/format/webp" alt=""></p>
</li>
<li>
<p>RDB文件格式<br>
RDB文件格式如下图所示（图片来源：《Redis设计与实现》）：<br>
<img src="https://upload-images.jianshu.io/upload_images/11450903-650a00f5917631c1?imageMogr2/auto-orient/strip%7CimageView2/2/w/554/format/webp" alt=""></p>
</li>
</ul>
<p>其中各个字段的含义说明如下：</p>
<p>REDIS：常量，保存着“REDIS”5个字符。</p>
<p>db_version：RDB文件的版本号，注意不是Redis的版本号。</p>
<p>SELECTDB 0 pairs：表示一个完整的数据库(0号数据库)，同理SELECTDB 3 pairs表示完整的3号数据库；只有当数据库中有键值对时，RDB文件中才会有该数据库的信息(上图所示的Redis中只有0号和3号数据库有键值对)；如果Redis中所有的数据库都没有键值对，则这一部分直接省略。其中：SELECTDB是一个常量，代表后面跟着的是数据库号码；0和3是数据库号码；pairs则存储了具体的键值对信息，包括key、value值，及其数据类型、内部编码、过期时间、压缩信息等等。</p>
<p>EOF：常量，标志RDB文件正文内容结束。</p>
<p>check_sum：前面所有内容的校验和；Redis在载入RBD文件时，会计算前面的校验和并与check_sum值比较，判断文件是否损坏。</p>
<p>压缩<br>
Redis默认采用LZF算法对RDB文件进行压缩。虽然压缩耗时，但是可以大大减小RDB文件的体积，因此压缩默认开启；可以通过命令关闭：</p>
<blockquote>
<p>config set rdbcompression no</p>
</blockquote>
<p>需要注意的是，RDB文件的压缩并不是针对整个文件进行的，而是对数据库中的字符串进行的，且只有在字符串达到一定长度(20字节)时才会进行。</p>
<h3 id="、启动时加载">4、启动时加载</h3>
<p>RDB文件的载入工作是在服务器启动时自动执行的，并没有专门的命令。但是由于AOF的优先级更高，因此当AOF开启时，Redis会优先载入AOF文件来恢复数据；只有当AOF关闭时，才会在Redis服务器启动时检测RDB文件，并自动载入。服务器载入RDB文件期间处于阻塞状态，直到载入完成为止。</p>
<p>Redis启动日志中可以看到自动载入的执行：<br>
<img src="https://upload-images.jianshu.io/upload_images/11450903-f6a5ec6439b6f3de?imageMogr2/auto-orient/strip%7CimageView2/2/w/554/format/webp" alt=""></p>
<p>Redis载入RDB文件时，会对RDB文件进行校验，如果文件损坏，则日志中会打印错误，Redis启动失败。</p>
<h3 id="、rdb常用配置总结">5、RDB常用配置总结</h3>
<p>下面是RDB常用的配置项，以及默认值（前面介绍过的这里不再详细介绍）：</p>
<p>save m n：bgsave自动触发的条件；如果没有save m n配置，相当于自动的RDB持久化关闭，不过此时仍可以通过其他方式触发。</p>
<p>stop-writes-on-bgsave-error yes：当bgsave出现错误时，Redis是否停止执行写命令；设置为yes，则当硬盘出现问题时，可以及时发现，避免数据的大量丢失；设置为no，则Redis无视bgsave的错误继续执行写命令，当对Redis服务器的系统(尤其是硬盘)使用了监控时，该选项考虑设置为no。</p>
<p>rdbcompression yes：是否开启RDB文件压缩。</p>
<p>rdbchecksum yes：是否开启RDB文件的校验，在写入文件和读取文件时都起作用；关闭checksum在写入文件和启动文件时大约能带来10%的性能提升，但是数据损坏时无法发现。</p>
<p>dbfilename dump.rdb：RDB文件名。</p>
<p>dir ./：RDB文件和AOF文件所在目录。</p>
<p>参考：<a href="https://www.jianshu.com/p/c0da5443b527">https://www.jianshu.com/p/c0da5443b527</a></p>
</div>
</body>

</html>
