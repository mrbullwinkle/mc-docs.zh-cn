---
title: include 文件
description: include 文件
services: storage
author: wmgries
ms.service: storage
ms.topic: include
ms.date: 05/05/2019
ms.author: wgries
ms.custom: include file
ms.openlocfilehash: a278a414ed59c4a4a1d5924c2fece0423777d1de
ms.sourcegitcommit: 1414c787aa13b802e43fc7317af96a9e14889e20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68332767"
---
| Resource | 目标 | 硬限制 |
|----------|--------------|------------|
| 每个区域的存储同步服务数 | 20 个存储同步服务 | 是 |
| 每个存储同步服务的同步组数 | 100 个同步组 | 是 |
| 每个存储同步服务的已注册服务器 | 99 台服务器 | 是 |
| 每个同步组的云终结点数 | 1 个云终结点 | 是 |
| 每个同步组的服务器终结点数 | 50 个服务器终结点 | 否 |
| 每个服务器的服务器终结点数 | 30 个服务器终结点 | 是 |
| 每个同步组的文件系统对象数（目录和文件） | 2500 万个对象 | 否 |
| 目录中的最大文件系统对象（目录和文件）数 | 100 万个对象 | 是 |
| 最大对象（目录和文件）安全描述符大小 | 64 KiB | 是 |
| 文件大小 | 100 GiB | 否 |
| 要进行分层的文件的最小文件大小 | 64 KiB | 是 |
| 并发同步会话 | V4 代理及更高版本：限制因可用系统资源而异。 <BR> V3 代理：每个处理器两个活动的同步会话，或每个服务器最多八个活动的同步会话。 | 是

> [!Note]  
> Azure 文件同步终结点可以纵向扩展到 Azure 文件共享的大小。 如果达到 Azure 文件共享大小限制，同步将无法运行。