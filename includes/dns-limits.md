---
author: rothja
ms.service: cost-management-billing
ms.topic: include
ms.date: 2/14/2020
ms.author: rohink
ms.openlocfilehash: b674f8e31eb61328f60bb24866f73d02653b655f
ms.sourcegitcommit: 98a5a6765da081e7f294d3cb19c1357d10ca333f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/20/2020
ms.locfileid: "77495150"
---
**パブリック DNS ゾーン**

| リソース | 既定の制限 |
| --- | --- |
| サブスクリプションあたりのパブリック DNS ゾーン数 |250 <sup>1</sup> |
| パブリック DNS ゾーンあたりのレコード セット数 |10,000 <sup>1</sup> |
| パブリック DNS ゾーン内のレコード セットあたりのレコード数 |20 |
| 1 つの Azure リソースのエイリアス レコードの数 |20|
| サブスクリプションあたりのプライベート DNS ゾーン数 |1000|
| プライベート DNS ゾーンあたりのレコード セット数 |25000|
| プライベート DNS ゾーン用のレコード セットあたりのレコード数 |20|
| プライベート DNS ゾーンあたりの仮想ネットワーク リンク数 |1000|
| 自動登録が有効なプライベート DNS ゾーンあたりの仮想ネットワーク リンク数 |100|
| 自動登録が有効な状態で仮想ネットワークがリンクできるプライベート DNS ゾーンの数 |1|
| 仮想ネットワークがリンクできるプライベート DNS ゾーンの数 |1000|
| 1 秒あたりに仮想マシンから Azure DNS リゾルバーに送信される DNS クエリの数 |500 <sup>2</sup> |
| 仮想マシンごとのキューに登録された (保留中の応答) DNS クエリの最大数 |200 <sup>2</sup> |

<sup>1</sup>これらの制限値を引き上げる必要がある場合は、Azure サポートにお問い合せください。

<sup>2</sup>これらの制限は、仮想ネットワーク レベルではなく、個々の仮想マシンごとに適用されます。 これらの制限を超える DNS クエリは削除されます。