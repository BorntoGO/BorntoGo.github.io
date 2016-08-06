---
layout: post
title: RPi.GPIO 和 HM-10 蓝牙模块 —— Raspberry Pi 使用笔记(1)
category: Raspberry Pi
tags: [Raspberry Pi, GPIO, HM-10]
keywords:
description:
---

后续笔记不再记录导入的模块和硬件的连接方法，请根据关键词自行搜索。

### RPi.GPIO模块

GPIO:General Purpose Input Output 即 通用输入/输出

RPi.GPIO是一个用来控制树莓派GPIO的python模块

```
import RPi.GPIO as GPIO`

GPIO.setmode(GPIO.BOARD)
#or GPIO.setmode(GPIO.BCM)
```

两种模式，BOARD就是板子上这种1-40实际引脚，BCM则是根据BCM2835的寄存器编号。

详见下图:
![Imgur](http://i.imgur.com/IvqnDrd.png)

可以用getmode()函数detect一下,查看当前模式。
#### 设置一个channnel
```
GPIO.setup(channel, GPIO.OUT, initial=GPIO.HIGH)
#通常是OUT对应IN,HIGH对应LOW反例不再列出.
```
#### 多个channel
```
c_lsit = [1, 2]
GPIO.setup(c_list,GPIO.OUT)
```

#### 数个channel
```
c_list = [11,12]    # tuples也可
GPIO.output(c_list, GPIO.LOW)     #全LOW
GPIO.output(c_list, (GPIO.HIGH, GPIO.LOW))    # 1HIGH ,2LOW
```

#### Output
```
设置GPIO针脚的输出状态

GPIO.output(channel, state)
#状态0/GPIO.LOW/False 或者相反.
```

#### Input
```
读取GPIO针脚值

GPIO.input(channel)
#返回上例output中的状态值.
```


#### Cleanup
```
GPIO.cleanup(channel)     #list,tuples皆可
```

#### 参考链接
<https://pypi.python.org/pypi/RPi.GPIO>



### HM-10蓝牙模块

HM-10蓝牙模块采用 TI CC2540 芯片,配置 256Kb 空间,支持AT 指令,用户可根据需要更改角色(主、从模式)以及串口波特率、设备名称、配对密码等参数,使用灵活。

HM 系列蓝牙模块出厂默认的串口配置为:波特率 9600,无校验,数据位 8,停止位 1,无流控。


#### 部分指令


```
AT 返回 OK 则OK

查询设置波特率
AT+BAUD?
AT+BAUD[para1]

查询设置串口校验:
AT+PARI?
AT+PARI[para]

查询设置停止位
AT+STOP?
AT+STOP[para]

查询设置PIO 口输出状态
AT+PIO[Para1]?
AT+PIO [para1][para2]

查询设置设备名指令
AT+NAME?
AT+NAME[para1]

设置模块工作模式
AT+MODE?
AT+MODE[para]

模块复位,重启
AT+RESET
恢复出厂设置
AT+RENEW

查询设置主从模式
AT+ROLE?
AT+ROLE[para1]

查询设置配对密码
AT+PASS?
AT+PASS[para1]

查询本机 MAC地址
AT+ADDR?

帮助
AT+HELP?


```

其他指令及详细参数见官方文档 [http://www.jnhuamao.cn/bluetooth40.zip](http://www.jnhuamao.cn/bluetooth40.zip)


#### 主从模式流程图

![Imgur](http://i.imgur.com/NeCXrhP.png)

![Imgur](http://i.imgur.com/LRg9z7J.png)
