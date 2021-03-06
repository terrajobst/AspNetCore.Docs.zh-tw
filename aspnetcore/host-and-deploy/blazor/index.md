---
title: 裝載和部署 ASP.NET Core Blazor
author: guardrex
description: 探索如何裝載和部署 Blazor 應用程式。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/11/2020
no-loc:
- Blazor
- SignalR
uid: host-and-deploy/blazor/index
ms.openlocfilehash: ddf70da29a82d462422c1bdf74ff45b92bb10b56
ms.sourcegitcommit: 5bdc54162d7dea8d9fa54ac3055678db23586af1
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/17/2020
ms.locfileid: "79434261"
---
# <a name="host-and-deploy-aspnet-core-blazor"></a>裝載及部署 ASP.NET Core Blazor

作者：[Luke Latham](https://github.com/guardrex)、[Rainer Stropek](https://www.timecockpit.com) 和 [Daniel Roth](https://github.com/danroth27)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

## <a name="publish-the-app"></a>發佈應用程式

應用程式會在發行組態中發佈以供部署。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. 從巡覽列中選取 [建置] > [發佈 {應用程式}]。
1. 選取「發佈目標」。 若要在本機發佈，請選取 [資料夾]。
1. 接受 [選擇資料夾] 欄位中的預設位置，或指定不同的位置。 選取 [發佈] 按鈕。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

使用 [dotnet publish](/dotnet/core/tools/dotnet-publish) 命令，搭配發行組態來發佈應用程式：

```dotnetcli
dotnet publish -c Release
```

---

發佈應用程式會觸發專案相依性的[還原](/dotnet/core/tools/dotnet-restore)並[建置](/dotnet/core/tools/dotnet-build)專案，然後才會建立資產以供部署。 在建置程序中，會移除未使用的方法和組件，減少應用程式的下載大小和載入時間。

發行位置：

* Blazor WebAssembly
  * 獨立 &ndash; 應用程式會發佈至 */BIN/RELEASE/{TARGET FRAMEWORK}/publish/wwwroot*資料夾中。 若要將應用程式部署為靜態網站，請將*wwwroot*資料夾的內容複寫到靜態網站主機。
  * 裝載 &ndash; 用戶端 Blazor WebAssembly 應用程式會發佈到伺服器應用程式的 */BIN/RELEASE/{TARGET FRAMEWORK}/publish/wwwroot*資料夾，以及伺服器應用程式的任何其他靜態 web 資產。 將*publish*資料夾的內容部署至主機。
* 應用程式 &ndash; Blazor 伺服器會發佈到 */BIN/RELEASE/{TARGET FRAMEWORK}/publish*資料夾中。 將*publish*資料夾的內容部署至主機。

該資料夾中的資產均會部署至 Web 伺服器。 部署可能是手動或自動的程序，視使用中的開發工具而定。

## <a name="app-base-path"></a>應用程式基底路徑

*應用程式基底路徑*是應用程式的根 URL 路徑。 請考慮下列 ASP.NET Core 應用程式和 Blazor 子應用程式：

* ASP.NET Core 應用程式的名稱 `MyApp`：
  * 應用程式實際位於*d：/MyApp*。
  * `https://www.contoso.com/{MYAPP RESOURCE}`收到要求。
* 名為 `CoolApp` Blazor 應用程式是 `MyApp`的子應用程式：
  * 子應用程式實際上位於*d：/MyApp/CoolApp*。
  * `https://www.contoso.com/CoolApp/{COOLAPP RESOURCE}`收到要求。

若未指定 `CoolApp`的額外設定，此案例中的子應用程式就不會知道其位於伺服器上的位置。 例如，應用程式無法在不知道其位於相對 URL 路徑 `/CoolApp/`的情況下，為其資源建立正確的相對 Url。

若要為 Blazor 應用程式的基底路徑 `https://www.contoso.com/CoolApp/`提供設定，`<base>` 標記的 `href` 屬性會設為*Pages/_Host. cshtml*檔案（Blazor 伺服器）或*wwwroot/index.html*檔（Blazor WebAssembly）中的相對根路徑：

```html
<base href="/CoolApp/">
```

Blazor 伺服器應用程式會在 `Startup.Configure`的應用程式要求管線中呼叫 <xref:Microsoft.AspNetCore.Builder.UsePathBaseExtensions.UsePathBase*>，以額外設定伺服器端基底路徑：

```csharp
app.UsePathBase("/CoolApp");
```

藉由提供相對的 URL 路徑，不在根目錄中的元件可以針對應用程式的根路徑來建立 Url。 不同目錄結構層級的元件可以在整個應用程式的位置建立其他資源的連結。 應用程式基底路徑也會用來攔截選取的超連結，其中連結的 `href` 目標是在應用程式基底路徑 URI 空間內。 Blazor 路由器會處理內部導覽。

在許多裝載案例中，應用程式的相對 URL 路徑是應用程式的根目錄。 在這些情況下，應用程式的相對 URL 基底路徑是正斜線（`<base href="/" />`），這是 Blazor 應用程式的預設設定。 在其他裝載案例（例如 GitHub 頁面和 IIS 子應用程式）中，應用程式基底路徑必須設定為應用程式的伺服器相對 URL 路徑。

若要設定應用程式的基底路徑，請更新*Pages/_Host. cshtml*檔案（Blazor 伺服器）或*wwwroot/index.html*檔（Blazor WebAssembly）的 `<head>` 標記元素內的 `<base>` 標記。 將 `href` 屬性值設定為 `/{RELATIVE URL PATH}/` （需要尾端斜線），其中 `{RELATIVE URL PATH}` 是應用程式的完整相對 URL 路徑。

針對具有非根相對 URL 路徑的 Blazor WebAssembly 應用程式（例如 `<base href="/CoolApp/">`），應用程式*在本機執行時*，無法找到其資源。 若要在本機開發和測試期間解決這個問題，您可以提供「基底路徑」引數，讓它在執行時符合 `href` 標籤的 `<base>` 值。 不要包含尾端斜線。 若要在本機執行應用程式時傳遞 path base 引數，請使用 `--pathbase` 選項，從應用程式的目錄執行 `dotnet run` 命令：

```dotnetcli
dotnet run --pathbase=/{RELATIVE URL PATH (no trailing slash)}
```

針對具有 `/CoolApp/` （`<base href="/CoolApp/">`）之相對 URL 路徑的 Blazor WebAssembly 應用程式，命令為：

```dotnetcli
dotnet run --pathbase=/CoolApp
```

Blazor WebAssembly 應用程式會在 `http://localhost:port/CoolApp`本機回應。

## <a name="deployment"></a>部署

如需部署指導方針，請參閱下列主題：

* <xref:host-and-deploy/blazor/webassembly>
* <xref:host-and-deploy/blazor/server>
