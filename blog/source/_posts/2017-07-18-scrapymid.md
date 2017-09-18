---
title: 使用scrapy的写并行爬虫
date: 2017-07-18 21:21:27
categories:
  - crawl
tags:
  - crawl
  - python
---


## 正文

&ensp;&ensp;&ensp;前段时间我们写了爬取电视剧，今天我把它改变成了爬取游戏，然后boss又来需求了，让这两个并行爬取（因为后面会使用并行爬取加快爬取速度），然后就自己鼓捣了一下。

<!--more-->
&ensp;&ensp;&ensp;先跑去[官网](https://doc.scrapy.org/en/latest/topics/practices.html)看了一下。

&ensp;&ensp;&ensp;找到了如何并行运行爬虫：

```python
import scrapy
from scrapy.crawler import CrawlerProcess

class MySpider1(scrapy.Spider):
    # Your first spider definition
    ...

class MySpider2(scrapy.Spider):
    # Your second spider definition
    ...

process = CrawlerProcess()
process.crawl(MySpider1)
process.crawl(MySpider2)
process.start() # the script will block here until all crawling jobs are finished
```

&ensp;&ensp;&ensp;这是一个简单的python脚本，使用`python script_name`即可运行它，然后自己写了个脚本运行，发现这种方法有个缺陷：不能经过pipeline处理。

&ensp;&ensp;&ensp;那怎么办？要想使用pipeline，需要使用`scrapy`的命令，如`scrapy crawl spider_name`，但是现有的scrapy命令不提供咱们要实现的功能怎么办？自定义呗^-^，于是一查，果然scrapy提供了自定义命令：

* 在与`spiders`同级的目录下创建目录，`mkdir commands`
* 在`commands`目录中添加`__init__.py`文件
* 在`commands`目录创建`crawlall.py`文件（文件名自拟，只要跟后面的配置相同即可),并写入下面代码
* `settings.py`目录下创建`setup.py`，写入：
    ```python
    from setuptools import setup, find_packages

    setup(name='scrapy-mymodule',
      entry_points={
        'scrapy.commands': [
          'crawlall=cnblogs.commands:crawlall',
        ],
      },
     )
 ```
&ensp;&ensp;&ensp;这个文件的含义是定义了一个crawlall命令，cnblogs.commands为命令文件目录，crawlall为命令名。
* 在settings.py中添加配置： `COMMANDS_MODULE = 'project_name.commands'`
* 运行命令scrapy crawlall
&ensp;&ensp;&ensp;写入setup.py中的代码
    ```python

from scrapy.commands import ScrapyCommand
from scrapy.crawler import CrawlerRunner
from scrapy.utils.conf import arglist_to_dict


class Command(ScrapyCommand):
    requires_project = True

    def syntax(self):
        return '[options]'

    def short_desc(self):
        return 'Runs all of the spiders'

    def add_options(self, parser):
        ScrapyCommand.add_options(self, parser)
        parser.add_option(
            "-a", dest="spargs",
            action="append", default=[], metavar="NAME=VALUE",
            help="set spider argument (may be repeated)")
        parser.add_option("-o", "--output", metavar="FILE",
                          help="dump scraped items \
                          into FILE (use - for stdout)")
        parser.add_option("-t", "--output-format", metavar="FORMAT",
                          help="format to use for dumping items with -o")

    def process_options(self, args, opts):
        ScrapyCommand.process_options(self, args, opts)
        try:
            opts.spargs = arglist_to_dict(opts.spargs)
        except ValueError:
            raise UsageError(
                "Invalid -a value, use -a NAME=VALUE", print_help=False)

    def run(self, args, opts):
        spider_loader = self.crawler_process.spider_loader
        for spidername in args or spider_loader.list():
            self.crawler_process.crawl(spidername, **opts.spargs)
        self.crawler_process.start()
    ```

&ensp;&ensp;&ensp;观察上面的代码，大部分都看不懂，但是`run`函数内的东西应该能看懂，大题知道它是在拿到所有的爬虫并运行它，那么事情就简单了，我们对爬虫的调度工作就放在这里了：

```python
    def run(self, args, opts):
        runner = CrawlerRunner()
        runner.crawl(GamespiderSpider)
        runner.crawl(TengxunSpider)
        d = runner.join()
        d.addBoth(lambda _: reactor.stop())

        ```

&ensp;&ensp;&ensp;运行发现可以抓取但是还是无法使用`pipeline`，对比代码发现我们的代码再`run,crawl`函数中没有传递`**opts.spargs`，那么现在一切都明了了，前面的脚本无法使用`pipeline`是因为无法得到相关参数（`args`,`opts`),这些参数与scrapy相关，包含了框架的一些行为和信息。

```python
    def run(self, args, opts):
        runner = CrawlerRunner()
        runner.crawl(GamespiderSpider, **opts.spargs)
        runner.crawl(TengxunSpider, **opts.spargs)
        d = runner.join()
        d.addBoth(lambda _: reactor.stop())

        ```
&ensp;&ensp;&ensp;改为上面的代码可以完美完成任务。

## 参考链接
