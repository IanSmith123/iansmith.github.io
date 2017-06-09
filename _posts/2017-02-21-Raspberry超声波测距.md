---
layout:		post
title:	"树莓派应用之超声波测距"
date:	2017-02-21 22:17:07 +0800
categories: jekyll update
subtitle:	"--HC-SR04模块的使用"
author:		"Les1ie"
header-img: "img/Raspberry-back-pic.png"
header-mask: 0.3
---
## 0x00
因为在做的项目的缘故，接触到树莓派的开发，这里使用了HC-SR04超声波模块。

## 0x01
树莓派是使用的3B，使用了4个引脚，一个GND一个Vcc，两个GPIO。分别使用的是02,39,03， 05这几个pin. 
![](/img/Raspberry-GPIO.png)
看图的时候是把网线和usb插口靠近自己这种摆放方式来编号每个pin的。
![](/img/Raspberry-HC-SR04-front.jpg)

## 0x02 使用过程
源代码如下，使用 filezilla传到树莓派里面，ssh登录进去运行即可。
```bash
ssh root@192.168.8.246
```
代码如下
```python
#/usr/bin/python
import RPi.GPIO as GPIO
import time

def checkdist():
    GPIO.output(2, GPIO.HIGH)
    time.sleep(0.000015)
    GPIO.output(2, GPIO.LOW)
    while not GPIO.input(3):
        pass

    t1 = time.time()
    while GPIO.input(3):
        pass
    t2 = time.time()

    return (t2-t1)*340/2

GPIO.setmode(GPIO.BCM)
GPIO.setup(2, GPIO.OUT, initial=GPIO.LOW)
GPIO.setup(3, GPIO.IN)
time.sleep(2)

try:
    while True:
        print 'Distance: %0.2f m' %checkdist()
        time.sleep(0.5)
        
except KeyboardInterrupt:
    GPIO.cleanup()
    print "user quit"
        
```
这个代码网上很多，我是照着敲下来的，跑起来的时候成就感还是满满的。

## 0x03 后续
之后要修改这个代码，项目里面可不能这样写╭(╯^╰)╮

## 2017年02月21日22:42:19
