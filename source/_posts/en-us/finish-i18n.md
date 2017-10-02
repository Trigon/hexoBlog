---
title: A simple way realize multi-language in Hexo
lang: en-us
date: 2016-08-30 20:24:08
tags:
categories:
- hexo
---
# Summary
We know, Hexo has i18n function, however it just works for the links in the layout, such as home tags and so on.
There isn't a simple way to show the same post in different languages.
This passage show how to make post be i18n by changing config, post, layout and copying&pasting.
<!--more-->

# I18N in Interface
## Change `_config.yml`
Change `language: ` to
{% codeblock %}
language:
- First lang
- Other lang1
- Other lang2
...
{% endcodeblock %}
such as
{% codeblock %}{% raw %}
language:
- en-us
- zh-cn
{% endraw %}{% endcodeblock %}
## Change post
Open `/_config.yml` and find
`new_post_name`. Change it to
`:lang/:title.md`
Then each time when we creat a new post, endclose `--lang` which lang is en-us, zh-cn and so on.
After this step, the address of the post should be `lang/title.md`
So every post will be created under the languages folder. Then hexo will detect the language in the url and change the language in the website.
*Now when we click posts in different language, we will see the website with the same language as post.*

## Change the languages files under `/languages`
Althought most of pages are other languages, but the subtitle is also the same as orginal. So we need to change language files.
*I will take theme `NexT` as example, the lang file is `zh-cn`*
The subtitle in NexT comes from the `subtitle` in `\_config.yml`. Open`\theme\next\layout\_partials\header.swig` and find the line 
{% codeblock %}{% raw %}<p class="site-subtitle"> {% endraw %} {% endcodeblock %}
Change the thing in brace into `__('sidebar.sub')`. Then add `sub` into `sidebar` of the lang file. The value is the translation of your subtitle. The name of the site can be also replaced by this method.
*Now the layout of website can totally show another language.*

## Add the website of tags and categories of other languages
This step is simpler. Create the folder with the name which is same as language. Then copy `categories` and `tags` from source to the new folder.
*It doesn't work for archives and homepage.*


Copy `/public/index.html` and `/public/archives` to `/source/(lang)`. Also add
{% codeblock %}{% raw %}
layout: false
---{% endraw %}{% endcodeblock %}
above two html files we just copied.
Finally, change `menu` `_config.yml`and <code>index.swig</code> under `layout` in 'theme' back to the original language and 'hexo g' again.

# Only show the posts in certain language
*This method is stupid. If anyone has a better method, please let me know*
*I use theme `NexT` as a example*
## Change posts list
Open `index.swig` under `layout` of `theme`. Find a for under `block content`, add phrase above `post_template.render(post,true)`
{% codeblock %} {% raw %} {% if post.lang === 'zh-cn' %} {% endraw %} {% endcodeblock %}
Then the list on the homepage will only show the posts whose lang is Chinese. To show English posts, change it to `en-us`.

So why I said this is a stupid method? I don't know why noone can reach page.posts except the homepage. So I just use this way to generate some homepages in different languages.
TAT so sad!

## Change prev and next of post
Open `\themes\next\layout\_macro\post.swig` and find `{% raw %}&lt;div class="post-nav"&gt;{% endraw %} (At about line 215 )`.
Change it into
{% codeblock post.swig lang:HTML %}
{% raw %}
        <div class="post-nav">
          <div class="post-nav-next post-nav-item">
			{% if post.next and post.next.lang === post.lang %}
			<a href="{{ url_for(post.next.path) }}" rel="next" title="{{ post.next.title }}">
                <i class="fa fa-chevron-left"></i> {{ post.next.title }}
              </a>
			  <p> {{post.lang}}</p>
			{% else %}
				{% set next_post = post.next %}
				{% if next_post.next and next_post.next.lang === post.lang  %}
				<a href="{{ url_for(next_post.next.path) }}" rel="next" title="{{ post.next.title }}">
					<i class="fa fa-chevron-left"></i> {{ next_post.next.title }}
				</a>
				{% endif %}
			{% endif %}
          </div>

          <div class="post-nav-prev post-nav-item">
            {% if post.prev and post.prev.lang === post.lang %}
              <a href="{{ url_for(post.prev.path) }}" rel="prev" title="{{ post.prev.title }}">
                {{post.prev.title}} <i class="fa fa-chevron-right"></i>
              </a>
			{% else %}
				{% set prev_post = post.prev %}
				{% if prev_post.prev and prev_post.prev.lang === post.lang %}
					<a href="{{ url_for(prev_post.prev.path) }}" rel="prev" title="{{ post.prev.title }}">
					{{prev_post.prev.title}} <i class="fa fa-chevron-right"></i>
					</a>	
					{% endif %}
            {% endif %}
          </div>
        </div>
{% endraw %}
{% endcodeblock %}
You just need to have a glance at codes, you will find how it works. I am not familiar with language `SWIG`. So I don't know how to write for loop in that language.

*now the pages looks very nice, the only problem is that we do not have a homepage in other language*

# Add the homepage
## Change links on menu
Open `\themes\next\_config.yml`, add the language name before each links of menu and add a item to change the language. My code is below:
```
  home: /zh-cn/ 
  categories: /zh-cn/categories
  about: /zh-cn/about
  archives: /zh-cn/archives
  tags: /zh-cn/tags
  lang: /
```

## Generate other language pages
Set the first choice language into other in `_config.yml` then `hexo g` to generate the blog.