---
title: "为 SQL 数据库多租户应用程序使用 Log Analytics | Microsoft Docs"
description: "通过多租户 Azure SQL 数据库 SaaS 应用设置和使用 Log Analytics (OMS)"
keywords: "sql 数据库教程"
services: sql-database
documentationcenter: 
author: stevestein
manager: craigg
editor: 
ms.assetid: 
ms.service: sql-database
ms.custom: scale out apps
ms.workload: Inactive
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2017
ms.author: billgib; sstein
ms.openlocfilehash: 48e8eb91a5febcc1109bee3404bb534bd0391f88
ms.sourcegitcommit: f847fcbf7f89405c1e2d327702cbd3f2399c4bc2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/28/2017
---
# <a name="set-up-and-use-log-analytics-oms-with-a-multi-tenant-azure-sql-database-saas-app"></a>通过多租户 Azure SQL 数据库 SaaS 应用设置和使用 Log Analytics (OMS)

在本教程中，将设置和使用 *Log Analytics ([OMS](https://www.microsoft.com/cloud-platform/operations-management-suite))* 来监视弹性池和数据库。 本教程基于[性能监视和管理教程](saas-dbpertenant-performance-monitoring.md)。 它演示如何使用 Log Analytics 加强 Azure 门户中提供的监视和警报。 Log Analytics 适合大规模监视和警报，因为它支持数百个池以及成千上万的数据库。 本教程还提供单个监视解决方案，该方案可以集成跨多个 Azure 订阅监视不同应用程序和 Azure 服务的功能。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 安装和配置 Log Analytics (OMS)
> * 使用 Log Analytics 监视池和数据库

若要完成本教程，请确保已完成了以下先决条件：

* 已部署 Wingtip Tickets SaaS Database Per Tenant 应用。 若要在五分钟内完成部署，请参阅[部署和浏览 Wingtip Tickets SaaS Database Per Tenant 应用程序](saas-dbpertenant-get-started-deploy.md)
* Azure PowerShell 已安装。 有关详细信息，请参阅 [Azure PowerShell 入门](https://docs.microsoft.com/powershell/azure/get-started-azureps)

请参阅[“性能监视和管理”教程](saas-dbpertenant-performance-monitoring.md)，了解 SaaS 方案和模式及其对解决方案监视要求的影响。

## <a name="monitoring-and-managing-performance-with-log-analytics-oms"></a>通过 Log Analytics (OMS) 监视和管理性能

对于 SQL 数据库来说，监视和警报功能可以在数据库和池上使用。 这种内置的监视和警报功能是特定于资源的，可以方便地用于少量资源，但不是很适合监视大型安装，也不是很适合跨不同的资源和订阅提供统一的视图。

对于大容量方案，可以使用 Log Analytics。 这是单独的 Azure 服务，可以针对发出的诊断日志以及在分析工作区中收集的遥测进行分析，该工作区可以收集多个服务的遥测，并且可以用来查询和设置警报。 Log Analytics 提供内置的查询语言和数据可视化工具，适用于操作数据分析和可视化。 SQL Analytics 解决方案提供多个预定义的弹性池和数据库监视和警报视图与查询，允许根据需要添加自己的临时查询并进行保存。 OMS 还提供自定义视图设计器。

Log Analytics 工作区和分析解决方案在 Azure 门户和 OMS 中都可以打开。 Azure 门户是更新的访问点，但在某些区域可能会在时间上落后于 OMS 门户。

### <a name="create-data-by-starting-the-load-generator"></a>通过启动负载生成器创建数据 

1. 在 PowerShell ISE 中打开 Demo-PerformanceMonitoringAndManagement.ps1。 请让此脚本保持打开状态，因为可能需要运行此教程中的多个负载生成方案。
1. 如果租户不到五个，请预配一批租户，以便提供更微妙的监视上下文：
   1. 设置 **$DemoScenario = 1**，**预配一批租户**
   1. 若要运行脚本，请按 F5。

1. 设置 $DemoScenario = 2，生成正常强度负载（约 40 DTU）。
1. 若要运行脚本，请按 F5。

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>获取 Wingtip Tickets SaaS Database Per Tenant 应用程序的脚本

在 [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant) Github 存储库中提供了 Wingtip Tickets SaaS 多租户数据库脚本和应用程序源代码。 有关下载和取消阻止 Wingtip Tickets SaaS 脚本的步骤，请参阅[常规指南](saas-tenancy-wingtip-app-guidance-tips.md)。

## <a name="installing-and-configuring-log-analytics-and-the-azure-sql-analytics-solution"></a>安装和配置 Log Analytics 和 Azure SQL Analytics 解决方案

Log Analytics 是一种需要配置的单独服务。 Log Analytics 在日志分析工作区中收集日志数据和遥测以及指标。 工作区是一种资源，就像 Azure 中的其他资源一样，必须创建。 虽然不要求创建的工作区和其所监视的应用程序位于同一资源组中，但通常情况下，将二者置于同一资源组中是最合理的。 对于 Wingtip Tickets SaaS Database Per Tenant SaaS 应用，这样配置时，只需删除资源组即可删除工作区和应用程序。

1. 在 PowerShell ISE 中打开 ...\\Learning Modules\\Performance Monitoring and Management\\Log Analytics\\Demo-LogAnalytics.ps1。
1. 若要运行脚本，请按 F5。

此时应可在 Azure 门户（或 OMS 门户）中打开 Log Analytics。 在 Log Analytics 工作区中收集遥测以及让其变得可见需要数分钟。 系统收集数据的时间越长，体验越微妙。 现在不妨休息一下，喝喝饮料 - 只需确保负载生成器仍在运行即可！


## <a name="use-log-analytics-and-the-sql-analytics-solution-to-monitor-pools-and-databases"></a>使用 Log Analytics 和 SQL Analytics 解决方案监视池和数据库


在本练习中，打开 Log Analytics 和 OMS 门户，查看为数据库和池收集的遥测数据。

1. 浏览到 [Azure 门户](https://portal.azure.com)，并通过单击“更多服务”并搜索“Log Analytics”打开 Log Analytics：

   ![打开 Log Analytics](media/saas-dbpertenant-log-analytics/log-analytics-open.png)

1. 选择名为 wtploganalytics-&lt;USER&gt; 的工作区。

1. 选择“概览”，在 Azure 门户中打开 Log Analytics 解决方案。
   ![overview-link](media/saas-dbpertenant-log-analytics/click-overview.png)

    > [!IMPORTANT]
    > 可能需要数分钟才能激活解决方案。 请耐心等待！

1. 单击“Azure SQL Analytics”磁贴将其打开。

    ![概览](media/saas-dbpertenant-log-analytics/overview.png)

    ![分析](media/saas-dbpertenant-log-analytics/analytics.png)

1. 解决方案边栏选项卡中的视图可以侧向滚动，其自身的滚动条位于底部（请根据需要刷新该边栏选项卡）。

1. 浏览各个视图的方法是单击相应的视图或各个资源，从而打开一个可以深入查看的浏览器，可以在其中使用左上角的时间滑块，或者单击某个垂直条，以便专注于更狭窄的时间片。 使用此视图时，可以选择单个数据库或池，以便专注于特定的资源：

    ![图表](media/saas-dbpertenant-log-analytics/chart.png)

1. 回到解决方案边栏选项卡，此时如果滚动到最右端，则会看到一些保存的查询，单击这些查询即可将其打开并进行浏览。 可以试着修改这些查询，并保存所生成的感兴趣的查询，随后再重新打开它们，将其与其他资源配合使用。

1. 回到 Log Analytics 工作区边栏选项卡，此时请选择 OMS 门户，在其中打开解决方案。

    ![oms](media/saas-dbpertenant-log-analytics/oms.png)

1. 在 OMS 门户中，可以配置警报。 单击数据库 DTU 视图的警报部分。

1. 在显示的“日志搜索”视图中，可以看到所代表指标的条形图。

    ![日志搜索](media/saas-dbpertenant-log-analytics/log-search.png)

1. 如果在工具栏中单击“警报”，则可看到警报配置，并可对其进行更改。

    ![添加警报规则](media/saas-dbpertenant-log-analytics/add-alert.png)

Log Analytics 和 OMS 中的监视和警报功能基于工作区中的数据查询，不像每个资源边栏选项卡上的警报功能，后者是特定于资源的。 因此，可以定义一个监视所有数据库的警报，而不必每个数据库定义一个。 也可以编写一个警报，对多种资源类型使用复合查询。 查询仅受工作区中提供的数据的限制。

适用于 SQL 数据库的 Log Analytics 按工作区中的数据量计费。 在本教程中，创建了一个免费的工作区，其限制是每天 500MB。 达到该限制后，不再向工作区添加数据。


## <a name="next-steps"></a>后续步骤

本教程介绍了如何：

> [!div class="checklist"]
> * 安装和配置 Log Analytics (OMS)
> * 使用 Log Analytics 监视池和数据库

[“租户分析”教程](saas-dbpertenant-log-analytics.md)

## <a name="additional-resources"></a>其他资源

* [构建初始 Wingtip Tickets SaaS Database Per Tenant 应用程序部署的其他教程](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [Azure Log Analytics](../log-analytics/log-analytics-azure-sql.md)
* [OMS](https://blogs.technet.microsoft.com/msoms/2017/02/21/azure-sql-analytics-solution-public-preview/)
