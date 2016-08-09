---
layout: "post"
title: "使用树莓派读取温度传感器(数字型)数据并上传Yeelink —— Raspberry Pi 使用笔记(6)"
category: Raspberry Pi
tags: [temperature transducer, Raspberry Pi, Yeelink]
---

### Pi读取温度并上传Yeelink

昨天找到一款数字温度传感器DS18B20,先拿来传到Yeelink

效果展示

![Imgur](http://i.imgur.com/C6f5oXQ.png)


在线查看
http://www.yeelink.net/devices/340430


#### 资料

DS18B20

数据手册 http://dlnmh9ip6v2uc.cloudfront.net/datasheets/Sensors/Temp/DS18B20.pdf

产品资料 http://keyes-robot.com/productshow.aspx?id=202

因为树莓派可以直接读取,所以不用关心具体内容.

Yeelink

Yeelink是一个免费的物联网传感器展示平台
Yeelink API
http://www.yeelink.net/develop/api


#### 硬件连接

DS18B20模块已经加了一个102贴片电阻,所以就不用再加电阻了,

!此处有一坑,更新内核后的树莓派为了防止冲突默认关闭了GPIO4口作为单总线,

需要进到`sudo nano /boot/config.txt`

添加 `dtoverlay=w1-gpio-pull，gpioin=4`

然后重启,使用lsmod查看模块是否正常加载

`ls /sys/bus/w1/devices/`

查看是否有28开头的设备cd到28*设备,

`cat w1_slave`

访问传感器(linux中设备都以文件形式使用),读取温度值,

![Imgur](http://i.imgur.com/8k45YTB.png)

结果中的第二行最后5位即为当前温度,除以1000获得正常温度值.


#### 代码

因为实时上传所以去掉时间参数,
参照cup温度上传,把终端curl读取文本上传改成直接上传.
设置为半小时上传一次啊，如参考代码，请替换apikey.

```
# -*- coding:utf-8 -*-

import time
import requests
import json
#import glob
#import os

#os.system('modprobe w1-gpio')
#os.system('modprobe w1-therm')

#base_dir = '/sys/bus/w1/devices/'
#device_folder = glob.glob(base_dir + '28*')[0]
#device_file = device_folder + '/w1_slave'
#print device_file

device_file = '/sys/bus/w1/devices/28-01157118a7ff/w1_slave'

def read_temp_raw():
    f = open(device_file, 'r')
    lines = f.readlines()
    f.close()
    return lines

def read_temp():
    lines = read_temp_raw()
    while lines[0].strip()[-3:] != 'YES':
        time.sleep(5)
        lines = read_temp_raw()
    equals_pos = lines[1].find('t=')
    if equals_pos != -1:
        temp_string = lines[1][equals_pos+2:]
        temp_c = float(temp_string) / 1000.0
    return temp_c

def upload_temp():
    temp = read_temp()
    apiheaders = {'U-ApiKey': '43291b7f8d5de2934bc2e58358512aca', 'content-type': 'application/json'}
    apiurl = 'http://api.yeelink.net/v1.0/device/340430/sensor/377028/datapoints'
    upload = {'value': temp}
    requests.post(apiurl, headers=apiheaders, data=json.dumps(upload))
#    print('temp: %f' % temp)

def upload_cup_temp():
    file = open("/sys/class/thermal/thermal_zone0/temp")
    cpu_temp = float(file.read()) / 1000.0
    file.close()
    apiheaders = {'U-ApiKey': '43291b7f8d5de2934bc2e58358512aca', 'content-type': 'application/json'}
    apiurl = 'http://api.yeelink.net/v1.0/device/340430/sensor/377057/datapoints'
    upload = {'value': cpu_temp}
    requests.post(apiurl, headers=apiheaders, data=json.dumps(upload))
#    print('cpu_temp: %f' % cpu_temp)

while True:
    upload_temp()
    upload_cup_temp()
    time.sleep(1800)
```


参考文档:

树莓派+DS18B20温度传感器+Yeelink实现家庭室内温度监控
http://www.2fz1.com/post/raspberry-pi-ds18b20-yeelink/

单总线控制DS18B20
http://www.waveshare.net/forum/article-607-1.html

定时向yeelink上传树莓派CPU温度
http://www.aiuxian.com/article/p-1673521.html

各种温度传感器
http://oszine.com/arduino%E4%BC%A0%E6%84%9F%E5%99%A8%E8%BF%9E%E8%BD%BD%E4%B9%8B%E6%B8%A9%E5%BA%A6%E6%B5%8B%E9%87%8F%E7%AF%87/

requests库
http://cn.python-requests.org/zh_CN/latest/index.html
