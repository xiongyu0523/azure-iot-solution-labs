# 实验3：存储时序数据到Cosmos DB

## 🎯实验目的

本节实验用户将了解IoT应用数据特点，学习cosmos的基本概念并使用Function App的Output binding将蜂窝网关获取的遥测数据写入Cosmos DB数据库，使用Portal Data Explorer和SQL语句查看数据。

## 📑基础阅读

### ❔IoT应用的数据特点和数据库选择

### ❔Azure Cosmos Database基础知识

```
- account
  - database
    - container
      - item
```


## 🧪实验步骤

### 1）创建Cosmos DB SQL API账户

1. Azure Portal左侧导航栏选择**Create a resource**，在**Database**分类中选择**Azure Cosmos DB**点击**Create**开启创建向导
2. 选择**Core (SQL) - Recommended**，点击**Create**
3. **Subscription**和**Resource group**选择自己的订阅和实验1中创建的资源组
4. **Account name**输入一个独立无二的的名称，比如`iot-lab-cosmosdb-<your-name>`，最终这个名字会成为Cosmos DB URL的前缀部分，最终完整的URL为：`iot-lab-function-<your-name>.documents.azure.com`
5. **Location**选择**East Asia**
5. **Capacity mode**选择**Serverless**
6. 点击**Review + Create**->**Create**创建Cosmos DB SQL API Account实例

### 2）创建Cosmos DB container

1. 进入Cosmos DB Account，左侧导航栏选择**Data Explorer**，在Explorer窗口中点击**New Container**
2. **Database id**选择**Create New**，输入一个Cosmos DB account范围内独一无二的名称，如`mydatabase`
3. 同样**Container id**输入一个Cosmos DB account范围内独一无二的名称，如`mycontainer`
4. **Partition Key**输入 **/deviceid**，点击**OK**创建container

### 3）配置Cosmos DB Output binding

Function App的binding功能支持Cosmos DB Trigger/Input/Output，大大简化了开发者在Function中编写的代码。在这一步中我们将使用Cosmos DB Output binding实现将温湿度和metadata数据写入文档数据库。

1. 回到上一步创建的Function中，在左侧导航栏选择**Integration**
2. **Outputs**点击**Add Output**
3. **Binding Type**选择**Azure Cosmos DB**
4. **Cosmos DB account connection**点击**New**，在窗口中选择刚刚创建的Cosmos DB Account，点击**OK**新建一个连接。
5. **database name**和**Collection Name**分别输入上一步创建的**database id**和**container id**，点击**OK**创建binding。

> 💡在Portal上配置binding实质上是通过GUI编写function.json文件，进入**Code+Test**页面选择function.json源码文件，可以看到上一个实验中我们用向导新建的IoT hub trigger binding和本实验创建Cosmos DB Output binding的详细配置。

### 3）编写Function代码写数据库

结合上一个实验中获取温湿度值和device id的代码，我们

```

```

## 📚扩展阅读

- 🔗[Welcome to Azure Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction)
- 🔗[Azure Cosmos DB in IoT workloads](https://docs.microsoft.com/en-us/azure/architecture/solution-ideas/articles/iot-using-cosmos-db)
- 🔗[Azure Cosmos DB trigger and bindings](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2?tabs=in-process%2Cfunctionsv2&pivots=programming-language-javascript)