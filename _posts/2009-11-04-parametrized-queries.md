---
layout: post
title: Preventing SQL Injection with Parametrized Queries
permalink: blog/2009/11/04/preventing-sql-injection-with-parametrized-queries/
---

<p>Still using magic quotes, or using addslashes to add slashes all $_POST and $_GET variables to prevent SQL Injection? Using magic quotes is considered <a href="http://arpad.co.uk/2008/09/the-adventure-of-php-and-the-magic-quotes/">not</a> <a href="http://www.sitepoint.com/blogs/2005/03/02/magic-quotes-headaches/">a</a> <a href="http://mordred.niama.net/blog/?p=122">good</a> <a href="http://nedbatchelder.com/blog/200310/php_and_magic_quotes.html">idea</a>, and using addslashes will also certainly drive you insane.</p>
<h1>Parameterized statements</h1>
<blockquote><p>To protect against SQL injection, user input must not directly be embedded in SQL statements. Instead, parameterized statements must be used (preferred), or user input must be carefully escaped or filtered. With most development platforms, parameterized statements can be used that work with parameters (sometimes called placeholders or bind variables) instead of embedding user input in the statement. In many cases, the SQL statement is fixed. The user input is then assigned (bound) to a parameter.<br />
From: <a href="http://en.wikipedia.org/wiki/SQL_injection#Preventing_SQL_injection">Wikipedia</a></p></blockquote>
<p><span id="more-34"></span></p>
<h1>MySQLi / PDO-only!?</h1>
<p>PHP has built-in support for parameterized statements using the MySQL Improved Extension (MySQLi) or PHP Data Objects. So no build-in support if you're stuck with the old MySQL extension. As I was using the old MySQL extension in most of my projects and I really wanted to use parameterized statements, I found a way to implement them myself.</p>
<p>As placeholders I used the question mark (?), as MySQLi also uses the same syntax. Say I want to fetch a user from the database by its field `id`, one could use the following code:</p>
<pre class="brush: php;">$result = mysql_query("SELECT * FROM `user` WHERE `id` = {$_GET['id']}");</pre>
<p>As obvious to most readers, this makes you vulnerable to SQL Injection as $_GET['id'] is used without any escaping or checking. Using a parametrized query, the same query would read:</p>
<pre class="brush: sql;">SELECT * FROM `user` WHERE `id` = ?</pre>
<p>I implemented the function 'parametrized_query' which takes a query and multiple substitution values. The query above is executed in the code like this:</p>
<pre class="brush: php;">$result = parametrized_query("SELECT * FROM `user` WHERE `id` = ?",
   $_GET['id']);</pre>
<h1>sprintf and mysql_real_escape_string</h1>
<p>How does parametrized_query work? First, I escape all the string values send to the function using <a href="http://php.net/mysql_real_escape_string">mysql_real_escape_string</a> and put them between a single apostrophe: 'string'. This makes strings safe from SQL injection.</p>
<p>Then, using <a href="http://php.net/sprintf">sprintf</a>, the parameters are injected in the query. As sprintf uses %s instead of ? for placeholders, I use <a href="http://php.net/str_replace">str_replace</a> to substitute ? for %s.</p>
<p>The constructed query then looks like this (remember, $_GET only returns strings):</p>
<pre class="brush: sql;">SELECT * FROM `user` WHERE `id` = '1'</pre>
<p>I could force the parameter into an integer by first casting $_GET['id'] into an int:</p>
<pre class="brush: php;">$result = parametrized_query("SELECT * FROM `user` WHERE `id` = ?", (int) $_GET['id']);</pre>
<p>The query then looks like this:</p>
<pre class="brush: sql;">SELECT * FROM `user` WHERE `id` = 1</pre>
<h1>The code</h1>
<p>Combining everything together, you end up with the following:</p>
{% highlight php %}
function parametrized_query()
{
    $query = func_get_args();
   $query[0] = str_replace("?", "%s", $query[0]);

    for($i = 1; $i &lt; count($query); $i++) {
        if(is_string($query[$i])) {
            $query[$i] =
               "'".mysql_real_escape_string($query[$i])."'";
        } else if(is_null($query[$i])) {
            $query[$i] = 'NULL';
        }
    }

    $sql = call_user_func_array("sprintf", $query);

    return mysql_query($sql);
}
{% endhighlight %}
