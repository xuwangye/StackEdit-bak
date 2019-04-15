---


---

<p>match operator and 与 or<br>
and</p>
<h2 id="es排序">es排序</h2>
<p>_score 为 null 的情况，匹配度无法确认（可能最不匹配的排第一位）</p>
<pre class=" language-java"><code class="prism  language-java"><span class="token string">"sort"</span><span class="token operator">:</span> <span class="token punctuation">[</span>
    <span class="token punctuation">{</span>
      <span class="token string">"publishTime"</span><span class="token operator">:</span> <span class="token punctuation">{</span>
        <span class="token string">"order"</span><span class="token operator">:</span> <span class="token string">"desc"</span>
      <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">]</span>
</code></pre>
<p>要求排序的同时，还要求最匹配的排第一，这时候就要指定_score排序</p>
<pre class=" language-java"><code class="prism  language-java"><span class="token string">"sort"</span><span class="token operator">:</span> <span class="token punctuation">[</span>
    <span class="token punctuation">{</span>
      <span class="token string">"_score"</span><span class="token operator">:</span> <span class="token punctuation">{</span>
        <span class="token string">"order"</span><span class="token operator">:</span> <span class="token string">"desc"</span>
      <span class="token punctuation">}</span><span class="token punctuation">,</span>
      <span class="token string">"publishTime"</span><span class="token operator">:</span> <span class="token punctuation">{</span>
        <span class="token string">"order"</span><span class="token operator">:</span> <span class="token string">"desc"</span>
      <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">]</span>
</code></pre>
<p>代码实现分数多字段排序：</p>
<pre class=" language-java"><code class="prism  language-java"><span class="token comment">//查询结果集的builder  </span>
SearchRequestBuilder searchRequestBuilder <span class="token operator">=</span> transportClient<span class="token punctuation">.</span><span class="token function">prepareSearch</span><span class="token punctuation">(</span>index<span class="token punctuation">)</span>  
        <span class="token punctuation">.</span><span class="token function">setTypes</span><span class="token punctuation">(</span>index<span class="token punctuation">)</span>  
        <span class="token punctuation">.</span><span class="token function">setQuery</span><span class="token punctuation">(</span>boolQueryBuilder<span class="token punctuation">)</span>  
        <span class="token punctuation">.</span><span class="token function">addSort</span><span class="token punctuation">(</span><span class="token keyword">new</span> <span class="token class-name">ScoreSortBuilder</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span>  <span class="token comment">//分数排序 默认desc</span>
        <span class="token punctuation">.</span><span class="token function">setFrom</span><span class="token punctuation">(</span>resourcePageSearcher<span class="token punctuation">.</span><span class="token function">getPageNum</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">&gt;=</span> <span class="token number">1</span> <span class="token operator">?</span> resourcePageSearcher<span class="token punctuation">.</span><span class="token function">getPageNum</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">-</span> <span class="token number">1</span> <span class="token operator">:</span> <span class="token number">0</span><span class="token punctuation">)</span>  
        <span class="token punctuation">.</span><span class="token function">setSize</span><span class="token punctuation">(</span>resourcePageSearcher<span class="token punctuation">.</span><span class="token function">getPageSize</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<h2 id="es-查看字符串分词情况">es 查看字符串分词情况</h2>
<p>post   /_analyze<br>
{<br>
“text”:“需求5号”,<br>
“analyzer”:“ik_max_word”<br>
}</p>
<p><img src="https://i.loli.net/2019/04/09/5cac639ed4127.png" alt="69C4912A-73AC-4fa3-9ACB-5AA01B0E5A49.png"></p>

