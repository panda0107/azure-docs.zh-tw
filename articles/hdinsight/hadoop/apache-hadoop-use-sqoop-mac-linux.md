---
title: 採用 Apache Hadoop 的 Apache Sqoop - Azure HDInsight
description: 了解如何使用 Apache Sqoop 在 Apache Hadoop on HDInsight 與 Azure SQL Database 之間進行匯入和匯出。
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 11/28/2019
ms.openlocfilehash: 21bc903349876a76576fb742840e9899f9d94bcd
ms.sourcegitcommit: 6bb98654e97d213c549b23ebb161bda4468a1997
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/03/2019
ms.locfileid: "74769382"
---
# <a name="use-apache-sqoop-to-import-and-export-data-between-apache-hadoop-on-hdinsight-and-sql-database"></a>使用 Apache Sqoop 在 Apache Hadoop on HDInsight 與 SQL Database 之間匯入及匯出資料

[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

了解如何使用 Apache Sqoop，在 Azure HDInsight 中的 Apache Hadoop 叢集與 Azure SQL Database 或 Microsoft SQL Server 資料庫之間進行匯入和匯出。 本文件中的步驟直接從 Hadoop 叢集的前端節點使用 `sqoop` 命令。 您可以使用 SSH 連接至前端節點，並執行本文件中的命令。 本文是在[HDInsight 中搭配使用 Apache Sqoop 與 Hadoop](./hdinsight-use-sqoop.md)的接續。

## <a name="prerequisites"></a>必要條件

* 完成[設定測試環境](./hdinsight-use-sqoop.md#create-cluster-and-sql-database) [，從在 HDInsight 中搭配使用 Apache Sqoop 與 Hadoop](./hdinsight-use-sqoop.md)。

* SSH 用戶端。 如需詳細資訊，請參閱[使用 SSH 連線至 HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md)。

* 熟悉 Sqoop。 如需詳細資訊，請參閱[Sqoop 使用者指南](https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html)。

## <a name="set-up"></a>設定

1. 使用[ssh 命令](../hdinsight-hadoop-linux-use-ssh-unix.md)連接到您的叢集。 以您叢集的名稱取代 CLUSTERNAME，然後輸入命令，以編輯下面的命令：

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. 為了方便使用，請設定變數。 以相關的值取代 `PASSWORD`、`MYSQLSERVER`和 `MYDATABASE`，然後輸入下列命令：

    ```bash
    export password='PASSWORD'
    export sqlserver="MYSQLSERVER"
    export database="MYDATABASE"


    export serverConnect="jdbc:sqlserver://$sqlserver.database.windows.net:1433;user=sqluser;password=$password"
    export serverDbConnect="jdbc:sqlserver://$sqlserver.database.windows.net:1433;user=sqluser;password=$password;database=$database"
    ```

## <a name="sqoop-export"></a>Sqoop export

從 Hive 到 SQL Server。

1. 若要確認 Sqoop 可以看到您的 SQL Database，請在開啟的 SSH 連線中輸入下列命令。 此命令會傳回資料庫的清單。

    ```bash
    sqoop list-databases --connect $serverConnect
    ```

1. 輸入下列命令，以查看指定之資料庫的資料表清單：

    ```bash
    sqoop list-tables --connect $serverDbConnect
    ```

1. 若要將資料從 Hive `hivesampletable` 資料表匯出至 SQL Database 中的 `mobiledata` 資料表，請在開啟的 SSH 連線中輸入下列命令：

    ```bash
    sqoop export --connect $serverDbConnect \
    -table mobiledata \
    --hcatalog-table hivesampletable
    ```

1. 若要確認資料已匯出，請從 SSH 連線使用下列查詢來查看匯出的資料：

    ```bash
    sqoop eval --connect $serverDbConnect \
    --query "SELECT COUNT(*) from dbo.mobiledata WITH (NOLOCK)"


    sqoop eval --connect $serverDbConnect \
    --query "SELECT TOP(10) * from dbo.mobiledata WITH (NOLOCK)"
    ```

## <a name="sqoop-import"></a>Sqoop import

從 SQL Server 到 Azure 儲存體。

1. 在您的 open SSH 連線中輸入下列命令，以將資料從 SQL Database 中的 `mobiledata` 資料表匯入至 HDInsight 上的 `wasbs:///tutorials/usesqoop/importeddata` 目錄。 資料中的欄位是以定位字元分隔，行是以換行字元終止。

    ```bash
    sqoop import --connect $serverDbConnect \
    --table mobiledata \
    --target-dir 'wasb:///tutorials/usesqoop/importeddata' \
    --fields-terminated-by '\t' \
    --lines-terminated-by '\n' -m 1
    ```

1. 或者，您也可以指定 Hive 資料表：

    ```bash
    sqoop import --connect $serverDbConnect \
    --table mobiledata \
    --target-dir 'wasb:///tutorials/usesqoop/importeddata2' \
    --fields-terminated-by '\t' \
    --lines-terminated-by '\n' \
    --create-hive-table \
    --hive-table mobiledata_imported2 \
    --hive-import -m 1
    ```

1. 匯入完成後，請在開啟的 SSH 連線中輸入下列命令，以列出新目錄中的資料：

    ```bash
    hadoop fs -tail /tutorials/usesqoop/importeddata/part-m-00000
    ```

1. 使用[beeline](./apache-hadoop-use-hive-beeline.md)來確認已在 Hive 中建立資料表。

    1. 連接

        ```bash
        beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http'
        ```

    1. 一次執行一個以下的查詢，並查看輸出：

        ```hql
        show tables;
        describe mobiledata_imported2;
        SELECT COUNT(*) FROM mobiledata_imported2;
        SELECT * FROM mobiledata_imported2 LIMIT 10;
        ```

    1. 結束具有 `!exit`的 beeline。

## <a name="limitations"></a>限制

* 大量匯出-使用以 Linux 為基礎的 HDInsight，用來將資料匯出至 Microsoft SQL Server 或 Azure SQL Database 的 Sqoop 連接器不支援大量插入。

* 批次處理 - 使用 Linux 型 HDInsight，執行插入時若使用 `-batch` 參數，Sqoop 將會執行多個插入，而不是批次處理插入作業。

## <a name="important-considerations"></a>重要考量︰

* HDInsight 與 SQL Server 必須位於相同的 Azure 虛擬網路。

    如需範例，請參閱[將 HDInsight 連線至內部部署網路](./../connect-on-premises-network.md)文件。

    如需使用 HDInsight 搭配 Azure 虛擬網路的詳細資訊，請參閱[使用 Azure 虛擬網路擴充 HDInsight](../hdinsight-plan-virtual-network-deployment.md)文件。 如需 Azure 虛擬網路的詳細資訊，請參閱[虛擬網路概觀](../../virtual-network/virtual-networks-overview.md)文件。

* SQL Server 也必須設定為允許 SQL 驗證。 如需詳細資訊，請參閱[選擇驗證模式](https://msdn.microsoft.com/ms144284.aspx)文件。

* 您可能必須設定 SQL Server 以接受遠端連線。 如需詳細資訊，請參閱[如何疑難排解 SQL Server Database Engine 連線](https://social.technet.microsoft.com/wiki/contents/articles/2102.how-to-troubleshoot-connecting-to-the-sql-server-database-engine.aspx)文件 (英文)。

## <a name="next-steps"></a>後續步驟

現在您已瞭解如何使用 Sqoop。 若要深入了解，請參閱：

* 搭配[HDInsight 使用 Apache Oozie](../hdinsight-use-oozie-linux-mac.md)：在 Oozie 工作流程中使用 Sqoop 動作。
* [使用 HDInsight 分析航班延誤資料](../interactive-query/interactive-query-tutorial-analyze-flight-data.md)：使用互動式查詢來分析航班延誤資料，然後使用 Sqoop 將資料匯出至 Azure SQL 資料庫。
* [將資料上傳至 HDInsight](../hdinsight-upload-data.md)：尋找可將資料上傳至 HDInsight/Azure Blob 儲存體的其他方法。
