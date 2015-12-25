# FMPP配置

&emsp;&emsp;我们最终想实现的是要将fmpp与前端构建工具gulp集成，所以我们可以在这里将fmpp看作是一个命令行工具，供在构建任务中调用运行；由于它不是原生的gulp任务编写的，所以它的配置无法通过gulp配置，需要使用一个专门的配置文件， ， 虽然配置文件的名称和路径可以任意指定，但是为了可以在gulp中方便配置， 统一将配置文件放于与执行fmpp的gulpfile.js同级，并且命名为config.fmpp, 因为如果配置文件名为config.fmpp或者fmpp.cfg时，执行时无需特殊指定；


&emsp;&emsp;主要配置如下：
```
sourceRoot: src                 // ftl目录
sources: index.ftl              // 需要编译为html的文件, 如果没有此项配置, 那么sourceRoot下的所有ftl都将被编译
outputRoot: dist                // 输出目录
logFile: log.fmpp               // 日志打印目录, 可查看出错信息
modes: [execute(*.ftl)]         // 对sourceRoot下的ftl文件进行操作
replaceExtensions:[ftl,html]    // 编译后后缀改为html
data:tdd(../mock/index.tdd)   // 数据文件, 路径默认相对于sourceRoot

```

* sourceRoot

    fmpp要处理文件的根目录，所有要处理的文件必须位于此目录下，并且ftl模板内的根目录就对应此虚拟目录；

* outputRoot
    
    sourceRoot对应的输出目录就是outputRoot，fmpp将解析后的文件输出至此目录；

* sources

    需要编译为html的文件, 如果没有此项配置, 那么sourceRoot下的所有ftl都将被编译；

* data

    data就是存放假数据的地方，还有一个属性是dataRoot，默认情况下dataRoot就是sourceRoot，推荐的做法是，在sourceRoot下创建一个子目录， 用于存放假数据文件，这些假数据是无需fmpp解析的， 所以在子目录下新建一个名为ignoredir.fmpp的文件， 就可以过滤此目录；数据格式默认是tdd格式，一种类似json的格式；

* modes

    一共有4中mode，mode就是fmpp提供了一些功能函数，如`excute`,`copy`,`renderXml`,`ignore`； 根据字面意思就可以理解copy和ignore的作用，copy就是不处理文件， 直接复制，ignore则是要忽略的文件，excute标识要处理的文件pattern;

* replaceExtensions

    修改后缀的方法，类似的还有removeExtensions，removePostfixes等；可参考[Output file name deduction](http://fmpp.sourceforge.net/settings.html#sect9)