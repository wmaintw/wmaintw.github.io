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

{{site.url}}/assets/project.png

在这个新建的项目中，有几个地方值得注意：

**build.gradle**

这个文件是插件自身的构建脚本，我们需要应用一些插件，设置插件的依赖，还有就是指定插件的`group`和`version`，代码如下所示：

```groovy
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
```

为了方便调试，通过运行下面这个命令，我们可以把构建好的插件先暂时放到本地的某个目录里(和当前插件根目录平级的名为repo的目录)，当插件开发完毕之后再来修改配置，把插件发布到互联网上。

```sh
$ gradle uploadArchives
```

**greeting.properties**

在项目的`src/main/META-INF/gradle-plugins`目录下，有一个`greeting.properties`文件，这个文件的文件名是插件的名字，文件内容只有一行代码，用来指定当前插件的实现类，在运行这个插件的时候，gradle就可以从这里读取到这些信息。

```properties
implementation-class=com.hello.GreetingPlugin
```

**GreetingPlugin.groovy**

这个文件就是插件的实现类，它的内容如下：

```groovy
package com.hello

import com.hello.GreetingConfigurationExtension
import org.gradle.api.Plugin
import org.gradle.api.Project

class GreetingPlugin implements Plugin<Project> {
    @Override
    void apply(Project project) {
        // write your custom code here
        project.task("greeting") << {
            println "Hello World"
        }
    }
}
```

在`apply`方法中，我们调用gradle的`project`对象，定义了一个名为`greeting`的task，这个task的功能就是输出一句"Hello World"

## 如何本地发布，使用插件

有了上面的这些文件和配置之后，我们的插件就制作得差不多了，接下来让我们运行`gradle uploadArchives`，把插件发布到本地目录，之后在客户端项目中应用这个插件：

** 客户端项目的build.gradle文件 **

```groovy
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
```
然后在命令行里运行下面这个命令，就可以看到我们这个插件的输出结果了：

```sh
$ gradle greeting

"Hello, World"
```

## 如何把插件发布到互联网
