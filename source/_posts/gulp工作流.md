title: "gulp工作流"
date: 2015-04-26 16:56:48
tags: gulp
---
## 前言
专职前端一年半了，经历了项目结构的一次一次升级，也逐渐加深了对前端工程化的认识。从最初的jQuery + grunt 的`1.0`版本，到`2.0` 引入 MVVM 框架 avalon，CSS 预处理工具 Less，HTML 模板引擎 Jade 以及前端模块化框架 seajs ，再到 gulp + browserify 和 静态资源的 md5 后缀以加强缓存机制。通过不断的改进项目结构，建立一个完善的多页面多模块的 Web 应用，形成了自己的一套前端工程化流程。

当然这套工作流离不开一个强大的构建工具--gulp
<!-- more -->
## 为什么要用 gulp
之前项目用的是 grunt，为什么要改成 gulp ？随着项目越来越复杂，前端构建脚本会越来越复杂，grunt 这种通过配置的方式会变得越来越难维护，而 gulp 这种基于 stream 的方式，更像是编程，更容易维护。软件开发有条定律 `约定优于配置`，维护 100 行有逻辑的代码和 维护 300 行类似 JSON 那样无逻辑的代码，必然前者占优。同时 gulp 不会频繁的读写文件，效率也会比 grunt 好一些。

当然，现在用些更先进的构建工具，比如 webpack，将前端模块化做的更加彻底，配合 React 很可能是未来的发展方向，不过这里不做介绍。

## My Tasks
gulp 出来也很久了，目前版本 3.X（坑爹的 4.X 都憋了一年多了），官方文档很完善，各种教程也数不胜数，这里我就介绍我在使用 gulp 中形成的工作流，供大家参考。有不对的地方，欢迎指正。

### 构建脚本的目录结构以及插件的加载
由于目前项目构建代码比较多，做了拆分，这里我的构建脚本的目录结构式这样的
```
- build_config
    - config.js
    - my_config.js
- gulpfile.js
```

config.js 中主要是一些路径的配置，插件的加载和工具函数：

```
// 判断是否是部署环境，决定是否压缩代码。
var argv = require('minimist')(process.argv.slice(2));
var RELEASE = !!argv.release;

// 基本的 src 和 dest 路径
var srcPath = 'src',
    destPath = 'dist',
    buid = destPath + '/build',
    deployPath = destPath + '/deploy'
    ...

// 引入插件
var gulp = require('gulp')，
    del: require('del'),
    fs: require('fs'),
    '$': require('gulp-load-plugins')()；
    ...

// 主要是 glup.src 和 gulp.dest 中的参数的路径配置
var config = {
    less: {
        src: srcPath + '/less/*.less',
        dest: buildPath + /css
    }
    ...
}
// 一些辅助函数
function modify(modifier) {
    return through2.obj(function(file, encoding, done) {
        var content = modifier(String(file.contents), file.path);
        file.contents = new Buffer(content);
        this.push(file);
        done();
    });
}
...
```
因为该项目有些针对不同企业有定制构建的部分，我放入了 my_config.js 中，项目的所有 task 都放在 gulpfile.js 中。
当然，我们自己在可以根据实际的项目需要自己判断构建脚本是否需要拆分以及如何拆分（因为我目前的项目构建代码有近千行，太不容易维护了），拆分后各模块只需以 node 的方式 `exports` 出去，并 `require` 进 gulpfile.js 即可。
这里用了一个 [gulp-load-plugins](https://www.npmjs.com/package/gulp-load-plugins)来加载 gulp 插件， 省去了书写的麻烦。

### 一些公共使用的插件

[gulp-changed](https://www.npmjs.com/package/gulp-changed)：用来实现增量构建必不可少的插件，试想如果每次运行 task 时，那些完全没更改的文件还需重新构建一遍，`时间就是金钱，我的朋友`，我们要尽量来减少带薪编译的时间。这里判断文件 是否更改，主要有两张方式，一种是判断文件的最后修改时间(changed.compareLastModifiedTime)，这种方式对一些单独的静态文件非常有用，比如图片，字体等等，可以只需要处理一次，后续完全不用再次处理，极大的节省了时间；另一张是判断文件的 md5 (changed.compareSha1Digest)，这种对 jade，less，js 处理有用，因为这些文件大多数时候需要 include 其他文件，最后生成一个入口文件，因而不能单纯的判断文件修改时间，需要判断文件的 md5，不过这种方式还是需要编译，只是节省了写文件的时间，实际效果不明显。当然官方还建议了一些插件[gulp-cached](https://github.com/wearefractal/gulp-cached)，[gulp-remember](https://github.com/ahaurw01/gulp-remember)和[gulp-newer](https://github.com/tschaub/gulp-newer)，来实现 incremental builds ，读者可以自行研究，并加入到自己的 task 中去。

[gulp-if](https://github.com/robrich/gulp-if)：来实现 stream 的三元操作，比如我本地开发的时候，需要调试，不需要压缩代码，而上线时需要压缩就能通过这个插件来实现。

[gulp-plumber](https://www.npmjs.com/package/gulp-plumber)：gulp 任务出错时会跳出当前任务，这在开发过程中 watch 文件变动时，比较坑爹。因此这个插件能替代系统的 onerror 事件，不退出当前任务。4.X 版本会加强 gulp 的错误处理机制，很是期待。

[run-sequence](https://github.com/OverZealous/run-sequence)，解决任务依赖关系的神器，由于 gulp 自身的任务依赖关系无法解决顺序执行的问题，在复杂的关系依赖时，显得力不从心。gulp 4.X 也着力去解决这个问题。

## 主要的 task
### CSS 处理
项目使用了 Less 做 CSS 预处理，需要使用 [gulp-less](https://github.com/plus3network/gulp-less) 插件，同时可以使用 [autoprefixer](https://github.com/postcss/autoprefixer) 来自动增加浏览器前缀，[gulp-minify-css](https://github.com/jonathanepollack/gulp-minify-css) 来压缩并清理CSS，需要注意的是这个插件默认处理IE9及以上的浏览器，需要指定IE兼容版本，否则写的兼容低版本的 IE hack 会被清理掉。
这里 autoprefixer 和 sourcemaps 同时用有些问题，暂时在开发和部署环境下分开使用。
```
gulp.task('less', function() {
    return gulp.src(config.less.src)
        .pipe($.plumber())
        .pipe($.if(!RELEASE, $.sourcemaps.init()))
        .pipe($.less())
        .on('error', console.error.bind(console))
        .pipe($.if(RELEASE, $.autoprefixer({browsers: AUTOPREFIXER_BROWSERS})))
        .pipe($.if(RELEASE, $.minifyCss({compatibility: 'ie8'})))
        .pipe($.if(!RELEASE, $.sourcemaps.write()))
        .pipe($.changed(config.less.dest, {hasChanged: $.changed.compareSha1Digest}))
        .pipe(gulp.dest(config.less.dest))
        .pipe($.if(!RELEASE, $.livereload()));
});
```

### image 压缩
使用 [gulp-imagemin](https://github.com/sindresorhus/gulp-imagemin) 来完成 png，jpg，gif的压缩。
```
gulp.task('imgmin', function() {
    return gulp.src([config.imgmin.src])
        .pipe($.changed(config.imgmin.dest))
        .pipe($.imagemin())
        .pipe(gulp.dest(config.imgmin.dest));
});
```

### HTML 处理
项目使用 Jade 来实现 HTML 的复用。主要 [gulp-jade](https://github.com/phated/gulp-jade) 插件来完成，
```
gulp.task('jade', function() {
    gulp.src(config.jade[name].src)
        .pipe($.jade())
        .pipe(gulp.dest(config.jade[name].dest))
        .pipe(!RESEASE, $.livereload());
});
```

### JavaScript 处理
这块是最复杂的地方，之前项目是使用的 seajs 做 JS 的模块化加载，但是其实所有 JS 在构建时已经做了 bundle 操作，只有一个语言文件是异步加载进的，对比 browserify, seajs 对我们失去了其本来的意义，而且其生态并不好，只有阿里前端在维护，我们自己还写了一个 gulp-seajs 来满足我们自己的构建需要。因此考虑了下换成了 browserify，使用 commonjs 方式来进行模块化的开发。

由于 browserify 并没有像 seajs 或者 requirejs 一样，提供一个还好的异步解决方案，因此，我们需要把根据环境，将不同语言文件写到 script 标签中，供入口文件使用。

```
gulp.task('locale', function() {
    var browserified = through2.obj(function(file, encoding, done) {
        var b = browserify();
        if (!RELEASE) {
            b = watchify(b);
        }
        b.require(file.path, {expose: 'locale'});
        b.bundle(function(err, res) {
            if (err) {
                return done(err);
            }
            file.contents = res;
            done(null, file);
        });
        b.on('update', function() {
            var filename = path.basename(file.path);
            var destPath = path.dirname(file.path)
            console.log(file.path + ' changed!!!');
            b.bundle()
                .pipe(source(filename))
                .pipe(buffer())
                .pipe(gulp.dest(destPath))
        })
    });
    return gulp.src(config.locale.src)
        .pipe(browserified)
        .pipe($.if(RELEASE, $.uglify()))
        .pipe(gulp.dest(config.locale.dest))
});
```
这里我通过 throungh2 模块来实现 [stream](https://github.com/rvagg/through2) 的内容更改，通过 [watchify](https://github.com/substack/watchify) 和 `update `事件来在开发环境下监听文件改动，并重新 bundle。`var source = require('vinyl-source-stream')` 和 `var buffer =  require('vinyl-buffer')` 可以将用 browserify bundle后生成的stream，转换成gulp需要的 stream。
这样就可以根据需要，引入不同的语言文件，主文件只需要 `var locale = require('locale')` 即可。其他异步模块可以以相同方式处理。

```
gulp.task('bundle', function() {
    var browserified = through2.obj(function(file, encoding, done) {
        var b = browserify({entries: file.path, debug: !productEnv});
        b.external('locale');
        if (!RELEASE) {
            b = watchify(b);
        }
        b.bundle(function(err, res) {
            if (err) {
                return done(err);
            }
            file.contents = res;
            done(null, file);
        })
        b.on('update', function() {
            var filename = path.basename(file.path);
            var destPath = path.dirname(file.path)
            console.log(file.path + ' changed!!!');
            b.bundle()
                .on('error', $.util.log.bind($.util, 'Browserify Error'))
                .pipe(source(filename))
                .pipe(buffer())
                .pipe($.if(!RELEASE, $.sourcemaps.init({loadMaps: true})))
                .pipe($.if(!RELEASE, $.sourcemaps.write('./')))
                .pipe(gulp.dest(destPath))
        })
    });

    return gulp.src(config.bundle.src, {base: basePath.src})
        .pipe($.plumber())
        .pipe(browserified)
        .pipe($.if(!RELEASE, $.sourcemaps.init({loadMaps: true})))
        .pipe($.if(RELEASE, $.uglify()))
        .pipe($.if(!RELEASE, $.sourcemaps.write('./')))
        .pipe(gulp.dest(config.bundle.dest))
});
```

这里使用差不多同样的方法，需要注意的是这里的 `b.external('locale');` 把 `locale` 模块排除掉，这里可以把两个任务公共的部分提取出来，这里为了看得直观代码啰嗦了下。

后面还有一些其他的 task，比如 `gul-jshint` --检查代码质量， 利用 `gulp-livereload` --监听文件的改动刷新当前页面， `gulp-jsonminify` 压缩 JSON 文件等等。可以去参考下文档，比较简单。

----
至此我们已经完成了，前端所需的所有代码的构建任务，但实际工作中，还有一个很重要的问题，如何消除静态文件的缓存。

由于 Web 项目迭代周期短，需要频繁上线，必然涉及到静态资源的缓存机制。最开始我们的项目采用了，在静态资源的加入构建时的时间戳来解决，不过这样造成每次构建都要把所有资源都更新一下。最好的解决办法是将静态资源命名加入文件的 md5 值， 云龙在知乎上关于这个问题有一个很精彩的[回答](http://www.zhihu.com/question/20790576/answer/32602154)。

用 [gulp-rev-all](https://github.com/smysnk/gulp-rev-all) 就可以完成我们所需的功能。
```
gulp.task('rev-all', ['clean:deploy', 'default'], function() {
    var revAll = new $.revAll({
        dontRenameFile: [/^\/favicon\..*$/, 'robots.txt', 'sitemap.xml', '.html', '.json'],
        dontUpdateReference: ['.html'],
        fileNameManifest: 'manifest.json',
    });
    var tasks = [], task;
    return gulp.src([basePath.build + '/**/*', '!' + basePath.build + '/libs/component/ueditor/**/*', '!' + basePath.build + '/**/*.map'])
        .pipe(revAll.revision())
        .pipe(gulp.dest(basePath.deploy))
        .pipe(revAll.manifestFile())
        .pipe(gulp.dest(basePath.deploy));
});
```
这样就可以生成需要的带有 md5 值得文件，需要注意的几个配置选项，排除一些不需要 md5 的文件，如果是公共的库的话，生成的 manifest.json 可以供其他项目调用时，进行替换。

## 其他有用的资源
Recipes: [https://github.com/gulpjs/gulp/tree/master/docs/recipes#recipes](https://github.com/gulpjs/gulp/tree/master/docs/recipes#recipes)
gulp-cheatsheet(非常赞的资源): [https://github.com/osscafe/gulp-cheatsheet](https://github.com/osscafe/gulp-cheatsheet)