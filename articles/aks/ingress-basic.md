---
title: 在 Azure Kubernetes Service (AKS) 中建立基本輸入控制器
description: 了解如何在 Azure Kubernetes Service (AKS) 叢集中安裝及設定基本 NGINX 輸入控制器。
services: container-service
author: mlearned
ms.service: container-service
ms.topic: article
ms.date: 12/20/2019
ms.author: mlearned
ms.openlocfilehash: 4299846f48524799084efa5456c169d18af4185d
ms.sourcegitcommit: c32050b936e0ac9db136b05d4d696e92fefdf068
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/08/2020
ms.locfileid: "75728866"
---
# <a name="create-an-ingress-controller-in-azure-kubernetes-service-aks"></a>在 Azure Kubernetes Service (AKS) 中建立輸入控制器

輸入控制器是一項可為 Kubernetes 服務提供反向 Proxy、可設定的流量路由和 TLS 終止的軟體。 Kubernetes 輸入資源可用來設定個別 Kubernetes 服務的輸入規則和路由。 透過輸入控制器和輸入規則，您可以使用單一 IP 位址將流量路由至 Kubernetes 叢集中的多個服務。

本文說明如何在 Azure Kubernetes Service （AKS）叢集中部署[NGINX 輸入控制器][nginx-ingress]。 然後，會有兩個應用程式在 AKS 叢集中執行，且均可透過單一 IP 位址來存取。

您也可以：

- [啟用 HTTP 應用程式路由附加元件][aks-http-app-routing]
- [建立使用內部、私人網路和 IP 位址的輸入控制器][aks-ingress-internal]
- [建立使用您自己的 TLS 憑證的輸入控制器][aks-ingress-own-tls]
- 建立輸入控制器，其使用 Let's Encrypt 自動產生[具有動態公用 ip 位址][aks-ingress-tls]或[具有靜態公用 IP 位址][aks-ingress-static-tls]的 TLS 憑證

## <a name="before-you-begin"></a>開始之前

本文使用 Helm 來安裝 NGINX 輸入控制器及範例 Web 應用程式。

本文也會要求您執行 Azure CLI 版本2.0.64 或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][azure-cli-install]。

## <a name="create-an-ingress-controller"></a>建立輸入控制器

若要建立輸入控制器，請使用 Helm 來安裝*nginx*輸入。 為了新增備援，您必須使用 `--set controller.replicaCount` 參數部署兩個 NGINX 輸入控制器複本。 為充分享有執行輸入控制器複本的好處，請確定 AKS 叢集中有多個節點。

輸入控制器也必須在 Linux 節點上排程。 Windows Server 節點（目前在 AKS 中處於預覽狀態）不應執行輸入控制器。 您可以使用 `--set nodeSelector` 參數來指定節點選取器，以指示 Kubernetes 排程器在以 Linux 為基礎的節點上執行 NGINX 輸入控制器。

> [!TIP]
> 下列範例會建立名為「輸入 *-基本*」的輸入資源的 Kubernetes 命名空間。 視需要指定您自己環境的命名空間。

> [!TIP]
> 如果您想要為叢集中的容器要求啟用[用戶端來源 IP 保留][client-source-ip]，請將 `--set controller.service.externalTrafficPolicy=Local` 新增至 Helm install 命令。 用戶端來源 IP 會儲存在要求標頭的 [ *X-轉送-* ] 下。 當使用已啟用用戶端來源 IP 保留的輸入控制器時，SSL 傳遞將無法運作。

```console
# Create a namespace for your ingress resources
kubectl create namespace ingress-basic

# Add the official stable repository
helm repo add stable https://kubernetes-charts.storage.googleapis.com/

# Use Helm to deploy an NGINX ingress controller
helm install nginx-ingress stable/nginx-ingress \
    --namespace ingress-basic \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```

為 NGINX 輸入控制器建立 Kubernetes 負載平衡器服務時，系統會指派動態公用 IP 位址，如下列範例輸出所示：

```
$ kubectl get service -l app=nginx-ingress --namespace ingress-basic

NAME                             TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
nginx-ingress-controller         LoadBalancer   10.0.61.144    EXTERNAL_IP   80:30386/TCP,443:32276/TCP   6m2s
nginx-ingress-default-backend    ClusterIP      10.0.192.145   <none>        80/TCP                       6m2s
```

尚未建立任何輸入規則，因此，如果您瀏覽至內部 IP 位址，將會顯示 NGINX 輸入控制器的預設 404 頁面。 輸入規則會於下列步驟中進行設定。

## <a name="run-demo-applications"></a>執行示範應用程式

為了檢視運作中的輸入控制器，我們將在 AKS 叢集中執行兩個示範應用程式。 在此範例中，Helm 是用來部署簡單*Hello world*應用程式的兩個實例。

在您可以安裝範例 Helm 圖表之前，先將 Azure 範例存放庫新增至您的 Helm 環境，如下所示：

```console
helm repo add azure-samples https://azure-samples.github.io/helm-charts/
```

使用下列命令，從 Helm 圖表建立第一個示範應用程式：

```console
helm install aks-helloworld azure-samples/aks-helloworld --namespace ingress-basic
```

現在，請安裝示範應用程式的第二個執行個體。 對第二個執行個體指定新的標題，以便在視覺上區分這兩個應用程式。 您也要指定唯一的服務名稱：

```console
helm install aks-helloworld-two azure-samples/aks-helloworld \
    --namespace ingress-basic \
    --set title="AKS Ingress Demo" \
    --set serviceName="aks-helloworld-two"
```

## <a name="create-an-ingress-route"></a>建立輸入路由

這兩個應用程式現在都已在您的 Kubernetes 叢集上執行。 若要將流量路由至每個應用程式，請建立 Kubernetes 輸入資源。 輸入資源會設定將流量路由至這兩個應用程式之一的規則。

在下列範例中， *EXTERNAL_IP*的流量會路由傳送至名為 `aks-helloworld`的服務。 *EXTERNAL_IP/hello-world-two*的流量會路由傳送至 `aks-helloworld-two` 服務。 *EXTERNAL_IP/靜態*的流量會路由傳送至靜態資產的名為 `aks-helloworld` 的服務。

建立名為 `hello-world-ingress.yaml` 的檔案，並複製到下列範例 YAML 中。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  namespace: ingress-basic
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: aks-helloworld
          servicePort: 80
        path: /(.*)
      - backend:
          serviceName: aks-helloworld-two
          servicePort: 80
        path: /hello-world-two(/|$)(.*)
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress-static
  namespace: ingress-basic
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/rewrite-target: /static/$2
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: aks-helloworld
          servicePort: 80
        path: /static(/|$)(.*)
```

使用 `kubectl apply -f hello-world-ingress.yaml` 命令建立輸入資源。

```
$ kubectl apply -f hello-world-ingress.yaml

ingress.extensions/hello-world-ingress created
ingress.extensions/hello-world-ingress-static created
```

## <a name="test-the-ingress-controller"></a>測試輸入控制器

若要測試輸入控制器的路由，請瀏覽至這兩個應用程式。 將網頁瀏覽器開啟至 NGINX 輸入控制器的 IP 位址，例如*EXTERNAL_IP*。 第一個示範應用程式會顯示在網頁瀏覽器中，如下列範例所示：

![執行於輸入控制器後方的第一個應用程式](media/ingress-basic/app-one.png)

現在，將 */hello-world-two*路徑新增至 IP 位址，例如*EXTERNAL_IP/hello-world-two*。 此時會顯示具有自訂標題的第二個示範應用程式：

![執行於輸入控制器後方的第二個應用程式](media/ingress-basic/app-two.png)

## <a name="clean-up-resources"></a>清除資源

本文使用 Helm 來安裝輸入元件和範例應用程式。 部署 Helm 圖表時會建立一些 Kubernetes 資源。 這些資源包含 Pod、部署和服務。 若要清除這些資源，您可以刪除整個範例命名空間或個別資源。

### <a name="delete-the-sample-namespace-and-all-resources"></a>刪除範例命名空間和所有資源

若要刪除整個範例命名空間，請使用 `kubectl delete` 命令，並指定您的命名空間名稱。 命名空間中的所有資源都會被刪除。

```console
kubectl delete namespace ingress-basic
```

然後，移除 AKS hello world 應用程式的 Helm 存放庫：

```console
helm repo remove azure-samples
```

### <a name="delete-resources-individually"></a>個別刪除資源

或者，更細微的方法是刪除所建立的個別資源。 使用 `helm list` 命令來列出 Helm 版本。 尋找名為nginx-ingress 和 aks-helloworld 的圖表，如下列範例輸出所示：

```
$ helm list --namespace ingress-basic

NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
aks-helloworld          ingress-basic   1               2020-01-06 19:57:00.131576 -0600 CST    deployed        aks-helloworld-0.1.0               
aks-helloworld-two      ingress-basic   1               2020-01-06 19:57:10.971365 -0600 CST    deployed        aks-helloworld-0.1.0               
nginx-ingress           ingress-basic   1               2020-01-06 19:55:46.358275 -0600 CST    deployed        nginx-ingress-1.27.1    0.26.1  
```

使用 `helm delete` 命令刪除版本。 下列範例會刪除 NGINX 輸入部署和兩個範例 AKS hello world 應用程式。

```
$ helm delete aks-helloworld aks-helloworld-two nginx-ingress --namespace ingress-basic

release "aks-helloworld" uninstalled
release "aks-helloworld-two" uninstalled
release "nginx-ingress" uninstalled
```

接下來，移除 AKS hello world 應用程式的 Helm 存放庫：

```console
helm repo remove azure-samples
```

移除將流量導向範例應用程式的輸入路由：

```console
kubectl delete -f hello-world-ingress.yaml
```

最後，您可以刪除本身的命名空間。 使用 `kubectl delete` 命令，並指定您的命名空間名稱：

```console
kubectl delete namespace ingress-basic
```

## <a name="next-steps"></a>後續步驟

本文包含 AKS 的一些外部元件。 若要深入了解這些元件，請參閱下列專案頁面：

- [Helm CLI][helm-cli]
- [NGINX 輸入控制器][nginx-ingress]

您也可以：

- [啟用 HTTP 應用程式路由附加元件][aks-http-app-routing]
- [建立使用內部、私人網路和 IP 位址的輸入控制器][aks-ingress-internal]
- [建立使用您自己的 TLS 憑證的輸入控制器][aks-ingress-own-tls]
- 建立輸入控制器，其使用 Let's Encrypt 自動產生[具有動態公用 ip 位址][aks-ingress-tls]或[具有靜態公用 IP 位址][aks-ingress-static-tls]的 TLS 憑證

<!-- LINKS - external -->
[helm-cli]: https://docs.microsoft.com/azure/aks/kubernetes-helm
[nginx-ingress]: https://github.com/kubernetes/ingress-nginx

<!-- LINKS - internal -->
[use-helm]: kubernetes-helm.md
[azure-cli-install]: /cli/azure/install-azure-cli
[aks-ingress-internal]: ingress-internal-ip.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-own-tls]: ingress-own-tls.md
[client-source-ip]: concepts-network.md#ingress-controllers
