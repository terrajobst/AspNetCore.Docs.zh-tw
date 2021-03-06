---
title: ASP.NET Core 中的連結標記協助程式
author: rick-anderson
ms.author: riande
description: 探索 ASP.NET Core 連結標記協助程式屬性，以及每個屬性在 HTML 連結標記的擴充行為中所扮演的角色。
ms.custom: mvc
ms.date: 09/24/2019
uid: mvc/views/tag-helpers/builtin-th/link-tag-helper
ms.openlocfilehash: d7514433bee8a138cd7d75bfd15c9798d4fd31a3
ms.sourcegitcommit: 9a129f5f3e31cc449742b164d5004894bfca90aa
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/06/2020
ms.locfileid: "78662726"
---
# <a name="link-tag-helper-in-aspnet-core"></a>ASP.NET Core 中的連結標記協助程式

由 [Rick Anderson](https://twitter.com/RickAndMSFT) 提供

[連結](xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper)標籤協助程式會產生主要或切換回 CSS 檔案的連結。 主要 CSS 檔案通常位於[內容傳遞網路](/office365/enterprise/content-delivery-networks#what-exactly-is-a-cdn)（CDN）上。

[!INCLUDE[](~/includes/cdn.md)]

連結標籤協助程式可讓您指定 CSS 檔案的 CDN，以及 CDN 無法使用時的回復。 連結標籤協助程式可提供 CDN 的效能優勢，以及本機裝載的穩定性。

下列 Razor 標記會顯示使用 ASP.NET Core web 應用程式範本所建立之配置檔案的 `head` 元素：

[!code-html[](link-tag-helper/sample/_Layout.cshtml?name=snippet)]

下列是上述程式碼（在非開發環境中）所呈現的 HTML：

[!code-csharp[](link-tag-helper/sample/HtmlPage1.html)]

在上述程式碼中，連結標記協助程式會產生 `<meta name="x-stylesheet-fallback-test" content="" class="sr-only" />` 專案和下列 JavaScript，用來驗證要求的*啟動程式。最小的 .css*檔案可在 CDN 上取得。 在此情況下，CSS 檔案可供使用，因此標記協助程式會使用 CDN CSS 檔案來產生 `<link />` 元素。

## <a name="commonly-used-link-tag-helper-attributes"></a>常用的連結標記協助程式屬性

請參閱[連結](xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper)標記協助程式，以取得所有的連結標籤 helper 屬性、屬性和方法。

### <a name="href"></a>href

連結資源的慣用位址。 在所有情況下，會將位址視為產生的 HTML。

### <a name="asp-fallback-href"></a>asp-fallback-href

當主要 URL 失敗時，要回復的 CSS 樣式表單 URL。

### <a name="asp-fallback-test-class"></a>asp-fallback-測試類別

在樣式表單中定義用於回溯測試的類別名稱。 如需詳細資訊，請參閱 <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestClass>。

### <a name="asp-fallback-test-property"></a>asp-fallback-測試-屬性

用於回退測試的 CSS 屬性名稱。 如需詳細資訊，請參閱 <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestProperty>。

### <a name="asp-fallback-test-value"></a>asp-fallback-測試-值

要用於 fallback 測試的 CSS 屬性值。 如需詳細資訊，請參閱 <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestValue>。

## <a name="additional-resources"></a>其他資源

* <xref:mvc/views/tag-helpers/intro>
* <xref:mvc/controllers/areas>
* <xref:razor-pages/index>
* <xref:mvc/compatibility-version>
