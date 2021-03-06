---
title: "使用 Linux VM 上用户分配的托管服务标识 (MSI) 访问 Azure Cosmos DB"
description: "本教程介绍使用 Linux VM 上用户分配的托管服务标识 (MSI) 访问 Azure Cosmos DB 的过程。"
services: active-directory
documentationcenter: 
author: daveba
manager: mtillman
editor: 
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/14/2018
ms.author: skwan
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 2c0c3597999e80af86f079385653d94ddfcab245
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/21/2018
---
# <a name="use-a-user-assigned-managed-service-identity-msi-on-a-linux-vm-to-access-azure-cosmos-db"></a>使用 Linux VM 上用户分配的托管服务标识 (MSI) 访问 Azure Cosmos DB 

[!INCLUDE[preview-notice](~/includes/active-directory-msi-preview-notice-ua.md)]

本教程介绍如何从 Linux 虚拟机创建并使用用户分配的托管服务标识 (MSI)，然后用它来访问 Azure Cosmos DB。 学习如何：

> [!div class="checklist"]
> * 创建用户分配的托管服务标识 (MSI)
> * 将用户分配的 MSI 分配给 Linux 虚拟机
> * 向 MSI 授予对 Azure Cosmos DB 实例的访问权限
> * 使用用户分配的 MSI 标识获取访问令牌，并用它访问 Azure Cosmos DB

## <a name="prerequisites"></a>先决条件

[!INCLUDE [msi-core-prereqs](~/includes/active-directory-msi-core-prereqs-ua.md)]

[!INCLUDE [msi-tut-prereqs](~/includes/active-directory-msi-tut-prereqs.md)]

若要运行本教程中的 CLI 脚本示例，你有两种选择：

- 通过 Azure 门户或每个代码块右上角的“试用”按钮使用 [Azure Cloud Shell](~/articles/cloud-shell/overview.md)。
- 如果喜欢使用本地 CLI 控制台，请[安装最新版 CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli)（2.0.23 或更高版本）。

## <a name="sign-in-to-azure"></a>登录 Azure

登录 Azure 门户 ([https://portal.azure.com](https://portal.azure.com))。

## <a name="create-a-linux-virtual-machine-in-a-new-resource-group"></a>在新的资源组中创建 Linux 虚拟机

本教程将新建一个 Linux VM。 另外，还可以在现有 VM 上启用 MSI。

1. 单击 Azure 门户左上角的“+ 创建资源”。
2. 选择“计算”，然后选择“Ubuntu Server 16.04 LTS VM”。
3. 输入虚拟机信息。 对于“身份验证类型”，选择“SSH 公钥”或“密码”。 使用创建的凭据可以登录 VM。

    ![MSI Linux VM](~/articles/active-directory/media/msi-tutorial-linux-vm-access-arm/msi-linux-vm.png)

4. 在“订阅”下拉列表中，选择虚拟机对应的订阅。
5. 在“资源组”下选择“新建”，然后键入此 VM 的资源组的名称。 完成后，单击“确定”。
6. 选择 VM 大小。 若要查看更多大小，请选择“全部查看”或更改“支持的磁盘类型”筛选器。 在“设置”边栏选项卡中保留默认值，然后单击“确定”。

## <a name="create-a-user-assigned-msi"></a>创建用户分配的 MSI

1. 如果使用的是 CLI 控制台（而不是 Azure Cloud Shell 会话），则先登录到 Azure。 使用与要在其下创建新 MSI 的 Azure 订阅关联的帐户：

    ```azurecli
    az login
    ```

2. 使用 [az identity create](/cli/azure/identity#az_identity_create) 创建用户分配的 MSI。 `-g` 参数指定要创建 MSI 的资源组，`-n` 参数指定其名称。 请务必将 `<RESOURCE GROUP>` 和 `<MSI NAME>` 参数值替换为自己的值：

    ```azurecli-interactive
    az identity create -g <RESOURCE GROUP> -n <MSI NAME>
    ```

    响应包含所创建的用户分配 MSI 的详细信息，类似于以下示例（请记下 MSI 的 `clientId` 和 `id` 值，因为这两个值会在后续步骤中用到）：

    ```json
    {
    "clientId": "73444643-8088-4d70-9532-c3a0fdc190fz",
    "clientSecretUrl": "https://control-westcentralus.identity.azure.net/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<MSI NAME>/credentials?tid=5678&oid=9012&aid=123444643-8088-4d70-9532-c3a0fdc190fz",
    "id": "/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<MSI NAME>",
    "location": "westcentralus",
    "name": "<MSI NAME>",
    "principalId": "c0833082-6cc3-4a26-a1b1-c4b5f90a981f",
    "resourceGroup": "<RESOURCE GROUP>",
    "tags": {},
    "tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533bl",
    "type": "Microsoft.ManagedIdentity/userAssignedIdentities"
    }
    ```

## <a name="assign-your-user-assigned-msi-to-your-linux-vm"></a>将用户分配的 MSI 分配给 Linux VM

与系统分配的 MSI 不同，用户分配的 MSI 可由多个 Azure 资源上的客户端使用。 在本教程中，将其分配给单个 VM。 也可以将其分配给多个 VM。

使用 [az vm assign-identity](/cli/azure/vm#az_vm_assign_identity) 将用户分配的 MSI 分配给 Linux VM。 请务必将 `<RESOURCE GROUP>` 和 `<VM NAME>` 参数值替换为自己的值。 将上一步返回的 `id` 属性用于 `--identities` 参数值：

```azurecli-interactive
az vm assign-identity -g <RESOURCE GROUP> -n <VM NAME> --identities "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<MSI NAME>"
```

## <a name="create-a-cosmos-db-account"></a>创建 Cosmos DB 帐户 

如果还没有 Cosmos DB 帐户，可以立即创建一个。 如果你愿意，也可以跳过此步骤，使用现有的 Cosmos DB 帐户。 

1. 单击 Azure 门户左上角的“+/创建新服务”按钮。
2. 单击“数据库”，然后单击“Azure Cosmos DB”，新的“新建帐户”面板便会显示。
3. 输入 Cosmos DB 帐户的 **ID**，供以后使用。  
4. **API** 应设置为“SQL”。 本教程中介绍的方法可以与其他可用的 API 类型配合使用，但本教程中的步骤是针对 SQL API 的。
5. 确保“订阅”和“资源组”与上一步中创建 VM 时指定的名称匹配。  选择提供 Cosmos DB 的“位置”。
6. 单击“创建”。

## <a name="create-a-collection-in-the-cosmos-db-account"></a>在 Cosmos DB 帐户中创建集合

接下来，在 Cosmos DB 帐户中添加数据集合，以便在后续步骤中进行查询。

1. 导航到新创建的 Cosmos DB 帐户。
2. 在“概览”选项卡中单击“+/添加集合”按钮，此时“添加集合”面板就会滑出。
3. 为集合提供数据库 ID、集合 ID，选择存储容量，输入分区键，输入吞吐量值，然后单击“确定”。  就本教程来说，使用“测试”作为数据库 ID 和集合 ID，选择固定的存储容量和最低吞吐量（400 RU/秒）就可以了。

## <a name="grant-your-user-assigned-msi-access-to-the-cosmos-db-account-access-keys"></a>向用户分配的 MSI 授予访问 Cosmos DB 帐户访问密钥的权限

Cosmos DB 原本不支持 Azure AD 身份验证。  但是，可以使用 MSI 从资源管理器检索 Cosmos DB 访问密钥，然后使用该密钥访问 Cosmos DB。  这一步向用户分配的 MSI 授予访问 Cosmos DB 帐户密钥的权限。

首先，向 MSI 标识授予在 Azure 资源管理器中使用 Azure CLI 访问 Cosmos DB 帐户的权限。 为环境更新 `<SUBSCRIPTION ID>`、`<RESOURCE GROUP>`、`<COSMOS DB ACCOUNT NAME>` 的值。 将 `<MSI PRINCIPALID>` 替换为在[创建用户分配的 MSI](#create-a-user-assigned-msi) 中由 `az identity create` 命令返回的 `principalId` 属性。  Cosmos DB 在使用访问密钥时支持两种级别的粒度：对帐户的读/写访问权限，以及对帐户的只读访问权限。  如果需要获取帐户的读/写密钥，请分配 `DocumentDB Account Contributor` 角色；如果需要获取帐户的只读密钥，请分配 `Cosmos DB Account Reader Role` 角色：

```azurecli-interactive
az role assignment create --assignee <MSI PRINCIPALID> --role '<ROLE NAME>' --scope "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMODS DB ACCOUNT NAME>"
```

响应包括所创建的角色分配的详细信息：

```
{
  "id": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS DB ACCOUNT>/providers/Microsoft.Authorization/roleAssignments/5b44e628-394e-4e7b-bbc3-d6cd4f28f15b",
  "name": "5b44e628-394e-4e7b-bbc3-d6cd4f28f15b",
  "properties": {
    "principalId": "c0833082-6cc3-4a26-a1b1-c4b5f90a981f",
    "roleDefinitionId": "/subscriptions/<SUBSCRIPTION ID>/providers/Microsoft.Authorization/roleDefinitions/fbdf93bf-df7d-467e-a4d2-9458aa1360c8",
    "scope": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS DB ACCOUNT>"
  },
  "resourceGroup": "<RESOURCE GROUP>",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

## <a name="get-an-access-token-using-the-user-assigned-msi-and-use-it-to-call-azure-resource-manager"></a>使用用户分配的 MSI 获取访问令牌，并使用它调用 Azure 资源管理器

至于本教程的剩余部分，请从先前创建的 VM 入手。

若要完成这些步骤，需要使用 SSH 客户端。 如果使用的是 Windows，可以在[适用于 Linux 的 Windows 子系统](https://msdn.microsoft.com/commandline/wsl/install_guide)中使用 SSH 客户端。 如果需要有关配置 SSH 客户端密钥的帮助，请参阅[如何在 Azure 上将 SSH 密钥与 Windows 配合使用](../../virtual-machines/linux/ssh-from-windows.md)或[如何创建和使用适用于 Azure 中 Linux VM 的 SSH 公钥和私钥对](../../virtual-machines/linux/mac-create-ssh-keys.md)。

1. 在 Azure 门户中，导航到“虚拟机”，转到 Linux 虚拟机，然后在“概述”页中单击顶部的“连接”。 复制用于连接到 VM 的字符串。 
2. 使用 SSH 客户端连接到 VM。  
3. 接下来，系统会提示你输入创建“Linux VM”时添加的“密码”。 然后应可以成功登录。  
4. 使用 CURL 获取 Azure 资源管理器的访问令牌。  

    下面是针对访问令牌的 CURL 请求和响应。  将 <CLIENT ID> 替换为用户分配的 MSI 的 clientId 值：
    
    ```bash
    curl 'http://localhost:50342/oauth2/token?resource=https://management.azure.com/&client_id=<CLIENT ID>' -H "Metadata:true"
    ```
    
    > [!NOTE]
    > 在上面的请求中，“resource”参数的值必须与 Azure AD 预期的值完全一致。 如果使用 Azure 资源管理器资源 ID，必须在 URI 的结尾添加斜线。
    > 在下面的响应中，为简洁起见，已缩短了 access_token 元素。
    
    ```bash
    {"access_token":"eyJ0eXAiOi...",
     "expires_in":"3599",
     "expires_on":"1518503375",
     "not_before":"1518499475",
     "resource":"https://management.azure.com/",
     "token_type":"Bearer",
     "client_id":"1ef89848-e14b-465f-8780-bf541d325cd5"}
     ```
    
## <a name="get-access-keys-from-azure-resource-manager-to-make-cosmos-db-calls"></a>从 Azure 资源管理器中获取访问密钥，以便进行 Cosmos DB 调用  

现在，使用在上一部分检索到的访问令牌通过 CURL 调用资源管理器，以便检索 Cosmos DB 帐户访问密钥。 有了访问密钥以后，即可查询 Cosmos DB。 请务必将 `<SUBSCRIPTION ID>`、`<RESOURCE GROUP>` 和 `<COSMOS DB ACCOUNT NAME>` 参数值替换为你自己的值。 将 `<ACCESS TOKEN>` 值替换为前面检索到的访问令牌。  若要检索读/写密钥，请使用密钥操作类型 `listKeys`。  若要检索只读密钥，请使用密钥操作类型 `readonlykeys`：

```bash 
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS DB ACCOUNT NAME>/<KEY OPERATION TYPE>?api-version=2016-03-31' -X POST -d "" -H "Authorization: Bearer <ACCESS TOKEN>" 
```

> [!NOTE]
> 上述 URL 中的文本区分大小写，因此如果对资源组使用了大小写格式，请务必在 URL 中相应地体现出来。 另外，请注意，这是 POST 请求而不是 GET 请求，请务必使用 -d 来传递一个值，以捕获长度限制，此值可以为 NULL。  

CURL 响应提供一个密钥列表。  例如，如果获取只读密钥：  

```bash 
{"primaryReadonlyMasterKey":"bWpDxS...dzQ==",
"secondaryReadonlyMasterKey":"38v5ns...7bA=="}
```

有了 Cosmos DB 帐户的访问密钥以后，即可将其传递给 Cosmos DB SDK 并通过调用来访问该帐户。  如需快速示例，可将该访问密钥传递给 Azure CLI。  在 Azure 门户中，可以从 Cosmos DB 帐户边栏选项卡上的“概览”选项卡获取 <COSMOS DB CONNECTION URL>。  将 <ACCESS KEY> 替换为在上面获取的值：

```bash
az cosmosdb collection show -c <COLLECTION ID> -d <DATABASE ID> --url-connection "<COSMOS DB CONNECTION URL>" --key <ACCESS KEY>
```

此 CLI 命令返回有关集合的详细信息：

```bash
{
  "collection": {
    "_conflicts": "conflicts/",
    "_docs": "docs/",
    "_etag": "\"00006700-0000-0000-0000-5a8271e90000\"",
    "_rid": "Es5SAM2FDwA=",
    "_self": "dbs/Es5SAA==/colls/Es5SAM2FDwA=/",
    "_sprocs": "sprocs/",
    "_triggers": "triggers/",
    "_ts": 1518498281,
    "_udfs": "udfs/",
    "id": "Test",
    "indexingPolicy": {
      "automatic": true,
      "excludedPaths": [],
      "includedPaths": [
        {
          "indexes": [
            {
              "dataType": "Number",
              "kind": "Range",
              "precision": -1
            },
            {
              "dataType": "String",
              "kind": "Range",
              "precision": -1
            },
            {
              "dataType": "Point",
              "kind": "Spatial"
            }
          ],
          "path": "/*"
        }
      ],
      "indexingMode": "consistent"
    }
  },
  "offer": {
    "_etag": "\"00006800-0000-0000-0000-5a8271ea0000\"",
    "_rid": "f4V+",
    "_self": "offers/f4V+/",
    "_ts": 1518498282,
    "content": {
      "offerIsRUPerMinuteThroughputEnabled": false,
      "offerThroughput": 400
    },
    "id": "f4V+",
    "offerResourceId": "Es5SAM2FDwA=",
    "offerType": "Invalid",
    "offerVersion": "V2",
    "resource": "dbs/Es5SAA==/colls/Es5SAM2FDwA=/"
  }
}
```

## <a name="next-steps"></a>后续步骤

- 有关 MSI 的概述，请参阅 [Azure 资源的托管服务标识 (MSI)](msi-overview.md)。

使用以下评论部分提供反馈，帮助我们改进内容。