---
layout: post
title: 手机通过BLE控制LED & 使用LED显示红外传感器状态 —— Raspberry Pi 笔记(3)
category: Raspberry Pi
tags: [BLE, GPIO, LED, infra-red sensor, Raspberry Pi]
---


#### 手机通过BLE控制LED
写了一段脚本,手机控制LED,并返回led状态给手机.

led233.py

```
# -*- coding:utf-8 -*-
__author__ = 'Kyle Yuan'
# 通过手机输入'R/B',控制红灯亮或者绿灯亮,同时返回给手机当前两个灯的状态.

import RPi.GPIO as GPIO
import serial
#import time

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
lry = [23,24]
# channel23:绿灯,channel24红灯
GPIO.setup(lry, GPIO.OUT)

def loop():
    while True:
        iin = ser.read()

        if iin == 'B':
            # 手机端输入'B'则把绿灯点亮红灯关掉,也就是23HIGH,24LOW.
            GPIO.output(lry, (1, 0))
            print 'BLUE'
        elif iin == 'R':
            # 手机端输入'R'则把绿灯关掉红灯点亮,也就是23LOW,24HIGH.
            GPIO.output(lry, (0, 1))
            print 'RED'

        # 读取当前23,24的值,显示给手机.
        blue = GPIO.input(23)
        red = GPIO.input(24)
        i = '\n BLUE=%d, RED=%d' % (blue, red)
        ser.write(i)

if __name__ == '__main__':
    ser = serial.Serial('/dev/ttyAMA0', 9600)
    try:
        loop()
    except KeyboardInterrupt:
        ser.close()
        GPIO.cleanup(lry)

```

#### 使用LED指示红外传感器状态

功能:接了一个人体红外感应传感器(23),探测到就亮灯(24),没什么技巧跟led一样一样的,

目前感觉GPIO口只能收发0/1?或者用很多GPIO口,自己规定不同的口表示不同的高度?


对了，之前有一张拍摄K-5IIs上取景器，天气好的情况下可以看到菲涅尔透镜
![Imgur](http://i.imgur.com/bYmavYC.jpg)



```
# -*- coding:utf-8 -*-
__author__ = 'Kyle Yuan'

import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)
GPIO.setup(23, GPIO.IN)
GPIO.setup(24, GPIO.OUT)
while True:
    i = GPIO.input(23)
    if i == 1:
        GPIO.output(24, 1)
    else:
        GPIO.output(24, 0)
    time.sleep(2.5)
    # 此处可根据硬件说明书进行适当延时

```
