---
title: "Azure Blob Storage との間でのデータのコピー | Microsoft Docs"
description: "Azure Data Factory で BLOB データをコピーする方法について説明します。 サンプルを使用して、Azure Blob Storage と Azure SQL Database の間でデータをコピーする方法を示します。"
keywords: "BLOB データ, Azure BLOB のコピー"
services: data-factory
documentationcenter: 
author: linda33wj
manager: jhubbard
editor: monicar
ms.assetid: bec8160f-5e07-47e4-8ee1-ebb14cfb805d
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/24/2017
ms.author: jingwang
translationtype: Human Translation
ms.sourcegitcommit: d49d7e6b4a9485c2371eb02ac8068adfde9bad6b
ms.openlocfilehash: aa2aabee72d1ca381502f9332df7fb88cf2384a2


---
# <a name="move-data-to-and-from-azure-blob-using-azure-data-factory"></a>Azure Data Factory を使用した Azure BLOB との間でのデータの移動
この記事では、Azure Data Factory のコピー アクティビティを利用して、他のデータ ストアからの BLOB データを Azure BLOB との間で移動する方法について説明します。 この記事は、コピー アクティビティによるデータ移動の概要とサポートされるデータ ストアの組み合わせについて説明する、 [データ移動アクティビティ](data-factory-data-movement-activities.md) に関する記事に基づいています。

## <a name="supported-sources-and-sinks"></a>サポートされているソースとシンク
コピー アクティビティによってソースまたはシンクとしてサポートされているデータ ストアの一覧については、 [サポートされているデータ ストア](data-factory-data-movement-activities.md#supported-data-stores-and-formats) の表をご覧ください。 サポートされる任意のソース データ ストアのデータを、Azure BLOB ストレージに移動したり、Azure BLOB ストレージのデータを、サポートされる任意のシンク データ ストアに移動したりできます。

コピー アクティビティは、汎用 Azure Strage アカウントとコールド/ホット Blob Storage との間でデータをコピーできるようになりました。 このアクティビティは、ブロック BLOB、追加 BLOB、ページ BLOB を読み取りますが、書き込みはブロック BLOB のみをサポートしています。

## <a name="create-pipeline"></a>パイプラインの作成
さまざまなツール/API を使用して、Azure BLOB ストレージとの間でデータを移動するコピー アクティビティでパイプラインを作成できます。  

* コピー ウィザード
* Azure ポータル
* Visual Studio
* Azure PowerShell
* .NET API
* REST API

さまざまな方法でコピー アクティビティによってパイプラインを作成する手順については、 [コピー アクティビティのチュートリアル](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) をご覧ください。

## <a name="copy-data-wizard"></a>データのコピー ウィザード
Azure Blob Storage との間でデータをコピーするパイプラインを作成する最も簡単な方法は、データのコピー ウィザードを使用することです。 データのコピー ウィザードを使用してパイプラインを作成する簡単な手順については、「 [チュートリアル: コピー ウィザードを使用してパイプラインを作成する](data-factory-copy-data-wizard-tutorial.md) 」をご覧ください。

以下の例は、[Azure Portal](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)、または [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) を使用してパイプラインを作成する際に使用できるサンプルの JSON 定義です。 ここでは、Azure Blob Storage と Azure SQL Database の間でデータをコピーする方法を示します。 ただし、任意のソースのデータを、サポートされている任意のシンクに**直接**コピーできます。 詳細については、「[コピー アクティビティを使用したデータの移動](data-factory-data-movement-activities.md)」の「サポートされるデータ ストアと形式」のセクションを参照してください。

## <a name="sample-copy-data-from-azure-blob-to-azure-sql"></a>サンプル: Azure BLOB から Azure SQL にデータをコピーする
次のサンプルは以下を示しています。

1. [AzureSqlDatabase](data-factory-azure-sql-connector.md#azure-sql-linked-service-properties)型のリンクされたサービス。
2. [AzureStorage](#azure-storage-linked-service-properties)型のリンクされたサービス。
3. [AzureBlob](#azure-blob-dataset-type-properties) 型の入力[データセット](data-factory-create-datasets.md)。
4. [AzureSqlTable](data-factory-azure-sql-connector.md#azure-sql-dataset-type-properties) 型の出力[データセット](data-factory-create-datasets.md)。
5. [BlobSource](#azure-blob-copy-activity-type-properties) と [SqlSink](data-factory-azure-sql-connector.md#azure-sql-copy-activity-type-properties) を使用するコピー アクティビティを含む[パイプライン](data-factory-create-pipelines.md)。

このサンプルは、Azure BLOB から Azure SQL テーブルに時系列データを&1; 時間おきにコピーします。 これらのサンプルで使用される JSON プロパティの説明はサンプルに続くセクションにあります。

**Azure SQL のリンクされたサービス:**

```json
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
**Azure Storage のリンクされたサービス:**

```json
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Azure Data Factory では、**AzureStorage** と **AzureStorageSas** という&2; 種類の Azure Storage のリンクされたサービスをサポートしています。 前者ではアカウント キーを含む接続文字列を指定し、後者では Shared Access Signature (SAS) の URI を指定します。 詳細については、「 [リンクされたサービス](#linked-services) 」をご覧ください。  

**Azure BLOB の入力データセット:**

データは新しい BLOB から 1 時間おきに取得されます (頻度: 時間、間隔: 1)。 BLOB のフォルダー パスとファイル名は、処理中のスライスの開始時間に基づき、動的に評価されます。 フォルダー パスでは開始時間の年、月、日の部分を使用し、ファイル名では開始時間の時刻部分を使用します。 "external": "true" の設定により、このテーブルが Data Factory の外部にあり、Data Factory のアクティビティによって生成されたものではないことが Data Factory に通知されます。

```json
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/",
      "fileName": "{Hour}.csv",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": "\n"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```
**Azure SQL の出力データセット:**

このサンプルは Azure SQL Database の「MyTable」というテーブルにデータをコピーします。 BLOB CSV ファイルに含まれることが予想される列数で Azure SQL Database にテーブルを作成します。 新しい行は&1; 時間ごとにテーブルに追加されます。

```json
{
  "name": "AzureSqlOutput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
**コピー アクティビティのあるパイプライン:**

パイプラインには、入力データセットと出力データセットを使用するように構成され、1 時間おきに実行するようにスケジュールされているコピー アクティビティが含まれています。 パイプラインの JSON 定義で、**source** 型が **BlobSource** に設定され、**sink** 型が **SqlSink** に設定されています。

```json
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[  
      {
        "name": "AzureBlobtoSQL",
        "description": "Copy Activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink"
          }
        },
       "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
      ]
   }
}
```
## <a name="sample-copy-data-from-azure-sql-to-azure-blob"></a>サンプル: Azure SQL から Azure BLOB にデータをコピーする
次のサンプルは以下を示しています。

1. [AzureSqlDatabase](data-factory-azure-sql-connector.md#azure-sql-linked-service-properties)型のリンクされたサービス。
2. [AzureStorage](#azure-storage-linked-service-properties)型のリンクされたサービス。
3. [AzureSqlTable](data-factory-azure-sql-connector.md#azure-sql-dataset-type-properties) 型の入力[データセット](data-factory-create-datasets.md)。
4. [AzureBlob](#azure-blob-dataset-type-properties) 型の出力[データセット](data-factory-create-datasets.md)。
5. [SqlSource](data-factory-azure-sql-connector.md#azure-sql-copy-activity-type-properties) と [BlobSink](#azure-blob-copy-activity-type-properties) を使用するコピー アクティビティを含む[パイプライン](data-factory-create-pipelines.md)。

このサンプルは、Azure SQL テーブルから Azure BLOB に時系列データを&1; 時間おきにコピーします。 これらのサンプルで使用される JSON プロパティの説明はサンプルに続くセクションにあります。

**Azure SQL のリンクされたサービス:**

```json
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
**Azure Storage のリンクされたサービス:**

```json
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Azure Data Factory では、**AzureStorage** と **AzureStorageSas** という&2; 種類の Azure Storage のリンクされたサービスをサポートしています。 前者ではアカウント キーを含む接続文字列を指定し、後者では Shared Access Signature (SAS) の URI を指定します。 詳細については、「 [リンクされたサービス](#linked-services) 」をご覧ください。  

**Azure SQL の入力データセット:**

このサンプルでは、Azure SQL で「MyTable」という名前のテーブルを作成し、時系列データ用に「timestampcolumn」という名前の列が含まれているものと想定しています。

"external": "true" の設定により、このテーブルが Data Factory の外部にあり、Data Factory のアクティビティによって生成されたものではないことが Data Factory サービスに通知されます。

```json
{
  "name": "AzureSqlInput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Azure BLOB の出力データセット:**

データは新しい BLOB に 1 時間おきに書き込まれます (頻度: 時間、間隔: 1)。 BLOB のフォルダー パスは、処理中のスライスの開始時間に基づき、動的に評価されます。 フォルダー パスは開始時間の年、月、日、時刻の部分を使用します。

```json
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}/",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**コピー アクティビティのあるパイプライン:**

パイプラインには、入力データセットと出力データセットを使用するように構成され、1 時間おきに実行するようにスケジュールされているコピー アクティビティが含まれています。 パイプライン JSON 定義で、**source** 型が **SqlSource** に設定され、**sink** 型が **BlobSink** に設定されています。 **SqlReaderQuery** プロパティに指定されている SQL クエリは過去のデータを選択してコピーします。

```json
{  
    "name":"SamplePipeline",
    "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[  
              {
                "name": "AzureSQLtoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                  {
                    "name": "AzureSQLInput"
                  }
                ],
                "outputs": [
                  {
                    "name": "AzureBlobOutput"
                  }
                ],
                "typeProperties": {
                    "source": {
                        "type": "SqlSource",
                        "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                      },
                      "sink": {
                        "type": "BlobSink"
                      }
                },
                   "scheduler": {
                      "frequency": "Hour",
                      "interval": 1
                },
                "policy": {
                      "concurrency": 1,
                      "executionPriorityOrder": "OldestFirst",
                      "retry": 0,
                      "timeout": "01:00:00"
                }
              }
         ]
    }
}
```
## <a name="linked-services"></a>リンクされたサービス
サンプルでは、 **AzureStorage** 型のリンクされたサービスを使用して、Azure Storage アカウントをデータ ファクトリにリンクしています。 次の表は、Azure Storage のリンクされたサービスに固有の JSON 要素の説明をまとめたものです。

Azure BLOB ストレージを Azure Data Factory にリンクするために使用できるリンクされたサービスは&2; 種類あります。 それらは、**AzureStorage** のリンクされたサービスと **AzureStorageSas** のリンクされたサービスです。 Azure Storage のリンクされたサービスは、Azure Storage へのグローバル アクセスを Data Factory に提供します。 一方、Azure Storage SAS (Shared Access Signature) のリンクされたサービスは、Azure Storage への制限付き/期限付きアクセスを Data Factory に提供します。 これら&2; つのリンクされたサービスには、これ以外の相違点はありません。 ニーズに適したリンクされたサービスを選択します。 以下のセクションで、これら&2; つのリンクされたサービスについて詳しく説明します。

[!INCLUDE [data-factory-azure-storage-linked-services](../../includes/data-factory-azure-storage-linked-services.md)]

## <a name="azure-blob-dataset-type-properties"></a>Azure BLOB データセットの type プロパティ
サンプルでは、 **AzureBlob** 型のデータセットを使用して、Azure BLOB ストレージの BLOB コンテナーとフォルダーを表しています。

データセットの定義に利用できる JSON のセクションとプロパティの完全一覧については、[データセットの作成](data-factory-create-datasets.md)に関する記事を参照してください。 データセット JSON の構造、可用性、ポリシーなどのセクションは、データセットのすべての型 (Azure SQL、Azure BLOB、Azure テーブルなど) でほぼ同じです。

**typeProperties** セクションは、データセットの型ごとに異なり、データ ストアのデータの場所や書式などに関する情報を提供します。 **AzureBlob** 型のデータセットの typeProperties セクションには次のプロパティがあります。

| プロパティ | 説明 | 必須 |
| --- | --- | --- |
| folderPath |BLOB ストレージのコンテナーとフォルダーのパス。 例: myblobcontainer\myblobfolder\ |はい |
| fileName |BLOB の名前です。 fileName は省略可能で、大文字と小文字を区別します。<br/><br/>fileName を指定すると、アクティビティ (コピーを含む) は特定の BLOB で動作します。<br/><br/>fileName が指定されていない場合、コピーには入力データセットの folderPath のすべての BLOB が含まれます。<br/><br/>出力データセットに fileName が指定されていない場合、生成されるファイル名は次の形式になります: Data.<Guid>.txt (例: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |いいえ |
| partitionedBy |partitionedBy は任意のプロパティです。 これを使用し、時系列データに動的な folderPath と fileName を指定できます。 たとえば、1 時間ごとのデータに対して folderPath をパラメーター化できます。 詳細と例については、「 [partitionedBy プロパティの使用](#using-partitionedBy-property) 」をご覧ください。 |なし |
| BlobSink の format | 次のファイル形式がサポートされます: **TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 形式の **type** プロパティをいずれかの値に設定します。 詳細については、[Text Format](#specifying-textformat)、[Json Format](#specifying-jsonformat)、[Avro Format](#specifying-avroformat)、[Orc Format](#specifying-orcformat)、[Parquet Format](#specifying-parquetformat) の各セクションを参照してください。 <br><br> ファイルベースのストア間で**ファイルをそのままコピー** (バイナリ コピー) する場合は、入力と出力の両方のデータセット定義で format セクションをスキップします。 |なし |
| compression | データの圧縮の種類とレベルを指定します。 サポートされる種類は **GZip**、**Deflate**、**BZip2**、**ZipDeflate** です。サポートされるレベルは **Optimal** と **Fastest** です。 詳細については、「[圧縮の指定](#specifying-compression)」セクションを参照してください。 |いいえ |

### <a name="using-partitionedby-property"></a>partitionedBy プロパティの使用
前のセクションで説明したように、 **partitionedBy** セクション、Data Factory マクロ、特定のデータ スライスの開始時刻と終了時刻を示すシステム変数の SliceStart と SliceEnd を使用して、時系列データの動的な folderPath と fileName を指定できます。

時系列データセット、スケジュール設定、およびスライスの詳細については、[データセットの作成](data-factory-create-datasets.md)に関する記事および[スケジュール設定と実行](data-factory-scheduling-and-execution.md)に関する記事をご覧ください。

#### <a name="sample-1"></a>サンプル 1

```json
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```

この例では、{Slice} は、指定された形式 (YYYYMMDDHH) で、Data Factory システム変数の SliceStart の値に置き換えられます。 SliceStart はスライスの開始時刻です。 folderPath はスライスごとに異なります。 例: wikidatagateway/wikisampledataout/2014100103 または wikidatagateway/wikisampledataout/2014100104

#### <a name="sample-2"></a>サンプル 2

```json
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
 [
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```

この例では、SliceStart の年、月、日、時刻が folderPath プロパティと fileName プロパティで使用される個別の変数に抽出されます。

[!INCLUDE [data-factory-file-format](../../includes/data-factory-file-format.md)]

[!INCLUDE [data-factory-compression](../../includes/data-factory-compression.md)]

## <a name="azure-blob-copy-activity-type-properties"></a>Azure BLOB のコピー アクティビティの type プロパティ
アクティビティの定義に利用できるセクションとプロパティの完全な一覧については、[パイプラインの作成](data-factory-create-pipelines.md)に関する記事を参照してください。 プロパティ (名前、説明、入力データセット、出力データセット、ポリシーなど) は、あらゆる種類のアクティビティで使用できます。

一方、アクティビティの typeProperties セクションで使用できるプロパティは、各アクティビティの種類によって異なります。 コピー アクティビティの場合、ソースとシンクの種類によって異なります

Azure BLOB ストレージからデータを移動する場合は、コピー アクティビティのソースの種類を **BlobSource**に設定します。 同様に、Azure BLOB ストレージにデータを移動する場合は、コピー アクティビティのシンクの種類を **BlobSink**に設定します。 このセクションでは、BlobSource と BlobSink でサポートされるプロパティの一覧を示します。

**BlobSource** の **typeProperties** セクションでは次のプロパティがサポートされます。

| プロパティ | 説明 | 使用できる値 | 必須 |
| --- | --- | --- | --- |
| recursive |データをサブ フォルダーから再帰的に読み取るか、指定したフォルダーからのみ読み取るかを指定します。 |True (既定値)、False |いいえ |

**BlobSink** の **typeProperties** セクションでは次のプロパティがサポートされます。

| プロパティ | 説明 | 使用できる値 | 必須 |
| --- | --- | --- | --- |
| copyBehavior |ソースが BlobSource または FileSystem である場合のコピー動作を定義します。 |**PreserveHierarchy:** ターゲット フォルダー内でファイル階層を保持します。 ソース フォルダーに対するソース ファイルの相対パスと、ターゲット フォルダーに対するターゲット ファイルの相対パスが一致します。<br/><br/>**FlattenHierarchy**: ソース フォルダーのすべてのファイルがターゲット フォルダーの最初のレベルになります。 ターゲット ファイルは、自動生成された名前になります。 <br/><br/>**MergeFiles: (既定値)** ソース フォルダーのすべてのファイルを&1; つのファイルにマージします。 ファイル/Blob の名前を指定した場合、マージされたファイル名は指定した名前になります。それ以外は自動生成されたファイル名になります。 |いいえ |

**BlobSource** は、下位互換性のために次の&2; つのプロパティもサポートしています。

* **treatEmptyAsNull**: null または空の文字列を null 値として処理するかどうかを指定します。
* **skipHeaderLineCount** - スキップする必要がある行数を指定します。 入力データセットで TextFormat を利用しているときにのみ適用されます。

同様に、 **BlobSink** も、下位互換性のために次のプロパティをサポートしています。

* **blobWriterAddHeader**: 出力データセットへの書き込み中に列定義のヘッダーを追加するかどうかを指定します。

データセットは、同じ機能を実装するプロパティとして、**treatEmptyAsNull**、**skipLineCount**、**firstRowAsHeader** をサポートするようになりました。

次の表は、BLOB ソース/シンク プロパティの代わりに新しいデータセットのプロパティを使用するためのガイドラインをまとめたものです。

| コピー アクティビティのプロパティ | データセットのプロパティ |
|:--- |:--- |
| BlobSource の skipHeaderLineCount |skipLineCount と firstRowAsHeader。 最初に行がスキップされ、その後、最初の行がヘッダーとして読み取られます。 |
| BlobSource の treatEmptyAsNull |入力データセットの treatEmptyAsNull |
| BlobSink の blobWriterAddHeader |出力データセットの firstRowAsHeader |

これらのプロパティの詳細については、「 [TextFormat の指定](#specifying-textformat) 」セクションを参照してください。    

### <a name="recursive-and-copybehavior-examples"></a>recursive と copyBehavior の例
このセクションでは、recursive 値と copyBhavior 値の組み合わせごとに、Copy 操作で行われる動作について説明します。

| recursive | copyBehavior | 行われる動作 |
| --- | --- | --- |
| true |preserveHierarchy |ソース フォルダー Folder1 が次のような構造の場合: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>ターゲット フォルダー Folder1 はソースと同じ構造で作成されます。<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 |
| true |flattenHierarchy |ソース フォルダー Folder1 が次のような構造の場合: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>ターゲットの Folder1 は、次の構造で作成されます。 <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 の自動生成された名前<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2 の自動生成された名前<br/>&nbsp;&nbsp;&nbsp;&nbsp;File3 の自動生成された名前<br/>&nbsp;&nbsp;&nbsp;&nbsp;File4 の自動生成された名前<br/>&nbsp;&nbsp;&nbsp;&nbsp;File5 の自動生成された名前 |
| true |mergeFiles |ソース フォルダー Folder1 が次のような構造の場合: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>ターゲットの Folder1 は、次の構造で作成されます。 <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1、File2、File3、File4、File5の内容は、自動生成されたファイル名を持つ 1 つのファイルにマージされます。 |
| false |preserveHierarchy |ソース フォルダー Folder1 が次のような構造の場合: <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>ターゲット フォルダー Folder1 は、次の構造で作成されます。<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/><br/>Subfolder1 と File3、File4、File5 は取得されません。 |
| false |flattenHierarchy |ソース フォルダー Folder1 が次のような構造の場合:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>ターゲット フォルダー Folder1 は、次の構造で作成されます。<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 の自動生成された名前<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2 の自動生成された名前<br/><br/><br/>Subfolder1 と File3、File4、File5 は取得されません。 |
| false |mergeFiles |ソース フォルダー Folder1 が次のような構造の場合:<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>ターゲット フォルダー Folder1 は、次の構造で作成されます。<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1、File2 の内容は、自動生成されたファイル名を持つ&1; つのファイルにマージされます。 File1 の自動生成された名前<br/><br/>Subfolder1 と File3、File4、File5 は取得されません。 |

[!INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

[!INCLUDE [data-factory-type-conversion-sample](../../includes/data-factory-type-conversion-sample.md)]

[!INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]

## <a name="performance-and-tuning"></a>パフォーマンスとチューニング
Azure Data Factory でのデータ移動 (コピー アクティビティ) のパフォーマンスに影響する主な要因と、パフォーマンスを最適化するための各種方法については、「[コピー アクティビティのパフォーマンスとチューニングに関するガイド](data-factory-copy-activity-performance.md)」を参照してください。



<!--HONumber=Jan17_HO2-->


