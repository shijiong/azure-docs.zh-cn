---
title: "分组计算机以便使用 Azure Migrate 进行评估 | Microsoft 文档"
description: "介绍如何在使用 Azure Migrate 服务运行评估之前分组计算机。"
author: rayne-wiselman
ms.service: azure-migrate
ms.devlang: na
ms.topic: article
ms.date: 12/12/2017
ms.author: raynew
ms.openlocfilehash: 429a9150d1fbf50c0e3fa2046eb64affc8db8e5d
ms.sourcegitcommit: aaba209b9cea87cb983e6f498e7a820616a77471
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/12/2017
---
# <a name="group-machines-for-assessment"></a>分组将计算机以进行评估

本文介绍如何创建计算机组以使用 [Azure Migrate](migrate-overview.md) 进行评估。 Azure Migrate 可评估组中的计算机，以检查其是否适合迁移到 Azure，并提供在 Azure 中运行计算机的大小和成本估计。


## <a name="create-a-group"></a>创建组

1. 在 Azure Migrate 项目的“仪表板”中，单击“组” >“+ 组”，然后指定组名。 **** ****
2. 将一个或多个计算机添加到组中，然后单击“创建” ****。 
3. 你可以视情况选择为组运行新评估。 

    ![创建组](./media/how-to-create-a-group/create-group.png)

创建组后，你可以在“组”页面上选择组并添加或删除计算机，对其进行修改。

## <a name="next-steps"></a>后续步骤

- 了解如何使用[计算机依赖项映射](how-to-create-group-machine-dependencies.md)创建更详细的组。
- [详细了解](concepts-assessment-calculation.md)如何计算评估。
