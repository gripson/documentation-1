##云-边缘同步服务
又称为模型管理系统(Model Management System)，为了便于机器学习模型或者大数据文件的传输。
###介绍
云-边缘同步服务是边缘计算的管理工具。它旨在通过提供工具在云和边缘之间同步对象来简化在边缘上运行的应用程序。
同步服务的用户可以在云中创建/更新对象，然后将该对象自动传播到相关的边缘节点。同样，可以在边缘更新对象并将其交付给云。示例用例包括配置，规则和操作，用户首选项，AI模型，监视统计信息，部署文件等的同步。

###同步服务组件
同步服务包含两个组件：

* 在云端运行的Cloud Sync Service（CSS）。 CSS支持多租户，高可用性和负载平衡。
* 在边缘节点中运行的边缘同步服务（ESS）。每个ESS节点都有一个类型和唯一的ID。

假设用户应用程序有边缘部分和云部分。该应用程序使用REST调用与同步服务（CSS和ESS）进行交互。 
CSS和ESS节点之间的通信可以通过MQTT或HTTP进行。同步服务旨在与Watson IoT Platform一起使用并通过它执行所有通信。还支持通过HTTP/HTTPS或通过MQTT代理直接通信。

![MMS](../assets/mms.png)

该应用程序向CSS / ESS提供一个对象，该对象包括定义了对象的属性的元数据，还可以是二进制数据。 对象的元数据包括对象的目的地。 
同步服务支持灵活的寻址，该寻址允许将对象从CSS发送到：

- 单个ESS（使用ESS ID）。
- 某个类型的所有ESS节点。
- 任何一组ESS节点。
- 所有ESS节点。

当ESS / CSS上收到对象的更新时，同步服务将通知该应用程序。 
然后，应用程序可以获取并处理对象。 应用程序处理完对象后，可以将其标记为已使用。 同步服务允许用户查看对象的传递状态。

###同步服务主要功能
以下是同步服务提供的主要功能的列表：
1. 同步服务处理云与边缘之间的所有通信方面:
   通信失败，边缘节点不可用，最大消息大小限制，下载恢复，重启后的持久性等等。
2. 简单灵活地控制对象:
   创建/更新/读取/删除操作，版本，持久性，到期时间，激活时间，交付时删除等。
3. 灵活的对象分配:
   动态寻址，直接向文件系统写入/从文件系统读取（文件传输），使用数据链接等。
4. 边缘节点重新/启动时自动同步
5. 跟踪交货状态:
   提供对象在每个目的地的交付状态的指示。
6. 跟踪边缘节点:
   允许用户查看边缘节点
7. 一个安全模型可以控制谁可以创建/更新/读取/删除哪些对象以及可以将这些对象发送到哪些边缘节点。
   请参阅[安全性](#安全性)部分

###开发
关于云-边缘同步服务组件的开发，运行，使用例子以及生成相应的swagger文档指南请参阅[此链接](https://github.com/open-horizon/edge-sync-service)

###安全性
同步服务提供了一个安全的基础结构，可以在该基础结构上从云到边缘以及从边缘到云同步对象。为此，通常需要进行身份验证和授权才能执行所有任务。需要身份验证时，对象和边缘节点（称为目标）均受到保护。

####同步服务认证
需要身份验证时，需要通过Sync Service的RESTful API访问Sync Service的所有应用程序在所有RESTful API调用上提供应用程序密钥和应用程序密钥对。
* 每个客户端SDK都有一个与语言有关的适当API调用，以设置对客户端句柄连接的Sync Service服务器的RESTful API调用时，由客户端“句柄”使用的应用密钥和应用密钥。
* 如果一个人未使用客户端SDK之一，而是一个人进行RESTful API调用，则每次调用时，都必须在基本授权标头中将应用程序密钥和应用程序秘密作为用户名和密码发送。

与Cloud Sync Service实例进行通信时，通常还需要Edge Sync Service实例来标识自己。

应用程序密钥和应用程序密钥对的确切形式取决于配置了与之通信的同步服务实例的方式。应用程序密钥和应用程序秘密对用于确定和验证用户的身份。用户的身份包括用户名，用户所属的组织以及用户的类型。用户可以是组织管理员，常规用户或边缘节点。

应当注意，可以将同步服务设置为完全不需要身份验证。默认设置虽然需要身份验证，但使大多数用户成为组织管理员。这种设置不应在生产设置中使用。

####同步服务授权
同步服务通过限制对对象类型的访问来保护对象。此外，它通过限制对目标类型的访问来保护边缘节点。

#####申请授权
为了使应用程序可以在Sync Service中创建，读取，更新或删除对象，它必须有权访问所讨论对象的对象类型。该应用程序通过以下方式执行此操作：
1. 已通过用户身份进行身份验证，该用户身份是对象组织中的组织管理员。
2. 通过身份验证为常规用户后，其用户名将在相关对象类型的访问控制列表中。
3. 有关对象类型的访问控制列表已标记为可公开访问。

除了应用程序要在Cloud Sync Service上创建对象外，它还必须有权访问对象元数据中提到的所有目标类型。该应用程序通过以下方式执行此操作：
1. 已通过用户身份进行身份验证，该用户身份是对象组织中的组织管理员。
2. 通过身份验证为常规用户后，其用户名就在有关目标类型的访问控制列表中。
3. 有关目标类型的访问控制列表已标记为可公开访问。

#####边缘节点授权
当边缘节点上运行的Sync Service实例与Cloud Sync Service实例进行通信时，它们会自行标识。这样做是为了防止Edge Sync Service实例伪装成其他某个目标。用于边缘同步服务实例的用户身份可以是同步服务支持的任何一种用户身份类型。特别是，如果用户身份为：
1. 边缘节点，身份必须与边缘同步服务实例的目标类型和目标ID相匹配。
2. 组织管理员，它必须是边缘节点所在组织的管理员。
3. 普通用户（该用户的用户名）必须位于边缘同步服务实例的目标类型的访问控制列表中。

####访问控制列表
同步服务使用访问控制列表（ACL）授予普通用户对组织内对象类型和目标类型的读写权限。 作为组织管理员的用户可以在ACL中添加和删除用户。 
当第一个用户的用户名添加到有问题的ACL中时，会自动创建一个ACL。从相关ACL中删除最后一个用户的用户名后，ACL将自动删除。 在ACL中添加用户名星号（*）可以使组织内的所有经过身份验证的用户访问所涉及的对象类型或目标类型。

使用适当的Sync Service RESTful API维护ACL。 所有Sync Service客户端SDK都为这些RESTful API提供了与语言有关的API。