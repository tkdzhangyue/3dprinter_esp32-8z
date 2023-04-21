# 3Dprinter_esp32-8z_tmc2226
<img src="https://github.com/tkdzhangyue/3dprinter_esp32-8z/blob/main/pic/printer.jpg">

## B站视频:
    1. 待更新
    2. 待

## 开源打印机过程中,遇到并且趟平的大坑们.
    1. 待总结.

## ESP32作3D打印机主控的优点
    1. 有wifi，升级固件很方便。
    2. 通过浏览器上传打印Gcode文件。
    3. 修改config参数，也可以在浏览器上进行。
## 缺点
    1. ESP32的引脚少， 不能接丰富的传感器。打印机腔室温度，用常开温度开关接风扇实现。
    2. 不能通过浏览器监控打印进度， 只能启动打印。
## Marlin固件
    根据B站祁同学的2.0.9版本，作了一份提交到2.1.2版本.
    主要修改: 
    1. 步进电机驱动修改为板载一体焊接的TMC2226，优点是比tmc2209好焊接，设置所有参数相同。
    2. 开了X/Y/E0的TMC驱动的UART模式, 速度超过100转换高速模式.注：需要在板上连接对应跳线。
    3. 1号风扇切片软件控制，2号风扇在打印头超过100℃时开启.
    4. 开了热床自动对齐(Z_STEPPER_AUTO_ALIGN),Trident的三点式:
        #define Z_STEPPER_ALIGN_XY { {270, 270 }, {80,  270}, { 175, 80 } }
        注：本机350*350*350尺寸。
    4. 开了热床自动调平(AUTO_BED_LEVELING_BILINEAR)
    5. 屏幕使用TINY_BOY小屏幕，2根线很节约ESP32的引脚资源。
    6. Marlin版本升级到2.1.2之后, 不会遇到2.0.9的bug,比如wifi和oled需要重新编译2次的问题.

## 硬件
    为了防止完全小白的朋友贸然掉入开源打印机的大坑里, 有好多半路翻车的voron2.4/Trident. 目前打算只上传电路图/去掉连线的PCB图. 
    所以, 需要一定的电路/PCB布线的知识储备,贴片焊接的技术手法.
    注: 如果嘉立创表示被大家白嫖完全不介意的话, 也有可能上传完全版图.
### VORON Trident
1. 打印机尺寸350*350*350，进程挤出机。
2. 热床： 
* 加热，碳纤维加热丝，DIY
* 打印平台，下边用弹簧支撑，大概1mm的弹性行程；上边0.5mm白铁板+1mm的普通镜子。
3. 打印头：
* V6散热+特氟龙M6喉管+火山加热块+0.6mm打印嘴。
* Q5MC2需要下移一些，量一下大概位置，用m3注塑螺母烫一下。
* 打印一些大尺寸的工程零件，比如dummy的零件，可以换成0.8mm的喷嘴。
4. 打印速度：目前1500加速度，150mm/s速度, 300mm/s空驶速度.
5. TMC2226电流:E0=1000mA, XY=1500mA.
6. 主板散热风扇常开, XY轴散热片垒3层. 腔室散热用65℃常开开关控制风扇.

## 附加打印件
    1. 主板散热风扇支架
    2. 腔室温控排风，接80螺纹管。
    3. 用8mm聚酯纤维板封箱的,2020管材扣件.

## Cura打印机设置
打印机尺寸是350*350*350,根据实际情况酌情修改.

开始G-code
```
G92 E0 ; Reset Extruder
G28 ; Home all axes
G34；bed align


M104 S{material_print_temperature_layer_0}
M190 S{material_bed_temperature_layer_0}
M109 S{material_print_temperature_layer_0}

G1 X320 Y320 Z0.3 F5000.0 ; Move to start position
G1 X220 Y320.0 Z0.3 F1500.0 E30 ; Draw the first line
G1 X220 Y316 Z0.3 F1500;
G1 X320 Y316 Z0.3 F1500.0 E60 ; Draw the second line
G92 E0 ; Reset Extruder
G1 Z4.0 F3000 ; Move Z Axis up little to prevent scratching of Heat Bed
G1 X175 Y175 Z0.3 F5000.0 ; Move over to prevent blob squish
```
结束G-code
```
G91 ;Relative positioning
G1 E-2 F2700 ;Retract a bit
G1 E-2 Z0.2 F2400 ;Retract and raise Z
G1 X5 Y5 F3000 ;Wipe out
G1 Z2 ;Raise Z more
G90 ;Absolute positioning

G1 X0 Y0 Z200 ;Present print
M106 S0 ;Turn-off fan
M104 S0 ;Turn-off hotend
M140 S0 ;Turn-off bed

M84 X Y E ;Disable all steppers but Z
```



``` git
commit 3e7ee928a060ad115e0529e708566712840bda35
Author: tkdzhangyue <tkdzhangyue@163.com>
Date:   Fri Apr 21 08:58:46 2023 +0800

    [esp32-8z_conf]

````
