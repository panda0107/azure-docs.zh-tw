---
title: 使用 PowerShell Azure 應用程式閘道的 HTTP 至 HTTPS 重新導向
description: 了解如何使用 Azure PowerShell，以建立可將流量從 HTTP 重新導向到 HTTPS 的應用程式閘道。
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 11/14/2019
ms.author: victorh
ms.openlocfilehash: ca742483bc8977327003ee18e9716ef9c43ebfe3
ms.sourcegitcommit: 79cbd20a86cd6f516acc3912d973aef7bf8c66e4
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/14/2020
ms.locfileid: "77251714"
---
# <a name="create-an-application-gateway-with-http-to-https-redirection-using-azure-powershell"></a>使用 Azure PowerShell 以建立具有 HTTP 到 HTTPS 重新導向功能的應用程式閘道

您可以使用 Azure PowerShell，以建立包含 SSL 終止憑證的[應用程式閘道](overview.md)。 路由規則可用來將 HTTP 流量重新導向至您應用程式閘道中的 HTTPS 連接埠。 在此範例中，您也會為應用程式閘道的後端集區，建立一個包含兩個虛擬機器執行個體的[虛擬機器擴展集](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)。 

在本文中，您將了解：

> [!div class="checklist"]
> * 建立自我簽署憑證
> * 設定網路
> * 建立包含憑證的應用程式閘道
> * 新增接聽程式和重新導向規則
> * 建立包含預設後端集區的虛擬機器擴展集

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

本教學課程需要 Azure PowerShell 模組 1.0.0 版或更新版本。 執行 `Get-Module -ListAvailable Az` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-az-ps)。 若要在本教學課程中執行命令，則也需要執行 `Login-AzAccount` 以建立與 Azure 的連線。

## <a name="create-a-self-signed-certificate"></a>建立自我簽署憑證

若要在生產環境中使用，您應該匯入由受信任提供者所簽署的有效憑證。 在此教學課程中，您要使用 [New-selfsignedcertificate](https://docs.microsoft.com/powershell/module/pkiclient/new-selfsignedcertificate) 來建立自我簽署的憑證。 您可以使用 [Export-PfxCertificate](https://docs.microsoft.com/powershell/module/pkiclient/export-pfxcertificate) 搭配系統所傳回的指紋，以將 pfx 檔案從憑證匯出。

```powershell
New-SelfSignedCertificate `
  -certstorelocation cert:\localmachine\my `
  -dnsname www.contoso.com
```

您應該會看到類似這個結果的內容：

```
PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\my

Thumbprint                                Subject
----------                                -------
E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630  CN=www.contoso.com
```

使用指紋來建立 pfx 檔案：

```powershell
$pwd = ConvertTo-SecureString -String "Azure123456!" -Force -AsPlainText
Export-PfxCertificate `
  -cert cert:\localMachine\my\E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630 `
  -FilePath c:\appgwcert.pfx `
  -Password $pwd
```

## <a name="create-a-resource-group"></a>建立資源群組

資源群組是在其中部署與管理 Azure 資源的邏輯容器。 使用[remove-azresourcegroup](/powershell/module/az.resources/new-azresourcegroup)建立名為*myResourceGroupAG*的 Azure 資源群組。 

```powershell
New-AzResourceGroup -Name myResourceGroupAG -Location eastus
```

## <a name="create-network-resources"></a>建立網路資源

使用 [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) 來建立 *myBackendSubnet* 和 *myAGSubnet* 的子網路設定。 使用 [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) 搭配子網路設定來建立名為 *myVNet* 的虛擬網路。 最後，使用 [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) 來建立名為 *myAGPublicIPAddress* 的公用 IP 位址。 這些資源可用來為應用程式閘道及其相關聯的資源提供網路連線。

```powershell
$backendSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
  -AddressPrefix 10.0.1.0/24
$agSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myAGSubnet `
  -AddressPrefix 10.0.2.0/24
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myVNet `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $backendSubnetConfig, $agSubnetConfig
$pip = New-AzPublicIpAddress `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myAGPublicIPAddress `
  -AllocationMethod Dynamic
```

## <a name="create-an-application-gateway"></a>建立應用程式閘道

### <a name="create-the-ip-configurations-and-frontend-port"></a>建立 IP 設定與前端連接埠

使用 [New-AzApplicationGatewayIPConfiguration](/powershell/module/az.network/new-azapplicationgatewayipconfiguration)，讓先前建立的 *myAGSubnet* 與應用程式閘道產生關聯。 使用 [New-AzApplicationGatewayFrontendIPConfig](/powershell/module/az.network/new-azapplicationgatewayfrontendipconfig)，將 *myAGPublicIPAddress* 指派給應用程式閘道。 接著，您可以使用[add-azapplicationgatewayfrontendport 來](/powershell/module/az.network/new-azapplicationgatewayfrontendport)建立 HTTPS 埠。

```powershell
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet
$subnet=$vnet.Subnets[0]
$gipconfig = New-AzApplicationGatewayIPConfiguration `
  -Name myAGIPConfig `
  -Subnet $subnet
$fipconfig = New-AzApplicationGatewayFrontendIPConfig `
  -Name myAGFrontendIPConfig `
  -PublicIPAddress $pip
$frontendPort = New-AzApplicationGatewayFrontendPort `
  -Name myFrontendPort `
  -Port 443
```

### <a name="create-the-backend-pool-and-settings"></a>建立後端集區和設定

使用 [New-AzApplicationGatewayBackendAddressPool](/powershell/module/az.network/new-azapplicationgatewaybackendaddresspool)，為應用程式閘道建立名為 *appGatewayBackendPool* 的後端集區。 使用 [New-AzApplicationGatewayBackendHttpSettings](/powershell/module/az.network/new-azapplicationgatewaybackendhttpsetting) 來設定後端集區的設定。

```powershell
$defaultPool = New-AzApplicationGatewayBackendAddressPool `
  -Name appGatewayBackendPool 
$poolSettings = New-AzApplicationGatewayBackendHttpSettings `
  -Name myPoolSettings `
  -Port 80 `
  -Protocol Http `
  -CookieBasedAffinity Enabled `
  -RequestTimeout 120
```

### <a name="create-the-default-listener-and-rule"></a>建立預設接聽程式和規則

需要接聽程式才能讓應用程式閘道將流量適當地路由到後端集區。 在此範例中，您會建立基本接聽程式，接聽根 URL 的 HTTPS 流量。 

使用[AzApplicationGatewaySslCertificate](/powershell/module/az.network/new-azapplicationgatewaysslcertificate)建立憑證物件，然後使用[new-azapplicationgatewayHTTPlistener](/powershell/module/az.network/new-azapplicationgatewayhttplistener)與您先前建立的前端設定、前端埠和憑證，建立名為*appGatewayHttpListener*的接聽程式。 接聽程式需要規則以便知道要針對連入流量使用哪個後端集區。 使用 [New-AzApplicationGatewayRequestRoutingRule](/powershell/module/az.network/new-azapplicationgatewayrequestroutingrule) 來建立名為 *rule1* 的基本規則。

```powershell
$pwd = ConvertTo-SecureString `
  -String "Azure123456!" `
  -Force `
  -AsPlainText
$cert = New-AzApplicationGatewaySslCertificate `
  -Name "appgwcert" `
  -CertificateFile "c:\appgwcert.pfx" `
  -Password $pwd
$defaultListener = New-AzApplicationGatewayHttpListener `
  -Name appGatewayHttpListener `
  -Protocol Https `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendPort `
  -SslCertificate $cert
$frontendRule = New-AzApplicationGatewayRequestRoutingRule `
  -Name rule1 `
  -RuleType Basic `
  -HttpListener $defaultListener `
  -BackendAddressPool $defaultPool `
  -BackendHttpSettings $poolSettings
```

### <a name="create-the-application-gateway"></a>建立應用程式閘道

現在您已建立必要的支援資源，請使用 [New-AzApplicationGatewaySku](/powershell/module/az.network/new-azapplicationgatewaysku) 為名為 myAppGateway 的應用程式閘道指定參數，然後使用 [New-AzApplicationGateway](/powershell/module/az.network/new-azapplicationgateway) 搭配憑證加以建立。

```powershell
$sku = New-AzApplicationGatewaySku `
  -Name Standard_Medium `
  -Tier Standard `
  -Capacity 2
$appgw = New-AzApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -BackendAddressPools $defaultPool `
  -BackendHttpSettingsCollection $poolSettings `
  -FrontendIpConfigurations $fipconfig `
  -GatewayIpConfigurations $gipconfig `
  -FrontendPorts $frontendPort `
  -HttpListeners $defaultListener `
  -RequestRoutingRules $frontendRule `
  -Sku $sku `
  -SslCertificates $cert
```

## <a name="add-a-listener-and-redirection-rule"></a>新增接聽程式和重新導向規則

### <a name="add-the-http-port"></a>新增 HTTP 連接埠

使用[add-azapplicationgatewayfrontendport 來](/powershell/module/az.network/add-azapplicationgatewayfrontendport)，將 HTTP 埠新增至應用程式閘道。

```powershell
$appgw = Get-AzApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG
Add-AzApplicationGatewayFrontendPort `
  -Name httpPort  `
  -Port 80 `
  -ApplicationGateway $appgw
```

### <a name="add-the-http-listener"></a>新增 HTTP 接聽程式

使用[new-azapplicationgatewayHTTPlistener](/powershell/module/az.network/add-azapplicationgatewayhttplistener)，將名為*myListener*的 HTTP 接聽程式新增至應用程式閘道。

```powershell
$fipconfig = Get-AzApplicationGatewayFrontendIPConfig `
  -Name myAGFrontendIPConfig `
  -ApplicationGateway $appgw
$fp = Get-AzApplicationGatewayFrontendPort `
  -Name httpPort `
  -ApplicationGateway $appgw
Add-AzApplicationGatewayHttpListener `
  -Name myListener `
  -Protocol Http `
  -FrontendPort $fp `
  -FrontendIPConfiguration $fipconfig `
  -ApplicationGateway $appgw
```

### <a name="add-the-redirection-configuration"></a>新增重新導向設定

使用[AzApplicationGatewayRedirectConfiguration](/powershell/module/az.network/add-azapplicationgatewayredirectconfiguration)，將 HTTP 至 HTTPS 重新導向設定新增至應用程式閘道。

```powershell
$defaultListener = Get-AzApplicationGatewayHttpListener `
  -Name appGatewayHttpListener `
  -ApplicationGateway $appgw
Add-AzApplicationGatewayRedirectConfiguration -Name httpToHttps `
  -RedirectType Permanent `
  -TargetListener $defaultListener `
  -IncludePath $true `
  -IncludeQueryString $true `
  -ApplicationGateway $appgw
```

### <a name="add-the-routing-rule"></a>新增路由規則

使用[new-azapplicationgatewayrequestroutingrule](/powershell/module/az.network/add-azapplicationgatewayrequestroutingrule)，將具有重新導向設定的路由規則新增至應用程式閘道。

```powershell
$myListener = Get-AzApplicationGatewayHttpListener `
  -Name myListener `
  -ApplicationGateway $appgw
$redirectConfig = Get-AzApplicationGatewayRedirectConfiguration `
  -Name httpToHttps `
  -ApplicationGateway $appgw
Add-AzApplicationGatewayRequestRoutingRule `
  -Name rule2 `
  -RuleType Basic `
  -HttpListener $myListener `
  -RedirectConfiguration $redirectConfig `
  -ApplicationGateway $appgw
Set-AzApplicationGateway -ApplicationGateway $appgw
```

## <a name="create-a-virtual-machine-scale-set"></a>建立虛擬機器擴展集

在此範例中，您會建立虛擬機器擴展集，以在應用程式閘道中提供後端集區的伺服器。 當您設定 IP 設定時，要將擴展集指派給後端集區。

```powershell
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet
$appgw = Get-AzApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway
$backendPool = Get-AzApplicationGatewayBackendAddressPool `
  -Name appGatewayBackendPool `
  -ApplicationGateway $appgw
$ipConfig = New-AzVmssIpConfig `
  -Name myVmssIPConfig `
  -SubnetId $vnet.Subnets[1].Id `
  -ApplicationGatewayBackendAddressPoolsId $backendPool.Id
$vmssConfig = New-AzVmssConfig `
  -Location eastus `
  -SkuCapacity 2 `
  -SkuName Standard_DS2 `
  -UpgradePolicyMode Automatic
Set-AzVmssStorageProfile $vmssConfig `
  -ImageReferencePublisher MicrosoftWindowsServer `
  -ImageReferenceOffer WindowsServer `
  -ImageReferenceSku 2016-Datacenter `
  -ImageReferenceVersion latest `
  -OsDiskCreateOption FromImage
Set-AzVmssOsProfile $vmssConfig `
  -AdminUsername azureuser `
  -AdminPassword "Azure123456!" `
  -ComputerNamePrefix myvmss
Add-AzVmssNetworkInterfaceConfiguration `
  -VirtualMachineScaleSet $vmssConfig `
  -Name myVmssNetConfig `
  -Primary $true `
  -IPConfiguration $ipConfig
New-AzVmss `
  -ResourceGroupName myResourceGroupAG `
  -Name myvmss `
  -VirtualMachineScaleSet $vmssConfig
```

### <a name="install-iis"></a>安裝 IIS

```powershell
$publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1"); 
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }
$vmss = Get-AzVmss -ResourceGroupName myResourceGroupAG -VMScaleSetName myvmss
Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings
Update-AzVmss `
  -ResourceGroupName myResourceGroupAG `
  -Name myvmss `
  -VirtualMachineScaleSet $vmss
```

## <a name="test-the-application-gateway"></a>測試應用程式閘道

您可使用 [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress) 來取得應用程式閘道的公用 IP 位址。 將公用 IP 位址複製並貼到您瀏覽器的網址列。 例如， http://52.170.203.149

```powershell
Get-AzPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

![安全警告](./media/redirect-http-to-https-powershell/application-gateway-secure.png)

若要在使用自我簽署憑證時接受安全性警告，請依序按一下 [詳細資料] 與 [繼續瀏覽網頁]。 接著會顯示受保護的 IIS 網站，如下列範例所示：

![在應用程式閘道中測試基底 URL](./media/redirect-http-to-https-powershell/application-gateway-iistest.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 建立自我簽署憑證
> * 設定網路
> * 建立包含憑證的應用程式閘道
> * 新增接聽程式和重新導向規則
> * 建立包含預設後端集區的虛擬機器擴展集
