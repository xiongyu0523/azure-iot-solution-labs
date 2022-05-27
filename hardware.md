# 实验硬件平台介绍与配置

本次实验硬件平台以理学Azure IoT终端为核心，外接CAN总线温湿度传感器与CAN总线IO模块来模拟IoT传感器与控制网络。
以下为系统框图



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