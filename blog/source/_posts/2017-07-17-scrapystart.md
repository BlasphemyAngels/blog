---
title: scrapy初涉
date: 2017-07-17 22:18:30
categories:
  - crawl
tags:
  - crawl
  - scrapy
---

## 正文

&ensp;&ensp;&ensp;scrapy是一个开源的爬虫框架，由于实习需要，今天就看了一下，用这个框架的话很省很多时间啊，以前写爬虫时使用requests或者urllib，都是从无一点一点向上写，现在scrapy帮我们实现了一些底层的东西，比如获得一个页面。下面讲一下scrapy的基本用法。

<!--more-->

&ensp;&ensp;&ensp;scrapy的基本结构：

![scrapy](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/scrapy.png?raw=true)
&ensp;&ensp;&ensp;包含的组件：
* 引擎(Scrapy Engine) 
* 调度器(Scheduler)
* 下载器(Downloader)
* 蜘蛛(Spiders)
* 项目管道(Item Pipeline)
* 下载器中间件(Downloader Middlewares)
* 蜘蛛中间件(Spider Middlewares)
* 调度中间件(Scheduler Middlewares)

&ensp;&ensp;&ensp;工作流程：
-->绿线是数据流向
* 从初始的URL开始，Scheduler会将其交给Downloader进行下载
* 下载之后会交给Spider进行分析
* Spider分析出来的结果有两种
  * 一种是需要进一步抓取的链接，如"下一页"的链接，它们会被传回Scheduler
  * 另一种是需要保存的数据，它们被送到Item Pipeline里进行后期的处理（详细分析，过滤，存储等）
* 在数据流动的通道里还可以安装各种各样的中间件，进行必要的处理。

## scrapy基本命令

&ensp;&ensp;&ensp;创建scrapy项目：

`scrapy startproject first_crawl`

![scrapy1](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/scrapy1.png?raw=true)

&ensp;&ensp;&ensp;通过tree命令查看项目结构:

![scrapy2](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/scrapy2.png?raw=true)
### scrapy shell

`scrapy shell http://你要调试xpath的网址`

![scrapy3](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/scrapy3.png?raw=true)

## 爬取腾讯视频的所有电视剧

&ensp;&ensp;&ensp;编写item，item的定义再`items.py`文件中。

```python
class TVItem(scrapy.Item):
    title = scrapy.Field()
    iid = scrapy.Field()
    content = scrapy.Field()
```

&ensp;&ensp;&ensp;编写Spiders，spider 可以使用`scrapy genspider spider_name`生成。

```python
class TengxunSpider(scrapy.Spider):
    name = 'tengxun'
    start_urls = ['http://v.qq.com/x/list/tv']
    #  start_urls = ['http://v.qq.com/x/list/tv?&offset=4980']

    def parse(self, response):

        for tv in response.xpath(
                '/html/body/div[3]/div/div/div[1]/div[2]/div/ul/li'):
            item = TVItem()
            item['title'] = tv.xpath('div[1]/strong/a/text()').extract_first()
            item['iid'] = tv.xpath('div[1]/strong/a/@href').extract_first()
            content = dict()
            content['score'] = tv.xpath(
                'div[1]/div/em[1]/text()').extract_first(
                ) + tv.xpath('div[1]/div/em[2]/text()').extract_first()
            actors = []
            for tvp in tv.xpath('div[2]/a/text()'):
                actors.append(tvp.extract())
            content['actors'] = actors
            #  item['content'] = json.dumps(content)
            item['content'] = content
            #  print(item['name'])
            #  print(item['score_l'])
            #  print(item['score_s'])
            #  print(item['actors'])
            yield item
        next_page = response.xpath(
            '/html/body/div[3]/div/div/div[2]/a[2]/@href').extract_first()
        if next_page != 'javascript:;':
            print(type(next_page))
            url = urljoin(response.url, next_page)
            yield scrapy.Request(url, callback=self.parse)
```

&ensp;&ensp;&ensp;编写`pipeline.py`文件。

```python
class TVPipeline(object):
    def process_item(self, item, spider):
        tmp = item['iid'][item['iid'].rfind('/') + 1:]
        tmp = tmp[:tmp.rfind('.')]
        item['iid'] = 'tv' + tmp
        #  with open('tv.txt', 'a+') as f:
        #      f.write(item['iid'] + '      ' + item['title'] +
        #              '       ' + item['content'] + '\n')
        return item
        ```
&ensp;&ensp;&ensp;这样就完成了一个爬取电视剧的爬虫。



## 参考链接
