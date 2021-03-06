---
title: 使用 PowerShell 的多網站裝載
titleSuffix: Azure Application Gateway
description: 了解如何使用 Azure PowerShell 來建立應用程式閘道，裝載多個站台。
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 11/14/2019
ms.author: victorh
ms.openlocfilehash: 449095c92c30638b25836a2c7803176f7f0512e5
ms.sourcegitcommit: b1a8f3ab79c605684336c6e9a45ef2334200844b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/13/2019
ms.locfileid: "74048072"
---
# <a name="create-an-application-gateway-with-multiple-site-hosting-using-azure-powershell"></a>使用 Azure PowerShell，建立有多站台裝載的應用程式閘道

您可以使用 Azure PowerShell，在建立[應用程式閘道](application-gateway-multi-site-overview.md)時設定[裝載多個站台](application-gateway-introduction.md)。 在本教學課程中，您可以使用虛擬機器擴展集建立後端集區。 接著，您可以根據擁有的網域來設定接聽程式和規則，確保網路流量會抵達集區中的適當伺服器。 本教學課程假設您擁有多個網域，並使用 *www.contoso.com* 和 *www.fabrikam.com* 的範例。

在本文中，您將了解：

> [!div class="checklist"]
> * 設定網路
> * 建立應用程式閘道
> * 建立接聽程式和路由規則
> * 建立包含後端集區的虛擬機器擴展集
> * 在網域中建立 CNAME 記錄

![多站台路由範例](./media/application-gateway-create-multisite-azureresourcemanager-powershell/scenario.png)

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 PowerShell，本教學課程會要求使用 Azure PowerShell 模組 1.0.0 版或更新版本。 若要尋找版本，請執行 `Get-Module -ListAvailable Az`。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-az-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzAccount` 以建立與 Azure 的連線。

## <a name="create-a-resource-group"></a>建立資源群組

資源群組是在其中部署與管理 Azure 資源的邏輯容器。 使用 [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) 來建立 Azure 資源群組。  

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroupAG -Location eastus
```

## <a name="create-network-resources"></a>建立網路資源

使用 *New-AzVirtualNetworkSubnetConfig* 來設定名為 *myBackendSubnet* 和 [myAGSubnet](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) 的子網路。 使用 *New-AzVirtualNetwork* 搭配子網路設定來建立名為 [myVNet](/powershell/module/az.network/new-azvirtualnetwork) 的虛擬網路。 最後，使用 *New-AzPublicIpAddress* 來建立名為 [myAGPublicIPAddress](/powershell/module/az.network/new-azpublicipaddress) 的公用 IP 位址。 這些資源可用來為應用程式閘道及其相關聯的資源提供網路連線。

```azurepowershell-interactive
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

使用 *New-AzApplicationGatewayIPConfiguration*，讓先前建立的 [myAGSubnet](/powershell/module/az.network/new-azapplicationgatewayipconfiguration) 與應用程式閘道產生關聯。 使用 *New-AzApplicationGatewayFrontendIPConfig*，將 [myAGPublicIPAddress](/powershell/module/az.network/new-azapplicationgatewayfrontendipconfig) 指派給應用程式閘道。

```azurepowershell-interactive
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
$frontendport = New-AzApplicationGatewayFrontendPort `
  -Name myFrontendPort `
  -Port 80
```

### <a name="create-the-backend-pools-and-settings"></a>建立後端集區和設定

使用[add-azapplicationgatewaybackendaddresspool](/powershell/module/az.network/new-azapplicationgatewaybackendaddresspool)，為應用程式閘道建立名為*contosoPool*和*fabrikamPool*的後端集區。 使用 [New-AzApplicationGatewayBackendHttpSettings](/powershell/module/az.network/new-azapplicationgatewaybackendhttpsetting) 設定集區的設定。

```azurepowershell-interactive
$contosoPool = New-AzApplicationGatewayBackendAddressPool `
  -Name contosoPool 
$fabrikamPool = New-AzApplicationGatewayBackendAddressPool `
  -Name fabrikamPool 
$poolSettings = New-AzApplicationGatewayBackendHttpSettings `
  -Name myPoolSettings `
  -Port 80 `
  -Protocol Http `
  -CookieBasedAffinity Enabled `
  -RequestTimeout 120
```

### <a name="create-the-listeners-and-rules"></a>建立接聽程式和規則

需要接聽程式才能讓應用程式閘道將流量適當地路由到後端集區。 在本教學課程中，您會為兩個網域各自建立接聽程式。 在此範例中，會為 www.contoso.com 和 www.fabrikam.com 網域建立接聽程式。

使用 New-azapplicationgatewayHTTPlistener 搭配您先前建立的前端設定和前端埠，來建立名為*contosoListener*和*fabrikamListener*的接聽[程式](/powershell/module/az.network/new-azapplicationgatewayhttplistener)。 接聽程式需要一些規則，以便知道要針對連入流量使用哪個後端集區。 使用 New-azapplicationgatewayrequestroutingrule 建立名為*contosoRule*和*fabrikamRule* [的](/powershell/module/az.network/new-azapplicationgatewayrequestroutingrule)基本規則。

```azurepowershell-interactive
$contosolistener = New-AzApplicationGatewayHttpListener `
  -Name contosoListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendport `
  -HostName "www.contoso.com"
$fabrikamlistener = New-AzApplicationGatewayHttpListener `
  -Name fabrikamListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendport `
  -HostName "www.fabrikam.com"
$contosoRule = New-AzApplicationGatewayRequestRoutingRule `
  -Name contosoRule `
  -RuleType Basic `
  -HttpListener $contosoListener `
  -BackendAddressPool $contosoPool `
  -BackendHttpSettings $poolSettings
$fabrikamRule = New-AzApplicationGatewayRequestRoutingRule `
  -Name fabrikamRule `
  -RuleType Basic `
  -HttpListener $fabrikamListener `
  -BackendAddressPool $fabrikamPool `
  -BackendHttpSettings $poolSettings
```

### <a name="create-the-application-gateway"></a>建立應用程式閘道

既然您已建立必要的支援資源，接著請使用 *New-AzApplicationGatewaySku* 為名為 [myAppGateway](/powershell/module/az.network/new-azapplicationgatewaysku) 的應用程式閘道指定參數，然後使用 [New-AzApplicationGateway](/powershell/module/az.network/new-azapplicationgateway) 來建立它。

```azurepowershell-interactive
$sku = New-AzApplicationGatewaySku `
  -Name Standard_Medium `
  -Tier Standard `
  -Capacity 2
$appgw = New-AzApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -BackendAddressPools $contosoPool, $fabrikamPool `
  -BackendHttpSettingsCollection $poolSettings `
  -FrontendIpConfigurations $fipconfig `
  -GatewayIpConfigurations $gipconfig `
  -FrontendPorts $frontendport `
  -HttpListeners $contosoListener, $fabrikamListener `
  -RequestRoutingRules $contosoRule, $fabrikamRule `
  -Sku $sku
```

## <a name="create-virtual-machine-scale-sets"></a>建立虛擬機器擴展集

在此範例中，您要建立兩個虛擬機器擴展集，以支援您所建立的兩個後端集區。 您所建立的擴展集名為 myvmss1 和 myvmss2。 每個擴展集都會包含兩個您安裝 IIS 的虛擬機器執行個體。 當您設定 IP 設定時，要將擴展集指派給後端集區。

```azurepowershell-interactive
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet
$appgw = Get-AzApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway
$contosoPool = Get-AzApplicationGatewayBackendAddressPool `
  -Name contosoPool `
  -ApplicationGateway $appgw
$fabrikamPool = Get-AzApplicationGatewayBackendAddressPool `
  -Name fabrikamPool `
  -ApplicationGateway $appgw
for ($i=1; $i -le 2; $i++)
{
  if ($i -eq 1) 
  {
    $poolId = $contosoPool.Id
  }
  if ($i -eq 2)
  {
    $poolId = $fabrikamPool.Id
  }
  $ipConfig = New-AzVmssIpConfig `
    -Name myVmssIPConfig$i `
    -SubnetId $vnet.Subnets[1].Id `
    -ApplicationGatewayBackendAddressPoolsId $poolId
  $vmssConfig = New-AzVmssConfig `
    -Location eastus `
    -SkuCapacity 2 `
    -SkuName Standard_DS2 `
    -UpgradePolicyMode Automatic
  Set-AzVmssStorageProfile $vmssConfig `
    -ImageReferencePublisher MicrosoftWindowsServer `
    -ImageReferenceOffer WindowsServer `
    -ImageReferenceSku 2016-Datacenter `
    -ImageReferenceVersion latest
  Set-AzVmssOsProfile $vmssConfig `
    -AdminUsername azureuser `
    -AdminPassword "Azure123456!" `
    -ComputerNamePrefix myvmss$i
  Add-AzVmssNetworkInterfaceConfiguration `
    -VirtualMachineScaleSet $vmssConfig `
    -Name myVmssNetConfig$i `
    -Primary $true `
    -IPConfiguration $ipConfig
  New-AzVmss `
    -ResourceGroupName myResourceGroupAG `
    -Name myvmss$i `
    -VirtualMachineScaleSet $vmssConfig
}
```

### <a name="install-iis"></a>安裝 IIS

```azurepowershell-interactive
$publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1"); 
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }

for ($i=1; $i -le 2; $i++)
{
  $vmss = Get-AzVmss `
    -ResourceGroupName myResourceGroupAG `
    -VMScaleSetName myvmss$i
  Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
    -Name "customScript" `
    -Publisher "Microsoft.Compute" `
    -Type "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -Setting $publicSettings
  Update-AzVmss `
    -ResourceGroupName myResourceGroupAG `
    -Name myvmss$i `
    -VirtualMachineScaleSet $vmss
}
```

## <a name="create-cname-record-in-your-domain"></a>在網域中建立 CNAME 記錄

在以公用 IP 位址建立應用程式閘道之後，您可以取得 DNS 位址並用以在網域中建立 CNAME 記錄。 您可以使用 [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress) 取得應用程式閘道的 DNS 位址。 複製 DNSSettings 的 *fqdn* 值，並用來作為所建立 CNAME 記錄的值。 不建議使用 A-records，因為重新啟動應用程式閘道時，可能會變更 VIP。

```azurepowershell-interactive
Get-AzPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

## <a name="test-the-application-gateway"></a>測試應用程式閘道

在瀏覽器的網址列中輸入您的網域名稱。 例如， https://www.contoso.com。

![在應用程式閘道中測試 contoso 網站](./media/application-gateway-create-multisite-azureresourcemanager-powershell/application-gateway-iistest.png)

將位址變更為您的另一個網域，您應該會看到類似下列的範例：

![測試應用程式閘道中的 fabrikam 網站](./media/application-gateway-create-multisite-azureresourcemanager-powershell/application-gateway-iistest2.png)

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何：

> [!div class="checklist"]
> * 設定網路
> * 建立應用程式閘道
> * 建立接聽程式和路由規則
> * 建立包含後端集區的虛擬機器擴展集
> * 在網域中建立 CNAME 記錄

> [!div class="nextstepaction"]
> [深入了解應用程式閘道的用途](application-gateway-introduction.md)
