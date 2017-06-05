---
layout: post
title: Optimizing SELECT * FROM with mysql_field_table() — speed impact results
permalink: blog/2009/11/11/optimizing-select-from-with-mysql_field_table-—-speed-impact-results/
---

<p>In my previous post, <a href="/blog/2009/11/01/optimizing-select-from-with-mysql_field_table/">Optimizing SELECT * FROM with mysql_field_table()</a>, I ended with the promise to publish some speed impact results with you. I found it hard to come up with a good measure of speed impact. On the one hand, there is the speed degradation as extra overhead is added. On the other hand, speed is improved as you execute less database queries.<span id="more-53"></span></p>

<h1>ApacheBench</h1>
<p>ApacheBench was used for measuring the performance of the different approaches. A total of 1000 requests were executed, and then the mean time of response calculated.</p>
<blockquote><p><em>ab</em> is a tool for benchmarking your Apache Hypertext Transfer Protocol (HTTP) server. It is designed to give you an impression of how your current Apache installation performs. This especially shows you how many requests per second your Apache installation is capable of serving.</p>
<p><cite>From: <a onclick="javascript:pageTracker._trackPageview('/outgoing/httpd.apache.org/docs/2.0/programs/ab.html');"  href="/web/20091123125826/http://httpd.apache.org/docs/2.0/programs/ab.html">apache.org</a></cite></p></blockquote>
<h2>Approach 1: no join</h2>
<p>This is the 'traditional approach'. The data is requested on a per-instance basis. In terms of the example in my previous post, there is 1 query requesting all posts and a query for every author. This approach is heavy on the database, less on the code.</p>
<p>Time per request: 56,772 ms (mean; lower is better)<br />
Requests per second: 17,61 (mean; higher is better)</p>
<h2>Approach 2: join with mysql_field_table()</h2>
<p>This is the approach promoted in my previous post. It performs a join query, iterates over the result set using mysql_field_table and then creates the instances. The total number of queries therefore is reduced to 1. This approach is less heavy on the database, but requires some more coding.</p>
<p>Time per request: 42,205 ms (mean; lower is better)<br />
Requests per second: 23,69 (mean; higher is better)</p>
<h2>Approach 3: join without mysql_field_table()</h2>
<p>This approach is somewhat like the second approach, but without the usage of mysql_field_table(). Instead of dynamically building the mapping of columns and table[fieldname], it uses a static array. This eliminates the usage of mysql_field_table(), but requires you to hard code all the mappings. This job is kinda cumbersome, and goes against the whole idea of lazy querying advocated in my previous post. Nevertheless, it gives a good comparison of the speed impact of the usage of mysql_field_table().</p>
<p>Time per  request: 41,351 ms (mean; lower is better)<br />
Requests per second: 24,18 (mean; higher is better)</p>
<h1>Comparison and conclusion</h1>
<p><img title="Time per request (ms; lower is better)" src="http://chart.apis.google.com/chart?chtt=Time+per+request+(ms)&amp;chts=000000,12&amp;chs=250x150&amp;chf=bg,s,ffffff|c,s,ffffff&amp;chxt=x,y&amp;chxl=0:||1:|0.00|28.38|56.77&amp;cht=bvg&amp;chd=t:100.00|74.34|72.83&amp;chdl=Approach+1|Approach+2|Approach+3&amp;chco=3366ff,009900,990000&amp;chbh=25" alt="" width="250" height="150" /> <img title="Requests per second (higher is better)" src="http://chart.apis.google.com/chart?chtt=Requests+per+second&amp;chts=000000,12&amp;chs=250x150&amp;chf=bg,s,ffffff|c,s,ffffff&amp;chxt=x,y&amp;chxl=0:||1:|0.00|12.09|24.18&amp;cht=bvg&amp;chd=t:100.00|97.97|72.82&amp;chdl=Approach+3|Approach+2|Approach+1&amp;chco=990000,009900,3366ff&amp;chbh=35" alt="" width="250" height="150" /></p>
<p>So what did I conclude from these data? First, it is obvious that <em>approach 1: no join</em>, is clearly the slowest approach. The difference might not seem that big, but imagine what would happen if the number of posts increased? My test case included 2 posts and 2 users, but what if 100 users post 100 messages? That would require a whopping 101 queries! Of course, you could do better by some clever programming... but still.</p>
<p>Second, approach 3 seems a bit faster then approach 2. This might seem logical as approach 2 iterates over all columns in the result set and requires 2 method calls (mysql_field_table and mysql_field_name). Approach 3 on the other hand does not, as the array is already given. Although statistic analysis is not my most favourite subject, one might argue whether the results of approach 2 and 3 really do differ. The outcomes are almost similar and I found a large deviation in both datasets.</p>
<h2>Conclusion</h2>
<p>Speedwise, your code can benefit from both approach 2 and 3. The code for splitting up the resultset should be refactored into some kind of a framework. Having refactored that code, your program remains easy to read and maintainable at the benefit of better performance!</p>

