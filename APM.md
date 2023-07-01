# 环境配置

具体配置过程参考[Ardupilot官方文档](https://ardupilot.org/dev/docs/building-setup-linux.html),本文档记录linux环境下docker环境的配置过程。

首先使用git拉取代码仓库，已从原仓库fork，fork仓库目前只更改了 `dockerfile` 和 `install-prereqs-ubuntu.sh` 两个脚本，更换了apt源和pip源。

``` shell
$ git clone https://github.com/shmxce/ardupilot.git
```

使用Ardupilot提供的dockerfile构建镜像时，会使用 `install-prereqs-ubuntu.sh` 脚本下载 `gcc-arm-none-eabi-*.tar.gz2` 编译工具，由于docker在构建镜像时会进入一个容器，容器内部的网络使用桥接模式与主机网络隔离，导致本机的系统代理无法生效，从而下载缓慢。

该问题可以在build时设置网络代理解决：

``` shell
$ cd ardupilot
$ docker build . -t <tag-name> --build-arg https_proxy=<proxy-address> --build-arg http_proxy=<proxy-address>
```

或者设置网络模式为 `host`，使用主机的网络：

``` shell
$ docker build . -t <tag-name> --network host
```

启动docker容器

``` shell
$ docker run --rm -it -v $(pwd):/root/ardupilot:rw <tag-name> bash
```

# Build Example

``` shell
$ waf configure --board fmuv3 build
$ waf copter
$ ln -s $ARDUPILOT_HOME/build/fmuv3/compile_commands.json build/
```

# 基本介绍

![apm|500](asset/Pasted%20image%2020230630231004.png)

ArduPilot代码库分为5个主要部分：
- vehicle code：不同车辆类型固件的顶层目录，6个顶层目录对应定义了6种不同类型车辆：
	- AntennaTracker：天线跟踪器。
	- ArduCopter：多旋翼无人机（主要使用这个类型）。
	- ArduPlane：飞机。
	- ArduSub：水下车辆。
	- Blimp：气艇。
	- Rover：地面无人车。
- libraries：所有车辆共用的代码库，包括传感器驱动，姿态和位置估计（EKF），控制器代码（PID）。
- libraries/AP_HAL：硬件抽象层HAL，用于移植不同的平台。
	- libraries/AP_HAL是顶层目录，定义了特定开发板功能的接口。
	- libraries/AP_HAL_XXX定义了不同开发板的接口。
- Tools：各种工具，例如环境安装，代码测试等。
- external support code：外部支持库的代码，modules下以submodule引入，例如ChibiOS，mavlink等。目前ArduPilot逐渐移除了Nuttx相关代码，使用更轻量的ChibiOS实时操作系统。

## libraries

### core

- [AP_AHRS](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_AHRS) -姿态估计，DCM/EKF。
- [AP_Common](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_Common) - 通用核心库。
- [AP_Math](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_Math) - 数学函数库，向量、矩阵、四元数等运算。
- [AC_PID](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AC_PID) - PID控制器库。
- [AP_InertialNav](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_InertialNav) -用于混合加速度计输入与GPS和气压数据的惯性导航库。
- [AC_AttitudeControl](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AC_AttitudeControl) - ArduCopter控制库包括基于PID控制的姿态、位置控制等多种功能。
- [AC_WPNav](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AC_WPNav) - 航点（waypoint）导航库。
- [AP_Motors](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_Motors) - 多旋翼机与传统直升机的电机混合。
- [RC_Channel](https://github.com/ArduPilot/ardupilot/tree/master/libraries/RC_Channel) - 将APM_RC的pwm输入输出转换为内部单位，例如角度。
- [AP_HAL](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_HAL), [AP_HAL_ChibiOS](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_HAL_ChibiOS), [AP_HAL_Linux](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_HAL_Linux) - 硬件抽象层实现，为上层代码提供相同的接口以便移植。

## sensors

- [AP_InertialSensor](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_InertialSensor) - 读取陀螺仪、加速度计数据，执行校准并以标准单位向上层代码提供数据。
- [AP_RangeFinder](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_RangeFinder) - 声纳和红外距离传感器接口库。
- [AP_Baro](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_Baro) - 气压计接口库。
- [AP_GPS](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_GPS) - GPS接口库。
- [AP_Compass](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_Compass) - 3轴罗盘接口库。
- [AP_OpticalFlow](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_OpticalFlow) - 光流传感器接口库。

### others

- [AP_Mount](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_Mount), [AP_Camera](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_Camera), [AP_Relay](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_Relay) - 相机挂载控制库，相机快门控制库。
- [AP_Mission](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_Mission) - 从EEPROM存储/读取任务命令。
- [AP_Buffer](https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_Buffer) - 用于惯性导航的FIFO buffer。

# 添加自定义控制器

> [!todo]
> 
> https://ardupilot.org/dev/docs/copter-adding-custom-controller.html

