---
title: "使用 Azure API 管理中的组管理开发人员帐户 | Microsoft 文档"
description: "了解如何使用 Azure API 管理中的组管理开发人员帐户"
services: api-management
documentationcenter: 
author: juliako
manager: cfowler
editor: 
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/30/2017
ms.author: apimpm
ms.openlocfilehash: 1587243bcd5f2b9af98b8b529c152ba49ef676be
ms.sourcegitcommit: b854df4fc66c73ba1dd141740a2b348de3e1e028
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2017
---
# <a name="how-to-create-and-use-groups-to-manage-developer-accounts-in-azure-api-management"></a>如何创建和使用组来管理 Azure API 管理中的开发人员帐户
在 API 管理中，使用组来管理产品对开发人员的可见性。 产品首次对组可见，这些组中的开发人员可以查看和订阅与组关联的产品。 

API 管理具有下列不可变的系统组。

* **管理员** - Azure 订阅管理员是此组的成员。 管理员管理 API 管理服务实例、创建 API、操作，以及开发人员所使用的产品。
* **开发人员** - 已经过身份验证的开发人员门户用户属于此组。 开发人员是使用 API 构建应用程序的客户。 开发人员有权访问开发人员门户，并构建调用 API 操作的应用程序。
* **来宾** - 未经身份验证的开发人员门户用户，如访问此组中 API 管理实例的开发人员门户的潜在客户。 它们可以被授予某些只读访问权限，如能够查看 API，但不能调用它们。

除了这些系统组，管理员还可以创建自定义组或[利用关联 Azure Active Directory 租户中的外部组][leverage external groups in associated Azure Active Directory tenants]。 自定义组和外部组可与系统组一起使用为开发人员提供可见性并可以访问 API 产品。 例如，可以为隶属于一个特定合作伙伴组织的开发人员创建一个自定义组并允许他们从仅包含相关 API 的产品中访问 API。 用户可以是多个组的成员。

本指南演示 API 管理实例的管理员如何添加新组并将它们关联产品和开发人员。

> [!NOTE]
> 除了在发布者门户中创建和管理组，还可使用 API 管理 REST API [组](https://msdn.microsoft.com/library/azure/dn776329.aspx)实体创建和管理组。
> 
> 

## <a name="create-group"> </a>创建组
若要创建新组，请单击 API 管理服务的 Azure 门户中的“发布者门户”。 这会转到 API 管理发布者门户。

![发布者门户][api-management-management-console]

> 如果尚未创建 API 管理服务实例，请参阅[创建 API 管理服务实例][Create an API Management service instance]。
> 
> 

单击左侧“API 管理”菜单中的“组”，并单击“添加组”。

![添加新组][api-management-add-group]

输入组和可选说明的唯一名称，并单击“保存”。

![添加新组][api-management-add-group-window]

新组显示在组选项卡中。若要编辑组的“名称”或“说明”，请单击列表中组的名称。 若要删除组，请单击“删除”。

![添加的组][api-management-new-group]

现在已创建组，它可以与产品和开发人员相关联。

## <a name="associate-group-product"> </a>将组与产品关联
要将某一组与产品相关联，请单击左侧“API 管理”菜单中的“产品”，然后单击所需产品的名称。

![设置可见性][api-management-add-group-to-product]

选择“可见性”选项卡添加和删除组，并查看有关该产品的当前组。 要添加或删除组，请选中或取消选中所需组的复选框，并单击“保存”。

![设置可见性][api-management-add-group-to-product-visibility]

> [!NOTE]
> 若要添加 Azure Active Directory 组，请参阅[如何在 Azure API 管理中使用 Azure Active Directory 授权开发人员帐户](api-management-howto-aad.md)。
> 
> 若要从产品的“可见性”选项卡配置组，请单击“管理组”。
> 
> 

一种产品与组相关联后，该组中的开发人员可以查看和订阅该产品。

## <a name="associate-group-developer"> </a>将组与开发人员关联
要将组与开发人员相关联，请单击左侧“API 管理”中的“用户”，然后选中要和一组相关联的开发人员旁边的框。

![将开发人员添加到组][api-management-add-group-to-developer]

选中所需的开发人员后，请单击“添加到组”下拉列表中所需的组。 使用“从组中删除”下拉列表可以从组中删除开发人员。 

![开发人员][api-management-add-group-to-developer-saved]

一旦开发人员和组之间添加关联，可以在“用户”选项卡中查看它。

## <a name="next-steps"> </a>后续步骤
* 一旦开发人员添加到组，他们可以查看和订阅与该组关联的产品。 有关详细信息，请参阅[如何在 Azure API 管理中创建和发布产品][How create and publish a product in Azure API Management]，
* 除了在发布者门户中创建和管理组，还可使用 API 管理 REST API [组](https://msdn.microsoft.com/library/azure/dn776329.aspx)实体创建和管理组。

[api-management-management-console]: ./media/api-management-howto-create-groups/api-management-management-console.png
[api-management-add-group]: ./media/api-management-howto-create-groups/api-management-add-group.png
[api-management-add-group-window]: ./media/api-management-howto-create-groups/api-management-add-group-window.png
[api-management-new-group]: ./media/api-management-howto-create-groups/api-management-new-group.png
[api-management-add-group-to-product]: ./media/api-management-howto-create-groups/api-management-add-group-to-product.png
[api-management-add-group-to-product-visibility]: ./media/api-management-howto-create-groups/api-management-add-group-to-product-visibility.png
[api-management-add-group-to-developer]: ./media/api-management-howto-create-groups/api-management-add-group-to-developer.png
[api-management-add-group-to-developer-saved]: ./media/api-management-howto-create-groups/api-management-add-group-to-developer-saved.png

[api-management-]: ./media/api-management-howto-create-groups/api-management-.png

[Create a group]: #create-group
[Associate a group with a product]: #associate-group-product
[Associate groups with developers]: #associate-group-developer
[Next steps]: #next-steps

[How create and publish a product in Azure API Management]: api-management-howto-add-products.md

[Get started with Azure API Management]: get-started-create-service-instance.md
[Create an API Management service instance]: get-started-create-service-instance.md
[leverage external groups in associated Azure Active Directory tenants]: api-management-howto-aad.md#how-to-add-an-external-azure-active-directory-group
