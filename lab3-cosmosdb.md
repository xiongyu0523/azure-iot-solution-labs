# 实验3：存储时序数据到Cosmos DB

## 🎯实验目的

本节实验将学习Azure Cosmos DB的基础概念，从IoT应用的数据特点开始，分析并理解为什么Cosmos DB是IoT场景下的hot path数据库的优选方案。在动手实验使用Function App的Cosmos DB Output binding将上一节实验中蜂窝网关获取的遥测数据写入到提前创建的Cosmos DB数据库container，使用Portal Data Explorer查看数据。

## 📑基础阅读

### ❔IoT应用数据的特点

### ❔Azure Cosmos Database基础知识

Cosmos DB是一个

```
- account               // 每个account
  - database             
    - container
      - item1
      ...
      - itemN
```

## 🧪实验步骤

### 1）创建Cosmos DB SQL API账户

1. Azure Portal左侧导航栏选择**Create a resource**，在**Database**分类中选择**Azure Cosmos DB**点击**Create**开启创建向导

2. 选择`Core (SQL) - Recommended`，点击**Create**

2. **Subscription**和**Resource group**分别选择实验订阅和新建的资源组

4. **Account name**输入一个独立无二的的名称，比如`iot-lab-cosmosdb-<your-name>`，它会成为Cosmos DB Account URL的前缀：`iot-lab-cosmosdb-<your-name>.documents.azure.com`

5. **Location**选择`East Asia`

5. **Capacity mode**选择`Serverless`

6. 点击**Review + Create**->**Create**创建Cosmos DB SQL API Account

### 2）创建Cosmos DB Container

1. 进入Cosmos DB Account，左侧导航栏选择**Data Explorer**，在Explorer窗口中点击**New Container**

2. **Database id**选择**Create New**，输入一个Cosmos DB account范围内独一无二的名称，如`mydatabase`

3. 同样**Container id**输入一个Cosmos DB account范围内独一无二的名称，如`mycontainer`

4. **Partition Key**输入 `/deviceid`，点击**OK**创建Container

### 3）增加Cosmos DB Output binding

Function App的binding功能支持Cosmos DB Trigger/Input/Output，大大简化了开发者的Function代码。在这一步中将使用Cosmos DB Output binding实现将温湿度和deviceid写入数据库。

1. 回到上一步创建的Function中，在左侧导航栏选择**Integration**

2. **Outputs**点击**Add Output**

3. **Binding Type**选择`Azure Cosmos DB`

4. **Cosmos DB account connection**点击**New**，在窗口中选择刚刚创建的Cosmos DB Account，点击**OK**新建一个连接

5. **document parameter name** 保持`outputDocument`

6. **database name**和**Collection Name**分别输入上一步创建的`mydatabase`和`mycontainer`，点击**OK**创建binding

> 💡在Portal上配置binding实质上是通过GUI编写function.json文件，进入**Code+Test**页面选择function.json源码文件，可以看到上一个实验中用向导新建的IoT hub trigger和本实验创建Cosmos DB Output binding的详细配置：

```json
{
    "bindings": [{
        "type": "cosmosDB",
        "name": "outputDocument",
        "direction": "out",
        "connectionStringSetting": "iot-lab-cosmosdb-<your-name>_DOCUMENTDB",
        "databaseName": "mydatabase",
        "collectionName": "mycontainer"
    }]
}
```

每个Binding的配置字段意义有相似之处也有独特的一些配置：

|字段|含义|
|---|---|
|**name**|字符串表示变量名，在这里将成为context.bindings下面一个变量，用作传递数据给Binding |
|**connectionStringSetting**|字符串是Cosmos DB Binding连接服务的connection string变量名，它的值存储在applciation settings中|
|**databaseName**|一个Cosmos DB Binding特有的字段，指明操作哪个database|
|**collectionName**|一个Cosmos DB Binding特有的字段，指明操作哪个container|

### 3）写遥测数据到Cosmos DB

有了上一个实验中获取温湿度值和device id的代码段，再加上一个进入IoT hub的时间戳的metadata数据，我们只要把这些数据整打包成JSON格式的item，通过Binding声明的变量**outputDocument**传递出去即可。

复制下面代码到Function中保存，观察Application Insight日志记录

```javascript
module.exports = async function (context, IoTHubMessages) {
    // 声明一个items数组，把一次Function调用所有的数据都写进去
    const items = [];
    IoTHubMessages.forEach((message, index) => {
        const parsed = JSON.parse(message);
        if (parsed.type === 'cycCan') {
            const temperature = Number('0x' + parsed.payload.c1.substring(6, 10)) / 100;
            const humidity = Number('0x' + parsed.payload.c1.substring(10, 14)) / 100;
            const deviceid = context.bindingData.systemPropertiesArray[index]["iothub-connection-device-id"];
            const arrived = context.bindingData.systemPropertiesArray[index]["iothub-enqueuedtime"];
            // 按照定义的schema构造对象
            const item = {
                deviceid: deviceid,
                arrived: arrived,
                temperature: temperature,
                humidity: humidity
            }
            context.log(item);
            // 把解析到的遥测数据插入到items数组
            items.push(item);
        }
    });
    // 通过Cosmos DB binding声明的outputDocument变量传递内容
    if (items.length != 0) {
        context.bindings.outputDocument = JSON.stringify(items);
    }
};
```

### 4）运行查看Cosmos DB

等待几分钟后进入Cosmos DB查看数据，可以通过SQL语句查询数据，比如一段时间温度传感器的平均值。

1. 进入Cosmos DB Account，左侧导航栏选择**Data Explorer**，在Explorer窗口找到建立好的**database/container/items**目录，默认会运行一条**SELECT * FROM c**的语句列出所有的item，点击任意的item可以查看JSON文档的内容，

    ```
    {
        "deviceid": "a22210001",
        "arrived": "2022-06-02T09:08:09.318Z",
        "temperature": 28.79,
        "humidity": 82.92,
        "id": "fc871121-199d-444f-8d97-77a21cc00924",
        "_rid": "YKI9ANj-mBMBAAAAAAAAAA==",
        "_self": "dbs/YKI9AA==/colls/YKI9ANj-mBM=/docs/YKI9ANj-mBMBAAAAAAAAAA==/",
        "_etag": "\"fa05a828-0000-1900-0000-62987e530000\"",
        "_attachments": "attachments/",
        "_ts": 1654160979
    }
    ```

    > 💡从**id**开始往下的都是系统自动添加的字段，比如_ts是存入Cosmos DB的时间，_etag用作并发控制目的，他们都是由Cosmos DB服务端维护并更新的。

2. 在Data Expolrer中点击**Edit Filter**按钮展开SQL语句WHERE子句部分编辑窗口，尝试的一些下面SQL语句查询一段时间内的遥测数据文档。

    ```sql
    SELECT * FROM c WHERE c.deviceid = "<your-deviceid>" AND (c.arrived between "<start-time>" AND "<end-time>")
    ```

    > 💡以上分别是查询Cosmos DB的两种数据读取方式，通过REST API单点读和SQL查询批量读取，他们的应用场景不同，消耗的RU也是不同的，大多数的重在读取的应用会结合两者进行，以平衡成本和效能。

## 📚扩展阅读

- 🔗[Welcome to Azure Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction)
- 🔗[Azure Cosmos DB in IoT workloads](https://docs.microsoft.com/en-us/azure/architecture/solution-ideas/articles/iot-using-cosmos-db)
- 🔗[Azure Cosmos DB trigger and bindings](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2?tabs=in-process%2Cfunctionsv2&pivots=programming-language-javascript)
- 🔗[Getting started with SQL queries](https://docs.microsoft.com/en-us/azure/cosmos-db/sql/sql-query-getting-started)