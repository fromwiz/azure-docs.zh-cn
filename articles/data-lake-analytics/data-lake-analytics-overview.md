---
title: "Microsoft Azure Data Lake Analytics 概述 | Microsoft Docs"
description: "Data Lake Analytics 是一种 Azure 大数据服务，允许用户使用各种数据，借助基于云中数据收集的见解推动企业的发展，而不用考虑数据的大小和位置。"
services: data-lake-analytics
documentationcenter: 
author: saveenr
manager: saveenr
editor: cgronlun
ms.assetid: 1e1d443a-48a2-47fb-bc00-bf88274222de
ms.service: data-lake-analytics
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 06/23/2017
ms.author: saveenr
ms.openlocfilehash: 316c35fa4b04bdb251c2b9ae14f6b32f4e4bf939
ms.sourcegitcommit: 0b02e180f02ca3acbfb2f91ca3e36989df0f2d9c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/05/2018
---
# <a name="overview-of-microsoft-azure-data-lake-analytics"></a>Microsoft Azure Data Lake Analytics 概述

## <a name="what-is-azure-data-lake-analytics"></a>什么是 Azure Data Lake Analytics？
Azure Data Lake Analytics 是一项按需分析作业服务，用于简化大数据。 无需部署、配置和调整硬件，只需编写查询即可转换数据并提取有价值的见解。 通过将表盘设置为所需值，该分析服务就可以立即处理任何规模的作业。 只需为运行作业付费，让服务变得更为经济高效。 此分析服务支持 U-SQL，这是一种有效结合了 SQL 的优点和强制性代码的功能的语言。 U-SQL 允许跨 Data Lake Store、Azure 中的 SQL Server、Azure SQL 数据库和 Azure SQL 数据仓库来分析数据。

## <a name="dynamic-scaling"></a>动态缩放
  
Data Lake Analytics 是针对云缩放和性能需求进行构建的。  它能动态地预配资源并让你以千吉字节甚至百亿亿字节为单位进行分析。 当作业完成时，它自动释放资源，仅需为所用的处理功能付费。 增加或减少存储数据的大小或使用的计算资源量时，无需重写代码。 用户可仅关注自己的业务逻辑，而非如何处理和存储大数据集。

## <a name="develop-faster-debug-and-optimize-smarter-using-familiar-tools"></a>使用熟悉的工具更快地进行开发、更智能地进行调试和优化
  
Data Lake Analytics 与 Visual Studio 深度集成，从而你可以使用熟悉的工具运行、调试和调整代码。 U-SQL 作业可视化允许看见代码如何大规模运行，因此可以轻松找到性能瓶颈并优化成本。


## <a name="u-sql-simple-and-familiar-powerful-and-extensible"></a>U-SQL：简单熟悉、功能强大且易于扩展
  
Data Lake Analytics 包含 U-SQL，这是一种查询语言，扩展了 SQL 的简单熟悉的声明性本质和 C# 的表现力。 U-SQL 语言基于在 Microsoft 内部支持大数据系统的同一分布式运行时。 现在，数以百万计的 SQL 和 .NET 开发人员可以凭借自身已有的技能处理和分析自己的数据。

## <a name="integrates-seamlessly-with-your-it-investments"></a>与 IT 投资无缝集成
  
Data Lake Analytics 可以使用现有的 IT 投资进行识别、管理、安全和数据仓库工作来应对这个挑战。 此方法简化了数据管理，使当前的数据应用程序更容易扩展。 Data Lake Analytics 与适用于用户管理和权限的 Active Directory 集成且随附内置监视与审核功能。

## <a name="affordable-and-cost-effective"></a>价格合理且经济高效

Data Lake Analytics 是用于运行大数据工作负荷的经济高效的解决方案。 处理数据时按每个作业付费。 无需硬件、许可证或服务特定的支持协议。 作业开始和完成时，系统自动缩放大小，所以你永远无需为你所需之外的东西付费。 [详细了解如何控制成本和节省资金](https://1drv.ms/f/s!AvdZLquGMt47h213Hg3rhl-Tym1c)。
    
## <a name="works-with-all-your-azure-data"></a>可用于所有 Azure 数据
  
Data Lake Analytics 适用于 Azure Data Lake Store，可以最大程度地提高性能、吞吐量和并行化；同时还适用于 Azure 存储 Blob、Azure SQL 数据库、Azure 仓库。

## <a name="next-steps"></a>后续步骤
 
  * 通过 [Azure 门户](data-lake-analytics-get-started-portal.md) | [Azure PowerShell](data-lake-analytics-get-started-powershell.md) | [CLI](data-lake-analytics-get-started-cli2.md) 使用 Data Lake Analytics 入门
  * 使用 [Azure 门户](data-lake-analytics-manage-use-portal.md) | [Azure PowerShell](data-lake-analytics-manage-use-powershell.md) | [CLI](data-lake-analytics-manage-use-cli.md) | [Azure .NET SDK](data-lake-analytics-manage-use-dotnet-sdk.md) | [Node.js](data-lake-analytics-manage-use-nodejs.md) 管理 Azure Data Lake Analytics
  * [如何使用 Data Lake Analytics 控制成本和节省资金](https://1drv.ms/f/s!AvdZLquGMt47h213Hg3rhl-Tym1c)
