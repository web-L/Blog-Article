# oppo 开放平台海外项目，添加支持 ES6 填坑总结

## 一、简介
用过 `ES6` 的帅哥美女都知道，`ES6` 新增了很多特性，语法上也有很多骚气的操作 总之用了之后就回不去了，刚好有一个海外的项目若是能添加 `ES6`支持在开发过程能够提供很大的便利，什么？ 还没听说过？ 那还不赶紧上车 [《ECMAScript 6 入门-阮一峰》](http://es6.ruanyifeng.com/)。

可惜现在现在大部分的浏览器都还没支持 `ES6`，所以我们需要用到 [BABEL](https://www.babeljs.cn/) 去编译成现在浏览器能解析的 js。

## 二、解析 海外项目 gulp 构建流程

1. 由于是项目的基于 `gulp+Jquery+Es2015` 搭建的，那就要在原来的 `gulp` 构建上添加编译 `ES6` ，又不能影响旧代码。
2. 在项目的根目录 `gulpfile.js` 配置文件 `build`（正式环境构建） `localbuild`（开发环境构建） 这两个任务

    ```
        gulp.task('localbuild', function (callback) {
            runSequence('browserify', 'destJs', 'useref', ['copyjquery', 'miniJsLocal', 'miniCssLocal'], ['miniHtmlLocal', 'images', 'pimgs', 'test_sign', 'fonts'], callback
            );
        });
    ```

    通过 `gulp localbuild` 执行，`runSequence` 这个方法用来 **串行的执行拆分的任务** 。
3. `browserify` 任务是将 static/modules 下 Js 输出到 src/dist/script/src 文件内
4. `destJs`  任务将src/dist/script/src 文件内的 Js 重命名，然后输出到 src/dist/script 目录
5. `useref` 找到 page 下面所有 html 内的 样式和Javascrip 通过下面 语法进行 查找 页面上引用 `js css` 合并，包含 `html css js`输出到 后端的 `views` 目录下。

    ```
    <!--build:js /static/scripts/theme_list.ES6.js -->
    <script src="../../dist/scripts/theme_list.js" type="text/javascript" charset="utf-8"></script>
    <!-- endbuild -->
    ```
6. `['copyjquery', 'miniJsLocal', 'miniCssLocal']`  第一个是将页面上引用的 插件复制到 `web/static/scripts` 后端目录下，第二个压缩打包 JS 加MD5 等（当然在开发环境没有这些处理），第三个压缩打包 CSS 加MD5，后面的任务也是一样的操作只是处理文件不一样，最终都会输出到 `web/static` 后端目录（这个目录才是存放静态资源的目录 也就是页面访问的需要加载的 `css js img` 等等）。

## 三、Babel 搭建编译 ES6
1. 通过上面了解这个项目的构建流程，等 `Js` 合并完之后再编译后到压缩，所以我们在 `destJs` 这个任务添加 `ES6` 的编。
2. 安装对应依赖模块（推荐使用淘宝境像 [cnpm](http://npm.taobao.org/) ）

    ```
    npm install --save-dev gulp-babel @babel/core @babel/preset-env
    ```
3. 在根目录添加 `.babelrc` Babel 的配置文件 内容如下：

    ```
    {
        "presets": ["@babel/preset-env"]
    }
    ```
4. 在 `gulpfile.js` 在头部引入 `var babel = require('gulp-babel')` ，找到`useref` 这个任务改写成 如下：

    ```
    gulp.task('useref', function () {
        var assets = useref({}, lazypipe().pipe(sourcemaps.init, {
            loadMaps: true
        }));
        return gulp.src(srcRoot + 'pages/**/*.html')
            .pipe(useref())
            .pipe(babel()))
            .pipe(gulp.dest(distRoot + 'views'));
    });
    ```
5. 然后在命令行执行 `gulp localbuild` 等好久 。。。。。。。。然后会出现一堆报错 :joy: ，原因是 `Babel` 编译是按照 `Javascrip` 严格模式来编译的，旧的代码里非常多有很多不是按照 严格模式 来书写的，不了解可以看一下 [JS-严格模式和非严格模式的区别](https://www.jianshu.com/p/39e295f4526d)，所有怎么办呢？ 思考中。。。。。。。。
6. 旧的代码没有 `ES6` 语法所以我过滤掉就好了呀，那需要用 `ES6` 昨整？  先安装一个模块：
   
   ```
   npm install --save-dev gulp-if
   ```
   完了之后在头部引入： `var gulpif = require('gulp-if')` 将     `useref` 任务改成如下：

    ```
    gulp.task('useref', function () {
        var assets = useref({}, lazypipe().pipe(sourcemaps.init, {
            loadMaps: true
        }));
        return gulp.src(srcRoot + 'pages/**/*.html')
            .pipe(useref())
            .pipe(
                gulpif(function (file) {
                    return new RegExp('.ES6.js', 'g').test(file.path);
                }, babel()
            ))
            .pipe(gulp.dest(distRoot + 'views'));
    });
    ```
    **意思是需要用 ES6 的语法的话只需要在对应的 html 文件里找到合并 Js 语法加上 `.ES6`** 例子如下：
    ```
    <!--build:js /static/scripts/theme_list.js -->
    改成即可
    <!--build:js /static/scripts/theme_list.ES6.js -->
    ```
## 四、完结
上面的配置我都已经搭建完了，你们更新下载只需要删除 目录下的 `node_modules` 重新 `cnpm install` 后面需要用 `ES6` 编写在对应 的 html 里加上 `.ES6`，**再次声明使用 ES6 语法编写需要注意编译是按照严格语法，去编译的如果代码书写不严谨会导致构建报错。**

## Babel 使用注意

babel 默认只转换语法,而不转换新的API，如需使用新的API,还需要使用对应的转换插件或者 [polyfill.js](https://cdn.bootcss.com/babel-polyfill/7.0.0-rc.4/polyfill.min.js)

    例如，默认情况下babel可以将箭头函数，class等语法转换为ES5兼容的形式，
    
    但是却不能转换Map，Set，Promise等新的全局对象，这时候就需要使用polyfill去模拟这些新特性
