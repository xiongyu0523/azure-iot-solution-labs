# 实验1：配置网关发送数据到IoT Hub

## 🎯实验目的

本节实验将通过IoT Hub DPS服务把蜂窝网关实验箱provision到IoT Hub上，并配置网关device twin使能网关从CAN总线指定地址开始采集温湿度传感器原始数据到IoT Hub。为了验证数据是否已经成功达到IoT Hub, 实验使用Azure IoT Explorer工具获取原始数据进行测试。

## 📑基础阅读

### ❔什么是Resource group

Resource group是位于订阅下面的资源容器，用户可以按照不同的组织、项目或资源的生命周期来决定如何组织和管理Resource group下的资源。本节实验将专门创建一个Resource group，既可以帮助分析MVP的总成本，也可以方便在实验结束后一次性清除所有相关的资源。

### ❔什么是IoT Hub和IoT Hub DPS

IoT Hub是在Azure上所有IoT解决方案都必须要用的核心服务。IoT Hub是一个云网关，它支持多种协议设备接入、设备管理以及IoT数据和控制指令的双向收发。IoT Hub具备多种连接其他Azure服务的能力，可以结合其他存储、分析、机器学习等服务实现一个完整的IoT解决方案。

IoT Hub DPS是IoT Hub的一个配套服务，负责帮助简化在设备和服务端的provisioning工作，通过DPS支持高安全的认证设备身份，能根据灵活的规则分配设备到不同的IoT Hub，无需干预的自动注册设备。

### ❔什么是Azure IoT Explorer

Azure IoT Explorer是一个跨平台的Azure IoT调试工具，它主要扮演服务端的角色，访问IoT Hub与设备进行交互。用户可以使用IoT Explorer可以完成设备管理，设备数据接收，发送控制指令和Azure IoT PnP相关操作。

## 🧪实验步骤

### 1）创建Resource group

1. 登录[**Azure Portal**](portal.azure.com)
2. 左侧导航栏选择**Resource group**，点击**Create**
3. **Subscription**选择实验用的订阅，**resource group**输入一个该订阅下独一无二的名称
4. **Region**选择最近的香港数据中心**East Asia**
5. 点击**Review + Create**->**Create**创建Resource group

> 💡这里选择Region并非说所有在该在Resource group下的服务都将部署到该Region，只是用于存储它所包含服务的metadata。

### 2）创建IoT Hub与IoT Hub DPS

1. Azure Portal左侧导航栏选择**Create a resource**，在**Internet of Things**分类中选择**IoT Hub**点击**Create**开启创建向导
2. **Resource group**选择资源组
3. **IoT Hub name**输入一个独立无二的的名称，比如**iot-lab-hub-your-name**，最终这个名字会成为IoT Hub URL的前缀部分，最终完整的URL为：**iot-lab-hub-your-name**.azure-devices.net
4. **Region**选择**East Asia**
5. 点击**Review + Create**->**Create**创建IoT Hub实例
6. 回到**Internet of Things**分类中选择**IoT Hub Device Provisioning Service**点击**Create**开启创建向导
7. **Resource group**选择资源组
8. **Name**输入一个独立无二的的名称，比如**iot-lab-your-name**
9. **Region**选择**East Asia**
10. 点击**Review + Create**->**Create**创建IoT Hub DPS实例

### 3）配置IoT Hub DPS服务

1. 进入IoT Hub DPS服务，左侧导航栏选择**Linked IoT hubs**，点击**Add**
2. 在打开的窗口中，选择订阅和上一步创建的IoT Hub，点击**Save**
3. 回到IoT Hub DPS服务，左侧导航栏选择**Certificates**，点击**Add**
4. 在打开的窗口中，**Certificate name**填写一个在当前DPS中独一无二的的名称，选择实验指南根本目录下的resource/root.pem证书并勾选**Set certificate status to verified on upload**，点击**Save**上传并信任证书。
   > 💡勾选**Set certificate status to verified on upload**可以省略随机数密钥挑战的步骤，但这样做法需要用户需要100%确认证书是正确的。
5. 回到IoT Hub DPS服务，左侧导航栏选择**Manage enrollments**，点击**Add enrollment group**
6. **Group name**输入一个在当前DPS中独一无二的的名称，
7. **Attestation Type**选择**Certificate**
8. **IoT Edge Device**选择**False**
9. **Certificate Type**选择**CA Certficiate**
10. **Primary Certificate**下拉菜单中选择刚刚上传的根证书
11. **Select how you want to assign devices to hubs**选择**Static Conifguration**
12. **Select the IoT hubs this group can be assigned to**选择上一步的IoT hub
13. **Initial Device Twin State**中填写以下内容，以确保蜂窝网关注册到IoT Hub后能够默认开始从CAN总线上采集温湿度数据，以60秒的间隔发送到IoT Hub。
    
    ```json
    {
        "tags": {},
        "properties": {
            "desired": {
                "devconfig": {
                    "devMsgInterval": 60,
                    "canMsgInterval": 60
                },
                "canconfig": {
                    "type": "PGN",
                    "bps": 250,
                    "cycle": {
                        "c1": 65257
                    }
                }
            }
        }
    }
    ```

14. **Enable entry**保持**Enable**，点击**Save**创建enrollment group

### 4）配置蜂窝网关连接IoT Hub

这一步通过使用蜂窝网关自带的网页服务器配置IoT Hub DPS ID Scope，以确保设备能连接到上面创建的DPS服务实例。

1. 启动实验箱电源，连接PC到**AzLektec-XXX**的WiFi热点，密码为**azurelektec**
2. 使用浏览器打开**192.168.4.1**进入配置网页服务器
3. 在**Azure IoT DPS配置**中，填入**ID Scope**后点击**设置**，ID Scope可以在DPS服务的页面右侧找到。
4. 关闭电源重启蜂窝网关。

> 💡实验用的每一个蜂窝网关已经预置了独有的ECC私钥和证书，用户也可以使用第三方CA签发的设备证书，或者使用OpenSSL在本地生成用于测试目的的根证书和设备证书/私钥。在上一步中导入自己的根证书（而非实验提供的根证书），并通过配置网页服务器上传设备证书/私钥到自己的设备上。

### 5）使用Azure IoT Explorer获取原始数据

在默认的情况下，IoT Hub将它所有收到的遥测数据自动存入内置的Event Hub终结点中。Event Hub是一个消息队列服务，它最多支持缓存近7天的数据以供客户端读取和回放，用户可以使用Azure IoT Explorer调试工具观察蜂窝网关发送的原始数据。

1. 打开创建的IoT Hub，左侧导航栏中点开**Security Settings**类别中的**Shared access polices**，在右侧打开的界面中点击**iothubowner** Policy Name，复制第三行**Primary conneciton string**。
2. 打开本地安装好的Azure IoT Explorer工具，点击**Add connection**，将上一步复制的内容贴到对话框中，点击**Save**保存。
3. 在打开的设备列表中找到并点击上一步通过IoT Hub DPS服务注册到IoT Hub中的设备，在左侧导航栏点击第三行**Telemetry**，再点击右边**Start**开始从IoT Hub内置的Event Hub中获取新发送上来的数据。
4. 等待最多一个发送周期的时间，界面上会刷新的原始数据。

## 📚扩展阅读

- 🔗[IoT concepts and Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/iot-concepts-and-iot-hub)
- 🔗[What is Azure IoT Hub Device Provisioning Service?](https://docs.microsoft.com/en-us/azure/iot-dps/about-iot-dps)
- 🔗[X.509 certificate attestation](https://docs.microsoft.com/en-us/azure/iot-dps/concepts-x509-attestation)
- 🔗[Using Microsoft-supplied scripts to create test certificates](https://docs.microsoft.com/en-us/azure/iot-hub/tutorial-x509-scripts)