---
title: "将Mac微信聊天记录备份到移动硬盘"
category: Life
#id: 57
date: 2023-8-31 20:42:32
tags: 
  - Mac
  - 微信
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: app  # 展示在时间线列表中
---
释放Mac微信备份占用的存储空间
<!--more-->

## 找到备份文件夹

进入访达，按下`command+shift+i`显示隐藏文件夹，然后点击菜单栏上的「前往」，点击「前往文件夹」。

将下面内容输入进去。记住更改里面的用户名。
```
/Users/这里改为你的用户名/Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/Backup/
```

`command+c`复制里面的32位字符的文件夹，`command+shift+v`移动文件夹到移动硬盘的任意文件夹中。

## 创建软连接

```shell
ln -s 移动硬盘里的32位字符文件夹地址 /Users/用户名/Library/Containers/com.tencent.xinWeChat/Data/Library/Application\ Support/com.tencent.xinWeChat/2.0b4.0.9/Backup
```


## 重新签名微信
退出微信，在终端app中输入下面的内容并回车。

```shell
sudo codesign --sign - --force --deep /Applications/WeChat.app
```

## 更新微信

更新了微信客户端之后需要重复上述操作。