#项目创建过程
全局安装gulpjs
`npm install -g gulp 全局安装` 
`npm install gulp --save-dev 局部安装`

1. 命令行创建npm的配置文件
  a. npm init
2. 添加一个gulp的依赖
  npm install gulp --save-dev
3. 在项目根目录下添加一个gulpfile.js文件，这个是gulp的主文件，这个文件名是固定的
4. 在gulpfile中抽象我们需要做的任务
5. gulpfile.js内容
`// 引入 gulp及组件
var gulp    = require('gulp'),                 //基础库
    imagemin = require('gulp-imagemin'),       //图片压缩
    sass = require('gulp-ruby-sass'),          //sass
    minifycss = require('gulp-minify-css'),    //css压缩
    jshint = require('gulp-jshint'),           //js检查
    uglify  = require('gulp-uglify'),          //js压缩
    rename = require('gulp-rename'),           //重命名
    concat  = require('gulp-concat'),          //合并文件
    clean = require('gulp-clean'),             //清空文件夹
    tinylr = require('tiny-lr'),               //livereload
    server = tinylr(),
    port = 35729,
    livereload = require('gulp-livereload');   //livereload

// HTML处理
gulp.task('html', function() {
    var htmlSrc = './src/*.html',
        htmlDst = './dist/';

    gulp.src(htmlSrc)
        .pipe(livereload(server))
        .pipe(gulp.dest(htmlDst))
});

// 样式处理
gulp.task('css', function () {
    var cssSrc = './src/scss/*.scss',
        cssDst = './dist/css';

    gulp.src(cssSrc)
        .pipe(sass({ style: 'expanded'}))
        .pipe(gulp.dest(cssDst))
        .pipe(rename({ suffix: '.min' }))
        .pipe(minifycss())
        .pipe(livereload(server))
        .pipe(gulp.dest(cssDst));
});

// 图片处理
gulp.task('images', function(){
    var imgSrc = './src/images/**/*',
        imgDst = './dist/images';
    gulp.src(imgSrc)
        .pipe(imagemin())
        .pipe(livereload(server))
        .pipe(gulp.dest(imgDst));
})

// js处理
gulp.task('js', function () {
    var jsSrc = './src/js/*.js',
        jsDst ='./dist/js';

    gulp.src(jsSrc)
        .pipe(jshint('.jshintrc'))
        .pipe(jshint.reporter('default'))
        .pipe(concat('main.js'))
        .pipe(gulp.dest(jsDst))
        .pipe(rename({ suffix: '.min' }))
        .pipe(uglify())
        .pipe(livereload(server))
        .pipe(gulp.dest(jsDst));
});

// 清空图片、样式、js
gulp.task('clean', function() {
    gulp.src(['./dist/css', './dist/js', './dist/images'], {read: false})
        .pipe(clean());
});

// 默认任务 清空图片、样式、js并重建 运行语句 gulp
gulp.task('default', ['clean'], function(){
    gulp.start('html','css','images','js');
});

// 监听任务 运行语句 gulp watch
gulp.task('watch',function(){

    server.listen(port, function(err){
        if (err) {
            return console.log(err);
        }

        // 监听html
        gulp.watch('./src/*.html', function(event){
            gulp.run('html');
        })

        // 监听css
        gulp.watch('./src/scss/*.scss', function(){
            gulp.run('css');
        });

        // 监听images
        gulp.watch('./src/images/**/*', function(){
            gulp.run('images');
        });

        // 监听js
        gulp.watch('./src/js/*.js', function(){
            gulp.run('js');
        });

    });
});`

6. 安装组件
`npm install gulp-util gulp-imagemin gulp-ruby-sass gulp-minify-css gulp-jshint gulp-uglify gulp-rename gulp-concat gulp-clean gulp-livereload tiny-lr --save-dev`

7. 项目目录
project(项目名称)
|–.git 通过git管理项目会生成这个文件夹
|–node_modules 组件目录
|–dist 发布环境
    |–css 样式文件(style.css style.min.css)
    |–images 图片文件(压缩图片)
    |–js js文件(main.js main.min.js)
    |–index.html 静态文件(压缩html)
|–src 生产环境
    |–sass sass文件
    |–images 图片文件
    |–js js文件
    |–index.html 静态文件
|–.jshintrc jshint配置文件
|–gulpfile.js gulp任务文件

8.通过npm安装http-server，快速建立http服务
`npm install http-server -g`

9、通过cd找到发布环境目录dist
-运行http-server，默认端口是8080
-访问路径localhost:8080
-再打开一个cmd，通过cd找到项目路径执行gulp，清空发布环境并初始化
-执行监控 gulp
