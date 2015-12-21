---
layout: post
title: 用Jekyll搭建博客
date: 2015-05-18 17:55:54
categories: [jekyll,others]
---
最近使用[`Jekyll`](http://jekyllrb.com/)搭建了一个博客，发现`Jekyll`用来搭建个人站点十分方便，推荐大家使用下。

### 安装`Jekyll`

`Jekyll`的安装相当方便，使用`gem`安装即可，当然前提是你已安装过`ruby`

{% highlight bash %}
[root@56fcb293cc4d /]$ yum install ruby
[root@56fcb293cc4d /]$ gem install jekyll
{% endhighlight %}

### 运行

安装完成后也不需要做太多的操作，使用`jekyll new [name]`命令创建你的站点所需要的全部文件。在刚刚创建的目录下运行`jekyll serve`，新创建的站点就可以访问了，`http://127.0.0.1:4000`，过程非常简单和方便。

{% highlight bash %}
[root@56fcb293cc4d /]$ jekyll new my-awesome-site
[root@56fcb293cc4d /my-awesome-site]$ jekyll serve
{% endhighlight %}


### 目录结构

使用`tree`可以查看文件下的目录结构

{% highlight bash %}
[root@56fcb293cc4d /my-awesome-site]$ tree
.
├── _config.yml
├── _includes
│   ├── footer.html
│   ├── head.html
│   └── header.html
├── _layouts
│   ├── default.html
│   ├── page.html
│   └── post.html
├── _posts
│   └── 2015-05-18-welcome-to-jekyll.markdown
├── _sass
│   ├── _base.scss
│   ├── _layout.scss
│   └── _syntax-highlighting.scss
├── about.md
├── css
│   └── main.scss
├── feed.xml
└── index.html

5 directories, 15 files
{% endhighlight %}

每个目录的作用  
`_config.yml` : 站点相关的配置信息，如`title，markdown editor, source`等  
`_includes` : 放置网页`header, footer`的地方  
`_layouts` : 页面模板  
`_post` : 你写的文章，使用`markdown`编写。命名遵循`YEAR-MONTH-DAY-title.markdown`，便于`Jekyll`识别和为文章生成永久链接`(permalink)`  
`_sass`，`css` : 存放你的`sass`  
`_site` : 这里存放就是你的站点的所有信息。`Jekyll`会将你写的文章全部生成静态的`html`文件，整个站点全部是静态文件，`Jekyll`做的只是根据你的配置，监测你的修改，一有变化就生成新的静态文件。

### 发布新文章

修改`_config.yml`里的`title`，`username`修改为你希望的名称。  
发布新文章，只需要在`_post`文件夹里创建新的文件，按照上面提到的命名格式命名文件，使用`markdown`编写你的文章，并确保在文件头部插入如下内容

{% highlight bash %}
---
layout: post
title: "My Post"
date: 2015-05-18 17:00:12
categories: [jekyll,mood]
---
{% endhighlight %}

这段内容成为[`YAML Front Matter`](http://jekyllrb.com/docs/frontmatter/)，`Jekyll`在处理这个文件时，会把它当成特殊文件处理，如果没有这段内容，`Jekyll`只会认为这个文件是一个静态文件，不会进行我们希望的`markdown2html`。  
`layout`指明这篇文章使用哪个模板，模板文件定义在`_layouts`文件夹下。  
`categories`表示这篇文章属于哪个类别，`Jekyll`会根据文章的类别进行归类，便于你浏览。其实`Jekyll`会为每个`category`创建一个文件夹，然后将生成的`html`文件放到对应的`category`下.
创建完新文章后，稍等几秒，刷新下你的站点，新文章就出现在页面上。

### 使用主题

我不太喜欢`Jekyll`默认的网站风格，虽然很简洁，但是不是非常好看。[`JekyllThemes`](http://jekyllthemes.org/)上有许多好看的`Jekyll`主题，而且都已经放在`GitHub`上，还有`Demo`。你可以将对应主题的`repo clone`下来，然后稍微修改里面的信息，一个全新的站点就有了。 [`Dribble`](https://dribbble.com/search?q=jekyll)上也有许多设计师分享的`Jekyll` 主题设计，很多是没有公开代码的，不过还是值得一看。  还有种做法就是创建自定义的模板，放到`_layouts`下，在创建文章的时候指明模板就行了。

### 发布到`GitHub`

目前都是在本地搭建的网站，在`GitHub上`创建一个`[username].github.com`的`repository`，将你的站点推送到这个`repo`下，稍等片刻你就可以在`http://[username].github.io`下访问你的站点了  
还有一种更快速的做法，[`JekyllBootstrap`](http://jekyllbootstrap.com/)，你只需提供你的`Github`用户名，几分钟就可以在`Github`创建好站点。
