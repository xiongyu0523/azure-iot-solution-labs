# 实验场景、需求与架构分析

## 实验场景

Contoso是一家提供冷链货运物流服务企业，正在研发一套冷链运输车辆状态与运输环境监测系统。从车辆收集运行数据、采集冷柜环境遥测数据和GPS位置，发送到云端进行近乎实时地处理和分析并产生数据洞察和告警，以便尽早发现和干预运输过程中的异常状况，以确保高价值的冷藏货物能大程度的得到保护。作为Contoso物联网系统架构师的你选择了微软Azure作为平台技术合作伙伴，你将从需求分析开始入手，设计一个系统架构并实现一个最小PoC验证可行性。

## 产品需求

### 设备端

由于Contoso不涉及硬件研发，需要从合作伙伴选型T-BOX设备来实现数据的采集和传输。按照要求，T-BOX要能快速部署在现有的大型物流车辆上，从CAN总线上采用J1939协议收集车身和冷柜遥测数据并通过蜂窝无线网络发送到后台。考虑到车辆移动时的网络不确定性，确保覆盖的前提下数据资费的优化，T-BOX需要支持4G LTE网络的CAT-1标准，同也要支持GNSS发送位置信息。在硬件可靠性和安全性上为了能有足够保障，更倾向于采用内嵌eSIM的设备。

### 服务端

系统最多需要支撑10万辆车同时服务，每台车辆最短可以按10分钟的间隔采集数据并进行实时传输，遥测数据进入到数据库后有快速、大量的访问和查询的需求。所有原始数据要求能长期保留用作调试和追踪目的用。设计同时要求能与公司既有的仓储环境监测系统进行集成，提供API给其它部门开发人员进行集成，形成统一的从运输到存储的冷链环节数据监控平台和看板。

## 云架构

### Azure服务选型

结合自身项目需求和微软[Azure IoT Well-Architected Framework](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-overview)指导，应该从以下五大维度考量相关Azure服务的选型：

- [可靠性](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-reliability)：选择拥有[99.9%+ SLA](https://azure.microsoft.com/en-us/support/legal/sla/summary/)保障的Azure PaaS服务构建方案

- [安全性](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-security)：采用[Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/iot-concepts-and-iot-hub)云网关和设备证书接入方式来保证身份和数据传输安全
- [卓越运维](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-operational-excellence)：采用[Azure Function](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview)事件驱动设计模式避免维护虚机等基础设施
- [性能效率](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-performance)：选择[Azure Cosmos](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction) NoSQL数据库支持ms级的时序数据写入和读取
- [成本优化](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-cost-optimization)：采购[Azure Device Catalog](https://devicecatalog.azure.com/)中通过认证的硬件，快速硬件开发&云接入
  
### PoC架构图

根据项目需求和服务选型定下的大致方案，设计PoC架构如下：

1. 设备采用南京理学T-BOX蜂窝网关，通过CAN总线连接I/O模块和温湿度模块数据

2. 设备通过DPS服务注册并连接到IoT hub，使用devcie twin管理设备配置

3. 设备发送的原始数据进入IoT hub后触发Azure Function进行解析后写入Cosmos DB

4. 设备发送的原始数据通过Message route的方式保存一份到Blob storage方便今后故障分析和调试

5. 通过API Management服务提供API给内部集成开发者，API采用Azure Function作为后端，实现Cosmos DB中数据的获取、IoT hub设备的控制，metadata的读取等能力

6. 实现PoC的同时也会用到Azure Monitor、Application Insight、Key Vault等支撑服务，实现服务之间的远程调试监控、安全与认证等功能

![](images/architecture.png)

> "Icon made by [Freepik](https://www.flaticon.com/authors/freepik), [Pixel perfect](https://www.flaticon.com/authors/pixel-perfect), [Surang](https://www.flaticon.com/authors/surang) from [www.flaticon.com](www.flaticon.com)
