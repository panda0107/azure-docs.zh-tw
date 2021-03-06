---
title: 在自訂原則中定義 OAuth2 技術設定檔
titleSuffix: Azure AD B2C
description: 在 Azure Active Directory B2C 的自訂原則中定義 OAuth2 技術設定檔。
services: active-directory-b2c
author: mmacy
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 02/13/2020
ms.author: marsma
ms.subservice: B2C
ms.openlocfilehash: d0fc5e6b5cafa22da6707a8f34675dcbdf5af8cc
ms.sourcegitcommit: 333af18fa9e4c2b376fa9aeb8f7941f1b331c11d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/13/2020
ms.locfileid: "77198015"
---
# <a name="define-an-oauth2-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>在 Azure Active Directory B2C 自訂原則中定義 OAuth2 技術設定檔

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C （Azure AD B2C）提供 OAuth2 通訊協定識別提供者的支援。 OAuth2 是用於授權和委派驗證的主要通訊協定。 如需詳細資訊，請參閱 [RFC 6749 The OAuth 2.0 授權架構](https://tools.ietf.org/html/rfc6749)。 透過 OAuth2 技術設定檔，您可以與以 OAuth2 為基礎的身分識別提供者（例如 Facebook）聯盟。 與身分識別提供者聯盟可讓使用者使用其現有的社交或企業身分識別登入。

## <a name="protocol"></a>通訊協定

**Protocol** 元素的 **Name** 屬性必須設定為 `OAuth2`。 例如，**Facebook-OAUTH** 技術設定檔的通訊協定是 `OAuth2`：

```XML
<TechnicalProfile Id="Facebook-OAUTH">
  <DisplayName>Facebook</DisplayName>
  <Protocol Name="OAuth2" />
  ...
```

## <a name="input-claims"></a>輸入宣告

**InputClaims** 和 **InputClaimsTransformations** 元素不是必要項目。 但您可能會想要將其他參數傳送至您的識別提供者。 下列範例會將 **domain_hint** 查詢字串參數 (包含 `contoso.com` 的值) 新增至授權要求。

```XML
<InputClaims>
  <InputClaim ClaimTypeReferenceId="domain_hint" DefaultValue="contoso.com" />
</InputClaims>
```

## <a name="output-claims"></a>輸出宣告

**OutputClaims** 元素包含 OAuth2 識別提供者傳回的宣告清單。 您可能需要將原則中定義的宣告名稱對應至識別提供者中定義的名稱。 只要設定了 `DefaultValue` 屬性，也可以加入識別提供者未傳回的宣告。

**OutputClaimsTransformations** 元素可能含有 **OutputClaimsTransformation** 的集合，用於修改輸出宣告或產生新的輸出宣告。

下列範例顯示 Facebook 識別提供者傳回的宣告：

- **first_name** 宣告對應至 **givenName** 宣告。
- **last_name** 宣告對應至 **surname** 宣告。
- **DisplayName**宣告沒有名稱對應。
- **email** 宣告沒有名稱對應。

技術設定檔也會傳回識別提供者未傳回的宣告：

- 包含識別提供者名稱的 **identityProvider** 宣告。
- 具有 **socialIdpAuthentication** 預設值的 **authenticationSource** 宣告。

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="id" />
  <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="first_name" />
  <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="last_name" />
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
  <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
  <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="facebook.com" />
  <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
</OutputClaims>
```

## <a name="metadata"></a>中繼資料

| 屬性 | 必要項 | 描述 |
| --------- | -------- | ----------- |
| client_id | 是 | 識別提供者的應用程式識別碼。 |
| IdTokenAudience | 否 | id_token 的對象。 如果已指定，Azure AD B2C 會檢查權杖是否在識別提供者傳回的宣告中，以及是否等於指定的宣告。 |
| authorization_endpoint | 是 | 依據 RFC 6749 的授權端點 URL。 |
| AccessTokenEndpoint | 是 | 依據 RFC 6749 的權杖端點 URL。 |
| ClaimsEndpoint | 是 | 依據 RFC 6749 的使用者資訊端點 URL。 |
| AccessTokenResponseFormat | 否 | 存取權杖端點呼叫的格式。 例如，Facebook 需要 HTTP GET 方法，但存取權杖回應採用 JSON 格式。 |
| AdditionalRequestQueryParameters | 否 | 其他要求查詢參數。 例如，您可以將其他參數傳送給識別提供者。 您可以使用逗號分隔符號包含多個參數。 |
| ClaimsEndpointAccessTokenName | 否 | 存取權杖查詢字串參數的名稱。 某些識別提供者的宣告端點支援 GET HTTP 要求。 在此情況下，會使用查詢字串參數來傳送持有人權杖，而不使用授權標頭。 |
| ClaimsEndpointFormatName | 否 | 格式查詢字串參數的名稱。 例如，您可以在 LinkedIn 宣告端點 `format` 中將名稱設定為 `https://api.linkedin.com/v1/people/~?format=json`。 |
| ClaimsEndpointFormat | 否 | 格式查詢字串參數的值。 例如，您可以在 LinkedIn 宣告端點 `json` 中將值設定為 `https://api.linkedin.com/v1/people/~?format=json`。 |
| ProviderName | 否 | 識別提供者的名稱。 |
| response_mode | 否 | 識別提供者用來將結果傳送回 Azure AD B2C 的方法。 可能的值：`query`、`form_post` (預設值) 或 `fragment`。 |
| 範圍 | 否 | 根據 OAuth2 識別提供者規格定義之要求的範圍。 例如，`openid`、`profile` 和 `email`。 |
| HttpBinding | 否 | 繫結至存取權杖和宣告權杖端點的預期 HTTP。 可能的值：`GET` 或 `POST`。  |
| ResponseErrorCodeParamName | 否 | 包含透過 HTTP 200 (Ok) 傳回之錯誤訊息的參數名稱。 |
| ExtraParamsInAccessTokenEndpointResponse | 否 | 包含可在某些識別提供者的 **AccessTokenEndpoint** 回應中傳回的額外參數。 例如，**AccessTokenEndpoint** 的回應中包含 `openid` 之類的額外參數，這是 **ClaimsEndpoint** 要求查詢字串中除了 access_token 以外的必要參數。 多個參數名稱應逸出，並以逗號 ',' 分隔。 |
| ExtraParamsInClaimsEndpointRequest | 否 | 包含可在某些識別提供者的 **ClaimsEndpoint** 要求中傳回的額外參數。 多個參數名稱應逸出，並以逗號 ',' 分隔。 |
| IncludeClaimResolvingInClaimsHandling  | 否 | 針對輸入和輸出宣告，指定技術設定檔中是否包含[宣告解析](claim-resolver-overview.md)。 可能的值： `true`或 `false` （預設）。 如果您想要在技術設定檔中使用宣告解析程式，請將此設定為 [`true`]。 |

## <a name="cryptographic-keys"></a>密碼編譯金鑰

**CryptographicKeys** 元素包含下列屬性：

| 屬性 | 必要項 | 描述 |
| --------- | -------- | ----------- |
| client_secret | 是 | 識別提供者應用程式的用戶端密碼。 只有在 **response_types** 中繼資料設為 `code` 時，才需要密碼編譯金鑰。 在此情況下，Azure AD B2C 會進行另一次呼叫，以交換存取權杖的授權碼。 如果中繼資料設定為 `id_token`，您可以省略密碼編譯金鑰。 |

## <a name="redirect-uri"></a>重新導向 URI

當您設定識別提供者的重新導向 URL 時，請輸入 `https://login.microsoftonline.com/te/tenant/policyId/oauth2/authresp`。 請務必將 **tenant** 取代為您的租用戶名稱 (例如 contosob2c.onmicrosoft.com)，並將 **policyId** 取代為原則的識別碼 (例如 b2c_1a_policy)。 重新導向 URI 必須全部小寫。

如果使用的是 **b2clogin.com** 網域，而非使用 **login.microsoftonline.com**，請務必使用 b2clogin.com，而非使用 login.microsoftonline.com。

範例：

- [使用自訂原則新增 Google+ 作為 OAuth2 識別提供者](identity-provider-google-custom.md)













