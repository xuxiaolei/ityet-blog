---
layout: post
title: Mac电脑ADB使用
date: '2024-10-23 15:31:31'
permalink: /post/mac-computer-adb-use-saqho.html
published: true
---

# Mac电脑ADB使用

---

在Android开发中，ADB（Android Debug Bridge）是一个非常重要的工具，它提供了开发者与Android设备之间进行通信的多种方式。安装ADB对于任何进行Android开发的人来说都是必不可少的，尤其是在Mac电脑上进行开发时。

### 1. 安装ADB

在Mac上安装ADB，推荐使用Homebrew：

```bash
$ brew install --cask android-platform-tools
```

### 2. **设备管理**

ADB允许开发者查看连接到计算机的Android设备列表。使用以下命令可以查看连接的设备：

```bash
$ adb devices
```

这个命令显示所有已连接设备的序列号和设备状态（比如是否已授权该电脑进行调试）。

> gamin@MBPmailto:gamin@MBP call-sign % adb devices
>
> List of devices attached
>
> KE6201 device
>
> emulator-5554 device

### 3. 无线安装调试

若只需要连接数据线进行有线安装调试，可以忽略这一步。

使用ADB通过IP地址远程安装APK到设备是一种常见的操作，特别适用于无线调试场景。

#### 确保设备和计算机在同一网络

确保您的Android设备和运行ADB的计算机连接到同一个网络（例如，同一个Wi-Fi网络）。

#### 在设备上启用开发者选项和USB调试

* 在设备上，前往“设置” -> “系统” -> “关于手机”。
* 多次点击“构建号”直到提示你已成为开发者。
* 返回到“设置”菜单，打开“开发者选项”。
* 启用“USB调试”。

#### 连接设备并启用通过WiFi的ADB调试

* 使用USB线将Android设备连接到计算机。
* 打开命令行或终端，输入以下命令来确认设备已正确连接：

```bash
$ adb devices
```

* 确认设备连接后，执行以下命令以启用通过WiFi的ADB调试：

```bash
$ adb tcpip 5555
```

#### 断开USB连接并连接至设备的IP地址

* 在设备上查找其IP地址：前往“设置” -> “网络和互联网” -> “Wi-Fi” -> 点击已连接的网络，查看“IP地址”。
* 断开设备的USB连接。
* 使用以下命令连接到设备的IP地址：

```bash
$ adb connect [设备IP地址]:5555
```

#### 安装APK

确保你有APK文件的路径。使用以下命令安装APK：

```bash
$ adb install /path/to/your/app.apk
```

或者

> $ adb devices  
> List of devices attached  
> 192.168.110.96:5555    device  
> emulator-5554    device
>
> $ adb -s 192.168.110.96:5555 install /Users/gamin/Documents/bb.apk

### 4. **安装和卸载应用**

ADB可以直接从命令行安装、更新和卸载Android应用。这对于测试和部署应用非常方便。

#### 安装应用

```bash
$ adb install path/to/your_app.apk
```

如果有多个设备或模拟器，直接用上面的指令安装会报错。需要用下面的方式，指定设备安装 APK：

```bash
$ adb -s KE6201 install path/to/your_app.apk
```

#### 卸载应用

```bash
$ adb uninstall com.example.yourapp
```

如果有多个设备或模拟器，需要使用下面的方式指定设备卸载：

```bash
$ adb -s KE6201 uninstall com.example.yourapp
```

### 5. **调试应用**

ADB允许开发者查看连接设备的日志输出，这对于调试应用非常重要。logcat 是ADB的一个功能，用于查看实时日志：

```bash
$ adb logcat
```

这一功能可以帮助开发者实时监控应用运行时的各种信息，包括错误和系统信息。

### 6. **文件传输**

ADB提供了文件传输功能，允许开发者向设备推送文件或从设备拉取文件。例如，将文件推送到设备：

```bash
$ adb push local_file_path /remote_file_path
```

从设备拉取文件到本地：

```bash
$ adb pull /remote_file_path local_file_path
```

### 7. **执行Shell命令**

ADB允许开发者在Android设备上执行Shell命令，这对于直接操作设备的文件系统、启动服务或进行系统管理等任务非常有用：

```bash
$ adb shell
```

进入shell后，你可以执行各种**Linux**命令。

### 8. **屏幕录制和截图**

ADB可以用来捕获屏幕截图和录制用户操作过程。这对于开发文档、Bug报告或用户支持非常有帮助。

截图命令：

```bash
$ adb shell screencap -p /path/to/save.png
```

屏幕录制：

```bash
$ adb shell screenrecord /path/to/save.mp4
```

### 9. **端口转发和网络管理**

ADB允许将设备上的某个端口转发到开发机上，这对于进行网络测试和调试客户端-服务器应用非常重要：

```bash
$ adb forward tcp:local_port tcp:remote_port
```

### 10. ADB配对

有些手机ADB连接之前需要配对。比如小米，可以使用pair命令配对，会提示输入密码，填写密码后再

再用 adb connect 连接就行：

```bash
$ adb pair tcp:local_port
```

### 10. 授予投影媒体权限

```bash
$ adb shell appops set org.autojs.autojs6 PROJECT_MEDIA allow
```

‍
