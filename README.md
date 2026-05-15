# USB Driverless Sound Card Tutorial

## Visual Testing Software (Windows)

[audio_tools.7z](https://juxitech.feishu.cn/wiki/NThUwAgW5iLyB3kEWzOcyO35nvW)

**Command Summary (Optional)**

- Update the system and install tools:
  - Execute：`sudo apt update && sudo apt full-upgrade`
  - Install ALSA:`sudo apt install alsa-base alsa-utils`
- Identified Hardware:
  - List audio devices:`aplay -l`
  - Check PCI/USB audio devices:`lspci | grep -i audio`, `lsusb`
- Basic Configuration and Verification: 
  - Run the configuration wizard:`sudo alsaconf`(if available)
  - Adjust volume:`alsamixer` (Press **M** to unmute, use arrow keys to adjust volume, and ESC to exit)
  - Save settings:`sudo alsactl store`
  - Playback Test: Test audio output (ensure speakers/headphones are connected):
  - ```Bash
    # Play test tone, -D specifies the USB sound card device (X is the card number displayed by aplay -l)
    speaker-test -c 2 -D plughw:X,0
    ```

  - Restart the audio service:`sudo systemctl restart alsa`(Some environments may require a system restart:`sudo reboot`)

## Jetson Series Main Controller & Ubuntu System & Raspberry Pi

### Command-line debugging

#### 1. USB Sound Card Connection

- Before inserting the USB sound card, we use the `lsusb` command to check the USB devices:

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=MjI1NWFiNGRhZTRkZmIwZjVkZmI5YzNiOTQwZTJlNmNfTVd0RE5xMm95T296ZUhxQTZHVXFsR3N4YWRCaDg3a0NfVG9rZW46SVpYdGJGdTIxb0FBZEt4cHJ3R2NRZ00zbnhjXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

- Then plug in the USB sound card, and use `lsusb`  to check. You can see that the extra one is the USB sound card: 

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=MDI4MDQ2ZWIzYTFjYzNiOTNhOWQ2MDcwYjc1NDE1YmVfN2kySEhLMlVKc1g5SDk3WUdYV3R0TzZ3NFZiRHdNclVfVG9rZW46VTl5S2JnS2Jpb0xHS254eXJZUWNveG1ObktkXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

- Then use ` arecord -l `  to list all recording devices. As you can see, our USB sound card device 

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=MTc1MjQwYWUyMTNiMGQ3YWFhMTFlNGZmZGJjNmY5MWVfUTQyNlRwTVpYZkZuWldFUjJ3UXF4RzVaV0ZsTDFXeWxfVG9rZW46WDBEV2JRaURIb3BvRkt4aGhMVGNQZlplbnBjXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

- Using `aplay -l` can list all playback devices

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDE3N2QzYjg4MmRhYmY1MDg5MjBiZDYwNGYwZTQzYjVfbmVmOHN4N05EbkllcnlTcDd4V0ljSW9qT3Z4OWphT29fVG9rZW46SEVkVGJBNE41bzZqSnJ4M2xXb2NBUkVFblZkXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

#### 2. Using USB Sound Card

`arecord -l`, for example, here UACDemoV1.0 is shown, which is our sound card, card 0; device 0, and in the command, modify it to plughw:0,0 to specify this recording device

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTk3NDY4OGVjYTk3NmEwYjcwMDk5ZTFjMDZkZTZjMDNfcHRXVDBBTzhpSWpETzFwR2ZMSUNkVXJiUURReGlWUFJfVG9rZW46TjZvVWIyUkQ5b0t5WDd4OTVIemN5bGdpbnpmXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

Execute the built-in Linux recording command to record a 5-second sound for testing

```
arecord -D plughw:0,0 -f S16_LE -r 16000 -d 5 -t wav test.wav
```

Among them `plughw:0,0` represents`card 0, device 0`, which is our USB sound card. It needs to be modified according to the device number found by`arecord -l` . If your UACDemoV1.0 is our sound card and it shows card 1; device 1, you need to change`plughw:0,0`in the command to`plughw:1,1`. `plughw`parameter provides automatic format conversion and can bridge between different data formats and hardware. Other parameters of arecord are as follows:

| Instruction | Meaning                             | Meaning of this instruction                   |
| ----------- | ----------------------------------- | --------------------------------------------- |
| -D          | Select device name                  | Use the external USB sound card "plughw:1.0"  |
| -f          | Recording Format                    | S16_LE represents signed 16-bit little-endian |
| -r          | Sampling Rate                       | 16000 is a 16KHz sampling rate                |
| -d          | Recording Duration                  | Record for 5 seconds                          |
| -t          | Recording Format                    | wav format                                    |
| test.wav    | File name, which can include a path | The file name is test.wav                     |

If the sound is too low, enter the command ` alsamixer ` , to adjust the volume, press ` F6 `, select the USB sound card, 

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjdmMTg5ZWRiMmMxYWNjZWMyMjg3Mzk1MzI3MmRkNjVfNDNCd0R5bjJTSTc1bVhiSm1FY3BWVWZCM2tnemhHWVJfVG9rZW46TUR6Q2JRZ05zb2hiT0p4cEE0V2NtQ2hzblpmXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

Then press ` F5 `, display both recording and playback devices, we increase the recording volume by pressing the up key, PCM is for playback, and CAPTURE MIC is for recording 

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=OWIyNDM5ZWFlZGMyYmZhMDc0ZmZhOWY4YTViYjEwNDFfYVhrS2NhN1gydWZGVG80UkljcWFBYTg5NFpBM3U5aU9fVG9rZW46TWRGOWJsTjVlb2xiWVh4UWo5M2NuQ0FkblhZXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

Then use ` aplay ` command to play 

```
aplay -D plughw:0,0 -f S16_LE -r 16000 -c 1 test.wav
```

Parameter descriptions are as follows:

- -D plughw:0,0: Specifies the recording device. plughw:0,0 indicates using the first device of the first sound card.
- -f S16_LE: Sets the audio file format. S16_LE represents 16-bit little endian format (Signed 16-bit Little Endian), a commonly used audio data format, where "little endian" means that the low-order ByteDance of the data is stored at the low address end of memory.
- -r 16000: Set the sampling rate.
- -c 1: Set the number of channels.
- -d 5: Set the recording duration/seconds.

### View PulseAudio Visualization Window

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=Yzg1ZjNiNjVkMWVkNTJkMGE5ZDhiNTliZGI0MTUwYzhfaTdrUGlHeVdPQjAxdm5pVVZxcXpjeGtldGc4NFpTUm5fVG9rZW46S0dWV2I2RXRIb2dhYTV4eWlRQWN3YklRbm9lXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

View via PulseAudio, [ command line ](https://so.csdn.net/so/search?q=命令行&spm=1001.2101.3001.7020) method 

`pactl list sources short`            # Lists all available audio sources in the current PulseAudio audio server

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=M2M4NDI0ZTQ5ZjYwNmVhNmM5ODYzNTIxMzQ5N2MzMWJfQ3BiUlJTVFEyTzJKSm9aeTVhQ0M4eGZxR2RKcmxqZEtfVG9rZW46VkhsUGJQQ25EbzFNVUZ4WWxnZGNPakZablFnXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

> 49 represents the source index
>
> Alsa _input.usb indicates that this is a USB input device, representing a microphone
>
> s16le represents the 16-bit little endian (Signed 16-bit Little Endian) audio sample format.
>
> 1ch represents mono.
>
> 48000Hz is the sampling rate, indicating 48000 samples per second
>
> SUSPENDED indicates that the current microphone is suspended 
>
> RUNNING indicates that the microphone is in use

### Python calls USB driverless sound card

Search for code examples on your own, such as searching for “[Python calling USB driverless sound card](https://blog.csdn.net/weixin_44463519/article/details/157463731?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2~default~YuanLiJiHua~Ctr-3-157463731-blog-105694458.235^v43^pc_blog_bottom_relevance_base9&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~YuanLiJiHua~Ctr-3-157463731-blog-105694458.235^v43^pc_blog_bottom_relevance_base9&utm_relevant_index=4)”

### Problem Summary

#### Jetson

1. Device Occupancy Issue

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTg2MTZkMjY0MTBkYTM0OWZmYzQwYWQ1NjIyN2Y4MjdfVHpuODcyQnQ5RU16bWxaeUhGSVBLMzlZYW1KZEI2MkNfVG9rZW46SVd0TmJEQW9pb1R0cDB4bVNuM2NYTlFNbkFWXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

Close the settings page and re-run the command

If it still doesn't work, try re-plugging or restarting 

Check which process is occupying the audio device 

```
sudo lsof /dev/snd/*
```

Before plugging in the sound card 

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=NjczNmQxN2JkMDU1YzgyY2U4YzZiMzRlNGRlOTdmZTFfeFI0amNXR0NSZ1pjdnl6bmpleVRHMmRObWdVTmM5VW5fVG9rZW46RllMTmJnZzlVb09qQTJ4U0hIYmM2dldqbmNlXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

After plugging in the sound card 

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=MzUzZjMzYmJmYWM1ZjU3NTE0OGU5NmViZmRjMzVmMzRfMTJWbndKQUNkdWdHRWFPa01IMEUzWEZpczNNb3NsWERfVG9rZW46VU9zY2JBY0Ywb1VTMnN4dnFPbWNQVWFKbndmXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

Kill the process ` kill -9 PID ` , where PID is the PID that appears after plugging in the sound card, and in the screenshot it is 33739 

Then re-record and play 

#### Raspberry Pi 

1. Problem of relatively high noise

First, set the microphone volume to 100 Open the terminal 

```Bash
$ sudo vi /boot/config.txt    #Or it may be in /boot/firmware/config.txt
```

Add at the end of the text

```Bash
audio_pwm_mode = 2
```

ESC input: wq to exit and save Then restart

```Bash
$ reboot
```

2. Each restart will initialize the volume settings.

After resetting the volume, 

Needs to save the current volume configuration to the system default configuration file

Execute the following command to persist the current settings

```Python
sudo chmod 664 /var/lib/alsa/asound.state
sudo alsactl store
```

#### Ubuntu Virtual Machine

1. There is noise and interference during recording

Solution: Change the USB controller compatibility to 3.0 or 3.1 

#### RDK x3&x5

##### View device number

Check if the sound card exists and verify the device number. 

Confirm whether the sound card is registered through the `cat /proc/asound/cards` command

```Shell
 0 [duplexaudio    ]: simple-card - duplex-audio
                      duplex-audio
```

Confirm the logical device via the `cat /proc/asound/devices` command

```Shell
root@ubuntu:~# cat /proc/asound/devices
  2: [ 0- 0]: digital audio playback
  3: [ 0- 0]: digital audio capture
  4: [ 0]   : control
 33:        : timer
```

Check the actual device files in user space via the `ls /dev/snd/` command

```Shell
root@ubuntu:~# ls /dev/snd/
by-path/   controlC0  pcmC0D0c   pcmC0D0p   timer    
```

Through the above query, it can be confirmed that sound card 0 corresponds to the onboard sound card; the device also exists, and its device number is `0-0`. In fact, the devices we operate should be `pcmC0D0p` and `pcmC0D0c`.

### Record a 5-second sound for testing

```
arecord -D plughw:0,0 -f S16_LE -r 16000 -d 5 -t wav test.wav
```

Among them `plughw:0,0` represents`card 0, device 0`, which is our USB sound card,`the plughw`parameter provides automatic format conversion and can bridge between different data formats and hardware. Other parameters of arecord are as follows:

| Instruction | Meaning                             | Meaning of this instruction                   |
| ----------- | ----------------------------------- | --------------------------------------------- |
| -D          | Select device name                  | Use external USB sound card "plughw:1.0"      |
| -f          | Recording Format                    | S16_LE represents signed 16-bit little-endian |
| -r          | Sampling Rate                       | 16000 is a 16KHz sampling rate                |
| -d          | Recording Duration                  | Record for 5 seconds                          |
| -t          | Recording Format                    | wav format                                    |
| test.wav    | File name, which can include a path | The file name is test.wav                     |

If the sound is too low, enter the command ` alsamixer ` , to adjust the volume, press ` F6 `, select the USB sound card, 

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=YWFiNGRkY2ZmODg4YWZjMmNkNDgzYjc2NzBmMzJlODVfV3F0MWRQcDMwNDZDSFIzc3pzZFNpdVBIWDJMd1FDdktfVG9rZW46Rkc1WWJhUnNKb0E1T3p4Uk9vNGNFbU40bnZlXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

Then press ` F5 `, display both recording and playback devices, we increase the recording volume by pressing the up key, PCM is for playback, and CAPTURE MIC is for recording 

![img](https://juxitech.feishu.cn/space/api/box/stream/download/asynccode/?code=YWU4MDA4OWM3MmE1YTdiMTAzODE4YjVjNWNkZmY4YzNfY2ZidFBLb2RIcXljb1o3UDQxUmVlQk40RmFGRmRZWlRfVG9rZW46QmJScGJlOTRFb2FGbW54V1pscGM0M1JXbktoXzE3Nzg2NDE1MjQ6MTc3ODY0NTEyNF9WNA)

Then use ` aplay ` command to play 

```
aplay -D plughw:0,0 -f S16_LE -r 16000 -c 1 test.wav
```

Parameter descriptions are as follows:

- -D plughw:0,0: Specifies the recording device. plughw:0,0 indicates using the first device of the first sound card.
- -f S16_LE: Sets the audio file format. S16_LE represents 16-bit little endian format (Signed 16-bit Little Endian), a commonly used audio data format, where "little endian" means that the low-order ByteDance of the data is stored at the low address end of memory.
- -r 16000: Set the sampling rate.
- -c 1: Set the number of channels.
- -d 5: Set the recording duration/seconds.

## Frequently Asked Questions 

### How does the RDK board distinguish between USB sound cards and onboard sound cards?

### How can the audio daughterboard of the RDK X3 series coexist and be used simultaneously with a USB sound card?

### How does RDKS100 support the use of audio functions through a graphical interface?

Refer to [RDK Multimedia Processing and Applications](https://developer.d-robotics.cc/rdk_doc/FAQ/multimedia#usb-声卡和板载声卡如何区分使用)

## Check the most basic audio driver

Whether a USB driverless sound card can be used  **ultimately depends on the kernel** 

- Whether USB Audio Class support (i.e., `CONFIG_USB_AUDIO`) is enabled;
- Is the corresponding kernel module (such as `snd-usb-audio`) loaded?

As long as the kernel supports it, simply install the basic audio tools to use it normally; if the kernel has been trimmed, you need to recompile the kernel to enable the driver. 

### **Step 1: Check if the kernel supports snd_usb_audio** 

```Plain
# Method 1: Check whether the driver module has been loaded
lsmod | grep snd_usb_audio

# Method 2: Check if the module is built into the kernel (even if not loaded)
modinfo snd_usb_audio  # Output exists = Kernel support; No output = The module is not compiled into the kernel
```

**If** **` modinfo `** **has no output** : It indicates that the system kernel has trimmed this driver, and the kernel needs to be recompiled. Enable it in `.config `: 

```Plain
CONFIG_SND_USB_AUDIO=m  # Compile as a module, or =y to build into the kernel
CONFIG_SND_USB_UA101=y
CONFIG_SND_USB_CAIAQ=y
```

**If** **` modinfo `** **has output** : directly load the module: 

```Bash
sudo modprobe snd_usb_audio
```

### Step 2: Install basic audio tools (not included by default in the Lite version) 

The stripped-down system usually does not have ` alsa-utils ` and other such tools, which need to be manually installed: 

```Bash
# Ubuntu/Debian
sudo apt update && sudo apt install -y alsa-utils usbutils

# Offline environment: Download the alsa-utils offline package and install it with dpkg -i
```

### Step 3: Verify USB Sound Card Recognition and Functionality

1. Insert the USB sound card and execute the command to confirm device recognition:

```Bash
# View USB device enumeration
lsusb | grep -i audio

# View the audio device list
aplay -l
```

The appearance of ` USB Audio ` related ` card X ` entries in the output indicates successful recognition. 

2. Test audio output (ensure speakers/headphones are connected):

```Bash
# Play test tone, -D specifies the USB sound card device (X is the card number displayed by aplay -l)
speaker-test -c 2 -D plughw:X,0
```

### Step 4: (Optional) Install audio service (for desktop/background playback requirements) 

If you need to play audio in the background or use it with a desktop environment, the Lite version requires additional installation of audio services: 

```Bash
# Lightweight Service (Recommended, usable without a desktop environment)
sudo apt install -y pulseaudio

# or PipeWire (Recommended for Ubuntu 22.04 and above)
sudo apt install -y pipewire pipewire-alsa
```

## Common Pitfalls and Solutions of the Lite Version System

**1. Insufficient permissions, ordinary users cannot access the sound card**

Solution: Add the user to the ` audio ` group, which will take effect after restart: 

```Bash
sudo usermod -aG audio $USER
```

2. **No sound, but device recognition is normal** 

Solution: Use ` alsamixer ` to increase the volume and unmute (press the ` M ` key to unmute): 

```Bash
alsamixer -c X  # X refers to the card number of the USB sound card
```

3. **When the kernel version is too low and does not support the new USB sound card, the following two scenarios apply** 

```Bash
sudo apt install -y linux-generic && sudo reboot
sudo modprobe snd-hda-intel model=generic #(Different models can try different model values)
# Create the sound card driver configuration file
sudo echo "options snd-hda-intel model=generic" > /etc/modprobe.d/sound.conf
sudo reboot
```
