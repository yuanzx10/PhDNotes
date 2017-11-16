# Linux I2C 简单的程序设计

这周正想写写I2C和与其相关的C/C++的程序设计，可巧今天在知乎上关注的[IC君](https://zhuanlan.zhihu.com/icstudy)就发了一篇文章：[简单优雅的总线协议--I2C](https://zhuanlan.zhihu.com/p/31086959)。这个文章写得很好，但我还是按照自己的理解重新组织了一点点，觉得不好的朋友可以去参考[IC君](https://zhuanlan.zhihu.com/icstudy)的文章。 

## I2C接口简介

I2C(Inter-Integrated Circuit)总线是菲利普公司在1982年为实现CPU和其外围芯片通信而开发的通信接口，能传输的数据率较低。在速度上有几档：标准档（100kbit/s）；快速档（400kbit/s）以及高速档（3.2Mbit/s）。但实际能运行的速度还是取决于你的硬件的，及驱动能力以及线上的负载。比如说我们通常让I2C稳定地跑在5Mbit/s的速度上，其实也能飙到12Mbit/s，但是感觉数据有点不稳定。

I2C是一个支持多个主设备的**两线式串行总线**，这两条线被称为串行数据线（SDA）和串行时钟线（SCL）。在I2C的通信协议的框架之中，没有所谓的从设备选择以及仲裁逻辑等复杂的结构；理论上任意多的从设备和主设备都可以连接到这两根线上进行彼此之间的通信，其所遵循的协议大致为：
1. 每个从设备有7-bit的地址，所有连接到总线上的设备的地址都是彼此不同的。_这其实也限制了能连接到同一条总线上的设备数目，当然这是可以拓展的_；
2. 数据以8-bit的字节来传送；
3. 每次通信中都有一些控制位，如通信START，STOP信号，RD/nWR读写方向信号和应答位。

我们仅考虑两个设备之间的通信，其物理的连接方式如下图所示。可以看出这两个设备是彼此之间对称的，在协议中通常把先启动通信的设备称之为主设备，而其他设备称为从设备。

## Linux上的I2C设备调试

为简化调试，通常我们在Linux机器上安装i2c-tools进行设备调试，安装方法为：

```sh
$ sudo apt-get install i2c-tools
$ sudo apt-get update
```

这个工具具有多种用途，具体可以实用``man i2c-tools``来查看。比如需要查看Rospberry Pi上的I2C0和I2C1的端口地址，可以实用如下命令：

```sh
$ sudo i2cdetect -y 0
```

其中``0``代表是I2C0端口，此时将返回其8-bit的数据，其中包括7-bit的端口地址以及为0的R/W应答位。如返回的数据为0000 0110，则最后一位是应答位，所以地址为0x03。


## I2C的程序设计

十分完善的Linux的I2C驱动也不是一件很容易的事情，在这里只是简单地介绍一下在实际测试中怎样使用这个接口来收发数据