---
title: 简单方法实现Hexo中的多语言
lang: zh-cn
date: 2016-08-30 20:22:38
tags:
categories: 
- hexo
---
# 摘要
我们知道,Hexo自带i18n的解决方案,但是那只是对于模板里一些连接,比如首页(home),标签(tags)等等. 如果想要让自己的文章可以分别显示2个版本就没有比较简单的方法了.
本文通过修改配置文件,post模板,layout布局文件以及几部手动复制粘贴完成对Hexo的整个多语言化.
<!--more-->

# 界面的多语言
## 修改`_config.yml`
把`language:`修改成
{% codeblock %}language:
- 默认语言
- 语言1
- 语言2
...{% endcodeblock %}
比如
{% codeblock %}
language:
- zh-cn
- en-us
{% endcodeblock %}
## 修改post
打开 <code>/_config.yml</code> 找到 <code>new_post_name</code>,把值修改成
{% codeblock %}
:lang/:title.md
{% endcodeblock %}
然后每次新建post时,在命令后写 <code>--lang 语言缩写</code> 语言缩写就和languages里的文件名相同即可. 这样post的地址就是<code>语言缩写/文章title</code>了. 这么做的目的主要是使得hexo生成时可以检测到地址里的语言缩写然后实行界面多语言转换.
*现在我们已经可以实现点开英文文章界面显示英文,点开中文文章界面显示中文了*

## 修改<code>/languages</code>下的语言文件
虽然界面大部分是英文了,但是对于简介等sidebar里的内容还是中文,这时候我们需要修改其他语言的yml文件.
*下面以Next主题为例,语言文件为<code>en-us</code>*
Next主题里的subtitle默认引用<code>\_config.yml</code>里的subtitle. 打开<code>\themes\next\layout\_partials\header.swig</code> 找到 
{% codeblock %}{% raw %} <p class="site-subtitle"> {% endraw %} {% endcodeblock %} 把后面大括号里的值修改成<code>__('sidebar.sub')</code>
然后在语言文件里的<code>sidebar</code>下添加<code>sub</code>,值填写对应语言下的subtitle就可以了. 网站名称等也可以用同样方法替换.
*现在界面里已经能完全显示多语言了,下面就是对<code>layout</code>修改实现只显示一种语言.*

## 添加英文的分类,标签
这个比较简单,在source下新建对应的语言缩写文件夹,把source下的`categories`和`tags`复制进去就可以了.
*归档和首页不一样,这个方法是不行的*

# 只显示同一种语言的文章
*这段我的处理方法很蠢, 如果有人有更好的方法烦请通过sidebar的联系方式告诉我, 谢谢!*
*我用Next主题为例,其他主题可能会不一样.*
## 修改文章列表
打开使用的<code>theme</code>里layout下的<code>index.swig</code>
找到<code>block content</code>下的一个for循环,在<code>post_template.render(post, true)</code>之前添加一个if语句
{% codeblock %} {% raw %} {% if post.lang === 'zh-cn' %} {% endraw %} {% endcodeblock %}
这样就是只显示`lang`是zh-cn的文章了.
我们先把这个改成en-us

为什么说这个方法很蠢呢, 因为不知道为什么除了生成的主页, 自己建的index都访问不到page.posts, 所以只能通过修改这个值,生成多个语言的index.html
QAQ 这也没办法啊! 有人会的话告诉我啊!

## 修改文章的上下篇
打开`\themes\next\layout\_macro\post.swig` 找到`{% raw %}&lt;div class="post-nav"&gt;{% endraw %}  大约215行 ` 改成
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
大家看一下应该就懂了,因为swig我不太会,好想没有while循环,就这么写了,如果是中英交替发的应该问题不是很大,后面有时间会把这段代码写的正常一点.
*现在界面看上去已经很好了,但是有一个很重要的问题就是没有英文的主页*

# 手动土又蠢添加首页!
## 修改menu链接
打开`\themes\next\_config.yml`在menu里把各个项的连接前都加上其他的语言缩写,并且在最后加一个换语言的按钮,比如我的:
```
  home: /en-us/ 
  categories: /en-us/categories
  about: /en-us/about
  archives: /en-us/archives
  tags: /en-us/tags
  lang: /
```

## 生成其他语言页面

在`_config.yml`里把`en-us`换成第一个语言, `hexo g`生成英语主要语言的页面.
把`/public/index.html`和`/public/archives`文件夹复制到`/source/en-us`下
并且在2个html文件最前面加上
{% codeblock %}{% raw %}
layout: false
---{% endraw %}{% endcodeblock %}
然后把`menu` `_config.yml`还有<code>theme</code>里layout下的<code>index.swig</code>设置改成原来的, 再次生成网站就可以了

