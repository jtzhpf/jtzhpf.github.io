---
title: "icns 图标制作"
category: CS&Maths
#id: 57
date: 2023-8-31 20:42:32
tags: 
  - app
#toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
#timeline: article  # 展示在时间线列表中
---

想要用 [pake](https://github.com/tw93/Pake) 来将常用的网页打包为 app，例如 [clash dashboard](http://clash.metacubex.one) 等，方便服务器的使用，需要制作 icns 格式的图标。
<!--more-->

- 准备一个 1024 * 1024 的png图片，假设名字为 `pic.png`
- 创建一个临时目录存放不同大小的图片
```shell
mkdir tmp.iconset
```
- 把原图片转为不同大小的图片，并放入上面的临时目录
```shell
# 全部拷贝到命令行回车执行，执行结束之后去tmp.iconset查看十张图片是否生成好
sips -z 16 16     pic.png --out tmp.iconset/icon_16x16.png
sips -z 32 32     pic.png --out tmp.iconset/icon_16x16@2x.png
sips -z 32 32     pic.png --out tmp.iconset/icon_32x32.png
sips -z 64 64     pic.png --out tmp.iconset/icon_32x32@2x.png
sips -z 128 128   pic.png --out tmp.iconset/icon_128x128.png
sips -z 256 256   pic.png --out tmp.iconset/icon_128x128@2x.png
sips -z 256 256   pic.png --out tmp.iconset/icon_256x256.png
sips -z 512 512   pic.png --out tmp.iconset/icon_256x256@2x.png
sips -z 512 512   pic.png --out tmp.iconset/icon_512x512.png
sips -z 1024 1024   pic.png --out tmp.iconset/icon_512x512@2x.png
```
- 通过`iconutil`生成 icns 文件，此时目录下即可得到想要的图标
```shell
 iconutil -c icns tmp.iconset -o Icon.icns
 ```