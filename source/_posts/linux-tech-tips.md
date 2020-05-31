---
title: Linux发行版不完全客制与使用指南
date: 2018-10-07 21:22:52
tags: [linux,script]
categories: [计算机民科]
---
毛胚房与精装修，你更偏爱哪一种？
<!-- more -->  
&emsp;&emsp;并不是所有发行版都能够开箱即用(常规用途意义上)，甚至都不一定会有GUI或者完整的桌面环境，因为他们并不是为了对标Windows或者MacOS而诞生的。又或者我们会受存储容量、使用习惯等因素影响而产生强烈的意愿自行搭配软件生态。  
&emsp;&emsp;从给服务器镜像附加xfce再硬装进笔电里找驱动，到伪装成Hackintosh扮猪吃老虎的渗透测试平台，当调教过的系统运行在日用机上的时候，那熟悉并且安心的操作手感就足够值回折腾所付出的时间成本了。这前后做了不少实用命令和配置的备忘笔记，谨以这篇文章存档在博客上。

## 外观
#### Wallpaper Engine的Linux替代
安装vlc后，在命令行键入  
`cvlc --video-wallpaper --no-audio /path/to/video.mp4`  
我可以不用，但不能没有

## 操作

## 文件处理  
#### 压缩与解压  
仅以存储级别打包一个文件，在命令行键入  
`zip -r -0 xx.zip xx/`  
对于linux常见的压缩格式:  
压缩格式 | 解压命令  
 :-: | :-:  
.tar.gz | `tar -zxvf xx.tar.gz`  
.tar.bz2 | `tar -jxvf xx.tar.bz2`  
.tar.xz | `xz -d && tar -xf`  

## 安装与卸载
#### 清除所有已删除包的残馀配置文件
在命令行键入  
`dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P`  
适合那些每次以purge方式卸载仍不满足的强迫症患者  

## 网络连接  
#### 蓝牙暴力扫描  
在命令行键入  
`fang -s`  
类似于aircrack-ng扫描WLAN设备，但不是所有的蓝牙硬件都支持长时间高频扫描  
#### 赋予shell全局代理  
在命令行键入  
`export ALL_proxy=http://ip:port`  
并不总是有条件享受透明代理环境的，不是吗  
