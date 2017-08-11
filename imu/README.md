# IMU-MPU6050 interfacing

This project shows how to interface 6 DoF IMU (Intertial Measurement Unit) with 96Boards. Sensor value will be read by a C program and passed to a Python script which does 
3D rendering with the help of OpenGL and pygame. This allows us to visualize the rotation about X and Y axes graphically. Unix socket is used for 
IPC (Inter Process Communication) between C and Python processes.

# Table of Contents
- [1) Hardware](#1-hardware)
   - [1.1) Hardware Requirements](#11-hardware-requirements)
   - [1.2) Hardware Setup](#12-hardware-setup)
- [2) Software](#2-software) 
   - [2.1) Operating System](#21-operating-system)
   - [2.2) Package Dependencies](#22-package-dependencies)
- [3) Building and running](#3-building-and-running)
- [4) Conclusion](#4-conclusion)

# 1. Hardware

## 1.1 Hardware Requirements:

1. [DragonBoard 410c](http://www.96boards.org/product/dragonboard410c/)
2. [Power Supply](https://www.amazon.com/Adapter-Regulated-Supply-Copper-String/dp/B015G8DZK2)
2. [Sensors Mezzanine](http://www.96boards.org/product/sensors-mezzanine/)
3. [Micro USB Cable](https://www.amazon.com/AmazonBasics-USB-Male-Micro-Cable/dp/B01EK87A82/ref=sr_1_3?ie=UTF8&qid=1497618343&sr=8-3&keywords=micro%2Busb&th=1)
4. [MPU6050 IMU](https://www.tindie.com/products/onehorse/gy-521-mpu-6050-breakout-board/)

## 1.2 Hardware Setup:

First, connect the Sensors Mezzanine board onto the DragonBoard via the low-speed expansion connector. Then, connect MPU6050 to I2C0 of Sensors Mezzanine.

# 2. Software

## 2.1 Operating System

- [Linaro Debian based OS (latest)](https://github.com/96boards/documentation/blob/master/ConsumerEdition/DragonBoard-410c/Downloads/Debian.md)

## 2.2 Package Dependencies
UPM Library
```
$ sudo apt-get install libmraa-dev python-opengl python-pygame
```

# 3. Building and Running:

```shell
$ git clone https://github.com/96boards/projects.git      
$ cd projects/imu																													
$ make
```
Now, hold the sensor in horizontal manner and run the generated binary as a background process
	
```shell
$ sudo ./imu&
```
Next, invoke the python script to visualize IMU data

```shell
$ sudo python visualize.py
```
# 4. Conclusion:

Sensor can be moved about X and Y axes so that the tile on display also moves accordingly. Simply close the pygame window to exit both processes gracefully.