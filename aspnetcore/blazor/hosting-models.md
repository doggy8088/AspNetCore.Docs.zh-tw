---
title: ASP.NET Core Blazor 裝載模型
author: guardrex
description: 瞭解 Blazor WebAssembly 和 Blazor 伺服器裝載模型。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/03/2019
uid: blazor/hosting-models
ms.openlocfilehash: bc3ad9c7c4731b685fc161844d9f55e51722c0ea
ms.sourcegitcommit: 73e255e846e414821b8cc20ffa3aec946735cd4e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2019
ms.locfileid: "71924667"
---
# <a name="aspnet-core-blazor-hosting-models"></a>ASP.NET Core Blazor 裝載模型

依[Daniel Roth](https://github.com/danroth27)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

Blazor 是一種 web 架構，設計用來在瀏覽器中以[WebAssembly](https://webassembly.org/)為基礎的 .net 執行時間（*Blazor WebAssembly*）或伺服器端在 ASP.NET Core （*Blazor server*）中執行用戶端。 無論裝載模型為何，應用程式和元件模型*都相同*。

若要為本文所述的主控模型建立專案，請參閱<xref:blazor/get-started>。

## <a name="blazor-webassembly"></a>Blazor WebAssembly

Blazor 的主要裝載模型是在 WebAssembly 的瀏覽器中執行用戶端。 Blazor 應用程式、其相依性，和 .NET 執行階段會下載至瀏覽器中。 應用程式會直接在瀏覽器 UI 執行緒上執行。 UI 更新和事件處理會在同一個進程中進行。 應用程式的資產會以靜態檔案的形式部署至 web 伺服器或服務，以提供靜態內容給用戶端。

![Blazor WebAssembly：Blazor 應用程式會在瀏覽器內的 UI 執行緒上執行。](hosting-models/_static/blazor-webassembly.png)

若要使用用戶端裝載模型來建立 Blazor 應用程式，請使用**Blazor WebAssembly 應用程式**範本（[dotnet new blazorwasm](/dotnet/core/tools/dotnet-new)）。

選取 [ **Blazor WebAssembly 應用程式**] 範本之後，您可以選擇將應用程式設定為使用 ASP.NET Core 後端，方法是選取 [ **ASP.NET Core**裝載] 核取方塊（[dotnet] [[新增 blazorwasm-](/dotnet/core/tools/dotnet-new)裝載）]。 ASP.NET Core 應用程式會將 Blazor 應用程式提供給用戶端。 Blazor WebAssembly 應用程式可以使用 Web API 呼叫或[SignalR](xref:signalr/introduction)，透過網路與伺服器互動。

這些範本包含處理下列內容的*blazor. webassembly*腳本：

* 下載 .NET 執行時間、應用程式和應用程式的相依性。
* 初始化執行時間以執行應用程式。

Blazor WebAssembly 裝載模型提供數個優點：

* 沒有 .NET 伺服器端相依性。 應用程式會在下載至用戶端之後完全正常運作。
* 用戶端資源和功能都可以充分運用。
* 工作會從伺服器卸載至用戶端。
* 不需要 ASP.NET Core web 伺服器來裝載應用程式。 無伺服器部署案例可行（例如，從 CDN 提供應用程式）。

Blazor WebAssembly 裝載有缺點：

* 應用程式僅限於瀏覽器的功能。
* 需要支援的用戶端硬體和軟體（例如 WebAssembly 支援）。
* 下載大小較大，且應用程式需要較長的時間來載入。
* .NET 執行時間和工具支援較不成熟。 例如， [.NET Standard](/dotnet/standard/net-standard)支援和偵錯工具中有限制。

## <a name="blazor-server"></a>Blazor 伺服器

透過 Blazor 伺服器裝載模型，應用程式會在伺服器上從 ASP.NET Core 應用程式中執行。 UI 更新、事件處理及 JavaScript 呼叫會透過 [SignalR](xref:signalr/introduction) 連線處理。

![瀏覽器會透過 SignalR 連線，與伺服器上的應用程式互動（裝載于 ASP.NET Core 應用程式內）。](hosting-models/_static/blazor-server.png)

若要使用 Blazor 伺服器裝載模型來建立 Blazor 應用程式，請使用 ASP.NET Core **Blazor Server 應用程式**範本（[dotnet new blazorserver](/dotnet/core/tools/dotnet-new)）。 ASP.NET Core 應用程式會裝載 Blazor 伺服器應用程式，並建立用戶端連接的 SignalR 端點。

ASP.NET Core 應用程式會參考要新增`Startup`的應用程式類別：

* 伺服器端服務。
* 應用程式至要求處理管線。

*Blazor*腳本&dagger;會建立用戶端連接。 應用程式會負責保存和還原所需的應用程式狀態（例如，萬一網路連線中斷時）。

Blazor 伺服器裝載模型提供數個優點：

* 下載大小明顯小於 Blazor WebAssembly 應用程式，而應用程式載入速度會更快。
* 應用程式會充分利用伺服器功能，包括使用任何 .NET Core 相容的 Api。
* 伺服器上的 .NET Core 是用來執行應用程式，因此現有的 .NET 工具（例如，「偵測」）會如預期般運作。
* 支援瘦用戶端。 例如，Blazor 伺服器應用程式適用于不支援 WebAssembly 的瀏覽器，以及在資源受限的裝置上。
* 應用程式的 .NET/C#程式碼基底（包括應用程式的元件代碼）不會提供給用戶端。

有 Blazor 伺服器裝載的缺點：

* 通常會有較高的延遲。 每個使用者互動都牽涉到網路躍點。
* 沒有離線支援。 如果用戶端連線失敗，應用程式就會停止運作。
* 對於具有許多使用者的應用程式而言，擴充性是極具挑戰性的。 伺服器必須管理多個用戶端連接並處理用戶端狀態。
* 需要 ASP.NET Core 伺服器才能服務應用程式。 無伺服器部署案例不可行（例如，從 CDN 提供應用程式）。

&dagger;*Blazor*腳本是從 ASP.NET Core 共用架構中的內嵌資源提供。

### <a name="comparison-to-server-rendered-ui"></a>與伺服器呈現的 UI 比較

瞭解 Blazor 伺服器應用程式的其中一種方式，就是了解它與傳統模型有何不同，可以使用 Razor views 或 Razor Pages ASP.NET Core 應用程式中呈現 UI。 這兩種模型都使用 Razor 語言來描述 HTML 內容，但在標記呈現的方式上有很大的差異。

當 Razor 頁面或視圖呈現時，每一行 Razor 程式碼都會以文字格式發出 HTML。 呈現之後，伺服器會處置頁面或 view 實例，包括任何已產生的狀態。 當頁面的另一個要求發生時（例如，當伺服器驗證失敗且顯示驗證摘要時）：

* 整個頁面會再次重新顯示為 HTML 文字。
* 此頁面會傳送至用戶端。

Blazor 應用程式是由可重複使用的 UI 元素（稱為「*元件*」）所組成。 元件包含C#程式碼、標記和其他元件。 呈現元件時，Blazor 會產生類似于 HTML 或 XML 檔物件模型（DOM）的內含元件圖形。 此圖表包含屬性和欄位中保留的元件狀態。 Blazor 會評估元件圖形，以產生標記的二進位標記法。 二進位格式可以是：

* 已轉換成 HTML 文字（在預建期間）。
* 用來在一般轉譯期間有效率地更新標記。

Blazor 中的 UI 更新會由下列觸發：

* 使用者互動，例如選取按鈕。
* 應用程式觸發程式，例如計時器。

圖形為重新顯示，且會計算 UI*差異*（差異）。 這項差異是更新用戶端上的 UI 所需的最小一組 DOM 編輯。 差異會以二進位格式傳送至用戶端，並由瀏覽器套用。

元件會在使用者從用戶端導覽出去之後處置。 當使用者與元件互動時，元件的狀態（服務、資源）必須保留在伺服器的記憶體中。 因為許多元件的狀態可能會由伺服器同時維護，所以記憶體耗盡是必須解決的問題。 如需有關如何撰寫 Blazor 伺服器應用程式以確保最佳使用伺服器記憶體的指引，請參閱<xref:security/blazor/server>。

### <a name="circuits"></a>獲得

Blazor 伺服器應用程式建置於[ASP.NET Core SignalR](xref:signalr/introduction)之上。 每個用戶端會透過一或多個稱為*線路*的 SignalR 連線來與伺服器通訊。 線路是 Blazor 的抽象概念，SignalR 連線可容忍暫時的網路中斷。 當 Blazor 用戶端看到 SignalR 連線已中斷連線時，它會嘗試使用新的 SignalR 連線重新連線到伺服器。

連線至 Blazor 伺服器應用程式的每個瀏覽器畫面（瀏覽器索引標籤或 iframe）都會使用 SignalR 連接。 相較于一般伺服器呈現的應用程式，這還是另一項重要的差異。 在伺服器呈現的應用程式中，在多個瀏覽器畫面中開啟相同的應用程式，通常不會轉譯成伺服器上的其他資源需求。 在 Blazor 伺服器應用程式中，每個瀏覽器畫面都需要個別的線路，且元件狀態的個別實例會由伺服器管理。

Blazor 會考慮關閉瀏覽器索引標籤，或流覽至外部 URL 的*正常*終止。 在正常終止的事件中，會立即釋放線路和相關聯的資源。 用戶端也可能會因為網路中斷而無法正常地中斷連線。 Blazor 伺服器會儲存已中斷連線的線路，以取得可設定的間隔，以允許用戶端重新連線。 如需詳細資訊，請參閱重新[連接到相同的伺服器](#reconnection-to-the-same-server)一節。

### <a name="ui-latency"></a>UI 延遲

UI 延遲是從起始的動作到 UI 更新時間所花費的時間。 較小的 UI 延遲值是應用程式對使用者進行回應的必要行為。 在 Blazor 伺服器應用程式中，會將每個動作傳送至伺服器、進行處理，並傳回 UI 差異。 因此，UI 延遲是網路延遲和處理動作之伺服器延遲的總和。

對於僅限於私人商業網路的企業營運應用程式，通常會 imperceptible 因網路延遲而對使用者的延遲所造成的影響。 對於透過網際網路部署的應用程式，使用者的延遲可能會很明顯，尤其是在地理位置廣泛散佈的使用者時。

記憶體使用量也會導致應用程式延遲。 增加記憶體使用量會導致頻繁的垃圾收集或將記憶體分頁到磁片，這兩者都會降低應用程式效能，因而增加 UI 延遲。 如需詳細資訊，請參閱<xref:security/blazor/server>。

Blazor 伺服器應用程式應該經過優化，藉由減少網路延遲和記憶體使用量，將 UI 延遲降到最低。 如需測量網路延遲的方法， <xref:host-and-deploy/blazor/server#measure-network-latency>請參閱。 如需 SignalR 和 Blazor 的詳細資訊，請參閱：

* <xref:host-and-deploy/blazor/server>
* <xref:security/blazor/server>

### <a name="reconnection-to-the-same-server"></a>重新連接到相同的伺服器

Blazor 伺服器應用程式需要伺服器的作用中 SignalR 連接。 如果連接中斷，應用程式會嘗試重新連線到伺服器。 只要用戶端的狀態仍在記憶體中，用戶端會話就會繼續，而不會失去狀態。

當用戶端偵測到連線已遺失時，會在用戶端嘗試重新連線時，向使用者顯示預設的 UI。 如果重新連線失敗，則會提供使用者重試的選項。 若要自訂 UI，請在 *_Host*的 [Razor] 頁面中，定義具有 `components-reconnect-modal` 的元素做為其 `id`。 用戶端會根據連接的狀態，使用下列其中一個 CSS 類別來更新這個元素：

* `components-reconnect-show` &ndash; 顯示 UI，表示連線中斷，而且用戶端正在嘗試重新連線。
* `components-reconnect-hide`&ndash;用戶端具有使用中的連線，並隱藏 UI。
* `components-reconnect-failed` &ndash; 重新連線失敗，可能是因為網路失敗。 若要嘗試重新連接，請呼叫 `window.Blazor.reconnect()`。
* `components-reconnect-rejected` &ndash; 個重新連線遭到拒絕。 已達到伺服器但拒絕連線，而且伺服器上的使用者狀態已消失。 若要重載應用程式，請呼叫 `location.reload()`。 當下列情況發生時，可能會產生此連接狀態：
  * 線路中發生損毀（伺服器端程式碼）。
  * 用戶端中斷連線的時間夠長，伺服器才會捨棄使用者的狀態。 系統會處置使用者與之互動的元件實例。

### <a name="stateful-reconnection-after-prerendering"></a>預呈現後的具狀態重新連接

在建立伺服器的用戶端連接之前，預設會設定 Blazor 伺服器應用程式，以預先呈現伺服器上的 UI。 這是在 *_Host* Razor 頁面中設定：

```cshtml
<body>
    <app>@(await Html.RenderComponentAsync<App>(RenderMode.ServerPrerendered))</app>

    <script src="_framework/blazor.server.js"></script>
</body>
```

`RenderMode`設定元件是否：

* 會資源清單到頁面中。
* 會在頁面上轉譯為靜態 HTML，或包含從使用者代理程式啟動 Blazor 應用程式所需的資訊。

| `RenderMode`        | 描述 |
| ------------------- | ----------- |
| `ServerPrerendered` | 將元件轉譯為靜態 HTML，並包含 Blazor 伺服器應用程式的標記。 當使用者代理程式啟動時，會使用此標記來啟動 Blazor 應用程式。 不支援參數。 |
| `Server`            | 呈現 Blazor 伺服器應用程式的標記。 不包含來自元件的輸出。 當使用者代理程式啟動時，會使用此標記來啟動 Blazor 應用程式。 不支援參數。 |
| `Static`            | 將元件轉譯為靜態 HTML。 支援參數。 |

不支援從靜態 HTML 網頁轉譯伺服器元件。

用戶端會重新連線至伺服器，其狀態與用來呈現應用程式的狀態相同。 如果應用程式的狀態仍在記憶體中，則在建立 SignalR 連接之後，元件狀態不會重新顯示。

### <a name="render-stateful-interactive-components-from-razor-pages-and-views"></a>從 Razor 頁面和 views 轉譯具狀態的互動式元件

可設定狀態的互動式元件可以新增至 Razor 頁面或視圖。

當頁面或視圖呈現時：

* 此元件是使用頁面或視圖所資源清單。
* 用來進行預呈現的初始元件狀態會遺失。
* 建立 SignalR 連接時，會建立新的元件狀態。

下列 Razor 頁面會呈現`Counter`元件：

```cshtml
<h1>My Razor Page</h1>

@(await Html.RenderComponentAsync<Counter>(RenderMode.ServerPrerendered))
```

### <a name="render-noninteractive-components-from-razor-pages-and-views"></a>從 Razor 頁面和 views 轉譯非互動式元件

在下列 Razor 頁面中， `MyComponent`元件是使用以表單指定的初始值進行靜態轉譯：

```cshtml
<h1>My Razor Page</h1>

<form>
    <input type="number" asp-for="InitialValue" />
    <button type="submit">Set initial value</button>
</form>

@(await Html.RenderComponentAsync<MyComponent>(RenderMode.Static, 
    new { InitialValue = InitialValue }))

@code {
    [BindProperty(SupportsGet=true)]
    public int InitialValue { get; set; }
}
```

由於`MyComponent`是以靜態方式呈現，因此元件不能是互動式的。

### <a name="detect-when-the-app-is-prerendering"></a>偵測應用程式何時已進行預呈現

[!INCLUDE[](~/includes/blazor-prerendering.md)]

### <a name="configure-the-signalr-client-for-blazor-server-apps"></a>設定適用于 Blazor 伺服器應用程式的 SignalR 用戶端

有時候，您需要設定 Blazor 伺服器應用程式所使用的 SignalR 用戶端。 例如，您可能會想要在 SignalR 用戶端上設定記錄，以診斷連線問題。

在*Pages/_Host. cshtml*檔案中設定 SignalR 用戶端：

* 將屬性新增`<script>`至 blazor 的標記。 `autostart="false"`
* 呼叫`Blazor.start`並傳入指定 SignalR 產生器的設定物件。

```html
<script src="_framework/blazor.server.js" autostart="false"></script>
<script>
  Blazor.start({
    configureSignalR: function (builder) {
      builder.configureLogging(2); // LogLevel.Information
    }
  });
</script>
```

## <a name="additional-resources"></a>其他資源

* <xref:blazor/get-started>
* <xref:signalr/introduction>
