---
title: Azure Cosmos DB の Cassandra API の概要
description: Azure Cosmos DB を使用して既存のアプリケーションを "リフトアンドシフト" し、Cassandra ドライバーと CQL を使用して新しいアプリケーションを構築する方法について説明します。
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: overview
ms.date: 05/21/2019
ms.openlocfilehash: 075b55b5a798924b55ef9d901e4d2e9ecfc9dc1e
ms.sourcegitcommit: 99ac4a0150898ce9d3c6905cbd8b3a5537dd097e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/25/2020
ms.locfileid: "77597577"
---
# <a name="introduction-to-the-azure-cosmos-db-cassandra-api"></a>Azure Cosmos DB の Cassandra API の概要

Azure Cosmos DB の Cassandra API は、[Apache Cassandra](https://cassandra.apache.org) 向けに作成されたアプリのデータ ストアとして使用できます。 つまり、CQLv4 に準拠した既存の [Apache ドライバー](https://cassandra.apache.org/doc/latest/getting_started/drivers.html?highlight=driver)を使用することにより、既存の Cassandra アプリケーションが Azure Cosmos DB の Cassandra API と通信できるようになりました。 多くの場合、接続文字列を変更するだけで、Apache Cassandra の使用から Azure Cosmos DB の Cassandra API の使用に切り替えることができます。 

Cassandra API を使用すると、使い慣れた Cassandra クエリ言語 (CQL)、Cassandra ベースのツール (cqlsh など)、および Cassandra クライアント ドライバーを使用して、Azure Cosmos DB に格納されたデータを操作できます。

## <a name="what-is-the-benefit-of-using-apache-cassandra-api-for-azure-cosmos-db"></a>Azure Cosmos DB 用の Apache Cassandra API を使用するメリット

**操作の管理はなし**:Azure Cosmos DB の Cassandra API は、フル マネージドのクラウド サービスとして、OS、JVM、および yaml ファイルやそれらの相互通信全体の無数の設定を管理および監視するためのオーバーヘッドを解消します。 Azure Cosmos DB には、スループット、待機時間、ストレージ、可用性、および構成可能なアラートの監視機能が用意されています。

**パフォーマンス管理**:Azure Cosmos DB では、SLA によってバックアップされた、99 パーセンタイルでの保証された低待ち時間の読み取りと書き込みが提供されます。 ユーザーは、高パフォーマンスおよび低待機時間の読み取りと書き込みを確保するために操作のオーバーヘッドについて心配する必要はありません。 つまり、ユーザーは圧縮のスケジュール設定、廃棄標識の管理、ブルーム フィルターやレプリカの設定などに手動で対処する必要はありません。 Azure Cosmos DB は、これらの問題を管理するためのオーバーヘッドを解消し、ユーザーがアプリケーションのロジックに焦点を絞ることができるようにします。

**既存のコードとツールを使用可能**:Azure Cosmos DB では、既存の Cassandra SDK およびツールとのワイヤ プロトコル レベルの互換性が提供されます。 この互換性により、Azure Cosmos DB の Cassandra API を少し変更するだけで、既存のコードベースを使用できることが保証されます。

**スループットとストレージの柔軟性**:Azure Cosmos DB は、すべてのリージョンにまたがる保証されたスループットを提供し、プロビジョニングされたスループットを Azure portal、PowerShell、または CLI 操作でスケーリングできます。 予測可能なパフォーマンスの必要に応じて、テーブルのストレージやスループットを柔軟にスケーリングできます。

**グローバル配布および可用性**:Azure Cosmos DB では、低待ち時間のデータ アクセスと高可用性を確保しながら、すべての Azure リージョンにまたがってデータをグローバルに配布し、それらのデータをローカルで処理する機能が提供されます。 Azure Cosmos DB では、リージョン内では 99.99% の高可用性を、複数のリージョン間では 99.999% の読み取りと書き込みの可用性を提供し、操作のオーバーヘッドはありません。 詳細については、「[データのグローバル配布](distribute-data-globally.md)」の記事を参照してください。 

**整合性の選択**:Azure Cosmos DB では、整合性とパフォーマンスの間の最適なトレードオフを実現するために、適切に定義された 5 つの整合性レベルの選択が提供されます。 これらの整合性レベルは、強固、有界整合性制約、セッション、一貫性のあるプレフィックス、および最終的です。 これらの適切に定義された、実際的で、直感的な整合性レベルにより、開発者は整合性、可用性、および待機時間の間の正確なトレードオフを検討できます。 詳細については、[整合性レベル](consistency-levels.md)に関する記事を参照してください。 

**エンタープライズ グレード**:Azure Cosmos DB は、[コンプライアンス認定](https://www.microsoft.com/trustcenter)を提供して、ユーザーが安全にプラットフォームを使用できることを保証します。 また、Azure Cosmos DB には、保存時および移動時の暗号化、IP ファイアウォール、およびコントロール プレーン アクティビティの監査ログも用意されています。

## <a name="next-steps"></a>次のステップ

* Cassandra API データを作成および管理するために、次の言語固有のアプリの構築をすばやく開始できます。
  - [Node.js アプリ](create-cassandra-nodejs.md)
  - [.NET アプリ](create-cassandra-dotnet.md)
  - [Python アプリ](create-cassandra-python.md)

* Java アプリケーションを使用した[ Cassandra API アカウント、データベースおよびテーブルの作成](create-cassandra-api-account-java.md)の開始

* Java アプリケーションを使用して、[Cassandra API テーブルにサンプル データを読み込みます](cassandra-api-load-data.md)。

* Java アプリケーションを使用して、[Cassandra API アカウントからデータのクエリを実行します](cassandra-api-query-data.md)。

* Azure Cosmos DB の Cassandra API によってサポートされる Apache Cassandra の機能の詳細については、「[Cassandra のサポート](cassandra-support.md)」の記事を参照してください。

* [よく寄せられる質問](faq.md#cassandra)を参照してください。
