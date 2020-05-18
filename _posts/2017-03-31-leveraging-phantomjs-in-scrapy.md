---
layout: post
title:  Leveraging PhantomJS in Scrapy
date:   2017-03-31 20:00:20 +0800
categories:
tags:
- PhantomJS
- Scrapy
- Python
- Selenium
- Web Driver
- Web Scraping
---

In the world of web scraping, [`Scrapy`] is kind of the de facto standard. Other similar tools exist like [`CasperJS`] which exposes pure JavaScript APIs. They are more lightweight as compared to feature-rich Scrapy.

Scrapy is great, but not without any drawback. One of the biggest drawbacks in Scrapy is that it has no builtin mechanism to handle JavaScript execution / dynamic content which is very common today given the popularity of Single Page Application. Luckily, Scrapy is designed to be extensible. It is absolutely possible to hook up a third party component that handles dynamic content. 

Due to its nature of a non-GUI-based browser, [`PhantomJS`] is an ideal candidate that runs on a server environment. In this post, I will briefly explain how to integrate PhantomJS into a Scrapy's spider to handle dynamic content and simple page interactions.

### Installation

First, we need to install PhantomJS and [`Selenium`] into the system. On MacOS, PhantomJS can be installed with homebrew:

{% highlight text %}
brew install phantom
{% endhighlight %}

Selenium will be used as a driver that manages PhantomJS browser later on. It can be installed with pip:

{% highlight text %}
pip install selenium
{% endhighlight %}


### Middleware

The basis of the whole idea is to start up a PhantomJS instance in Scrapy's [Downloader Middleware]. It will hijack a http response, execute a few pieces of JavaScript, and then return the execution result as a new http response to the appropriate Spider. The advantage of utilizing Downloader Middleware's `process_request` API is that it prevents duplicated requests sent to the server (first from scrapy, then from webdriver) which can be in efficient.

{% highlight Python %}

import time
from scrapy.http import HtmlResponse
from selenium import webdriver

class PhantomMiddleware(object):

    def process_request(self, request, spider):
        try:
            # instantiate a driver that drives PhantomJS browser
            driver = webdriver.PhantomJS()

            # navigate to the given url
            driver.get(request.url)
            time.sleep(1)

            # find test-button in the page and simulate a click
            driver.execute_script('''
                var btn = document.getElementById("test-button");
                btn && btn.click();''')
            time.sleep(1)

            # get the page source, construct a new HtmlResponse and return it
            # to Spider
            body = driver.page_source
            return HtmlResponse(
                driver.current_url, 
                body=body, 
                encoding='utf-8', 
                request=request)
        except Exception, e:
            logging.error(e)
            return None

{% endhighlight %}

### Spider code

[Spider] code is really simple, we just need to tell Scrapy that we need to use the Downloader Middleware we just created in `custom_settings` override. I cannot think of a better name, so for now let's just call the spider `DynamicSpider`.

{% highlight Python %}

class DynamicSpider(scrapy.Spider):
    name = "DynamicSpider"

    custom_settings = {
        'DOWNLOADER_MIDDLEWARES': {
            'PhantomMiddleware': 720
        }
    }

    def start_requests(self):
        # initiate a web request
        # this request will be hijacked by PhantomMiddleware
        return scrapy.Request(url='#...', callback=self.parse)

    def parse(self, response):
        # response will come from PhantomMiddleware
        self.logger.info(response.body)
        # ...

{% endhighlight %}


### Regarding user agent

That's it! We have created a Scrapy spider that handles / executes JavaScript content. 

Depending on the usage, there is one thing probably worth mentioning: some websites rejects PhantomJS's default user agent, so it is best to change it to something more recognizable, e.g. Chrome's user agent. We can do this fairly easily when we initialize `webdriver.PhantomJS`:

{% highlight Python %}
# ...
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
# ...
class PhantomMiddleware(object):

    def process_request(self, request, spider):
        try:
            user_agent = dict(DesiredCapabilities.PHANTOMJS)
            user_agent['phantomjs.page.settings.userAgent'] = (
                'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) '
                'AppleWebKit/537.36 (KHTML, like Gecko) '
                'Chrome/57.0.2987.133 Safari/537.36"'
            )
            driver = webdriver.PhantomJS(desired_capabilities=dcuser_agentap)
            # ...

{% endhighlight %}

[`CasperJS`]: https://github.com/casperjs/casperjs
[`Scrapy`]: https://github.com/scrapy/scrapy
[`PhantomJS`]: https://github.com/ariya/phantomjs
[`Selenium`]: https://pypi.python.org/pypi/selenium
[Spider]: https://doc.scrapy.org/en/1.3/topics/spiders.html
[Downloader Middleware]: https://doc.scrapy.org/en/1.3/topics/downloader-middleware.html

