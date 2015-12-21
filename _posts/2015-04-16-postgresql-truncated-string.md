---
layout: post
title: "Postgresql Truncated String"
description: "使用postgresql遇到的问题"
category: Programming
tags: []
---
最近手上有个活，要向`postgresql`数据库中导入一些数据，不是一个非常大的活，就用写个`shell`脚本，生成一个包含所有数据插入语句的`sql`文件。格式大概是这样

{% highlight sql %}
insert into Table("col1", "col2", "col3") values
('xxxxx','Beijing','en_US'),
('xxxxx','Tianjin','en_US'),
...
{% endhighlight %}
不是很麻烦，但是用`psql`执行`sql`文件时，出错了
{% highlight bash %}
psql:sql/lang.sql:1345: 错误:  语法错误 在 "an" 或附近的
LINE 157: ,('000000000000000000000155','Xi'an','en_US')
{% endhighlight %}

好吧，考虑不周全。改下程序使最后的`sql`像这样
{% highlight sql %}
('000000000000000000000155','Xi\'an','en_US')
{% endhighlight %}

怎奈`shell`技能掌握不全，折腾一会没有实现。那我改成这样

{% highlight sql %}
insert into Table("col1", "col2", "col3") values
("000000000000000000000155","Xi'an","en_US")
...
{% endhighlight %}

结果新的问题来了，

{% highlight bash %}
psql:sql/lang.sql:1345: 注意:  标识符"The Viewers that did not watch any game webpages from 5.10 to 6.13"将会被截断为"The Viewers that did not watch any game webpages from 5.10 to 6"
psql:sql/lang.sql:1345: 注意:  标识符"The Viewers that watched any baby care & hygiene webpages more than twice in past 6 months with lifecycle which more than three months"将会被截断为"The Viewers that watched any baby care & hygiene webpages more "
psql:sql/lang.sql:1345: 注意:  标识符"The Viewers that did not watch any baby care & hygiene webpages more than twice in past 6 months and lifecycle which less than three months"将会被截断为"The Viewers that did not watch any baby care & hygiene webpages"
{% endhighlight %}


`postgresql`居然要将我的数据截断，真是惊呆了。但是提示`标识符`，看来`postgresql`并没有后面的字符串当做普通数据来对待。Google了一下后，`""`是`postgresql`的保留字符[[1]]，用来标识Table，Column的。不知道你有没有注意，过长的字符串都会被截断为前64个字符，这个也是`postgresql`其内部对象名称的最大长度[[2]].


#### Update:

{% highlight sql %}
('000000000000000000000155','Xi\'an','en_US')
{% endhighlight %}

即使将`sql`改成这样也是执行不了的，标准的做法[[3]]是

{% highlight sql %}
('000000000000000000000155','Xi''an','en_US')
{% endhighlight %}

[1]: http://www.postgresql.org/message-id/1059511597.10852.4.camel@eric
[2]: http://www.postgresql.org/docs/9.1/static/datatype-character.html
[3]: http://stackoverflow.com/questions/12316953/insert-varchar-with-single-quotes-in-postgresql
