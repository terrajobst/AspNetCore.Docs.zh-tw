---
title: ASP.NET Core Blazor 裝載模型設定
author: guardrex
description: 深入瞭解 Blazor 裝載模型設定，包括如何將 Razor 元件整合到 Razor Pages 和 MVC 應用程式中。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/24/2020
no-loc:
- Blazor
- SignalR
uid: blazor/hosting-model-configuration
ms.openlocfilehash: 1f71ac63bbe9dc9d56cfca2ded19a5b863be828f
ms.sourcegitcommit: 6ffb583991d6689326605a24565130083a28ef85
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/26/2020
ms.locfileid: "80306426"
---
# <a name="aspnet-core-blazor-hosting-model-configuration"></a>ASP.NET Core Blazor 裝載模型設定

依[Daniel Roth](https://github.com/danroth27)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

本文涵蓋主控模型設定。

## <a name="blazor-webassembly"></a>Blazor WebAssembly

從 ASP.NET Core 3.2 Preview 3 版本開始，Blazor WebAssembly 支援從下列設定：

* *wwwroot/appsettings. json*
* *wwwroot/appsettings。{環境}. json*

在 Blazor 裝載的應用程式中，[執行時間環境](xref:fundamentals/environments)與伺服器應用程式的值相同。

在本機執行應用程式時，環境會預設為開發。 當應用程式發佈時，環境會預設為 [生產]。 如需詳細資訊，包括如何設定環境，請參閱 <xref:fundamentals/environments>。

> [!WARNING]
> 使用者可以看到 Blazor WebAssembly 應用程式中的設定。 **請勿在設定中儲存應用程式秘密或認證。**

系統會快取設定檔以供離線使用。 使用[漸進式 Web 應用程式（pwa）](xref:blazor/progressive-web-app)，您只能在建立新的部署時更新設定檔。 在部署之間編輯設定檔沒有任何作用，因為：

* 使用者有檔案的快取版本，這些檔案會繼續使用。
* PWA 的*service-worker*和*service-worker-assets*檔案必須在編譯時重建，在使用者下一次線上流覽應用程式時，該應用程式會發出通知。

如需 Pwa 如何處理背景更新的詳細資訊，請參閱 <xref:blazor/progressive-web-app#background-updates>。

## <a name="blazor-server"></a>Blazor 伺服器

### <a name="reflect-the-connection-state-in-the-ui"></a>反映 UI 中的連接狀態

當用戶端偵測到連線已遺失時，會在用戶端嘗試重新連線時，向使用者顯示預設的 UI。 如果重新連線失敗，則會提供使用者重試的選項。

若要自訂 UI，請在 *_Host*的 [Razor 頁面] `<body>` 中，定義具有 `components-reconnect-modal` `id` 的元素：

```cshtml
<div id="components-reconnect-modal">
    ...
</div>
```

下表描述套用至 `components-reconnect-modal` 元素的 CSS 類別。

| CSS 類別                       | 表示&hellip; |
| ------------------------------- | ----------------- |
| `components-reconnect-show`     | 遺失的連接。 用戶端正在嘗試重新連線。 顯示強制回應。 |
| `components-reconnect-hide`     | 系統會將使用中的連接重新建立至伺服器。 隱藏強制回應。 |
| `components-reconnect-failed`   | 重新連接失敗，可能是因為網路失敗。 若要嘗試重新連接，請呼叫 `window.Blazor.reconnect()`。 |
| `components-reconnect-rejected` | 已拒絕重新連接。 已達到伺服器但拒絕連線，而且伺服器上的使用者狀態遺失。 若要重載應用程式，請呼叫 `location.reload()`。 當下列情況發生時，可能會產生此連接狀態：<ul><li>伺服器端線路發生損毀。</li><li>用戶端中斷連線的時間夠長，伺服器才會捨棄使用者的狀態。 系統會處置與使用者互動之元件的實例。</li><li>伺服器會重新開機，或回收應用程式的背景工作進程。</li></ul> |

### <a name="render-mode"></a>轉譯模式

在建立伺服器的用戶端連接之前，預設會設定 Blazor 伺服器應用程式，以預先呈現伺服器上的 UI。 這會在 *_Host. cshtml* Razor 頁面中設定：

```cshtml
<body>
    <app>
      <component type="typeof(App)" render-mode="ServerPrerendered" />
    </app>

    <script src="_framework/blazor.server.js"></script>
</body>
```

`RenderMode` 設定元件是否：

* 會資源清單到頁面中。
* 會在頁面上轉譯為靜態 HTML，或包含從使用者代理程式啟動 Blazor 應用程式所需的資訊。

| `RenderMode`        | 描述 |
| ------------------- | ----------- |
| `ServerPrerendered` | 將元件轉譯為靜態 HTML，並包含 Blazor 伺服器應用程式的標記。 當使用者代理程式啟動時，會使用此標記來啟動 Blazor 應用程式。 |
| `Server`            | 呈現 Blazor 伺服器應用程式的標記。 不包含來自元件的輸出。 當使用者代理程式啟動時，會使用此標記來啟動 Blazor 應用程式。 |
| `Static`            | 將元件轉譯為靜態 HTML。 |

不支援從靜態 HTML 網頁轉譯伺服器元件。

### <a name="render-stateful-interactive-components-from-razor-pages-and-views"></a>從 Razor 頁面和 views 轉譯具狀態的互動式元件

可設定狀態的互動式元件可以新增至 Razor 頁面或視圖。

當頁面或視圖呈現時：

* 此元件是使用頁面或視圖所資源清單。
* 用來進行預呈現的初始元件狀態會遺失。
* 建立 SignalR 連接時，會建立新的元件狀態。

下列 Razor 頁面會呈現 `Counter` 元件：

```cshtml
<h1>My Razor Page</h1>

<component type="typeof(Counter)" render-mode="ServerPrerendered" 
    param-InitialValue="InitialValue" />

@code {
    [BindProperty(SupportsGet=true)]
    public int InitialValue { get; set; }
}
```

### <a name="render-noninteractive-components-from-razor-pages-and-views"></a>從 Razor 頁面和 views 轉譯非互動式元件

在下列 Razor 頁面中，`Counter` 元件會以靜態方式轉譯，並使用以表單指定的初始值：

```cshtml
<h1>My Razor Page</h1>

<form>
    <input type="number" asp-for="InitialValue" />
    <button type="submit">Set initial value</button>
</form>

<component type="typeof(Counter)" render-mode="Static" 
    param-InitialValue="InitialValue" />

@code {
    [BindProperty(SupportsGet=true)]
    public int InitialValue { get; set; }
}
```

由於 `MyComponent` 是以靜態方式呈現，因此元件不能是互動式的。

### <a name="configure-the-opno-locsignalr-client-for-opno-locblazor-server-apps"></a>設定 Blazor 伺服器應用程式的 SignalR 用戶端

有時候，您需要設定 Blazor 伺服器應用程式所使用的 SignalR 用戶端。 例如，您可能會想要在 SignalR 用戶端上設定記錄，以診斷連接問題。

若要設定*Pages/_Host. cshtml*檔案中的 SignalR 用戶端：

* 將 `autostart="false"` 屬性加入至 `blazor.server.js` 腳本的 `<script>` 標記。
* 呼叫 `Blazor.start` 並傳入指定 SignalR 產生器的設定物件。

```html
<script src="_framework/blazor.server.js" autostart="false"></script>
<script>
  Blazor.start({
    configureSignalR: function (builder) {
      builder.configureLogging("information"); // LogLevel.Information
    }
  });
</script>
```
