# 集成gulp

## 需解决的问题

&emsp;&emsp;网络上基本都是以一个页面为例进行介绍， 实际工程中, 往往有很多模板文件，每次针对单一页面修改配置的方法容易造成文件冲突，并且每次修改也比较麻烦；通过网络的查找，发现了[freemarker.js](https://github.com/ijse/freemarker.js)这个工具，它是对fmpp进行了一层封装，像前面章节所提到的，fmpp提供了命令行的接口方便集成其他工具，freemarker.js就是使用了fmpp这个特性，在freemarker.js的代码包中可以看到如下的引用和实现：

![freemarkerjs代码包](http://haitao.nos.netease.com/4913dd6e115c453d9a6aa5e3a752c4a0.jpg)

&emsp;&emsp;从上图可以看到， freemarker.js依赖于fmpp.jar和freemarker.jar文件；freemarker.js内的实现很简单， 就是通过nodejs的exec方法执行fmpp命令；fmpp命令提供的可选参数与第一章节介绍的fmpp配置的参数一致，freemarker.js主要用到的是如下的命令格式：
```
fmpp sourcefile -C configfile
```
sourcefile就是模板文件， -C标识配置文件的位置， 如果当前目录下同时包含名为fmpp.config的文件， 则fmpp会对配置做一次merge， merge的规则是根据属性的类型决定的， 如果类型不是map或者sequence， 那么具有更高优先级的命令行参数会替代配置文件内的相同属性，如果是map/sequence，那么会将二者合并使用；详情可参考[The merging of setting values](http://fmpp.sourceforge.net/settings.html)；

![freemarkerjs源码](http://haitao.nos.netease.com/fab79db26ce44f60b8599fecb4f53991.jpg)

&emsp;&emsp;在了解过freemaker.js之后，可以发现它仅仅负责了fmpp的调用，体现不出与mock数据的关联，如果近使用它，那么还需要自行实现mock数据与ftl文件的一对一的映射关系，为每个ftl文件生成一个配置文件，文件中指定对应的mock数据与其他信息，对于一个工程的整个ftl目录， 要深度遍历每个文件，执行上述过程，虽然这个实现并不复杂， 但是当前已有现成的工具[gulp-freemarker](https://www.npmjs.com/package/gulp-freemarker)替我们完成了这个工作， 先要从它的配置说起，如下：
![](http://haitao.nos.netease.com/97b412b0030044588288a797dee5ea28.jpg)

&emsp;&emsp;首先是输入, gulp.src指定了mock文件所在的位置， `./mock/**/*.json`可以指定mock目录下的所有json文件；接下来对这些输入文件，执行freemarker命令，只需配置一下viewRoot（ftl文件所在目录）即可；那么这么多json文件是如何与viewRoot下的ftl文件对应的呢？在图片下方位置的地方可以看到一个json文件示例，文件内需指定`tpl`属性，和`data`属性，这个`tpl`就是链接二者的桥梁，此json文件仅对`tpl`指定的ftl模板文件生效； `data`属性则无需多说，就是mock数据， json格式；

&emsp;&emsp;如果你使用的是官方的`freemarker.jar`包， 利用上面的工具， 已经可以解决最初提出的问题； 但是在我们工程实际使用过程中， 还发现工程中使用的`freemarker_netease.jar`对官方的包进行了功能扩充和修改， 其中在工程中常见的就是`?no_encode`这个buildin；原生freemarker不具有这个buildIn，那么这个NoEncodeBI(buildin)到底有什么功能， 下面是对此问题的调研：

以如下代码段为例：
```
...(表示省略部分)
<#escape x as x?html>
...
<#assign timeList = [{"test":"abc'123'abc"}] />

<#noescape>
<script>
    var timeList = ${stringify(timeList![])?no_encode};
</script>
</#noescape>
...
</#escape>
```
1. 首先遇到这个问题， 想到了有没有什么其他原生的buildin可以替换?no_encode，经过一番尝试无果，如果不加?no_encode， 输出的结果都会变成：
```
var timeList = [{&quot;test&quot;:&quot;abc\&#39;123\&#39;abc&quot;}];
```

&emsp;&emsp;单引号和双引号都被encode了， 难道freemarker默认会对输出的变量encode，或者noescape对script内的变量无效？通过研究freemarker.jar的源码， 找到了`NoEncodeBuiltIn`这个类，发现了在`BuildIn`类中列出了全部的buildin，其中包含如下代码段：
```
builtins.put("string", new BuiltIn.stringBI());
builtins.put("substring", new substringBI());
builtins.put("time", new BuiltIn.dateBI(1));
builtins.put("trim", new BuiltIn.trimBI());
builtins.put("no_encode", new NoEncodeBuiltIn());
```
&emsp;&emsp;最后一个就是我们要找得no_encoe，  对应NoEncodeBuildIn这个类， 这个类里面只有一个`calculateResult`方法， 通过与其他buildin对比， 发现这个类直接将输入的字符串， 原样返回，没有做任何处理； 到这里仍然说不通为什么加了?no_encode就可以不encode；
```
TemplateModel calculateResult(String s, Environment env) throws TemplateException {
    return new SimpleScalar(s);
}
```
&emsp;&emsp;那么对比一下在freemarker_netease.jar中的其他buildin，发现基本上大部分都有类似如下的处理， 而不是直接返回new SimpleScalar(s)；看到SimpleScalar你可能会怀疑是不是在这里做了手脚，SimpleScalar只是负责存储value，提供一个getAsString方法，getAsString方法也是直接返回value， 没有做任何处理；
```
static class htmlBI extends StringBuiltIn {
    htmlBI() {}

    TemplateModel calculateResult(String s, Environment env) {
        return new SimpleScalar(StringUtil.HTMLEnc(s));
    }
}
```
&emsp;&emsp;通过对比后，发现了其他buildin都会执行各种encode(s)才返回，这就解释了为什么在最开始尝试找其他buildin替换no_encode是行不通的；那么不加任何buildin， 直接`${stringify(timelist)}`是否可行？通过实验， 发现仍然会被encode，这样就变得清晰一些， 难道默认的freemarker就会对输出的值进行encode？在另外一个类`DollarVariable`(${}操作)中， 找到了答案：
```
private boolean needEncode() {
    return this.escapedExpression instanceof NoEncodeBuiltIn?false:(this.escapedExpression instanceof htmlBI?false:(this.escapedExpression instanceof urlBI?false:(this.escapedExpression instanceof xhtmlBI?false:(this.escapedExpression instanceof xmlBI?false:!(this.escapedExpression instanceof rtfBI)))));
}
```
&emsp;&emsp;这段代码很长， 但是只要看第一个instanceof就够了， 看到了熟悉的`NoEncodeBuildIn`， 如果是`NoEncodeBuildIn`那么就会return false,表示无需encode；如果仔细看， 后面还有很多种情况return false, 那是为什么呢？ 难道除了?no_encode， 还有其他方法可以不encode? 仔细看一下这些条件， 发现如果要满足后续的条件， 则执行到此步骤之前就会被encode，例如上面的htmlBI代码段的返回值； 无需这下就可以得出结论， 网易的freemarker_netease.jar将默认的`${}`操作加上了encode， 所以要不想encode， 必须加上`?no_encode`;为了确定结论的正确， 将工程的freemarker_netease.jar替换为原生的freemarker.jar， 直接输出下面的代码， 无需?no_encode， 就可以得到和上面加入?no_encode同样的效果；这也是为什么我们老的页面， 没有在最外层加入<#escape x as html>的原因；
```
<#noescape>
<script>
    var timeList = ${stringify(timeList![])};
</script>
</#noescape>
```




&emsp;&emsp;在实际工程中，通常会将公用的代码块提取为模块， 供每个文件include， 这样每个子页面都需要这些公用模块的mock数据，在每个json文件重复的使用这些变量，总有些麻烦， 如果注释不好， 也无法区分哪些是公用的变量， 哪些才是对应页面真正需要的值；那么联想到上面提到的fmpp提供的配置merge方法， 能解决此问题么？ 待补充......

## 完整的workflow
&emsp;&emsp;有了上面两个工具， 已经实现了与gulp的集成，这里要补充的是与gulp其他工具的集成，也就是一个完整的gulp文件， 如下， 每次执行gulp server， 就可以保证持续流畅的开发了；

```
var gulp = require('gulp'),
    clean = require('gulp-clean'),
    connect = require('gulp-connect'),
    watch = require('gulp-watch'),
    sourcemaps = require('gulp-sourcemaps'),
    freemarker = require('gulp-freemarker');

var ext_replace = require('gulp-ext-replace');
var path = require('path');

// 需要配置项
var PathConfig = {
    livereloadSrc: ['./javascript/*.js', './css/*.css', './dist/*.html'], // 自动刷新监听文件/目录
}

// 静态服务器
// 并开启自动刷新
gulp.task('webserver', function() {
    connect.server({
        root: ['./', './dist'],
        livereload: true
    })
})

// 通知服务器何时进行自动刷新
gulp.task('livereload', function() {
    gulp.src(PathConfig.livereloadSrc)
        .pipe(watch(PathConfig.livereloadSrc))
        .pipe(connect.reload())
})

gulp.task('clean', function() {
    gulp.src('dist', {read: false})
        .pipe(clean({force: true}));
});

gulp.task('watchFmpp', function() {
    gulp.watch(PathConfig.livereloadSrc, ['ftl']);
});

gulp.task('ftl',['clean'], function() {
    gulp.src("./mock/**/*.json")
        .pipe(freemarker({
            viewRoot: __dirname + '/WEB-INF/ftl/',
            options: {}
        }))
        .pipe(ext_replace('.html'))
        .pipe(gulp.dest("./dist"));
});

gulp.task('server', [ 'webserver', 'livereload', 'watchFmpp']);
```
