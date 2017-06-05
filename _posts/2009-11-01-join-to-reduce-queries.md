---
layout: post
title: Optimizing SELECT * FROM with mysql_field_table()
permalink: blog/2009/11/01/optimizing-select-from-with-mysql_field_table/
---

<p>Creating your own data access layer, can be quite some work. After some time, you will tackle the basics of querying the database and parsing the results into objects. Each query only retrieves a set of data from a single table, so you push the code to production like I did.</p>
<p>Then, after using the code for some time you want to be able to execute slightly more complex queries (like for example JOIN or sub queries). Then that piece of neat code, once running smoothly, started to degrade very fast.<br />

<span id="more-3"></span></p>
<h1>Database structure</h1>
<p>Our sample database contains two message entries written by two different authors.</p>
<h2>Table: user</h2>
<table>
<thead>
<tr>
<th>id</th>
<th>name</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>Bouke</td>
</tr>
<tr>
<td>2</td>
<td>Tim</td>
</tr>
</tbody>
</table>
<h2>Table: post</h2>
<table>
<thead>
<tr>
<th>id</th>
<th>userId</th>
<th>name</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>1</td>
<td>This is Bouke's message</td>
</tr>
<tr>
<td>2</td>
<td>2</td>
<td>This is Tim's message</td>
</tr>
</tbody>
</table>
<h1>Objects</h1>
<p>So, let's talk some code. The system contains posts written by some user. The objective is to list all the messages and the user who posted the message. The following objects are given.</p>
{% highlight php %}
class Post
{
    public $id;
    public $userId;
    public $message;

    protected $_user;

    public function setUser(User $user)
    {
        $this-&gt;_user = $user;
    }

    public function getUser()
    {
        if(!isset($this-&gt;_user)) {
            $result = mysql_query("SELECT * FROM `user`
               WHERE `id` = {$this-&gt;userId}");
            $this-&gt;setUser(mysql_fetch_object($result, "User"));
        }

        return $this-&gt;_user;
    }
}

class User
{
    public $id;
    public $name;
}
{% endhighlight %}
<h1>Initially: 3 queries</h1>
<p>This one is simple and straight forward. First, you fetch all the posts. Then for each post, you fetch the person who posted it. Note, there is initially a single query for requesting all the posts, but each post results in a query for the user.</p>
{% highlight php %}
$result = mysql_query("SELECT * FROM `post`");

while($post = mysql_fetch_object($result, "Post")) {
   echo "{$post-&gt;getUser()-&gt;name}: {$post-&gt;message}", "&lt;br/&gt;\n";
}
{% endhighlight %}
<h1>Improvement with a JOIN</h1>
<p>The solution seems simple, using a JOIN only performs a single query:</p>
{% highlight php %}
SELECT * FROM `post` JOIN `user` ON (`post`.`userId` = `user`.`id`)
{% endhighlight %}
<p>But this has also its affects on the code issuing this query. The code needs to build two objects from each row it receives. Using a simple associative array will not help you here, as both tables have a column called `id`. </p>
<p>Aha, you might think, use an alias and case solved! But what if you are a bit lazy, like me. You would not want to specify all the columns with their aliasses, but still use SELECT *. This is where mysql_field_table() appears on the stage:</p>
<blockquote><p>string mysql_field_table ( resource $result , int $field_offset )<br />
Returns the name of the table that the specified field is in.</p>
<p>From: <a href="http://www.php.net/mysql_field_table">PHP Manual</a></p></blockquote>
<p>So, first we query the database using a join:</p>
{% highlight php %}
$result = mysql_query("SELECT * FROM `post`
   JOIN `user` ON (`post`.`userId` = `user`.`id`)");
{% endhighlight %}
<p>Then, we build the mapping of column index => {tablename, fieldname}</p>
{% highlight php %}
$columns = array();
for($i = 0; $i &lt; mysql_num_fields($result); $i++) {
    $columns[] = array(
        mysql_field_table($result, $i),
        mysql_field_name($result, $i));
}

// Result:
// [["post","id"],
//  ["post","userId"],
//  ["post","message"],
//  ["user","id"],
//  ["user","name"]]
{% endhighlight %}
<p>Now we know which column relates to which tablename and fieldname, we iterate over all rows and create a two-dimensional array:</p>
{% highlight php %}
$rows = array();
while($row = mysql_fetch_row($result)) {
    foreach($row as $i =&gt; $value) {
        $row[$columns[$i][0]][$columns[$i][1]] = $value;
        unset($row[$i]);
    }
    $rows[] = $row;
}

// Result:
// [{   "post":{"id":"1","userId":"1","message":"This is Bouke's message"},
//  "user":{"id":"1","name":"Bouke"}},
//  {   "post":{"id":"2","userId":"2","message":"This is Tim's message"},
//  "user":{"id":"2","name":"Tim"}}]
{% endhighlight %}
<p>Then the last step concerns transforming the two-dimensional array into object:</p>
{% highlight php %}
class Factory
{
    public static function build($object, array $data)
    {
        foreach($data as $property =&gt; $value) {
            $object-&gt;$property = $value;
        }
        return $object;
    }
}

foreach($rows as $row) {
    $post = Factory::build(new Post(), $row["post"]);
    $post-&gt;setUser(Factory::build(new User(), $row["user"]));

    echo "{$post-&gt;getUser()-&gt;name}: {$post-&gt;message}", "&lt;br/&gt;\n";
}
{% endhighlight %}
<p>And you're done; all down to a single query! You might wander, why would you rewrite four lines of code to a whoppin' 32 lines? Only for the benefit of using lazy SELECT * querying?</p>
<p>Well, those 32 lines decompose into 25 lines for the framework and only a stunning 7 lines for the fetching and object building. Is the method worth rewriting your code?</p>
<p>The method explained has not yet been taken into production, as I first want to perform an speed impact vs ease of code analysis. For now, the code provided is only a proof of concept.</p>

