---
title: "インポート ジョブ用のハード ドライブを準備するためのサンプル ワークフロー | Microsoft Docs"
description: "Azure Import/Export サービスでインポート ジョブ用のドライブを準備するプロセスの手順について説明します。"
author: muralikk
manager: syadav
editor: tysonn
services: storage
documentationcenter: 
ms.assetid: 6eb1b1b7-c69f-4365-b5ef-3cd5e05eb72a
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/09/2017
ms.author: muralikk
translationtype: Human Translation
ms.sourcegitcommit: fbcc295284d060b9a8bc5cf1cb9f8b2fca566db4
ms.openlocfilehash: 775deb2717e5062e8eb4be7f10dd5bf3c2682f1b


---

# <a name="sample-workflow-to-prepare-hard-drives-for-an-import-job"></a>インポート ジョブ用のハード ドライブを準備するためのサンプル ワークフロー
このトピックでは、インポート ジョブ用のドライブを準備するプロセスの手順について説明します。  
  
この例では、`mystorageaccount` という名前の Microsoft Azure Storage アカウントに次のデータをインポートします。  
  
|場所|Description|  
|--------------|-----------------|  
|H:\Video|一連のビデオ (合計 5 TB)。|  
|H:\Photo|一連の写真 (合計 30 GB)。|  
|K:\Temp\FavoriteMovie.ISO|Blu-Ray™ ディスク イメージ (25 GB)。|  
|\\\bigshare\john\music|ネットワーク共有上の一連の音楽ファイル (合計 10 GB)。|  
  
インポート ジョブでは、ストレージ アカウント内の次のインポート先にこのデータをインポートします。  
  
|から|インポート先の仮想ディレクトリまたは BLOB|  
|------------|-------------------------------------------|  
|H:\Video|https://mystorageaccount.blob.core.windows.net/video|  
|H:\Photo|https://mystorageaccount.blob.core.windows.net/photo|  
|K:\Temp\FavoriteMovie.ISO|https://mystorageaccount.blob.core.windows.net/favorite/FavoriteMovies.ISO|  
|\\\bigshare\john\music|https://mystorageaccount.blob.core.windows.net/music|  
  
このマッピングにより、`H:\Video\Drama\GreatMovie.mov` ファイルが `https://mystorageaccount.blob.core.windows.net/video/Drama/GreatMovie.mov` BLOB にインポートされます。  
  
次に、必要なハード ドライブの数を決定するために、データのサイズを計算します。  
  
`5TB + 30GB + 25GB + 10GB = 5TB + 65GB`  
  
この例では、3 TB のハード ドライブが 2 つあれば十分です。 ただし、ソース ディレクトリ `H:\Video` には 5 TB のデータがありますが、1 つのハード ドライブの容量は 3 TB しかないため、Microsoft Azure Import/Export ツールを実行する前に、`H:\Video` を `H:\Video1` と `H:\Video2` の 2 つの小さなディレクトリに分割する必要があります。 この手順により、次のソース ディレクトリが作成されます。  
  
|場所|サイズ|インポート先の仮想ディレクトリまたは BLOB|  
|--------------|----------|-------------------------------------------|  
|H:\Video1|2.5 TB|https://mystorageaccount.blob.core.windows.net/video|  
|H:\Video2|2.5 TB|https://mystorageaccount.blob.core.windows.net/video|  
|H:\Photo|30 GB|https://mystorageaccount.blob.core.windows.net/photo|  
|K:\Temp\FavoriteMovies.ISO|25 GB|https://mystorageaccount.blob.core.windows.net/favorite/FavoriteMovies.ISO|  
|\\\bigshare\john\music|10 GB|https://mystorageaccount.blob.core.windows.net/music|  
  
 `H:\Video` ディレクトリが&2; つのディレクトリに分割されても、これらのディレクトリはストレージ アカウント内の同じインポート先仮想ディレクトリを参照していることに注意してください。 これにより、すべてのビデオ ファイルがストレージ アカウント内の&1; つの `video` コンテナーに保持されます。  
  
 次に、上記のソース ディレクトリが&2; つのハード ドライブに均等に配置されます。  
  
||||  
|-|-|-|  
|ハード ドライブ|ソース ディレクトリ|合計サイズ|  
|最初のドライブ|H:\Video1|2.5 TB + 30 GB|  
||H:\Photo||  
|2 番目のドライブ|H:\Video2|2.5 TB + 35 GB|  
||K:\Temp\BlueRay.ISO||  
||\\\bigshare\john\music||  
  
さらに、すべてのファイルの次のメタデータを設定できます。  
  
-   **UploadMethod:** Windows Azure Import/Export Service  
  
-   **DataSetName:** SampleData  
  
-   **CreationDate:** 10/1/2013  
  
インポートするファイルのメタデータを設定するには、次の内容が含まれたテキスト ファイル `c:\WAImportExport\SampleMetadata.txt` を作成します。  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<Metadata>  
    <UploadMethod>Windows Azure Import/Export Service</UploadMethod>  
    <DataSetName>SampleData</DataSetName>  
    <CreationDate>10/1/2013</CreationDate>  
</Metadata>  
```
  
`FavoriteMovie.ISO` BLOB の一部のプロパティを設定することもできます。  
  
-   **Content-Type:** application/octet-stream  
  
-   **Content-MD5:** Q2hlY2sgSW50ZWdyaXR5IQ==  
  
-   **Cache-Control:** no-cache  
  
これらのプロパティを設定するには、テキスト ファイル `c:\WAImportExport\SampleProperties.txt` を作成します。  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<Properties>  
    <Content-Type>application/octet-stream</Content-Type>  
    <Content-MD5>Q2hlY2sgSW50ZWdyaXR5IQ==</Content-MD5>  
    <Cache-Control>no-cache</Cache-Control>  
</Properties>  
```
  
これで、Azure Import/Export ツールを実行して&2; つのハード ドライブを準備できます。 以下の点に注意してください。  
  
-   最初のドライブは、ドライブ X としてマウントされます。  
  
-   2 番目のドライブは、ドライブ Y としてマウントされます。  
  
-   ストレージ アカウント `mystorageaccount` のキーは `8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg==` です。  

## <a name="preparing-disk-for-import-when-data-is-pre-loaded"></a>データが事前に読み込まれている場合のインポート用ディスクの準備
 
 インポートするデータがディスク上に既に存在する場合は、/skipwrite フラグを使用します。 /t と /srcdir の値は、どちらもインポート用に準備するディスクを参照する必要があります。 ディスク上のすべてのデータを、ストレージ アカウントの同じインポート先仮想ディレクトリまたはルートに送信する必要があるわけではない場合は、各ディレクトリに対して同じコマンドを個別に実行します。その際に、/id には毎回同じ値を使用します。

>[!NOTE] 
>/format を指定すると、ディスク上のデータが消去されるので、これを指定しないでください。 データが既に暗号化されているかどうかに応じて、/encrypt または /bk を指定できます。 
>

```
    When data is already present on the disk for each drive run the following command.
    WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:Video1 /logdir:c:\logs /sk:8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg== /t:x /format /encrypt /srcdir:x:\Video1 /dstdir:video/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt /skipwrite
```

最初のドライブでは、Azure Import/Export ツールを&2; 回実行して、2 つのソース ディレクトリをコピーします。  
  
```
## First copy session for first drive  
WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:Video1 /logdir:c:\logs /sk:8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg== /t:x /format /encrypt /srcdir:H:\Video1 /dstdir:video/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt  
```

```  
## Second copy session for first drive  
WAImportExport.exe PrepImport /j:FirstDrive.jrn /id:Photo /srcdir:H:\Photo /dstdir:photo/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt
```
  
2 番目のドライブでは、Azure Import/Export ツールを&3; 回 (ソース ディレクトリごとに&1; 回と、スタンドアロンの Blu-Ray™ イメージ ファイルに対して&1; 回) 実行します。  
  
```
## First copy session  
WAImportExport.exe PrepImport /j:SecondDrive.jrn /id:Video2 /logdir:c:\logs /sk:8ImTigJhIwvL9VEIQKB/zbqcXbxrIHbBjLIfOt0tyR98TxtFvUM/7T0KVNR6KRkJrh26u5I8hTxTLM2O1aDVqg== /t:y /format /encrypt /srcdir:H:\Video2 /dstdir:video/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt  
```
  
```
## Second copy session  
WAImportExport.exe PrepImport /j:SecondDrive.jrn /id:Music /srcdir:\\bigshare\john\music /dstdir:music/ /MetadataFile:c:\WAImportExport\SampleMetadata.txt  
```  
  
```
## Third copy session  
WAImportExport.exe PrepImport /j:SecondDrive.jrn /id:BlueRayIso /srcfile:K:\Temp\BlueRay.ISO /dstblob:favorite/BlueRay.ISO /MetadataFile:c:\WAImportExport\SampleMetadata.txt /PropertyFile:c:\WAImportExport\SampleProperties.txt  
```
  
コピー セッションが完了したら、コピー用コンピューターから&2; つのドライブを切断し、適切な Microsoft Azure データ センターに送付できます。 [Microsoft Azure 管理ポータル](https://manage.windowsazure.com/)でインポート ジョブを作成するときに、`FirstDrive.jrn` と `SecondDrive.jrn` の&2; つのジャーナル ファイルをアップロードします。  
  
## <a name="see-also"></a>関連項目  
[インポート ジョブ用のハード ドライブを準備する](storage-import-export-tool-preparing-hard-drives-import-v1.md)   
[頻繁に使用するコマンドのクイック リファレンス](storage-import-export-tool-quick-reference-v1.md) 



<!--HONumber=Dec16_HO3-->


