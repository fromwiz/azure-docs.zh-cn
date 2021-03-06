---
title: "Azure CLI 脚本示例 - Batch 中的 Windows 池 | Microsoft Docs"
description: "Azure CLI 脚本示例 - 在 Batch 中创建和管理 Windows 池"
services: batch
documentationcenter: 
author: dlepow
manager: jeconnoc
editor: 
ms.assetid: 
ms.service: batch
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 01/29/2018
ms.author: danlep
ms.openlocfilehash: b1fe98169e47c5f5386867cf2b6c23f8bc3bfb65
ms.sourcegitcommit: 9d317dabf4a5cca13308c50a10349af0e72e1b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/01/2018
---
# <a name="cli-example-create-and-manage-a-windows-pool-in-azure-batch"></a>CLI 示例：在 Azure Batch 中创建和管理 Windows 池

此脚本演示了 Azure Batch 中一些可用于在 Azure CLI 中创建和管理 Windows 计算节点池的命令。 可通过两种方式配置 Windows 池：使用云服务配置，或使用虚拟机配置。 此示例介绍如何使用云服务配置来创建 Windows 池。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果选择在本地安装并使用 CLI，本文要求运行 Azure CLI 2.0.20 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI 2.0](/cli/azure/install-azure-cli)。 

## <a name="example-script"></a>示例脚本

[!code-azurecli-interactive[main](../../../cli_scripts/batch/manage-pool/manage-pool-windows.sh "Manage Windows Cloud Services Pool")]

## <a name="clean-up-deployment"></a>清理部署

运行以下命令以删除资源组及其相关的所有资源。

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | 创建用于存储所有资源的资源组。 |
| [az batch account create](/cli/azure/batch/account#az_batch_account_create) | 创建批处理帐户。 |
| [az batch account login](https://docs.microsoft.com/cli/azure/batch/account#az_batch_account_login) | 针对指定的批处理帐户进行身份验证，以便进一步进行 CLI 交互。 |
| [az batch pool create](https://docs.microsoft.com/cli/azure/batch/pool#az_batch_pool_create) | 创建计算节点池。  |
| [az batch pool set](https://docs.microsoft.com/cli/azure/batch/pool#az_batch_pool_set) | 更新池的属性。  |
| [az batch pool autoscale enable](https://docs.microsoft.com/cli/azure/batch/pool/autoscale#az_batch_pool_autoscale_enable) | 对池启用自动缩放并应用公式。  |
| [az batch pool show](https://docs.microsoft.com/cli/azure/batch/pool#az_batch_pool_show) | 显示池的属性。  |
| [az batch pool autoscale disable](https://docs.microsoft.com/cli/azure/batch/pool/autoscale#az_batch_pool_autoscale_disable) | 对池禁用自动缩放。 |
| [az group delete](/cli/azure/group#az_group_delete) | 删除资源组，包括所有嵌套的资源。 |


## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/cli/azure/overview)。
