[gitbook](http://nupthale.github.io/fmppdoc/)

# 背景

&emsp;&emsp;我们一般的开发过程是， 写好ftl模板，在tomcat的web环境下模拟所需要的数据，将其输出为html文件；这种工作流需要一个java的web环境模拟数据，解析freemarker模板，由于前端对java的环境不熟悉，使得web环境没办法被很好的利用起来； 而且页面的异步请求都需要借助于额外的工具来处理，工具的不统一也降低了开发效率；

&emsp;&emsp;要展示一个页面，需要满足两个条件：1.web运行环境，2. 可以解析ftl模板的工具； 如果第二点可以分离独立出来运行， 那么就可以很容易的将这个过程与现有前端构建工具集成；fmpp就是这样的工具；

&emsp;&emsp;![开发流程](http://haitao.nos.netease.com/4957f8ba10de486f993cc5970cc2bff6.png)
&emsp;&emsp;![freemarker](http://haitao.nos.netease.com/083569bfd55a4d23b49835076724e7b4.png)

# FMPP介绍

&emsp;&emsp;它是一个文本解析工具，不仅限于解析freemarker，由于它是基于java实现的，所以可以跨平台使用，并且它还提供了UN*X-style command-line interface（也就是说它可以在终端直接通过命令运行）， 使得它可以任何构建工具集成。FMPP is a general-purpose text file preprocessing tool that uses FreeMarker templates. It process entire directories recursively. It can be used for generating complete static websites, source code, configuration files, etc. It can insert data from sources like CSV, XML, and JSON into the generated files. 我们主要把他当做ftl模板解析工具， 所以更多的配置可以自行参考[fmpp官网](http://fmpp.sourceforge.net/)