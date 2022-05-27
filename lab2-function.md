# 实验2：编写Function实现数据解析

## 🎯实验目的

本节实验将使用Azure Function App处理蜂窝网关上传到IoT Hub的原始数据，根据其CAN协议解析成可以存到数据库的格式

## 📑基础阅读

### ❔什么是Azure Function

### ❔什么是Binding

## 🧪实验步骤

### 1）创建Function App

1. Azure Portal左侧导航栏选择**Create a resource**，在**Computer**分类中选择**Function App**点击**Create**开启创建向导
2. **Subscription**和**Resource group**分别选择自己的订阅上一个实验中创建的资源组
3. **Function App name**输入一个独立无二的的名称，比如**iot-lab-function-your-name**，最终这个名字会成为Function App URL的前缀部分，最终完整的URL为：**iot-lab-function-your-name**.azurewebsites.net
4. **Publish**选择**Code**
5. **Runtime Stack**选择**Node.js**
6. **Version**选择默认的**16 LTS**
6. **Region**选择**East Asia**
7. **Operating System**选择**Windows**
8. **Plan Type**选择**Consumption(Serverless)**
5. 点击**Review + Create**->**Create**创建Function App实例

### 2）创建并执行IoT hub Trigger Function

1. 进入Function App服务，左侧导航栏选择**Functions**，点击**Create**
2. 在打开的窗口中，选择**Develop in Portal**，**Template**选择**IoT Hub(Event Hub)**
3. **New Function**输入一个Function App中独立无二的的名称
4. **事件中心连接**处点击**New**，点击**IoT Hub**分类选择上一个实验创建的IoT Hub实例，下面选择**Events(built-in endpoint)**，点击**OK**
5. **Consumer group**保持默认的$Default
6. Function创建完成后在左侧**Developer**导航栏中点击**Code + Test**后可以看到Function的源码文件**index.js**，默认只是将收到的消息记录到日志。
6. 在左侧**Developer**导航栏中点击**Monitor**后在展开的页面选择**Logs**，稍等片刻可以看到Function执行的日志信息
    ```
    Connected!
    2022-05-27T12:42:33  Welcome, you are now connected to log-streaming service. The default timeout is 2 hours. Change the timeout with the App Setting SCM_LOGSTREAM_TIMEOUT (in seconds).
    2022-05-27T12:43:11.791 [Information] Executing 'Functions.IoTHub_EventHub1' (Reason='(null)', Id=dc9a33cc-e56e-4e91-bc5d-96d9bfbc0515)
    2022-05-27T12:43:11.792 [Information] Trigger Details: PartionId: 2, Offset: 27648-27648, EnqueueTimeUtc: 2022-05-27T12:43:11.7650000Z-2022-05-27T12:43:11.7650000Z, SequenceNumber: 48-48, Count: 1
    2022-05-27T12:43:11.796 [Information] JavaScript eventhub trigger function called for message array: {"common":{"tsp":[0,22,5,27,20,43,9],"did":"89860476262091398282","gnss":{"vld":false,"lon":0,"lat":0,"alt":0,"sat":0,"hdop":0}},"type":"cycDev","payload":{"soc":0,"csq":24,"extV":12.630000114440918,"lockSt":false,"altTime":120360}}
    2022-05-27T12:43:11.797 [Information] Processed message: {"common":{"tsp":[0,22,5,27,20,43,9],"did":"89860476262091398282","gnss":{"vld":false,"lon":0,"lat":0,"alt":0,"sat":0,"hdop":0}},"type":"cycDev","payload":{"soc":0,"csq":24,"extV":12.630000114440918,"lockSt":false,"altTime":120360}}
    2022-05-27T12:43:11.797 [Information] Executed 'Functions.IoTHub_EventHub1' (Succeeded, Id=dc9a33cc-e56e-4e91-bc5d-96d9bfbc0515, Duration=6ms)
    ```

### 3）提取和解析温湿度数据

```javascript
module.exports = async function (context, IoTHubMessages) {
    IoTHubMessages.forEach((message, index) => {
        const messageobj = JSON.parse(message);
        if (messageobj.type === "cycCan") {
            const temperature = (Number('0x' + messageobj.payload.c1.substr(6, 4)) * 0.01).toFixed(2);
            const humidity = (Number('0x' + messageobj.payload.c1.substr(10, 4)) * 0.01).toFixed(2);
            context.log(`Humidity = ${humidity}, Temperature = ${temperature}`);
            context.log(`Device Id = ${context.bindingData.systemPropertiesArray[index]['iothub-connection-device-id']}`)
        }
    });
};
```


### 4）从Function获取metadata

## 📚扩展阅读

- 🔗[Azure Function Overview](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview)
- 🔗[Azure Functions triggers and bindings concepts](https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings?tabs=csharp)