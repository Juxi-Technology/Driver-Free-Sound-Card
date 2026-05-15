# USB-Driverless-Sound-Card-Tutorial

## USB免驱声卡教程

## **可视化测试软件（Windows）**
命令汇总（可跳过）
- 更新系统并安装工具：
  - 执行：`sudo apt update && sudo apt full-upgrade`
  - 安装 ALSA：`sudo apt install alsa-base alsa-utils`
- 识别硬件：
  - 列出音频设备：`aplay -l`
  - 查看 PCI/USB 音频设备：`lspci | grep -i audio、lsusb`
- 基础配置与验证：
  - 运行配置向导：`sudo alsaconf`（若可用）
  - 调节音量：`alsamixer`（按 M 取消静音，方向键调音量，ESC 退出）
  - 保存设置：`sudo alsactl store`
  - 播放测试：测试音频输出（确保扬声器 / 耳机已连接）：


```dart
# 播放测试音，-D 指定 USB 声卡设备（X 为 aplay -l 显示的 card 编号）

speaker-test -c 2 -D plughw:X,0
```

  - 重启音频服务：`sudo systemctl restart alsa`（部分环境可能需要重启系统：`sudo reboot`）

## **Jetson系列主控&Ubuntu系统&树莓派**
**命令行调试**
**一、USB声卡接入**
1. 在插入USB声卡之前，我们使用 lsusb 命令查看一下USB设备：
![\[图片\]](https://i-blog.csdnimg.cn/direct/8d75a3264bf049e88810af54b82dcc0a.png)

2. 然后把USB声卡插上，再使用 lsusb 查看一下，可以看到，多出来的那个就是USB声卡：
![\[图片\]](https://i-blog.csdnimg.cn/direct/363cf81edebe41218c5e1fc131fb166b.png)

3. 然后使用 arecord -l 可以列出所有录音设备，可以看到，我们的USB声卡设备
![\[图片\]](https://i-blog.csdnimg.cn/direct/1d79e43c504a4a79876158ad2807a639.png)

4. 使用 aplay -l 可以列出所有播放设备                                         
![\[图片\]](https://i-blog.csdnimg.cn/direct/990646c1a46f49cfb71d13a61c146b1f.png)

**二、USB声卡使用**
`arecord -l`，例如这里显示UACDemoV1.0即是我们的声卡，`card 0；device 0`，在命令中修改为plughw:0,0指定该录音设备
[图片]
接执行Linux自带的录音命令，录制一段5秒的声音进行测试

```
arecord -D plughw:0,0 -f S16_LE -r 16000 -d 5 -t wav test.wav
```

其中 `plughw:0,0` 表示`card 0` , `device 0`，即我们的USB声卡，需根据`arecord -l` 找到自身设备号进行修改，如果你的UACDemoV1.0即是我们的声卡，显示`card 1`；`device 1`，就需要将命令中的`plughw:0,0`改为`plughw:1,1`。 plughw参数提供了自动的格式转换，可以在不同的数据格式和硬件之间进行桥接。arecord 其他的参数如下：
|指令  | 含义 |  本指令含义
|:- | :-: | - |
| -D | 选择设备名称 |使用外接USB声卡“plughw:1.0”
|-f|录音格式|S16_LE代表有符号16位小端序
|-r|采样率|16000是16KHz采样
|-d|录音时长|录音5秒
|-t|录音格式|wav格式
|test.wav|文件名，可以包含路径|文件名字叫test.wav

如果声音过小，输入命令 `alsamixer` ，来对音量进行调整，按下F6，选择USB声卡，
![\[图片\]](https://i-blog.csdnimg.cn/direct/ff0d86433f3844f28d65f1ca69de5354.png)

然后按下F5，将录音和播音设备都展示出来，我们将录音的音量按上键调高，PCM是播放，CAPTURE MIC是录音
![\[图片\]](https://i-blog.csdnimg.cn/direct/f6f944f32ac54d6d870c02b56235260e.png)

然后使用`aplay`命令来播放

```bash
aplay -D plughw:0,0 -f S16_LE -r 16000 -c 1 test.wav
```

参数说明如下：
- -D plughw:0,0：指定录音设备。plughw:0,0 表示使用第一个声卡的第一个设备。
- -f S16_LE：设置音频文件格式。S16_LE 表示 16 位小端格式（Signed 16-bit Little Endian），一种常用的音频数据格式，“小端”指数据的低位字节存储在内存的低地址端。
- -r 16000：设置采样率。
- -c 1：设置声道数。
- -d 5：设置录音时长/秒。

## **PulseAudio 可视化窗口查看**

![\[图片\]](https://i-blog.csdnimg.cn/direct/e0c4151c8d9d4c609bbcae5c06c50dd8.png)

通过PulseAudio，命令行方式查看
`pactl list sources short`            #  列出当前 PulseAudio 音频服务器中所有可用的音频源
![\[图片\]](https://i-blog.csdnimg.cn/direct/df27d5b11ec340c395f27e13020a7969.png)

> 49 代表源索引 
> Alsa _input.usb 表示这是一个USB输入设备，表示是一个麦克风 
> s16le 表示 16 位小端（Signed 16-bit Little Endian）的音频采样格式。 
> 1ch 表示单声道。 
> 48000Hz 是采样率，表示每秒采样 48000 次
> SUSPENDED 代表当前麦克风是挂起的 
> RUNNING 代表麦克风被占用中

## **问题汇总**

**Jetson**
1. 设备被占用问题                                                                            
![\[图片\]](https://i-blog.csdnimg.cn/direct/99493a8905724429b6bff2e6cdd8acd5.png)                                        
关闭设置页面，重新运行命令
若还不行，重新插拔或者重新启动

查看哪个进程占用音频设备

```bash
sudo lsof /dev/snd/*
```

插上声卡前                                                                                              
![**\[图片\]**](https://i-blog.csdnimg.cn/direct/10a2294f3f6e4a1a827b3841d3a3b7f8.png)                                                      
插上声卡后                                                                                                                                         
![\[图片\]](https://i-blog.csdnimg.cn/direct/10e035846d404e5baa825155f2c95f79.png)                                                                              
杀死进程 `kill -9 PID` ，PID为插上声卡后出现的PID，截图里是33739
然后重新录音、播放

**树莓派**
1.噪音较大问题

> 先将麦克风音量置于100 
> 打开终端 
> sudo vi /boot/config.txt
> #或者可能在/boot/firmware/config.txt 
> 在文本最后添加 
> audio_pwm_mode = 2 
> ESC输入:wq退出保存 
> 然后重启 
> reboot

2.每次重新启动会初始化音量设置
重新设置好音量后，
需要保存当前音量配置到系统默认配置文件
执行以下命令将当前设置持久化

```bash
sudo chmod 664 /var/lib/alsa/asound.state
sudo alsactl store
```

**Ubuntu虚拟机**
1. 录音时有噪音杂音
解决办法：USB控制器兼容性改为3.0或3.1


**RDK x3&x5**
查看设备编号
检查声卡是否存在，检查设备编号。
通过 `cat /proc/asound/cards` 命令确认声卡是否注册
 

```bash
0 [duplexaudio    ]: simple-card - duplex-audio
                      duplex-audio
```

通过 `cat /proc/asound/devices` 命令确认逻辑设备

```bash
root@ubuntu:~# cat /proc/asound/devices
  2: [ 0- 0]: digital audio playback
  3: [ 0- 0]: digital audio capture
  4: [ 0]   : control
 33:        : timer
```

通过 `ls /dev/snd/` 命令检查用户空间的实际设备文件

```bash
root@ubuntu:~# ls /dev/snd/
by-path/   controlC0  pcmC0D0c   pcmC0D0p   timer  
```

  
通过上述查询，可以确认，声卡0对应的是板载声卡；设备也是存在的, 且设备号为 0-0 , 实际我们操作的设备应该是 `pcmC0D0p` 和 `pcmC0D0c`。
录制一段5秒的声音进行测试

```bash
arecord -D plughw:0,0 -f S16_LE -r 16000 -d 5 -t wav test.wav
```

其中 `plughw:0,0` 表示`card 0 , device 0`，即我们的USB声卡，plughw参数提供了自动的格式转换，可以在不同的数据格式和硬件之间进行桥接。arecord 其他的参数如下：
|指令  | 含义 |  本指令含义
|:- | :-: | - |
| -D | 选择设备名称 |使用外接USB声卡“plughw:1.0”
|-f|录音格式|S16_LE代表有符号16位小端序
|-r|采样率|16000是16KHz采样
|-d|录音时长|录音5秒
|-t|录音格式|wav格式
|test.wav|文件名，可以包含路径|文件名字叫test.wav

如果声音过小，输入命令 `alsamixer` ，来对音量进行调整，按下F6，选择USB声卡，
![\[图片\]](https://i-blog.csdnimg.cn/direct/ed85ef64e9db449a8384e3dc35393785.png)

然后按下F5，将录音和播音设备都展示出来，我们将录音的音量按上键调高，PCM是播放，CAPTURE MIC是录音
![\[图片\]](https://i-blog.csdnimg.cn/direct/02c7e5349f814c5a9b498df7cc3df25e.png)

然后使用aplay命令来播放

```bash
aplay -D plughw:0,0 -f S16_LE -r 16000 -c 1 test.wav
```
参数说明如下：
- -D plughw:0,0：指定录音设备。plughw:0,0 表示使用第一个声卡的第一个设备。
- -f S16_LE：设置音频文件格式。S16_LE 表示 16 位小端格式（Signed 16-bit Little Endian），一种常用的音频数据格式，“小端”指数据的低位字节存储在内存的低地址端。
- -r 16000：设置采样率。
- -c 1：设置声道数。
- -d 5：设置录音时长/秒。



## **检查最基本的音频驱动**

能否使用 USB 免驱声卡，核心取决于内核
- 是否启用了 USB Audio Class 支持‌（即 `CONFIG_USB_AUDIO`）；
- 是否加载了相应的内核模块（如 `snd-usb-audio`）

只要内核支持，再补装基础音频工具，就能正常使用；若内核被裁剪，则需要重新编译内核启用驱动。
**步骤 1： 检查内核是否支持 snd_usb_audio**

```bash
方法1： 检查是否已加载驱动模块

lsmod | grep snd_usb_audio

方法2： 检查内核是否内置该模块（即使未加载）

modinfo snd_usb_audio  # 有输出=内核支持；无输出=内核未编译该模块
```

若 `modinfo` 无输出：说明系统内核裁剪了该驱动，需重新编译内核，在 `.config` 中开启：

```bash
CONFIG_SND_USB_AUDIO=m  # 编译为模块，或=y 内置到内核
CONFIG_SND_USB_UA101=y
CONFIG_SND_USB_CAIAQ=y
```

若 `modinfo` 有输出：直接加载模块：

```bash
sudo modprobe snd_usb_audio
```

**步骤 2： 安装基础音频工具（精简版默认无）**
精简版系统通常没有 `alsa-utils` 这类工具，需手动安装：

```bash
Ubuntu/Debian 系统

sudo apt update && sudo apt install -y alsa-utils usbutils

无网络环境： 下载 alsa-utils 离线包，用 dpkg -i 安装
```

**步骤 3： 验证 USB 声卡识别与功能**
1.插入 USB 声卡，执行命令确认设备识别：

```bash
查看 USB 设备枚举

lsusb | grep -i audio

查看音频设备列表

aplay -l
```

输出中出现 `USB Audio` 相关的 `card X` 条目，说明识别成功。
2.测试音频输出（确保扬声器 / 耳机已连接）：

```dart
播放测试音，-D 指定 USB 声卡设备（X 为 aplay -l 显示的 card 编号）

speaker-test -c 2 -D plughw:X,0
```

**步骤 4： （可选）安装音频服务（桌面 / 后台播放需求）**
若需要在后台播放音频、或搭配桌面环境使用，精简版需额外安装音频服务：

```dart
轻量级服务（推荐，无桌面也能用）

sudo apt install -y pulseaudio

或 PipeWire（Ubuntu 22.04+ 推荐）

sudo apt install -y pipewire pipewire-alsa
```

**精简版系统的常见坑点及解决**
**1.权限不足，普通用户无法访问声卡**
解决：将用户加入 audio组，重启后生效：

```dart
sudo usermod -aG audio $USER
```

**2.无声音，但设备识别正常**
解决：用 alsamixer调大音量、解除静音（按 M 键取消静音）：

```dart
alsamixer -c X  # X 为 USB 声卡的 card 编号
```

**3.内核版本过低，不支持新型 USB 声卡时，分以下两种情况**
```dart
# Ubuntu/Debian
sudo apt install -y linux-generic && sudo reboot
```
```dart
# Linux系统
sudo modprobe snd-hda-intel model=generic #（不同机型可尝试不同 model 值）

创建声卡驱动配置文件

sudo echo "options snd-hda-intel model=generic" > /etc/modprobe.d/sound.conf
sudo reboot
```
