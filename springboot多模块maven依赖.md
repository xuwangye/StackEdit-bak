---


---

<p>dependencyManagement 里面的依赖不会真实的去下载jar包，如果子模块中的jar没有指定，就会到父类的中寻找对应的version，如果没有报错！</p>
<p>spring-boot-dependencies与spring-cloud-dependencies就相当于boot与cloud的不同版本，里面集合了很多配套的jar包</p>
<pre class=" language-java"><code class="prism  language-java"><span class="token operator">&lt;</span>dependencyManagement<span class="token operator">&gt;</span>
<span class="token operator">&lt;</span>dependencies<span class="token operator">&gt;</span>  
	<span class="token operator">&lt;</span><span class="token operator">!</span><span class="token operator">--</span> 这里是spring boot 的版本，里面有很多很多jar包 <span class="token operator">--</span><span class="token operator">&gt;</span>  
	<span class="token operator">&lt;</span>dependency<span class="token operator">&gt;</span>  
	    <span class="token operator">&lt;</span>groupId<span class="token operator">&gt;</span>org<span class="token punctuation">.</span>springframework<span class="token punctuation">.</span>boot<span class="token operator">&lt;</span><span class="token operator">/</span>groupId<span class="token operator">&gt;</span>  
	    <span class="token operator">&lt;</span>artifactId<span class="token operator">&gt;</span>spring<span class="token operator">-</span>boot<span class="token operator">-</span>dependencies<span class="token operator">&lt;</span><span class="token operator">/</span>artifactId<span class="token operator">&gt;</span>  
	    <span class="token operator">&lt;</span>version<span class="token operator">&gt;</span><span class="token number">2.0</span><span class="token punctuation">.</span><span class="token number">8</span><span class="token punctuation">.</span>RELEASE<span class="token operator">&lt;</span><span class="token operator">/</span>version<span class="token operator">&gt;</span>  
	    <span class="token operator">&lt;</span>type<span class="token operator">&gt;</span>pom<span class="token operator">&lt;</span><span class="token operator">/</span>type<span class="token operator">&gt;</span>  
	    <span class="token operator">&lt;</span>scope<span class="token operator">&gt;</span><span class="token keyword">import</span><span class="token operator">&lt;</span><span class="token operator">/</span>scope<span class="token operator">&gt;</span>  
	<span class="token operator">&lt;</span><span class="token operator">/</span>dependency<span class="token operator">&gt;</span>  
	  
	<span class="token operator">&lt;</span><span class="token operator">!</span><span class="token operator">--</span> 这里是spring cloud 的版本，里面有很多很多jar包 <span class="token operator">--</span><span class="token operator">&gt;</span>  
	<span class="token operator">&lt;</span>dependency<span class="token operator">&gt;</span>  
	    <span class="token operator">&lt;</span>groupId<span class="token operator">&gt;</span>org<span class="token punctuation">.</span>springframework<span class="token punctuation">.</span>cloud<span class="token operator">&lt;</span><span class="token operator">/</span>groupId<span class="token operator">&gt;</span>  
	    <span class="token operator">&lt;</span>artifactId<span class="token operator">&gt;</span>spring<span class="token operator">-</span>cloud<span class="token operator">-</span>dependencies<span class="token operator">&lt;</span><span class="token operator">/</span>artifactId<span class="token operator">&gt;</span>  
	    <span class="token operator">&lt;</span>version<span class="token operator">&gt;</span>$<span class="token punctuation">{</span>spring<span class="token operator">-</span>cloud<span class="token punctuation">.</span>version<span class="token punctuation">}</span><span class="token operator">&lt;</span><span class="token operator">/</span>version<span class="token operator">&gt;</span>  
	    <span class="token operator">&lt;</span>type<span class="token operator">&gt;</span>pom<span class="token operator">&lt;</span><span class="token operator">/</span>type<span class="token operator">&gt;</span>  
	    <span class="token operator">&lt;</span>scope<span class="token operator">&gt;</span><span class="token keyword">import</span><span class="token operator">&lt;</span><span class="token operator">/</span>scope<span class="token operator">&gt;</span>  
	<span class="token operator">&lt;</span><span class="token operator">/</span>dependency<span class="token operator">&gt;</span>
<span class="token operator">&lt;</span><span class="token operator">/</span>dependencies<span class="token operator">&gt;</span>  
<span class="token operator">&lt;</span><span class="token operator">/</span>dependencyManagement<span class="token operator">&gt;</span>
</code></pre>

