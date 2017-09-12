# 项目创建过程

#### 全局安装gulpjs

> npm install -g gulp 全局安装
> npm install gulp --save-dev 局部安装 

1. 命令行创建npm的配置文件
  > npm init
2. 添加一个gulp的依赖
  > npm install gulp --save-dev
3. 在项目根目录下添加一个gulpfile.js文件，这个是gulp的主文件，这个文件名是固定的
4. 在gulpfile中抽象我们需要做的任务
5. gulpfile.js内容

### 引入 gulp及组件

```
var gulp = require('gulp'),
    rename = require('gulp-rename'), //文件重命名
    concat = require('gulp-concat'), //链接web服务器
    jshint = require('gulp-jshint'), //js检查
    uglify = require('gulp-uglify'), //js压缩
    connect = require('gulp-connect'), //文件合并
    imagemin = require('gulp-imagemin'), //图片压缩
    minifycss = require('gulp-minify-css'), //压缩css
    rev = require('gulp-rev-append'), //给页面引用添加版本号，清除页面缓存
    minifyhtml = require('gulp-minify-html'), //压缩html
    notify = require('gulp-notify'),           //更动通知
    fileinclude = require('gulp-file-include'),
    processhtml = require('gulp-processhtml'),
    clean = require('gulp-clean'),
    ejs = require('gulp-ejs'),
    sass = require('gulp-sass');

//定义打开的端口
//gulp.task(name[, deps], fn) 定义任务  name：任务名称 deps：依赖任务名称 fn：回调函数       
//gulp.src(globs[, options]) 执行任务处理的文件  globs：处理的文件路径(字符串或者字符串数组) 
//gulp.dest(path[, options]) 处理完后文件生成路径  

//配置开发和发布路径
var path = {
     //开发环境
    src: {
        html: ['./src/*.html'],
        css: ['./src/css/*.css'],
        sass: ['./src/sass/*.scss'],
        js: ['./src/js/*.js'],
        images: ['./src/images/*']
    },
    //发布环境
    dist: {
        html:'./dist/',
        css:'./dist/css/',
        js:'./dist/js/',
        images:'./dist/images/',
    },
     //不被处理的文件得复制
    copy:[
        {from:'./src/audio/*',to:'./dist/audio'},
        {from:['./src/js/*','!./src/js/index.js'],to:'./dist/js'}
    ],
    clear:['./dist/*.html',
           './dist/css/main.css',
           './dist/js/main.js',
           './dist/images'
    ]
}
//定义web服务器
gulp.task('server', function(){
    connect.server({
        root:['./'],
        port: 7003,
        livereload: true
    });
});

//定义web服务模块，增加浏览器同步浏览
gulp.task('browser-sync', function(){
    browserSync({
        server: {
            baseDir: '.'
        }
    });
});

//css压缩
gulp.task('css', function(){
    gulp.src(path.src.sass)
    .pipe(concat('main.scss'))//concat 合并后的文件名
    .pipe(sass()) //执行cass编辑
    .pipe(gulp.dest(path.dist.css)) //保存在dist的地址

    .pipe(minifycss()) //压缩
    .pipe(rename({suffix:'.min'})) //重命名
    .pipe(gulp.dest(path.dist.css)) //重命名后保存在dist地址
    .pipe(connect.reload());
});

//HTML压缩
gulp.task('html', function(){
    gulp.src(path.src.html)
    .pipe(gulp.dest(path.dist.html))
    .pipe(connect.reload());
});

//图片压缩
gulp.task('images', function(){
    gulp.src(path.src.images)
    .pipe(imagemin())
    .pipe(gulp.dest(path.dist.images))
    .pipe(connect.reload());
});

//js压缩
gulp.task('js', function () {
    gulp.src(path.src.js)
    .pipe(jshint())
    .pipe(jshint.reporter())
    .pipe(uglify())
    .pipe(gulp.dest(path.dist.js))
    .pipe(connect.reload())
    .pipe(notify({message:'Javascript task complete'}));
});

//编辑ejs
gulp.task('ejs', function(){
    gulp.src('src/**.ejs')
    .pipe(ejs())
    .pipe(gulp.dest('dist'))
    .pipe(connect.reload());
});

//复制文件
gulp.task('copy', function(){
    for(var i=0,items=path.copy,len=items.length;i<len;i++){
        gulp.src(items[i].from).pipe(gulp.dest(items[i].to))
            .pipe(notify({message:'copy task complete'}));
    }
});

//清空图片、样式、js
gulp.task('clean', function(){
    gulp.src(['./dist/css', './dist/js', './dist/images'],{read: false})
    .pipe(clean())
    .pipe(notify({message: 'Clean task complete'}));
});

gulp.task('fileinclude', function(){
    gulp.src(['src/*.html', '!src/include/*.html'])
    .pipe(fileinclude({
        prefix: '@@',
        basepath: '@file'
    }))
    .pipe(gulp.dest('dist'))
    .pipe(connect.reload());
});

//监听人物 运行语句 gulp watch
gulp.task('watch', function () {
    gulp.watch(path.src.html, function(event){
        gulp.run('html');
    }); //监听html
    gulp.watch(['src/*.ejs'], ['ejs']); 
    gulp.watch(path.src.js, function(event){
        gulp.run('js')
    }); //监听js
    gulp.watch(path.src.sass, function(event){
        gulp.run('css')
    }); //监听css
});

//默认人物 清空图片、样式、js并重建 运行语句 gulp 
gulp.task('default',['clean'],function(){
    gulp.start('server', 'html', 'images', 'fileinclude', 'js', 'css', 'watch', 'ejs')
});
```

6. 安装组件
`
npm install gulp-util gulp-imagemin gulp-ruby-sass gulp-minify-css gulp-jshint gulp-uglify gulp-rename gulp-concat gulp-clean gulp-livereload tiny-lr --save-dev
`

7. 项目目录
```
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
```

8. 执行项目
> 通过cd找到项目路径执行gulp
