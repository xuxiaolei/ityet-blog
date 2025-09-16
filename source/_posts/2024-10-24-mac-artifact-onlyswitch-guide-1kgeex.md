---
layout: post
title: Mac神器OnlySwitch指南
date: '2024-10-24 19:25:31'
permalink: /post/mac-artifact-onlyswitch-guide-1kgeex.html
tagline: OnlySwitch操作指南
tags:
  - mac
published: true
---

# Mac神器OnlySwitch指南

	OnlySwitch是一款专为macOS设计的状态栏工具，它集成了多种快速切换功能，如暗黑模式、隐藏桌面图标、甚至能够隐藏新款MacBook Pro的屏幕凹槽（notch），并且支持自定义扩展。通过简单的点击或者快捷键操作，即可实现一系列工作环境的即时调整，大大提升了用户的效率和便捷性。

![image](https://ityet.com/img/image-20241024193212-o32kb56.png)  
	基于SwiftUI构建的UI界面，赋予了OnlySwitch现代化的视觉体验和流畅的操作响应。它完美适配macOS Monterey及以上平台，并采用MIT许可协议，鼓励开发者参与贡献和定制。不仅限于此，从版本1.7起集成的快捷方式导入功能到2.0版本的键盘快捷键支持，展现了其技术迭代的速度和对用户体验的极致追求。

![image](https://ityet.com/img/image-20241024193502-5t4vwzs.png)  
OnlySwitch适用于广泛的日常工作与生活场景。对于开发者来说，可以快速调整开发环境；对设计师而言，一键隐藏桌面图标可迅速整理工作空间；而对于所有Mac用户，无论是切换暗黑模式、快速启动屏保还是控制音乐播放，都变得异常简单。更令人兴奋的是，Version 2.5.0后对Apple Widgets的支持，允许用户将这些强大功能直接放在桌面上或通知中心，进一步提升了个性化和即时访问的便利。

​![image](https://ityet.com/img/image-20241024193615-v4legi7.png)​

​![image](https://ityet.com/img/image-20241024193640-jgljxu8.png)

​![image](https://ityet.com/img/image-20241024193735-wiwxqnd.png)​

​![image](https://ityet.com/img/image-20241024193704-k4cjsq3.png)

​![image](https://ityet.com/img/image-20241024193727-kixhboo.png)​​​

# 支持项目功能：

	**一体化开关控制：**提供全方位的系统功能快速切换，包括但不限于隐藏桌面图标、调节暗黑模式等。  
	**高度可定制化：**用户可根据个人习惯增减、排序状态栏上的开关和快捷方式。  
	**快捷键支持：**从2.0版本起，引入键盘快捷键，无需鼠标就能完成切换，为高效人士量身打造。  
	**支持Apple Widgets与Evolution：**自Version 2.5.0起，OnlySwitch与时俱进，整合苹果的小部件功能，让功能扩展更加灵活多样。而Evolution更是允许用户自定义脚本，开启无限可能。  
	**多语言支持：**包括英语、简体中文、德语等多国语言，覆盖全球用户。

‍

# **Evolution**配置：

​![image](https://ityet.com/img/image-20241024194216-a0t2t4w.png)

​![image](https://ityet.com/img/image-20241024194239-srwxr6k.png)​​

以安卓模拟器为例，原本要启动Android Studio的emulator，需要先打开Android Studio然后进到Device Manager界面才能操作启动，非常麻烦，后面试过用Shell脚本启动，但是每次都会弹出终端窗口，比较不友好。现在有了OnlySwitch，就可以通过如下方法定制自己的“快捷开关”

![image](https://ityet.com/img/image-20241024194559-7yqyiax.png)

## 1.开启命令配置

​![image](https://ityet.com/img/image-20241024193829-3gpolrt.png)

```shell
/Users/leo/Library/Android/sdk/emulator/emulator -avd Pixel_3A > /dev/null 2>&1 &
```

## 2.关闭命令配置

​![image](https://ityet.com/img/image-20241024193921-7t0akpk.png)

```shell
/Users/leo/Library/Android/sdk/platform-tools/adb emu kill
```

## 3.检查状态命令配置

![image](https://ityet.com/img/image-20241024192546-wipoidb.png)​

```shell
ps -ef | grep -v grep | grep -q 'Pixel_3A' && echo 1 || echo 0
```

## 4.其它举例
### 4.1.FRP启动
开启命令：
```shell
nohup /Users/edy/Documents/frp/frpc -c /Users/edy/Documents/frp/frpc.toml >> /Users/edy/Documents/frp/frpc.log 2>&1 &
```
注意:配置文件和日志最好指定绝对路径，避免启动失败

关闭命令：
```shell
pkill -9 -f '/Users/edy/Documents/frp/frpc' 2>&1 &
```

检查状态：
```shell
ps -ef | grep -v grep | grep -q '/Users/edy/Documents/frp/frpc' && echo 1 || echo 0
```

### 4.2.启动Nodejs应用
开启命令：
```shell
nohup sh /Users/edy/Documents/scripts/startwhale.sh > /Users/edy/Documents/scripts/startwhale.log 2>&1 &
```

关闭命令：
```shell
pkill -9 -f '/Users/edy/Documents/frp/frpc' 2>&1 &
```

检查状态：
```shell
ps -ef | grep -v grep | grep -q '/usr/local/bin/node /Users/edy/IdeaProjects/whale-cloud-frontend/node_modules/umi/bin/forkedDev.js' && echo 1 || echo 0
```

附：shell启动脚本，因为启动过程复杂所以用shell脚本文件
```shell
#!/bin/bash

# dev.sh – 先 git pull，再 npm run dev
export PATH="/bin:/usr/local/bin:/opt/homebrew/bin"   # 根据你 which 出来的路径调整

set -e  # 任何一步出错立即退出

PROJECT_DIR="/Users/edy/IdeaProjects/whale-cloud-frontend"

echo ">>> 进入项目目录：$PROJECT_DIR"
cd "$PROJECT_DIR"

# echo ">>> 拉取最新代码"
git pull --rebase --autostash

# echo ">>> 安装依赖（如有更新）"
cnpm ci --silent

echo ">>> 启动开发服务器"
npm run dev &
DEV_PID=$!                 # 记录后台进程的 PID
echo $DEV_PID > /Users/edy/Documents/scripts/.whale-cloud.pid   # 可选：写到文件，方便别处 kill

echo ">>> dev server PID=$DEV_PID 已启动"
echo ">>> 想停止时执行：kill $DEV_PID"
```
 
‍
