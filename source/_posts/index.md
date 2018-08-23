---
title: 开发随手记
date: 2018-08-23
categories:
- 开发经验
tags:
- 视屏
- 音频
- css
- webpack
---

星云链来的请点这里 ➡ [星云链DApp《Article》](https://weyos.github.io/article/index.html#/)

-----

# 开发随手记

## 视屏自动播放方案

[jsmepg](http://jsmpeg.com/)

## 音频加载及自动播放方案

[soundjs](http://www.createjs.cc/soundjs)

## 手动控制失焦

```js
if(document.activeElement.nodeName === 'TEXTAREA' || document.activeElement.nodeName === 'INPUT') {
  document.activeElement.blur();
}
```

## 弹性布局

| 属性 | 说明 | 资料 |
| -- | -- | -- |
| box | 2012年之前的规范 | [查看详情](http://www.zhangxinxu.com/wordpress/?p=1338) |
| flex | 新规范，iOS8.0不兼容 | [查看详情](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html) |

1. flex

>1. 只设置`flex-grow: 1;`是无法保证盒模型按比例划分的，还需要设置原始长度`flex-basis:0%;`。

## css小坑

1. 父亲容器如果设置了`overflow:hidden;`,并且子元素是`position:fixed;`，在pc上子元素会全部展示，但在mobile上子元素超出父级那部分会被隐藏
2. `-webkit-overflow-scrolling: touch;`该属性在iOS上极大的提升了滚动效果，但是会带来很多莫名其妙的BUG。（暂未有很好的解决方案）

>1. 在vue中会出现部分元素渲染不出来，甚至是白屏。
>2. iOS下`::-webkit-scrollbar {display:none}`将会失效。

## webpack项目相对路径解决方案

webpack配置使用几乎都是绝对路径，对相对路径不是很友好，如果需要配置使用相对路径需：

1. 配置`webpack.config`

```js
output: {
  path: config.build.assetsRoot,
  filename: '[name].js',
  publicPath: './'
},
```

或，无需配置`publicPath`

```js
output: {
  path: config.build.assetsRoot,
  filename: '[name].js',
},
```

2. 使用node处理build之后的css中的部分静态文件，如字体文件

```js
const path = require('path');
const fs = require('fs');

exports.replaceFontsPath = (cb) => {
  const root = path.join(__dirname, '../');
  fs.readdir(path.join(root, 'dist/static/css'), (err, files) => {
    if (err) throw err;
    files.forEach((item) => {
      fs.readFile(path.join(root, 'dist/static/css', item), 'utf8', (errRead, data) => {
        if (errRead) throw errRead;
        if(!data.match(/static\/fonts/g)) {
          cb('Fonts Path, No need to modify');
          return;
        }
        let str = data.replace(/static\/fonts/g, '../fonts');
        fs.writeFile(path.join(root, 'dist/static/css', item), str, 'utf8', (errWrite) => {
          if (errWrite) throw errWrite;
          cb('Font Path, modify success')
        });
      });
    });
  });
}
```

## 关于babel

includes等新增方法，需要引入polyfill才能兼容，iOS 8.2不支持

其他解决方案待更新。。。
