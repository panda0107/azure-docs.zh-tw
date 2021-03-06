---
title: JAVA.lang.noclassdeffounderror-使用 Azure HDInsight 中的 Apache Kafka 資料進行 Apache Spark
description: 從 Apache Kafka 叢集讀取資料的 Apache Spark 串流作業失敗，並出現 JAVA.lang.noclassdeffounderror，Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/29/2019
ms.openlocfilehash: 4659274110add96613ca88560edfb459b20a99cb
ms.sourcegitcommit: 8e9a6972196c5a752e9a0d021b715ca3b20a928f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2020
ms.locfileid: "75894355"
---
# <a name="apache-spark-streaming-job-that-reads-apache-kafka-data-fails-with-noclassdeffounderror-in-hdinsight"></a>在 HDInsight 中使用 JAVA.lang.noclassdeffounderror 讀取 Apache Kafka 資料的 Apache Spark 串流作業失敗

本文說明在 Azure HDInsight 叢集中使用 Apache Spark 元件時，疑難排解步驟和問題的可能解決方法。

## <a name="issue"></a>問題

Apache Spark 叢集會執行可從 Apache Kafka 叢集讀取資料的 Spark 串流作業。 如果 Kafka 資料流程壓縮已開啟，Spark 串流作業就會失敗。 在此情況下，Spark 串流 Yarn 應用程式 application_1525986016285_0193 失敗，因為發生錯誤：

```
18/05/17 20:01:33 WARN YarnAllocator: Container marked as failed: container_e25_1525986016285_0193_01_000032 on host: wn87-Scaled.2ajnsmlgqdsutaqydyzfzii3le.cx.internal.cloudapp.net. Exit status: 50. Diagnostics: Exception from container-launch.
Container id: container_e25_1525986016285_0193_01_000032
Exit code: 50
Stack trace: ExitCodeException exitCode=50: 
 at org.apache.hadoop.util.Shell.runCommand(Shell.java:944)
```

## <a name="cause"></a>原因

此錯誤可能是因為指定的 `spark-streaming-kafka` jar 檔案版本與您正在執行的 Kafka 叢集版本不同所造成。

例如，如果您執行的是 Kafka 叢集版本0.10.1，下列命令將會導致錯誤：

```
spark-submit \
--packages org.apache.spark:spark-streaming-kafka-0-8_2.11:2.2.0
--conf spark.executor.instances=16 \
...
~/Kafka_Spark_SQL.py <bootstrap server details>
```

## <a name="resolution"></a>解析度

使用 [Spark-提交] 命令與 [`–packages`] 選項，並確定 Spark 串流 kafka jar 檔案的版本與您正在執行的 Kafka 叢集版本相同。

## <a name="next-steps"></a>後續步驟

如果您沒有看到您的問題，或無法解決您的問題，請瀏覽下列其中一個管道以取得更多支援：

* 透過[Azure 社區支援](https://azure.microsoft.com/support/community/)取得 azure 專家的解答。

* 連接[@AzureSupport](https://twitter.com/azuresupport) -官方 Microsoft Azure 帳戶，藉由將 Azure 社區連接至適當的資源，來改善客戶體驗：解答、支援和專家。

* 如果您需要更多協助，您可以從[Azure 入口網站](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/)提交支援要求。 從功能表列選取 [**支援**]，或開啟 [說明 **+ 支援**] 中樞。 如需詳細資訊，請參閱[如何建立 Azure 支援要求](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request)。 您的 Microsoft Azure 訂用帳戶包含訂用帳戶管理和帳單支援的存取權，而技術支援則透過其中一項[Azure 支援方案](https://azure.microsoft.com/support/plans/)提供。
