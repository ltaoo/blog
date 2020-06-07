---
title: EPERM:operation not permitted
categories: 工具
tags:
- gulp
- nodejs
---

公司使用 svn 作为版本控制，但是在使用 gulp 后，gulp 执行过程中却总是出现`EPERM:operation not permitted`错误。
这篇笔记“初步”解决了这个问题，解决方法涉及到 gulp 的任务执行顺序、node 执行 shell 命令 两个知识点。

<!--more-->
## 问题描述
svn 会在提交文件后，将提交的文件修改为只读，如果这时 gulp 仍然在运行并监听了文件，就会执行相应任务（因为发生了文件修改），重点是， gulp 生成的文件，和源文件（src下的文件）有相同的读写属性。

所以，在将文件提交至 SVN 后，gulp 从 src 编译文件到 dist 的文件就会是“只读”属性。

OK，那问题就出现了，这时再去修改文件，同时 gulp 自动编译，但 gulp 在重新生成修改的这个文件到 dist 目录时，会提示没有权限而导致 gulp 任务中断。下面以代码来说明：

```javascript
const gulp = require('gulp')
const sass = require('gulp-sass')

gulp.task('compile-sass', function () {
    return gulp.src('src/*.scss')
        .pipe(sass())
        .pipe(gulp.dest('dist'))
})
gulp.watch('src/*.scss', ['compile-sass'])
```

代码用来编译 scss 文件成 css 文件。在命令行执行`gulp compile-sass`后，将会生成`dist`文件。

OK，现在仍一切正常。但这时将文件 src 下的 scss 文件提交至 SVN，提交成功后，svn 将刚刚提交的 scss 文件修改为`只读`，gulp 监听到 scss 文件发生改变，执行`compile-sass`任务，再次生成并覆盖原来的`dist`目录。

但是，由于这时 src 下的 scss 文件是`只读`的，生成的`css`也是`只读`的。我们再次编辑刚刚提交的 scss 文件，在`sublime`下会显示对话框，
```
File is write-protected

Overwrite write-protected file
```
这也证明了文件的确是变成了只读。我们点击对话框的确定，可以将当前正在修改的 scss 文件取消只读属性，所以我们可以正常修改文件内容，但是，gulp 却没有一个这样的过程，gulp 在重新生成新的 css 文件时，需要覆盖（修改？）原先的 css 文件，但是原先的文件是只读的，导致了无法成功覆盖，并提示
```bash
Error: EPERM: operation not permitted, open 'fileName'
```

> 百度或者 google 都没有找到解决办法。可能是很少有公司会使用 svn 的同时用 gulp？ 毕竟用 gulp 了也会用 git 吧？ 总之需要自己解决了。

## 解决方法
解决办法肯定是在 gulp 编译前将 src 的只读属性取消掉。

### 使用 fs 模块
```javascript
const fs = require('fs')
const path = require('path')
function readDir(dir){
  fs.stat(dir, function (err, stat) {
    if(err) {
      return
    }
    if(stat.isDirectory()) {
      fs.readdir(dir, function (error, files) {
          if(error) return;
          files.forEach(function (file) {
            readDir(path.join(dir, file))
          })
      })
    }else {
      fs.chmod(dir, 0666)
    }
  })
}
 
readDir(path.join(__dirname, 'dist'))
```


定义了一个`readDir`函数，因为需要遍历文件夹，所以是一个递归函数。

使用`fs.stat`来读取一个文件或文件夹，返回`stat`保存了文件或文件夹信息（只是信息）。然后判断是文件夹还是文件，如果是文件夹，就读取文件夹下的子文件与子文件夹，并再调用一次`readDir`函数。

如果不是文件夹，就是文件了，那就使用`fs.chmod`改变文件的权限为可读写。

### 调用 shell 命令

最简单的就是`chmod -R 777 dist`了，但是不能在windows 的 cmd 里执行。

cmd 中用来改变只读属性的是`attrib`
```bash
attrib -r dist/* /s /d
```

- `-r`表示清除只读属性
- `dist/*`指定要修改 dist 下的所有文件及文件夹
- `/s`表示处理子文件夹中的匹配文件
- `/d`表示也处理当前文件夹

当然了，要在 gulp 任务中执行，所以就要用到`child_process`模块了
```javascript
const exec = require('child_process').exec
exec('attrib -r dist/* /s /d')
```

所以最后写在`gulpfile.js`中的代码会是这样的：
```javascript
const gulp = require('gulp')
const sass = require('gulp-sass')
//
const exec = require('child_process').exec
//
const clean = require('gulp-clean')

gulp.task('compile-sass', function () {
    return gulp.src('src/*.scss')
        .pipe(sass())
        .pipe(gulp.dest('dist'))
})
gulp.watch('src/*.scss', ['compile-sass'])
//
gulp.task('cancel-readonly', function (cb) {
  exec('attrib -r src/* /s /d')
  cb(null)
})
// clean dist
gulp.task('clean-dist', functoin () {
  return gulp.src('./dist')
    .pipe(clean())
})
// default
gulp.task('default', ['clean-dist', 'cancel-readonly', 'compile-sass'])
```
预想的是，先执行`clean-dist`清除 dist 文件夹，然后改变 src 文件夹下所有文件的只读属性，最后编译 scss 到 dist 文件夹。

理想很丰满，显示很骨感。由于 gulp 暂时不支持？配置任务执行顺序，所以这三个任务是一起执行的，所以导致了还没有改变只读属性时，就开始编译 scss 到 css 导致错误。

所以要进行优化。官方教程提到的方法没有实现。。

最后使用了`gulp-sequence`插件。
```javascript
const sequence = require('gulp-sequence')
gulp.task('default', sequence('clean-dist', 'cancel-readonly', 'compile-sass'))
```
这样就是按照写的顺序执行了。。

## 总结
不使用 svn 应该不会遇到这种情况，所以这篇笔记并没有多大的参考价值。

不过意识到查询资料的时候，关键词太重要了。开始时查询“命令行修改文件权限”，第一次找到`chmod`，结果同事告诉我并没有用，才发现是因为自己使用的`git`命令行。然后搜索“windows修改文件权限”，找到了`cacls`和`icacls`但都不行。。。

最后以“windows取消只读属性”作为关键词，终于找到了有用的。


## 参考

- [gulp顺序执行任务](http://zhangruojun.com/gulpshun-xu-zhi-xing-ren-wu/)

