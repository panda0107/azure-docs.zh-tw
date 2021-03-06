---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 05/13/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 2bfdf1046c67ed1651f792191923bf4c533d0299
ms.sourcegitcommit: 333af18fa9e4c2b376fa9aeb8f7941f1b331c11d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/13/2020
ms.locfileid: "77205713"
---
此文章將回答有關 Azure 受控磁碟和 Azure 進階 SSD 磁碟的一些常見問題。

## <a name="managed-disks"></a>受控磁碟

**何謂 Azure 受控磁碟？**

受控磁碟是可以為您管理儲存體帳戶而簡化 Azure IaaS VM 磁碟管理的一項功能。 如需詳細資訊，請參閱[受控磁碟概觀](../articles/virtual-machines/windows/managed-disks-overview.md)。

**如果我從現有的 VHD (大小為 80 GB) 建立標準受控磁碟，需要多少費用？**

從 80 GB VHD 建立的標準受控磁碟會被視為下一個可用的標準磁碟大小 (S10 磁碟)。 將根據 S10 磁碟定價向您收費。 如需詳細資訊，請參閱[價格頁面](https://azure.microsoft.com/pricing/details/storage)。

**標準受控磁碟有任何交易成本嗎？**

是的。 我們會根據每一筆交易向您收費。 如需詳細資訊，請參閱[價格頁面](https://azure.microsoft.com/pricing/details/storage)。

**對於標準受控磁碟，收費是依據磁碟上的資料實際大小，還是磁碟的佈建容量？**

收費是依據磁碟的佈建容量。 如需詳細資訊，請參閱[價格頁面](https://azure.microsoft.com/pricing/details/storage)。

**進階受控磁碟和非受控磁碟的價格有何不同？**

進階受控磁碟與進階非受控磁碟的價格相同。

**我是否可以變更受控磁碟的儲存體帳戶類型 (標準或進階)？**

是的。 您可以使用 Azure 入口網站、PowerShell 或 Azure CLI 來變更受控磁碟的儲存體帳戶類型。

**我是否可以使用 Azure 儲存體帳戶中的 VHD 檔案，透過不同的訂用帳戶來建立受控磁碟？**

是的。

**我是否可以使用 Azure 儲存體帳戶中的 VHD 檔案在不同的區域中建立受控磁碟？**

No。

**客戶使用受控磁碟時是否有任何規模限制？**

受控磁碟沒有儲存體帳戶方面的限制。 然而，上限為一個訂用帳戶每一區域和每一磁碟類型 50,000 個受控磁碟。

**我是否可以建立受控磁碟的增量快照集？**

No。 目前的快照集功能會建立受控磁碟的完整複本。

**可用性設定組中的 VM 是否可以由受控和非受控磁碟混合組成？**

No。 可用性設定組中的 VM 必須全部使用受控磁碟，或全部使用非受控磁碟。 當您建立可用性設定組時，可以選擇想要使用的磁碟類型。

**受控磁碟是否為 Azure 入口網站中的預設選項？**

是的。

**我是否可以建立空的受控磁碟？**

是的。 您可以建立空的磁碟。 受控磁碟可在 VM 外獨立建立，例如，不連結至 VM。

**使用受控磁碟的可用性設定組支援的容錯網域計數為何？**

根據使用受控磁碟的可用性設定組所在區域，支援的容錯網域計數為 2 或 3。

**如何設定診斷的標準儲存體帳戶？**

您需要為 VM 診斷設定私人儲存體帳戶。

**受控磁碟適用何種角色型存取控制支援？**

受控磁碟支援三個主要預設角色︰

* 擁有者：可以管理所有項目，包括存取
* 參與者：可以管理存取以外的所有項目
* 讀取者：可以檢視所有項目，但是無法進行變更

**是否有方法可以將受控磁碟複製或匯出至私人儲存體帳戶？**

您可以產生受控磁碟的唯讀共用存取簽章 (SAS) URI，並使用它將內容複製到私人儲存體帳戶或內部部署儲存體。 您可以透過 Azure 入口網站、Azure PowerShell、Azure CLI 或 [AzCopy](../articles/storage/common/storage-use-azcopy.md) 來使用 SAS URI

**我是否可以建立受控磁碟的複本？**

客戶可以建立受控磁碟的快照集，然後使用快照集建立另一個受控磁碟。

**是否仍然支援非受控磁碟？**

是，受控磁碟和非受控磁碟都受到支援。 我們建議您使用受控磁碟來處理新的工作負載，並將您目前的工作負載移轉至受控磁碟。

**是否可以在相同的虛擬機器上共置非受控和受控磁碟？**

No。

**如果我建立 128 GB 的磁片，然後將大小增加到 130 gib （GiB），則會向我收取下一個磁片大小的費用（256 GiB）嗎？**

是的。

**我是否可以建立本地備援儲存體、異地備援儲存體和區域備援儲存體受控磁碟？**

Azure 受控磁碟目前只支援本地備援儲存體受控磁碟。

**我是否可以壓縮受控磁碟或縮減其大小？**

No。 目前不受支援此功能。

**可以中斷磁碟上的租用嗎？**

No。 目前不支援此作業，因為租用的存在可防止在使用磁碟時意外刪除。

**當特製化 (不是透過使用系統準備工具或一般化所建立) 作業系統磁碟用來佈建 VM 時，我是否可以變更電腦名稱屬性？**

No。 您無法更新電腦名稱屬性。 新的 VM 將從其父 VM 繼承它，並用來建立作業系統磁碟。 

**哪裡可以找到 Azure Resource Manager 範本範例以建立具有受控磁碟的 VM**
* [使用受控磁碟的範本清單](https://github.com/Azure/azure-quickstart-templates/blob/master/managed-disk-support-list.md)
* https://github.com/chagarw/MDPP

**從 Blob 建立磁碟時，與該來源 Blob 是否有任何繼續存在的關係？**

否，建立新磁碟時，新磁碟會是該時間該 Blob 的完整獨立複本，兩者之間沒有關係。 您可以視需要在建立磁碟之後刪除來源 Blob，而不會對新建立的磁碟有任何影響。

**在建立受控或非受控磁碟之後，是否可以將其重新命名？**

就受控磁碟而言，您無法將其重新命名。 不過，針對非受控磁碟，只要該磁碟目前未連接至 VHD 或 VM，您便可以將其重新命名。

**我可以在 Azure 磁碟上使用 GPT 分割嗎？**

第1代映射只能在資料磁片（而不是 OS 磁片）上使用 GPT 分割。 OS 磁碟必須使用 MBR 分割區樣式。

[第2代映射](https://docs.microsoft.com/azure/virtual-machines/linux/generation-2)可以在 OS 磁片和資料磁片上使用 GPT 分割。

**哪些磁片類型支援快照集？**

進階 SSD、標準 SSD 和標準 HDD 支援快照集。 針對這三種磁片類型，所有磁片大小（包括大小上限為 32 TiB 的磁片）都支援快照集。 Ultra 磁片不支援快照集。

**什麼是 Azure 磁片保留專案？**
磁片保留可讓您事先購買一年的磁片儲存體，以降低總成本。 如需有關 Azure 磁片保留的詳細資訊，請參閱主題的文章：[瞭解保留折扣如何套用至 Azure 磁片](../articles/cost-management-billing/reservations/understand-disk-reservations.md)。

**Azure 磁片保留提供哪些選項？** Azure 磁片保留可讓您選擇從 P30 （1 TiB）中的指定 Sku 購買 Premium Ssd （一年期），最高可達 P80 （32 TiB）。 購買磁片保留所需的最小磁片量並無任何限制。 此外，您也可以選擇使用單一、預先付款或每月付款來支付費用。 進階 SSD 受控磁碟不會套用額外的交易成本。 

保留是以磁片的形式進行，而不是容量。 換句話說，當您保留 P80 （32 TiB）磁片時，您會取得單一 P80 磁片，然後無法將該特定的保留區分割成兩個較小的 P70 （16個 TiB）磁片。 當然，您可以盡可能保留所需的磁片數量或數目，包括兩個不同的 P70 （16 TiB）磁片。

**Azure 磁片保留的套用方式為何？**  
磁片保留會遵循類似于保留的虛擬機器（VM）實例的模型。 其差異在於，磁片保留無法套用至不同的 Sku，而 VM 實例則可以。 如需 VM 實例的詳細資訊，請參閱[使用 Azure 保留的 VM 執行個體節省成本](../articles/virtual-machines/linux/prepay-reserved-vm-instances.md)。    

我**可以在多個區域中使用透過 Azure 磁片保留購買的資料儲存體嗎？**     
Azure 磁片保留會針對特定區域和 SKU （例如美國東部2中的 P30）購買，因此不能在這些結構外使用。 您一律可以針對其他區域或 Sku 中的磁片儲存體需求購買額外的 Azure 磁片保留。 

**當我的 Azure 磁片保留過期時，會發生什麼事？**    
您將會在到期前30天收到電子郵件通知，並在到期日再次出現。 保留到期後，部署的磁片將會繼續執行，且會以最新的[隨用隨付費率](https://azure.microsoft.com/pricing/details/managed-disks/)計費。

### <a name="azure-shared-disks"></a>Azure 共用磁片

**共用磁片功能支援非受控磁片或分頁 blob 嗎？**

否，只有 premium SSD 受控磁片才支援。

**哪些區域支援共用磁片？**

目前只有美國中西部。

**可以使用共用磁片做為 OS 磁片嗎？**

否，只有資料磁片才支援共用磁片。

**哪些磁片大小支援共用磁片？**

只有 P15 或更高支援共用磁片的 premium Ssd。

**如果我有現有的 premium SSD，可以在其上啟用共用磁片嗎？**

使用 API 版本2019-07-01 或更高版本建立的所有受控磁片都可以啟用共用磁片。 若要這樣做，您必須將磁片從其所連接的所有 Vm 卸載。 接下來，編輯磁片上的 `maxShares` 屬性。

**如果我不想再使用共用模式的磁片，如何停用它？**

將磁片從其所連接的所有 Vm 卸載。 然後，將磁片上的 maxShare 屬性編輯為1。

**您可以調整共用磁片的大小嗎？**

是的。

**我可以在同時啟用共用磁片的磁片上啟用寫入加速器嗎？**

No。

**我可以為已啟用共用磁片的磁片啟用主機快取嗎？**

唯一支援的主機快取選項為 [無]。

## <a name="ultra-disks"></a>Ultra 磁片

**我該如何將 ultra 磁片輸送量設定為？**
如果您不確定要如何設定磁片輸送量，建議您先從假設 IO 大小 16 KiB 開始，並在監視應用程式時從該處調整效能。 此公式為：輸送量（MBps = IOPS） * 16/1000。

**我將磁片設定為 40000 IOPS，但我只看到 12800 IOPS，為什麼看不到磁片的效能？**
除了磁片節流，還有在 VM 層級強加的 IO 節流。 請確定您使用的 VM 大小可支援磁片上所設定的層級。 如需 VM 所加諸之 IO 限制的詳細資訊，請參閱[Azure 中的 Windows 虛擬機器大小](../articles/virtual-machines/windows/sizes.md)。

**我可以搭配使用快取層級與 ultra 磁片嗎？**
否，ultra 磁片不支援其他磁片類型支援的不同快取方法。 將磁碟快取設定為 [無]。

**我可以將 ultra 磁片連結至現有的 VM 嗎？**
可能的話，您的 VM 必須位於支援 Ultra 磁片的區域和可用性區域配對中。 如需詳細資訊，請參閱[開始使用 ultra 磁片](../articles/virtual-machines/windows/disks-enable-ultra-ssd.md)。

**我可以使用 ultra 磁片做為 VM 的 OS 磁片嗎？**
否，ultra 磁片僅支援做為資料磁片，且僅支援4K 原生磁片。

**我可以將現有的磁片轉換成 ultra 磁片嗎？**
否，但是您可以將資料從現有的磁片遷移到 ultra 磁片。 若要將現有的磁片遷移至 ultra 磁片，請將這兩個磁片連接到相同的 VM，並將磁片的資料從一個磁碟複製到另一個，或利用協力廠商解決方案進行資料移轉。

**我可以建立 ultra 磁片的快照集嗎？**
否，快照集尚無法使用。

**Azure 備份適用于 ultra 磁片嗎？**
否，尚未提供 Azure 備份支援。

**我可以將 ultra 磁片連結至在可用性設定組中執行的 VM 嗎？**
否，尚不支援這種情況。

**我可以使用 ultra 磁片啟用 Vm 的 Azure Site Recovery 嗎？**
否，尚未支援 ultra 磁片的 Azure Site Recovery。

## <a name="uploading-to-a-managed-disk"></a>上傳至受控磁片

**我可以將資料上傳到現有的受控磁片嗎？**

否，只有在建立具有**ReadyToUpload**狀態的新空磁片時，才可以使用上傳。

**如何? 上傳至受控磁片？**

建立受控磁片，並將[creationData](https://docs.microsoft.com/rest/api/compute/disks/createorupdate#creationdata)的[createOption](https://docs.microsoft.com/rest/api/compute/disks/createorupdate#diskcreateoption)屬性設定為「上傳」，然後您就可以將資料上傳至其中。

**是否可以在 VM 處於上傳狀態時，將磁片連結至 VM？**

No。

**我可以在上傳狀態中拍攝受控磁片的快照集嗎？**

No。

## <a name="standard-ssd-disks"></a>標準 SSD 磁碟

**Azure 標準 SSD 磁碟是什麼？**
標準 SSD 磁碟是受固態媒體支援的標準磁碟，針對在較低 IOPS 層級中需要一致效能的工作負載，最佳化為符合成本效益的儲存體。

<a id="standard-ssds-azure-regions"></a>**目前支援標準 SSD 磁碟的區域有哪些？**
所有 Azure 區域現在都支援「標準 SSD」磁碟。

**使用標準 SSD 時，可使用 Azure 備份嗎？**
是，Azure 備份現在可供使用。

**如何建立標準 SSD 磁碟？**
您可以使用 Azure Resource Manager 範本、SDK、PowerShell 或 CLI 來建立標準 SSD 磁片。 以下是在 Resource Manager 範本中建立標準 SSD 磁碟所需的參數：

* Microsoft.Compute 的 apiVersion 必須設定為 `2018-04-01` (或更新版本)
* 將 managedDisk.storageAccountType 指定為 `StandardSSD_LRS`

下列範例示範使用標準 SSD 磁碟的虛擬機器 properties.storageProfile.osDisk 區段：

```json
"osDisk": {
    "osType": "Windows",
    "name": "myOsDisk",
    "caching": "ReadWrite",
    "createOption": "FromImage",
    "managedDisk": {
        "storageAccountType": "StandardSSD_LRS"
    }
}
```

如需如何使用範本建立標準 SSD 磁碟的完整範本範例，請參閱[從具有標準 SSD 資料磁碟的 Windows 映像建立虛擬機器](https://github.com/azure/azure-quickstart-templates/tree/master/101-vm-with-standardssd-disk/)。

**是否可以將我現有的磁碟轉換成標準的 SSD？**
是，您可以。 如需轉換受控磁碟的一般指導方針，請參閱[將 Azure 受控磁碟儲存體從標準轉換至進階，反之亦然](https://docs.microsoft.com/azure/virtual-machines/windows/convert-disk-storage)。 此外，使用下列值，將磁碟類型更新為標準 SSD。
-AccountType StandardSSD_LRS

**使用標準 SSD 磁碟而不使用 HDD 有何優點？**
相較于 HDD 磁片，標準 SSD 磁片提供更佳的延遲、一致性、可用性和可靠性。 因此，應用程式工作負載在「標準 SSD」上執行起來更加順暢。 請注意，針對大多數 IO 密集型生產環境工作負載，建議的解決方案是「進階 SSD」磁碟。

**可以使用標準 SSD 作為非受控磁碟嗎？**
否，標準 SSD 磁碟僅適用於受控磁碟。

**標準 SSD 是否支援「單一執行個體虛擬機器 SLA」？**
否，標準 SSD 沒有單一執行個體虛擬機器 SLA。 請針對單一執行個體虛擬機器 SLA 使用進階 SSD 磁碟。

## <a name="migrate-to-managed-disks"></a>移轉到受控磁碟

**移轉對於受控磁碟的效能是否有任何影響？**

移轉涉及從一個儲存體位置將磁碟移動到另一個儲存體位置。 這會透過資料的背景複本來協調，這可能需要幾個小時才能完成，通常不到24小時，視磁片中的資料量而定。 在這段期間，應用程式可能出現比平常高的讀取延遲，因為某些讀取會重新導向至原始位置，因此可能需要較長的時間才能完成。 在這段期間，對於寫入延遲沒有任何影響。  

**在移轉至受控磁碟前/後，預先存在的 Azure 備份服務設定需要哪些變更？**

不需要任何變更。

**在移轉繼續運作之前，是否會透過 Azure 備份服務建立我的 VM 備份？**

是，備份會順利運作。

**在移轉至受控磁碟前/後，預先存在的 Azure 磁碟加密設定需要哪些變更？**

不需要任何變更。

**是否將現有虛擬機器擴展集從非受控磁片自動遷移至受控磁碟支援？**

No。 您可以使用舊擴展集中的映像搭配非受控磁碟，建立包含受控磁碟的新擴展集。

**在移轉至受控磁碟之前，是否可以從頁面 blob 快照集建立受控磁碟？**

No。 您可以匯出分頁 Blob 快照集作為分頁 Blob，然後再從匯出的分頁 Blob 建立受控磁碟。

**是否可以將受 Azure Site Recovery 保護的內部部署機器容錯移轉至具有受控磁碟的 VM？**

是，您可以選擇容錯移轉至具有受控磁碟的 VM。

**在透過 Azure 受 Azure Site Recovery 保護的 Azure 虛擬機器上移轉至 Azure 複寫是否有任何影響？**

No。 針對具有受控磁碟的 Vm，Azure Site Recovery Azure 到 Azure 的保護。

**我是否可以將具有非受控磁碟 (位於之前已加密的儲存體帳戶上) 的 VM 移轉至受控磁碟？**

是

## <a name="managed-disks-and-storage-service-encryption"></a>受控磁碟和儲存體服務加密

**當我建立受控磁碟時，依預設是否啟用 Azure 儲存體服務加密？**

是的。

**受控磁片上的開機磁碟區預設是否加密？**

是的。 根據預設，所有受控磁片都會加密，包括 OS 磁片。

**誰負責管理加密金鑰？**

Microsoft 負責管理加密金鑰。

**我是否可以停用受控磁碟的儲存體服務加密？**

No。

**儲存體服務加密是否僅供特定地區使用？**

No。 它在受控磁碟可以使用的區域中都有提供。 受控磁碟在所有公開區域和德國都有提供。 也可以在中國使用，不過僅適用於 Microsoft 受控金鑰，而不適用於客戶受控金鑰。

**如何查看我的受控磁碟是否加密？**

您可以從 Azure 入口網站、Azure CLI 和 PowerShell 找出建立受控磁碟的時間。 如果時間是在 2017 年 6 月 9 日之後，那麼您的磁碟是加密的。

**如何加密我在 2017 年 6 月 10 日之前建立的現有磁碟？**

自 2017 年 6 月 10 日起，系統會自動將寫入現有受控磁碟的新資料加密。 我們也計劃對現有資料進行加密，加密將以非同步方式在背景處理。 如果您現在必須加密現有資料，請建立一份磁碟複本。 新的磁碟將會加密。

* [使用 Azure CLI 複製受控磁碟](../articles/virtual-machines/scripts/virtual-machines-linux-cli-sample-copy-managed-disks-to-same-or-different-subscription.md?toc=%2fcli%2fmodule%2ftoc.json)
* [使用 PowerShell 複製受控磁碟](../articles/virtual-machines/scripts/virtual-machines-windows-powershell-sample-copy-managed-disks-to-same-or-different-subscription.md?toc=%2fcli%2fmodule%2ftoc.json)

**受控快照集和映像是否加密？**

是的。 2017 年 6 月 9 日後建立之所有受控快照集和映像都將自動加密。 

**我是否可以將具有非受控磁碟 (位於之前已加密的儲存體帳戶上) 的 VM 轉換為受控磁碟？**

是

**從受控磁碟或快照集匯出的 VHD 是否也會加密？**

No。 但是，如果將 VHD 從加密的受控磁碟或快照集匯出到加密儲存體帳戶，則會將它加密。 

## <a name="premium-disks-managed-and-unmanaged"></a>進階磁碟：受控和非受控

**如果虛擬機器使用的大小系列支援進階 SSD 磁碟，例如 DSv2，我是否可以同時連結進階和標準資料磁碟？** 

是的。

**我是否可以同時將進階和標準資料磁碟連結至不支援進階 SSD 磁碟的大小系列，例如 D、Dv2、G 或 F 系列？**

No。 只有當虛擬機器未使用支援進階 SSD 磁碟的大小系列時，您才能將標準資料磁碟連結至虛擬機器。

**如果我從現有的 VHD (大小為 80 GB) 建立進階資料磁碟，需要多少費用？**

從 80 GB VHD 建立的進階資料磁碟會被視為下一個可用的進階磁碟大小 (P10 磁碟)。 將根據 P10 磁碟定價向您收費。

**使用進階 SSD 磁碟是否有交易成本？**

依 IOPS 和輸送量的特定限制而佈建的每個磁碟大小，都有固定成本。 其他成本包括輸出頻寬和快照集容量 (如果適用的話)。 如需詳細資訊，請參閱[價格頁面](https://azure.microsoft.com/pricing/details/storage)。

**我可以從磁碟快取取得的 IOPS 和輸送量有何限制？**

DS 系列的快取和本機 SSD 合併限制是每個核心 4,000 IOPS，以及每個核心每秒 33 MiB。 GS 系列提供每個核心 5000 IOPS，以及每個核心每秒 50 MiB。

**受控磁碟 VM 是否支援本機 SSD？**

本機 SSD 是受控磁碟 VM 隨附的暫時儲存體。 暫時儲存體不需額外的成本。 建議您不要使用本機 SSD 來儲存應用程式資料，因為它不會保留在 Azure Blob 儲存體中。

**在進階磁碟上使用 TRIM 是否有任何影響？**

在進階或標準磁碟的 Azure 磁碟上使用 TRIM 並無任何不妥。

## <a name="new-disk-sizes-managed-and-unmanaged"></a>新磁碟大小：受控和非受控

**哪些區域支援適用的 premium SSD 磁片大小的高載功能？**

Azure 美國中西部目前支援高載功能。

**支援的區域有 4/8/16 GiB 受控磁片大小（P1/P2/P3、E1/E2/E3）？**

Azure 美國中西部目前支援這些新的磁片大小。

**非受控磁片或分頁 blob 是否支援 P1/P2/P3 磁片大小？**

否，只有在 premium SSD 受控磁片上才支援。 

**非受控磁片或分頁 blob 支援 E1/E2/E3 磁片大小嗎？**

否，任何大小的標準 SSD 受控磁片不能與非受控磁片或分頁 blob 搭配使用。

**作業系統和資料磁碟支援的最大受控磁碟大小是多少？**

Azure 針對作業系統磁碟所支援的磁碟分割類型是主開機記錄 (MBR)。 MBR 格式支援的磁碟大小上限為 2 TiB。 Azure 針對作業系統磁碟支援的大小上限為 2 TiB。 Azure 支援最大 32 TiB 的受控資料磁碟。

**作業系統和資料磁碟支援的最大非受控磁碟大小是多少？**

Azure 針對作業系統磁碟所支援的磁碟分割類型是主開機記錄 (MBR)。 MBR 格式支援的磁碟大小上限為 2 TiB。 Azure 針對作業系統非受控磁碟支援的大小上限為 2 TiB。 Azure 支援最大 4 TiB 的非受控資料磁碟。

**支援的分頁 Blob 大小上限是多少？**

Azure 支援的分頁 Blob 大小上限是 8 TiB (8,191 GiB)。 連結至 VM 作為資料或作業系統磁碟時，分頁 Blob 大小上限為 4 TiB (4,095 GiB)。

**我是否需要使用新版 Azure 工具來建立磁碟、連結磁碟、調整磁碟大小及上傳大於 1 TiB 的磁碟？**

您不需要升級現有的 Azure工具來建立磁碟、連結磁碟或調整大於1 TiB 的磁碟大小。 若要將 VHD 檔案從內部部署直接上傳到 Azure 作為分頁 Blob 或非受控磁碟，您需要使用下表所列的最新工具集。 我們僅支援最多 8 TiB 的 VHD 上傳。

|Azure 工具      | 支援的版本                                |
|-----------------|---------------------------------------------------|
|Azure PowerShell | 版本號碼 4.1.0：2017 年 6 月發行或更新版本|
|Azure CLI v1     | 版本號碼 0.10.13：2017 年 5 月發行或更新版本|
|Azure CLI v2     | 版本號碼 2.0.12：2017 年 7 月發行或更新版本|
|AzCopy           | 版本號碼 6.1.0：2017 年 6 月發行或更新版本|

**針對非受控磁碟或分頁 Blob，是否支援 P4 和 P6 磁碟大小？**

非受控磁碟和分頁 Blob 的預設磁碟層不支援 P4 (32 GiB) 和 P6 (64 GiB) 磁碟大小。 您需要明確地[將 Blob 層設為](https://docs.microsoft.com/rest/api/storageservices/set-blob-tier) P4 和 P6，才能讓磁碟對應至這些層。 若您以小於 32 GitB 或介於 32 GiB 到 64 GiB 之間的磁碟大小或內容長度來部署非受控磁碟或分頁 Blob，但並未先設定 Blob 層，則您會持續使用 P10 搭配 500 IOPS 和每秒 100 MiB，以及對應的定價層。

**如果小於 64 GiB 的現有進階受控磁碟，是在啟用小型磁碟 (2017 年 6 月 15 日前後) 之前建立，該如何計費？**

根據 P10 定價層，小於 64 GiB 的現有小型進階磁碟繼續計費。

**我要如何將小於 64 GiB 的小型進階磁碟的磁碟層，從 P10 切換到 P4 或 P6？**

您可以為小型磁碟建立快照集，然後建立一個磁碟，根據佈建大小自動將定價層切換為 P4 或 P6。

**您是否可以將現有的受控磁碟大小從少於 4 tib （TiB）調整為新引進的最高 32 TiB 磁片大小？**

是的。

**Azure 備份和 Azure Site Recovery 服務支援的最大磁片大小為何？**

Azure 備份支援的最大磁片大小為 32 TiB （適用于加密磁片的4個 TiB）。 Azure Site Recovery 支援的最大磁片大小為 8 TiB。 Azure Site Recovery 中尚未提供最多 32 TiB 的較大磁片支援。

**適用于標準 SSD 和標準 HDD 磁片的最大磁片大小（> 4 TiB）建議的 VM 大小為何，以達到優化的磁片 IOPS 和頻寬？**

若要達到標準 SSD 的磁片輸送量，並標準 HDD 超過 500 IOPS 和 60 MiB/秒的大型磁片大小（> 4 TiB），我們建議您從下列其中一種 VM 大小部署新的 VM，以優化您的效能： B 系列、DSv2 系列、Dsv3 系列、ESv3 系列、Fs 系列、Fsv2 系列、M 系列、GS 系列、NCv2 系列、NCv3 系列或 Ls 系列 Vm。 將大型磁片連結至現有的 Vm 或未使用上述建議大小的 Vm，可能會遇到較低的效能。

**我要如何升級在較大的磁片大小預覽期間部署的磁片（> 4 TiB），以便在 GA 中取得更高的 IOPS & 頻寬？**

您可以停止並啟動磁片所連接的 VM，或是卸離並重新連接磁片。 Premium Ssd 和 standard Ssd 在 GA 中的效能目標較大。

**有哪些區域支援 8 TiB、16 TiB 和 32 TiB 的受控磁片大小？**

全球 Azure、Microsoft Azure Government 和 Azure 中國世紀境內的所有區域都支援 8 TiB、16 TiB 和 32 TiB 磁片 Sku。

**是否支援在所有磁片大小上啟用主機快取？**

在小於 4 TiB 的磁片大小上，我們支援唯讀和讀取/寫入的主機快取。 針對超過 4 TiB 的磁碟大小，我們只支援設定為「無」的快取選項。 建議您利用快取較小型磁碟大小的方式，以便您透過快取到 VM 的資料增進效能。

## <a name="what-if-my-question-isnt-answered-here"></a>如果這裡沒有解答我的問題該怎麼辦？

如果這裡未列出您的問題，請告訴我們，我們將協助您找到答案。 您可以在此文章結尾處將問題張貼在註解中。 若要就此文章與 Azure 儲存體小組和其他社群成員互動，請使用 MSDN [Azure 儲存體論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazuredata)。

若要提出功能要求，請將要求和想法提交到 [Azure 儲存體意見反應論壇](https://feedback.azure.com/forums/217298-storage)。
