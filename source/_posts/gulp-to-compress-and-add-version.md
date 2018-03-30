---
title: Gulp 构建前端
date: 2018-03-30 14:41:54
tags:
 - gulp
categories: Frontend
lede: "通过 gulp 来构建前端代码，压缩以及版本号管理"
thumbnail: /img/gulp.png
---

[gulp](https://gulpjs.com/) is a toolkit that helps you automate painful or time-consuming tasks in your development workflow.

本文以 [spring-boot](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/) 项目为例，首先来看一下项目的目录结构  

```bash  
/project
|-- src/main/java/{.java}
|-- src/main/resources/
|           |-------- application.properties {配置文件}
|           |-------- src/ {静态文件 js/css 原始文件}
|                     |-- components/
|                     |-- css/
|                     |-- js/
|                     |-- templates/
|           |-------- gulpfile.js
|           |-------- bower.json {前端依赖管理}
|           |-------- static {静态文件 js/css 编译后的文件}
|           |-------- templates {html 编译后文件}
|-- pom.xml
|-- README.md
```
`spring-boot` 项目所有前端文件都放置在 `resource` 目录， 当我们操作使用 gulp 的过程中，都先 cd 到 resources 作为工作环境。  

首先，使用 npm 初始化 package.json 文件（`npm init`），并添加以下依赖并安装

```json  
{
  "scripts": {
    "dev": "gulp watch",
    "build": "gulp --production"
  },
  "dependencies": {
    "del": "^3.0.0",
    "gulp": "^3.9.1",
    "gulp-babel": "^7.0.0",
    "gulp-clean-css": "^3.9.0",
    "gulp-imagemin": "^3.4.0",
    "gulp-jshint": "^2.0.4",
    "gulp-notify": "^3.0.0",
    "gulp-rename": "^1.2.2",
    "gulp-rev": "^8.1.0",
    "gulp-rev-replace": "^0.4.3",
    "gulp-uglify": "^3.0.0",
    "gulp-util": "^3.0.8",
    "jshint": "^2.9.5",
    "run-sequence": "^2.2.0"
  },
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-preset-es2015": "^6.24.1"
  }
}
```

gulp 运行默认依赖 `gulpfile.js` 文件，下面我们直接拿该文件来描述  

```js  
var gulp = require('gulp');
var uglify = require('gulp-uglify');
var del = require('del');
var rename = require('gulp-rename');
var jshint = require('gulp-jshint');
var rev = require('gulp-rev');
var revreplace = require('gulp-rev-replace');
var runSequence = require('run-sequence');
var cleanCSS = require('gulp-clean-css');
var imagemin = require('gulp-imagemin');
var notify = require('gulp-notify');
var util = require('gulp-util');
const babel = require('gulp-babel');

var config = {
    production: !!util.env.production
};

gulp.task('default', ['clean'], function() {
  runSequence('commons', ['scripts', 'styles', 'images'], 'replace');
});

gulp.task('commons', function() {
    gulp.start('copy_common', 'copy_common_wx', 'copy_common_ico');
});

gulp.task('copy_common', function() {
  return gulp.src('src/components/**/*')
    .pipe(gulp.dest('static/components/'));
});

gulp.task('copy_common_wx', function() {
  return gulp.src('src/*.txt')
    .pipe(gulp.dest('static/'));
});

gulp.task('copy_common_ico', function() {
  return gulp.src('src/*.ico')
    .pipe(gulp.dest('static/'));
});

gulp.task('scripts', function() {
  return gulp.src('src/js/**/*.js') // 这个目录对应你未压缩之前的目录
    .pipe(babel({
      presets: ['es2015']
    }))
    .pipe(jshint.reporter('default'))
    .pipe(rename({suffix: '.min'}))
    .pipe(config.production ? uglify({mangle: false}) : util.noop())
    .pipe(rev())
    .pipe(gulp.dest('static/js/'))  // 这个是你压缩完代码输出目录
    .pipe(rev.manifest('rev-manifest-js.json'))
    .pipe(gulp.dest('./'))
    .pipe(notify({ message: 'JS 处理完毕' }));
});

gulp.task('styles', function() {
  return gulp.src('src/css/**/*.css', { style: 'expanded' }) // 这个目录对应你未压缩之前的目录
    .pipe(jshint.reporter('default'))
    .pipe(rename({suffix: '.min'}))
    .pipe(cleanCSS({compatibility: 'ie8'}))
    .pipe(rev())
    .pipe(gulp.dest('static/css/'))  // 这个是你压缩完代码输出目录
    .pipe(rev.manifest('rev-manifest-css.json'))
    .pipe(gulp.dest('./'))
    .pipe(notify({ message: 'CSS 处理完毕' }));
});

gulp.task('images', function() {
  return gulp.src('src/img/**/*')
    .pipe(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true }))
    .pipe(gulp.dest('static/img/'))
    .pipe(rev())
    .pipe(gulp.dest('static/img/'))
    .pipe(rev.manifest('rev-manifest-img.json'))
    .pipe(gulp.dest(''))
    .pipe(notify({ message: 'IMG 处理完毕' }));
});

gulp.task('replace', function(){
  return gulp.src('src/templates/**/*')
    .pipe(revreplace({replaceInExtensions: ['.html'], manifest: gulp.src('rev-manifest-*.json')}))
    .pipe(gulp.dest('templates/'))
    .pipe(notify({ message: 'HTML 处理完毕' }));
});

gulp.task('clean', function() {
  return del(['static/**/*', 'rev-manifest-*.json', 'templates/**/*']);
});

gulp.task('clean-scripts', function() {
    return del('static/js/**/*');
});

gulp.task('clean-styles', function() {
    return del('static/css/**/*');
});

gulp.task('clean-templates', function() {
    return del('templates/**/*');
});

gulp.task('watch', ['default'], function() {

    gulp.watch('src/components/**/*', ['copy_common']);

    gulp.watch('src/css/**/*.css', function() {
        runSequence('clean-styles', 'clean-templates', 'styles', 'replace');
    });

    gulp.watch('src/js/**/*.js', function() {
        runSequence('clean-scripts','clean-templates','scripts', 'replace');
    });

    gulp.watch('src/img/**/*', function() {
        runSequence('clean-templates','images', 'replace');
    });

    gulp.watch('src/templates/**/*', ['replace']);
});
```

下面，我们可以通过 npm run 来运行 `package.json` 里面我们预先定义好的 script. npm run dev 或者 npm run build. 这里面的区别在于，本地环境我们为了测试方便，是不需要压缩的，项目上线，可使用 build 压缩打包。
