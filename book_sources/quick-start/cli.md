##HZN CLI 命令行工具
HZN命令行界面（CLI）是与Horizon代理和管理中心服务进行交互的主要方式。
> 注意：在成功安装好hzn cli后，在命令行窗口输入hzn以查看有关CLI入门的信息。

###尝试命令
使**version**查看CLI和代理版本号：
```bash
hzn verson
```
在此示例中，命令返回：
```bash
Horizon CLI version: 2.26.12
Horizon Agent version: 2.26.12
```

###查看边缘节点配置
使用**node**从代理中查找节点配置：
```bash
hzn node list
```
>请注意典型的CLI格式为 [CLI] [Object Noun] [Action Verb].

命令返回示例：
```bash
{ "id": "node1", "organization": "myorg", "pattern": "", "name": "node1", "nodeType": "device", "token_last_valid_time": "2020-08-04 16:00:30 -0400 EDT", "token_valid": true, "ha": false, "configstate": { "state": "configured", "last_update_time": "2020-08-04 16:00:31 -0400 EDT" }, "configuration": { "exchange_api": "http://host.docker.internal:3090/v1/", "exchange_version": "2.37.0", "required_minimum_exchange_version": "2.23.0", "preferred_exchange_version": "2.23.0", "mms_api": "http://host.docker.internal:9443", "architecture": "amd64", "horizon_version": "2.26.12" } }
```

在前两行中，id是分配给此边缘节点的ID，organization是拥有该设备的临时组。

如果配置了configstate.state属性值，则当前已为服务注册边缘节点。

最后，如果您在configuration.exchange_version属性中看不到版本号值，则可能是因为代理无法查询交换服务的版本。 这可能表明您的代理未连接到管理中心，但这在all-in-one单机安装中并不常见。

###检查协议
检查您的边缘节点注册服务时形成的协议：
```bash
hzn agreement list
```

此命令显示对象的JSON数组，这些对象显示边缘节点和Agbots之间的一个或多个活动协议：

```bash
[ { "name": "Policy for myorg/node1 merged with myorg/policy-ibm.helloworld_1.0.0", "current_agreement_id": "83785680de5d22c874a0f54dc229738a0e744db22b2c0deeb320b9fdf0967138", "consumer_id": "IBM/agbot", "agreement_creation_time": "2020-08-04 16:44:36 -0400 EDT", "agreement_accepted_time": "", "agreement_finalized_time": "", "agreement_execution_start_time": "", "agreement_data_received_time": "", "agreement_protocol": "Basic", "workload_to_run": { "url": "ibm.helloworld", "org": "IBM", "version": "1.0.0", "arch": "amd64" } } ]
```

请注意以下属性：

| 属性| 说明 |
| ------- | ------- |
|0.agreement_creation_time |AgBot提出协议时|
|0.agreement_accepted_time |边缘节点接受协议的时间|
|0.agreement_finalized_time |协议完成时|
|0.agreement_execution_start_time |边缘节点开始下载容器映像的时间|

在看到protocol_execution_start_time值之后，可以运行docker ps确认工作负载已启动。

###其他常用命令：

| 功能 | 命令 |
|----|----|
|查看默认的“ helloworld” 例子的服务| hzn service log -f ibm.helloworld|
|查看协议协商记录的步骤| hzn eventlog list|
|查看节点策略 |hzn policy list|

###与exchange交互
要使用exchange通信命令，请设置两个环境变量。

您需要安装管理中心，代理和CLI时显示的多合一安装摘要消息中的信息。 摘要消息类似于此示例：
```bash
----------- Summary of what was done:
  1. Started Horizon management hub services: agbot, exchange, postgres DB, CSS, mongo DB
  2. Created exchange resources: system org (IBM) admin user, user org (myorg) and admin user, and agbot
     - Exchange root user generated password: 1234567890
     - System org admin user generated password: AbcDEfg
     - Agbot generated token: Abc123YZ
     - User org admin user generated password: XLmdsg236
     - Node generated token: uingtw398J
     Important: save these generated passwords/tokens in a safe place. You will not be able to query them from Horizon.
  3. Installed the Horizon agent and CLI (hzn)
  4. Created a Horizon developer key pair
  5. Installed the Horizon examples
  6. Created and registered an edge node to run the helloworld example edge service
For what to do next, see: https://github.com/open-horizon/devops/blob/master/mgmt-hub/README.md#all-in-1-what-next
```
使用上一条消息中的“User org admin user generated password”密码替换以下中括号内和其中的内容，并在命令行窗口中输入替换内容后的命令来配置环境变量：
```bash
export HZN_ORG_ID=myorg
export HZN_EXCHANGE_USER_AUTH=admin:[User org admin user generated password]
```
>现在，您应该可以使用CLI连接到exchange。

使用**hzn exchange status**或**hzn exchange node list**来确认它没有错误。 此外，**hzn exchange user list**显示您当前正在使用的用户帐户，以及是否已成功通过身份验证。

同时以下这些命令现在可用：

|功能|命令|
|---|---|
|查看示例边缘服务|hzn exchange service list IBM/|
|查看示例模式|hzn exchange pattern list IBM/|
|查看示例部署策略|hzn exchange deployment listpolicy|

###更多信息
使用 hzn [command] (sub-command) -h模式来探索其他命令。

例：
```bash
hzn -h
hzn exchange -h
hzn exchange status -h
```


