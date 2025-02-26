---
title: 查看租户应用程序 - Azure Active Directory | Microsoft Docs
description: 使用 Azure 门户查看 Azure Active Directory (Azure AD) 租户中的应用程序。
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
origin.date: 04/09/2019
ms.date: 07/04/2019
ms.author: v-junlch
ms.reviewer: arvinh
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 398785194516882aea59a4421f334dfbebf01a07
ms.sourcegitcommit: 5f85d6fe825db38579684ee1b621d19b22eeff57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67568587"
---
# <a name="view-your-azure-active-directory-tenant-applications"></a>查看 Azure Active Directory 租户应用程序

本快速入门使用 Azure 门户查看 Azure Active Directory (Azure AD) 租户中的应用程序。

## <a name="before-you-begin"></a>准备阶段

若要查看结果，Azure AD 租户中需至少有一个应用程序。

以 Azure AD 租户全局管理员、云应用程序管理员或应用程序管理员的身份登录到 [Azure 门户](https://portal.azure.cn)。

## <a name="find-the-list-of-tenant-applications"></a>查找租户应用程序的列表

Azure AD 租户应用程序可以在 Azure 门户的“企业应用”  部分查看。

若要查找租户应用程序，请执行以下操作：

1. 在 **[Azure 门户](https://portal.azure.cn)** 的左侧导航面板中，选择“Azure Active Directory”。  

1. 在“Azure Active Directory”窗格中，选择“企业应用程序”。   

1. 在“应用程序类型”下拉菜单中选择“所有应用程序”，然后选择“应用”    。 此时会显示租户应用程序的随机示例。
   
1. 若要查看更多应用程序，请选择列表底部的“加载更多”  。 根据租户中应用程序的数目，也许[搜索特定应用程序](#search-for-a-tenant-application)比滚动浏览整个列表要容易些。

## <a name="select-viewing-options"></a>选择查看选项

选择查找的内容选择所需的选项。

1. 可以根据“应用程序类型”、“应用程序状态”和“应用程序可见性”查看应用程序。    

1. 在“应用程序类型”下选择下述选项之一： 

    - “企业应用程序”显示非 Microsoft 应用程序  。
    - “Microsoft 应用程序”显示  Microsoft 应用程序。
    - “所有应用程序”显示  非 Microsoft 应用程序和 Microsoft 应用程序。

1. 在“应用程序状态”下选择“任何”、“已禁用”或“已启用”。     “任何”  选项包括已禁用和已启用的应用程序。

1. 在“应用程序可见性”下选择“任何”或“已隐藏”    。 “已隐藏”选项显示租户中存在的、但对用户不可见的应用程序。 

1. 选择所需的选项后，选择“应用”  。
 

## <a name="search-for-a-tenant-application"></a>搜索租户应用程序

搜索特定的应用程序：

1. 在“应用程序类型”菜单中，选择“所有应用程序”，然后选择“应用”    。

1. 输入要查找的应用程序的名称。 如果该应用程序已添加到 Azure AD 租户中，则会显示在搜索结果中。 此示例显示 GitHub 尚未添加到租户应用程序中。

    ![搜索应用程序](./media/view-applications-portal/search-for-tenant-application.png)

1. 尝试输入应用程序名称的头几个字母。 此示例显示以 **Sales** 开头的所有应用程序。

    ![使用前缀搜索](./media/view-applications-portal/search-by-prefix.png)

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已了解如何查看 Azure AD 租户中的应用程序， 以及如何按应用程序类型、状态和可见性筛选应用程序列表。 另外还介绍了如何搜索特定应用程序。

现在，你已找到了要查找的应用程序，可以继续选择该应用程序来查看或编辑属性和配置选项。



<!-- Update_Description: update metedata properties -->
