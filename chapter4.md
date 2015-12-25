# demo总结

&emsp;&emsp;前面章节从各个角度介绍了fmpp的使用以及与gulp的集成， 下面按照搭建过程， 整理使用步骤， 以java web工程为例，假设工程目录如下：

![](http://haitao.nos.netease.com/73b5414292ce45e196221777c4deaef7.jpg)

1. 在webapp目录下，新建package.json文件， 执行`npm install`
```
{
  "name": "fmpp_online",
  "version": "1.0.0",
  "devDependencies": {
        "gulp": "^3.9.0",
        "gulp-clean": "^0.3.1",
        "gulp-connect": "^2.2.0",
        "gulp-ext-replace": "^0.2.0",
        "gulp-freemarker": "^0.1.2",
        "gulp-replace": "^0.5.4",
        "gulp-sourcemaps": "^1.6.0",
        "gulp-watch": "^4.3.5"
  }
}
```

2. 新建gulpfile.js， 配置输出目录为dist

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


3. 在webapp目录下， 尽力mock目录， 如下图， 其中的结构建议与`WEB-INF/ftl`模板目录下的结构保持一致， 在mock目录下新建一个index.json文件， 如下， 在`WEB-INF/ftl`下新建tpl对应的test.ftl, test.ftl内容包含`${word!'hello'}`获取数据；
```
{
    "tpl":"test.ftl",
    "data":{
        "word":"World"
    }
}
```

4. 执行`gulp server`， 由于配置了dist和webapp两个目录为虚拟根目录， 所以打开浏览器以`/xxx.html`( xxx为dist目录下对应的文件路径)访问即可看到解析后的ftl文件；