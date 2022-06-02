# 实验硬件平台介绍与配置

本次实验硬件平台以理学Azure IoT TBox终端为核心，配以总线型温湿度传感器和IO控制盒，以此模拟了一个车载应用环境。

基于此硬件平台，用户可以体验完整的远程数据采集与控制体验，包括：

- 将CAN总线采集的温湿度传感器通过蜂窝无线网络传送到Azure IoT hub，完成车载数据的模拟采集；
- 应用Azure的C2D Message，通过蜂窝无线网络对实验平台进行配置，控制IO控制盒的四个继电器动作并点亮继电器小灯；



![](G:\Azure\azure-iot-solution-labs\images\硬件平台示意图.jpg)





- 性能
  - 主控
    - NXP Coretex-M4 核心
    - 主频180MHZ
  - 存储
    - EEPROM：16KB
    - Flash：16MB~512MB
  - 通信：
    - 4G全网通
    - WIFI，支持AP/STATION模式
  - 位置服务：
    - GPS/BEIDOU/GLONASS
    - 支持基站定位
  - 总线接口：
    - CAN Bus：2路
    - J1708:1路
- 认证：
  - 3C
  - 无线电型号核准
  - 入网许可证