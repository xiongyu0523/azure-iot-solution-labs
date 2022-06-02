# 实验硬件平台介绍与配置

本次实验硬件平台以理学Azure IoT TBox终端为核心，配以总线型温湿度传感器和IO控制盒，以此模拟了一个车载应用环境。

基于此硬件平台，用户可以体验完整的远程数据采集与控制体验，包括：

- 将CAN总线采集的温湿度传感器通过蜂窝无线网络传送到Azure IoT hub，完成车载数据的模拟采集；
- 应用Azure的C2D Message，通过蜂窝无线网络对实验平台进行配置，控制IO控制盒的四个继电器动作并点亮继电器小灯；



![硬件平台示意图](https://github.com/xiongyu0523/azure-iot-solution-labs/blob/main/images/%E7%A1%AC%E4%BB%B6%E5%B9%B3%E5%8F%B0%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg)



Azure T-BOX性能介绍：

LEKTEC Azure T-BOX是一款通用T-BOX产品。该款产品实现了Azure DPS，用户通过其内置的webserver配置ScopeID以及X509证书后，即可连接上对应的IoT Hub，实现数据通信。在应用层面，用户更可以基于自己的CAN网络对需要采集的总线数据进行配置后获取对应的D2C数据，实现不同CAN网络的应用。

![]()

![Azure T-BOX介绍](https://github.com/xiongyu0523/azure-iot-solution-labs/blob/main/images/Azure%20T-BOX%E4%BB%8B%E7%BB%8D.jpg)

