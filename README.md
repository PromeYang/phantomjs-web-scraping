# 基于phantomjs实现的爬虫
`phantomjs-web-scraping`

## 准备工作

* node 4.x以上版本均可
* [phantomjs 最新版本](http://phantomjs.org/)

## 示例demo

* `app-book.js`文件

```
'use strict';

var fs = require('fs');
var spawn = require('child_process').spawn;

var phantomJsPath = '/data/LRKJ-CMS.com/LRKJ-CMS/src/tasks/phantomjs/phantomjs';
var actionPath = '/data/LRKJ-CMS.com/LRKJ-CMS/src/tasks/phantomjs/book/index.js';
var dataPath = '/data/LRKJ-CMS.com/LRKJ-CMS/src/tasks/phantomjs/book/data.txt';
var bookPath = '/data/LRKJ-CMS.com/LRKJ-CMS/src/tasks/phantomjs/book/book.txt';

var data = {
    bookName: 'wudizhenjimo',
    bookPath: 'http://m.biqukan.com/50/50034/',
    startPath: 'http://m.biqukan.com/50/50034/18427247.html'
};

var autoBook = function (data) {
    fs.writeFileSync(dataPath, JSON.stringify(data));
    if (data.bookName) {
        fs.writeFileSync(bookPath.replace('book.txt', data.bookName + '.txt'), '');
    } else {
        fs.writeFileSync(bookPath, '');
    }
    var free = spawn(phantomJsPath, [actionPath]);
    var info = '';
    // 捕获标准输出并将其打印到控制台
    free.stdout.on('data', function (data) {
        console.log('' + data);
        info += data;
    });
    // 捕获标准错误输出并将其打印到控制台
    free.stderr.on('data', function (data) {
        console.log('' + data);
        info += data;
    });
    // 注册子进程关闭事件
    free.on('exit', function (code, signal) {
        console.log('^o^ ---task finish--- ^o^');
        // console.log('操作信息:\n' + info);
        var next = info.match(/---.*---/)[0].slice(3, -3);
        if (next !== data.bookPath) {
            // autoBook({
            //     bookPath: data.bookPath,
            //     startPath: next
            // });
        }
    });
};

autoBook(data);
```

* `phantomjs` 入口文件 `index.js`

```
console.log('解析小说地址 start ...');
var page = require('webpage').create();
var fs = require('fs');

var bookPath = '/data/LRKJ-CMS.com/LRKJ-CMS/src/tasks/phantomjs/book/book.txt';
var dataPath = '/data/LRKJ-CMS.com/LRKJ-CMS/src/tasks/phantomjs/book/data.txt';
var data = JSON.parse(fs.read(dataPath));
if (data.bookName) {
    bookPath = '/data/LRKJ-CMS.com/LRKJ-CMS/src/tasks/phantomjs/book/' + data.bookName + '.txt';
}

doTask(data.startPath);

function doTask(startPath) {
    console.log('doTask ... ' + startPath);
    page.open(startPath, function (status) {
        console.log('page opened status: ' + status);
        console.log('building ...');

        var result = page.evaluate(function () {
            var result = {};
            var pb_next = document.getElementById('pb_next').href;
            var bookContent = document.getElementById('chaptercontent').innerText;
            bookContent = bookContent.replace(/\(欢迎您来您的支持，就是我最大的动力。\)/g, '');
            bookContent = bookContent.replace(/\{www\.感谢各位书友的支持，您的支持就是我们最大的动力\}/g, '');
            bookContent = bookContent.replace(/福利色色漫画，各种言情小说!你懂的!\(记得自备纸巾\)长按复制 xlmanhua 搜索公众号!/g, '');
            bookContent = bookContent.replace(/笔趣阁阅读网址：m\.biqukan\.com/g, '');
            result.title = document.title;
            result.content = bookContent;
            result.next = pb_next;
            return result;
        });

        console.log(result.title);
        console.log('---' + result.next + '---');

        try {
            fs.write(bookPath, result.content, 'a');
        } catch (err) {
            console.log(err)
        }

        console.log('解析小说地址结果 end ...');
        if (result.next === data.bookPath) {
            page.close();
            phantom.exit();
        } else {
            doTask(result.next);
        }
    });
}
```

## 关于爬虫的思考

* 效率与性能
* 异常处理与报警通知
* 反爬虫对抗

