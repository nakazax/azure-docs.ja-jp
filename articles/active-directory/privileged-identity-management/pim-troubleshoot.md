---
title: Privileged Identity Management に関する問題のトラブルシューティング - Azure Active Directory | Microsoft Docs
description: Azure AD Privileged Identity Management (PIM) でロールに関するシステム エラーのトラブルシューティング方法を説明します。
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: pim
ms.date: 10/18/2019
ms.author: curtand
ms.collection: M365-identity-device-management
ms.openlocfilehash: 474f2634e6f7ddc1840548c39ae86cb54c3bf08e
ms.sourcegitcommit: f915d8b43a3cefe532062ca7d7dbbf569d2583d8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2020
ms.locfileid: "78299671"
---
# <a name="troubleshoot-a-problem-with-privileged-identity-management"></a>Privileged Identity Management に関する問題のトラブルシューティング

Azure Active Directory (Azure AD) の Privileged Identity Management (PIM) に関して問題がありますか。 サービスをもう一度正常に動作させるために、以下の情報が役立ちます。

## <a name="access-to-azure-resources-denied"></a>Azure リソースへのアクセスが拒否されました

### <a name="problem"></a>問題

Azure リソースのアクティブな所有者またはユーザー アクセス管理者は、Privileged Identity Management 内部でリソースを表示することはできますが、資格のある割り当てを作成したり、リソース概要ページからロールの割り当ての一覧を表示したりするなどのアクションを実行することはできません。 これらのアクションを実行すると、承認エラーが発生します。

### <a name="cause"></a>原因

この問題は、PIM サービス プリンシパルのユーザー アクセス管理者ロールが、誤ってサブスクリプションから削除された場合に発生する可能性があります。 Privileged Identity Management サービスから Azure リソースへのアクセスができるようにするには、Azure サブスクリプションで MS-PIM サービス プリンシパルに[ユーザー アクセス管理者ロール](../../role-based-access-control/built-in-roles.md#user-access-administrator)が常に割り当てられている必要があります。

### <a name="resolution"></a>解像度

Privileged identity Management サービス プリンシパル名 (MS-PIM) にサブスクリプション レベルでユーザー アクセス管理者ロールを割り当てます。 この割り当てにより、Privileged identity Management サービスから Azure リソースにアクセスできるようになります。 そのロールは、実際の要件に応じて、管理グループ レベルまたはサブスクリプション レベルで割り当てることができます。 サービス プリンシパルの詳細については、「[アプリケーションをロールに割り当てる](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#assign-a-role-to-the-application)」を参照してください。

## <a name="next-steps"></a>次のステップ

- [Privileged Identity Management を使用するためのライセンスの要件](subscription-requirements.md)
- [Azure AD でのハイブリッドおよびクラウド デプロイ用の特権アクセスをセキュリティで保護する](../users-groups-roles/directory-admin-roles-secure.md?toc=%2fazure%2factive-directory%2fprivileged-identity-management%2ftoc.json)
- [Privileged Identity Management をデプロイする](pim-deployment-plan.md)
