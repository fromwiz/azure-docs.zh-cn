---
title: "什么是 Azure 时序见解？ | Microsoft Docs"
description: "简要介绍 Azure 时序见解，这是一种用于时序数据分析和 IoT 解决方案的新服务。"
services: time-series-insights
ms.service: time-series-insights
author: ashannon7
ms.author: anshan, jasonh
manager: jhubbard
editor: MarkMcGeeAtAquent
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.workload: big-data
ms.topic: article
ms.date: 01/26/2018
ms.openlocfilehash: e31cebfd027e93096e233f2963445e4fc50a7e9d
ms.sourcegitcommit: 9d317dabf4a5cca13308c50a10349af0e72e1b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/01/2018
---
# <a name="what-is-azure-time-series-insights"></a>什么是 Azure 时序见解？

时序见解是用于存储、可视化和查询大量时序数据（例如 IoT 设备所生成的数据）的新服务。  如果你想要在云中存储、管理、查询或可视化时序数据，则时序见解可能会很适合你。  

![Time Series Insights flowchart] (media/overview/time-series-insights-flowchart.png)

时序见解包含四个关键作业：

- 首先，它与 Azure IoT 中心和 Azure 事件中心等云网关完全集成。 它可以轻松连接到这些事件源，并根据消息和结构（在干净的行和列中包含数据）分析 JSON。 它将元数据与遥测数据联接，并在纵栏式存储中对数据编制索引。
- 其次，时序见解管理数据存储。 为确保始终能够轻松访问数据，它将数据存储在内存和 SSD 中长达 400 天。 你可以在数秒内以交互方式按需查询数以亿计的事件。
- 第三，时序见解通过 TSI 资源管理器提供全新的可视化效果。  
- 第四，时序见解通过两种途径提供查询服务：在 TSI 资源管理器中，以及使用可轻松集成以便将时序数据嵌入自定义应用程序的 API。  

如果你正在构建应用程序，供内部使用或供外部客户使用，则可以将时序见解用作索引、存储和聚合时序数据的后端。 你可以在此之上构建自定义可视化效果和用户体验。  时序见解会公开查询 API 来启用此方案。  

如果你不确定你的数据是否为时序的，请参考下文将介绍的一些内容。  时序数据表示资产或过程是如何随时间变化的。  它的独特之处在于它有一个时间戳，而时间是最有意义的轴。  时序数据通常按时间顺序到达，通常被视为插入，而不是数据库的更新。  因为时序见解会捕获每一个新事件并储存为一行，所以更改是随着时间进行度量的，让你可以回顾及预测将来的更改。  在大型卷中，存储、索引、查询、分析和可视化时序数据可能很具有挑战性。  

## <a name="primary-scenarios"></a>主要方案

- 以可缩放方式存储时序数据。  
  - 在其核心，时序见解有一个数据库，是专门围绕时序数据而设计的。  因为它是可缩放且完全托管，所以时序见解负责处理存储和管理事件。

- 近实时的数据浏览。  
  - 时序见解提供了一个资源管理器，用于可视化所有流入环境的数据。  连接事件源后，很快就可以在时序见解中查看、浏览和查询事件数据。  数据可用于验证设备是否按预期方式送出数据并监视 IoT 资产的运行状况、生产效率和总体效率。  

- 根本原因分析和异常情况检测。
  - 时序见解具有模式和透视视图等工具，用于执行和保存多步骤根本原因分析。  此外，时序见解与 Azure 流分析等警报服务结合使用，以便可以在时序见解资源管理器中近实时地查看警报和检测到的异常情况。  

- 来自不同位置的时序数据流的全局视图，用于进行多资产/站点的比较。
  - 你可以将多个事件源连接到时序见解环境。  这意味着，可以近实时地查看来自多个不同位置的数据流。  用户可以利用这种可见性与业务领导者共享数据并与域专家更好地协作，他们可以使用其专业技能帮助你解决问题、应用最佳实践和共享知识。

- 在时序见解的基础上构建客户应用程序。 
  - 时序见解会公开 REST 查询 API，让你能够构建使用时序数据的应用程序。

## <a name="capabilities"></a>功能

- 快速入门：Azure 时序见解无需前期数据准备。 只需数分钟即可连接到 Azure IoT 中心或事件中心的数百万个事件。 一旦连接，即可快速可视化传感器数据并与之交互，从而快速验证 IoT 解决方案。 无需编写代码即可与你的数据交互。
不需要学习新语言；时序见解既为高级用户提供精细的自定义文本查询图面，又提供点击浏览体验。
- 近实时见解：时序见解每天可以引入数百万个传感器事件，只有一分钟的延迟。 可以通过时序见解洞察各种趋势和异常，对传感器数据进行分析，并执行根本原因分析，避免代价高昂的停机。 时序见解可以在实时数据和历史数据之间进行交叉关联，有助于发现数据中隐藏的趋势。
- 生成自定义解决方案：将 Azure 时序见解数据嵌入现有应用程序，或通过时序见解 REST API 创建新的自定义解决方案。 创建个性化视图，以便通过这种共享方便他人浏览你的见解。
- 可伸缩性：时序见解旨在支持大规模 IoT。 时序见解每天可以引入 100 万到 1 亿个事件，默认保留时间为 31 天。 可以通过近实时方式可视化和分析实时数据流以及历史数据。 在将来，引入率和保留率将会提高，以便适应企业规模。

## <a name="getting-started"></a>入门
入门时间不超过 5 分钟。 

1.  若要开始使用，请在 Azure 门户中预配一个时序见解环境。 
2.  连接 Azure IoT 中心或事件中心等事件源。  
3.  上传参考数据（这不是一项附加服务）。
4.  使用时序见解资源管理器，几分钟内就可以看到你的数据。

## <a name="time-series-insights-explorer"></a>时序见解资源管理器
此图表显示了通过资源管理器查看时序见解数据的示例：![Time Series Insights explorer] (media/time-series-insights-explorer/explorer4.png)

## <a name="next-steps"></a>后续步骤
 - [在演示环境中使用时序见解资源管理器浏览](./time-series-quickstart.md)
 - [计划自己的时序见解环境](time-series-insights-environment-planning.md)

