---
title: "如何删除 HDInsight 群集 - Azure | Microsoft Docs"
description: "删除 HDInsight 群集的各种方式的相关信息。"
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
ms.assetid: 55f7838b-9786-47ff-96db-1b64437bd0bb
ms.service: hdinsight
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 01/17/2018
ms.author: larryfr
ms.custom: H1Hack27Feb2017,hdinsightactive
ms.openlocfilehash: 04b89b2cceb18a0e3c88d0d1deada1a05b8187f6
ms.sourcegitcommit: f1c1789f2f2502d683afaf5a2f46cc548c0dea50
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/18/2018
---
# <a name="delete-an-hdinsight-cluster-using-your-browser-powershell-or-the-azure-cli"></a>使用浏览器、PowerShell 或 Azure CLI 删除 HDInsight 群集

创建群集后便开始 HDInsight 群集计费，删除群集后停止计费。 群集以每分钟按比例收费，因此无需再使用群集时，应始终将其删除。 本文档介绍如何使用 Azure 门户、Azure PowerShell 和 Azure CLI 1.0 删除群集。

> [!IMPORTANT]
> 删除 HDInsight 群集时不会删除 Azure 存储帐户或与群集关联的 Data Lake Store。 可重新使用以后存储在这些服务中的数据。

## <a name="azure-portal"></a>Azure 门户

1. 登录 [Azure 门户](https://portal.azure.com)，并选择 HDInsight 群集。 如果 HDInsight 群集未固定到仪表板，可使用搜索字段按名称进行搜索。
   
    ![门户搜索](./media/hdinsight-delete-cluster/navbar.png)

2. 在群集设置中，选择“删除”图标。 出现提示时，选择“是”即可删除该群集。
   
    ![删除图标](./media/hdinsight-delete-cluster/deletecluster.png)

## <a name="azure-powershell"></a>Azure PowerShell

在 PowerShell 提示符处，使用以下命令删除群集：

    Remove-AzureRmHDInsightCluster -ClusterName CLUSTERNAME

将 **CLUSTERNAME** 替换为 HDInsight 群集名。

## <a name="azure-cli-10"></a>Azure CLI 1.0

在提示符处，使用以下命令删除群集：

    azure hdinsight cluster delete CLUSTERNAME

将 **CLUSTERNAME** 替换为 HDInsight 群集名。

> [!NOTE]
> Azure CLI 2.0 暂不支持删除 HDInsight 群集（2017 年 10 月 23 日）。