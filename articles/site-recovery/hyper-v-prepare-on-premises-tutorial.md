---
title: "准备本地 Hyper-V 服务器用于将 Hyper-V VM 灾难恢复到 Azure| Microsoft 文档"
description: "了解如何使用 Azure Site Recovery 服务，为到 Azure 的灾难恢复准备并非由 System Center VMM 托管的本地 Hyper-V VM。"
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: article
ms.date: 02/14/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 9524ffde4a588d3ac029bc8a3df91726082e157d
ms.sourcegitcommit: d1f35f71e6b1cbeee79b06bfc3a7d0914ac57275
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/22/2018
---
# <a name="prepare-on-premises-hyper-v-servers-for-disaster-recovery-to-azure"></a>准备本地 Hyper-V 服务器用于灾难恢复到 Azure

本教程将介绍想要将 Hyper-V VM 复制到 Azure 时，如何为灾难恢复准备本地 Hyper-V 基础结构。 Hyper-V 主机可以由 System Center Virtual Machine Manager (VMM) 进行托管，但这不是必需的。  本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 查看 Hyper-V 要求和 VMM 要求（如果适用）。
> * 准备 VMM（如果适用）
> * 验证对 Azure 位置的 Internet 访问
> * 准备 VM，以便可以在故障转移到 Azure 后访问它们

这是教程系列中的第二个教程。 请确保你已[设置 Azure 组件](tutorial-prepare-azure.md)，如上一个教程中所述。



## <a name="review-server-requirements"></a>查看服务器要求

确保 Hyper-V 主机满足以下要求。 如果要在 System Center Virtual Machine Manager (VMM) 云中管理主机，请验证 VMM 要求。


**组件** | **由 VMM 托管的 Hyper-V** | **不包含 VMM 的 Hyper-V**
--- | --- | ---
**Hyper-V 主机操作系统** | Windows Server 2016、2012 R2 | 不可用
**VMM** | VMM 2012、VMM 2012 R2 | 不可用


## <a name="review-hyper-v-vm-requirements"></a>查看 Hyper-V VM 要求

请确保 VM 符合表中总结的要求。

**VM 要求** | **详细信息**
--- | ---
**来宾操作系统** | [Azure 支持的](https://technet.microsoft.com/library/cc794868.aspx)任何来宾操作系统。
**Azure 要求** | 本地 Hyper-V VM 必须满足 Azure VM 要求 (site-recovery-support-matrix-to-azure.md)。

## <a name="prepare-vmm-optional"></a>准备 VMM（可选）

如果 Hyper-V 主机由 VMM 管理，则需准备本地 VMM 服务器。 

- 请确保 VMM 服务器至少有一个云，包含一个或多个主机组。 运行在虚拟机上的 Hyper-V 主机应位于云中。
- 准备 VMM 服务器以用于网络映射。

### <a name="prepare-vmm-for-network-mapping"></a>为网络映射准备 VMM

如果使用 VMM，则[网络映射](site-recovery-network-mapping.md)在本地 VMM VM 网络和 Azure 虚拟网络之间进行映射。 映射将确保在故障转移后创建 Azure VM 时，将其连接到正确的网络。

为网络映射准备 VMM，如下所示：

1. 确保有与 Hyper-V 主机所在的云相关联的 [VMM 逻辑网络](https://docs.microsoft.com/system-center/vmm/network-logical)。
2. 确保有链接到逻辑网络的[虚拟机网络](https://docs.microsoft.com/system-center/vmm/network-virtual)。
3. 在 VMM 中，将 VM 连接到 VM 网络。

## <a name="verify-internet-access"></a>验证 Internet 访问

1. 对于本教程，最简单的配置是 Hyper-V 主机和 VMM 服务器（如果适用）能够直接访问 Internet 而无需使用代理。 
2. 请确保该 Hyper-V 主机和 VMM 服务器（如果相关）可以访问以下 URL： 

    [!INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)]
    
3. 请确保：
    - 任何基于 IP 地址的防火墙规则都应允许与 Azure 通信。
    - 允许 [Azure 数据中心 IP 范围](https://www.microsoft.com/download/confirmation.aspx?id=41653)和 HTTPS (443) 端口。
    - 允许订阅的 Azure 区域的 IP 地址范围以及美国西部的 IP 地址范围（用于访问控制和标识管理）。


## <a name="prepare-to-connect-to-azure-vms-after-failover"></a>准备在故障转移后连接到 Azure VM

在实施故障转移方案期间，可能需要连接到已复制的本地网络。

若要在故障转移后使用 RDP 连接到 Windows VM，请执行以下操作：

1. 若要通过 Internet 访问，请在故障转移之前在本地 VM 上启用 RDP。 请确保已针对“公共”配置文件添加了 TCP 和 UDP 规则，并确保在“Windows 防火墙” > “允许的应用”中针对所有配置文件允许 RDP。
2. 若要通过站点到站点 VPN 进行访问，请在本地计算机上启用 RDP。 应在“Windows 防火墙” -> “允许的应用和功能”中针对“域和专用”网络允许 RDP。
   检查操作系统的 SAN 策略是否已设置为 OnlineAll。 [了解详细信息](https://support.microsoft.com/kb/3031135)。 触发故障转移时，VM 上不应存在待处理的 Windows 更新。 如果存在，则在更新完成之前无法登录到虚拟机。
3. 在 Windows Azure VM 上执行故障转移后，请选中“启动诊断”查看 VM 的屏幕截图。 如果无法连接，请检查 VM 是否正在运行，并查看这些[疑难解答提示](http://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx)。


## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [为 Hyper-V VM 设置到 Azure 的灾难恢复](tutorial-hyper-v-to-azure.md)
> [为 VMM 云中的 Hyper-V VM 设置到 Azure 的灾难恢复](tutorial-hyper-v-vmm-to-azure.md)
