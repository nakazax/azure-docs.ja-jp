---
author: aahill
ms.service: cognitive-services
ms.topic: include
ms.date: 02/26/2019
ms.author: aahi
ms.openlocfilehash: 8e3f08be70a2a6b932b849ff28dd762828abf229
ms.sourcegitcommit: 3c925b84b5144f3be0a9cd3256d0886df9fa9dc0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/28/2020
ms.locfileid: "78155456"
---
<a name="HOLTop"></a>

#### <a name="version-30-preview"></a>[Version 3.0-preview](#tab/version-3)

[v3 リファレンス ドキュメント](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-ai-textanalytics/1.0.0b2/azure.ai.textanalytics.html) | [v3 ライブラリのソース コード](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics) | [v3 パッケージ (PiPy)](https://pypi.org/project/azure-ai-textanalytics/) | [v3 サンプル](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/samples)

#### <a name="version-21"></a>[バージョン 2.1](#tab/version-2)

[v2 リファレンス ドキュメント](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/textanalytics?view=azure-python) | [v2 ライブラリのソース コード](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-language-textanalytics) | [v2 パッケージ (PiPy)](https://pypi.org/project/azure-cognitiveservices-language-textanalytics/) | [v2 サンプル](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples)

---

## <a name="prerequisites"></a>前提条件

* Azure サブスクリプション - [無料アカウントを作成します](https://azure.microsoft.com/free/)
* [Python 3.x](https://www.python.org/)
* Azure サブスクリプションを入手したら、Azure portal で <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="Text Analytics リソースを作成"  target="_blank">Text Analytics リソースを作成<span class="docon docon-navigate-external x-hidden-focus"></span></a>し、キーとエンドポイントを取得します。 
    * アプリケーションを Text Analytics API に接続するには、作成するリソースのキーとエンドポイントが必要です。 この作業は、このクイックスタートの中で後から行います。
    * Free 価格レベルを使ってサービスを試用し、後から運用環境用の有料レベルにアップグレードすることができます。

## <a name="setting-up"></a>設定

### <a name="install-the-client-library"></a>クライアント ライブラリをインストールする

Python をインストールしたら、次を使用してクライアント ライブラリをインストールすることができます。

#### <a name="version-30-preview"></a>[Version 3.0-preview](#tab/version-3)

```console
pip install azure-ai-textanalytics
```

#### <a name="version-21"></a>[バージョン 2.1](#tab/version-2)

```console
pip install --upgrade azure-cognitiveservices-language-textanalytics
```

---

### <a name="create-a-new-python-application"></a>新しい Python アプリケーションを作成する

新しい Python ファイルを作成し、リソースの Azure エンドポイントとサブスクリプション キー用の変数を作成します。

[!INCLUDE [text-analytics-find-resource-information](../find-azure-resource-info.md)]

```python
key = "<paste-your-text-analytics-key-here>"
endpoint = "<paste-your-text-analytics-endpoint-here>"
```


## <a name="object-model"></a>オブジェクト モデル

#### <a name="version-30-preview"></a>[Version 3.0-preview](#tab/version-3)

Text Analytics クライアントは、キーを使用して Azure に対して認証を行う `TextAnalyticsClient` オブジェクトです。 このクライアントには、テキストをバッチとして分析するためのメソッドがいくつか備わっています。 

バッチ処理時に、テキストは、`documents` (使用したメソッドに応じて `id`、`text`、および `language` の各属性の組み合わせを保持する `dictionary` オブジェクト) の一覧として API に送信されます。 `text` 属性には、分析対象のテキストが元の `language` で格納され、`id` には任意の値を指定できます。 

応答オブジェクトは、各ドキュメントの分析情報を格納するリストです。 

#### <a name="version-21"></a>[バージョン 2.1](#tab/version-2)

Text Analytics クライアントは、ご利用のキーを使用して Azure に対して認証を行う [TextAnalyticsClient](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-textanalytics/azure.cognitiveservices.language.textanalytics.textanalyticsclient?view=azure-python) オブジェクトです。 このクライアントには、テキストを単一の文字列として、またはバッチとして分析するためのメソッドがいくつか備わっています。 

テキストは、`documents` (使用したメソッドに応じて `id`、`text`、`language` の各属性の組み合わせを保持する `dictionary` オブジェクト) のリストとして API に送信されます。 `text` 属性には、分析対象のテキストが元の `language` で格納され、`id` には任意の値を指定できます。 

---

## <a name="code-examples"></a>コード例

これらのコード スニペットでは、Python 用 Text Analytics クライアント ライブラリを使用して以下のタスクを実行する方法が示されています。

* [クライアントを認証する](#authenticate-the-client)
* [感情分析](#sentiment-analysis)
* [言語検出](#language-detection)
* [名前付きエンティティの認識](#named-entity-recognition-ner) 
* [エンティティ リンク設定](#entity-linking)
* [キー フレーズ抽出](#key-phrase-extraction)

## <a name="authenticate-the-client"></a>クライアントを認証する

#### <a name="version-30-preview"></a>[Version 3.0-preview](#tab/version-3)

上で作成した `key` と `endpoint` を使用して `TextAnalyticsClient` オブジェクトをインスタンス化する関数を作成します。 次に、新しいクライアントを作成します。 

```python
from azure.ai.textanalytics import TextAnalyticsClient, TextAnalyticsApiKeyCredential

def authenticate_client():
    ta_credential = TextAnalyticsApiKeyCredential(key)
    text_analytics_client = TextAnalyticsClient(
            endpoint=endpoint, credential=ta_credential)
    return text_analytics_client

client = authenticate_client()
```

#### <a name="version-21"></a>[バージョン 2.1](#tab/version-2)

[!code-python[imports statements](~/samples-cognitive-services-python-sdk/samples/language/text_analytics_samples.py?name=imports)]

上で作成した `key` と `endpoint` を使用して `TextAnalyticsClient` オブジェクトをインスタンス化する関数を作成します。 次に、新しいクライアントを作成します。 

[!code-python[version 2 authentication](~/samples-cognitive-services-python-sdk/samples/language/text_analytics_samples.py?name=authentication)]

--- 

## <a name="sentiment-analysis"></a>センチメント分析

#### <a name="version-30-preview"></a>[Version 3.0-preview](#tab/version-3)

クライアントを引数として受け取り、`analyze_sentiment()` 関数を呼び出す、`sentiment_analysis_example()` という名前の新しい関数を作成します。 返される応答オブジェクトには、入力ドキュメント全体のセンチメント ラベルとスコアが含まれます。また、各文の感情分析も含まれます。


```python
def sentiment_analysis_example(client):

    document = ["I had the best day of my life. I wish you were there with me."]
    response = client.analyze_sentiment(inputs=document)[0]
    print("Document Sentiment: {}".format(response.sentiment))
    print("Overall scores: positive={0:.3f}; neutral={1:.3f}; negative={2:.3f} \n".format(
        response.sentiment_scores.positive,
        response.sentiment_scores.neutral,
        response.sentiment_scores.negative,
    ))
    for idx, sentence in enumerate(response.sentences):
        print("[Offset: {}, Length: {}]".format(sentence.offset, sentence.length))
        print("Sentence {} sentiment: {}".format(idx+1, sentence.sentiment))
        print("Sentence score:\nPositive={0:.3f}\nNeutral={1:.3f}\nNegative={2:.3f}\n".format(
            sentence.sentiment_scores.positive,
            sentence.sentiment_scores.neutral,
            sentence.sentiment_scores.negative,
        ))

            
sentiment_analysis_example(client)
```

### <a name="output"></a>出力

```console
Document Sentiment: positive
Overall scores: positive=1.000; neutral=0.000; negative=0.000 

[Offset: 0, Length: 30]
Sentence 1 sentiment: positive
Sentence score:
Positive=1.000
Neutral=0.000
Negative=0.000

[Offset: 31, Length: 30]
Sentence 2 sentiment: neutral
Sentence score:
Positive=0.210
Neutral=0.770
Negative=0.020
```

#### <a name="version-21"></a>[バージョン 2.1](#tab/version-2)

クライアント オブジェクトを認証し、[sentiment()](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-textanalytics/azure.cognitiveservices.language.textanalytics.textanalyticsclient?view=azure-python#sentiment-show-stats-none--documents-none--custom-headers-none--raw-false----operation-config-) 関数を呼び出します。 結果を反復処理し、各ドキュメントの ID とセンチメント スコアを出力します。 0 に近いスコアは否定的センチメント、1 に近いスコアは肯定的センチメントを示します。

[!code-python[sentiment analysis](~/samples-cognitive-services-python-sdk/samples/language/text_analytics_samples.py?name=sentimentAnalysis)]

### <a name="output"></a>出力

```console
Document ID: 1 , Sentiment Score: 0.87
Document ID: 2 , Sentiment Score: 0.11
Document ID: 3 , Sentiment Score: 0.44
Document ID: 4 , Sentiment Score: 1.00
```

---

## <a name="language-detection"></a>言語検出

#### <a name="version-30-preview"></a>[Version 3.0-preview](#tab/version-3)

クライアントを引数として受け取り、`detect_language()` 関数を呼び出す、`language_detection_example()` という名前の新しい関数を作成します。 返される応答オブジェクトには、成功した場合は検出された言語が `primary_language` に含まれ、そうでない場合は `error` が含まれます。

> [!Tip]
> 場合によっては、入力に基づいて言語を明確に区別するのが困難なことがあります。 `country_hint` パラメーターを使用して、2 文字の国番号を指定できます。 API の既定では、既定の countryHint として "US" が使用されます。この動作を削除するには、この値を空の文字列に設定して (`country_hint : ""`)、このパラメーターをリセットします。 

```python
def language_detection_example(client):
    try:
        document = ["Ce document est rédigé en Français."]
        response = client.detect_language(inputs = document, country_hint = 'us')[0]
        print("Language: ", response.primary_language.name)

    except Exception as err:
        print("Encountered exception. {}".format(err))
language_detection_example(client)
```


### <a name="output"></a>出力

```console
Language:  French
```

#### <a name="version-21"></a>[バージョン 2.1](#tab/version-2)

以前に作成したクライアントを使用して、[detect_language()](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-textanalytics/azure.cognitiveservices.language.textanalytics.textanalyticsclient?view=azure-python#detect-language-show-stats-none--documents-none--custom-headers-none--raw-false----operation-config-) を呼び出して結果を取得します。 結果を反復処理し、各ドキュメントの ID と最初に返された言語を印刷します。

[!code-python[language detection](~/samples-cognitive-services-python-sdk/samples/language/text_analytics_samples.py?name=languageDetection)]


### <a name="output"></a>出力

```console
Document ID: 1 , Language: English
Document ID: 2 , Language: Spanish
Document ID: 3 , Language: Chinese_Simplified
```

---

## <a name="named-entity-recognition-ner"></a>名前付きエンティティの認識 (NER)

#### <a name="version-30-preview"></a>[Version 3.0-preview](#tab/version-3)

> [!NOTE]
> バージョン `3.0-preview`:
> * NER には、個人情報を検出するための専用のメソッドが含まれます。 
> * エンティティ リンク設定は、NER から切り離された要求です。

クライアントを引数として受け取り、`recognize_entities()` 関数を呼び出して、結果を反復処理する、`entity_recognition_example` という名前の新しい関数を作成します。 返される応答オブジェクトには、成功した場合は検出されたエンティティの一覧が `entity` に含まれ、そうでない場合は `error` が含まれます。 検出されたエンティティごとに、カテゴリとサブカテゴリ (ある場合) を出力します。

```python
def entity_recognition_example(client):

    try:
        document = ["I had a wonderful trip to Seattle last week."]
        result = client.recognize_entities(inputs= document)[0]

        print("Named Entities:\n")
        for entity in result.entities:
                print("\tText: \t", entity.text, "\tCategory: \t", entity.category, "\tSubCategory: \t", entity.subcategory,
                      "\n\tOffset: \t", entity.offset, "\tLength: \t", entity.offset, 
                      "\tConfidence Score: \t", round(entity.score, 3), "\n")

    except Exception as err:
        print("Encountered exception. {}".format(err))
entity_recognition_example(client)
```

### <a name="output"></a>出力

```console
Named Entities:

    Text:    Seattle    Category:    Location   SubCategory:     GPE 
    Offset:      26     Length:      26     Confidence Score:    0.92 

    Text:    last week  Category:    DateTime   SubCategory:     DateRange 
    Offset:      34     Length:      34     Confidence Score:    0.8 
```

## <a name="using-ner-to-detect-personal-information"></a>NER を使用して個人情報を検出する

クライアントを引数として受け取り、`recognize_pii_entities()` 関数を呼び出して結果を取得する、`entity_pii_example()` という名前の新しい関数を作成します。 結果を反復処理し、エンティティを出力します。

```python
def entity_pii_example(client):

        document = ["Insurance policy for SSN on file 123-12-1234 is here by approved."]


        result = client.recognize_pii_entities(inputs= document)[0]
        
        print("Personally Identifiable Information Entities: ")
        for entity in result.entities:
            print("\tText: ",entity.text,"\tCategory: ", entity.category,"\tSubCategory: ", entity.subcategory)
            print("\t\tOffset: ", entity.offset, "\tLength: ", entity.length, "\tScore: {0:.3f}".format(entity.score), "\n")
        
entity_pii_example(client)
```

### <a name="output"></a>出力

```console
Personally Identifiable Information Entities: 
    Text:  123-12-1234  Category:  U.S. Social Security Number (SSN)    SubCategory:  None
        Offset:  33     Length:  11     Score: 0.850 
```


## <a name="entity-linking"></a>Entity Linking

クライアントを引数として受け取り、`recognize_linked_entities()` 関数を呼び出して、結果を反復処理する、`entity_linking_example()` という名前の新しい関数を作成します。 返される応答オブジェクトには、成功した場合は検出されたエンティティの一覧が `entities` に含まれ、そうでない場合は `error` が含まれます。 リンクされたエンティティは一意に識別されるため、同じエンティティの出現は、`match` オブジェクトの一覧として `entity` オブジェクトの下にグループ化されます。

```python
def entity_linking_example(client):

    try:
        document = ["""Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, 
        to develop and sell BASIC interpreters for the Altair 8800. 
        During his career at Microsoft, Gates held the positions of chairman,
        chief executive officer, president and chief software architect, 
        while also being the largest individual shareholder until May 2014."""]
        result = client.recognize_linked_entities(inputs= document)[0]

        print("Linked Entities:\n")
        for entity in result.entities:
            print("\tName: ", entity.name, "\tId: ", entity.id, "\tUrl: ", entity.url,
            "\n\tData Source: ", entity.data_source)
            print("\tMatches:")
            for match in entity.matches:
                print("\t\tText:", match.text)
                print("\t\tScore: {0:.3f}".format(match.score), "\tOffset: ", match.offset, 
                      "\tLength: {}\n".format(match.length))
            
    except Exception as err:
        print("Encountered exception. {}".format(err))
entity_linking_example(client)
```

### <a name="output"></a>出力

```console
Linked Entities:

    Name:  Altair 8800  Id:  Altair 8800    Url:  https://en.wikipedia.org/wiki/Altair_8800 
    Data Source:  Wikipedia
    Matches:
        Text: Altair 8800
        Score: 0.777    Offset:  125    Length: 11

    Name:  Bill Gates   Id:  Bill Gates     Url:  https://en.wikipedia.org/wiki/Bill_Gates 
    Data Source:  Wikipedia
    Matches:
        Text: Bill Gates
        Score: 0.555    Offset:  25     Length: 10

        Text: Gates
        Score: 0.555    Offset:  179    Length: 5

    Name:  Paul Allen   Id:  Paul Allen     Url:  https://en.wikipedia.org/wiki/Paul_Allen 
    Data Source:  Wikipedia
    Matches:
        Text: Paul Allen
        Score: 0.533    Offset:  40     Length: 10

    Name:  Microsoft    Id:  Microsoft  Url:  https://en.wikipedia.org/wiki/Microsoft 
    Data Source:  Wikipedia
    Matches:
        Text: Microsoft
        Score: 0.469    Offset:  0  Length: 9

        Text: Microsoft
        Score: 0.469    Offset:  168    Length: 9

    Name:  April 4  Id:  April 4    Url:  https://en.wikipedia.org/wiki/April_4 
    Data Source:  Wikipedia
    Matches:
        Text: April 4
        Score: 0.248    Offset:  54     Length: 7

    Name:  BASIC    Id:  BASIC  Url:  https://en.wikipedia.org/wiki/BASIC 
    Data Source:  Wikipedia
    Matches:
        Text: BASIC
        Score: 0.281    Offset:  98     Length: 5

```

#### <a name="version-21"></a>[バージョン 2.1](#tab/version-2)

> [!NOTE]
> バージョン 2.1 では、エンティティ リンク設定が NER の応答に含まれています。

以前に作成したクライアントを使用して、[entities()](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-textanalytics/azure.cognitiveservices.language.textanalytics.textanalyticsclient?view=azure-python#entities-show-stats-none--documents-none--custom-headers-none--raw-false----operation-config-) 関数を呼び出して結果を取得します。 続いて、結果を反復処理し、各ドキュメントの ID とそれに含まれているエンティティを印刷します。

[!code-python[Entity recognition](~/samples-cognitive-services-python-sdk/samples/language/text_analytics_samples.py?name=entityRecognition)]

### <a name="output"></a>出力

```console
Document ID: 1
        Name: Microsoft,        Type: Organization,     Sub-Type: N/A
        Offset: 0, Length: 9,   Score: 1.0

        Name: Bill Gates,       Type: Person,   Sub-Type: N/A
        Offset: 25, Length: 10, Score: 0.999847412109375

        Name: Paul Allen,       Type: Person,   Sub-Type: N/A
        Offset: 40, Length: 10, Score: 0.9988409876823425

        Name: April 4,  Type: Other,    Sub-Type: N/A
        Offset: 54, Length: 7,  Score: 0.8

        Name: April 4, 1975,    Type: DateTime, Sub-Type: Date
        Offset: 54, Length: 13, Score: 0.8

        Name: BASIC,    Type: Other,    Sub-Type: N/A
        Offset: 89, Length: 5,  Score: 0.8

        Name: Altair 8800,      Type: Other,    Sub-Type: N/A
        Offset: 116, Length: 11,        Score: 0.8

Document ID: 2
        Name: Microsoft,        Type: Organization,     Sub-Type: N/A
        Offset: 21, Length: 9,  Score: 0.999755859375

        Name: Redmond (Washington),     Type: Location, Sub-Type: N/A
        Offset: 60, Length: 7,  Score: 0.9911284446716309

        Name: 21 kilómetros,    Type: Quantity, Sub-Type: Dimension
        Offset: 71, Length: 13, Score: 0.8

        Name: Seattle,  Type: Location, Sub-Type: N/A
        Offset: 88, Length: 7,  Score: 0.9998779296875
```

---

## <a name="key-phrase-extraction"></a>キー フレーズの抽出


#### <a name="version-30-preview"></a>[Version 3.0-preview](#tab/version-3)

クライアントを引数として受け取り、`extract_key_phrases()` 関数を呼び出す、`key_phrase_extraction_example()` という名前の新しい関数を作成します。 成功した場合は `key_phrases` 内で検出されたキー フレーズのリストが結果に含まれ、失敗した場合は `error` が含まれます。 検出されたキー フレーズを出力します。

```python
def key_phrase_extraction_example(client):

    try:
        document = ["My cat might need to see a veterinarian."]

        response = client.extract_key_phrases(inputs= document)[0]

        if not response.is_error:
            print("\tKey Phrases:")
            for phrase in response.key_phrases:
                print("\t\t", phrase)
        else:
            print(response.id, response.error)

    except Exception as err:
        print("Encountered exception. {}".format(err))
        
key_phrase_extraction_example(client)
```


### <a name="output"></a>出力

```console
    Key Phrases:
         cat
         veterinarian
```

#### <a name="version-21"></a>[バージョン 2.1](#tab/version-2)

以前に作成したクライアントを使用して、[key_phrases()](https://docs.microsoft.com/python/api/azure-cognitiveservices-language-textanalytics/azure.cognitiveservices.language.textanalytics.textanalyticsclient?view=azure-python#key-phrases-show-stats-none--documents-none--custom-headers-none--raw-false----operation-config-) 関数を呼び出して結果を取得します。 続いて、結果を反復処理し、各ドキュメントの ID とそれに含まれているキー フレーズを印刷します。

[!code-python[key phrase extraction](~/samples-cognitive-services-python-sdk/samples/language/text_analytics_samples.py?name=keyPhrases)]


### <a name="output"></a>出力

```console
Document ID: 1
         Key phrases:
                幸せ
Document ID: 2
         Key phrases:
                Stuttgart
                Hotel
                Fahrt
                Fu
Document ID: 3
         Key phrases:
                cat
                veterinarian
Document ID: 4
         Key phrases:
                fútbol
```

---