---
title: gulp压缩脚本
tags: [gulp]
---
自己手动写了一个压缩脚本，主要实现了对比压缩，没有改变的文件不会压缩大大缩小了压缩时间
<!--more-->
![图片](http://7xr7dg.com1.z0.glb.clouddn.com/hexoblog/images/gulp4.0.png)
```bash
var gulp = require("gulp"),
    del = require('del'),
    rev = require("gulp-rev"),
    revReplace = require("gulp-rev-replace"),
    uglify = require("gulp-uglify"),
    minifyCss = require('gulp-minify-css'),
    concat = require('gulp-concat'),
    replace = require('gulp-replace-task'),
    htmlmin = require('gulp-htmlmin'),
    runSequence = require('run-sequence'),
    imagemin = require('gulp-imagemin'),
    pngquant = require('imagemin-pngquant'),
    gulpif = require('gulp-if'),
    fs = require('fs');

var path = ".build"
var opt = {
    backpath: path + "/back",
    zippath: path + "/zip",
    revpath: path + "/rev"
}

opt.ziprevpath = opt.zippath + "/rev"
///////////////////////////////////压缩////////////////////////////////////////////////////////
gulp.task("delrev", function(cb) {
    del('rev', {
        force: true
    }).then(function() {
        cb();
    });
})
gulp.task("ziporiginal", function() {
    return gulp.src([opt.zippath + "/**", opt.zippath + "/*.*"])
        .pipe(gulp.dest(""))
})

gulp.task("copyrev", function() {
    return gulp.src(opt.revpath + "/rev.json")
        .pipe(gulp.dest(opt.ziprevpath))
})

gulp.task("uglifyall", function(cb) {
    //如果有旧版本文件，就开始对比
    if (zipbackdir_rev_status) {

        console.log("开始对比");
        //得出对比后有的差异
        var obj = revCompared(opt.revpath + "/rev.json", opt.ziprevpath + "/rev.json")

        if (!obj.arr.length) {
            console.log("没有文件改变")
        }

        if (obj.del.arr.length) {
            for (var i = 0; i < obj.del.arr.length; i++) {

                //先检查一下，zipback里面是否还有这个文件，如果没有则跳过
                if (!fs.existsSync(opt.zippath + "/" + obj.del.arr[i])) {
                    continue;
                }

                //走到这里说明有删除文件，开始走删除文件方法
                deldir(obj.del.arr[i])

                function deldir(url) {

                    //解析文件路径，从第一级开始比较，如果，第一级或是第二级。。。新文件里没有了。而老文件里有，说明要删除
                    urlarr = url.split("/")

                    var str = urlarr[0];
                    for (var i = 0; i < urlarr.length; i++) {
                        //得出文件夹转台
                        var dir = fs.existsSync(str)
                            //得出备份文件夹状态
                        var zip_dir = fs.existsSync(opt.zippath + "/" + str)
                            //如果新文件夹里没有。旧文件里有，说明是删除，则执行删除文件
                        if (!dir && zip_dir) {

                            console.log("del：" + str)
                            del.sync(opt.zippath + "/" + str, {
                                force: true
                            })
                        }
                        //走到这一步说明新文件夹里有，则进入下一级目录
                        str += "/" + urlarr[i + 1]
                    }
                }
                //如果删除文件后是空文件夹，应该把备份里的空文件夹也删除
            }
        }
    } else {
        //没有旧版本文件，就只读取新文件，然后执行
        console.log("全量压缩")
        var obj = revCompared(opt.revpath + "/rev.json")
    }
    console.log(obj.arr)
    return gulp.src(obj.arr, {
            base: "./"
        })
        .pipe(gulpif(["*.js", "**/*.js", "!js/config**"], uglify()))
        .pipe(gulpif("*.css", minifyCss()))
        .pipe(gulpif(["*.html"], htmlmin({ collapseWhitespace: true })))
        .pipe(gulpif(["*.tpl"],htmlmin({collapseWhitespace: true})))
        .pipe(gulpif("*.jpg", imagemin({
            progressive: true,
            svgoPlugins: [{
                removeViewBox: false
            }],
            use: [pngquant()]
        })))
        .pipe(gulp.dest(opt.zippath));

})

//以上开始压缩js
//以下都是为了得出文件版本号

gulp.task("main_production", function() {
        return gulp.src(['js/main_production.js',opt.revpath+'/rev.json'])
            .pipe(concat("main.js"))
            //.pipe(uglify())
            .pipe(gulp.dest(opt.zippath+"/js"))
    })
gulp.task("test", function() {
        //return gulp.src(["js/templates/base/wrap.tpl"])
        //return gulp.src(["js/templates/base/header.tpl"])
        return gulp.src(["js/templates/base/menu.tpl"])
            .pipe(uglify())
            .pipe(gulp.dest(opt.zippath + "htmlmin"))
    })
    //以上开始压缩js
    //以下都是为了得出文件版本号
    // gulp.task("concatconfig", function() {
    //     return gulp.src(["js/config.production.js", opt.revpath + "/rev.json"])
    //         .pipe(concat("config.js"))
    //         .pipe(gulp.dest(opt.zippath + "/js"))
    // })
gulp.task("rev", function() {
        return gulp.src(excludefile.concat('!js/main_production.js','!js/main.js'))
            .pipe(rev())
            .pipe(rev.manifest("rev.json"))
            .pipe(replace({ //替换字符串
                patterns: [{
                    match: /['|"](.{1,})['|"].{1,}['|"].*-{1,}?(.[^.]*).{1,}['|"]/g,
                    replacement: '"$1":"$1?v=$2"'
                }]
            }))
            .pipe(gulp.dest(opt.revpath))
    })
    //图片，字体，样式都完成之后，开始替换js，再生成js文件版本号（注意这里只是生成js文件里js文件版本号，）
gulp.task("js", function() {
        var rev_static = gulp.src(".rev/rev-static.json");
        var rev_style = gulp.src(".rev/rev-style.json");
        var rev_tpl = gulp.src(".rev/rev-tpl.json");
        return gulp.src(["*/**/*.*js", "*/*.*js"])
            .pipe(revReplace({
                manifest: rev_static
            }))
            .pipe(revReplace({
                manifest: rev_style
            }))
            .pipe(revReplace({
                manifest: rev_tpl
            }))
            .pipe(gulp.dest(""))
            .pipe(rev())
            .pipe(rev.manifest("rev-js.json"))
            .pipe(replace({ //替换字符串
                patterns: [{
                    match: /['|"](.{1,})['|"].{1,}['|"].*-{1,}?(.[^.]*).{1,}['|"]/g,
                    replacement: '"$1":"$1?v=$2"'
                }]
            }))
            .pipe(gulp.dest(opt.revpath))
    })
    //图片，字体，样式都完成之后，开始替换js，再生成js文件版本号（注意这里只是生成js文件里js文件版本号，）
gulp.task("tpl", function() {
        var rev_static = gulp.src(".rev/rev-static.json");
        var rev_style = gulp.src(".rev/rev-style.json");
        return gulp.src(["*/**/*.*tpl", "*/*.*tpl"])
            .pipe(revReplace({
                manifest: rev_static
            }))
            .pipe(revReplace({
                manifest: rev_style
            }))
            .pipe(gulp.dest(""))
            .pipe(rev())
            .pipe(rev.manifest("rev-tpl.json"))
            .pipe(replace({ //替换字符串
                patterns: [{
                    match: /['|"](.{1,})['|"].{1,}['|"].*-{1,}?(.[^.]*).{1,}['|"]/g,
                    replacement: '"$1":"$1?v=$2"'
                }]
            }))
            .pipe(gulp.dest(opt.revpath))
    })
    //替换css里在的图片路径，并生成CSS版本号文件rev-style.json
gulp.task("css", function() {
        var rev_static = gulp.src(opt.revpath + "/rev-static.json");
        return gulp.src(['*/**/*.*css', '*/*.*css'])
            .pipe(revReplace({
                manifest: rev_static
            }))
            .pipe(gulp.dest(""))
            .pipe(rev())
            .pipe(rev.manifest("rev-style.json"))
            .pipe(replace({ //替换字符串
                patterns: [{
                    match: /['|"](.{1,})['|"].{1,}['|"].*-{1,}?(.[^.]*).{1,}['|"]/g,
                    replacement: '"$1":"$1?v=$2"'
                }]
            }))
            .pipe(gulp.dest(opt.revpath))
    })
    //生成所有静态资源版本号文件
gulp.task("static", function() {
        return gulp.src(["*/**/*.!(*css|*js|*tpl)", "*/*.!(*css|*js|*tpl)"])
            .pipe(rev())
            .pipe(rev.manifest("rev-static.json"))
            .pipe(replace({ //替换字符串
                patterns: [{
                    match: /['|"](.{1,})['|"].{1,}['|"].*-{1,}?(.[^.]*).{1,}['|"]/g,
                    replacement: '"$1":"$1?v=$2"'
                }]
            }))
            .pipe(gulp.dest(opt.revpath))
    })
    //压缩之前先把所有的文件备份一遍
gulp.task("copy", function() {
    return gulp.src(excludefile)
        .pipe(gulp.dest(opt.backpath))
})

gulp.task('zip', function(cb) {
    runSequence('copy', 'static', 'css', 'tpl', 'js', 'rev','main_production', 'uglifyall', "copyrev",  'ziporiginal','delrev', cb);
});

function revCompared(newfile, oldfile) {
    //文件对比。对比两个json版本文件。
    //如果新文件里有，旧文件里没有，说明是新加
    //如果旧文件里有，新文件里没有，说明是删除
    //如果新文件里有，旧文件里也有，但不一样，说明文件内容有改变
    //以上三种情况都会被添加到返回的对象中


    //读取新版本文件
    var rev = fs.readFileSync(newfile);
    //新文件转化为json
    var json_rev = JSON.parse(rev);
    //返回的对象,存放所有对比结果
    var obj = {};
    //存放新加、改动文件的对象
    obj.arr = []
    obj.json = {}
        //存放新增文件的对象
    obj.add = {}
    obj.add.arr = []
    obj.add.json = {}
        //存放删除文件的对象
    obj.del = {}
    obj.del.arr = []
    obj.del.json = {}
        //只存放改动文件的对象
    obj.change = {}
    obj.change.arr = []
    obj.change.json = {}

    //看是否有老版本文件
    if (oldfile) {
        //读取老版本文件
        var ziprev = fs.readFileSync(oldfile)
            //老版本文件转为json
        var json_ziprev = JSON.parse(ziprev)

        //先循环新版本文件，跟老版本文件进行文件版本对比
        for (a in json_rev) {
            //如果老文件里面没有这个文件，说明是新增文件
            if (!json_ziprev[a]) {
                obj.add.arr.push(a)
                obj.add.json[a] = json_rev[a]

                obj.arr.push(a)
                obj.json[a] = json_rev[a]
                    //如果老文件有，但版本号不一样，说明内容有变动
            } else if (json_rev[a] != json_ziprev[a]) {
                obj.change.arr.push(a);
                obj.change.json[a] = json_rev[a];

                obj.arr.push(a);
                obj.json[a] = json_rev[a];
            }
        };
        (function() {
            //循环旧版本文件
            for (a in json_ziprev) {
                //如果旧版本文件里有，新版本文件里没有则是删除
                if (!json_rev[a]) {
                    //将改动的文件放入删除对象中
                    obj.del.arr.push(a)
                    obj.del.json[a] = json_ziprev[a]
                }
            }
        })();
        return obj
    } else {
        //如果没有老版本文件，循环新版本文件。放入到对象中
        for (a in json_rev) {
            obj.arr.push(a)
            obj.json[a] = json_rev[a]
        }
        return obj
    }

}

/////////////////////////////解压///////////////////////////////////////////////////////
//要还原之前，首先把原来的代码进行备份
gulp.task("zipback", function() {
        return gulp.src(excludefile.concat("!rev/**", "!rev*"))
            .pipe(gulp.dest(opt.zippath))
    })
    //压缩代码备份完之后，开始还原代码,这不是还原，这是覆盖，应该删除之前的所有文件夹，再还原。。。待改
gulp.task("recovery", function() {
    return gulp.src(opt.backpath + "/**")
        .pipe(gulp.dest(""))
})

gulp.task("delback", function(cb) {
    del([opt.backpath], {
        force: true
    }).then(function() {
        cb();
    })
})
gulp.task('unzip', function(cb) {
    runSequence('zipback', 'recovery', 'delback', cb);
});
////////////////////////////////////////////////////////////////////////

var gulpbackdir_status = fs.existsSync(opt.backpath)
var zipbackdir_rev_status = fs.existsSync(opt.ziprevpath + "/rev.json");
//跑脚本时忽略文件
var excludefile = ["**",'!node_modules/**','!package.json', "!gulpfile.js", "!" + path + "/**", "!" + path + "**"]
var build;
if (gulpbackdir_status) {
    build = ["unzip"]
} else {
    build = ["zip"]
}
//入口
gulp.task("default", build, function(cb) {
    cb()
});
```