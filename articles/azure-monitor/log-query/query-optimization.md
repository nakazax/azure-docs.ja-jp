---
title: Azure Monitor でログ クエリを最適化する
description: Azure Monitor でログ クエリを最適化するためのベスト プラクティス。
ms.service: azure-monitor
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/25/2019
ms.openlocfilehash: 521fd84e79196439ea220bd7ffa7cc6d0750f045
ms.sourcegitcommit: 96dc60c7eb4f210cacc78de88c9527f302f141a9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/27/2020
ms.locfileid: "77648837"
---
# <a name="optimize-log-queries-in-azure-monitor"></a>Azure Monitor でログ クエリを最適化する
Azure Monitor ログでは、[Azure Data Explorer (ADX)](/azure/data-explorer/) を使用して、ログ データを格納し、そのデータを分析するためのクエリを実行します。 これにより、ADX クラスターが作成、管理、保持され、ログ分析ワークロードに合わせて最適化されます。 クエリを実行すると、クエリが最適化され、ワークスペース データを格納する適切な ADX クラスターにルーティングされます。 Azure Monitor ログと Azure Data Explorer のどちらにも、クエリの自動最適化メカニズムが多数使用されています。 自動最適化によって大幅に処理が促進される一方で、クエリのパフォーマンスを飛躍的に向上させることができるケースもいくつかあります。 この記事では、パフォーマンスに関する考慮事項とそれを調整するいくつかの手法について説明します。

その手法の大半は、Azure Data Explorer と Azure Monitor ログに対して直接実行されるクエリに共通しますが、ここでは Azure Monitor ログに固有の考慮事項をいくつか取り上げます。 Azure Data Explorer の最適化に関するヒントについては、「[クエリのベスト プラクティス](/azure/kusto/query/best-practices)」を参照してください。

最適化されたクエリは次のようになります。

- 実行速度が上がり、クエリ実行全体の期間が短縮されます。
- スロットルまたは拒否される可能性が低くなります。

ダッシュボード、アラート、Logic Apps、Power BI など、繰り返し集中的に使用されるクエリには、特に注意を払う必要があります。 このような場合に効果のないクエリが及ぼす影響はかなり大きくなります。

## <a name="query-performance-pane"></a>クエリ パフォーマンス ペイン
Log Analytics でクエリを実行した後、クエリ結果の上にある下矢印をクリックすると、クエリ パフォーマンス ペインが表示されます。ここには、クエリのいくつかのパフォーマンス指標の結果が示されます。 これらの各パフォーマンス指標については、次のセクションで説明します。

![クエリ パフォーマンス ペイン](media/query-optimization/query-performance-pane.png)


## <a name="query-performance-indicators"></a>クエリ パフォーマンス指標

実行されるクエリごとに、次のクエリ パフォーマンス指標を使用できます。

- [[合計 CPU]](#total-cpu): すべての計算ノードでクエリを処理するために使用された全体的な計算。 これは、計算、解析、データの取得に使用された時間を表します。 

- [[処理されたクエリに使用するデータ]](#data-used-for-processed-query): クエリを処理するためにアクセスされたデータ全体。 対象のテーブルのサイズ、使用された期間、適用されたフィルター、参照されている列の数の影響を受けます。

- [[処理されたクエリの期間]](#time-span-of-the-processed-query): クエリを処理するためにアクセスされた最新データと最も古いデータの差分。 クエリに指定された明示的な時間の範囲の影響を受けます。

- [[処理されたデータの期間]](#age-of-processed-data): 現在と、クエリを処理するためにアクセスされた最も古いデータの差分。 これは、データの取得の効率に大きく影響します。

- [[ワークスペースの数]](#number-of-workspaces): 暗黙的または明示的な選択によってクエリ処理中にアクセスされたワークスペースの数。

- [[リージョンの数]](#number-of-regions): 暗黙的または明示的なワークスペースの選択によってクエリ処理中にアクセスされたリージョンの数。 複数リージョンのクエリでは効率がかなり低下し、パフォーマンス指標は部分的にしかカバーしなくなります。

- [[並行処理]](#parallelism):システムにより、このクエリをどの程度、複数ノードで実行できたかを示します。 CPU 使用率が高いクエリにのみ関連します。 特定の関数と演算子の使用による影響を受けます。


## <a name="total-cpu"></a>合計 CPU
すべてのクエリ処理ノードでこのクエリを処理するために実際に投入されたコンピューティング CPU です。 ほとんどのクエリは多数のノード上で実行されるため、通常、これはクエリの実行に実際にかかった期間よりもはるかに大きくなります。 

クエリの処理時間は以下に費やされます。
- データ取得 - 古いデータの取得には、最近のデータの取得よりも時間がかかります。
- データ処理 - データのロジックと評価。 

クエリ処理ノードでかかった時間以外にも、ユーザーの認証とそのユーザーがこのデータにアクセスできることの確認、データ ストアの特定、クエリの解析、クエリ処理ノードの割り当てのために Azure Monitor ログではさらに時間が費やされます。 この時間は、クエリの合計 CPU 時間には含まれません。

クエリのコマンドと関数の中には、CPU 使用率の高いものがあります。 これは特に、JSON と XML の解析や複雑な正規表現の抽出を実行するコマンドに当てはまります。 このような解析は、[parse_json ()](/azure/kusto/query/parsejsonfunction) または [parse_xml ()](/azure/kusto/query/parse-xmlfunction) 関数で明示的に行われることもあれば、動的な列の参照時に暗黙的に行われることもあります。

これらの関数では、処理対象の行数に比例して CPU が消費されます。 最も効率的な最適化では、CPU を集中的に使用する関数が実行される前にできるだけ多くのレコードを除外できる where 条件を、クエリ内の初期段階で追加します。

たとえば、次のクエリではまったく同じ結果が得られますが、2 番目のクエリは、解析前の [where]() 条件によって多くのレコードが除外されるためはるかに効率的になります。

```Kusto
//less efficient
SecurityEvent
| extend Details = parse_xml(EventData)
| extend FilePath = tostring(Details.UserData.RuleAndFileData.FilePath)
| extend FileHash = tostring(Details.UserData.RuleAndFileData.FileHash)
| summarize count() by FileHash, FilePath
| where FileHash != "" and FilePath !startswith "%SYSTEM32"  // Problem: irrelevant results are filtered after all processing and parsing is done
```
```Kusto
//more efficient
SecurityEvent
| where EventID == 8002 //Only this event have FileHash
| where EventData !has "%SYSTEM32" //Early removal of unwanted records
| extend Details = parse_xml(EventData)
| extend FilePath = tostring(Details.UserData.RuleAndFileData.FilePath)
| extend FileHash = tostring(Details.UserData.RuleAndFileData.FileHash)
| summarize count() by FileHash, FilePath
```

データセットに物理的に存在する列ではなく、評価済みの列に [where](/azure/kusto/query/whereoperator) 句が配置されているクエリでは、効率が低下します。 大規模なデータ セットを処理する場合、評価済みの列に対してフィルター処理を行ってもシステムは最適化されない場合があります。
たとえば、次のクエリではまったく同じ結果が得られますが、[where](/azure/kusto/query/whereoperator) 条件が組み込み列を参照するため、2 番目のクエリの方が効率的です。

```Kusto
//less efficient
Heartbeat 
| extend IPRegion = iif(RemoteIPLongitude  < -94,"WestCoast","EastCoast")
| where IPRegion == "WestCoast"
| summarize count() by Computer
```
```Kusto
//more efficient
Heartbeat 
| where RemoteIPLongitude  < -94
| extend IPRegion = iif(RemoteIPLongitude  < -94,"WestCoast","EastCoast")
| summarize count() by Computer
```

[max()](/azure/kusto/query/max-aggfunction)、[sum()](/azure/kusto/query/sum-aggfunction)、[count()](/azure/kusto/query/count-aggfunction)、[avg()](/azure/kusto/query/avg-aggfunction) などの集計コマンドには、そのロジックにより CPU への影響は小さくなるものもありますが、より複雑で、効率的な実行を可能にするヒューリスティックや推定が含まれるものもあります。 たとえば、[dcount ()](/azure/kusto/query/dcount-aggfunction) では、HyperLogLog アルゴリズムを使用することで、各値を実際にカウントすることなく、大規模なデータ セットの個別カウントに近い推定値が提供されます。また、パーセンタイル関数は、最も近いランク パーセンタイル アルゴリズムを使用して同様の概算を行います。 コマンドのいくつかには、影響を軽減するための省略可能なパラメーターが含まれています。 たとえば、[makeset ()](/azure/kusto/query/makeset-aggfunction) 関数には、CPU とメモリに大きく影響する、最大セット サイズを定義するための省略可能なパラメーターがあります。

[join](/azure/kusto/query/joinoperator?pivots=azuremonitor) および [summarize](/azure/kusto/query/summarizeoperator) コマンドを使用すると、大規模なデータ セットを処理する際に CPU 使用率が高くなる可能性があります。 これらの複雑さは、summarize 内で `by` として使用されている列または join 属性として使用されている列に指定可能な値の数 ("*カーディナリティ*" と呼ばれます) に直接関連しています。 join および summarize の説明と最適化については、それぞれのドキュメント記事と最適化のヒントを参照してください。

たとえば、次のクエリでは、**CounterPath** が常に 1 対 1 で **CounterName** および **ObjectName** にマップされるため、まったく同じ結果が得られます。 2 番目のクエリは、集計ディメンションが小さくなるにつれて、効率が高まります。

```Kusto
//less efficient
Perf
| summarize avg(CounterValue) 
by CounterName, CounterPath, ObjectName
```
```Kusto
//make the group expression more compact improve the performance
Perf
| summarize avg(CounterValue), any(CounterName), any(ObjectName) 
by CounterPath
```

CPU 使用率は、集中的なコンピューティングを必要とする where 条件や拡張列による影響を受ける可能性もあります。 [equal ==](/azure/kusto/query/datatypes-string-operators) や [startswith](/azure/kusto/query/datatypes-string-operators) などの単純な文字列比較では CPU への影響がほぼ変わらないのに対して、高度なテキスト一致では影響が大きくなります。 具体的には、[has](/azure/kusto/query/datatypes-string-operators) 演算子は [contains](/azure/kusto/query/datatypes-string-operators) 演算子よりも効率的です。 文字列処理の手法が原因で、短い文字列よりも 4 文字を超える文字列を検索する方が効率的となります。

たとえば、次のクエリでは、コンピューターの名前付けポリシーに応じて、同様の結果が得られますが、2 番目のクエリの方が効率的です。

```Kusto
//less efficient – due to filter based on contains
Heartbeat
| where Computer contains "Production" 
| summarize count() by ComputerIP 
```
```Kusto
//less efficient – due to filter based on extend
Heartbeat
| extend MyComputer = Computer
| where MyComputer startswith "Production" 
| summarize count() by ComputerIP 
```
```Kusto
//more efficient
Heartbeat
| where Computer startswith "Production" 
| summarize count() by ComputerIP 
```

> [!NOTE]
> この指標は、直近のクラスターの CPU のみを示します。 複数リージョンのクエリでは、リージョンのうち 1 つだけが表示されます。 マルチワークスペース クエリでは、一部のワークスペースが含まれない場合があります。


## <a name="data-used-for-processed-query"></a>処理されたクエリに使用するデータ

クエリの処理における重要な要因は、クエリ処理のためにスキャン、使用されるデータの量です。 Azure Data Explorer では、他のデータ プラットフォームと比べ、データ量を大幅に削減する積極的な最適化が使用されています。 それでも、クエリには、使用されるデータ量に影響を与える可能性のある重要な要因があります。
Azure Monitor ログでは、データにインデックスを付ける手段として、**TimeGenerated** 列が使用されます。 **TimeGenerated** の値をできるだけ狭い範囲に絞り込むと、処理すべきデータの量がかなり制限されることで、クエリのパフォーマンスが大幅に向上します。

処理するデータが増加するもう 1 つの要因は、多数のテーブルの使用です。 これは通常、`search *` および `union *` コマンドの使用時に発生します。 これらのコマンドを実行すると、システムでは、強制的にワークスペース内のすべてのテーブルのデータが評価され、スキャンされます。 場合によっては、ワークスペース内には数百ものテーブルが存在することがあります。 "search *" の使用や、範囲に特定のテーブルを指定しない検索は、できる限り避けるようにしてください。

たとえば、次のクエリではまったく同じ結果が得られますが、最後が最も効率的です。

```Kusto
// This version scans all tables though only Perf has this kind of data
search "Processor Time" 
| summarize count(), avg(CounterValue)  by Computer
```
```Kusto
// This version scans all strings in Perf tables – much more efficient
Perf
| search "Processor Time" 
| summarize count(), avg(CounterValue)  by Computer
```
```Kusto
// This is the most efficient version 
Perf 
| where CounterName == "% Processor Time"  
| summarize count(), avg(CounterValue)  by Computer
```

データ量を削減するもう 1 つの方法として、クエリの初期段階で [where](/azure/kusto/query/whereoperator) 条件を使用します。 Azure Data Explorer プラットフォームには、特定の where 条件に関連するデータを含むパーティションを認識できるようにするキャッシュが含まれています。 たとえば、クエリに `where EventID == 4624` が含まれている場合、一致するイベントが含まれるパーティションを処理するノードにのみクエリが分散されます。

次のクエリ例ではまったく同じ結果が得られますが、2 番目の方が効率的です。

```Kusto
//less efficient
SecurityEvent
| summarize LoginSessions = dcount(LogonGuid) by Account
```
```Kusto
//more efficient
SecurityEvent
| where EventID == 4624 //Logon GUID is relevant only for logon event
| summarize LoginSessions = dcount(LogonGuid) by Account
```

Azure Data Explorer は列形式のデータ ストアであるため、各列の取得は他の列とは関係がありません。 取得される列の数は、全体のデータ量に直接影響します。 結果を[集計](/azure/kusto/query/summarizeoperator)したり、特定の列を[射影](/azure/kusto/query/projectoperator)したりすることで、必要な列のみを出力に含める必要があります。 Azure Data Explorer には、取得される列の数を減らすために複数の最適化を行っています。 ある列が不要であると判断された場合 (たとえば、[summarize](/azure/kusto/query/summarizeoperator) コマンドで参照されていない場合)、その列は取得されません。

たとえば、2 番目のクエリでは、1 つの列ではなく 3 つの列を取得する必要があるため、処理するデータが 3 倍になる可能性があります。

```Kusto
//Less columns --> Less data
SecurityEvent
| summarize count() by Computer  
```
```Kusto
//More columns --> More data
SecurityEvent
| summarize count(), dcount(EventID), avg(Level) by Computer  
```

## <a name="time-span-of-the-processed-query"></a>処理されたクエリの期間

Azure Monitor ログ内のすべてのログは、**TimeGenerated** 列に従ってパーティション分割されます。 アクセスされるパーティションの数は、期間に直接関係します。 時間範囲を短縮することは、迅速なクエリ実行を確実にするための最も効率的な方法となります。

時間範囲は、「[Azure Monitor Log Analytics のログ クエリのスコープと時間範囲](scope.md#time-range)」で説明されているように、Log Analytics 画面で時間範囲セレクターを使用して設定できます。 選択した時間範囲がクエリ メタデータを使用してバックエンドに渡されるため、この方法をお勧めします。 

別の方法として、クエリの **TimeGenerated** に [where](/azure/kusto/query/whereoperator) 条件を明示的に含めることもできます。 この方法を使用すると、クエリが別のインターフェイスから使用される場合でも、期間が確実に固定されます。
クエリのすべての要素には必ず **TimeGenerated** フィルターを指定する必要があります。 複数のテーブルまたは同じテーブルからデータを取得するサブクエリがクエリに含まれている場合、それぞれに独自の [where](/azure/kusto/query/whereoperator) 条件を含める必要があります。

たとえば、次のクエリでは、**Perf** テーブルで最終日のみがスキャンされますが、**Heartbeat** テーブルではその履歴すべて (最大 2 年間の場合があります) がスキャンされます。

```Kusto
Perf
| where TimeGenerated > ago(1d)
| summarize avg(CounterValue) by Computer, CounterName
| join kind=leftouter (
    Heartbeat
    //No time span filter in this part of the query
    | summarize IPs = makeset(ComputerIP, 10) by  Computer
) on Computer
```

このような誤りが発生する一般的なケースとして、直近の出現箇所の検索に [arg_max ()](/azure/kusto/query/arg-max-aggfunction) が使用されている場合が挙げられます。 次に例を示します。

```Kusto
Perf
| where TimeGenerated > ago(1d)
| summarize avg(CounterValue) by Computer, CounterName
| join kind=leftouter (
    Heartbeat
    //No time span filter in this part of the query
    | summarize arg_max(TimeGenerated, *), min(TimeGenerated)   
by Computer
) on Computer
```

これは、次のように、内部クエリに時間フィルターを追加することで簡単に修正できます。

```Kusto
Perf
| where TimeGenerated > ago(1d)
| summarize avg(CounterValue) by Computer, CounterName
| join kind=leftouter (
    Heartbeat
    | where TimeGenerated > ago(1d) //filter for this part
    | summarize arg_max(TimeGenerated, *), min(TimeGenerated)   
by Computer
) on Computer
```

> [!IMPORTANT]
> この指標は、リージョンをまたぐクエリでは使用できません。

## <a name="age-of-processed-data"></a>処理されたデータの期間
Azure Data Explorer では、インメモリ、ローカルの SSD ディスク、さらに処理速度の遅い Azure BLOB という複数のストレージ層が使用されています。 データが新しいほど、待ち時間が短い、より高性能な層に格納される確率が高くなり、クエリ期間と CPU が低減されます。 システムには、データ自体に加え、メタデータ用のキャッシュもあります。 データが古くなるほど、メタデータがキャッシュ内に存在する確率が低くなります。

クエリによっては古いデータの使用が必要な場合がありますが、古いデータが誤って使用されるケースもあります。 これは、メタデータに時間範囲を指定せずにクエリを実行した際に、一部のテーブル参照に **TimeGenerated** 列に対するフィルターが含まれていない場合に発生します。 このような場合、システムによって、そのテーブルに格納されているすべてのデータがスキャンされます。 データ保有期間が長い場合は、データ保有期間と同じ長い時間範囲 (したがって古いデータ) が対象となることがあります。

たとえば、次のようなケースがあります。

- Log Analytics で、制限されないサブクエリを使用して時間範囲が設定していない (上記の例を参照)。
- 時間範囲の省略可能なパラメーターを指定せずに API を使用している。
- Power BI コネクタなど、時間範囲を強制しないクライアントを使用している。

前のセクションの例と注意事項は、このケースにも関連するため、確認してください。

## <a name="number-of-regions"></a>リージョンの数
1 つのクエリがさまざまなリージョンにわたって実行される可能性がある状況がいくつか考えられます。

- 明示的に一覧表示されている複数のワークスペースが異なるリージョンに存在する場合。
- リソースでスコープが指定されたクエリでデータを取得し、異なるリージョンに存在する複数のワークスペースにそのデータが格納される場合。

リージョンをまたがるクエリを実行するには、通常はクエリの最終結果よりはるかに大きい、バックエンドの中間データ チャンクをシステムでシリアル化して転送する必要があります。 また、最適化、ヒューリスティック、キャッシュの利用を行うためのシステムの機能も制限されます。
これらのリージョンすべてをスキャンする理由がない場合は、対象のリージョンが絞り込まれるようにスコープを調整する必要があります。 リソースのスコープが最小化されても、多くのリージョンが使用されている場合は、構成の誤りが原因で生じる可能性があります。 たとえば、監査ログと診断設定が異なるリージョンにある別々のワークスペースに送信されたり、診断設定の構成が複数存在したりします。 

> [!IMPORTANT]
> この指標は、リージョンをまたぐクエリでは使用できません。

## <a name="number-of-workspaces"></a>ワークスペースの数
ワークスペースは、ログ データを分離して管理するために使用される論理コンテナーです。 バックエンドで、選択したリージョン内の物理クラスターにおけるワークスペースの配置が最適化されます。

複数のワークスペースが使用されるのは、次の場合が考えられます。 

- 複数のワークスペースが明示的に一覧表示されている場合。
- リソースでスコープが指定されたクエリでデータを取得し、複数のワークスペースにそのデータが格納される場合。
 
リージョンおよびクラスターにまたがるクエリを実行するには、通常はクエリの最終結果よりはるかに大きい、バックエンドの中間データ チャンクをシステムでシリアル化して転送する必要があります。 また、最適化、ヒューリスティック、キャッシュの利用を行うためのシステムの機能も制限されます。

> [!IMPORTANT]
> マルチワークスペースのシナリオでは、CPU とデータの測定値が正確ではなくなり、ごく一部のワークスペースに測定値のみが表示されます。

## <a name="parallelism"></a>Parallelism
Azure Monitor ログでは、Azure Data Explorer の大規模なクラスターを使用してクエリを実行しますが、これらのクラスターの規模は変動します。 このシステムでは、ワークスペースの配置ロジックと容量に応じて、クラスターが自動的にスケーリングされます。

クエリを効率的に実行するために、クエリは、その処理に必要なデータに基づいてパーティション分割され、計算ノードに分散されます。 システムでこれを効率的に実行できない状況もあります。 この結果、クエリの期間が長くなる可能性があります。 

並列処理を低減できるクエリ動作は次のとおりです。

- シリアル化関数やウィンドウ関数 ([serialize 演算子](/azure/kusto/query/serializeoperator)、[next ()](/azure/kusto/query/nextfunction)、[prev ()](/azure/kusto/query/prevfunction)、[row](/azure/kusto/query/rowcumsumfunction) 関数など) の使用。 このようなケースのいくつかでは、時系列およびユーザー分析関数を使用できます。 また、[range](/azure/kusto/query/rangeoperator)、[sort](/azure/kusto/query/sortoperator)、[order](/azure/kusto/query/orderoperator)、[top](/azure/kusto/query/topoperator)、[top-hitters](/azure/kusto/query/tophittersoperator)、[getschema](/azure/kusto/query/getschemaoperator) 演算子がクエリの末尾以外で使用されている場合にも、非効率的なシリアル化が発生する可能性があります。
-   [dcount ()](/azure/kusto/query/dcount-aggfunction) 集計関数を使用すると、システムでは個別の値の中央コピーが強制的に保持されます。 データのスケールが大きい場合は、dcount 関数の省略可能なパラメーターを使用して精度を下げることを検討してください。
-   多くの場合、[join](/azure/kusto/query/joinoperator?pivots=azuremonitor) 演算子を使用すると全体的な並列処理が低下します。 パフォーマンスに問題がある場合は、代替手段としてシャッフル結合を試してください。
-   リソースのスコープが指定されたクエリで、非常に多くの RBAC 割り当てがある状況では、実行前の RBAC チェックに時間がかかることがあります。 その結果、チェックにかかる時間が長くなり、並列処理が低下する可能性があります。 たとえば、多数のリソースがあり、各リソースに、サブスクリプションやリソース グループのレベルではなくリソースレベルでロールの割り当てが多数存在するサブスクリプションに対してクエリが実行されます。
-   クエリで小さなデータ チャンクを処理している場合、システムがクエリを多数の計算ノードに分散させないため、その並列処理は少なくなります。



## <a name="next-steps"></a>次のステップ

- [Kusto クエリ言語のリファレンス ドキュメント](/azure/kusto/query/)。
