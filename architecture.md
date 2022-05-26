# 实验场景、需求与架构分析

## 实验场景

Contoso是一家提供冷链货运物流服务企业，你们正在研发一套冷链运输车辆状态与运输环境监测系统。从车辆收集运行数据、发送GPS位置和冷柜环境遥测数据，发送到云端进行近乎实时地处理和分析并产生数据洞察和告警，尽早发现和干预运输过程中的异常状况，以确保高价值的冷藏货物能最大程度的得到保护。作为Contoso物联网系统架构师的你选择了微软Azure作为平台技术合作伙伴

## 产品需求

### 设备端

设计要求能快速部署在现有定的物流车辆上，通过CAN总线和J1939协议接入车身数据和传感器。考虑到车辆位置的网络不确定性和传输数据资费优化，需要支持国内LTE网络CAT-1标准和eSIM，以及GNSS定位等功能。你们公司不涉及到硬件开发，将从第三方合作伙伴选型设备

### 服务端

设计要求系统可以支撑最多10000辆车同时服务，每台车辆最短可以按10分钟的间隔采集数据并进行实时传输，所有原始数据要求能保留用作调试和追踪目的，遥测数据进入到数据库后有被快速、大量的访问和查询的需求，但只需在服务保留3个月时间。设计同时还要求能与公司既有仓储环境监测系统进行集成形成统一、从运输到存储的冷链环节数据监控平台和看板。

## 云架构

### 服务选型

作为Contoso物联网系统架构师的你选择了微软Azure作为平台技术合作伙伴。你结合自身项目需求和微软[Azure IoT Well-Architected Framework](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-overview)指南确定了方案的技术路线图，从下面几个维度考量相关Azure服务的选型：
- [可靠性](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-reliability)：选择拥有[99.9%+ SLA](https://azure.microsoft.com/en-us/support/legal/sla/summary/)保障的Azure PaaS服务构建方案
- [安全性](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-security)：采用[Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/iot-concepts-and-iot-hub)云网关和设备证书接入方式来保证身份和数据传输安全
- [卓越运维](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-operational-excellence)：采用[Azure Function](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview)事件驱动设计模式避免维护虚机等基础设施
- [性能效率](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-performance)：选择[Azure Cosmos](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction) NoSQL数据库支持ms级的时序数据写入和读取
- [成本优化](https://docs.microsoft.com/en-us/azure/architecture/framework/iot/iot-cost-optimization)：采购[Azure Device Catalog](https://devicecatalog.azure.com/)中通过认证的硬件，快速硬件开发&云接入
  
### 架构图

