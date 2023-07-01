# Bootloader

## 固件烧录错误

**问题描述：** 如果烧录固件时发生了错误，导致[pixhawk 1](https://docs.px4.io/main/en/flight_controller/pixhawk.html#_3dr-pixhawk-1-flight-controller-discontinued)无法正常启动，具体表现为飞控板IO输入输出指示灯B/E闪烁，且通过USB连接电脑无法找到对应的飞控板端口，表明bootloader有问题，具体指示灯不同状态参考[LED Meanings](https://docs.px4.io/main/en/getting_started/led_meanings.html)。

**解决办法：** 通过[SWD](https://docs.px4.io/main/en/flight_controller/pixhawk.html#swd-port)端口重新烧录对应的bootloader。具体步骤参考：http://pix.rctoysky.com/how-to-update-bootloader-for-pixhawk-with-jlink-swd.html
文中用到的两个文件均上传至：链接：https://pan.baidu.com/s/1l1XM6flojCtJ2OeAHOtoYQ 提取码：rzoz。

参考焊接方式：
![焊接图 | 300](./asset/troubleshooting_soldering.jpg)
连线方式：
![接线图 | 300](asset/troubleshooting_wiring.jpg)