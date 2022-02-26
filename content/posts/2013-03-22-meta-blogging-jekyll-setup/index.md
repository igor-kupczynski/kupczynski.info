---
title: Blogging Like a Hacker
tags: []
aliases:
- /2013/03/22/meta-blogging-jekyll-setup.html
---
A description of my blogging setup.
I'm Igor ... wait. To learn more about me or my motivation go and read
the [*about me*][about] page. The very first post will be *meta*. It
describes and praise my blogging setup. It should give you a solid
idea which tools I use and how to configure a similar stack. I try to
list its pros and cons and share my personal opinion on the various
quirks. Hope you will like it!

## Setup

The keyword to my setup is [Jekyll][jekyll].

Jekyll is a *blog aware* static site generator. I use it to generate
html pages from the [markdown][md] documents. It allows specifying a
common layout and then focus only on the content. It takes your
markdown posts, converts them to html and injects the content into the
layout. It also takes care of generating a structure of folders, so
your urls can be nice and blogish,
e.g. `/2013/03/01/my-new-framework-rocks.html`.

From this point on it is straightforward - when Jekyll has done its
job you've got a directory containing a set of html files - a full
website which can be served from any static webserver like apache. I
push the content to my server from where [Nginx][nginx] serves it.

### Local workstation

#### Html generation
Jekyll is a small piece of code written in ruby. It's straightforward
to install it on ubuntu.

{% highlight bash %}
$ sudo apt-get install ruby1.9.1-dev  # only if you do not have ruby
$ sudo gem install jekyll
{% endhighlight %}

I'm not a ruby programmer, so this may be a *con* from my
perspective. On the other hand, you do not need to dive into the code
to use Jekyll.

To use Jekyll in your project you need to prepare a directory
structure:

	.
	├── _includes
	├── _layouts
	├── _plugins
	├── _posts
	├── _site
	├── _config.yml
	└── index.html

Jekyll browsers your directories and parses any file starting with a
[yaml front matter][jekyll-yfm], for example:

{% highlight yaml %} 
---
title: Lolcatz!
tagline: We teach you how make funny cat pictures
---
(... file content ommitted ...)
{% endhighlight %}

`_config.yml` is a configuration file for Jekyll. Your posts go to
`_posts` directory. A base template stays in the `_layouts` and you
can put reusable chucks of html in `_includes`. Jekyll's behavior can
be customized with a set of plugins. They go to the `_plugins`
directory.

Firstly, Jekyll converts markdown/textile files into html. Htmls are
considered templates and run through a [liquid][liquid] processor. It
allows you to use constructs like loops, conditions and filters in
your pages. For example, this can be used to generate a list of all
posts for your archive.

{% highlight django %}
{% raw %}
{% for post in site.posts %}
  <h3>{{ post.title }}</h3>, {{ post.date }}
  <p>{{ post.excerpt }}</p>
  <p><a href="{{ post.url }}">Read more</a></p>
{% endfor %}
{% endraw %}
{% endhighlight %}

Generated html is copied to `_site` directory. An example from my
blog:

	_site/
	├── 2013
	│   └── 03
	│       └── 22
	│           └── meta-blogging-jekyll-setup.html
	├── about.html
	├── index.html
	├── rss.xml
	└── static
	    ├── bootstrap
	    │   ├── css
	    │   │   └── (...)
	    │   ├── img
	    │   │   └── (...)
	    │   └── js
	    │       └── (...)
	    ├── css
	    │   └── geekigor.css
	    └── img
		├── linkedin-icon.png
		├── mail-icon.png
		├── rss-feed-icon.png
		└── twitter-icon.png

#### Markdown

[Markdown][md] is a *lightweight markup language*. It is a plain text,
easy to read and easy to write format, which can be converted to valid
(X)HTML. The format is quite popular, most notably in sites like
[stack overflow][so] and [github][gh]. An example of its syntax:

{% highlight rest %}
This is a title
===============

## And this is a header

**Strong** paragraphs are good. But you know what is better?

Lists. Lists are better:

- Because the have multiple items,
- And they are not boring.

## Another header 

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do
eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
minim veniam, quis nostrud exercitation ullamco laboris nisi ut
aliquip ex ea commodo consequat. Duis aute irure dolor in
reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla
pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
culpa qui officia deserunt mollit anim id est laborum.

{% endhighlight %}

With markdown you can use *any text editor* to create the posts. No
need to run a web browser and type your thought into one of these
small, ugly text areas. As matter of fact, for me *any text editor* is
emacs.

#### Version control

I'm a big fan of [Git][git]. I use it wherever possible. By the way, I
highly recommend to familiarize with the
[Vincent Driessen's git workflow][git-wf].

It seems natural to version control the blog as well. Since markdown
is a plain-text format it is a perfect fit for git or any other VCS.

I turns out that [a lot of blogs][jekyll-blogs] are stored on
github. Just go there to find some inspiration or see how people use
Jekyll.


### Server side

Jekyll leaves you with a static web page, which is great. No need for
a database or an application server. All you need is a plain old
webserver like apache or nginx.

The usage of static html has many benefits:
* it scales,
* it is easy to deploy,
* hosting is cheap,
* no moving parts that hackers can get to.

Jekyll has its drawbacks though, most notably no dynamic server side
content. You end up with a bunch of files which may or may not be what
you want. You can add some spice using javascript, but having no
option for dynamism means Jekyll is useful only for simple use
cases. Luckily, we are building a blog. It's simple.

#### Github pages

You can host your blog on github. [Github pages][gh-pages] is a free
and easy to use solution to host a static content. It can be connected
to your domain. It's not as flexible as a web server, yet it is
powerful enough for many scenarios.

## Customizing Jekyll

Out-of-a-box Jekyll is really a plain vanilla system and you need to
do some work to get the features which are usually taken for granted
on modern web pages.

Please note there are projects like [Octopres][oct] which aim to
deliver a preconfigured and themed Jekyll - so you can start right
away.  Even if you don't want to use their solution it's worth to go
through its source code for idea to organize and customize your setup.

### RSS

To publish an rss feed means essentially to prepare a template (we can
call it `rss.xml`) which will take your latest posts and generate a
file compliant with the [RSS specification][rss-spec].

I started from this [rss.xml][rss-base] file and ended up with
something like that:

{% highlight xml %}
{% raw %}
---
layout: nil
---
<?xml version="1.0"?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>Geek Igor</title>
    <link>{{ site.url }}</link>
    <atom:link href="{{ site.url }}/rss.xml" rel="self"
	       type="application/rss+xml" />
    <description>A blog on software development by Igor Kupczyński</description>
    <language>en-us</language>
    <pubDate>{{ site.time | date: "%a, %d %b %Y %H:%M:%S %z" }}</pubDate>
    <lastBuildDate>{{ site.time | date: "%a, %d %b %Y %H:%M:%S %z" }}
		</lastBuildDate>

    {% for post in site.posts limit:5 %}
    <item>
      <title>{{ post.title }}</title>
      <link>{{ site.url }}{{ post.url }}</link>
      <pubDate>{{ post.date | date: "%a, %d %b %Y %H:%M:%S %z" }}</pubDate>
      <author>{{ site.author.name }} {{ site.author.email }} </author>
      <guid>{{ site.url }}{{ post.id }}</guid>
      <description>
	  &lt;p&gt;Tags: {{ post.tags | join ', ' }}&lt;/p&gt;
	  {{ post.excerpt | xml_escape }}
	  &lt;/p&gt;| &lt;a href=&quot;{{ site.url }}{{ post.url }}&quot;&gt;
	  Read more&lt;/a&gt;&lt;/p&gt;
      </description>
    </item>
    {% endfor %}

  </channel>
</rss>
{% endraw %}
{% endhighlight %}

It generates a header for the feed, then iterates through the latest
five posts and puts them in the feed.

You should also attach the feed address to `head` section of your
pages. Just like this.

{% highlight html %}
<link rel="alternate" type="application/rss+xml" title="Latest posts"
	href="/rss.xml">
{% endhighlight %}

It will result in an icon which indicates that a feed is available.
  
### Excerpts

Jekyll uses a [plugin mechanism][jekyll-plug] to let the world extend
its behavior. To install a plugin just drop a ruby file into the
`_plugins` directory.

Since I do not do ruby I need to use plugins released by the
community. A following scenario may be a good example of how plugins
are useful.

You need to be able to extract a lead of an article. Be it the first
paragraph that you want to include on your homepage and in your rss
feed. Nobody will click on a title alone, so we need to let our
visitors peek into an article - to catch their interest.

Such a future is not available in a latest version of Jekyll in ubuntu
repository (`Jekyll 0.12.1`). Hence we need to find a plugin. I use
this one: [excerpt.rb][ex-rb]. It simply exposes `excerpt` as an
attribute of a post.

{% highlight django %}
{% raw %}
{% for post in site.posts %}
	<p>{{ post.excerpt }}</p>
{% endfor %}
{% endraw %}
{% endhighlight %}
	
  
### Pagination

A lot of blogs on Jekyll does not use pagination at all! People tend
to have a flat list of their articles, containing only titles. This
approach for sure lets you compress a lot of posts on your homepage,
but it just does not feel right for me. I wanted to have more blogish
approach. Namely, a few posts with their excerpts on the home page and
a pagination to access older posts.

Luckily there is a built-in solution. It's described on the
[Jekyll wiki][jekyll-pagination]. You need to set up how many posts
per page should be present. This is done in your `_config.yml` file.

	paginate: 5
	
And then you add some liquid syntax on the home page. Here's an example
from this blog:

{% highlight django %}
{% raw %}
<div class="pagination pagination-large pagination-centered">
  <ul>
    {% if paginator.previous_page %}
        <li>
            {% if paginator.previous_page == 1 %}
                <a href="/">&laquo;</a>
            {% else %}
                <a href="/page{{paginator.previous_page}}">&laquo;</a>
            {% endif %}
        </li>
    {% else %}
        <li class="disabled"><a>&laquo;</a></li>
    {% endif %}
    <li>
        {% if paginator.page == 1 %}
		    <span>1</span>
        {% else %}
		    <a href="/">1</a>
        {% endif %}
    </li>
    {% for count in (2..paginator.total_pages) %}
        <li>
            {% if count == paginator.page %}
                <span>{{count}}</span>
            {% else %}
                <a href="/page{{count}}">{{count}}</a>
            {% endif %}
        </li>
    {% endfor %}
    {% if paginator.next_page %}
        <li><a href="/page{{paginator.next_page}}">&raquo;</a></li>
    {% else %}
        <li class="disabled"><a>&raquo;</a></li>
    {% endif %}
  </ul>
</div>
{% endraw %}
{% endhighlight %}

I've put it in the `_include` directory as `pagination.html` and set
it up on home page like this.

{% highlight django %}
{% raw %}
{% include pagination.html %}
{% endraw %}
{% endhighlight %}

## Theming

That's the hard part. At least it used to be. A clean, browser
compatible and good looking design is hard to achieve. Especially
that, nowadays, you're expected to cover mobile devices as well- your
design needs to be *responsive*.

Luckily, there is a hope for us. It's called [Bootstrap][[bootstrap]],
it's from twitter and it's hot. Believe it or not, but hundreds of
pages nowadays base their layout on bootstrap.

Bootstrap calls itself a front-end framework. It provides a layout,
i.e., a responsive grid, a lot of components, and a ready-made design
in a form of cascade style stylesheets.  If you don't like its default
look, or just want to stand out from the crowd, there are good-quality
themes, both free and paid, available at
[bootswatch][bootswatch]. Bootstrap csses are very clean and therefore
it is easy to [customize it][bootstrap-customize] if you so wish.

It is worth noting that there are some alternatives, like
[zurb foundation][zurb] or [yaml4][yaml4]. I'm not a css geek so I
decided to use *Bootstrap* as [other cool kids][build-with-bootstrap]
do.

## Future

In a truly agile sprint I decided to prioritize my features and
deliver a working software with only a subset of them. As a part of
the first iteration of course.

Couple of items are missing and I hope to add them in near future. On
my short list are currently:

### Figure tag

Markdown is great, but inserting images with it may be tedious. It's
better to use a liquid tag, like [this][oct-image-tag] or
[this][img-gist]. I want it also to support new
[html5 figure syntax][html5-figure].

Another open question is how to handle images. I keep the source code
for this blog in a repository, but I do not want to bloat it with
images accompanying posts. There are some options here to evaluate.

### Asset pipeline and build process 

The csses attached to this page are not optimized or mimified in any
way. It seems a waste not to optimize them. I do not want to do it
manually, therefore I need an automated process to do it for me. This
process will be also useful for building and pushing the blog content
to my remote server. Automation is very high on my priority list.

### Random pages

My current version of "About Me" sucks. I need to work on that. Also,
there is a lot of nice examples of 404 pages out there. Need one for
this weblog.

### Social items
	
I have a vague plan to attract a lot of traffic to this weblog using
[my twitter account][twitter-puszczyk].

Apart from that, I plan to include comments on the site.

### Semantic search

Sounds nice, but it means simply to have a page archiving all posts
under a given tag and to link to similar posts in an automated manner.

### Org mode

Emacs rocks. Period. And it's great for markdown.

However I heavily rely on [org-mode][org-mode] as a note-taking
tool. Org-mode has a great html exporter. There should be a way to
combine it with Jekyll and write my posts with the org syntax.

<figure>
	<img src="https://orgmode.org/img/main.jpg" alt="Org mode FTW">
	<figcaption>Org mode is for keeping notes, maintaining TODO lists, planning projects, and authoring content.</figcaption>
</figure>


## What's next

Time to do some real work. I hope more posts will follow, and
moreover, I hope they will be useful.


[about]: /about.html About Me
[bootstrap]: http://twitter.github.io/bootstrap/
[bootstrap-customize]: http://coding.smashingmagazine.com/2013/03/12/customizing-bootstrap/
[bootswatch]: http://bootswatch.com/
[build-with-bootstrap]: http://builtwithbootstrap.com/
[ex-rb]: https://github.com/ixti/ixti.github.com/blob/source/_plugins/excerpt.rb
[gh]: https://github.com/
[gh-pages]: http://pages.github.com/
[git]: http://git-scm.com/
[git-wf]: http://nvie.com/posts/a-successful-git-branching-model/ A successful Git branching model
[html5-figure]: http://html5doctor.com/the-figure-figcaption-elements/
[img-gist]: https://gist.github.com/pichfl/1548864
[jekyll]: http://jekyllrb.com/
[jekyll-blogs]: https://github.com/mojombo/jekyll/wiki/Sites Sites using jekyll.
[jekyll-plug]: https://github.com/mojombo/jekyll/wiki/Plugins
[jekyll-pagination]: https://github.com/mojombo/jekyll/wiki/Pagination
[jekyll-yfm]: https://github.com/mojombo/jekyll/wiki/YAML-Front-Matter
[liquid]: https://github.com/Shopify/liquid/wiki
[md]: http://daringfireball.net/projects/markdown/ Text to html conversion tool for web writers
[nginx]: http://wiki.nginx.org/Main
[rss-base]: https://github.com/coyled/coyled.com/blob/master/rss.xml
[rss-spec]: http://www.rssboard.org/rss-specification
[so]: http://stackoverflow.com/
[oct]: http://octopress.org/
[oct-image-tag]: https://github.com/imathis/octopress/blob/master/plugins/image_tag.rb
[org-mode]: http://orgmode.org/
[twitter-puszczyk]: https://twitter.com/puszczyk
[yaml4]: http://www.yaml.de/
[zurb]: http://foundation.zurb.com/
