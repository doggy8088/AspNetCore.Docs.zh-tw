---
title: ASP.NET Core SignalR 設定
author: bradygaster
description: 瞭解如何設定 ASP.NET Core SignalR 應用程式。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 08/05/2019
uid: signalr/configuration
ms.openlocfilehash: 156ffac83fbdf61fd88ad8acc307c2c701c46bca
ms.sourcegitcommit: f65d8765e4b7c894481db9b37aa6969abc625a48
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2019
ms.locfileid: "70773923"
---
# <a name="aspnet-core-signalr-configuration"></a>ASP.NET Core SignalR 設定

## <a name="jsonmessagepack-serialization-options"></a>JSON/MessagePack 序列化選項

ASP.NET Core SignalR 支援兩種通訊協定來編碼訊息：[JSON](https://www.json.org/)和[MessagePack](https://msgpack.org/index.html)。 每個通訊協定都有序列化設定選項。

您可以在伺服器上使用[AddJsonProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.jsonprotocoldependencyinjectionextensions.addjsonprotocol)擴充方法來設定 JSON 序列化，這可在您`Startup.ConfigureServices`的方法中[AddSignalR](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr)之後加入。 方法會接受可`options`接收物件的委派。 `AddJsonProtocol` 該物件上的[PayloadSerializerSettings](/dotnet/api/microsoft.aspnetcore.signalr.jsonhubprotocoloptions.payloadserializersettings)屬性是 JSON.NET `JsonSerializerSettings`物件，可用於設定引數和傳回值的序列化。 如需詳細資訊，請參閱[JSON.NET 檔](https://www.newtonsoft.com/json/help/html/Introduction.htm)。

例如，若要設定序列化程式使用 "PascalCase" 屬性名稱，而不是預設的 "camelCase" 名稱，請使用下列程式碼：

```csharp
services.AddSignalR()
    .AddJsonProtocol(options => {
        options.PayloadSerializerSettings.ContractResolver =
        new DefaultContractResolver();
    });
```

在 .net 用戶端中，相同`AddJsonProtocol`的擴充方法存在於[HubConnectionBuilder](/dotnet/api/microsoft.aspnetcore.signalr.client.hubconnectionbuilder)上。 必須匯入命名空間，才能解析擴充方法： `Microsoft.Extensions.DependencyInjection`

```csharp
// At the top of the file:
using Microsoft.Extensions.DependencyInjection;

// When constructing your connection:
var connection = new HubConnectionBuilder()
    .AddJsonProtocol(options => {
        options.PayloadSerializerSettings.ContractResolver =
            new DefaultContractResolver();
    })
    .Build();
```

> [!NOTE]
> 此時不可能在 JavaScript 用戶端中設定 JSON 序列化。

### <a name="messagepack-serialization-options"></a>MessagePack 序列化選項

MessagePack 序列化可以藉由提供委派給[AddMessagePackProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.msgpackprotocoldependencyinjectionextensions.addmessagepackprotocol)呼叫來設定。 如需詳細資訊，請參閱[SignalR 中的 MessagePack](xref:signalr/messagepackhubprotocol) 。

> [!NOTE]
> 此時不可能在 JavaScript 用戶端中設定 MessagePack 序列化。

## <a name="configure-server-options"></a>設定伺服器選項

下表說明設定 SignalR 中樞的選項：

::: moniker range=">= aspnetcore-3.0"

| 選項 | 預設值 | 說明 |
| ------ | ------------- | ----------- |
| `ClientTimeoutInterval` | 30 秒 | 如果用戶端未在此間隔內收到訊息（包括 keep-alive），伺服器會將它視為已中斷連線。 這可能需要比此逾時間隔更長的時間，讓用戶端實際被標示為已中斷連線，因為這是如何實行的。 建議的值為值的`KeepAliveInterval`雙精度浮點數。|
| `HandshakeTimeout` | 15 秒 | 如果用戶端未在此時間間隔內傳送初始交握訊息，連接就會關閉。 這是一種只有在因網路延遲嚴重而發生交握逾時錯誤時，才應該修改的「高級」設定。 如需交握程式的詳細資訊，請參閱[SignalR 中樞通訊協定規格](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 如果伺服器未在此間隔內傳送訊息，則會自動傳送 ping 訊息，讓連接保持開啟。 在變更時`ServerTimeout` / `serverTimeoutInMilliseconds` ，請變更用戶端上的設定。 `KeepAliveInterval` 建議`ServerTimeout` / `KeepAliveInterval`的值為值的雙精度浮點數。 `serverTimeoutInMilliseconds`  |
| `SupportedProtocols` | 所有已安裝的通訊協定 | 此中樞支援的通訊協定。 根據預設，允許在伺服器上註冊的所有通訊協定，但是可以從這份清單中移除通訊協定，以停用個別中樞的特定通訊協定。 |
| `EnableDetailedErrors` | `false` | 如果`true`為，當中樞方法擲回例外狀況時，會將詳細的例外狀況訊息傳回給用戶端。 預設值為`false`，因為這些例外狀況訊息可能包含機密資訊。 |
| `StreamBufferCapacity` | `10` | 可以針對用戶端上傳資料流程進行緩衝處理的專案數上限。 若達到此限制，則在伺服器處理資料流程專案之前，會封鎖調用的處理。|
| `MaximumReceiveMessageSize` | 32 KB | 單一傳入中樞訊息的大小上限。 |

::: moniker-end

::: moniker range="= aspnetcore-2.2"

| 選項 | 預設值 | 描述 |
| ------ | ------------- | ----------- |
| `ClientTimeoutInterval` | 30 秒 | 如果用戶端未在此間隔內收到訊息（包括 keep-alive），伺服器會將它視為已中斷連線。 這可能需要比此逾時間隔更長的時間，讓用戶端實際被標示為已中斷連線，因為這是如何實行的。 建議的值為值的`KeepAliveInterval`雙精度浮點數。|
| `HandshakeTimeout` | 15 秒 | 如果用戶端未在此時間間隔內傳送初始交握訊息，連接就會關閉。 這是一種只有在因網路延遲嚴重而發生交握逾時錯誤時，才應該修改的「高級」設定。 如需交握程式的詳細資訊，請參閱[SignalR 中樞通訊協定規格](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 如果伺服器未在此間隔內傳送訊息，則會自動傳送 ping 訊息，讓連接保持開啟。 在變更時`ServerTimeout` / `serverTimeoutInMilliseconds` ，請變更用戶端上的設定。 `KeepAliveInterval` 建議`ServerTimeout` / `KeepAliveInterval`的值為值的雙精度浮點數。 `serverTimeoutInMilliseconds`  |
| `SupportedProtocols` | 所有已安裝的通訊協定 | 此中樞支援的通訊協定。 根據預設，允許在伺服器上註冊的所有通訊協定，但是可以從這份清單中移除通訊協定，以停用個別中樞的特定通訊協定。 |
| `EnableDetailedErrors` | `false` | 如果`true`為，當中樞方法擲回例外狀況時，會將詳細的例外狀況訊息傳回給用戶端。 預設值為`false`，因為這些例外狀況訊息可能包含機密資訊。 |

::: moniker-end

::: moniker range="= aspnetcore-2.1"

| 選項 | 預設值 | 描述 |
| ------ | ------------- | ----------- |
| `HandshakeTimeout` | 15 秒 | 如果用戶端未在此時間間隔內傳送初始交握訊息，連接就會關閉。 這是一種只有在因網路延遲嚴重而發生交握逾時錯誤時，才應該修改的「高級」設定。 如需交握程式的詳細資訊，請參閱[SignalR 中樞通訊協定規格](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 如果伺服器未在此間隔內傳送訊息，則會自動傳送 ping 訊息，讓連接保持開啟。 在變更時`ServerTimeout` / `serverTimeoutInMilliseconds` ，請變更用戶端上的設定。 `KeepAliveInterval` 建議`ServerTimeout` / `KeepAliveInterval`的值為值的雙精度浮點數。 `serverTimeoutInMilliseconds`  |
| `SupportedProtocols` | 所有已安裝的通訊協定 | 此中樞支援的通訊協定。 根據預設，允許在伺服器上註冊的所有通訊協定，但是可以從這份清單中移除通訊協定，以停用個別中樞的特定通訊協定。 |
| `EnableDetailedErrors` | `false` | 如果`true`為，當中樞方法擲回例外狀況時，會將詳細的例外狀況訊息傳回給用戶端。 預設值為`false`，因為這些例外狀況訊息可能包含機密資訊。 |

::: moniker-end

您可以為所有中樞設定選項，方法是提供選項委派給`AddSignalR`中`Startup.ConfigureServices`的呼叫。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSignalR(hubOptions =>
    {
        hubOptions.EnableDetailedErrors = true;
        hubOptions.KeepAliveInterval = TimeSpan.FromMinutes(1);
    });
}
```

單一中樞的選項會覆寫中提供的全域`AddSignalR`選項，並可使用<xref:Microsoft.Extensions.DependencyInjection.SignalRDependencyInjectionExtensions.AddHubOptions*>下列方式進行設定：

```csharp
services.AddSignalR().AddHubOptions<MyHub>(options =>
{
    options.EnableDetailedErrors = true;
});
```

### <a name="advanced-http-configuration-options"></a>Advanced HTTP 設定選項

::: moniker range=">= aspnetcore-3.0"

使用`HttpConnectionDispatcherOptions`來設定與傳輸和記憶體緩衝區管理相關的 advanced 設定。 這些選項的設定方式是將委派傳遞至中`Startup.Configure`的[MapHub\<T >](/dotnet/api/microsoft.aspnetcore.builder.hubendpointroutebuilderextensions.maphub) 。

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<MyHub>("/myhub", options =>
        {
            options.Transports =
                HttpTransportType.WebSockets |
                HttpTransportType.LongPolling;
        });
    });
}
```
::: moniker-end

::: moniker range="<= aspnetcore-2.2"

使用`HttpConnectionDispatcherOptions`來設定與傳輸和記憶體緩衝區管理相關的 advanced 設定。 這些選項的設定方式是將委派傳遞至中`Startup.Configure`的[MapHub\<T >](/dotnet/api/microsoft.aspnetcore.signalr.hubroutebuilder.maphub) 。

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseSignalR((configure) =>
    {
        var desiredTransports =
            HttpTransportType.WebSockets |
            HttpTransportType.LongPolling;

        configure.MapHub<MyHub>("/myhub", (options) =>
        {
            options.Transports = desiredTransports;
        });
    });
}
```

::: moniker-end

下表說明設定 ASP.NET Core SignalR 的 advanced HTTP 選項的選項：

::: moniker range=">= aspnetcore-3.0"

| 選項 | 預設值 | 說明 |
| ------ | ------------- | ----------- |
| `ApplicationMaxBufferSize` | 32 KB | 在套用背壓之前，伺服器會緩衝的用戶端接收的最大位元組數目。 增加此值可讓伺服器更快速地接收較大的訊息，而不需套用背壓，但可能會增加記憶體耗用量。 |
| `AuthorizationData` | 從套用至中樞類別`Authorize`的屬性自動收集的資料。 | 用來判斷是否授權用戶端連線到中樞的[IAuthorizeData](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizedata)物件清單。 |
| `TransportMaxBufferSize` | 32 KB | 在觀察到背壓之前，伺服器會緩衝的應用程式所傳送的最大位元組數目。 增加此值可讓伺服器更快速地緩衝較大的訊息，而不需等待背壓，但可能會增加記憶體耗用量。 |
| `Transports` | 所有傳輸都已啟用。 | `HttpTransportType`值的位旗標列舉，可以限制用戶端可用來連接的傳輸。 |
| `LongPolling` | 請參閱下方。 | 長輪詢傳輸特定的其他選項。 |
| `WebSockets` | 請參閱下方。 | Websocket 傳輸特定的其他選項。 |

::: moniker-end

::: moniker range="< aspnetcore-3.0"

| 選項 | 預設值 | 說明 |
| ------ | ------------- | ----------- |
| `ApplicationMaxBufferSize` | 32 KB | 從用戶端接收伺服器緩衝區的最大位元組數目。 增加這個值可讓伺服器接收較大的訊息，但可能會對記憶體耗用量造成負面影響。 |
| `AuthorizationData` | 從套用至中樞類別`Authorize`的屬性自動收集的資料。 | 用來判斷是否授權用戶端連線到中樞的[IAuthorizeData](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizedata)物件清單。 |
| `TransportMaxBufferSize` | 32 KB | 由伺服器所緩衝的應用程式所傳送的最大位元組數目。 增加這個值可讓伺服器傳送較大的訊息，但可能會對記憶體耗用量造成負面影響。 |
| `Transports` | 所有傳輸都已啟用。 | `HttpTransportType`值的位旗標列舉，可以限制用戶端可用來連接的傳輸。 |
| `LongPolling` | 請參閱下方。 | 長輪詢傳輸特定的其他選項。 |
| `WebSockets` | 請參閱下方。 | Websocket 傳輸特定的其他選項。 |

::: moniker-end

長輪詢傳輸具有其他選項，可以使用`LongPolling`屬性來設定：

| 選項 | 預設值 | 描述 |
| ------ | ------------- | ----------- |
| `PollTimeout` | 90秒 | 在終止單一輪詢要求之前，伺服器等候訊息傳送至用戶端的最大時間量。 降低此值會導致用戶端更頻繁地發出新的輪詢要求。 |

WebSocket 傳輸具有可使用`WebSockets`屬性來設定的其他選項：

| 選項 | 預設值 | 說明 |
| ------ | ------------- | ----------- |
| `CloseTimeout` | 5 秒 | 伺服器關閉之後，如果用戶端無法在此時間間隔內關閉，連接就會終止。 |
| `SubProtocolSelector` | `null` | 可以用來將`Sec-WebSocket-Protocol`標頭設定為自訂值的委派。 委派會接收用戶端要求的值做為輸入，而且預期會傳回所需的值。 |

## <a name="configure-client-options"></a>設定用戶端選項

您可以在`HubConnectionBuilder`類型上設定用戶端選項（可在 .net 和 JavaScript 用戶端中使用）。 它也可在 JAVA 用戶端中使用，但`HttpHubConnectionBuilder`子類別則包含產生器設定選項， `HubConnection`以及本身。

### <a name="configure-logging"></a>設定記錄

使用`ConfigureLogging`方法，在 .net 用戶端中設定記錄。 記錄提供者和篩選器可以用與伺服器上相同的方式進行註冊。 如需詳細資訊，請參閱[ASP.NET Core 檔中的登入](xref:fundamentals/logging/index)。

> [!NOTE]
> 若要註冊記錄提供者，您必須安裝必要的套件。 如需完整清單，請參閱檔的[內建記錄提供者](xref:fundamentals/logging/index#built-in-logging-providers)一節。

例如，若要啟用主控台記錄，請安裝`Microsoft.Extensions.Logging.Console` NuGet 套件。 `AddConsole`呼叫擴充方法：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub")
    .ConfigureLogging(logging => {
        logging.SetMinimumLevel(LogLevel.Information);
        logging.AddConsole();
    })
    .Build();
```

在 JavaScript 用戶端中，有`configureLogging`類似的方法存在。 `LogLevel`提供值，指出要產生的記錄訊息的最小層級。 記錄檔會寫入至瀏覽器主控台視窗。

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub")
    .configureLogging(signalR.LogLevel.Information)
    .build();
```

::: moniker range=">= aspnetcore-3.0"

您也可以`string`提供代表記錄層級名稱的值，而不是值。`LogLevel` 在您無法存取常數的`LogLevel`環境中設定 SignalR 記錄時，這會很有用。

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub")
    .configureLogging("warn")
    .build();
```

下表列出可用的記錄層級。 您所提供的值`configureLogging` ，用來設定將記錄的**最低**記錄層級。 記錄在此層級的訊息，**或在資料表中所列的層級**將會記錄下來。

| String                      | LogLevel               |
| --------------------------- | ---------------------- |
| `trace`                     | `LogLevel.Trace`       |
| `debug`                     | `LogLevel.Debug`       |
| `info`**或**`information` | `LogLevel.Information` |
| `warn`**或**`warning`     | `LogLevel.Warning`     |
| `error`                     | `LogLevel.Error`       |
| `critical`                  | `LogLevel.Critical`    |
| `none`                      | `LogLevel.None`        |

::: moniker-end

> [!NOTE]
> 若要完全停`signalR.LogLevel.None` `configureLogging`用記錄，請在方法中指定。

如需記錄的詳細資訊，請參閱[SignalR 診斷檔](xref:signalr/diagnostics)。

SignalR JAVA 用戶端會使用[SLF4J](https://www.slf4j.org/)程式庫進行記錄。 它是高階記錄 API，可讓程式庫的使用者藉由帶入特定的記錄相依性來選擇自己的特定記錄執行。 下列程式碼片段示範如何搭配 SignalR JAVA `java.util.logging`用戶端使用。

```gradle
implementation 'org.slf4j:slf4j-jdk14:1.7.25'
```

如果您未在相依性中設定記錄，SLF4J 會載入預設的無作業記錄器，並顯示下列警告訊息：

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

可以放心地忽略這種情況。

### <a name="configure-allowed-transports"></a>設定允許的傳輸

SignalR 所使用的傳輸可以在`WithUrl`呼叫中設定（`withUrl`以 JavaScript 為限）。 值的`HttpTransportType`位 or 可以用來限制用戶端只使用指定的傳輸。 預設會啟用所有傳輸。

例如，若要停用伺服器傳送的事件傳輸，但允許 Websocket 和較長的輪詢連接：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub", HttpTransportType.WebSockets | HttpTransportType.LongPolling)
    .Build();
```

在 JavaScript 用戶端中，會透過在提供給`transport` `withUrl`的選項物件上設定欄位，來設定傳輸：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub", { transport: signalR.HttpTransportType.WebSockets | signalR.HttpTransportType.LongPolling })
    .build();
```

::: moniker range=">= aspnetcore-2.2"

在此版本的 JAVA 用戶端 websocket 是唯一可用的傳輸。

::: moniker-end

::: moniker range="= aspnetcore-3.0"

在 JAVA 用戶端中，會使用`withTransport` `HttpHubConnectionBuilder`上的方法來選取傳輸。 JAVA 用戶端預設為使用 Websocket 傳輸。

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/myhub")
    .withTransport(TransportEnum.WEBSOCKETS)
    .build();
```

> [!NOTE]
> SignalR JAVA 用戶端尚不支援傳輸回退。

::: moniker-end

### <a name="configure-bearer-authentication"></a>設定持有人驗證

若要提供驗證資料以及 SignalR 要求，請使用`AccessTokenProvider`選項（`accessTokenFactory`在 JavaScript 中）來指定函式，以傳回所需的存取權杖。 在 .net 用戶端中，此存取權杖會當做 HTTP 「持有人驗證」權杖（使用`Authorization`類型為的`Bearer`標頭）傳入。 在 JavaScript 用戶端中，存取權杖會用來做為持有人權杖，**但**在少數情況下，瀏覽器 api 會限制套用標頭的功能（特別是在伺服器傳送的事件和 websocket 要求中）。 在這些情況下，存取權杖會以查詢字串值`access_token`的形式提供。

在 .net 用戶端中， `AccessTokenProvider`可以使用中`WithUrl`的選項委派來指定選項：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub", options => {
        options.AccessTokenProvider = async () => {
            // Get and return the access token.
        };
    })
    .Build();
```

在 JavaScript 用戶端中，存取權杖是藉由在`accessTokenFactory` `withUrl`的選項物件上設定欄位來設定：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub", {
        accessTokenFactory: () => {
            // Get and return the access token.
            // This function can return a JavaScript Promise if asynchronous
            // logic is required to retrieve the access token.
        }
    })
    .build();
```

在 SignalR JAVA 用戶端中，您可以藉由提供存取權杖 factory 給[HttpHubConnectionBuilder](/java/api/com.microsoft.signalr._http_hub_connection_builder?view=aspnet-signalr-java)，設定要用於驗證的持有人權杖。 使用[withAccessTokenFactory](/java/api/com.microsoft.signalr._http_hub_connection_builder.withaccesstokenprovider?view=aspnet-signalr-java#com_microsoft_signalr__http_hub_connection_builder_withAccessTokenProvider_Single_String__)提供[RxJAVA](https://github.com/ReactiveX/RxJava) [單一\<字串 >](https://reactivex.io/documentation/single.html)。 只要呼叫[單一. defer](https://reactivex.io/RxJava/javadoc/io/reactivex/Single.html#defer-java.util.concurrent.Callable-)，您就可以撰寫邏輯來產生用戶端的存取權杖。

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/myhub")
    .withAccessTokenProvider(Single.defer(() -> {
        // Your logic here.
        return Single.just("An Access Token");
    })).build();
```

### <a name="configure-timeout-and-keep-alive-options"></a>設定 timeout 和 keep-alive 選項

設定 timeout 和 keep-alive 行為的其他選項可`HubConnection`用於物件本身：


::: moniker range=">= aspnetcore-2.2"

# <a name="nettabdotnet"></a>[.NET](#tab/dotnet)

| 選項 | 預設值 | 描述 |
| ------ | ------------- | ----------- |
| `ServerTimeout` | 30秒（30000毫秒） | 伺服器活動的超時時間。 如果伺服器未在此間隔內傳送訊息，用戶端會將伺服器視為已中斷連線， `Closed`並觸發`onclose`事件（在 JavaScript 中）。 這個值必須夠大，才能從伺服器傳送 ping 訊息 **，並**在逾時間隔內接收用戶端。 建議的值是至少兩個伺服器`KeepAliveInterval`值的數位，以允許時間到達 ping。 |
| `HandshakeTimeout` | 15 秒 | 初始伺服器交握的超時時間。 如果伺服器未在此間隔內傳送交握回應，用戶端會取消交握並觸發`Closed`事件（`onclose`在 JavaScript 中）。 這是一種只有在因網路延遲嚴重而發生交握逾時錯誤時，才應該修改的「高級」設定。 如需交握程式的詳細資訊，請參閱[SignalR 中樞通訊協定規格](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 決定用戶端傳送 ping 訊息的間隔。 從用戶端傳送任何訊息時，會將計時器重設為間隔的開始。 如果用戶端未在伺服器上的`ClientTimeoutInterval`集合中傳送訊息，伺服器會將用戶端視為已中斷連線。 |

在 .net 用戶端中，timeout 值會指定`TimeSpan`為值。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

| 選項 | 預設值 | 說明 |
| ------ | ------------- | ----------- |
| `serverTimeoutInMilliseconds` | 30秒（30000毫秒） | 伺服器活動的超時時間。 如果伺服器未在此間隔內傳送訊息，用戶端會將伺服器視為已中斷連線， `onclose`並觸發事件。 這個值必須夠大，才能從伺服器傳送 ping 訊息 **，並**在逾時間隔內接收用戶端。 建議的值是至少兩個伺服器`KeepAliveInterval`值的數位，以允許時間到達 ping。 |
| `keepAliveIntervalInMilliseconds` | 15秒（15000毫秒） | 決定用戶端傳送 ping 訊息的間隔。 從用戶端傳送任何訊息時，會將計時器重設為間隔的開始。 如果用戶端未在伺服器上的`ClientTimeoutInterval`集合中傳送訊息，伺服器會將用戶端視為已中斷連線。 |

# <a name="javatabjava"></a>[Java](#tab/java)

| 選項 | 預設值 | 描述 |
| ------ | ------------- | ----------- |
| `getServerTimeout` / `setServerTimeout` | 30秒（30000毫秒） | 伺服器活動的超時時間。 如果伺服器未在此間隔內傳送訊息，用戶端會將伺服器視為已中斷連線， `onClose`並觸發事件。 這個值必須夠大，才能從伺服器傳送 ping 訊息 **，並**在逾時間隔內接收用戶端。 建議的值是至少兩個伺服器`KeepAliveInterval`值的數位，以允許時間到達 ping。 |
| `withHandshakeResponseTimeout` | 15 秒 | 初始伺服器交握的超時時間。 如果伺服器未在此間隔內傳送交握回應，用戶端會取消交握並觸發`onClose`事件。 這是一種只有在因網路延遲嚴重而發生交握逾時錯誤時，才應該修改的「高級」設定。 如需交握程式的詳細資訊，請參閱[SignalR 中樞通訊協定規格](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `getKeepAliveInterval` / `setKeepAliveInterval` | 15秒（15000毫秒） | 決定用戶端傳送 ping 訊息的間隔。 從用戶端傳送任何訊息時，會將計時器重設為間隔的開始。 如果用戶端未在伺服器上的`ClientTimeoutInterval`集合中傳送訊息，伺服器會將用戶端視為已中斷連線。 |

::: moniker-end

::: moniker range="< aspnetcore-2.2"

# <a name="nettabdotnet"></a>[.NET](#tab/dotnet)

| 選項 | 預設值 | 說明 |
| ------ | ------------- | ----------- |
| `ServerTimeout` | 30秒（30000毫秒） | 伺服器活動的超時時間。 如果伺服器未在此間隔內傳送訊息，用戶端會將伺服器視為已中斷連線， `Closed`並觸發`onclose`事件（在 JavaScript 中）。 這個值必須夠大，才能從伺服器傳送 ping 訊息 **，並**在逾時間隔內接收用戶端。 建議的值是至少兩個伺服器`KeepAliveInterval`值的數位，以允許時間到達 ping。 |
| `HandshakeTimeout` | 15 秒 | 初始伺服器交握的超時時間。 如果伺服器未在此間隔內傳送交握回應，用戶端會取消交握並觸發`Closed`事件（`onclose`在 JavaScript 中）。 這是一種只有在因網路延遲嚴重而發生交握逾時錯誤時，才應該修改的「高級」設定。 如需交握程式的詳細資訊，請參閱[SignalR 中樞通訊協定規格](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |

在 .net 用戶端中，timeout 值會指定`TimeSpan`為值。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

| 選項 | 預設值 | 描述 |
| ------ | ------------- | ----------- |
| `serverTimeoutInMilliseconds` | 30秒（30000毫秒） | 伺服器活動的超時時間。 如果伺服器未在此間隔內傳送訊息，用戶端會將伺服器視為已中斷連線， `onclose`並觸發事件。 這個值必須夠大，才能從伺服器傳送 ping 訊息 **，並**在逾時間隔內接收用戶端。 建議的值是至少兩個伺服器`KeepAliveInterval`值的數位，以允許時間到達 ping。 |

# <a name="javatabjava"></a>[Java](#tab/java)

| 選項 | 預設值 | 描述 |
| ------ | ------------- | ----------- |
| `getServerTimeout` / `setServerTimeout` | 30秒（30000毫秒） | 伺服器活動的超時時間。 如果伺服器未在此間隔內傳送訊息，用戶端會將伺服器視為已中斷連線， `onClose`並觸發事件。 這個值必須夠大，才能從伺服器傳送 ping 訊息 **，並**在逾時間隔內接收用戶端。 建議的值是至少為伺服器`KeepAliveInterval`值的兩個數字，以允許時間到達 ping。 |
| `withHandshakeResponseTimeout` | 15 秒 | 初始伺服器交握的超時時間。 如果伺服器未在此間隔內傳送交握回應，用戶端會取消交握並觸發`onClose`事件。 這是一種只有在因網路延遲嚴重而發生交握逾時錯誤時，才應該修改的「高級」設定。 如需交握程式的詳細資訊，請參閱[SignalR 中樞通訊協定規格](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |

::: moniker-end

---

### <a name="configure-additional-options"></a>設定其他選項

`WithUrl`您可以在上`withUrl` `HubConnectionBuilder`的（JavaScript）方法中， `HttpHubConnectionBuilder`或在 JAVA 用戶端中的各種設定 api 上設定其他選項：

# <a name="nettabdotnet"></a>[.NET](#tab/dotnet)

| .NET 選項 |  預設值 | 描述 |
| ----------- | -------------- | ----------- |
| `AccessTokenProvider` | `null` | 傳回字串的函式，在 HTTP 要求中提供為持有人驗證權杖。 |
| `SkipNegotiation` | `false` | 將此設`true`為以略過協商步驟。 **只有在 websocket 傳輸為唯一啟用的傳輸時才支援**。 使用 Azure SignalR Service 時，無法啟用此設定。 |
| `ClientCertificates` | Empty | 要傳送以驗證要求的 TLS 憑證集合。 |
| `Cookies` | Empty | 要與每個 HTTP 要求一起傳送的 HTTP cookie 集合。 |
| `Credentials` | Empty | 每個 HTTP 要求傳送的認證。 |
| `CloseTimeout` | 5 秒 | 僅限 Websocket。 關閉伺服器以認可關閉要求之後，用戶端等待的最大時間量。 如果伺服器在這段時間內未認可關閉，用戶端就會中斷連線。 |
| `Headers` | Empty | 要隨每個 HTTP 要求傳送之其他 HTTP 標頭的對應。 |
| `HttpMessageHandlerFactory` | `null` | 委派，可以用來設定或取代用來傳送`HttpMessageHandler` HTTP 要求的。 不用於 WebSocket 連接。 這個委派必須傳回非 null 值，而且會收到預設值做為參數。 請修改該預設值的設定並加以傳回，或傳回新`HttpMessageHandler`的實例。 **取代處理常式時，請務必從提供的處理常式中複製您想要保留的設定，否則設定的選項（例如 Cookie 和標頭）將不會套用至新的處理常式。** |
| `Proxy` | `null` | 傳送 HTTP 要求時所要使用的 HTTP proxy。 |
| `UseDefaultCredentials` | `false` | 設定此布林值，以傳送 HTTP 和 Websocket 要求的預設認證。 這可讓您使用 Windows 驗證。 |
| `WebSocketConfiguration` | `null` | 可以用來設定其他 WebSocket 選項的委派。 接收可用於設定選項的[ClientWebSocketOptions](/dotnet/api/system.net.websockets.clientwebsocketoptions)實例。 |

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

| JavaScript 選項 | 預設值 | 描述 |
| ----------------- | ------------- | ----------- |
| `accessTokenFactory` | `null` | 傳回字串的函式，在 HTTP 要求中提供為持有人驗證權杖。 |
| `skipNegotiation` | `false` | 將此設`true`為以略過協商步驟。 **只有在 websocket 傳輸為唯一啟用的傳輸時才支援**。 使用 Azure SignalR Service 時，無法啟用此設定。 |

# <a name="javatabjava"></a>[Java](#tab/java)

| JAVA 選項 | 預設值 | 描述 |
| ----------- | ------------- | ----------- |
| `withAccessTokenProvider` | `null` | 傳回字串的函式，在 HTTP 要求中提供為持有人驗證權杖。 |
| `shouldSkipNegotiate` | `false` | 將此設`true`為以略過協商步驟。 **只有在 websocket 傳輸為唯一啟用的傳輸時才支援**。 使用 Azure SignalR Service 時，無法啟用此設定。 |
| `withHeader` `withHeaders` | Empty | 要隨每個 HTTP 要求傳送之其他 HTTP 標頭的對應。 |

---

在 .NET 用戶端中，這些選項可以由提供給`WithUrl`的選項委派進行修改：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub", options => {
        options.Headers["Foo"] = "Bar";
        options.Cookies.Add(new Cookie(/* ... */);
        options.ClientCertificates.Add(/* ... */);
    })
    .Build();
```

在 JavaScript 用戶端中，可以在提供給`withUrl`的 javascript 物件中提供這些選項：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub", {
        skipNegotiation: true,
        transport: signalR.HttpTransportType.WebSockets
    })
    .build();
```

在 JAVA 用戶端中，您可以使用所`HttpHubConnectionBuilder`傳回之上的方法來設定這些選項。`HubConnectionBuilder.create("HUB URL")`

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/myhub")
        .withHeader("Foo", "Bar")
        .shouldSkipNegotiate(true)
        .withHandshakeResponseTimeout(30*1000)
        .build();
```

## <a name="additional-resources"></a>其他資源

* <xref:tutorials/signalr>
* <xref:signalr/hubs>
* <xref:signalr/javascript-client>
* <xref:signalr/dotnet-client>
* <xref:signalr/messagepackhubprotocol>
* <xref:signalr/supported-platforms>
