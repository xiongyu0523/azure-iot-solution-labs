# 实验0：准备Azure资源组和公共服务

## 🎯实验目的

在进行其他实验之前，我们先创建好资源组和一些公共服务，包括：

- [Resource group](###-1）创建Resource-group)（容纳所有其他实验资源）
- [Storage Account](###-2）创建Storage-Account)（保存各个服务日志&文件）
- [Application Insights](###-4）创建Application-Insights)（为Function提供应用日志服务）
- [Log Analytics workspaces](###-3）创建Log-Analytics-workspaces)（存储Application Insights的日志）

## 🧪实验步骤

### 1）创建Resource group

> 💡如果是现场做实验的同学可能讲师已经完成各个资源组的创建，可以略过本小节。

1. 登录[**Azure Portal**](portal.azure.com)

2. 左侧导航栏选择**Resource group**，点击**Create**

3. **Subscription**选择实验用的订阅，**resource group**输入一个该订阅下独一无二的名称，比如`iot-lab-rg-<your-name>`

4. **Region**选择最近的香港数据中心`East Asia`

5. 点击**Review + Create**->**Create**创建Resource group

> 💡这里选择Region并非说所有在该在Resource group下的服务都将部署到该Region，只是用于存储它所包含服务的metadata。

### 2）创建Storage Account

1. Azure Portal左侧导航栏选择**Create a resource**，在**Storage**分类中选择**Storage Account**点击**Create**开启创建向导

2. **Subscription**和**Resource group**分别选择实验订阅和资源组

3. **Storage account name**输入一个独一无二的的名称，比如`iot-lab-storage-<your-name>`，它会成为Storage Account URL的前缀：`iot-lab-storage-<your-name>.blob.core.windows.net`

4. **Region**选择`East Asia`，其他项目保持不变

5. 点击**Review + Create**->**Create**创建Storage Account

### 3）创建Log Analytics workspaces

1. Azure Portal左侧导航栏选择**Create a resource**，在**Monitoring & Diagnostics**分类中选择**Log Analytics workspaces**点击**Create**开启创建向导

2. **Subscription**和**Resource group**分别选择实验订阅和资源组

3. **Name**输入任意的名称，比如`iot-lab-log-workspace`

4. **Region**选择`East Asia`，其他项目保持不变

5. 点击**Review + Create**->**Create**创建Log Analytics workspaces

### 4）创建Application Insights

1. Azure Portal左侧导航栏选择**Create a resource**，在**Developer Tools**分类中选择**Application Insights**点击**Create**开启创建向导

2. **Subscription**和**Resource group**分别选择实验订阅和资源组

3. **Name**输入任意的名称，比如`iot-lab-app-insights`

4. **Region**选择`East Asia`

5. **Resource Mode**保持`Workspace-based`

6. **Log Analytics workspaces**选择刚刚创建的`iot-lab-log-workspace`

7. 点击**Review + Create**->**Create**创建Application Insights


## 📚扩展阅读

- 🔗[Azure Resource Hierarchy](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview)

- 🔗[Introduction to Azure Storage](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json)

- 🔗[Application Insights overview](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)

- 🔗[Log Analytics workspace overview](https://docs.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview)

