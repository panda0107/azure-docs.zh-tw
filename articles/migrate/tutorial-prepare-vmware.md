---
title: 使用 Azure Migrate 準備 VMware VM 以進行評量/移轉
description: 了解如何使用 Azure Migrate 準備進行 VMware VM 的評量/移轉。
ms.topic: tutorial
ms.date: 11/19/2019
ms.custom: mvc
ms.openlocfilehash: f00d5ba4841427098b0ab79ad1930e357008b6e0
ms.sourcegitcommit: f0f73c51441aeb04a5c21a6e3205b7f520f8b0e1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/05/2020
ms.locfileid: "77030790"
---
# <a name="prepare-vmware-vms-for-assessment-and-migration-to-azure"></a>準備 VMware VM 以進行評量並移轉至 Azure

本文將協助您使用 [Azure Migrate](migrate-services-overview.md) 來準備內部部署 VMware VM 的評量並 (或) 將其移轉至 Azure。

[Azure Migrate](migrate-overview.md) 會提供工具中樞，協助您探索和評估應用程式、基礎結構和工作負載，並且將這些項目遷移至 Microsoft Azure。 此中樞包含 Azure Migrate 工具和第三方獨立軟體廠商 (ISV) 供應項目。


本教學課程是一個系列中的第一篇，說明如何評估及遷移 VMware VM。 在本教學課程中，您會了解如何：

> [!div class="checklist"]
> * 準備 Azure 以使用 Azure Migrate。
> * 準備 VMware 以進行 VM 評量。
> * 準備 VMware 以進行 VM 移轉。

> [!NOTE]
> 這些教學課程示範案例的最簡單部署路徑。 當您了解如何設定部署並作為快速的概念證明時，這些功能就很實用。 教學課程在情況允許時都會使用預設選項，且不會顯示所有可能的設定與路徑。 如需詳細指示，請參閱 VMware 評量和移轉的操作說明。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。


## <a name="prepare-azure"></a>準備 Azure

您需要這些權限。

**Task** | **權限**
--- | ---
**建立 Azure Migrate 專案** | 您的 Azure 帳戶需要可建立專案的權限。
**註冊 Azure Migrate 設備** | Azure Migrate 會使用輕量的 Azure Migrate 設備搭配 Azure Migrate 伺服器評量來評估 VMware VM，以及搭配 Azure Migrate 伺服器移轉來執行[無代理程式移轉](server-migrate-overview.md)。 此設備會探索 VM，並將 VM 的中繼資料和效能資料傳送至 Azure Migrate。<br/><br/>在設備註冊期間，下列資源提供者會使用設備中選擇的訂用帳戶進行註冊 - Microsoft.OffAzure、Microsoft.Migrate 和 Microsoft.KeyVault。 註冊資源提供者可將您的訂用帳戶設定為可搭配資源提供者使用。 若要註冊資源提供者，您必須具有訂用帳戶的「參與者」或「擁有者」角色。<br/><br/> 在上線期間，Azure Migrate 會建立兩個 Azure Active Directory (Azure AD) 應用程式：<br/> - 第一個應用程式可供在設備上執行的代理程式與其在 Azure 上執行的各項服務之間進行通訊 (驗證和授權)。 此應用程式沒有在任何資源上進行 ARM 呼叫或 RBAC 存取的權限。<br/> - 第二個應用程式專門用來存取在使用者的訂用帳戶中建立的 KeyVault，以進行無代理程式移轉。 從設備起始探索時，此應用程式具有 Azure Key Vault 上的 RBAC 存取權 (建立於客戶的租用戶中)。
**建立 Key Vault** | 若要使用 Azure Migrate 伺服器移轉來遷移 VMware VM，Azure Migrate 會建立 Key Vault 來管理訂用帳戶中複寫儲存體帳戶的存取金鑰。 若要建立保存庫，您在 Azure Migrate 專案所在的資源群組上必須有角色指派權限。






### <a name="assign-permissions-to-create-project"></a>指派建立專案的權限

1. 在 Azure 入口網站中開啟訂用帳戶，然後選取 [存取控制 (IAM)]  。
2. 在 [檢查存取權]  中，尋找相關的帳戶，然後按一下以查看權限。
3. 您應該會具有「參與者」  或「擁有者」  權限。
    - 如果您剛建立免費的 Azure 帳戶，您就是訂用帳戶的擁有者。
    - 如果您不是訂用帳戶擁有者，請與擁有者合作以指派角色。

### <a name="assign-permissions-to-register-the-appliance"></a>指派權限以註冊設備

若要註冊設備，您將指派 Azure Migrate 的權限，以在設備註冊期間建立 Azure AD 應用程式。 您可以使用下列其中一種方法來指派權限：

- 租用戶/全域管理員可將權限授與租用戶中的使用者，以建立及註冊 Azure AD 應用程式。
- 租用戶/全域管理員可為帳戶指派應用程式開發人員角色 (具有權限)。

> [!NOTE]
> - 除了前述權限外，應用程式在訂用帳戶上沒有任何其他存取權限。
> - 只有當您註冊新的設備時，才需要這些權限。 完成設備的設定後，您可以移除權限。


#### <a name="grant-account-permissions"></a>授與帳戶權限

租用戶/全域管理員可依照下列方式授與權限

1. 在 Azure AD 中，租用戶/全域管理員應該瀏覽至 [Azure Active Directory]   > [使用者]   > [使用者設定]  。
2. 管理員應將 [應用程式註冊]  設定為 [是]  。 這是一個不敏感的預設設定。 [深入了解](https://docs.microsoft.com/azure/active-directory/develop/active-directory-how-applications-are-added#who-has-permission-to-add-applications-to-my-azure-ad-instance)。

    ![Azure AD 權限](./media/tutorial-prepare-vmware/aad.png)



#### <a name="assign-application-developer-role"></a>指派應用程式開發人員角色

租用戶/全域管理員可為帳戶指派應用程式開發人員角色。 [深入了解](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-users-assign-role-azure-portal)。

### <a name="assign-permissions-to-create-a-key-vault"></a>指派建立金鑰保存庫的權限

若要讓 Azure Migrate 建立 Key Vault，請依照下列方式指派權限：

1. 在 Azure 入口網站的資源群組中，選取 [存取控制 (IAM)]  。
2. 在 [檢查存取權]  中，尋找相關的帳戶，然後按一下以查看權限。

    - 若要執行伺服器評量，「參與者」  權限就已足夠。
    - 若要執行無代理程式的伺服器移轉，您應具有「擁有者」  (或「參與者」  及「使用者存取系統管理員」  ) 權限。

3. 如果您沒有必要權限，請向資源群組擁有者提出要求。



## <a name="prepare-for-vmware-vm-assessment"></a>準備 VMware VM 評量

若要準備 VMware VM 評量，您必須：

- **確認 VMware 設定**。 確定您想要遷移的 vCenter Server 和 VM 符合需求。
- **設定評量帳戶**。 Azure Migrate 需要存取 vCenter Server，才能探索用於評量的 VM。
- **確認設備需求**。 確認用於評量的 Azure Migrate 設備有何部署需求。

### <a name="verify-vmware-settings"></a>確認 VMware 設定

1. [確認](migrate-support-matrix-vmware.md#vmware-requirements)用於評量的 VMware 伺服器需求。
2. [確認](migrate-support-matrix-vmware.md#port-access) vCenter 伺服器上已開啟您需要的連接埠。
3. 在 vCenter Server 上，確定您的帳戶有權使用 OVA 檔案建立 VM。 您可以使用 OVA 檔案，將 Azure Migrate 設備部署為 VMware VM。


### <a name="set-up-an-account-for-assessment"></a>設定用於評量的帳戶

Azure Migrate 需要存取 vCenter Server，才能探索用於評估和無代理程式移轉的 VM。

- 如果您打算探索應用程式，或以無代理程式的方式將相依性視覺化，請建立 vCenter Server 帳戶，並使其具有唯讀存取權以及針對 [虛擬機器]   > [客體作業]  啟用的權限。

  ![vCenter Server 帳戶權限](./media/tutorial-prepare-vmware/vcenter-server-permissions.png)

- 如果您不打算執行應用程式探索和無代理程式相依性視覺效果，請為 vCenter Server 設定唯讀帳戶。

### <a name="verify-appliance-settings-for-assessment"></a>確認用於評量的設備設定

在下一個教學課程中設定 Azure Migrate 設備並開始進行評量之前，請先準備設備部署。

1. [確認](migrate-appliance.md#appliance---vmware) VMware VM 的設備需求。
2. [檢閱](migrate-appliance.md#url-access)設備需要存取的 Azure URL。 如果您使用以 URL 為基礎的防火牆或 Proxy，請務必允許必要 URL 的存取。
3. [檢閱](migrate-appliance.md#collected-data---vmware)設備將在探索和評量期間收集的資料。
4. [注意](migrate-support-matrix-vmware.md#port-access)設備的連接埠存取需求。




## <a name="prepare-for-agentless-vmware-migration"></a>準備無代理程式的 VMware 移轉

請檢閱 VMware VM 無代理程式移轉的需求。

1. [檢閱](migrate-support-matrix-vmware-migration.md#agentless-vmware-servers) VMware 伺服器需求，以及讓 Azure Migrate 可存取 vCenter Server 並使用 Azure Migrate 伺服器移轉進行無代理程式移轉的[權限](migrate-support-matrix-vmware-migration.md#agentless-vmware-servers)。
2. [檢閱](migrate-support-matrix-vmware-migration.md#agentless-vmware-vms)透過無代理程式移轉將 VMware VM 遷移至 Azure 的需求。
4. [檢閱](migrate-support-matrix-vmware-migration.md#agentless-azure-migrate-appliance)透過 Azure Migrate 設備進行無代理程式移轉的需求。
5. 請注意無代理程式移轉所需的 [URL 存取](migrate-appliance.md#url-access)和[連接埠存取](migrate-support-matrix-vmware-migration.md#agentless-ports)。


## <a name="prepare-for-agent-based-vmware-migration"></a>準備代理程式型 VMware 移轉

請檢閱 VMware VM 進行[代理程式型移轉](server-migrate-overview.md)時的需求。

1. [檢閱](migrate-support-matrix-vmware-migration.md#agent-based-vmware-servers) VMware 伺服器需求，以及讓 Azure Migrate 可存取 vCenter Server 並使用 Azure Migrate 伺服器移轉進行代理程式型移轉的權限。
2. [檢閱](migrate-support-matrix-vmware-migration.md#agent-based-vmware-vms)透過代理程式型移轉將 VMware VM 遷移至 Azure 時的需求，包括在每個要遷移的 VM 上安裝行動服務。
3. 代理程式型移轉會使用複寫設備：
    - [檢閱](migrate-replication-appliance.md#appliance-requirements)複寫設備的部署需求，以及在設備上安裝 MySQL 的[選項](migrate-replication-appliance.md#mysql-installation)。
    - 檢閱複寫設備的 [URL](migrate-replication-appliance.md#url-access) 和[連接埠](migrate-replication-appliance.md#port-access)存取需求。
    
## <a name="next-steps"></a>後續步驟

在本教學課程中，您：

> [!div class="checklist"]
> * 設定 Azure 權限。
> * 準備VMware 以進行評估及移轉。


請繼續進行第二個教學課程，以設定 Azure Migrate 專案，並評估要遷移至 Azure 的 VMware VM。

> [!div class="nextstepaction"]
> [評估 VMware VM](./tutorial-assess-vmware.md)
