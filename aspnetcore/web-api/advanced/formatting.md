---
title: 在 ASP.NET Core Web API 中格式化回應資料
author: ardalis
description: 了解如何在 ASP.NET Core Web API 中格式化回應資料。
ms.author: riande
ms.custom: H1Hack27Feb2017
ms.date: 8/22/2019
uid: web-api/advanced/formatting
ms.openlocfilehash: e503df3d81efbb2800503c0cb4ff5ae093b6e1ac
ms.sourcegitcommit: 023495344053dc59115c80538f0ece935e7490a2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2019
ms.locfileid: "71592357"
---
# <a name="format-response-data-in-aspnet-core-web-api"></a>在 ASP.NET Core Web API 中格式化回應資料

由 [Rick Anderson](https://twitter.com/RickAndMSFT) 與 [Steve Smith](https://ardalis.com/) 撰寫

ASP.NET Core MVC 支援格式化回應資料。 您可以使用特定格式或回應用戶端要求的格式來格式化回應資料。

[檢視或下載範例程式碼](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/formatting) \(英文\) ([如何下載](xref:index#how-to-download-a-sample))

## <a name="format-specific-action-results"></a>特定格式的動作結果

某些動作結果類型是特定格式所特有的，例如 <xref:Microsoft.AspNetCore.Mvc.JsonResult> 和 <xref:Microsoft.AspNetCore.Mvc.ContentResult>。 無論用戶端喜好設定為何，動作都可以傳回以特定格式格式化的結果。 例如， `JsonResult`傳回會傳回 JSON 格式的資料。 傳回`ContentResult`或字串會傳回純文字格式的字串資料。

動作不需要傳回任何特定的類型。 ASP.NET Core 支援任何物件傳回值。  傳回不<xref:Microsoft.AspNetCore.Mvc.IActionResult>是型別物件之動作的結果，會使用適當<xref:Microsoft.AspNetCore.Mvc.Formatters.IOutputFormatter>的實作為序列化。 如需詳細資訊，請參閱<xref:web-api/action-return-types>。

內建 helper 方法<xref:Microsoft.AspNetCore.Mvc.ControllerBase.Ok*>會傳回 JSON 格式的資料：[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_get)]

下載範例會傳回作者清單。 使用 F12 瀏覽器開發人員工具或[Postman](https://www.getpostman.com/tools)搭配先前的程式碼：

* 顯示包含**內容類型：** `application/json; charset=utf-8`的回應標頭。
* 系統會顯示要求標頭。 例如， `Accept`標頭。 前面的程式碼會忽略標頭。`Accept`

若要傳回純文字格式化資料，請使用 <xref:Microsoft.AspNetCore.Mvc.ContentResult.Content> 和 <xref:Microsoft.AspNetCore.Mvc.ContentResult.Content> 協助程式：

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_about)]

在上述程式碼中， `Content-Type`傳回的`text/plain`是。 傳回字串傳遞`Content-Type` `text/plain`下列內容：

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_string)]

針對具有多個傳回類型的動作`IActionResult`，傳回。 例如，根據執行的作業結果傳回不同的 HTTP 狀態碼。

## <a name="content-negotiation"></a>內容協調

當用戶端指定[Accept 標頭](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)時，就會發生內容協商。 ASP.NET Core 使用的預設格式為[JSON](https://json.org/)。 內容協商為：

* 由<xref:Microsoft.AspNetCore.Mvc.ObjectResult>執行。
* 內建自 helper 方法所傳回的狀態碼特定動作結果。 動作結果 helper 方法是以為基礎`ObjectResult`。

傳回模型型別時，傳回型別會是`ObjectResult`。

下列動作方法使用 `Ok` 和 `NotFound` 協助程式方法：

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_search)]

除非要求另一個格式，且伺服器可以傳回要求的格式，否則會傳回 JSON 格式的回應。 [Fiddler](https://www.telerik.com/fiddler)或[Postman](https://www.getpostman.com/tools)這類工具`Accept`可以設定標頭來指定傳回格式。 `Accept`當包含伺服器支援的類型時，就會傳回該類型。

根據預設，ASP.NET Core 只支援 JSON。 對於不會變更預設值的應用程式，一律會傳回 JSON 格式的回應，而不論用戶端要求為何。 下一節將說明如何新增其他格式器。

控制器動作可以傳回 Poco （簡單的 CLR 物件）。 當 POCO 傳回時，執行時間會自動建立`ObjectResult`包裝物件的。 用戶端會取得已格式化的序列化物件。 如果傳回的物件是`null` `204 No Content` ，則會傳迴響應。

傳回物件類型：

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_alias)]

在上述程式碼中，有效作者別名`200 OK`的要求會傳回含有作者資料的回應。 對無效別名`204 No Content`的要求會傳迴響應。

### <a name="the-accept-header"></a>Accept 標頭

當要求中出現`Accept`標頭時，就會進行內容協商。 當要求包含 accept 標頭時，ASP.NET Core：

* 依照喜好設定順序，列舉 accept 標頭中的媒體類型。
* 嘗試尋找可以使用其中一種指定的格式產生回應的格式器。

如果找不到可滿足用戶端要求的格式器，請 ASP.NET Core：

* `406 Not Acceptable` 如果<xref:Microsoft.AspNetCore.Mvc.MvcOptions>已設定，則會傳回，或-
* 嘗試尋找可產生回應的第一個格式器。

如果未針對要求的格式設定格式器，則會使用可格式化物件的第一個格式器。 如果要求`Accept`中未出現標頭：

* 第一個可以處理物件的格式器是用來序列化回應。
* 沒有任何進行中的協商。 伺服器正在決定要傳回的格式。

如果 Accept 標頭包含`*/*`，除非在上<xref:Microsoft.AspNetCore.Mvc.MvcOptions>將設`RespectBrowserAcceptHeader`為 true，否則會忽略標頭。

### <a name="browsers-and-content-negotiation"></a>瀏覽器和內容協調

與一般 API 用戶端不同的是`Accept` ，網頁瀏覽器會提供標頭。 網頁瀏覽器會指定許多格式，包括萬用字元。 根據預設，當架構偵測到要求來自瀏覽器時：

* 已`Accept`忽略標頭。
* 除非另有設定，否則內容會以 JSON 格式傳回。

使用 Api 時，這會在瀏覽器之間提供更一致的體驗。

若要設定應用程式以接受瀏覽器 accept 標<xref:Microsoft.AspNetCore.Mvc.MvcOptions.RespectBrowserAcceptHeader>頭`true`，請將設為：

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](./formatting/3.0sample/StartupRespectBrowserAcceptHeader.cs?name=snippet)]
::: moniker-end
::: moniker range="< aspnetcore-3.0"
[!code-csharp[](./formatting/sample/StartupRespectBrowserAcceptHeader.cs?name=snippet)]
::: moniker-end

### <a name="configure-formatters"></a>設定格式器

需要支援其他格式的應用程式可以新增適當的 NuGet 套件，並設定支援。 輸入和輸出有個別的格式器。 [模型](xref:mvc/models/model-binding)系結會使用輸入格式器。 輸出格式器是用來格式化回應。 如需建立自訂格式器的詳細資訊，請參閱[自訂格式化](xref:web-api/advanced/custom-formatters)器。

::: moniker range=">= aspnetcore-3.0"

### <a name="add-xml-format-support"></a>新增 XML 格式支援

使用<xref:System.Xml.Serialization.XmlSerializer>執行的 XML 格式器是透過<xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*>呼叫來設定：

[!code-csharp[](./formatting/3.0sample/Startup.cs?name=snippet)]

上述程式碼會使用來`XmlSerializer`序列化結果。

使用上述程式碼時，控制器方法應該根據要求的`Accept`標頭傳回適當的格式。

### <a name="configure-systemtextjson-based-formatters"></a>設定 System.Text.Json-based 格式器

`System.Text.Json` 型格式器可以使用 `Microsoft.AspNetCore.Mvc.JsonOptions.SerializerOptions` 來設定。

```csharp
services.AddControllers().AddJsonOptions(options =>
{
    // Use the default property (Pascal) casing.
    options.SerializerOptions.PropertyNamingPolicy = null;

    // Configure a custom converter.
    options.SerializerOptions.Converters.Add(new MyCustomJsonConverter());
});
```

以每個動作為基礎的輸出序列化選項，可以使用`JsonResult`進行設定。 例如:

```csharp
public IActionResult Get()
{
    return Json(model, new JsonSerializerOptions
    {
        options.WriteIndented = true,
    });
}
```

### <a name="add-newtonsoftjson-based-json-format-support"></a>新增 Newtonsoft.Json 型 JSON 格式支援

在 ASP.NET Core 3.0 之前，預設使用的`Newtonsoft.Json` JSON 格式器會使用封裝來執行。 在 ASP.NET Core 3.0 或更新版本中，預設 JSON 格式器是以 `System.Text.Json` 為基礎。 藉由`Newtonsoft.Json`安裝[NewtonsoftJson](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) NuGet 套件並在中`Startup.ConfigureServices`進行設定，即可取得根據格式的格式器和功能的支援。

[!code-csharp[](./formatting/3.0sample/StartupNewtonsoftJson.cs?name=snippet)]

某些功能可能不適用於以`System.Text.Json`為基礎的格式器，而且需要參考以為基礎的`Newtonsoft.Json`格式器。 如果應用程式`Newtonsoft.Json`有下列情況，請繼續使用以為基礎的格式器：

* 使用`Newtonsoft.Json`屬性。 例如，`[JsonProperty]` 或 `[JsonIgnore]`。
* 自訂序列化設定。
* 依賴提供的`Newtonsoft.Json`功能。
* 設定 `Microsoft.AspNetCore.Mvc.JsonResult.SerializerSettings`。 在 ASP.NET Core 3.0 版之前，`JsonResult.SerializerSettings` 接受 `Newtonsoft.Json` 專屬的 `JsonSerializerSettings` 執行個體。
* 產生 [OpenAPI](<xref:tutorials/web-api-help-pages-using-swagger>) 文件。

您可以使用`Newtonsoft.Json` `Microsoft.AspNetCore.Mvc.MvcNewtonsoftJsonOptions.SerializerSettings`來設定以為基礎之格式器的功能：

```csharp
services.AddControllers().AddNewtonsoftJson(options =>
{
    // Use the default property (Pascal) casing
    options.SerializerSettings.ContractResolver = new DefaultContractResolver();

    // Configure a custom converter
    options.SerializerOptions.Converters.Add(new MyCustomJsonConverter());
});
```

以每個動作為基礎的輸出序列化選項，可以使用`JsonResult`進行設定。 例如:

```csharp
public IActionResult Get()
{
    return Json(model, new JsonSerializerSettings
    {
        options.Formatting = Formatting.Indented,
    });
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

### <a name="add-xml-format-support"></a>新增 XML 格式支援

XML 格式設定需要[AspNetCore 的 xml](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Formatters.Xml/) NuGet 套件。

使用<xref:System.Xml.Serialization.XmlSerializer>執行的 XML 格式器是透過<xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*>呼叫來設定：

[!code-csharp[](./formatting/sample/Startup.cs?name=snippet)]

上述程式碼會使用來`XmlSerializer`序列化結果。

使用上述程式碼時，控制器方法應該根據要求的`Accept`標頭傳回適當的格式。

::: moniker-end

### <a name="specify-a-format"></a>指定格式

若要限制回應格式，請[`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute)套用篩選準則。 就像大部分的`[Produces]` [篩選器](xref:mvc/controllers/filters)一樣，可以在動作、控制器或全域範圍套用：

[!code-csharp[](./formatting/3.0sample/Controllers/WeatherForecastController.cs?name=snippet)]

上述[`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute)篩選準則：

* 強制控制器內的所有動作傳回 JSON 格式的回應。
* 如果設定了其他格式器，而且用戶端指定了不同的格式，則會傳回 JSON。

如需詳細資訊，請參閱[篩選](xref:mvc/controllers/filters)。

### <a name="special-case-formatters"></a>特殊案例格式器

有些特殊案例是使用內建格式器所實作。 根據預設， `string`傳回類型的格式為*text/純文字*（如果透過`Accept`標頭要求，則為*text/html* ）。 您可以藉由移除<xref:Microsoft.AspNetCore.Mvc.Formatters.TextOutputFormatter>來刪除這個行為。 已移除方法中的`Configure`格式器。 `204 No Content` 傳回`null`時，具有模型物件傳回類型的動作會返回。 您可以藉由移除<xref:Microsoft.AspNetCore.Mvc.Formatters.HttpNoContentOutputFormatter>來刪除這個行為。 下列程式碼會移除 `TextOutputFormatter` 和 `HttpNoContentOutputFormatter`。

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](./formatting/3.0sample/StartupTextOutputFormatter.cs?name=snippet)]
::: moniker-end
::: moniker range="< aspnetcore-3.0"
[!code-csharp[](./formatting/sample/StartupTextOutputFormatter.cs?name=snippet)]
::: moniker-end

若沒有`string` `406 Not Acceptable`，傳回類型會傳回。 `TextOutputFormatter` 如果 XML 格式器存在，則會`string`格式化傳回的`TextOutputFormatter`類型（如果已移除）。

如果沒有 `HttpNoContentOutputFormatter`，則會使用已設定的格式器來格式化 Null 物件。 例如:

* JSON 格式器會傳回具有主體的`null`回應。
* Xml 格式器會傳回具有屬性`xsi:nil="true"`集的空 xml 元素。

## <a name="response-format-url-mappings"></a>回應格式 URL 對應

用戶端可以要求特定格式做為 URL 的一部分，例如：

* 在查詢字串或部分路徑中。
* 使用格式特定的副檔名，例如 .xml 或. json。

應該在 API 所使用的路由中指定要求路徑的對應。 例如:

[!code-csharp[](./formatting/sample/Controllers/ProductsController.cs?name=snippet)]

先前的路由可讓要求的格式指定為選用的副檔名。 屬性會檢查`RouteData`中的格式值是否存在，並在建立回應時，將回應格式對應至適當的格式器。 [`[FormatFilter]`](xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute)

|           路由        |             格式器              |
|------------------------|------------------------------------|
|   `/api/products/5`    |    預設輸出格式器    |
| `/api/products/5.json` | JSON 格式器 (如果已設定) |
| `/api/products/5.xml`  | XML 格式器 (如果已設定)  |
