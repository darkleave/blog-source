---
title: Scrapy 教程
date: 2018-05-23 23:09:42
tags: [docker]
---

在这个教程里面, 我们会假设Scrapy 已经安装在你的系统里面,如果还没安装的话，查看[安装教程](https://docs.scrapy.org/en/latest/intro/install.html#intro-install)

<!--more-->

我们准备爬取 [quotes.toscrape.com]()，这个网站引用了许多知名作者的名言.

这个教程将通过下述任务带领你学习:

1. 创建一个新的Scrapy(爬虫)项目
2. 编写一个[spider](https://docs.scrapy.org/en/latest/topics/spiders.html#topics-spiders) 来爬取网站和解析数据
3. 使用命令行导出爬取的数据
4. 使用spider 去递归超链接
5. 使用spider 参数

Scrapy 是使用[Python](https://www.python.org/)写的.

如果你已经熟悉其它语言，并且想要快速地学习Python,我们推荐你阅读[Dive Into Python3](http://www.diveintopython3.net/).或者阅读[Python 教程](https://docs.python.org/3/tutorial).

如果你是编程新手并且想要开始学习Python,你可能需要找到有用的网络书籍 [Learn Python The Hard Way](https://learnpythonthehardway.org/book/). 你也可以同样看看[ this list of Python resources for non-programmers](https://wiki.python.org/moin/BeginnersGuide/NonProgrammers).

## 创建一个项目

在你开始爬取数据之前，你必须去创建一个新的Scrpay 爬虫项目.进入到你想要存储代码的目录并且运行:

	scrapy startproject tutorial

这将会创建一个`tutorial`目录并且包含以下内容:

	tutorial/
		scrapy.cfg         #deploy configuration file
	
		tutorial/          #project's Python module,you'll import your code from here
			__init__.py
			items.py       #project items definition file
			middlewares.py #project middlewares file
			piplines.py    #project pipelines file
			settings.py	   #project settings file
			spiders/	   #a directory where you'll later put your spiders
				__init__.py


## 我们的第一只Spider 蜘蛛

Spiders 是你定义的类，Scrapy 使用它们来从网站爬取信息(或者一系列的网站).它们必须是`scrapy.Spider`的子类, 并且定义初始化的请求, 比如怎样追踪页面的一些超链接, 还有怎样转换解析下载页面的内容数据.

如下是我们第一只Spider的代码. 保存到你项目路径下的`tutorial/spiders`目录下的`quotes_spider.py`文件中：


	import scrapy
	
	class QuotesSpider(scrapy.Spider):
		name = "quotes"
		def start_requests(self):
			urls = [
				'http://quotes.toscrape.com/page/1/',
				'http://quotes.toscrape.com/page/2/',
			]
			for url in urls:
				yield scrapy.Request(url=url,callback=self.parse)
	
		def parse(self,response):
			page = response.url.split("/")[-2]
			filename = 'quotes-%s.html' % page
			with open(filename, '2b') as f:
				f.write(response.body)
			self.log('Saved file %s' % filename)


正如你所看到的,我们的Spider 继承了 `scrapy.Spider` 并且定义了一些属性和方法:

* `name`:Spider的唯一标识. 它必须在一个项目中保持唯一,这意味着,你不能在不同的Spiders之间使用同样的名称.

* `start_requests()`: 必须返回一个可迭代的Requests请求(你可以返回一个requests列表或者写一个生成器方法) ,Spider会使用它们来嗅探你想要的内容. 后续的requests请求将从这些初始请求中成功生成.

* `parse()` : 这个方法将用来处理每个成功请求响应下载的内容. 响应参数是`TextResponse` 的实例,这个实例持有整个页面的内容并且含有进一步的处理这些内容的方法.

`parse()` 方法通常用来转换reponse, 提取爬取的数据作为字段并且寻找新的URLs去追踪并且从它们那里创建新的请求.

## 如何运行你的spider

为了让我们的spider 工作, 回到项目的顶级目录并且运行:

	scrapy crawl quotes

这个命令通过spider名称`quotes` 来运行我们刚刚添加的spider, 它将会发送一些请求到 `quotes.toscrape.com` 域名. 你将会收到的输出类似这样:

	... (omitted for brevity)
	2016-12-16 21:24:05 [scrapy.core.engine] INFO: Spider opened
	2016-12-16 21:24:05 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
	2016-12-16 21:24:05 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
	2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (404) <GET http://quotes.toscrape.com/robots.txt> (referer: None)
	2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
	2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/2/> (referer: None)
	2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-1.html
	2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-2.html
	2016-12-16 21:24:05 [scrapy.core.engine] INFO: Closing spider (finished)
	...


现在，在当前目录检查文件.你应该能注意到两个新的文件已经被创建: quotes-1.html 和 quotes-2.html ,分别对应URLs请求的内容，并且通过`parse`方法进行了转换.

	Note
	如果你好奇为什么我们还没有转换HTML,稍等一下，我们很快会涉及到


## 在神秘面纱的掩盖下还发生了什么?

Scrapy 通过 Spider的`start_requests`方法获得**`scrapy.Request`**对象.通过从每个请求接收到response,它会实例化`**Response**`对象并且调用与request相关联的回调方法(在这个例子中，就是`parse`方法),并且将response作为参数.

## start_requests 方法的一个捷径

相比实现**`start_requests()`**方法来从URLs生成**`scrapy.Request`**对象,你可以仅定义一个**`start_urls`**类属性来包含一个URLs列表. 这个泪飙将会被**`start_requests()`**的默认实现用来为你的spider创建默认请求:


	import scrapy
	
	
	class QuotesSpider(scrapy.Spider):
	    name = "quotes"
	    start_urls = [
	        'http://quotes.toscrape.com/page/1/',
	        'http://quotes.toscrape.com/page/2/',
	    ]
	
	    def parse(self, response):
	        page = response.url.split("/")[-2]
	        filename = 'quotes-%s.html' % page
	        with open(filename, 'wb') as f:
	            f.write(response.body)


`parse()`将用来处理那些URL所对应的请求,尽管我们没有明确指定Scrapy这样去做. 这是因为`parse()`方法是Scrapy的默认回调方法,用来处理那些没有明确指定回调方法的请求.

## 提取数据


学习使用Scrpay提取数据的最好方式是使用 [Scrapy shell](https://doc.scrapy.org/en/latest/topics/shell.html#topics-shell).运行:

	scrapy shell 'http://quotes.toscrape.com/page/1/'



	Note:
	当在命令行运行Scrapy shell的时候，记得要用引号闭合URL,否则url包含的参数(比如.`&`带的参数)将不会工作.
	
	在Windows环境，使用双引号:
	
	scrapy shell "http://quotes.toscrape.com/page/1/"


你将会看到如下输出:

	[ ... Scrapy log here ... ]
	2016-09-19 12:09:27 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
	[s] Available Scrapy objects:
	[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
	[s]   crawler    <scrapy.crawler.Crawler object at 0x7fa91d888c90>
	[s]   item       {}
	[s]   request    <GET http://quotes.toscrape.com/page/1/>
	[s]   response   <200 http://quotes.toscrape.com/page/1/>
	[s]   settings   <scrapy.settings.Settings object at 0x7fa91d888c10>
	[s]   spider     <DefaultSpider 'default' at 0x7fa91c8af990>
	[s] Useful shortcuts:
	[s]   shelp()           Shell help (print this help)
	[s]   fetch(req_or_url) Fetch request (or URL) and update local objects
	[s]   view(response)    View response in a browser
	>>>


使用shell，你可以通过使用response 对象的[CSS](https://www.w3.org/TR/selectors)语法来选择元素.

	>>> response.css('title')
	[<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]


运行`response.css('title')`的结果是一个类似列表的对象，命名为**`SelectorList`**,它代表了一个**`Selector`**对象列表，这些对象包含了XML/HTML元素并且允许你运行一些查询来获得更小粒度的选择器或者提取数据.

从上述的above提取text，如下:

	>>> response.css('title::text').extract()
	['Quotes to Scrape']

这里有两个我们需要注意的地方,其一是我们添加了`::text`到css查询,这意味着我们只想要选择在`<title>`元素中的text元素.如果我们不指定`::text`,我们将会会的这个title元素,包含它的tags:

	>>> response.css('title').extract()
	['<title>Quotes to Scrape</title>']

另一件事就是调用的结果.`.extract()`是一个列表,因为我们处理的是**`SelectorList`**的实例.当你知道你仅仅只想要首个结果，就像在这个例子中,你可以这样做:

	>>> response.css('title::text').extract_first()
	'Quotes to Scrape'


或者:

	>>> response.css('title::text')[0].extract()
	'Quotes to Scrape'


然而,使用`.extract_first()` 避免了一个 `IndexError`错误 并且当它没有找到任何可匹配的元素时返回 `None`.

注意：对于大多数scraping 代码，你想要当在页面上查找不到你想要的数据而发生错误时显得更有弹性的话,所以即使有一些部分对scraped失败了,你至少可以获得**部分**数据.


除了`extract()`方法和 `extract_first()` 方法,你同样能使用`re()` 方法来使用正则表达式来提取数据:
	
	>>> response.css('title::text').re(r'Quotes.*')
	['Quotes to Scrape']
	>>> response.css('title::text').re(r'Q\w+')
	['Quotes']
	>>> response.css('title::text').re(r'(\w+) to (\w+)')
	['Quotes', 'Scrape']


为了找到适当的CSS 选择器来使用,你可以通过`view(response)`来在浏览器直接打开响应页面. 你可以使用你的浏览器开发者工具或者扩展类似Firebug(详细查看[Using Firebug for scraping](https://doc.scrapy.org/en/latest/topics/firebug.html#topics-firebug) 和 [Using Firefox for scraping](https://doc.scrapy.org/en/latest/topics/firefox.html#topics-firefox)).

[Selector Gadget](http://selectorgadget.com/)同样是一个能够快速找到CSS选择器的绝佳工具,在很多浏览器它都工作良好.

## Xpath:简要介绍

除了[CSS](https://www.w3.org/TR/selectors),Scrapy 选择器同样支持使用[XPath](https://www.w3.org/TR/xpath)表达式:

	>>> response.xpath('//title')
	[<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
	>>> response.xpath('//title/text()').extract_first()
	'Quotes to Scrape'

XPath表达式功能非常强大,并且是Scrapy选择器的基础. 事实上,CSS选择器最后是转化为XPath表达式.

也许XPath没有CSS选择器那样受欢迎,但是XPath表达式提供更多强大的功能因为除了结构导航,它同样可以查看内容.使用XPath,你可以同样选中像如下的事物: 选中包含"Next Page"内容的超链接.这使得XPath 与scraping任务非常适宜,尽管你已经知道如何去操作CSS选择器我们仍然鼓励你去学习XPath,它会使得scraping更加容易.

我们不想再这个教程涉及太多XPath的内容,详情阅读[using XPath with Scrapy Selectors here](https://doc.scrapy.org/en/latest/topics/selectors.html#topics-selectors). 学习更多XPath的内容，我们推荐[this tutorial to learn XPath through examples](http://zvon.org/comp/r/tut-XPath_1.html)或者[this tutorial to learn "how to think in XPath"](http://plasmasturm.org/log/xpath101/).

## 提取名言和作者

现在你大致了解如何选中和解析了,接下来完成从web页面解析名言的spider代码.

[http://quotes.toscrape.com]()页面的每个引用名言都被HTML元素包含，类似这样:


	<div class="quote">
	    <span class="text">“The world as we have created it is a process of our
	    thinking. It cannot be changed without changing our thinking.”</span>
	    <span>
	        by <small class="author">Albert Einstein</small>
	        <a href="/author/Albert-Einstein">(about)</a>
	    </span>
	    <div class="tags">
	        Tags:
	        <a class="tag" href="/tag/change/page/1/">change</a>
	        <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
	        <a class="tag" href="/tag/thinking/page/1/">thinking</a>
	        <a class="tag" href="/tag/world/page/1/">world</a>
	    </div>
	</div>


在scrapy shell 中执行如下命令:

	$ scrapy shell 'http://quotes.toscrape.com'

我们获得一系列quote HTML元素的选择器:

	>>> response.css("div.quote")

上述查询返回的每个选择器允许我们运行更多查询来获得它们的子元素等.让我们将首个选择器复制给一个变量,这样我们能通过运行我们的CSS选择器来直接操作一个特别的 “名言引用”:

	>>> quote = response.css("div.quote")[0]

现在，通过我们刚刚创建的`quote`对象提取`title`,`author`和`tags` :

	>>> title = quote.css("span.text::text").extract_first()
	>>> title
	'“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'
	>>> author = quote.css("small.author::text").extract_first()
	>>> author
	'Albert Einstein'

使用`.extract()` 方法获取所有的tags:

	>>> tags = quote.css("div.tags a.tag::text").extract()
	>>> tags
	['change', 'deep-thoughts', 'thinking', 'world']


现在我们可以迭代所有的名言引用元素，并且将它们放置到一个Python目录:

	>>> for quote in response.css("div.quote"):
	...     text = quote.css("span.text::text").extract_first()
	...     author = quote.css("small.author::text").extract_first()
	...     tags = quote.css("div.tags a.tag::text").extract()
	...     print(dict(text=text, author=author, tags=tags))
	{'tags': ['change', 'deep-thoughts', 'thinking', 'world'], 'author': 'Albert Einstein', 'text': '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'}
	{'tags': ['abilities', 'choices'], 'author': 'J.K. Rowling', 'text': '“It is our choices, Harry, that show what we truly are, far more than our abilities.”'}
	    ... a few more of these, omitted for brevity
	>>>

## 在我们的spider中提取数据

现在让我们回到自己的spider.直到现在,它并不提取任何特别的数据，仅仅保存整个HTML页面到本地 .现在集成提取逻辑到我们的spider中。

一个Scrpay spider 通常生成很多包含从页面中解析出来的数据的字典. 为了实现这一点，我们使用`yield` Python 关键字在回调上,正如你在如下看到的那样:

	import scrapy
	
	
	class QuotesSpider(scrapy.Spider):
	    name = "quotes"
	    start_urls = [
	        'http://quotes.toscrape.com/page/1/',
	        'http://quotes.toscrape.com/page/2/',
	    ]
	
	    def parse(self, response):
	        for quote in response.css('div.quote'):
	            yield {
	                'text': quote.css('span.text::text').extract_first(),
	                'author': quote.css('small.author::text').extract_first(),
	                'tags': quote.css('div.tags a.tag::text').extract(),
	            }

如果你运行这只spider，它将会用日志输出如下数据:

	2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
	{'tags': ['life', 'love'], 'author': 'André Gide', 'text': '“It is better to be hated for what you are than to be loved for what you are not.”'}
	2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
	{'tags': ['edison', 'failure', 'inspirational', 'paraphrased'], 'author': 'Thomas A. Edison', 'text': "“I have not failed. I've just found 10,000 ways that won't work.”"}


## 存储我们爬取的数据

存储爬取数据的最简单方法是使用[Feed exports](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-exports),使用如下命令:

	scrapy crawl quotes -o quotes.json

这将会生成一个 `quotes.json`文件，包含所有爬取的items,序列化成[JSON](https://en.wikipedia.org/wiki/JSON)的形式.

因为一些历史上的原因，Scrapy 会将爬取的数据内容添加到所指定的文件后方，而不是直接重写覆盖. 如果你连续运行这个命令两次，并且在第二次命令之前没有移除这个文件，你将会获得拥有两份重复json数据的json文件.

其它的格式，像[JSON Lines](http://jsonlines.org/):

	scrapy crawl quotes -o quotes.jl

[JSON Lines](http://jsonlines.org/)格式非常有用，因为它类似流那样,你可以轻松地在它后面添加新的记录行.当你同样运行两次它不会产生跟JSON格式一样的问题.同样的，由于每条记录都是一行，最后你可能要遍历一个巨大的文件,但是你不必在内存中完成所有事情,通过[JQ](https://stedolan.github.io/jq)可以在命令行完成这些事情.

在小型的项目中(类似这个教程),这可能足够了.然而，如果你想要使用scrapy完成更复杂的事情.你可以写一个[Item Pipeline](https://doc.scrapy.org/en/latest/topics/item-pipeline.html#topics-item-pipeline).

当项目被创建的时候一个Item Pipelines 的占位符文件就已经被创建了，就在 `tutorial/pipelines.py` 目录下面.如果你只是想要存储scraped items 的话你不需要实现任何item pipelines .

## 追踪超链接

与在[http://quotes.toscrape.com]()网站前两个页面爬取一些信息相比，你想要这个网站所有页面的名言引用.

现在你知道如何从这些页面提取数据.接下来让我们了解一下如何从它们追踪超链接:

我们要做的第一件事就是从页面解析我们想要追踪的超链接, 我们可以看到在下个页面有一个如下标记的超链接:

	<ul class="pager">
	    <li class="next">
	        <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
	    </li>
	</ul>

我们可以在shell中对其进行解析:

	>>> response.css('li.next a').extract_first()
	'<a href="/page/2/">Next <span aria-hidden="true">→</span></a>'

这能获取 `a` 元素，但是我们想要的是 `href` 属性. Scrapy 支持CSS扩展能够让你选中属性内容,就像这样:

	>>> response.css('li.next a::attr(href)').extract_first()
	'/page/2/'

现在我们的spider 代码修改为能够自动追踪下个页面的超链接，并从它解析数据:
	
	import scrapy
	
	
	class QuotesSpider(scrapy.Spider):
	    name = "quotes"
	    start_urls = [
	        'http://quotes.toscrape.com/page/1/',
	    ]
	
	    def parse(self, response):
	        for quote in response.css('div.quote'):
	            yield {
	                'text': quote.css('span.text::text').extract_first(),
	                'author': quote.css('small.author::text').extract_first(),
	                'tags': quote.css('div.tags a.tag::text').extract(),
	            }
	
	        next_page = response.css('li.next a::attr(href)').extract_first()
	        if next_page is not None:
	            next_page = response.urljoin(next_page)
	            yield scrapy.Request(next_page, callback=self.parse)


现在，在解析完数据之后, `parse()` 方法从下一个页面寻找超链接,使用`urljoin()` 方法创建完全路径的URL(因为超链接是相关的)并且yields 一个新的请求到下一个页面,注册它本身作为回调来处理下个页面的数据解析并且确保能爬取到所有页面的数据.

你在这里看到的就是Scrapy 追踪超链接的原理:
但你在一个回调方法中yield 一个请求,Scrapy 将会安排这个请求在发送并且请求完成后执行它所注册的回调方法.

通过这个原理, 你可以根据你定义的规则构建追踪超链接的负责爬取器,并且根据它访问的页面解析不同类型的数据.

在我们的例子中，它创建了一个一种循环，追踪所有到下个页面的超链接直到它不能再找到下一个位置 - 这对于爬取依赖分页的博客，论坛或者其它网站非常遍历.

## 创建请求的捷径

你可以直接通过**`response.follow`**创建请求:


	import scrapy
	
	
	class QuotesSpider(scrapy.Spider):
	    name = "quotes"
	    start_urls = [
	        'http://quotes.toscrape.com/page/1/',
	    ]
	
	    def parse(self, response):
	        for quote in response.css('div.quote'):
	            yield {
	                'text': quote.css('span.text::text').extract_first(),
	                'author': quote.css('span small::text').extract_first(),
	                'tags': quote.css('div.tags a.tag::text').extract(),
	            }
	
	        next_page = response.css('li.next a::attr(href)').extract_first()
	        if next_page is not None:
	            yield response.follow(next_page, callback=self.parse)



与scrapy.Request不同, `response.follow` 直接支持关联的URL  不需要调用 urljoin. 需要注意的是 `response.follow` 只是返回一个Request实例; 你仍然需要 yield 这个请求.

你同样可以传递一个选择器到 `response.follow` 而不是一个string; 这个选择器需要解析必要的属性:

	for href in response.css('li.next a::attr(href)'):
	    yield response.follow(href, callback=self.parse)


对于 `<a>` 元素: `response.follow` 自动使用它们的href属性. 这样能进一步的简化代码:

	for a in response.css('li.next a'):
	    yield response.follow(a, callback=self.parse)

**Note:**

	`response.follow(response.css('li.next a'))` 是无效的 ,因为`response.css` 返回一个所有a元素结果集的选择器列表对象,不是一个单独的选择器. 需要用一个 `for` 循环来处理上述的例子,或者 `response.follow(response.css('li.next a')[0]` 也可以.

## 更多的例子和模式

这是另一个阐明回调和追踪超链接的spider,这次是为了爬取作者的信息:

	import scrapy
	
	
	class AuthorSpider(scrapy.Spider):
	    name = 'author'
	
	    start_urls = ['http://quotes.toscrape.com/']
	
	    def parse(self, response):
	        # follow links to author pages
	        for href in response.css('.author + a::attr(href)'):
	            yield response.follow(href, self.parse_author)
	
	        # follow pagination links
	        for href in response.css('li.next a::attr(href)'):
	            yield response.follow(href, self.parse)
	
	    def parse_author(self, response):
	        def extract_with_css(query):
	            return response.css(query).extract_first().strip()
	
	        yield {
	            'name': extract_with_css('h3.author-title::text'),
	            'birthdate': extract_with_css('.author-born-date::text'),
	            'bio': extract_with_css('.author-description::text'),
	        }


spider 将会从主页面开始,它将会追踪所有到作者页面的超链接并且调用`parse_author` 回掉方法来处理返回的每一个响应页面,并且同样追踪分页的超链接并调用`parse`回调方法如同之前的一样.

这里我们传递回调到 `response.follow` 作为位置参数来简化代码,它同样对 `scrapy.Request` 有效.

`parse_author` 回调方法定义了一个工具方法用来解析和清理从CSS查询获得的数据并且最终 yields 一个包含作者数据的Python 字典.

另一个有趣的事情是,在这个示例中，即使有很多名言引用来自同一个作者，但是我们并不需要担心会访问同一个作者页面多次.因为默认情况下,scrapy 会过滤已经访问过的URL请求，来防止因为程序错误而过多访问服务器的问题. 这个配置可在 **[DUPEFILTER_CLASS](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DUPEFILTER_CLASS)**中配置.


查看[CrawlSpider](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.CrawlSpider) 类，这是一个泛用的spider,实现了一些比较小的规则引擎，你可以在这个基础上开发自己的crawlers.

同时，一个普通的模式是从多个页面构建一个数据项,使用[trick to pass additional data to the callbacks](https://doc.scrapy.org/en/latest/topics/request-response.html#topics-request-response-ref-request-callback-arguments) 来实现.

## 使用spider 参数

你可以通过使用 `-a` 选项在spider运行的时候附加命令行参数:

	scrapy crawl quotes -o quotes-humor.json -a tag=humor

这些参数会被传递到 Spider 的 `__init__` 方法并且默认成为spider的属性》

在这里例子中, `tag` 参数可以通过 `self.tag` 进行调用, 你可以通过它来控制spider只爬取特定标签的的名言引用, 同时基于参数构建URL:


	import scrapy
	
	
	class QuotesSpider(scrapy.Spider):
	    name = "quotes"
	
	    def start_requests(self):
	        url = 'http://quotes.toscrape.com/'
	        tag = getattr(self, 'tag', None)
	        if tag is not None:
	            url = url + 'tag/' + tag
	        yield scrapy.Request(url, self.parse)
	
	    def parse(self, response):
	        for quote in response.css('div.quote'):
	            yield {
	                'text': quote.css('span.text::text').extract_first(),
	                'author': quote.css('small.author::text').extract_first(),
	            }
	
	        next_page = response.css('li.next a::attr(href)').extract_first()
	        if next_page is not None:
	            yield response.follow(next_page, self.parse)

如果你传递 `tag=humor` 参数到spider，那么你将会注意到spider将会只访问 `humor` 标签的URL,比如 `http://quotes.toscrape.com/tag/humor`.

更多spider参数查看[learn more about handling spider arguments here](https://doc.scrapy.org/en/latest/topics/spiders.html#spiderargs).

## 下一步

这个教程只涉及Scrapy最基本的东西,还有一大坨没有被提及的新特性.查看[还有啥?](https://doc.scrapy.org/en/latest/intro/overview.html#topics-whatelse) ,在[Scrapy at a glance](https://doc.scrapy.org/en/latest/intro/overview.html#intro-overview)章节中快速查看比较重要的部分.

你可以继续从小节[Basic concepts](https://doc.scrapy.org/en/latest/index.html#section-basics)中知道更多关于命令行工具,spiders,selectors 选择器和其它在教程中没有涉及的东西比如 模块化爬取数据. 如果你更喜欢通过例子项目来学习，查看[Examples](https://doc.scrapy.org/en/latest/intro/examples.html#intro-examples)小节.

