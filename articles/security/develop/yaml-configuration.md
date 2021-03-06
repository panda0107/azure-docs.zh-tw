---
title: Microsoft Azure 安全性程式碼分析工作自訂指南
description: 本文說明在 Microsoft 安全性程式碼分析延伸模組中自訂所有工作的清單 YAML 設定選項
author: vharindra
manager: sukhans
ms.author: terrylan
ms.date: 11/29/2019
ms.topic: article
ms.service: security
services: azure
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.openlocfilehash: 82802ceae8f9fdd24d1f5642d80d90ebfd8b92a3
ms.sourcegitcommit: f4f626d6e92174086c530ed9bf3ccbe058639081
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/25/2019
ms.locfileid: "75460663"
---
# <a name="yaml-configuration-options-to-customize-the-build-tasks"></a>YAML 設定選項以自訂群組建工作

本文列出每個組建工作中可用的所有 YAML 設定選項。 本文會從安全性程式碼分析工具的工作開始。 結尾為後置處理工作。

## <a name="anti-malware-scanner-task"></a>反惡意程式碼掃描程式工作

| **InputType**      | **型別**     | **可行**            | **必要** | **預設值**             | **選項（適用于挑選清單）**                                   | **說明**                                                                                                                                                                                                                                                                                                                            |
|------------|---------------|-----------------------|----------|---------------------------|----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| InputType | 清單 | always | 是 | 基本 | 基本、自訂 | 
| ScanType | 清單 | InputType = 基本 | 是 | CustomScan | CustomScan, FullSystemScan, QuickScan, YourConfiguredScan | 用於反惡意程式碼掃描的掃描類型。
| FileDirPath | filePath | ScanType = CustomScan | 是 | $ （Build.stagingdirectory） |  | 表示要掃描的檔案或目錄。
| DisableRemediation | boolean | ScanType = CustomScan | 否 | true |  | 核取時：1）會忽略檔案排除專案。 2）掃描封存檔案。 3）在偵測之後，不會套用動作。 4）偵測之後，不會寫入事件記錄檔專案。 5）自訂掃描的偵測不會顯示在使用者介面中。 6）主控台輸出將會顯示自訂掃描的偵測清單。
| BootSectorScan | boolean | ScanType = CustomScan | 否 | false |  | 若已核取，則會啟用開機磁區掃描。
| 引數 | string | InputType = 自訂 | 是 | -Scan-ScanType 3-DisableRemediation-File $ （Build.stagingdirectory） |  | 命令列引數，其中-File 的引數是絕對路徑，或是在您的組建代理程式上預先定義之 $ （Build.stagingdirectory）的相對路徑。 注意：如果您未提供-File 的引數做為最後一個引數，則會預設為 $ （Build.stagingdirectory）。 您也可以提供 MpCmdRun 工具所允許的引數。<br/><br/>如需此工具的命令列引數的詳細資訊，請輸入<strong>-h</strong>或<strong>-？</strong> 在 [引數] 欄位中，執行組建工作。
| EnableServices | boolean | always | 是 | false |  | 若已核取，則會嘗試針對 Windows Update 啟用必要的服務，以防其已停用。<br/>**注意**：請確定群組原則不會停用服務，且執行此組建的帳戶具有系統管理員許可權。
| SupportLogOnError | boolean | always | 是 | false |  | 若已核取，則會在發生錯誤時收集支援檔案以進行診斷。 這可能需要幾分鐘的時間。<br/>**注意**：請確認執行此組建的帳戶具有系統管理員許可權。
| TreatSignatureUpdateFailureAs | 清單 | always | 是 | 警告 | 錯誤，標準，警告 | 當簽章無法在執行時間更新時，所使用的記錄層級。 當設定為 [**錯誤**] 時，更新簽章失敗將會導致組建工作失敗。 請注意，在裝載的組建代理程式上，簽章更新會失敗，即使簽章可能相對最新（少於3小時）。
| SignatureFreshness | 清單 | always | 是 | UpToDate | OneDay、ThreeDays、TwoDays、UpToDate | 反惡意程式碼簽章允許的最長存留期。 如果簽章無法更新，而且比此值還舊，則組建工作會根據 [驗證簽章**存留期**] 欄位中選取的值來表現。 注意：如果您選擇 [**最**新]，則簽章最多可以有3小時的舊版本。
| TreatStaleSignatureAs | 清單 | always | 是 | 錯誤 | 錯誤，標準，警告 | 當簽章存留期比選取的**反惡意**代碼簽章存留期間還舊時，所使用的記錄層級。 過期的簽章可能會被視為**警告**或**資訊**，以繼續進行反惡意程式碼掃描，但不建議這麼做。

## <a name="binskim-task"></a>BinSkim 工作

| **InputType**      | **型別**     | **可行**            | **必要** | **預設值**             | **選項（適用于挑選清單）**                                   | **說明**                                                                                                                                                                                                                                                                                                                            |
|------------|---------------|-----------------------|----------|---------------------------|----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| InputType | 清單 | always | 是 | 基本 | 基本、命令列 | 
| 引數 | string | InputType = 命令列 | 是 |  |  | 要執行的標準 Binskim 命令列引數。 輸出路徑將會被移除並被取代。<br>如需此工具的命令列引數的詳細資訊，請在 [引數] 欄位中輸入**help** ，然後執行組建工作。
| 函式 | 清單 | InputType = 基本 | 是 | 分析 | 分析、傾印、exportConfig、exportRules | 
| AnalyzeTarget | filePath | InputType = 基本 & & 函數 = 分析 | 是 | $ （ArtifactStagingDirectory）\*.dll;<br>$ （ArtifactStagingDirectory）\*.exe |  | 檔案、目錄或篩選模式的一或多個規範，可解析為一或多個要分析的二進位檔。 （'; ' 分隔清單）
| AnalyzeSymPath | string | InputType = 基本 & & 函數 = 分析 | 否 |  |  | 目標的符號檔路徑。
| AnalyzeConfigPath | string | InputType = 基本 & & 函數 = 分析 | 否 | 預設 |  | 將用來設定分析的原則檔案路徑。 傳遞 ' default ' 的值以使用內建設定。
| AnalyzePluginPath | string | InputType = 基本 & & 函數 = 分析 | 否 |  |  | 將針對分析集中的所有目標叫用之外掛程式的路徑。
| AnalyzeRecurse | boolean | InputType = 基本 & & 函數 = 分析 | 否 | true |  | 在評估檔案規範引數時遞迴至子目錄。
| AnalyzeVerbose | boolean | InputType = 基本 & & 函數 = 分析 | 否 | false |  | 發出詳細資訊輸出。 產生的完整報告是為了提供合規性案例的適當辨識項而設計的。
| AnalyzeHashes | boolean | InputType = 基本 & & 函數 = 分析 | 否 | false |  | 發出 SARIF 報表時，輸出 SHA-256 分析目標的雜湊。
| AnalyzeStatistics | boolean | InputType = 基本 & & 函數 = 分析 | 否 | false |  | 產生分析會話的計時和其他統計資料。
| AnalyzeEnvironment | boolean | InputType = 基本 & & 函數 = 分析 | 否 | false |  | 執行至輸出檔的記錄機器環境詳細資料。 警告：此選項會將可能的機密資訊（例如所有環境變數值）記錄到任何發出的記錄檔。
| ExportRulesOutputType | 清單 | InputType = 基本 & & 函數 = exportRules | 否 | SARIF | SARIF、SonarQube | 要輸出的規則描述元檔案類型。 這會包含在「發行安全性分析記錄」組建工作所發行的 BinSkim logs 資料夾中。
| DumpTarget | filePath | InputType = 基本 & & 函數 = 傾印 | 是 | $ （ArtifactStagingDirectory） |  | 檔案、目錄或篩選模式的一或多個規範，可解析為一或多個要分析的二進位檔。 （'; ' 分隔清單）
| DumpRecurse | boolean | InputType = 基本 & & 函數 = 傾印 | 否 | true |  | 在評估檔案規範引數時遞迴至子目錄。
| DumpVerbose | boolean | InputType = 基本 & & 函數 = 傾印 | 否 | true |  | 發出詳細資訊輸出。 產生的完整報告是為了提供合規性案例的適當辨識項而設計的。
| Toolversion 設 | 清單 | always | 否 | 最新 | 1.5.0、最新、LatestPreRelease | 要執行的工具版本。

## <a name="credential-scanner-task"></a>認證掃描器工作

| **InputType**      | **型別**     | **可行**            | **必要** | **預設值**             | **選項（適用于挑選清單）**                                   | **說明**                                                                                                                                                                                                                                                                                                                            |
|------------|---------------|-----------------------|----------|---------------------------|----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| OutputFormat | 清單 | always | 否 | pre | csv、pre、tsv | 認證掃描器結果檔案的輸出格式。
| Toolversion 設 | 清單 | always | 否 | 最新 | 1.27.7、最新、LatestPreRelease | 要執行的工具版本。
| scanFolder | filePath | always | 否 | $ （SourcesDirectory） |  | 存放庫中要掃描認證的資料夾。
| searchersFileType | 清單 | always | 否 | 預設 | Custom、Default、DefaultAndCustom | 用來尋找用於掃描之 searchers 檔案的選項。
| searchersFile | filePath | searchersFileType = = Custom 或 searchersFileType = = DefaultAndCustom | 否 |  |  | 要執行之檢查的認證掃描程式 searchers 設定檔案。 藉由提供認證掃描器 searchers 檔的路徑清單（以逗號分隔），可以包含和使用多個值。
| suppressionsFile | filePath | always | 否 |  |  | 要用來隱藏輸出記錄檔中問題的認證掃描器禁止檔案。
| suppressAsError | boolean | always | 否 | false |  | 隱藏的相符專案會輸出到輸出檔 [-O]-相符專案。[-f]，而不是預設隱藏的輸出檔 [-O]-已隱藏。[-f]。 （預設為 ' False '）
| verboseOutput | boolean | always | 否 | false |  | 輸出詳細資訊。
| batchSize | string | always | 否 |  |  | 用來平行執行認證掃描器的平行線程數目。 （預設為20）<br/>值必須在1-2147483647 的範圍內。
| RegExMatchTimeoutInSeconds | string | always | 否 |  |  | 在放棄檢查之前，嘗試搜尋搜尋所花費的時間量（以秒為單位）。<br/>將 ``-Co RegexMatchTimeoutInSeconds=<Value>`` 加入至命令列。
| fileScanReadBufferSize | string | always | 否 |  |  | 讀取內容時的緩衝區大小（以位元組為單位）。 （預設為524288）<br/>將 ``-Co FileScanReadBufferSize=<Value>`` 加入至命令列。
| maxFileScanReadBytes | string | always | 否 |  |  | 在內容分析期間，從指定檔案讀取的最大位元組數。 （預設為104857600）<br/>將 ``-Co MaxFileScanReadBytes=<Value>`` 加入至命令列。

## <a name="microsoft-security-risk-detection-task"></a>Microsoft 安全性風險偵測工作

| **InputType**      | **型別**     | **可行**            | **必要** | **預設值**             | **選項（適用于挑選清單）**                                   | **說明**                                                                                                                                                                                                                                                                           |
|------------|---------------|-----------------------|----------|---------------------------|----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 為 serviceendpointname | connectedService：泛型 | always | 是 |  |  | 您的 VSTS 專案上預先設定的服務端點（泛型型別）名稱，可儲存 MSRD 實例 URL （您已上架至）和 REST API 存取權杖（從您的 [帳戶設定] 頁面產生並允許完整存取您的帳戶）。
| AccountId | string | always | 是 |  |  | 識別帳戶的 GUID。 它可以從帳戶 URL 抓取。
| BinariesURL | string | always | 是 |  |  | 模糊化機器用來下載二進位檔的 Url 清單（以分號分隔）。 確定 Url 可公開使用。
| SeedsURL | string | always | 否 |  |  | 模糊化機器用來下載種子的 Url 清單（以分號分隔）。 請確定 Url 可公開使用。
| OSPlatformType | 清單 | always | 是 | Windows | Linux、Windows | 要在其上執行模糊作業的電腦 OS 平臺類型。
| WindowsEdition | string | OSPlatformType = Windows | 是 | Server 2008 R2 |  | 要在其上執行模糊化作業的電腦作業系統版本。
| LinuxEdition | string | OSPlatformType = Linux | 是 | Redhat |  | 要在其上執行模糊化作業的電腦作業系統版本。
| PreSubmissionCommand | string | always | 否 |  |  | 需要在測試電腦上執行的腳本，以便在進行模糊化作業提交之前，先安裝測試目的程式及其相依性。
| SeedDirectory | string | always | 是 |  |  | 模糊電腦上包含種子的目錄路徑。 如需詳細資訊，請參閱[Seed File 目錄](https://docs.microsoft.com/security-risk-detection/how-to/submit-windows-fuzzing-job/03-choose-seed-files#seed-file-directory)。
| SeedExtension | string | always | 是 |  |  | 種子的副檔名。
| TestDriverExecutable | string | always | 是 |  |  | 模糊電腦上目標可執行檔的路徑。 如需詳細資訊，請參閱[EPE 的完整路徑](https://docs.microsoft.com/security-risk-detection/how-to/submit-windows-fuzzing-job/02-choose-exe#full-path-to-the-epe)。
| TestDriverExeType | 清單 | always | 是 | x86 | amd64、x86 | 目標可執行檔架構。
| TestDriverParameters | string | always | 是 | "% testfile.txt%" |  | 傳遞至測試目標可執行檔的命令列引數。 請注意， **"% testfile.txt%"** 符號（包括雙引號）將會自動取代為測試驅動程式應剖析之目標檔案的完整路徑，而且是必要的。 如需詳細資訊，請參閱[命令列引數](https://docs.microsoft.com/security-risk-detection/how-to/submit-windows-fuzzing-job/02-choose-exe#command-line-arguments)。
| ClosesItself | boolean | always | 是 | true |  | 檢查測試驅動程式是否會在完成時自行終止（例如，測試驅動程式會剖析輸入檔並立即結束）;如果需要強制關閉測試驅動程式（例如，測試驅動程式是在剖析輸入之後維持開啟狀態的 GUI 應用程式），請取消核取。 如需詳細資訊，請參閱[自我終止](https://docs.microsoft.com/security-risk-detection/how-to/submit-windows-fuzzing-job/05-scope-exe-lifetime#self-termination)。
| MaxDurationInSeconds | string | always | 是 | 5 |  | 測試驅動程式的最長持續時間（秒）。 提供目的程式剖析輸入檔所需的最長合理預期時間的估計。 這種估計越精確，模糊執行的效率就愈高。 如需詳細資訊，請參閱[預期的最大執行持續時間](https://docs.microsoft.com/security-risk-detection/how-to/submit-windows-fuzzing-job/05-scope-exe-lifetime#maximum-expected-execution-duration)。
| CanRunRepeat | boolean | always | 是 | true |  | 檢查測試驅動程式是否可以重複執行，而不需視保存/共用全域狀態而定。 如需詳細資訊，請參閱[從頭開始執行](https://docs.microsoft.com/security-risk-detection/how-to/submit-windows-fuzzing-job/04-describe-exe-characteristics#runs-from-scratch)。
| CanTestDriverBeRenamed | boolean | always | 是 | false |  | 檢查是否可以重新命名測試驅動程式可執行檔，而且仍然可以正常運作。 如需詳細資訊，請參閱[可重新命名和](https://docs.microsoft.com/security-risk-detection/how-to/submit-windows-fuzzing-job/04-describe-exe-characteristics#can-be-renamed-and-parallelized)平行處理。
| SingleOsProcess | boolean | always | 是 | false |  | 檢查測試驅動程式是否在單一作業系統進程下執行;如果測試驅動程式會產生額外的進程，請取消核取。 如需詳細資訊，請參閱[單一進程](https://docs.microsoft.com/security-risk-detection/how-to/submit-windows-fuzzing-job/04-describe-exe-characteristics#single-process)。

## <a name="roslyn-analyzers-task"></a>Roslyn 分析器工作

| **InputType**      | **型別**     | **可行**            | **必要** | **預設值**             | **選項（適用于挑選清單）**                                   | **說明**                                                                                                                                                                                                                                                                                                                   |
|------------|---------------|-----------------------|----------|---------------------------|----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| userProvideBuildInfo | 清單 | always | 是 | 自動 | auto、msBuildInfo | 提供 MSBuild 版本、MSBuild 架構和組建命令列以進行 Roslyn 分析的使用者選項。 如果選取了 [**自動**]，此工作將會在相同的管線中，從先前的**MSBuild**、 **VSBuild**和/或 **.net Core** （適用于組建）工作中取出組建資訊。
| msBuildVersion | 清單 | userProvideBuildInfo = = msBuildInfo | 是 | 16.0 | 15.0、16。0 | MSBuild 版本。
| msBuildArchitecture | 清單 | userProvideBuildInfo = = msBuildInfo | 是 | x86 | DotNetCore、x64、x86 | MSBuild 架構。 注意：如果組建命令列呼叫**dotnet**，請選擇 [ **Via .Net Core** ] 選項。
| msBuildCommandline | string | userProvideBuildInfo = = msBuildInfo | 是 |  |  | 用來編譯方案或專案的完整組建命令列。<br/><br/>注意：命令列的開頭應為**msbuild.exe**或**dotnet**的完整路徑。<br/>此命令將使用 $ （SourcesDirectory）做為工作目錄來執行。
| rulesetName | 清單 | always | 否 | 建議 | 自訂、無、建議、必要 | 要使用的命名規則集。<br/><br/>如果選擇 `Ruleset Configured In Your Visual Studio Project File(s)`，將會使用在您的 VS 專案檔中預先設定的規則集。 如果選擇 `Custom`，則可以設定自訂規則集路徑選項。
| rulesetVersion | 清單 | rulesetName = = Required 或 rulesetName = = 建議使用 | 否 | 最新 | 8.0、8.1、8.2、最新、LatestPreRelease | 所選 SDL 規則集的版本。
| customRuleset | string | rulesetName = 自訂 | 否 |  |  | 可存取的規則集路徑，可供使用。 相對路徑會正規化為來源存放庫的根目錄（`$(Build.SourcesDirectory)`）。<br/><br/>如果規則集指定 `Rules`，且 `Actions` 設定為 `Error`，組建工作將會失敗。 若要使用執行此作業的規則集，請檢查組建工作的 `Control Options`中 `Continue on error`。
| microsoftAnalyzersVersion | 清單 | always | 否 | 最新 | 2.9.3、2.9.4、2.9.6、最新、LatestPreRelease | 要執行的[CodeAnalysis FxCopAnalyzers](https://www.nuget.org/packages/Microsoft.CodeAnalysis.FxCopAnalyzers)套件版本。
| suppressionFileForCompilerWarnings | filePath | always | 否 |  |  | 要隱藏C#和 VB 編譯器警告的禁止檔案。<br/><br/>一個純文字檔案，其中每個警告識別碼分別列出一行。<br/>針對編譯器警告，只指定警告識別碼的數位部分。 例如，1018會隱藏 CS1018，而 CA1501 會隱藏 CA1501。<br/><br/>相對檔案路徑將會附加至來源存放庫的根目錄（`$(Build.SourcesDirectory)`）。

## <a name="tslint-task"></a>TSLint 工作

| **InputType**      | **型別**     | **可行**            | **必要** | **預設值**             | **選項（適用于挑選清單）**                                   | **說明**                                                                                                                                                                                                                                                                                                                            |
|------------|---------------|-----------------------|----------|---------------------------|----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| RuleLibrary | 清單 | always | 是 | tslint | custom、microsoft、tslint | 所有結果都包含所選 TSLint 版本隨附的規則（**僅限基底**）。<br/><br/>**僅限基底-** 只有 TSLint 隨附的規則。<br/><br/>**包含 Microsoft 規則-** 下載[tslint-microsoft contrib](https://github.com/Microsoft/tslint-microsoft-contrib)並包含其規則，可在 tslint 執行中使用。 選擇此選項會隱藏 [`Type Checking`] 核取方塊，因為 Microsoft 的規則需要它，而且會自動使用。 它也會 unhides [`Microsoft Contribution Version`] 欄位，允許選取[npm](https://www.npmjs.com/package/tslint-microsoft-contrib)的 `tslint-microsoft-contrib` 版本。<br/><br/>**包含自訂規則-** Unhides [`Rules Directory`] 欄位，它會接受 TSLint 規則目錄的可存取路徑，以供 TSLint 執行使用。<br/><br/>**注意：** 預設值已變更為 tslint，因為許多使用者在設定 Microsoft 規則集時遇到問題。 如需特定版本設定，請參閱[GitHub 上的 tslint-microsoft contrib](https://github.com/microsoft/tslint-microsoft-contrib)。
| RulesDirectory | string | RuleLibrary = = custom | 是 |  |  | 可存取的目錄，其中包含可在 TSLint 執行中使用的其他 TSLint 規則。
| Ruleset | 清單 | RuleLibrary！ = microsoft | 是 | tsrecommended | custom、tslatest、tsrecommended | 定義要針對 TypeScript 檔案執行的規則。<br/><br/>**[tslint：最新](https://github.com/palantir/tslint/blob/master/src/configs/latest.ts) -** 擴充 `tslint:recommended` 並持續更新，以包含每個 TSLint 版本中最新規則的設定。 使用此設定可能會在次要版本之間引進重大變更，因為新的規則會啟用，而導致您的程式碼發生不起毛的失敗。 當 TSLint 達到主要版本增加時，`tslint:recommended` 會更新為與 `tslint:latest`相同。<br/><br/>**[tslint：建議](https://github.com/palantir/tslint/blob/master/src/configs/recommended.ts)的 -** 一組穩定、有點固定的規則，TSLint 鼓勵一般 TypeScript 程式設計。 此設定會遵循 `semver`，因此*不*會在次要或修補程式版本之間有重大變更。
| RulesetMicrosoft | 清單 | RuleLibrary = = microsoft | 是 | mssdlrequired | custom、msrecommended、mssdlrecommended、mssdlrequired、tslatest、tsrecommended | 定義要針對 TypeScript 檔案執行的規則。<br/><br/>**[microsoft： sdl-必要](https://github.com/Microsoft/tslint-microsoft-contrib/wiki/TSLint-and-the-Microsoft-Security-Development-Lifecycle) -** 執行 tslint 所提供的所有可用檢查，以及符合*必要*[安全性開發生命週期（sdl）](https://www.microsoft.com/sdl/)原則的 tslint-microsoft contrib 規則。<br/><br/>**[microsoft： sdl 建議](https://github.com/Microsoft/tslint-microsoft-contrib/wiki/TSLint-and-the-Microsoft-Security-Development-Lifecycle)的 -** 執行 tslint 所提供的所有可用檢查，以及符合*必要和建議*[安全性開發週期（sdl）](https://www.microsoft.com/sdl/)原則的 tslint-microsoft contrib 規則。<br/><br/>**[microsoft：建議](https://github.com/Microsoft/tslint-microsoft-contrib/blob/master/recommended_ruleset.js)的 -** Tslint 的建立者所建議的所有檢查-microsoft contrib 規則。 這包括安全性和非安全性檢查。<br/><br/>**[tslint：最新](https://github.com/palantir/tslint/blob/master/src/configs/latest.ts) -** 擴充 `tslint:recommended` 並持續更新，以包含每個 TSLint 版本中最新規則的設定。 使用此設定可能會在次要版本之間引進重大變更，因為新的規則會啟用，而導致您的程式碼發生不起毛的失敗。 當 TSLint 達到主要版本增加時，`tslint:recommended` 會更新為與 `tslint:latest`相同。<br/><br/>**[tslint：建議](https://github.com/palantir/tslint/blob/master/src/configs/recommended.ts)的 -** 一組穩定、有點固定的規則，TSLint 鼓勵一般 TypeScript 程式設計。 此設定會遵循 `semver`，因此*不*會在次要或修補程式版本之間有重大變更。
| RulesetFile | string | 規則集 = = custom 或 RulesetMicrosoft = = custom | 是 |  |  | 指定要執行哪些規則的[設定檔](https://palantir.github.io/tslint/usage/cli/)。<br/><br/>設定的路徑將會加入做為[自訂規則](https://palantir.github.io/tslint/develop/custom-rules/)的路徑。
| FileSelectionType | 清單 | always | 是 | fileGlob | fileGlob、projectFile | 
| 檔案 | string | FileSelectionType = = fileGlob | 是 | **\*. ts |  | 檔案[glob](https://www.npmjs.com/package/glob) ，可決定要處理的檔案。 相對於 `Build.SourcesDirectory` 值的路徑。<br/><br/>Microsoft 的投稿文件庫需要使用專案檔。 如果您使用 Microsoft 的投稿程式庫搭配 [`File Glob Pattern`] 選項，系統就會為您產生專案檔案。
| ECMAScriptVersion | 清單 | FileSelectionType = = fileGlob & & RuleLibrary = = microsoft | 是 | 以及 ES3 | ES2015、ES2016 功能、ES2017、以及 ES3、ES5、ES6、ESNext | 使用 TypeScript 編譯器設定的 ECMAScript 目標版本。 使用專案檔時，這是 TypeScript tsconfig 檔案的 [compilerOptions] 欄位。
| 隨附此逐步解說的專案 | string | FileSelectionType = = projectFile | 是 |  |  | [Tsconfig](http://www.typescriptlang.org/docs/handbook/tsconfig-json.html)檔案的路徑，該檔案會指定要在其上執行 TSLint 的 TypeScript 檔案。 相對於 `Build.SourcesDirectory` 值的路徑。
| TypeCheck | boolean | RuleLibrary！ = microsoft & & FileSelectionType = = projectFile | 否 | true |  | 執行 linting 規則時啟用類型檢查程式。
| ExcludeFiles | string | always | 否 |  |  | [Glob](https://www.npmjs.com/package/glob) ，表示要從 linting 排除的檔案。 相對於 `Build.SourcesDirectory` 值的路徑。 可以指定多個值，並以分號分隔。
| OutputFormat | 清單 | always | 是 | json | checkstyle、codeFrame、filesList、json、msbuild、pmd、prose、時尚、verbose、vso | 要用來產生輸出的[格式](https://palantir.github.io/tslint/formatters/)器。 請注意，JSON 格式與 Post 分析相容。
| NodeMemory | string | always | 否 |  |  | 要配置給節點以執行 TSLint 的明確記憶體數量（以 Mb 為單位）。 範例：8000<br/><br/>對應至節點的 `--max_old_space=<value>` CLI 選項，也就是 `v8 option`。
| Toolversion 設 | 清單 | RuleLibrary！ = microsoft | 是 | latest | 4.0.0、4.0.1、4.0.2、4.1.0、4.1.1、4.2.0、4.3.0、4.3.1、4.4.0、4.4.1、4.4.2、4.5.0、4.5.1、5.0.0、5.1.0、5.2.0、5.3.0、5.3.2、5.4.0、5.4.1 版、5.4.2、5.4.3、5.5.0、最新 | 要下載並執行的 TSLint[版本](https://github.com/palantir/tslint/releases)。
| TypeScriptVersion | 清單 | always | 是 | latest | 0.8.0 版、0.8.1 版、0.8.2、0.8.3、0.9.0、0.9.1、0.9.5、0.9.7 版、1.0.0、1.0.1、1.3.0、1.4.1、1.5.3、1.6.2、1.7.3、1.7.5、1.8.0、1.8.10、1.8.2、1.8.5 版、1.8.6、1.8.7、1.8.9、1.9.0、2.0.0、2.0.10 版、2.0.2、2.0.3、2.0.6、2.0.7、2.0.8、2.0.9、2.1.1、2.1.4、2.1.5、2.1.6、2.2.0、2.2.1、custom、最新 | 要下載並使用的[typescript](https://www.npmjs.com/package/typescript)版本。<br/>**注意：** 這必須與用來編譯器代碼的 TypeScript 版本相同。
| TypeScriptVersionCustom | string | TypeScriptVersion = = custom | 是 | latest |  | 要下載並使用的[typescript](https://www.npmjs.com/package/typescript)版本。<br/>**注意：** 這必須與用來編譯器代碼的 TypeScript 版本相同。
| MicrosoftContribVersion | 清單 | RuleLibrary = = microsoft |  | latest | 4.0.0、4.0.1、5.0.0、5.0.1、最新 | 要下載並使用的[tslint 版本-microsoft contrib](https://www.npmjs.com/package/tslint-microsoft-contrib) （SDL 規則）。</br>**注意：** 將會選擇與針對 tslint 所選版本相容的[tslint](https://www.npmjs.com/package/tslint)版本-microsoft contrib。 Tslint 的更新-microsoft contrib 將由此組建工作進行閘道，直到有一段測試可以進行。

## <a name="publish-security-analysis-logs-task"></a>發行安全性分析記錄工作

| **InputType**      | **型別**     | **可行**            | **必要** | **預設值**             | **選項（適用于挑選清單）**                                   | **說明**                                                                                                                                                                                                                                                                                                                            |
|------------|---------------|-----------------------|----------|---------------------------|----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ArtifactName | string | always | 是 | CodeAnalysisLogs |  | 要建立之成品的名稱。
| ArtifactType | 清單 | always | 是 | 容器 | 容器、FilePath | 要建立之成品的型別。
| TargetPath | string | ArtifactType = FilePath | 否 | \\my\share\$（DefinitionName）<br>\$（BuildNumber） |  | 要將檔案複製到其中的檔案共用
| AllTools | boolean | always | 是 | true |  | 發行所有安全開發工具組建工作所產生的結果。
| AntiMalware | boolean | AllTools = false | 是 | true |  | 發行反惡意程式碼組建工作所產生的結果。
| BinSkim | boolean | AllTools = false | 是 | true |  | 發行 BinSkim build 工作所產生的結果。
| CredScan | boolean | AllTools = false | 是 | true |  | 發佈認證掃描器組建工作所產生的結果。
| MSRD | boolean | AllTools = false | 是 | true |  | 發行 MSRD build 工作所啟動之 MSRD 作業的作業資訊和作業 url。 MSRD 作業長時間執行，並提供個別的報表。
| RoslynAnalyzers | boolean | AllTools = false | 是 | false |  | 發行由 Roslyn 分析器組建工作所產生的結果。
| TSLint | boolean | AllTools = false | 是 | true |  | 發行 TSLint build 工作所產生的結果。 請注意，報表僅支援 JSON 格式的 TSLint 記錄。 如果您選擇了不同的格式，請據以更新您的 TSLint 組建工作。
| ToolLogsNotFoundAction | 清單 | always | 是 | Standard | 錯誤，無，標準，警告 | 找不到所選工具（或所有工具皆已核取）時要採取的動作，表示該工具未執行。<br/><br/>**選項︰**<br/>**無：** 只有將 VSTS 變數**system.object**設定為**true**，才能將訊息寫入詳細資訊輸出資料流程。<br/>**標準：** （預設）會寫入標準輸出訊息，找不到工具的任何記錄。<br/>**警告：** 寫入黃色的警告訊息，找不到工具的任何記錄檔，這會在組建摘要頁面上顯示為警告。<br/>**錯誤：** 寫入紅色錯誤訊息，並擲回例外狀況，以中斷組建。 使用此選項可確保個別的工具選項，以確保執行的工具。

## <a name="security-report-task"></a>安全性報告工作

| **InputType**      | **型別**     | **可行**            | **必要** | **預設值**             | **選項（適用于挑選清單）**                                   | **說明**                                                                                                                                                                                                                                                                                                                            |
|------------|---------------|-----------------------|----------|---------------------------|----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| VstsConsole | boolean | always | 否 | true |  | 將結果寫入管線主控台。
| TsvFile | boolean | always | 否 | true |  | 產生 tsv 檔案（定位字元分隔值），並在每個找到的結果中加上一行，並以 tab 鍵分隔結果的資訊。
| HtmlFile | boolean | always | 否 | true |  | 產生 html 報表檔案。
| AllTools | boolean | always | 是 | false |  | 報告所有安全開發工具組建工作所產生的結果。
| BinSkim | boolean | AllTools = false | 是 | false |  | BinSkim 組建工作所產生的報表結果。
| BinSkimBreakOn | 清單 | AllTools = true 或 BinSkim = true | 是 | 錯誤 | Error、WarningAbove | 要報告的結果層級。
| CredScan | boolean | AllTools = false | 是 | false |  | 認證掃描器組建工作所產生的報表結果。
| MSRD | boolean | AllTools = false | 是 | false |  | MSRD build 工作所啟動之 MSRD 作業的報表作業資訊和作業 url。 MSRD 作業長時間執行，並提供個別的報表。
| RoslynAnalyzers | boolean | AllTools = false | 是 | false |  | Roslyn Analyzer 組建工作所產生的報表結果。
| RoslynAnalyzersBreakOn | 清單 | AllTools = true 或 RoslynAnalyzers = true | 是 | 錯誤 | Error、WarningAbove | 要報告的結果層級。
| TSLint | boolean | AllTools = false | 是 | false |  | TSLint 組建工作所產生的報表結果。 請注意，報表僅支援 JSON 格式的 TSLint 記錄。 如果您選擇了不同的格式，請據以更新您的 TSLint 組建工作。
| TSLintBreakOn | 清單 | AllTools = true 或 TSLint = true | 是 | 錯誤 | Error、WarningAbove | 要報告的結果層級。
| ToolLogsNotFoundAction | 清單 | always | 是 | Standard | 錯誤，無，標準，警告 | 找不到所選工具（或所有工具皆已核取）時要採取的動作，表示該工具未執行。<br/><br/>**選項︰**<br/>**無：** 只有將 VSTS 變數**system.object**設定為**true**，才能將訊息寫入詳細資訊輸出資料流程。<br/>**標準：** （預設）會寫入標準輸出訊息，找不到工具的任何記錄。<br/>**警告：** 寫入黃色的警告訊息，找不到工具的任何記錄檔，這會在組建摘要頁面上顯示為警告。<br/>**錯誤：** 寫入紅色錯誤訊息，並擲回例外狀況，以中斷組建。 使用此選項可確保個別的工具選項，以確保執行的工具。
| CustomLogsFolder | string | always | 否 |  |  | 分析工具記錄所在的基底資料夾;個別記錄檔會在此路徑底下的每個工具後面命名的子資料夾中。

## <a name="post-analysis-task"></a>後續分析工作

| **InputType**      | **型別**     | **可行**            | **必要** | **預設值**             | **選項（適用于挑選清單）**                                   | **說明**                                                                                                                                                                                                                                                                                                                            |
|------------|---------------|-----------------------|----------|---------------------------|----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| AllTools | boolean | always | 是 | false |  | 如果任何 Microsoft 安全性程式碼分析組建工作發現任何問題，請中斷組建。
| BinSkim | boolean | AllTools = false | 是 | false |  | 如果找到任何 BinSkim 問題，請根據您所選取的 [中斷] 選項來中斷組建。
| BinSkimBreakOn | 清單 | AllTools = true 或 BinSkim = true | 是 | 錯誤 | Error、WarningAbove | 要中斷組建的問題層級。
| CredScan | boolean | AllTools = false | 是 | false |  | 如果找到任何認證掃描器問題，請中斷組建。
| RoslynAnalyzers | boolean | AllTools = false | 是 | false |  | 如果找到任何 Roslyn 分析器問題，請中斷組建。
| RoslynAnalyzersBreakOn | 清單 | AllTools = true 或 RoslynAnalyzers = true | 是 | 錯誤 | Error、WarningAbove | 要中斷組建的問題層級。
| TSLint | boolean | AllTools = false | 是 | false |  | 如果發現任何 TSLint 問題，請中斷組建。 請注意，只有 JSON 格式的 TSLint 記錄支援 post 分析。 如果您選擇了不同的格式，請據以更新您的 TSLint 組建工作。
| TSLintBreakOn | 清單 | AllTools = true 或 TSLint = true | 是 | 錯誤 | Error、WarningAbove | 要中斷組建的問題層級。
| VstsConsole | boolean | always | 否 | true |  | 將結果寫入管線主控台。
| ToolLogsNotFoundAction | 清單 | always | 是 | Standard | 錯誤，無，標準，警告 | 找不到所選工具（或所有工具皆已核取）時要採取的動作，表示該工具未執行。<br/><br/>**選項︰**<br/>**無：** 只有將 VSTS 變數**system.object**設定為**true**，才能將訊息寫入詳細資訊輸出資料流程。<br/>**標準：** （預設）會寫入標準輸出訊息，找不到工具的任何記錄。<br/>**警告：** 寫入黃色的警告訊息，找不到工具的任何記錄檔，這會在組建摘要頁面上顯示為警告。<br/>**錯誤：** 寫入紅色錯誤訊息，並擲回例外狀況，以中斷組建。 使用此選項可確保個別的工具選項，以確保執行的工具。

## <a name="next-steps"></a>後續步驟

如果您對安全性程式碼分析延伸模組和提供的工具有進一步的疑問，請查看[我們的常見問題頁面](security-code-analysis-faq.md)。