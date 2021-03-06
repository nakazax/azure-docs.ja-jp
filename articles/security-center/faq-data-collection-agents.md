---
title: Azure Security Center FAQ - データ収集とエージェント
description: 脅威の防御、検出、対応を可能にする製品である Azure Security Center でのデータ収集、エージェント、およびワークスペースに関してよく寄せられる質問
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/25/2020
ms.author: memildin
ms.openlocfilehash: 8317a13b9ef87679836f55627268deefa4500dce
ms.sourcegitcommit: 99ac4a0150898ce9d3c6905cbd8b3a5537dd097e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/25/2020
ms.locfileid: "77599676"
---
# <a name="faq---questions-about-data-collection-agents-and-workspaces"></a>FAQ - データ収集、エージェント、およびワークスペースに関する質問

Security Center では、セキュリティの脆弱性と脅威を監視するために、Azure 仮想マシン (VM)、仮想マシン スケール セット、IaaS コンテナー、非 Azure コンピューター (オンプレミスのマシンを含む) からデータを収集します。 データは、Microsoft Monitoring Agent を使用して収集されます。Microsoft Monitoring Agent は、セキュリティ関連のさまざまな構成とイベント ログをマシンから読み取り、分析のためにデータをワークスペースにコピーします。


## <a name="am-i-billed-for-azure-monitor-logs-on-the-workspaces-created-by-security-center"></a>Security Center で作成されたワークスペース上の Azure Monitor ログには課金されますか?

いいえ。 ノードごとの Azure Monitor ログの課金が構成されている場合でも、Security Center で作成されるワークスペースに Azure Monitor ログの料金はかかりません。 Security Center の課金は、常に Security Center セキュリティ ポリシーとワークスペースにインストールされているソリューションに基づいています。

- **Free レベル** - Security Center によって既定のワークスペースに 'SecurityCenterFree' ソリューションが有効化されます。 Free レベルの料金はかかりません。

- **Standard レベル** - Security Center によって既定のワークスペースに 'Security' ソリューションが有効化されます。

価格の詳細については、「[Security Center の価格](https://azure.microsoft.com/pricing/details/security-center/)」を参照してください。

> [!NOTE]
> Security Center で作成されたワークスペースのログ分析の価格レベルは、Security Center の課金には影響しません。

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]


## <a name="what-qualifies-a-vm-for-automatic-provisioning-of-the-microsoft-monitoring-agent-installation"></a>Microsoft Monitoring Agent インストールの自動プロビジョニングでは、何をもって VM を適格と見なしますか?

Windows または Linux IaaS VM は、次の条件で適格とします。

- Microsoft Monitoring Agent 拡張機能が現在、VM 上にインストールされていない。
- VM が実行状態である。
- Windows または Linux [Azure 仮想マシン エージェント](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-windows)がインストールされている。
- VM は、Web アプリケーション ファイアウォールや次世代ファイアウォールなどのアプライアンスとしては使用されません。


## <a name="can-i-delete-the-default-workspaces-created-by-security-center"></a>Security Center で作成された既定のワークスペースは削除できますか?

**既定のワークスペースを削除することは推奨されません。** Security Center は、既定のワークスペースを使用して VM のセキュリティ データを格納します。 ワークスペースを削除すると、Security Center はこのデータを収集できず、一部のセキュリティの推奨とアラートと使用できなくなります。

復旧するには、削除したワークスペースに接続されている VM 上の Microsoft Monitoring Agent を削除します。 Security Center によってエージェントが再インストールされ、新しい既定のワークスペースが作成されます。

## <a name="how-can-i-use-my-existing-log-analytics-workspace"></a>既存の Log Analytics ワークスペースを使用するにはどうすればよいですか?

Security Center によって収集されたデータを保存する既存の Log Analytics ワークスペースを選択できます。 既存の Log Analytics ワークスペースを使用する場合、次の要件があります。

- 選択した Azure サブスクリプションにワークスペースを関連付ける必要があります。
- ワークスペースにアクセスするには、少なくとも読み取りアクセス許可が必要です。

既存の Log Analytics ワークスペースを選択するには、次の手順に従います。

1. **[セキュリティ ポリシー - データ収集]** で、 **[Use another workspace]\(別のワークスペースを使用する\)** を選択します。

    ![別のワークスペースを使用する][4]

1. プルダウン メニューから、収集したデータを保存するワークスペースを選択します。

    > [!NOTE]
    > プルダウン メニューには、Azure サブスクリプション内の、アクセスできるワークスペースだけが表示されます。

1. **[保存]** を選択します。 監視対象の VM を再構成するかどうかをたずねられます。

    - 新しいワークスペース設定を**新しい VM にのみ適用**する場合は、 **[いいえ]** を選択します。 この場合、新しいワークスペース設定は、エージェントの新しいインストール (Microsoft Monitoring Agent がインストールされていない、新たに検出された VM) にのみ適用されます。
    - 新しいワークスペース設定を**すべての VM に適用**する場合は、 **[はい]** を選択します。 この場合、Security Center によって作成されたワークスペースに接続されているすべての VM が、新しいターゲット ワークスペースに再接続されます。

    > [!NOTE]
    > **[はい]** を選択した場合、すべての VM が新しいターゲット ワークスペースに再接続されるまで、Security Center によって作成されたワークスペースを削除しないでください。 ワークスペースの削除が早すぎると、この操作は失敗します。

    - 操作を取り消すには、 **[キャンセル]** を選択します。

## Microsoft Monitoring Agent が拡張機能として VM に既にインストールされている場合はどうなりますか?<a name="mmaextensioninstalled"></a>

Monitoring Agent が拡張機能としてインストールされている場合、拡張機能の構成では 1 つのワークスペースにのみレポートできます。 Security Center は、ユーザー ワークスペースへの既存の接続をオーバーライドしません。 Security Center では、"Security" または "SecurityCenterFree" ソリューションがインストールされている場合は、既に接続されているワークスペースに VM からのセキュリティ データが保存されます。 Security Center では、このプロセスで拡張機能のバージョンを最新バージョンにアップグレードする可能性があります。

詳細については、「[既にインストールされているエージェントが存在する場合の自動プロビジョニング](security-center-enable-data-collection.md#preexisting)」をご覧ください。



## マシンに Microsoft Monitoring Agent が拡張機能 (直接エージェント) としてではなく、直接インストールされている場合はどうなりますか?<a name="directagentinstalled"></a>

Microsoft Monitoring Agent が VM に (Azure 拡張機能としてではなく) 直接インストールされている場合、Security Center によって Microsoft Monitoring Agent 拡張機能がインストールされ、Microsoft Monitoring Agent が最新バージョンにアップグレードされる可能性があります。

インストールされているエージェントでは、既に構成されているワークスペースに引き続きレポートし、さらに Security Center で構成されたワークスペースにもレポートします (Windows マシンでは、マルチ ホームがサポートされています)。

構成済みのワークスペースが (Security Center の既定のワークスペースではなく) ユーザー ワークスペースの場合、そのワークスペースにレポートする VM およびコンピューターからのイベントの処理を Security Center で始めるには、"Security/SecurityCenterFree" ソリューションをインストールする必要があります。

Linux マシンでは、エージェントのマルチ ホームはまだサポートされていません。そのため、既存のエージェントのインストールが検出されても、自動プロビジョニングは行われず、マシンの構成は変更されません。

2019 年 3 月 17 日より前に Security Center にオンボードされたサブスクリプションの既存のマシンでは、既存のエージェントが検出されると、Microsoft Monitoring Agent 拡張機能はインストールされず、マシンに影響はありません。 これらのマシンについては、マシンでのエージェントのインストールに関する問題を解決するために、"マシンの監視エージェント正常性の問題を解決する" の推奨事項を参照してください。

詳細については、次のセクションの「[System Center Operations Manager または OMS のダイレクト エージェントが既に VM にインストールされている場合、どうなりますか?](#scomomsinstalled)」をご覧ください

## VM に System Center Operations Manager エージェントが既にインストールされている場合はどうなりますか?<a name="scomomsinstalled"></a>

Security Center によって、Microsoft Monitoring Agent 拡張機能は、既存の System Center Operations Manager エージェントと並行してインストールされます。 既存のエージェントは、引き続き通常どおり System Center Operations Manager サーバーに報告します。 Operations Manager エージェントと Microsoft Monitoring Agent は、このプロセス中に最新バージョンに更新される、共通のランタイム ライブラリを共有することに注意してください。 注 - Operations Manager エージェントのバージョン 2012 がインストールされている場合は、自動プロビジョニングを有効にしないでください (Operations Manager サーバーもバージョン 2012 である場合、管理容易性機能が失われる可能性があり)。


## <a name="what-is-the-impact-of-removing-these-extensions"></a>これらの拡張機能を削除するとどのような影響がありますか?

Microsoft Monitoring Extension を削除すると、Security Center は VM からセキュリティ データを収集できなくなります。また、一部のセキュリティの推奨とアラートを使用できなくなります。 Security Center は、VM に拡張機能が存在せず、拡張機能を再インストールすることが 24 時間以内に判断されます。


## <a name="how-do-i-stop-the-automatic-agent-installation-and-workspace-creation"></a>自動的なエージェントのインストールとワークスペースの作成をオプトアウトするにはどうすればよいですか?

セキュリティ ポリシーでサブスクリプションの自動プロビジョニングを無効にすることはできますが、これは推奨されません。 自動プロビジョニングを無効にすると、Security Center の推奨事項とアラートが制限されます。 自動プロビジョニングを無効にするには、次の手順に従います。

1. Standard レベルのサブスクリプションを構成する場合、そのサブスクリプションのセキュリティ ポリシーを開き、 **[Free]** レベルを選択します。

   ![Pricing tier][1]

1. 次に、 **[セキュリティ ポリシー - データ収集]** ページで **[オフ]** を選択して自動プロビジョニングを無効にします。
   ![データ収集][2]


## <a name="should-i-opt-out-of-the-automatic-agent-installation-and-workspace-creation"></a>自動的なエージェントのインストールとワークスペースの作成をオプトアウトすべきでしょうか?

> [!NOTE]
> 自動プロビジョニングをオプトアウトする場合は、[オプトアウトの影響](#what-are-the-implications-of-opting-out-of-automatic-provisioning)および[オプトアウトする場合に推奨される手順](#what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning)に関するセクションを必ず確認してください。

次の条件に該当する場合、自動プロビジョニングのオプトアウトを検討します。

- Security Center による自動的なエージェントのインストールが、サブスクリプション全体に適用されている。 VM のサブセットには自動インストールを適用できません。 Microsoft Monitoring Agent ではインストールできない重要な VM がある場合は、自動プロビジョニングをオプトアウトする必要があります。
- Microsoft Monitoring Agent (MMA) 拡張機能をインストールすると、エージェントのバージョンが更新される。 これは、ダイレクト エージェントと System Center Operations Manager エージェントに該当します (後者の場合、Operations Manager と MMA で共通のランタイム ライブラリが共有され、それはプロセスで更新されます)。 インストールされている Operations Manager エージェントがバージョン 2012 であり、これがアップグレードされている場合、Operations Manager サーバーもバージョン 2012 になっていると、管理容易性の機能が失われる恐れがあります。 インストールされている Operations Manager エージェントがバージョン 2012 の場合、自動プロビジョニングのオプトアウトを検討してください。
- サブスクリプション (一元化されたワークスペース) の外部にカスタム ワークスペースを保持している場合、自動プロビジョニングをオプトアウトする必要があります。 手動で Microsoft Monitoring Agent 拡張機能をインストールして、Security Center が接続をオーバーライドせずに、お使いのワークスペースに接続できます。
- サブスクリプションごとに複数のワークスペースが作成されることを回避したいと考えていて、サブスクリプション内に独自のカスタム ワークスペースがある場合、次の 2 つの選択肢があります。

   1. 自動プロビジョニングをオプトアウトできます。 移行後に、「[既存の Log Analytics ワークスペースを使用するにはどうすればよいですか?](#how-can-i-use-my-existing-log-analytics-workspace)」の説明に従って、既定のワークスペース設定を設定します。

   1. 移行の完了を許可して、VM 上に Microsoft Monitoring Agent がインストールされ、作成されたワークスペースに VM が接続されるようにします。 その後、既にインストールされているエージェントを再構成し、既定のワークスペース設定を設定して、独自のカスタム ワークスペースを選択します。 詳細については、「[既存の Log Analytics ワークスペースを使用するにはどうすればよいですか?](#how-can-i-use-my-existing-log-analytics-workspace)」をご覧ください。


## <a name="what-are-the-implications-of-opting-out-of-automatic-provisioning"></a>自動プロビジョニングをオプトアウトすると、どのような影響がありますか?

移行が完了すると、Security Center は VM からセキュリティ データを収集できなくなります。また、一部のセキュリティの推奨とアラートを使用できなくなります。 オプトアウトした場合、Microsoft Monitoring Agent を手動でインストールします。 [オプトアウトする場合に推奨される手順](#what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning)に関するセクションをご覧ください。


## <a name="what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning"></a>自動プロビジョニングをオプトアウトする場合、どのような手順が推奨されますか?

Security Center がお使いの VM からセキュリティ データを収集して、推奨や通知を提示できるように、Microsoft Monitoring Agent 拡張機能を手動でインストールします。 インストールのガイダンスとして、[Windows VM 用のエージェントのインストール](../virtual-machines/extensions/oms-windows.md)または [Linux VM 用のエージェントのインストール](../virtual-machines/extensions/oms-linux.md)に関するページを参照してください。

エージェントをいずれかの既存のカスタム ワークスペースまたは Security Center が作成したワークスペースに接続できます。 カスタム ワークスペースの 'Security' または 'SecurityCenterFree' ソリューションが有効になっていない場合は、ソリューションを適用する必要があります。 適用するには、 **[セキュリティ ポリシー – 価格レベル]** ページを利用して、カスタム ワークスペースまたはサブスクリプションを選択します。

   ![Pricing tier][1]

Security Center は、選択された価格レベルに基づいて、ワークスペース上で適切なソリューションを有効にします。


## Security Center にインストールされている OMS 拡張機能を削除するにはどうすればよいですか?<a name="remove-oms"></a>

Microsoft Monitoring Agent は手動で削除することができます。 ただし、Security Center の推奨とアラートが制限されるため、削除は推奨されません。

> [!NOTE]
> データ収集が有効な場合、削除しても、Security Center でエージェントが再インストールされます。  エージェントを手動で削除する前に、データ収集を無効にする必要があります。 データ収集を無効にする手順については、「自動的なエージェントのインストールとワークスペースの作成を停止するにはどうすればよいですか?」を参照してください。

エージェントを手動で削除するには:

1.  ポータルで、 **[Log Analytics]** を開きます。

1.  [Log Analytics] ページで、ワークスペースを選択します。

1.  監視しない VM を選択し、 **[切断]** を選択します。

   ![エージェントを削除する][3]

> [!NOTE]
> Linux VM に拡張機能ではない OMS エージェントが既にある場合、拡張機能を削除するとエージェントも削除されるので、これを再インストールする必要があります。


## <a name="how-do-i-disable-data-collection"></a>データ収集を無効にするにはどうしたらよいですか。

自動プロビジョニングは、既定ではオフです。 リソースの自動プロビジョニングは、セキュリティ ポリシーでこの設定をオフにすることで、いつでも無効にすることができます。 セキュリティ アラートや、システムの更新プログラム、OS の脆弱性、Endpoint Protection に関する推奨事項を取得するために、自動プロビジョニングを使用することを強くお勧めします。

データ収集を無効にするには、[Azure portal にサインイン](https://portal.azure.com)して、 **[参照]** 、 **[セキュリティ センター]** 、 **[ポリシーの選択]** の順に選びます。 自動プロビジョニングを無効にするサブスクリプションを選択します。 サブスクリプションを選ぶと、 **[セキュリティ ポリシー] - [データ収集]** が開きます。 **[自動プロビジョニング]** で **[オフ]** を選びます。


## <a name="how-do-i-enable-data-collection"></a>データ収集を有効にするにはどうしたらよいですか。

Azure サブスクリプションのデータ収集の有効化は、セキュリティ ポリシーで行うことができます。 データ収集を有効にするには、 [Azure portal にサインイン](https://portal.azure.com)して、 **[参照]** 、 **[セキュリティ センター]** 、 **[セキュリティ ポリシー]** の順に選びます。 自動プロビジョニングを有効にするサブスクリプションを選択します。 サブスクリプションを選ぶと、 **[セキュリティ ポリシー] - [データ収集]** が開きます。 **[自動プロビジョニング]** で **[オン]** を選びます。


## <a name="what-happens-when-data-collection-is-enabled"></a>データ収集を有効にするとどうなりますか。

自動プロビジョニングを有効にすると、Security Center は、サポートされているすべての Azure VM と新しく作成される VM に Microsoft Monitoring Agent をプロビジョニングします。 自動プロビジョニングをお勧めしますが、エージェントを手動でインストールすることもできます。 [Microsoft Monitoring Agent の拡張機能をインストールする方法を参照してください](../azure-monitor/learn/quick-collect-azurevm.md#enable-the-log-analytics-vm-extension)。 

エージェントで、プロセス作成イベント 4688 とイベント 4688 内の *CommandLine* フィールドが有効になります。 VM に作成された新しいプロセスは EventLog に記録され、Security Center の検出サービスによって監視されます。 新しいプロセスごとに記録される詳細については、[4688 の説明フィールド](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4688#fields)を参照してください。 また、エージェントは VM に作成された 4688 イベントも収集し、検索に保存します。

さらに、エージェントは、[適応型アプリケーション制御](security-center-adaptive-application.md)のデータ収集を有効にでき、Security Center は、ローカル AppLocker ポリシーを監査モードで構成して、すべてのアプリケーションを許可します。 このポリシーをきっかけとして AppLocker がイベントを生成するようになり、そのイベントが Security Center によって収集されて活用されます。 ただし、既に AppLocker ポリシーが構成されているマシンでは、このポリシーが構成されないので注意してください。 

[セキュリティ連絡先の情報](security-center-provide-security-contact-details.md)が指定されている場合、Security Center が VM で疑わしいアクティビティを検出すると、電子メールでユーザーに通知します。 Security Center のセキュリティ アラート ダッシュボードにもアラートが表示されます。


## <a name="will-security-center-work-using-an-oms-gateway"></a>Security Center は OMS ゲートウェイを使用して動作しますか。

はい。 Azure Security Center は、Microsoft Monitoring Agent を使用して、Azure VM およびサーバーからデータの収集に Azure Monitor を活用します。
データを収集するには、各 VM とサーバーが HTTPS を使用してインターネットに接続する必要があります。 インターネット接続には、直接接続、プロキシを使用した接続、[OMS ゲートウェイ](../azure-monitor/platform/gateway.md)を介した接続を使用できます。


## <a name="does-the-monitoring-agent-impact-the-performance-of-my-servers"></a>Monitoring Agent はサーバーのパフォーマンスに影響しますか。

エージェントは、システム リソースのわずかな量しか消費しないため、パフォーマンスにほとんど影響しません。 パフォーマンスの影響と、エージェントおよび拡張機能の詳細については、[計画および運用ガイド](security-center-planning-and-operations-guide.md#data-collection-and-storage)を参照してください。


## <a name="where-is-my-data-stored"></a>データはどこに格納されますか。

このエージェントから収集されたデータは、サブスクリプションに関連付けられている既存の Log Analytics ワークスペースまたは新規のワークスペースのいずれかに格納されます。 詳細については、[データ セキュリティ](security-center-data-security.md)に関するページを参照してください。


<!--Image references-->
[1]: ./media/security-center-platform-migration-faq/pricing-tier.png
[2]: ./media/security-center-platform-migration-faq/data-collection.png
[3]: ./media/security-center-platform-migration-faq/remove-the-agent.png
[4]: ./media/security-center-platform-migration-faq/use-another-workspace.png