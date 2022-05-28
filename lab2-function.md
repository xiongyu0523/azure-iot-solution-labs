# 实验2：编写Function实现数据解析

## 🎯实验目的

本节实验将使用Azure Function App处理蜂窝网关上传到IoT Hub的原始数据，根据其CAN协议解析成可读的格式，同时获取device id为后面存储到数据库作准备。

## 📑基础阅读

### ❔什么是Azure Function

### ❔什么是Binding

### ❔一个Function的基本结构

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

```javascript
// Javascript Function使用module.exports声明入口
// context参数总是作为第一个参数
// IoTHubMessages是按照function.json中binding的配置和顺序来命名的
module.exports = function (context, IoTHubMessages) {
    
    //记录日志到Appliation Insight
    context.log(`JavaScript eventhub trigger function called for message array: ${IoTHubMessages}`);
    
    // 当Function配置支持多个消息打包为一条消息触发时，IoTHubMessages是一个[]数组对象
    // forEach接收一个回调函数，message => {}是匿名箭头函数内联写法，表示该函数拥有一个message参数
    IoTHubMessages.forEach(message => {
        context.log(`Processed message: ${message}`);
    });

    // 在Function v1.x runtime中指示函数结束
    context.done();
};
```


6. 在左侧**Developer**导航栏中点击**Monitor**，在展开的页面**Invocation**可以看到Function被调用的记录和成功与否的状态。选择**Logs**，可以看到类似日志：

    ```
    2022-05-28T07:18:15.418 [Information] Executing 'Functions.IoTHub_EventHub1' (Reason='(null)', Id=0cc6c415-3237-4a8b-b1b4-e9bcf835c0d6)
    2022-05-28T07:18:15.418 [Information] Trigger Details: PartionId: 2, Offset: 259968-259968, EnqueueTimeUtc: 2022-05-28T07:18:15.4000000Z-2022-05-28T07:18:15.4000000Z, SequenceNumber: 447-447, Count: 1
    2022-05-28T07:18:15.423 [Information] Processed message: {"common":{"tsp":[0,22,5,28,15,18,14],"did":"89860476262091398282","gnss":{"vld":false,"lon":0,"lat":0,"alt":0,"sat":0,"hdop":0}},"type":"cycCan","payload":{"c1":"0103040b821dff00"}}
    2022-05-28T07:18:15.423 [Information] Executed 'Functions.IoTHub_EventHub1' (Succeeded, Id=0cc6c415-3237-4a8b-b1b4-e9bcf835c0d6, Duration=6ms)
    ```

### 3）提取和解析温湿度数据

蜂窝网关会产生包含**cycDev**和**cycCAN**两种类型的消息，这里关心的是**cycCan**的消息，他的**payload**中会按照device twin配置的CAN ID采集并返回原始数据，下面是

```json
{
    "common": {
        ...
    },
    "type": "cycCan",
    "payload": {
        "c1": "01030400a670e5800"
        "c2": ...
        "c3": ... // 其他
    }
}

```

在本实验中，c1的值为温湿度传感器原始数据，这个字符串的各个字符的含义如下

|字符索引|0-1|2-3|4-5|6-9|10-13|14-15| 
|---|---|---|---|---|---|---|
|示例|01|03|04|0A67|0E58|00|
|含义|帧ID|功能码|数据长度|温度 x 100|湿度 x 100|保留

本节重写Function的代码，根据协议解析转换原始数据为浮点数据。把下面代码复制粘贴到**index.js**中点击**Save**，界面下方自动显示log日志窗口，稍等片刻观察结果。

```javascript

// Function v2.x后的runtime推荐使用async函数，且无需在结束的位置调用context.done()
module.exports = async function (context, IoTHubMessages) {

    IoTHubMessages.forEach(message => {
        // message是一个字符串，先转换为JSON方便处理
        const parsed = JSON.parse(message);
        if (parsed.type === 'cycCan') {
            // substring返回一个范围为[indexStart, indexEnd)字符串
            const temperature = (Number('0x' + parsed.payload.c1.substring(6, 10)) * 0.01).toFixed(2);
            const humidity = (Number('0x' + parsed.payload.c1.substring(10, 14)) * 0.01).toFixed(2);

            context.log(`Temperature = ${temperature}, Humidity = ${humidity}`);
        }
    });

    // Function v2.x后的runtime使用async函数，无需在结束的位置调用context.done()
}
```

正常执行可看到如下日志：

```
2022-05-28T06:53:13.080 [Information] Executing 'Functions.IoTHub_EventHub1' (Reason='(null)', Id=d35c9e79-3d69-4c5e-a755-62c0a651a053)
2022-05-28T06:53:13.080 [Information] Trigger Details: PartionId: 2, Offset: 230768-230768, EnqueueTimeUtc: 2022-05-28T06:53:13.0560000Z-2022-05-28T06:53:13.0560000Z, SequenceNumber: 397-397, Count: 1
2022-05-28T06:53:13.083 [Information] Temperature = 29.26, Humidity = 77.39
2022-05-28T06:53:13.084 [Information] Executed 'Functions.IoTHub_EventHub1' (Succeeded, Id=d35c9e79-3d69-4c5e-a755-62c0a651a053, Duration=3ms)
```


### 4）从Function获取metadata

从Function参数传入的**IoTHubMessages**只包含了Telemetry消息的内容，不包括properties，enqueuedtime等metadata数据。Azure Function javascript规范规定了这些信息通过**context.bindingData**传递，具体不同服务的binding的数据不同。

尝试使用下面代码，记录每条消息的device id

```javascript
module.exports = async function (context, IoTHubMessages) {
    IoTHubMessages.forEach((message, index) => {
        const deviceid = context.bindingData.systemPropertiesArray[index]['iothub-connection-device-id'];
        context.log(`Message ${index} is from ${deviceid}`)
    })
}
```

正常执行可看到如下日志：

```
2022-05-28T06:56:14.606 [Information] Executing 'Functions.IoTHub_EventHub1' (Reason='(null)', Id=85a6f65a-03e6-40c7-b29a-7c4ac33469c9)
2022-05-28T06:56:14.607 [Information] Trigger Details: PartionId: 2, Offset: 234272-234272, EnqueueTimeUtc: 2022-05-28T06:56:14.5840000Z-2022-05-28T06:56:14.5840000Z, SequenceNumber: 403-403, Count: 1
2022-05-28T06:56:14.610 [Information] Message 0 is from n210001
2022-05-28T06:56:14.611 [Information] Executed 'Functions.IoTHub_EventHub1' (Succeeded, Id=85a6f65a-03e6-40c7-b29a-7c4ac33469c9, Duration=5ms)
```

## 📚扩展阅读

- 🔗[Azure Function Overview](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview)
- 🔗[Azure Functions triggers and bindings concepts](https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings?tabs=csharp)
- 🔗[Azure IoT Hub trigger for Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-iot-trigger?tabs=in-process%2Cfunctionsv2%2Cextensionv5&pivots=programming-language-javascript)
- 🔗[Azure Functions JavaScript developer guide](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node?tabs=v2-v3-v4-export%2Cv2-v3-v4-done%2Cv2%2Cv2-log-custom-telemetry%2Cv2-accessing-request-and-response%2Cwindows-setting-the-node-version)