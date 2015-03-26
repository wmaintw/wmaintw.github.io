---
layout: post
title: 在Protractor中动态设置代理
category: tech
---

[Protractor](http://angular.github.io/protractor)的`conf.js`文件中可以设置浏览器的代理，就像下面这样：

{% highlight javascript linenos %}
exports.config = {
  seleniumAddress: 'http://localhost:4444/wd/hub',
  specs: ['demo_spec.js'],
  capabilities: {
    browserName: 'chrome',

    proxy: {
      proxyType: 'manual',
      httpProxy: 'localhost:8080'
    }
  }
}
{% endhighlight %}

大多数情况下通过这种方式来设置代理就足够满足日常需求了，不过始终有例外的情况，比如说，在本地运行测试的时候不想经过代理，但是在CI服务器上想要使用代理。
有多种方式可以达到这样的目的，例如在不同的机器上使用不同的配置文件，但是这样做带来的维护成本比较高，这里将要介绍一种使用环境变量来动态设置代理的方法。

在`conf.js`文件里，这么设置proxy:

{% highlight javascript linenos %}
exports.config = {
  seleniumAddress: 'http://localhost:4444/wd/hub',
  specs: ['demo_spec.js'],
  capabilities: {
    browserName: 'chrome',

    proxy: {
      proxyType: process.env.PROTRACTOR_PROXY_TYPE,
      httpProxy: process.env.PROTRACTOR_HTTP_PROXY
    }
  }
}
{% endhighlight %}

可以看到，`conf.js`使用了环境变量`PROTRACTOR_PROXY_TYPE`和`PROTRACTOR_HTTP_PROXY`，当然你也可以更改成你喜欢的变量名。
之后，在命令行里分别设置这两个环境变量：

{% highlight sh %}
export PROTRACTOR_PROXY_TYPE=manual
export PROTRACTOR_HTTP_PROXY=localhost:8080
{% endhighlight %}

最后，你就可以像平常那样运行Protractor命令，不过现在浏览器的代理已经被设置到`localhost:7070`了：

{% highlight sh %}
protractor conf.js
{% endhighlight %}

另外，如果不想让浏览器使用代理，可以再设置一下`PROTRACTOR_PROXY_TYPE`变量，修改其值为`direct`，之后Protractor就不会再给浏览器设置代理了，就像这样：

{% highlight sh %}
export PROTRACTOR_PROXY_TYPE=direct
{% endhighlight %}

**参考：**

- [Protractor DesiredCapabilities](https://github.com/SeleniumHQ/selenium/wiki/DesiredCapabilities)
- [Protractor configuration file reference](https://github.com/angular/protractor/blob/master/docs/referenceConf.js)
