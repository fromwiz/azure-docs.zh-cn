---
title: "创建内部负载均衡器 - Azure 门户 | Microsoft 文档"
description: "了解如何使用 Azure 门户在 Resource Manager 中创建内部负载均衡器"
services: load-balancer
documentationcenter: na
author: KumudD
manager: jennoc
editor: 
tags: azure-service-management
ms.assetid: 1ac14fb9-8d14-4892-bfe6-8bc74c48ae2c
ms.service: load-balancer
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: kumud
ms.openlocfilehash: 5274ec13ec2d04194e2dd4c8ec93be0f78329b23
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/21/2018
---
# <a name="create-an-internal-load-balancer-in-the-azure-portal"></a>在 Azure 门户中创建内部负载均衡器

> [!div class="op_single_selector"]
> * [Azure portal](../load-balancer/load-balancer-get-started-ilb-arm-portal.md)
> * [PowerShell](../load-balancer/load-balancer-get-started-ilb-arm-ps.md)
> * [Azure CLI](../load-balancer/load-balancer-get-started-ilb-arm-cli.md)
> * [模板](../load-balancer/load-balancer-get-started-ilb-arm-template.md)


[!INCLUDE [load-balancer-basic-sku-include.md](../../includes/load-balancer-basic-sku-include.md)]

[!INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

[!INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

## <a name="get-started-creating-an-internal-load-balancer-using-azure-portal"></a>开始使用 Azure 门户创建内部负载均衡器

使用以下步骤从 Azure 门户创建内部负载均衡器。

1. 打开浏览器，导航到 [Azure 门户](http://portal.azure.com)，并使用 Azure 帐户登录。
2. 在屏幕的左上方，单击“创建资源” > “网络” > “负载均衡器”。
3. 在“创建负载均衡器”边栏选项卡中，输入负载均衡器的“名称”。
4. 在“类型”下，单击“内部”。
5. 单击“虚拟网络”，并选择要在其中创建负载均衡器的虚拟网络。

   > [!NOTE]
   > 如果看不到要使用的虚拟网络，请选中要用于负载均衡器的**位置**，并进行相应的更改。

6. 单击“子网”，并选择要在其中创建负载均衡器的子网。
7. 在“IP 地址分配”下，单击“动态”或“静态”，具体取决于是要固定（静态）负载均衡器的 IP 地址。

   > [!NOTE]
   > 如果选择使用静态 IP 地址，则必须为负载均衡器提供一个地址。

8. 在“资源组”下，为负载均衡器指定新资源组的名称，或者单击“选择现有”，并选择现有资源组。
9. 单击“创建”。

## <a name="configure-load-balancing-rules"></a>配置负载均衡规则

创建负载均衡器之后，导航到负载均衡器资源对其进行配置。
在配置负载均衡规则之前，请配置后端地址池和探测。

### <a name="step-1-configure-a-backend-pool"></a>步骤 1：配置后端池

1. 在 Azure 门户中，单击“浏览” > “负载均衡器”，并单击前面创建的负载均衡器。
2. 在“设置”页中，单击“后端池”。
3. 在“后端地址池”页中，单击“添加”。
4. 在“添加后端池”页中，输入后端池的**名称**，并单击“确定”。

### <a name="step-2-configure-a-probe"></a>步骤 2：配置探测器

1. 在 Azure 门户中，单击“浏览” > “负载均衡器”，并单击前面创建的负载均衡器。
2. 在“设置”页中，单击“运行状况探测”。
3. 在“运行状况探测”页中，单击“添加”。
4. 在“添加运行状况探测”页中，输入探测的**名称**。
5. 在“协议”下，选择“HTTP”（用于网站）或“TCP”（用于其他基于 TCP 的应用程序）。
6. 在“端口”下，指定访问探测器时要使用的端口。
7. 在“路径”下（仅适用于 HTTP 探测器），指定要用作探测器的路径。
8. 在“时间间隔”下，指定探测应用程序的频率。
9. 在“不正常阈值”下，指定应在多少次尝试失败后，将后端虚拟机标记为不正常。
10. 单击“确定”以创建探测器。

### <a name="step-3-configure-load-balancing-rules"></a>步骤 3：配置负载均衡规则

1. 在 Azure 门户中，单击“浏览” > “负载均衡器”，并单击前面创建的负载均衡器。
2. 在“设置”页中，单击“负载均衡规则”。
3. 在“负载均衡规则”页中，单击“添加”。
4. 在“添加负载均衡规则”页中，输入规则的**名称**。
5. 在“协议”下选择“TCP”或“UDP”。
6. 在“端口”下，指定负载均衡器中客户端连接到的端口。
7. 在“后端端口”下，指定要在后端池中使用的端口（通常情况下，负载均衡器端口与后端端口是相同的）。
8. 在“后端池”下，选择前面创建的后端池。
9. 在“会话持久性”下，选择要保留会话的方式。
10. 在“空闲超时(分钟)”下，指定空闲超时。
11. 在“浮动 IP (直接服务器返回)”下，单击“禁用”或“启用”。
12. 单击“确定”。

## <a name="next-steps"></a>后续步骤

[配置负载均衡器分发模式](load-balancer-distribution-mode.md)

[配置负载均衡器的空闲 TCP 超时设置](load-balancer-tcp-idle-timeout.md)

