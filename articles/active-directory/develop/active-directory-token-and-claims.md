---
title: "了解 Azure AD 支持的不同令牌和声明类型 | Microsoft Docs"
description: "本指南帮助你了解和评估 Azure Active Directory (AAD) 颁发的 SAML 2.0 令牌和 JSON Web 令牌 (JWT) 令牌中的声明"
documentationcenter: na
author: dstrockis
services: active-directory
manager: mtillman
editor: 
ms.assetid: 166aa18e-1746-4c5e-b382-68338af921e2
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/07/2017
ms.author: dastrock
ms.custom: aaddev
ms.openlocfilehash: 3104b47d7ff8585142674b0ee545012f1e291ddd
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/11/2017
---
# <a name="azure-ad-token-reference"></a>Azure AD 令牌参考
Azure Active Directory (Azure AD) 在每个身份验证流的处理中发出多种安全令牌。 本文档说明每种令牌的格式、安全特征和内容。

## <a name="types-of-tokens"></a>类型的令牌
Azure AD 支持 [OAuth 2.0 授权协议](active-directory-protocols-oauth-code.md)，该协议使用 access_token 与 refresh_token。  它还支持通过 [OpenID Connect](active-directory-protocols-openid-connect-code.md) 进行身份验证和登录，其引入了第三种类型的令牌：id_token。  每个令牌表示为“持有者令牌”。

持有者令牌是一种轻型安全令牌，它授予对受保护资源的“持有者”访问权限。 从这个意义上来说，“持有者”是可以提供令牌的任何一方。 虽然接收持有者令牌需要 Azure AD 身份验证，但是仍必须采取步骤来保护令牌，以防止被不速之客拦截令牌。 因为持有者令牌没有内置机制来防止未经授权人员使用它们，所以必须在安全的通道（例如传输层安全性 (HTTPS)）中传输这些令牌。 如果持有者令牌以明文传输，则可以利用中间人攻击来获得令牌，并对受保护资源进行未经授权的访问。 当存储或缓存持有者令牌供以后使用时，也应遵循同样的安全原则。 请始终确保应用以安全的方式传输和存储持有者令牌。 有关持有者令牌的更多安全注意事项，请参阅 [RFC 6750 第 5 部分](http://tools.ietf.org/html/rfc6750)。

Azure AD 颁发的许多令牌都以 JSON Web 令牌或 JWT 的方式实现。  JWT 是一种精简的 URL 安全方法，可在两方之间传输信息。  JWT 中包含的信息也称为令牌持有者及使用者相关信息的“声明”或断言。  JWT 中的声明是为了传输而编码和序列化的 JSON 对象。  由于 Azure AD 所颁发的 JWT 已签名但未加密，因此可以轻松地检查 JWT 的内容以进行调试。  有多个工具可用于执行此操作，例如 [jwt.ms](https://jwt.ms/)。 有关 JWT 的详细信息，请参阅 [JWT 规范](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html)。

## <a name="idtokens"></a>Id_tokens
Id_token 是应用使用 [OpenID Connect](active-directory-protocols-openid-connect-code.md) 执行身份验证时收到的一种登录安全令牌形式。  它以 [JWT](#types-of-tokens) 表示，包含可让用户登录应用的声明。  可以适时使用 id_token 中的声明 - 通常用于显示帐户信息或在应用程序中进行访问控制决策。

Id_token 已签名，但目前不会加密。  应用收到 ID 令牌后，必须[验证签名](#validating-tokens)以证明令牌的真实性，并验证令牌中的几个声明来证明其有效性。  应用验证的声明根据方案要求而有所不同，但存在一些[常见的声明验证](#validating-tokens)，应用必须在每种方案中执行。

有关 id_token 声明以及 id_token 示例的信息，请参阅以下部分内容。  请注意，id_token 中的声明不按任何特定顺序返回。  此外，新声明可随时引入 id_token 中 - 应用不会在引入新声明时中断。  下面的列表中包含了撰写本文时应用确定能够解释的声明。  如有需要，可以在 [OpenID Connect 规范](http://openid.net/specs/openid-connect-core-1_0.html)中获取更多详细信息。

#### <a name="sample-idtoken"></a>示例 id_token
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83ZmU4MTQ0Ny1kYTU3LTQzODUtYmVjYi02ZGU1N2YyMTQ3N2UvIiwiaWF0IjoxMzg4NDQwODYzLCJuYmYiOjEzODg0NDA4NjMsImV4cCI6MTM4ODQ0NDc2MywidmVyIjoiMS4wIiwidGlkIjoiN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlIiwib2lkIjoiNjgzODlhZTItNjJmYS00YjE4LTkxZmUtNTNkZDEwOWQ3NGY1IiwidXBuIjoiZnJhbmttQGNvbnRvc28uY29tIiwidW5pcXVlX25hbWUiOiJmcmFua21AY29udG9zby5jb20iLCJzdWIiOiJKV3ZZZENXUGhobHBTMVpzZjd5WVV4U2hVd3RVbTV5elBtd18talgzZkhZIiwiZmFtaWx5X25hbWUiOiJNaWxsZXIiLCJnaXZlbl9uYW1lIjoiRnJhbmsifQ.
```

> [!TIP]
> 练习时，请尝试将示例 id_token 中的声明粘贴到 [jwt.ms](https://jwt.ms/) 中进行检查。
> 
> 

#### <a name="claims-in-idtokens"></a>Id_tokens 中的声明
> [!div class="mx-codeBreakAll"]
| JWT 声明 | Name | 说明 |
| --- | --- | --- |
| `appid` |应用程序 ID |标识使用令牌访问资源的应用程序。 该应用程序可以自身名义或者代表用户进行操作。 应用程序 ID 通常表示应用程序对象，但它还可以表示 Azure AD 中的服务主体对象。 <br><br> **JWT 值示例**： <br> `"appid":"15CB020F-3984-482A-864D-1D92265E8268"` |
| `aud` |目标受众 |令牌的目标接收方。 接收令牌的应用程序必须验证受众值是否正确，并拒绝任何针对其他受众的令牌。 <br><br> **SAML 值示例**： <br> `<AudienceRestriction>`<br>`<Audience>`<br>`https://contoso.com`<br>`</Audience>`<br>`</AudienceRestriction>` <br><br> **JWT 值示例**： <br> `"aud":"https://contoso.com"` |
| `appidacr` |应用程序身份验证上下文类引用 |表示对客户端进行身份验证的方式。 对于公共客户端，该值为 0。 如果使用客户端 ID 和客户端机密，则该值为 1。 <br><br> **JWT 值示例**： <br> `"appidacr": "0"` |
| `acr` |身份验证上下文类引用 |表示使用者的身份验证方式，此方式与应用程序的身份验证上下文类引用声明中的客户端身份验证截然不同。 值为“0”指示最终用户身份验证不符合 ISO/IEC 29115 要求。 <br><br> **JWT 值示例**： <br> `"acr": "0"` |
| 即时身份验证 |记录身份验证发生的日期和时间。 <br><br> **SAML 值示例**： <br> `<AuthnStatement AuthnInstant="2011-12-29T05:35:22.000Z">` | |
| `amr` |身份验证方法 |标识对令牌使用者的身份验证方式。 <br><br> **SAML 值示例**： <br> `<AuthnContextClassRef>`<br>`http://schemas.microsoft.com/ws/2008/06/identity/claims/authenticationmethod/password`<br>`</AuthnContextClassRef>` <br><br> **JWT 值示例**：`“amr”: ["pwd"]` |
| `given_name` |名字 |和对 Azure AD 用户对象的设置一样，指定用户的名。 <br><br> **SAML 值示例**： <br> `<Attribute Name=”http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname”>`<br>`<AttributeValue>Frank<AttributeValue>` <br><br> **JWT 值示例**： <br> `"given_name": "Frank"` |
| `groups` |组 |指定表示使用者的组成员身份的对象 ID。 这些值是唯一的（请参阅对象 ID），可安全地用于管理访问，例如强制要求授权才能访问资源。 组声明中包含的组通过应用程序清单的“groupMembershipClaims”属性，基于每个应用程序进行配置。 值为 null 将排除所有组；值为“SecurityGroup”将只包括“Active Directory 安全组”成员身份；值为“All”将包括安全组和 Office 365 通讯组列表。 <br><br> **SAML 值示例**： <br> `<Attribute Name="http://schemas.microsoft.com/ws/2008/06/identity/claims/groups">`<br>`<AttributeValue>07dd8a60-bf6d-4e17-8844-230b77145381</AttributeValue>` <br><br> **JWT 值示例**： <br> `“groups”: ["0e129f5b-6b0a-4944-982d-f776045632af", … ]` |
| `idp` |标识提供程序 |记录对令牌使用者进行身份验证的标识提供程序。 除非用户帐户与颁发者不在同一租户中，否则此值与颁发者声明的值相同。 <br><br> **SAML 值示例**： <br> `<Attribute Name=” http://schemas.microsoft.com/identity/claims/identityprovider”>`<br>`<AttributeValue>https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/<AttributeValue>` <br><br> **JWT 值示例**： <br> `"idp":”https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/”` |
| `iat` |IssuedAt |存储颁发令牌的时间。 它通常用于度量令牌新鲜度。 <br><br> **SAML 值示例**： <br> `<Assertion ID="_d5ec7a9b-8d8f-4b44-8c94-9812612142be" IssueInstant="2014-01-06T20:20:23.085Z" Version="2.0" xmlns="urn:oasis:names:tc:SAML:2.0:assertion">` <br><br> **JWT 值示例**： <br> `"iat": 1390234181` |
| `iss` |颁发者 |标识构造并返回令牌的安全令牌服务 (STS)。 在 Azure AD 返回的令牌中，颁发者是 sts.windows.net。 颁发者声明值中的 GUID 是 Azure AD 目录的租户 ID。 租户 ID 是目录的固定不变且可靠的标识符。 <br><br> **SAML 值示例**： <br> `<Issuer>https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/</Issuer>` <br><br> **JWT 值示例**： <br>  `"iss":”https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/”` |
| `family_name` |姓氏 |按照 Azure AD 用户对象中的定义，指定用户的姓。 <br><br> **SAML 值示例**： <br> `<Attribute Name=” http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname”>`<br>`<AttributeValue>Miller<AttributeValue>` <br><br> **JWT 值示例**： <br> `"family_name": "Miller"` |
| `unique_name` |Name |提供了一个用户可读值，用于标识令牌使用者。 此值不一定在租户中唯一，它旨在仅用于显示目的。 <br><br> **SAML 值示例**： <br> `<Attribute Name=”http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name”>`<br>`<AttributeValue>frankm@contoso.com<AttributeValue>` <br><br> **JWT 值示例**： <br> `"unique_name": "frankm@contoso.com"` |
| `oid` |对象 ID |包含 Azure AD 中的对象的唯一标识符。 此值是固定不变的，无法重新分配或重复使用。 在对 Azure AD 进行的查询中，可使用对象 ID 来标识对象。 <br><br> **SAML 值示例**： <br> `<Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">`<br>`<AttributeValue>528b2ac2-aa9c-45e1-88d4-959b53bc7dd0<AttributeValue>` <br><br> **JWT 值示例**： <br> `"oid":"528b2ac2-aa9c-45e1-88d4-959b53bc7dd0"` |
| `roles` |角色 |表示直接和间接通过组成员身份授予使用者的所有应用程序角色，可用于实施基于角色的访问控制。 可通过应用程序清单的 `appRoles` 属性，对每个应用程序定义应用程序角色。 每个应用程序角色的 `value` 属性是角色声明中显示的值。 <br><br> **SAML 值示例**： <br> `<Attribute Name="http://schemas.microsoft.com/ws/2008/06/identity/claims/role">`<br>`<AttributeValue>Admin</AttributeValue>` <br><br> **JWT 值示例**： <br> `“roles”: ["Admin", … ]` |
| `scp` |范围 |表示向客户端应用程序授予的模拟权限。 默认权限为 `user_impersonation`。 受保护资源的所有者可在 Azure AD 中注册其他值。 <br><br> **JWT 值示例**： <br> `"scp": "user_impersonation"` |
| `sub` |使用者 |标识令牌断言信息的主体，例如应用程序的用户。 此值不可变且不能重新分配或重复使用，因此可以使用它来安全地执行授权检查。 因为使用者始终会在 Azure AD 颁发的令牌中存在，我们建议在通用授权系统中使用此值。 <br> `SubjectConfirmation` 不是声明。 它描述如何对令牌的使用者进行验证。 `Bearer` 指示通过其拥有的令牌确认使用者。 <br><br> **SAML 值示例**： <br> `<Subject>`<br>`<NameID>S40rgb3XjhFTv6EQTETkEzcgVmToHKRkZUIsJlmLdVc</NameID>`<br>`<SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer" />`<br>`</Subject>` <br><br> **JWT 值示例**： <br> `"sub":"92d0312b-26b9-4887-a338-7b00fb3c5eab"` |
| `tid` |租户 ID |一个不可变且不能重复使用的标识符，用于标识颁发令牌的目录租户。 可以使用此值访问多租户应用程序中特定于租户的目录资源。 例如，可以在调用 Graph API 时使用此值标识租户。 <br><br> **SAML 值示例**： <br> `<Attribute Name=”http://schemas.microsoft.com/identity/claims/tenantid”>`<br>`<AttributeValue>cbb1a5ac-f33b-45fa-9bf5-f37db0fed422<AttributeValue>` <br><br> **JWT 值示例**： <br> `"tid":"cbb1a5ac-f33b-45fa-9bf5-f37db0fed422"` |
| `nbf`, `exp` |令牌生存期 |定义令牌保持有效状态的时间间隔。 服务验证令牌的当前日期是否在令牌生存期内，如果不是，则拒绝令牌。 考虑到 Azure AD 与服务之间可能存在时钟时间差异（“时间偏差”），服务可能允许超出令牌生存期最多五分钟。 <br><br> **SAML 值示例**： <br> `<Conditions`<br>`NotBefore="2013-03-18T21:32:51.261Z"`<br>`NotOnOrAfter="2013-03-18T22:32:51.261Z"`<br>`>` <br><br> **JWT 值示例**： <br> `"nbf":1363289634, "exp":1363293234` |
| `upn` |用户主体名称 |存储用户主体的用户名。<br><br> **JWT 值示例**： <br> `"upn": frankm@contoso.com` |
| `ver` |版本 |存储令牌的版本号。 <br><br> **JWT 值示例**： <br> `"ver": "1.0"` |

## <a name="access-tokens"></a>访问令牌
身份验证成功后，Azure AD 会返回访问令牌，用于访问受保护的资源。 访问令牌是 base64 编码的 JSON Web 令牌 (JWT)，其内容可以通过解码器运行以进行检查。

如果应用仅*使用*访问令牌来获取对 API 的访问权限，则可（而且应）将访问令牌视为完全不透明 - 它只是在 HTTP 请求中应用可传递至资源的字符串。

请求访问令牌时，Azure AD 还针对应用的使用返回一些有关访问令牌的元数据。  此信息包括访问令牌的到期时间及其有效范围。  这可让应用程序执行访问令牌的智能缓存，而无需分析访问令牌本身。

如果应用是通过 Azure AD 保护的 API，需要 HTTP 请求中的访问令牌，则应验证和检查接收到的令牌。 应用应先对访问令牌执行验证，再用其访问资源。 有关验证的详细信息，请参阅[验证令牌](#validating-tokens)。  
有关如何使用 .NET 执行此操作的详细信息，请参阅[使用 Azure AD 的持有者令牌保护 Web API](active-directory-devquickstarts-webapi-dotnet.md)。

## <a name="refresh-tokens"></a>刷新令牌

刷新令牌是应用可用于在 OAuth 2.0 流中获取新访问令牌的安全令牌。  它可让应用表示用户长期访问资源，而无需用户交互。

刷新令牌属于多资源令牌。  也就是说，在一个资源的令牌请求期间收到的刷新令牌可以兑换完全不同资源的访问令牌。 为此，请在目标资源的请求中设置 `resource` 参数。

刷新令牌对应用完全不透明。 它们属于长效令牌，但你不应将应用编写成预期刷新令牌将持续任何一段时间。  刷新令牌可能由于各种原因而随时失效。  让应用知道刷新令牌是否有效的唯一方式就是对 Azure AD 令牌终结点发出令牌请求以尝试兑换刷新令牌。

使用刷新令牌兑换新的访问令牌时，会在令牌响应中收到新的刷新令牌。  应该保存新颁发的刷新令牌，并替换请求中使用的刷新令牌。  这会保证刷新令牌尽可能长期保持有效。

## <a name="validating-tokens"></a>验证令牌

为了验证 id_token 或 access_token，应用应验证该令牌的签名和声明。 要验证访问令牌，应用还应验证颁发者、目标受众和签名令牌。 这些需要根据 OpenID 发现文档中的值进行验证。 例如，文档的租户独立版本位于 [https://login.microsoftonline.com/common/.well-known/openid-configuration](https://login.microsoftonline.com/common/.well-known/openid-configuration)。 Azure AD 中间件具有验证访问令牌的内置功能，可以浏览我们的[示例](https://docs.microsoft.com/azure/active-directory/active-directory-code-samples)，以所选语言进行查找。 有关如何显式验证 JWT 令牌的详细信息，请参阅[手动 JWT 验证示例](https://github.com/Azure-Samples/active-directory-dotnet-webapi-manual-jwt-validation)。  

我们提供了库和代码示例（用于演示如何轻松处理令牌验证），想要了解基本过程的用户可以参阅以下信息。  另外还提供了多个适用于 JWT 验证的第三方开放源代码库 - 几乎每个平台和语言都至少有一个选项。 有关 Azure AD 身份验证库和代码示例的详细信息，请参阅 [Azure AD 身份验证库](active-directory-authentication-libraries.md)。

#### <a name="validating-the-signature"></a>验证签名

JWT 包含三个段（以 `.` 字符分隔）。  第一个段称为**标头**，第二个称为**主体**，第三个称为**签名**。  签名段可用于验证令牌的真实性，使应用信任它。

使用行业标准非对称加密算法（如 RSA 256）对 Azure AD 颁发的令牌进行签名。 JWT 标头包含有关用于对令牌进行签名的密钥和加密方法的信息：

```
{
  "typ": "JWT",
  "alg": "RS256",
  "x5t": "kriMPdmBvx68skT8-mPAB3BseeA"
}
```

`alg` 声明表示用于对令牌进行签名的算法，而 `x5t` 声明表示用于对令牌进行签名的特定公钥。

在任何给定时间点，Azure AD 可以使用一组特定公钥 - 私钥对中的一个对来签名 id_token。 Azure AD 定期换用一组可能的密钥，因此应将应用编写成自动处理这些密钥更改。  检查 Azure AD 所用公钥的更新的合理频率大约为每 24 小时一次。

可以使用位于以下位置的 OpenID Connect 元数据文档来获取验证签名所需的签名密钥数据：

```
https://login.microsoftonline.com/common/.well-known/openid-configuration
```

> [!TIP]
> 在浏览器中尝试打开此 URL！
> 
> 

此元数据文档是一个 JSON 对象，包含一些有用的信息，例如执行 OpenID Connect 身份验证所需的各种终结点的位置。  

它还包含 `jwks_uri`，其提供用于对令牌进行签名的公钥集的位置。  位于 `jwks_uri` 的 JSON 文档包含在该特定时间点使用的所有公钥信息。  应用可以使用 JWT 标头中的 `kid` 声明选择本文档中已用于对特定令牌进行签名的公钥。  然后可以使用正确的公钥和指定的算法来执行签名验证。

执行签名验证超出了本文档的范围 - 有许多开放源代码库可帮助这么做（如有必要）。

#### <a name="validating-the-claims"></a>验证声明

当应用接收到令牌（用户登录时的 id_token，或 HTTP 请求中作为持有者令牌的访问令牌）后，还应对令牌中的声明执行一些检查。  这些检查包括但不限于：

* **受众**声明 - 验证是否计划将令牌提供给应用。
* **不早于**和**过期时间**声明 - 验证令牌是否未过期。
* **颁发者**声明 - 验证令牌是否确实由 Azure AD 颁发给应用。
* **Nonce** - 缓和令牌重播攻击。
* 等等...

有关应用对 ID 令牌应该执行的声明验证的完整列表，请参阅 [OpenID Connect](http://openid.net/specs/openid-connect-core-1_0.html#IDTokenValidation) 规范。 这些声明的预期值详细信息包含在前面的 [id_token](#id-tokens) 部分中。

## <a name="sample-tokens"></a>示例令牌

本部分显示 Azure AD 返回的 SAML 和 JWT 令牌的示例。 这些示例让你查看上下文中的声明。

### <a name="saml-token"></a>SAML 令牌

这是典型 SAML 令牌的一个示例。

    <?xml version="1.0" encoding="UTF-8"?>
    <t:RequestSecurityTokenResponse xmlns:t="http://schemas.xmlsoap.org/ws/2005/02/trust">
      <t:Lifetime>
        <wsu:Created xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">2014-12-24T05:15:47.060Z</wsu:Created>
        <wsu:Expires xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">2014-12-24T06:15:47.060Z</wsu:Expires>
      </t:Lifetime>
      <wsp:AppliesTo xmlns:wsp="http://schemas.xmlsoap.org/ws/2004/09/policy">
        <EndpointReference xmlns="http://www.w3.org/2005/08/addressing">
          <Address>https://contoso.onmicrosoft.com/MyWebApp</Address>
        </EndpointReference>
      </wsp:AppliesTo>
      <t:RequestedSecurityToken>
        <Assertion xmlns="urn:oasis:names:tc:SAML:2.0:assertion" ID="_3ef08993-846b-41de-99df-b7f3ff77671b" IssueInstant="2014-12-24T05:20:47.060Z" Version="2.0">
          <Issuer>https://sts.windows.net/b9411234-09af-49c2-b0c3-653adc1f376e/</Issuer>
          <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
            <ds:SignedInfo>
              <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
              <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256" />
              <ds:Reference URI="#_3ef08993-846b-41de-99df-b7f3ff77671b">
                <ds:Transforms>
                  <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature" />
                  <ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
                </ds:Transforms>
                <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256" />
                <ds:DigestValue>cV1J580U1pD24hEyGuAxrbtgROVyghCqI32UkER/nDY=</ds:DigestValue>
              </ds:Reference>
            </ds:SignedInfo>
            <ds:SignatureValue>j+zPf6mti8Rq4Kyw2NU2nnu0pbJU1z5bR/zDaKaO7FCTdmjUzAvIVfF8pspVR6CbzcYM3HOAmLhuWmBkAAk6qQUBmKsw+XlmF/pB/ivJFdgZSLrtlBs1P/WBV3t04x6fRW4FcIDzh8KhctzJZfS5wGCfYw95er7WJxJi0nU41d7j5HRDidBoXgP755jQu2ZER7wOYZr6ff+ha+/Aj3UMw+8ZtC+WCJC3yyENHDAnp2RfgdElJal68enn668fk8pBDjKDGzbNBO6qBgFPaBT65YvE/tkEmrUxdWkmUKv3y7JWzUYNMD9oUlut93UTyTAIGOs5fvP9ZfK2vNeMVJW7Xg==</ds:SignatureValue>
            <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
              <X509Data>
                <X509Certificate>MIIDPjCCAabcAwIBAgIQsRiM0jheFZhKk49YD0SK1TAJBgUrDgMCHQUAMC0xKzApBgNVBAMTImFjY291bnRzLmFjY2Vzc2NvbnRyb2wud2luZG93cy5uZXQwHhcNMTQwMTAxMDcwMDAwWhcNMTYwMTAxMDcwMDAwWjAtMSswKQYDVQQDEyJhY2NvdW50cy5hY2Nlc3Njb250cm9sLndpbmRvd3MubmV0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAkSCWg6q9iYxvJE2NIhSyOiKvqoWCO2GFipgH0sTSAs5FalHQosk9ZNTztX0ywS/AHsBeQPqYygfYVJL6/EgzVuwRk5txr9e3n1uml94fLyq/AXbwo9yAduf4dCHTP8CWR1dnDR+Qnz/4PYlWVEuuHHONOw/blbfdMjhY+C/BYM2E3pRxbohBb3x//CfueV7ddz2LYiH3wjz0QS/7kjPiNCsXcNyKQEOTkbHFi3mu0u13SQwNddhcynd/GTgWN8A+6SN1r4hzpjFKFLbZnBt77ACSiYx+IHK4Mp+NaVEi5wQtSsjQtI++XsokxRDqYLwus1I1SihgbV/STTg5enufuwIDAQABo2IwYDBeBgNVHQEEVzBVgBDLebM6bK3BjWGqIBrBNFeNoS8wLTErMCkGA1UEAxMiYWNjb3VudHMuYWNjZXNzY29udHJvbC53aW5kb3dzLm5ldIIQsRiM0jheFZhKk49YD0SK1TAJBgUrDgMCHQUAA4IBAQCJ4JApryF77EKC4zF5bUaBLQHQ1PNtA1uMDbdNVGKCmSp8M65b8h0NwlIjGGGy/unK8P6jWFdm5IlZ0YPTOgzcRZguXDPj7ajyvlVEQ2K2ICvTYiRQqrOhEhZMSSZsTKXFVwNfW6ADDkN3bvVOVbtpty+nBY5UqnI7xbcoHLZ4wYD251uj5+lo13YLnsVrmQ16NCBYq2nQFNPuNJw6t3XUbwBHXpF46aLT1/eGf/7Xx6iy8yPJX4DyrpFTutDz882RWofGEO5t4Cw+zZg70dJ/hH/ODYRMorfXEW+8uKmXMKmX2wyxMKvfiPbTy5LmAU8Jvjs2tLg4rOBcXWLAIarZ</X509Certificate>
              </X509Data>
            </KeyInfo>
          </ds:Signature>
          <Subject>
            <NameID Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">m_H3naDei2LNxUmEcWd0BZlNi_jVET1pMLR6iQSuYmo</NameID>
            <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer" />
          </Subject>
          <Conditions NotBefore="2014-12-24T05:15:47.060Z" NotOnOrAfter="2014-12-24T06:15:47.060Z">
            <AudienceRestriction>
              <Audience>https://contoso.onmicrosoft.com/MyWebApp</Audience>
            </AudienceRestriction>
          </Conditions>
          <AttributeStatement>
            <Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">
              <AttributeValue>a1addde8-e4f9-4571-ad93-3059e3750d23</AttributeValue>
            </Attribute>
            <Attribute Name="http://schemas.microsoft.com/identity/claims/tenantid">
              <AttributeValue>b9411234-09af-49c2-b0c3-653adc1f376e</AttributeValue>
            </Attribute>
            <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
              <AttributeValue>sample.admin@contoso.onmicrosoft.com</AttributeValue>
            </Attribute>
            <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname">
              <AttributeValue>Admin</AttributeValue>
            </Attribute>
            <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname">
              <AttributeValue>Sample</AttributeValue>
            </Attribute>
            <Attribute Name="http://schemas.microsoft.com/ws/2008/06/identity/claims/groups">
              <AttributeValue>5581e43f-6096-41d4-8ffa-04e560bab39d</AttributeValue>
              <AttributeValue>07dd8a89-bf6d-4e81-8844-230b77145381</AttributeValue>
              <AttributeValue>0e129f4g-6b0a-4944-982d-f776000632af</AttributeValue>
              <AttributeValue>3ee07328-52ef-4739-a89b-109708c22fb5</AttributeValue>
              <AttributeValue>329k14b3-1851-4b94-947f-9a4dacb595f4</AttributeValue>
              <AttributeValue>6e32c650-9b0a-4491-b429-6c60d2ca9a42</AttributeValue>
              <AttributeValue>f3a169a7-9a58-4e8f-9d47-b70029v07424</AttributeValue>
              <AttributeValue>8e2c86b2-b1ad-476d-9574-544d155aa6ff</AttributeValue>
              <AttributeValue>1bf80264-ff24-4866-b22c-6212e5b9a847</AttributeValue>
              <AttributeValue>4075f9c3-072d-4c32-b542-03e6bc678f3e</AttributeValue>
              <AttributeValue>76f80527-f2cd-46f4-8c52-8jvd8bc749b1</AttributeValue>
              <AttributeValue>0ba31460-44d0-42b5-b90c-47b3fcc48e35</AttributeValue>
              <AttributeValue>edd41703-8652-4948-94a7-2d917bba7667</AttributeValue>
            </Attribute>
            <Attribute Name="http://schemas.microsoft.com/identity/claims/identityprovider">
              <AttributeValue>https://sts.windows.net/b9411234-09af-49c2-b0c3-653adc1f376e/</AttributeValue>
            </Attribute>
          </AttributeStatement>
          <AuthnStatement AuthnInstant="2014-12-23T18:51:11.000Z">
            <AuthnContext>
              <AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:Password</AuthnContextClassRef>
            </AuthnContext>
          </AuthnStatement>
        </Assertion>
      </t:RequestedSecurityToken>
      <t:RequestedAttachedReference>
        <SecurityTokenReference xmlns="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns:d3p1="http://docs.oasis-open.org/wss/oasis-wss-wssecurity-secext-1.1.xsd" d3p1:TokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV2.0">
          <KeyIdentifier ValueType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLID">_3ef08993-846b-41de-99df-b7f3ff77671b</KeyIdentifier>
        </SecurityTokenReference>
      </t:RequestedAttachedReference>
      <t:RequestedUnattachedReference>
        <SecurityTokenReference xmlns="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns:d3p1="http://docs.oasis-open.org/wss/oasis-wss-wssecurity-secext-1.1.xsd" d3p1:TokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV2.0">
          <KeyIdentifier ValueType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLID">_3ef08993-846b-41de-99df-b7f3ff77671b</KeyIdentifier>
        </SecurityTokenReference>
      </t:RequestedUnattachedReference>
      <t:TokenType>http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV2.0</t:TokenType>
      <t:RequestType>http://schemas.xmlsoap.org/ws/2005/02/trust/Issue</t:RequestType>
      <t:KeyType>http://schemas.xmlsoap.org/ws/2005/05/identity/NoProofKey</t:KeyType>
    </t:RequestSecurityTokenResponse>

### <a name="jwt-token---user-impersonation"></a>JWT 令牌 - 用户模拟
这是在授权代码授予流程中使用的典型 JSON Web 令牌 (JWT) 的一个示例。
除了声明外，该令牌还包括 **ver** 和 **appidacr** 中的版本号，即身份验证上下文类引用，它指示如何对客户端进行身份验证。 对于公共客户端，该值为 0。 如果使用客户端 ID 或客户端机密，则该值为 1。

    {
     typ: "JWT",
     alg: "RS256",
     x5t: "kriMPdmBvx68skT8-mPAB3BseeA"
    }.
    {
     aud: "https://contoso.onmicrosoft.com/scratchservice",
     iss: "https://sts.windows.net/b9411234-09af-49c2-b0c3-653adc1f376e/",
     iat: 1416968588,
     nbf: 1416968588,
     exp: 1416972488,
     ver: "1.0",
     tid: "b9411234-09af-49c2-b0c3-653adc1f376e",
     amr: [
      "pwd"
     ],
     roles: [
      "Admin"
     ],
     oid: "6526e123-0ff9-4fec-ae64-a8d5a77cf287",
     upn: "sample.user@contoso.onmicrosoft.com",
     unique_name: "sample.user@contoso.onmicrosoft.com",
     sub: "yf8C5e_VRkR1egGxJSDt5_olDFay6L5ilBA81hZhQEI",
     family_name: "User",
     given_name: "Sample",
     groups: [
      "0e129f6b-6b0a-4944-982d-f776000632af",
      "323b13b3-1851-4b94-947f-9a4dacb595f4",
      "6e32c250-9b0a-4491-b429-6c60d2ca9a42",
      "f3a161a7-9a58-4e8f-9d47-b70022a07424",
      "8d4c81b2-b1ad-476d-9574-544d155aa6ff",
      "1bf80164-ff24-4866-b19c-6212e5b9a847",
      "76f80127-f2cd-46f4-8c52-8edd8bc749b1",
      "0ba27160-44d0-42b5-b90c-47b3fcc48e35"
     ],
     appid: "b075ddef-0efa-123b-997b-de1337c29185",
     appidacr: "1",
     scp: "user_impersonation",
     acr: "1"
    }.

## <a name="related-content"></a>相关内容
* 请参阅 Azure AD Graph [策略操作](https://msdn.microsoft.com/library/azure/ad/graph/api/policy-operations)和[策略实体](https://msdn.microsoft.com/library/azure/ad/graph/api/entity-and-complex-type-reference#policy-entity)以了解有关通过 Azure AD Graph API 管理令牌生存期策略的详细信息。
* 有关通过 PowerShell cmdlet 管理策略的详细信息和示例，请参阅 [Azure AD 中的可配置令牌生存期](../active-directory-configurable-token-lifetimes.md)。 
