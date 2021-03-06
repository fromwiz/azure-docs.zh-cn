---
title: "在 Azure 搜索中保护数据和操作 | Microsoft Docs"
description: "Azure 搜索安全性基于 Azure 搜索筛选器中通过用户和组安全标识符进行的 SOC 2 合规、加密、身份验证和标识访问。"
services: search
documentationcenter: 
author: HeidiSteen
manager: cgronlun
editor: 
ms.assetid: 
ms.service: search
ms.devlang: 
ms.workload: search
ms.topic: article
ms.tgt_pltfrm: na
ms.date: 01/19/2018
ms.author: heidist
ms.openlocfilehash: c3aa4883e33b1f3494f8502fe7f8b12f7d64a72f
ms.sourcegitcommit: 9cc3d9b9c36e4c973dd9c9028361af1ec5d29910
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/23/2018
---
# <a name="security-and-controlled-access-in-azure-search"></a>Azure 搜索中的安全性和受控访问权限

Azure 搜索[符合 SOC 2 规范](https://servicetrust.microsoft.com/ViewPage/MSComplianceGuide?command=Download&downloadType=Document&downloadId=93292f19-f43e-4c4e-8615-c38ab953cf95&docTab=4ce99610-c9c0-11e7-8c2c-f908a777fa4d_SOC%20%2F%20SSAE%2016%20Reports)，采用覆盖物理安全性、加密传输、加密存储和平台级软件防护的综合性安全体系结构。 在操作上，Azure 搜索仅接受经过身份验证的请求。 你可以选择针对内容添加基于用户的访问控制。 本文会涉及每个层的安全性，但侧重于有关如何在 Azure 搜索中保护数据和操作。

![安全层框图](media/search-security-overview/azsearch-security-diagram.png)

## <a name="physical-security"></a>物理安全性

Microsoft 数据中心提供行业领先的物理安全性，符合广泛的标准和法规要求。 若要了解详细信息，请转到[全球数据中心](https://www.microsoft.com/cloud-platform/global-datacenters)页，或观看有关数据中心安全性的简短视频。

> [!VIDEO https://www.youtube.com/embed/r1cyTL8JqRg]

## <a name="encrypted-transmission-and-storage"></a>加密的传输和存储

加密扩展到整个索引管道：从连接通过传输向下到存储在 Azure 搜索中的索引数据。

| 安全层 | 说明 |
|----------------|-------------|
| 传输中加密 | Azure 搜索在 HTTPS 端口 443 上侦听。 与 Azure 服务建立的跨平台连接经过加密。 |
| 静态加密 | 加密在索引过程中完全进行内部化处理，而不会显著影响完成索引所需的时间或索引大小。 加密自动对所有索引进行，包括对未完全加密的索引（在 2018 年 1 月前创建）的增量更新。<br><br>在内部，加密基于 [Azure 存储服务加密](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)，使用 256 位 [AES 加密](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)进行。|
| [SOC 2 符合性](https://www.aicpa.org/interestareas/frc/assuranceadvisoryservices/aicpasoc2report.html) | 提供 Azure 搜索的所有数据中心中的所有搜索服务都完全符合 AICPA SOC 2。 若要查看完整报告，请转到 [Azure - Azure 政府版 SOC 2 类型 II 报告](https://servicetrust.microsoft.com/ViewPage/MSComplianceGuide?command=Download&downloadType=Document&downloadId=93292f19-f43e-4c4e-8615-c38ab953cf95&docTab=4ce99610-c9c0-11e7-8c2c-f908a777fa4d_SOC%20%2F%20SSAE%2016%20Reports)。 |

加密在 Azure 搜索内部进行，证书和加密密钥由 Microsoft 内部管理，并广泛应用。 无法在门户中或以编程方式打开或关闭加密、管理或替换为自己的密钥，或者查看加密设置。 

静态加密已于 2018 年 1 月 24 日宣布推出并应用于所有区域中的所有服务层，包括共享（免费）服务。 对于完全加密，必须删除该日期之前创建的索引并重新生成，以便进行加密。 否则，仅对 1 月 24 日以后添加的新数据进行加密。

## <a name="azure-wide-logical-security"></a>Azure 范围的逻辑安全性

整个 Azure Stack 提供多种安全机制，因此，创建的 Azure 搜索资源也会自动获得这些安全机制。

+ [订阅或资源级别的锁可防止删除](../azure-resource-manager/resource-group-lock-resources.md)
+ [基于角色的访问控制 (RBAC) 可以控制对信息和管理操作的访问](../active-directory/role-based-access-control-what-is.md)

所有 Azure 服务支持使用基于角色的访问控制 (RBAC) 在不同的服务之间以一致的方式设置访问级别。 例如，仅限“所有者”和“参与者”角色查看敏感数据（如管理密钥），而任何角色的成员都可以查看服务状态。 RBAC 提供“所有者”、“参与者”和“读取者”角色。 默认情况下，所有服务管理员是“所有者”角色的成员。

## <a name="service-access-and-authentication"></a>服务访问和身份验证

虽然 Azure 搜索继承了 Azure 平台的安全保护，但它还提供其自己的基于密钥的身份验证。 密钥的类型（管理员或查询）确定访问的级别。 提交有效密钥被视为请求源自受信任实体的证明。 

需要对每个请求进行身份验证，而每个请求由必需密钥、操作和对象组成。 链接在一起后，两个权限级别（完全或只读）加上上下文便足以针对服务操作提供全面的安全性。 

|密钥|说明|限制|  
|---------|-----------------|------------|  
|管理员|授予所有操作的完全控制权限，包括管理服务以及创建和删除索引、索引器与数据源的能力。<br /><br /> 创建服务时，会在门户中生成两个管理 **api-keys**（称为主密钥和辅助密钥），可按需单独重新生成其中的每个密钥。 由于有两个密钥，可以在滚动其中一个密钥时，使用另一个密钥来持续访问服务。<br /><br /> 只能在 HTTP 请求标头中指定管理密钥。 不能在 URL 中放置管理 API 密钥。|每个服务最多有 2 个密钥|  
|查询|授予对索引和文档的只读访问权限，通常分发给发出搜索请求的客户端应用程序。<br /><br /> 按需创建查询密钥。 可以在门户中手动创建查询密钥，或者通过[管理 REST API](https://docs.microsoft.com/rest/api/searchmanagement/) 以编程方式创建查询密钥。<br /><br /> 可以在 HTTP 请求标头中为搜索、建议或查找操作指定查询密钥。 或者，可以在 URL 中以参数形式传递查询密钥。 根据客户端应用程序编写请求的方式，以查询参数的形式传递密钥可能更方便：<br /><br /> `GET /indexes/hotels/docs?search=*&$orderby=lastRenovationDate desc&api-version=2016-09-01&api-key=A8DA81E03F809FE166ADDB183E9ED84D`|每个服务 50 个密钥|  

 在表面上，管理密钥与查询密钥之间没有区别。 这两个密钥都是由 32 个随机生成的字母数字字符组成的字符串。 如果无法跟踪应用程序中指定了哪种类型的密钥，可以[在门户中检查密钥值](https://portal.azure.com)，或使用 [REST API](https://docs.microsoft.com/rest/api/searchmanagement/) 返回值和密钥类型。  

> [!NOTE]  
>  在请求 URI 中传递敏感数据（例如 `api-key`）被认为是不良的安全做法。 为此，Azure 搜索只接受查询字符串中 `api-key` 形式的查询密钥。除非必须公开索引的内容，否则应避免这样做。 作为经验法则，我们建议以请求标头的形式传递 `api-key`。  

### <a name="how-to-find-the-access-keys-for-your-service"></a>如何查找服务的访问密钥

可以在门户中或通过[管理 REST API](https://docs.microsoft.com/rest/api/searchmanagement/) 获取访问密钥。 有关详细信息，请参阅[管理密钥](search-manage.md#manage-api-keys)。

1. 登录到 [Azure 门户](https://portal.azure.com)。
2. 列出订阅的[搜索服务](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。
3. 在服务页上选择服务，找到“设置” >“密钥”即可查看管理密钥和查询密钥。

![门户页上的“设置”>“密钥”部分](media/search-security-overview/settings-keys.png)

## <a name="index-access"></a>索引访问

在 Azure 搜索中，单个索引不是安全对象。 对索引的访问权限是根据服务层（读取或写入访问）以及操作上下文确定的。

对于最终用户访问，可以使用查询密钥在要连接的应用程序中构建查询请求，将任何请求设置为只读，并包含应用使用的特定索引。 在查询请求中，没有联接索引或同时访问多个索引的概念，所有请求都会根据定义以单个索引为目标。 因此，查询请求本身的结构（密钥加上单个目标索引）定义了安全边界。

管理员和开发人员对索引的访问权限没有区别：两者都需要写访问权限才能创建、删除和更新服务管理的对象。 拥有服务管理密钥的任何人都可以读取、修改或删除同一服务中的任何索引。 为了防止意外删除或恶意删除索引，代码资产的内部源代码管理作为一种补救机制，可以还原意外的索引删除或修改。 Azure 搜索在群集中提供故障转移功能来确保可用性，但它不会存储或执行用于创建或加载索引的专属代码。

需要索引级安全边界的多租户解决方案通常包含一个中间层，客户可以使用它来处理索引隔离。 有关多租户用例的详细信息，请参阅[多租户 SaaS 应用程序与 Azure 搜索的设计模式](search-modeling-multitenant-saas-applications.md)。

## <a name="admin-access-from-client-apps"></a>从客户端应用进行管理访问

Azure 搜索管理 REST API 是 Azure 资源管理器的扩展并共享其依赖关系。 因此，要对 Azure 搜索进行服务管理，必须事先安装 Active Directory。 必须使用 Azure Active Directory 对客户端代码发出的所有管理请求进行身份验证，然后请求才能到达资源管理器。

针对 Azure 搜索服务终结点发出的数据请求（例如“创建索引（Azure 搜索服务 REST API）”或“搜索文档（Azure 搜索服务 REST API）”）在请求标头中使用 api-key。

如果应用程序代码需要处理针对搜索索引或文档执行的服务管理操作以及数据操作，请在代码中实现两种身份验证方法：Azure 搜索的本机访问密钥，以及资源管理器所需的 Active Directory 身份验证方法。 

有关在 Azure 搜索中构建请求的信息，请参阅 [Azure 搜索服务 REST](https://docs.microsoft.com/rest/api/searchservice/)。 有关资源管理器的身份验证要求的详细信息，请参阅[使用资源管理器身份验证 API 访问订阅](../azure-resource-manager/resource-manager-api-authentication.md)。

## <a name="user-access-to-index-content"></a>用户对索引内容的访问

可以通过查询中的安全筛选器实施对索引内容的基于用户的访问，并返回与给定安全标识关联的文档。 基于标识的访问控制不是预定义的角色和角色分配，它是作为一个筛选器实现的，该筛选器可以根据标识修整文档和内容的搜索结果。 下表描述了修整未经授权内容的搜索结果的两种方法。

| 方法 | 说明 |
|----------|-------------|
|[基于标识筛选器的安全修整](search-security-trimming-for-azure-search.md)  | 阐述实现用户标识访问控制的基本工作流。 该工作流包括将安全标识符添加到索引，然后解释如何针对该字段进行筛选，以修整受禁内容的结果。 |
|[Azure Active Directory 标识的安全修整](search-security-trimming-for-azure-search-with-aad.md)  | 此文延伸了前一篇文章的内容，提供了有关从 Azure Active Directory (AAD)（Azure 云平台中的一个[免费服务](https://azure.microsoft.com/free/)）检索标识的步骤。 |

## <a name="table-permissioned-operations"></a>表：权限操作

下表汇总了 Azure 搜索中允许的操作，以及哪个密钥可以解锁特定操作的访问。

| Operation | 权限 |
|-----------|-------------------------|
| 创建服务 | Azure 订阅持有者|
| 缩放服务 | 管理密钥，资源中的 RBAC 所有者或参与者  |
| 删除服务 | 管理密钥，资源中的 RBAC 所有者或参与者 |
| 创建、修改、删除服务中的对象： <br>索引和组件部分（包括分析器定义、评分配置文件、CORS 选项）、索引器、数据源、同义词、建议器。 | 管理密钥，资源中的 RBAC 所有者或参与者  |
| 查询索引 | 管理密钥或查询密钥（RBAC 不适用） |
| 查询系统信息，例如返回统计信息、计数和对象列表。 | 管理密钥，资源的 RBAC（所有者、参与者、读取者） |
| 管理管理密钥 | 管理密钥，资源中的 RBAC 所有者或参与者。 |
| 管理查询密钥 |  管理密钥，资源中的 RBAC 所有者或参与者。 RBAC 读取者可以查看查询密钥。 |


## <a name="see-also"></a>另请参阅

+ [.NET 入门（演示如何使用管理密钥创建索引）](search-create-index-dotnet.md)
+ [REST 入门（演示如何使用管理密钥创建索引）](search-create-index-rest-api.md)
+ [使用 Azure 搜索筛选器进行基于标识的访问控制](search-security-trimming-for-azure-search.md)
+ [使用 Azure 搜索筛选器进行 Active Directory 基于标识的访问控制](search-security-trimming-for-azure-search-with-aad.md)
+ [Azure 搜索中的筛选器](search-filters.md)