# 实验4：使用API Management托管API

## 🎯实验目的

本节实验将学习Azure Cosmos DB的基础概念，从IoT应用的数据特点开始，分析并理解为什么Cosmos DB是IoT场景下的hot path数据库的优选方案。在动手实验使用Function App的Cosmos DB Output binding将上一节实验中蜂窝网关获取的遥测数据写入到提前创建的Cosmos DB数据库container，使用Portal Data Explorer查看数据。

## 📑基础阅读

### ❔面向API编程

### 

## 🧪实验步骤

### 1）创建HTTP Trigger Function

Function App binding支持HTTP Request作为Trigger触发Function执行，并通过Output binding提供HTTP response。这是serverless架构中非常重要的一个功能，一个个Function可以组成起来多个一系列API驱动的无服务后端。这一步中从学习一个最简单的HTTP Trigger实例代码开始，了解HTTP Trigger Function的使用和配置。

1. 进入Function App服务，左侧导航栏选择**Functions**，点击**Create**

2. 在打开的窗口中，选择`Develop in Portal`，**Template**选择`HTTP Trigger`

3. **New Function**输入一个该Function App中独立无二的的名称，比如`func_http`

4. **身份验证级别**处选择`Anonymous`，点击**Create**创建Function

5. 进入创建的Function，点击**Code + Test**后可以看到Function的源码文件**index.js**和**function.json**，默认的代码将HTTP request中的name query参数的值填到HTTP responsed的body返回。下面是代码的基本结构和注释：

    ```javascript
    module.exports = async function (context, req) {
        context.log('JavaScript HTTP trigger function processed a request.');

        // req参数在function.json中声明，传递HTTP Request相关信息，包括route参数, query参数和body等
        const name = (req.query.name || (req.body && req.body.name));
        const responseMessage = name
            ? "Hello, " + name + ". This HTTP triggered function executed successfully."
            : "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.";

        // context.res在function.json中声明，用于提供HTTP响应结果
        context.res = {
            // status: 200, /* Defaults to 200 */
            body: responseMessage
        };
    }
    ```
6. 点击**Get Function URL**，在打开的窗口中点击**文件图标**复制Function URL

    ```
    https://iot-lab-function-app-<your-name>.azurewebsites.net/api/<func-name>
    ```

7. 打开浏览器粘贴URL，在URL后增加`?name=<your-name>`，输入回车。从浏览器窗口可以看到回复的 Hello文本。

### 2）修改route参数

上一步中创建默认的URL是`/api/<funct-name>`，用户也可以修改路由参数

1. 在左侧导航栏选择**Integration**

2. 在展开的页面中，**Trigger**选择`HTTP(req)`，打开刚刚创建的HTTP Trigger的配置

3. 在**route template**处修改为`telemetry/{device}`，点击**Save**保存。

4. 完成这一步后，我们的URL已经改变，用户可以重复上一步复制URL粘贴到浏览器测试，{device}可以用任意字符串替代。

### 3）增加Cosmos DB Input binding

这一步结合将

1. 回到上一步创建的Function中，在左侧导航栏选择**Integration**

2. **Input**点击**Add Input**

3. **Binding Type**选择`Azure Cosmos DB`

4. **Cosmos DB account connection**选择上一实验中已创建的Application Setting

5. **document parameter name** 保持`inputDocument`

5. **database name**和**Collection Name**分别输入之前创建的`mydatabase`和`mycontainer`，点击**OK**创建binding。

6. 其他保持不变，最下面**SQL Query**输入下面查询语句，点击**Save**保存。

```
SELECT TOP 10 c.temperature, c.humidity FROM c WHERE c.deviceid = {device} ORDER BY c.arrived DESC
```

8. 下面是向导自动创建的Function.json，无需再进行手动编辑，它各字段的含义与上一实验中介绍的Cosmos DB Output binding是一样的，**sqlQuery**则保存了我们要运行SQL语句的模板。

```json
{
    "bindings": [{
        "type": "cosmosDB",
        "name": "inputDocument",
        "direction": "in",
        "connectionStringSetting": "iot-lab-cosmosdb-<your-name>_DOCUMENTDB",
        "databaseName": "mydatabase",
        "collectionName": "mycontainer",
        "sqlQuery": "SELECT TOP 10 c.temperature, c.humidity FROM c WHERE c.deviceid = {device} ORDER BY c.arrived DESC",
    }]
}
```

> **sqlQuery**中包含了{device}，

### 4）编写和测试Function代码

到了编写代码的环节，可以看到代码几乎是什么事情都不用干，仅仅是将SQL语句查询的结果通过HTTP repsonse body传递出去。

```javascript
module.exports = async function (context, req, inputDocument) {
    context.res = {
        status: 200,

        // inputDocument是在Cosmos DB Input binding中声明的变量名，SQL语句查询到结果会通过它传递给Function
        body: JSON.stringify(inputDocument)
    };
};
```

1. 选择**index.js**文件，复制粘贴上面代码，点**Save**保存。

2. 点击**Get Function URL**，在打开的窗口中点击**文件图标**复制Function URL。

    ```
    https://iot-lab-function-app-<your-name>.azurewebsites.net/api/<func-name>
    ```

3. 打开浏览器粘贴URL，将最后route参数修改为自己的deviceid，输入回车。从浏览器窗口可以看到5条最近的温湿度数据

    ```json
    [
        {"temperature":29.09,"humidity":81.22}, 
        {"temperature":29.08,"humidity":81.33},
        {"temperature":29.05,"humidity":81.29},
        {"temperature":29.05,"humidity":81.28},
        {"temperature":29.06,"humidity":81.24}
    ]
    ```

### 4）



## 📚扩展阅读

- 🔗[Welcome to Azure Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction)
- 🔗[Azure Cosmos DB in IoT workloads](https://docs.microsoft.com/en-us/azure/architecture/solution-ideas/articles/iot-using-cosmos-db)
- 🔗[Azure Cosmos DB trigger and bindings](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2?tabs=in-process%2Cfunctionsv2&pivots=programming-language-javascript)
- 🔗[Getting started with SQL queries](https://docs.microsoft.com/en-us/azure/cosmos-db/sql/sql-query-getting-started)