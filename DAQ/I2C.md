<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

# Linux I2C 简单的程序设计

这周正想写写I2C和与其相关的C/C++的程序设计，可巧今天在知乎上关注的[IC君](https://zhuanlan.zhihu.com/icstudy)就发了一篇文章：[简单优雅的总线协议--I2C](https://zhuanlan.zhihu.com/p/31086959)。这个文章写得很好，但我还是按照自己的理解重新组织了一点点，觉得不好的朋友可以去参考[IC君](https://zhuanlan.zhihu.com/icstudy)的文章。 

## I2C接口简介

I2C(Inter-Integrated Circuit)总线是菲利普公司在1982年为实现CPU和其外围芯片通信而开发的通信接口，能传输的数据率较低。在速度上有几档：标准档（100kbit/s）；快速档（400kbit/s）以及高速档（3.2Mbit/s）。但实际能运行的速度还是取决于你的硬件的，及驱动能力以及线上的负载。比如说我们通常让I2C稳定地跑在5Mbit/s的速度上，其实也能飙到12Mbit/s，但是感觉数据有点不稳定。

I2C是一个支持多个主设备(multi-master)的**两线式串行总线**，这两条线被称为串行数据线（SDA）和串行时钟线（SCL）。在I2C的通信协议的框架之中，没有所谓的从设备(slave device)选择以及仲裁逻辑(arbitration logic)等复杂的结构；理论上任意多的从设备和主设备(master device)都可以连接到这两根线上进行彼此之间的通信，其所遵循的协议大致为：
1. 每个从设备有7-bit的地址，所有连接到总线上的设备的地址都是彼此不同的。_这其实也限制了能连接到同一条总线上的设备数目，当然这是可以拓展的_；
2. 数据以8-bit的字节来传送；
3. 每次通信中都有一些控制位，如通信START，STOP信号，RD/nWR读写方向信号和应答位。

我们仅考虑两个设备之间的通信，其物理的连接方式如下图所示。可以看出这两个设备是彼此之间对称的，在协议中通常把先启动通信的设备称之为主设备，而其他设备称为从设备。

![](https://github.com/yuanzx10/PhDNotes/raw/master/DAQ/figures/I2C-physics-connection.png)

一个具体的通信分为以下几个步骤：

1. 在通信开始的时候，主设备将发出一个START信号；此时从设备在接收到这个开始信号之后，进入监视状态。
2. 主设备接着发出第一个8-bit的数据，其中前7-bit是所欲与之通信的从设备的地址和1-bit的RD/nWR信号，这个信号确定了主设备是要向从设备读数据(RD/nWR=1)还是写数据(RD/nWR=0)。
3. 所有的从设备接收到这个地址信号后将其与自身的地址相比较，如果地址不一致，一直到STOP信号来之前不进行任何动作；如果地址一致，则向SDA发送一个应答(ACK=0)信号。
4. 一旦主设备收到了这个来自从设备的ACK信号，其就可以开始进行数据的读写了。在写数据的时候，每写八位数据，等待从设备的ACK信号；然后接着发下一个字节，等待从设备的ACK信号；如此重复，在发送完最后一个字节以及受到从涉的ACK信号之后，接着发送STOP信号，结束这次数据传输。在读数据的时候，每读入8-bit数据，就像总线上发送一个ACK信号；读完最后一个字节的信号之后，发送一个NACK=1信号，表明已经读取了足够的数据，并发送STOP信号，结束这次数据传输。

一个简单的读数据和写数据的例子如下所示：

![](https://github.com/yuanzx10/PhDNotes/raw/master/DAQ/figures/I2C-data-transfer.png)

基本的读写就是上面这些了，由于我仅仅涉足过简单的I2C使用，所以没有怎么仔细考虑是否有纠错机制和各种可能出错的情况，这里就不详述了。下面简单讲讲两个重要的信号，START和STOP信号，如下图所示。在没有通信的时候，SDA和SCL两条总线均为1。

- START信号： 当SCL=1时，SDA由1变为0
- STOP信号：  当SCL=1时，SDA由0变为1

![](https://github.com/yuanzx10/PhDNotes/raw/master/DAQ/figures/I2C-start-stop.png)
![](https://github.com/yuanzx10/PhDNotes/raw/master/DAQ/figures/I2C-data.png)

而在数据传输之间，I2C协议对$I^2C$

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
