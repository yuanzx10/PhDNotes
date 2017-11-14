# 关于数据采集系统的内容简介

设计的芯片需要测试，但是在开始没有大规模应用之前，是需要芯片设计相关人员进行数据采集系统的，尤其是在由诸多寄存器参数需要配置的情况下。所以，这里的数据采集系统包含两个方面的内容，一是PC通过FPGA等读取芯片的输出数据；二是PC通过FPGA等对芯片的各种寄存器参数进行设置。当然，还有其他的方式，但是我所熟悉的就是这两种而已。

还有在数据采集的时候一个非常重要的方面，是通过PC来控制各种仪器生成所需要的数据，以及通过PC来获取示波器的所采集的数据。这个半自动化对芯片进行测试，个人认为是一种比较高效和专业的做法。

## PC控制测试仪器

该子仓库下关于PC控制仪器的主要内容有：

  - 使用PC控制测试设备之一： 综述和简易调试方法
  - 使用PC控制测试设备之二： 基于C++的采集程序框架
  - 使用PC控制测试设备之三： 控制信号发生器
  - 使用PC控制测试设备之四： 控制直流稳压电源
  - 使用PC控制测试设备之五： 自动读取示波器的数据

## 简单的Linux设备驱动

本子仓库下关于采集和控制芯片寄存器设置的主要内容有：

  - GNU make的简易使用
  - Linux I2C 简单的程序设计
  - Linux SPI 简单的程序设计
  - Linux USB 简单的程序设计
  - Linux TCP 简单的程序设计
  - 使用glade进行图形界面的开发

## 一个较为小型的项目构建

  - 小型数据采集系统的构建之一： 项目介绍和基于C++的程序框架
  - 小型数据采集系统的构建之二： 代码笔记