---


---

<p>在spring boot里面，多模块让各模块使用自己的配置可以多创建几个application-redis.yml、application-log.yml等</p>
<pre class=" language-yml"><code class="prism  language-yml"><span class="token key atrule">application.yml</span><span class="token punctuation">:</span>
<span class="token key atrule">spring</span><span class="token punctuation">:</span>  
  <span class="token key atrule">profiles</span><span class="token punctuation">:</span>  
    <span class="token key atrule">active</span><span class="token punctuation">:</span> dev
</code></pre>
<h2 id="application.yml-与application-dev.yml-一点小小的区别">application.yml 与application-dev.yml 一点小小的区别</h2>
<ul>
<li>
<p><font size="3pt" color="red"><b>application.yml 是启动文件，所以配置在各模块中不会被扫描  </b></font></p>
</li>
<li>
<p><font size="3pt" color="red"><b>application-dev.yml 是环境文件，会被扫描。 </b></font></p>
</li>
</ul>

