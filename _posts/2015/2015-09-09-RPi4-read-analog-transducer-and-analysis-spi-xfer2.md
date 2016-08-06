---
layout: "post"
title: "树莓派读取模拟传感器数据(ADC0832)的方法以及对spi.xfer2方法的理解 —— Raspberry Pi 使用笔记(4)"
category: Raspberry Pi
tags: [analog transducer, Raspberry Pi]
---

#### 树莓派如何读取模拟传感器数据

由于树莓派没有AO模拟接口,所以传感器的模拟数据无法采集.
目前搜集到两种解决办法:
- 一种是在面包板上接一个模数转换器(ADC 0832ccn)然后在接回到树莓派的SPI口
- 另一种是接有AO口的Ardurno

问题:
- 第一种存在的问题是淘宝上买到的模拟传感器通常没有具体的把电压转换成可用数据的公式
- 第二种方法需要考虑电压问题,也就是连接不同的电阻，然后同样面临无转换公式可用的问题。

下边介绍第一种方法
```
连接方式:温度传感器 -> ADC0832 -> 树莓派
在树莓派上使用SPI,用到 Python Spidev

```

#### Serial Peripheral Interface(SPI)

主从式,可一对多

同步序列协定

```
/CS:片选
CLK: 时钟
DI: 主往从送
DO: 从送主往
```
启用SPI

从ssh 登录后,设置:

`sudo raspi-config`

找到高级设置,开启SPI,重启.
(树莓派默认启用了ssh,所以初次使用也无需外接显示器),

查看模块是否载入:

`lsmod`

查看通道:

`ls /dev/spidev*`

测试

相关组件:

```
sudo apt-get install python-dev
sudo pip install spidev
```

让SPI_MOSI 接 SPI_MISO,测试是否可以正常传送,
可以拔掉一根试一下,
```
>>> import spidev
>>> spi = spidev.SpiDev()
>>> spi.open(0,0)
>>> spi.xfer2([1, 2])
[1, 2]
>>> spi.xfer([1, 2])
# 此处拔掉一根
[0, 0]
```

ADO与树莓派的连接:
```
/CS - SPI_CE0_N/SPI_CE1_N  #此处连接了0
CLK - SPI_SCLK
DI - SPI_MOSI
DO - SPI_MISO
(相关信息可参考上文GPIO一节,此处I对I,O对O就好了,不要学串口的RXD对TXD)
```

#### 对Spidev 中 spi.xfer2 方法 的理解

根据上面的连接方式,在树莓派上使用SPI,用到 Python Spidev

ADC我用的是ADC0832(2通道,8位),找到的资料都是以MCP3008(8通道,10位)为例子,对照着弄了一下,接线没什么问题,但是代码部分analog_read(channel)这个函数刚开始没看明白.

adc_tmp36.py
```
import spidev, time

spi = spidev.SpiDev()
spi.open(0,0)

def analog_read(channel):
    # !下面这两行不懂!
    r = spi.xfer2([1, (8 + channel) << 4, 0])
    adc_out = ((r[1]&3) << 8) + r[2]  #3字节?
    return adc_out

while True:
    reading = analog_read(0)
    voltage = reading * 3.3 / 1024
    temp_c = voltage * 100 - 50
    temp_f = temp_c * 9.0 / 5.0 + 32
    print("Temp C=%f\t\tTemp f=%f" % (temp_c, temp_f))
    time.sleep(1)
```
https://github.com/simonmonk/raspberrypi_cookbook/blob/master/code/adc_tmp36.py

用到的方法:
```
open(bus, device)

xfer2(list of values[, speed_hz, delay_usec, bits_per_word])
#Performs an SPI transaction. Chip-select should be held active
#between blocks.
```


对spi.xfer2()的理解
adc_tmp36.py
```
r = spi.xfer2([1, (8 + channel) << 4, 0])
adc_out = ((r[1]&3) << 8) + r[2]
```
https://github.com/simonmonk/raspberrypi_cookbook/blob/master/code/adc_tmp36.py

在上次问题的引用链里看到这个:

```
程式解說:
程式中有一行spi.xfer2，他會送出3 Bytes 給Device，第一個位元是1，相當於二進位的00000001，″8+ch″表示Device的頻道位置，變成"00001000"，″<<4″往左移 4個位元(bits)，變成 ″10000000"，最後一個字元是0，亦即 "00000000"。而程式中 "spi.xfer2([1,(8+channel)<<4,0])"會送出 " 00000001 10000000 00000000" 給接收設備，設備回應三個Bytes回來，而傳回值會放在最右邊10個Bits位置，該值介於0跟1023之間。我們可根據這個數字，判斷光度、溫度等類比的訊號。
```

用表格表示

![Imgur](http://i.imgur.com/9Pejoy9.png)

左移四位(<<4)后 变成

![Imgur](http://i.imgur.com/nO2eK7z.png)

这时候想到上文中那个SPI测试,让SPI_MOSI 接 SPI_MISO,输入[1, 2]后原样返回,也就是说spi.xfer()函数只是按某种格式输入,然后再按相同格式取回数据,不关心外接的设备,也就是不知道你接的是个ADC还是其他什么,

这个时候需要找到MCP3008的数据手册,翻到时序图,
![Imgur](http://i.imgur.com/u5UeMc8.png)

要使用这个芯片,先要设置/CS为低电平0,这里由开头的spi函数实现,
我们通过函数发给ADC的是 `000000001000000000000001` ,共三个字节
ADC一次可以读取1字节,也就是8位,由低到高,需要读三次.

![Imgur](http://i.imgur.com/GHMLhYa.png)

- 第一次是1,表示起始位;返回0.
- 第二次高四位是1000,表示选用单端、通道CH0;返回2位,放在低两位上.
- 第三次是0;返回8位.

- 最后返回的就是list [00000000,000000xx,xxxxxxxx],通过r[1],r[0]读出

由此我们知道第一句

`r = spi.xfer2([1, (8 + channel) << 4, 0])`

的作用就是设定传送方式,并取回数据.

然后看第二句

`adc_out = ((r[1]&3) << 8) + r[2]`

就是取r[1]返回值中的低两位然后和r[2]中的返回值合并

(ADC2832只有2^8个精度所以就无需这一步了),

以上就是对这两句的理解,

唉,我突然好想我的组成原理老师  ![Imgur](http://i.imgur.com/ouvtWcr.png)


##### 参考链接

https://github.com/doceme/py-spidev

ADC0832 datasheet:

http://html.alldatasheet.com/html-pdf/158145/NSC/ADC0832CCN/120/2/ADC0832CCN.html
http://blog.sina.com.cn/s/blog_66eab1060100j78l.html

MCP3008 datasheet 和 连接方法(跟我看的那个例子比较像):

http://atceiling.blogspot.com/2014/04/raspberry-pi-mcp3008.html#.Ve01WXvG6JU

http://tec.gekius.com/blog/935.html



Linux下相关总线介绍

http://www.haifux.org/lectures/258/gpio_spi_i2c_userspace.pdf
