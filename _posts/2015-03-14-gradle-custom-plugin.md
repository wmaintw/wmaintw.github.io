---
layout: post
title: 如何自定义Gradle Plugin
category: tech
---

作为构建脚本，尽管Gradle的语法已经足够灵活，而且还有大量的插件，能够满足大部分的构建需求，不过黑天鹅事件总是会发生，总有些时候我们不得不自己开发一些插件来满足某些特殊需求。另外，有可能好几个项目要使用同一段构建脚本，为了重用代码，这时候就可以把这部分构建脚本抽出来制作成插件，接下来只需要在这些项目里加入几行代码就可以使用这个插件了。

这篇文章将会介绍：

- 如何自定义一个Gradle插件
- 如何本地发布 & 使用插件
- 如何把插件发布到互联网

## 如何自定义一个Gradle插件

首先，让我们给Gradle插件新建一个项目，由于Gradle的语法是基于Groovy的，所以这个项目也得是Groovy的项目。创建好的项目结构如下图所示。

![项目结构]({{site.url}}/assets/2015-03/project.jpg)

在这个新建的项目中，有几个地方值得注意：

**build.gradle**

这个文件是插件自身的构建脚本，我们需要应用一些插件，设置插件的依赖，还有就是指定插件的`group`和`version`，代码如下所示：

{% highlight groovy linenos %}
apply plugin: 'idea'
apply plugin: 'groovy'
apply plugin: 'maven'

repositories {
    mavenCentral()
}

dependencies {
    compile localGroovy()
    compile gradleApi()
}

group = 'com.hello'
version = '1.0.0'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: uri('../repo'))
        }
    }
}
{% endhighlight %}

为了方便调试，通过运行下面这个命令，我们可以把构建好的插件先暂时放到本地的某个目录里(和当前插件根目录平级的名为repo的目录)，当插件开发完毕之后再来修改配置，把插件发布到互联网上。

{% highlight sh %}
$ gradle uploadArchives
{% endhighlight %}

**greeting.properties**

在项目的`src/main/META-INF/gradle-plugins`目录下，有一个`greeting.properties`文件，这个文件的文件名是插件的名字，文件内容只有一行代码，用来指定当前插件的实现类，在运行这个插件的时候，gradle就可以从这里读取到这些信息。

{% highlight text linenos %}
implementation-class=com.hello.GreetingPlugin
{% endhighlight %}

**GreetingPlugin.groovy**

这个文件就是插件的实现类，它的内容如下：

{% highlight groovy linenos %}
package com.hello

import com.hello.GreetingConfigurationExtension
import org.gradle.api.Plugin
import org.gradle.api.Project

class GreetingPlugin implements Plugin<Project> {
    @Override
    void apply(Project project) {
        // write your custom code here
        project.task("greeting") << {
            println "Hello, World"
        }
    }
}
{% endhighlight %}

在`apply`方法中，我们调用gradle的`project`对象，定义了一个名为`greeting`的task，这个task的功能就是输出一句"Hello World"

## 如何本地发布，使用插件

有了上面的这些文件和配置之后，我们的插件就制作得差不多了，接下来让我们运行`gradle uploadArchives`，把插件发布到本地目录，之后在客户端项目中应用这个插件：

** 客户端项目的build.gradle文件 **

{% highlight groovy linenos %}
apply plugin: 'greeting'

buildscript {
    repositories {
        maven {
            url uri('../repo')
        }
    }
    dependencies {
        classpath('com.hello:greeting:1.0.0')
    }
}
{% endhighlight %}

然后在命令行里运行下面这个命令，就可以看到我们这个插件的输出结果了：

{% highlight sh %}
$ gradle greeting

"Hello, World"
{% endhighlight %}

## 如何把插件发布到互联网

到目前为止我们还只是在本地开发插件，在本地使用，如果要让其他机器上的项目也使用这个插件的话，可以把这个插件发布到公司内部的Maven库里，或者[发布到Gradle的插件库][1]，也可以发布到公共的Maven库。关于如何把插件发布到公共的Maven库基本上可以再单独写一篇文章了，所以在这里我们暂时从简，先把插件发布到Bintray，其他项目可以通过Bintray来下载使用我们的插件。

要把插件发布到Bintray，可以借助另一个gradle插件`plugindev`，于是我们只需在`build.gradle`文件里进行配置，然后在Bintray网站上申请一个账号就可以了。具体步骤如下：

**step 1. 应用`plugindev`插件**

修改`build.gradle`文件，引入`plugindev`插件，并配置相关信息

{% highlight groovy linenos %}
plugins {
    id 'nu.studer.plugindev' version '1.0.3'
}

apply plugin: 'idea'
apply plugin: 'groovy'
apply plugin: 'maven'

repositories {
    mavenCentral()
}

dependencies {
    compile localGroovy()
    compile gradleApi()
}

group = 'com.hello'
version = '1.0.0'

plugindev {
    pluginId = 'com.hello.greeting'
    pluginName = 'greeting'
    pluginImplementationClass 'com.hello.GreetingPlugin'
    pluginDescription 'Gradle plugin that ...'
    pluginLicenses 'Apache-2.0'
    pluginTags 'greeting', 'hello'
    authorId 'your (githut) id'
    authorName 'your name'
    authorEmail 'your email'
    projectUrl 'http://your/project/url'
    projectIssuesUrl 'http://your/project/issues'
    projectVcsUrl 'http://your/project/vcs'
    projectInceptionYear '2015'
    done()
}

bintray {
    user = "your bintray username"
    key = "your bintray api key"
    pkg.repo = 'maven'
}
{% endhighlight %}

关于`plugindev`的详细使用方法及其配置，请[参考这里][2]

**step 2. 申请Bintray账号**

上一步当中有一部分是Bintray的配置，这需要一个Bintray的账号，如果你还没有账号，那就赶紧去[注册一个][3]吧。访问速度有点慢，翻墙会快点。

*账号注册完毕后，在编辑个人信息的页面里可以看到自己的Bintray api key*

**step 3. 发布到Bintray**

如果上面的步骤都设置正确了的话，现在只需要运行一个gradle的task就可以把插件发布到Bintray了：

{% highlight sh %}
$ gradle publishPluginToBintray
{% endhighlight %}

**step 4. 在客户端项目通过Bintray使用插件**

插件发布到Bintray之后，就只差最后一步了，那就是在客户端项目进行少量的配置，通过Bintray获取到插件并使用。修改客户端项目的`build.gradle`文件：

{% highlight groovy linenos %}
apply plugin: "greeting"

buildscript {
    repositories {
        maven {
            url "http://dl.bintray.com/your-bintray-name/maven"
        }
        mavenCentral()
    }
    dependencies {
        classpath(
                'com.hello:hello.greeting:1.0.0'
        )
    }
}
{% endhighlight %}

在命令行里运行下面这个命令，就能看到我们通过Bintray下载的插件已经能正常工作了：

{% highlight sh %}
$ gradle greeting

"Hello, World"
{% endhighlight %}


[1]:http://plugins.gradle.org/submit
[2]:https://github.com/etiennestuder/gradle-plugindev-plugin
[3]:https://bintray.com
