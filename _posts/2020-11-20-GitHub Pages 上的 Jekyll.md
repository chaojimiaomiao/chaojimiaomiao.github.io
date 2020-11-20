---
layout: post
title:  "GitHub Pages 上的 Jekyll "
date:   2020-11-19 20:04:22 +0800
categories: jekyll update
---


GitHub 与 Gitee 提供的 Pages 服务中，均内嵌了 Jekyll 支持（Gitee 还提供了 Hugo 与 Hexo 支持）。所谓「支持」，即指这些生成工具挂在云端；你只需要提供原始代码（如 Markdown 文档、Sass/Stylus/Less 样式表），再由 Pages 服务自动编译、部署即可。这样，搭建网站的技术门槛进一步下降，你只需要会两件事就能搭建网站了：   
  
1. 会写 Markdown 文档:
2. 注册 GitHub 或 Gitee 账号，点点鼠标，在你的代码仓库中启用 Pages 服务。  


&nbsp;&nbsp; 因为技术门槛如此之低，导致不少用户压根就意识不到 Pages 服务内置了 Jekyll 工具，甚至以为每一个 Markdown 文档理所当然地就能变成一个网页。此外，另一个常被忽视的问题是：由 Pages 服务调用的 Jekyll 工具，并非最新版本，而且隐性地增添了许多插件，这可能使用户在本地使用 Jekyll 或迁移平台时碰上「不协调」的问题。最常见的一个问题就是：在 GitHub Pages 上正常生成的代码仓库，到 Gitee Pages 上就变得一团糟。这不是因为 Gitee Pages 的功能「不如」GitHub Pages，而是因为：  
&nbsp;&nbsp; GitHub Pages 没有告诉你它们为自己的 Jekyll 多加了几个插件，Gitee Pages 也没有告诉你它们的 Jekyll 并没有这些插件。  
这里对 GitHub Pages 与 Gitee Pages 所使用的 Jekyll 进行一个简单的分析（后面姑且简称为 GitHub Jekyll 与 Gitee Jekyll），以说明它们隐性地附加了哪些功能，需要特别注意。  
<br>

### <b>Jekyll on Github Pages</b>
----


GitHub Pages 中所采用的 Jekyll 及插件、依赖，被汇总到名为 github-pages 的 Gem 中（主页）。如果你在本地安装了这个 Gem，可以运行它来查看其所要求的各项依赖版本。以目前最新的 204 版本为例，在 shell 中运行：  
>   
$ github-pages versions  
+------------------------------+---------+  
| Gem | Version |  
+------------------------------+---------+  
| jekyll | 3.8.5 |  
| jekyll-sass-converter | 1.5.2 |  
| kramdown | 1.17.0 |  
| jekyll-commonmark-ghpages | 0.1.6 |  
| liquid | 4.0.3 |  
| rouge | 3.13.0 |  
| github-pages-health-check | 1.16.1 |  
| jekyll-redirect-from | 0.15.0 |  
| jekyll-sitemap | 1.4.0 |  
| jekyll-feed | 0.13.0 |  
| jekyll-gist | 1.5.0 |  
| jekyll-paginate | 1.1.0 |  
| jekyll-coffeescript | 1.1.1 |  
| jekyll-seo-tag | 2.6.1 |  
| jekyll-github-metadata | 2.13.0 |  
| jekyll-avatar | 0.7.0 |  
| jekyll-remote-theme | 0.4.1 |  
| jemoji | 0.11.1 |  
| jekyll-mentions | 1.5.1 |  
| jekyll-relative-links | 0.6.1 |  
| jekyll-optional-front-matter | 0.3.2 |  
| jekyll-readme-index | 0.3.0 |  
| jekyll-default-layout | 0.1.4 |  
| jekyll-titles-from-headings | 0.5.3 |  
| jekyll-swiss | 1.0.0 |  
| minima | 2.5.1 |  
| jekyll-theme-primer | 0.5.4 |  
| jekyll-theme-architect | 0.1.1 |  
| jekyll-theme-cayman | 0.1.1 |  
| jekyll-theme-dinky | 0.1.1 |  
| jekyll-theme-hacker | 0.1.1 |  
| jekyll-theme-leap-day | 0.1.1 |  
| jekyll-theme-merlot | 0.1.1 |  
| jekyll-theme-midnight | 0.1.1 |  
| jekyll-theme-minimal | 0.1.1 |  
| jekyll-theme-modernist | 0.1.1 |  
| jekyll-theme-slate | 0.1.1 |  
| jekyll-theme-tactile | 0.1.1 |  
| jekyll-theme-time-machine | 0.1.1 |  
+------------------------------+---------+  

<br/>

&nbsp;&nbsp; 能看到其列出来一大串的 Gem。通过这个页面也可以看到 GitHub Pages 上的 Jekyll 版本及相关依赖。  
以上这些 Gem，可以大致划分为四类：  

	1. Jekyll 及其依赖，比如 Sass 转换、Kramdown 引擎、Liquid 模板语言、Rouge 高亮器等等。这些算是常规构件，不可或缺。注意，目前 GitHub Pages 使用的 Jekyll 版本为 3.8.5，而最新版本是 4.0.0，有一个「适当」的延迟。  
	2. 为 GitHub Pages 定制的额外功能，主要有两个：  jekyll-commonmark-ghpages，在 Commonmark 基础上改出来的 GFM 引擎（但 Jekyll 仍然默认用 Kramdown）；github-pages-health-check，用于检查域名（DNS 服务）和 GitHub Pages 服务是否正常。  
	3. 若干 Jekyll 插件，基本上都是 jekyll 开头。后面会详细分析。  
	4. 若干 Jekyll 主题，除了 Jekyll 的默认主题 minima 和一个基本主题 jekyll-swiss 之外，还有 13 个 jekyll-theme 开头的，它们就是你在 GitHub Pages 服务里即选即用的 13 个主题。  


&nbsp;&nbsp; 从以上后三类可以看到，GitHub Jekyll 其实「加持」了很多的辅助件，并不单纯。而这样多的辅助构件，最终营造出了前面所提的「每一个 Markdown 文档理所当然地可以变成一个网页」之幻觉。事实上，如果是纯粹用 Jekyll 搭建网站，所需要做的工作仍然是不少的。  
下面再详细分析一下 GitHub Jekyll 所采用的插件。

&nbsp;&nbsp; Jekyll 的 Markdown 引擎在 Maruku 停止更新后，Jekyll 的默认 Markdown 引擎变成了 Kramdown，同样也是一个用 Ruby 开发的工具。Kramdown 实现了相当多的拓展功能，典型者如 LaTeX 公式、行内属性标记等，拓展了用 Markdown 实现网页（HTML）的可能性。
GitHub Jekyll 也是默认用 Kramdown 渲染 Markdown，不过前面看到其也提供了一个 GFM 引擎。在 GitHub 的官方文档中对此有特别说明，并强调「只有使用后者才能保证网站效果与 GitHub 中（渲染的 Markdown 页面）的一样」。仔细想想，有这个需求的用户应该不在少数。  

#### 常规插件

在 GitHub Jekyll 所用插件之中，下面这些是比较常规、常见的（强调者表示默认启用）：   

	* jekyll-sitemap，用于生成站点地图文件 sitemap.xml 供搜索引擎抓取；  
	* jekyll-feed，用于生成 RSS 订阅链接 feed.xml；  
	* jekyll-coffeescript，CoffeeScript 转换器；  
	* jekyll-redirect-from，重定向插件，从功能上可以理解为 permalink 的反面；  
	* jekyll-paginate，分页器；  
	* jemoji，表情包；  
	* jekyll-avatar，提供了形如 avatar [username] 的标签，用于获取 GitHub 用户的头像；  
	* jekyll-remote-theme，使你能够使用挂在 GitHub 上的 Jekyll 主题；  
	* jekyll-gist，提供了形如  gist xxx 的标签，用于在页面上展示 Gist 的内容；  
	* jekyll-mentions，使得 GitHub 上的 @ 用户功能在网站中得到支持；  
	* jekyll-relative-links，能够将指向 Markdown 文档的链接转换为指向对应 HTML 页面的链接（有点鸡肋）。  


&nbsp;&nbsp; 其中许多算是 Jekyll 的标配插件，经常被使用。它们更多地是提供了一种可选项，不会对网站的生成效果有太大影响。  
<br>
#### 静默增强插件

&nbsp;&nbsp; 除了上面所提的基本插件，另外的插件则非常「阴险」，默认启用，发挥了一些你根本意识不到的功能。包括：   

	* jekyll-seo-tag：定制了 {% seo %} 这个 Liquid 标签的功能。SEO 的其他方面不说，网站的 <title> 元素就是它搞定的（使用了 _configs.yml 文件中的 title 和 description 属性）。所以在用 GitHub Jekyll 时要想改变网页标题的格式，就必须要求它停止输出标题： `seo title=false`，否则你会以为标题是「无中生有」的。  
	* jekyll-github-metadata：用于从 GitHub 获取元信息，比如项目名称、作者之类的。它主要是给 GitHub Pages 生成的网站提供一些默认参数，比如上面的 SEO 插件就会使用 GitHub 仓库的项目名称、描述作为网站的标题和副标题。在本地用 Jekyll 构建 Pages 上的网站时，十有八九会出现「No GitHub API authentication」的警告，这个锅也得由它来背（它要用 GitHub API 来获取这些信息）。  
	* jekyll-optional-front-matter：根据 Jekyll 的机制，其只会转换有 YAML 头信息（哪怕是空的）的 Markdown 文件，这个插件则取消了这一要求。所以如果你发现用在其他场合使用 Jekyll 时许多 Markdown 文件没有被转换，你会意识到这个插件的作用：让你不用写头信息。（另外，这个规则对 Sass 文件不使用，所以你得对自己写的、放在 _sass 目录以外的 Sass 文件至少给一个空的头信息。）  
	* jekyll-readme-index：这个插件使得 Jekyll 在找不到 index.html 或 index.md 时，将 README.md 转换为 index.html 作为替代。这个功能的好处在于实现了 GitHub 页面预览和网站构建的统一，因为在 GitHub 页面上 README 的作用就相当于一般网站的 index.html。  
	* jekyll-default-layout：帮助你自动给首页套 layout: home、给推送文章套 layout: post、给一般页面套 layout: page、实在不行就套 layout: default。作用很明显：让你不用写头信息。  
	* jekyll-titles-from-headings：自动将一个没有指明 title 的 Markdown 文件之首级标题提取为 title。从页面显示来说，一个页面有没有 title 其实无关紧要，但需要生成网站导航、文章列表等的时候就必须确保每个页面都有 title。这个功能的作用也很明显：让你不用写头信息。  

<br>
以上几个插件，都是「静默」生效；其中不少在 Jekyll 中并不默认启用，但它们在 GitHub Jekyll 中全都是默认启用的。它们发挥的作用，也许你之前从未意识到，但现在一看即知。
<br><br>

### <b>GitHub Pages  </b>

&nbsp;&nbsp; 主题GitHub Pages 提供的 13 个基本主题，也被包含在依赖当中，这意味着你不需要安装就能使用它们。它们在 _configs.yml 中用 theme 属性启用（也许这是许多人见过的第一行 YAML 代码？），这会给初学者造成一种误解，以为其他的主题也可以这样通过一行代码来使用。  
&nbsp;&nbsp; 事实上，如果要在 GitHub Jekyll 中使用其他主题，有这样两种办法：  

	1. 启用 jekyll-remote-theme 插件，这样你就可以使用任意一个在 GitHub 上公开的 Jekyll 主题（其他地方的不行）；
	2. 把主题下载下来，将对应文件拷贝到指定位置 —— 注意清理之前的主题。（Jekyll 的主题管理不是很灵活，不如 Hugo、Hexo 等工具。）


&nbsp;&nbsp; 当然，如果你是在本地生成网站文件后再借 Pages 的服务器发布，方法就比较多了。

### 总结

&nbsp;&nbsp; 经过以上对各个依赖的分析，我们可以发现：GitHub Jekyll 提供了相当多的辅助功能，极大的化简了网站的生成，而我们甚至还不自知。在不清楚这些背景的情况下，尝试从 GitHub Pages 服务下迁移出 Jekyll 项目，很有可能会踩坑，比如：   

	* 为什么这个 Markdown 文件没有被转换？（因为你没有写头文件，怪 jekyll-optional-front-matter）
	* 为什么文章列表里的文章标题都是空的？（因为它们的标题是正文中的 h1 标签，没有写到头信息中的 title 里，怪 jekyll-titles-from-headings）
	* 为什么这个路径下的页面不见了？（因为你原来用的是 README.md 转换成 index.html，怪 jekyll-readme-index）


&nbsp;&nbsp; 当然，问题的可能性不多，一一排除总能解决。所以归结出一个结论：与其花时间琢磨上面这么多 Gem 的关系，还不如自己去踩踩坑。  
&nbsp;&nbsp; 坑归坑，好话也还是要说几句：如果只是一直用 GitHub Jekyll，以上这些都不算是问题，而算优势。{ % highlight}「Markdown 文件自动变成网页」{ % highlight} 这样的好事，还是人人所欲的；它毕竟能让许多完全不了解前端技术的人构建一整个网站出来，应该算是大好事。
<br>