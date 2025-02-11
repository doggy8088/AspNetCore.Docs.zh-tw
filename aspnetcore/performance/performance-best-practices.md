---
title: ASP.NET Core 效能最佳做法
author: mjrousos
description: ASP.NET Core 應用程式中的效能和避免常見效能問題的秘訣。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 05/10/2019
uid: performance/performance-best-practices
ms.openlocfilehash: 7651dff18f98c60057660c8946c3daa66d272f6a
ms.sourcegitcommit: ffe3ed7921ec6c7c70abaac1d10703ec9a43374c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2019
ms.locfileid: "65536075"
---
# <a name="aspnet-core-performance-best-practices"></a>ASP.NET Core 效能最佳做法

藉由[Mike Rousos](https://github.com/mjrousos)

本主題提供指導方針的效能與 ASP.NET Core 的最佳作法。

<a name="hot"></a>

本文件中，*程式碼路徑*定義為程式碼路徑，常被稱為和大部分的執行時間發生的位置。 熱門的程式碼路徑通常會限制應用程式向外延展與效能。

## <a name="cache-aggressively"></a>積極地快取

快取會討論這份文件的幾個部分。 如需詳細資訊，請參閱 <xref:performance/caching/response>。

## <a name="avoid-blocking-calls"></a>避免封鎖呼叫

ASP.NET Core 應用程式應該可同時處理許多要求。 非同步 Api 可讓小型以處理數千個並行要求，不等待封鎖呼叫的執行緒集區。 而不是等待長時間執行同步工作完成，執行緒可以使用另一個要求。

常見的效能問題，ASP.NET Core 應用程式中的封鎖可能是非同步的呼叫。 許多同步封鎖呼叫會導致[執行緒集區耗盡](https://blogs.msdn.microsoft.com/vancem/2018/10/16/diagnosing-net-core-threadpool-starvation-with-perfview-why-my-service-is-not-saturating-all-cores-or-seems-to-stall/)且降低回應時間。

**不建議**：

* 封鎖呼叫的非同步執行[Task.Wait](/dotnet/api/system.threading.tasks.task.wait)或是[Task.Result](/dotnet/api/system.threading.tasks.task-1.result)。
* 取得常見的程式碼路徑中的鎖定。 ASP.NET Core 應用程式是架構以平行方式執行程式碼時的最有效。

**建議**：

* 製作[經常性存取程式碼路徑](#hot)非同步。
* 以非同步方式呼叫資料存取和長時間執行作業的 Api。
* 讓控制站/Razor 頁面動作非同步。 整個呼叫堆疊為了受益於[非同步/等候](/dotnet/csharp/programming-guide/concepts/async/)模式，因此是非同步的。

程式碼剖析工具，例如[PerfView](https://github.com/Microsoft/perfview)，可用來尋找執行緒不斷新增到[執行緒集區](/windows/desktop/procthread/thread-pools)。 `Microsoft-Windows-DotNETRuntime/ThreadPoolWorkerThread/Start`事件表示加入至執行緒集區的執行緒。 <!--  For more information, see [async guidance docs](TBD-Link_To_Davifowl_Doc  -->

## <a name="minimize-large-object-allocations"></a>最小化大型物件配置

<!-- TODO review Bill - replaced original .NET language below with .NET Core since this targets .NET Core -->
[.NET Core 記憶體回收行程](/dotnet/standard/garbage-collection/)會自動管理 ASP.NET Core 應用程式中的記憶體配置與釋放。 自動記憶體回收通常表示開發人員不需要擔心如何或何時釋放記憶體。 不過，清除未參考的物件會占用 CPU 時間，因此開發人員應該盡可能減少在[經常性存取程式碼路徑](#hot)中配置物件的情況。 在大型物件 (> 85,000 個位元組) 上執行記憶體回收特別耗費成本。 大型物件儲存在[大型物件堆積](/dotnet/standard/garbage-collection/large-object-heap)中，而且需要完整 (第 2 代) 記憶體回收來清除。 不同於第 0 代與第 1 代回收，第 2 代回收需要暫時停止應用程式執行。 常見的大型物件配置與取消配置可能會導致不一致的效能。

建議：

* **請**考慮快取經常使用的大型物件。 快取大型物件，可避免耗費資源的配置。
* **請**使用 [`ArrayPool<T>`](/dotnet/api/system.buffers.arraypool-1) 將緩衝區放入集區以儲存大型陣列。
* **請勿**在[經常性存取程式碼路徑](#hot)上配置許多存留期短的大型物件。

藉由檢閱中的記憶體回收 (GC) 統計資料，您可以診斷記憶體問題，例如先前所述， [PerfView](https://github.com/Microsoft/perfview)並檢查：

* 記憶體回收暫停時間。
* 在記憶體回收花費的處理器時間百分比。
* 多少記憶體回收是層代 0、 1 和 2。

如需詳細資訊，請參閱 <c0> [ 回收和效能](/dotnet/standard/garbage-collection/performance)。

## <a name="optimize-data-access"></a>最佳化資料存取

與資料存放區和其他的遠端服務互動通常是 ASP.NET Core 應用程式最慢的一部分。 讀取和寫入有效的資料是很重要的良好的效能。

建議：

* **請**以非同步方式呼叫所有資料存取 API。
* **請勿**擷取比所需資料多的資料。 撰寫查詢來只傳回目前的 HTTP 要求所需的資料。
* **請**考將擷取自資料庫或遠端服務的頻繁存取資料放入快取 (若可接受稍微過時的資料)。 視案例而定，使用 [MemoryCache](xref:performance/caching/memory) 或 [DistributedCache](xref:performance/caching/distributed)。 如需詳細資訊，請參閱 <xref:performance/caching/response>。
* **請**將網路來回行程降到最低。 目標是在單一呼叫中而非數個呼叫擷取所需的資料。
* **請**使用 Entity Framework Core 中的[無追蹤查詢](/ef/core/querying/tracking#no-tracking-queries) (針對唯讀目的存取資料時)。 EF Core 能以更有效率的方式傳回無追蹤查詢的結果。
* **請**篩選及彙總 LINQ 查詢 (例如，使用 `.Where`、`.Select` 或 `.Sum` 陳述式)，讓篩選由資料庫執行。
* **請**考慮 EF Core 會解析用戶端上的一些查詢運算子，這可能會導致沒有效率的查詢執行。 如需詳細資訊，請參閱[用戶端評估效能問題](/ef/core/querying/client-eval#client-evaluation-performance-issues)。
* **請勿**在集合上使用投影查詢，這可能會導致執行 "N+1" 個 SQL 查詢。 如需詳細資訊，請參閱[相互關聯子查詢的最佳化](/ef/core/what-is-new/ef-core-2.1#optimization-of-correlated-subqueries)。

請參閱[EF 高效能](/ef/core/what-is-new/ef-core-2.0#explicitly-compiled-queries)如可改善高級別應用程式的效能的解決方法：

* [DbContext 共用](/ef/core/what-is-new/ef-core-2.0#dbcontext-pooling)
* [明確地編譯的查詢](/ef/core/what-is-new/ef-core-2.0#explicitly-compiled-queries)

我們建議您衡量前面的高效能方法，再認可程式碼基底的影響。 已編譯查詢的額外複雜性也不是合理的效能改進。

藉由檢閱時間偵測到問題的查詢所花費的與的存取資料[Application Insights](/azure/application-insights/app-insights-overview)或程式碼剖析工具。 大部分的資料庫也提供統計資料相關常執行的查詢。

## <a name="pool-http-connections-with-httpclientfactory"></a>與 HttpClientFactory 的集區 HTTP 連線

雖然[HttpClient](/dotnet/api/system.net.http.httpclient)實作`IDisposable`介面，就適合重複使用。 關閉`HttpClient`執行個體保留通訊端在中開啟`TIME_WAIT`狀態一段時間。 如果程式碼路徑，會建立並處置`HttpClient`物件經常會，應用程式可能會耗盡可用的通訊端。 [HttpClientFactory](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)引進了 ASP.NET Core 2.1 中做為方案這個問題。 它會處理共用的 HTTP 連線，以最佳化效能和可靠性。

建議：

* **請勿**直接建立及處置 `HttpClient` 執行個體。
* **請**使用 [HttpClientFactory](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests) 來擷取 `HttpClient` 執行個體。 如需詳細資訊，請參閱[使用 HttpClientFactory 實作具有恢復功能的 HTTP 要求](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)。

## <a name="keep-common-code-paths-fast"></a>快速的常見程式碼路徑

您想要您的程式碼，快速、 經常呼叫的程式碼路徑都以最佳化最重要：

* 在應用程式的要求處理管線的中介軟體元件，特別是中介軟體提早在管線中執行。 這些元件會對效能很大的影響。
* 每個要求或多個時間每個要求所執行的程式碼。 比方說，自訂的記錄、 授權的處理常式或暫時性服務初始化。

建議：

* **請勿**搭配長時間執行的工作使用自訂中介軟體元件。
* **請**使用效能分析工具 (例如 [Visual Studio 診斷工具](/visualstudio/profiling/profiling-feature-tour)或 [PerfView](https://github.com/Microsoft/perfview)) 來識別[經常性存取程式碼路徑](#hot)。

## <a name="complete-long-running-tasks-outside-of-http-requests"></a>完成長時間執行的工作以外的 HTTP 要求

控制站或呼叫必要的服務，並傳回 HTTP 回應的頁面模型可以處理大部分的要求，以將 ASP.NET Core 應用程式。 某些涉及長時間執行工作的要求，最好讓整個要求-回應程序的非同步。

建議：

* **請勿**等候長時間執行的工作在一般 HTTP 要求處理期間完成。
* **請**考慮透過[背景服務](xref:fundamentals/host/hosted-services)或透過 [Azure Function](/azure/azure-functions/) 在處理序外處理長時間執行的要求。 對於需要大量 CPU 的工作，在處理序外完成工作特別有幫助。
* **請**使用即時通訊選項 (例如 [SignalR](xref:signalr/introduction))，以非同步方式與用戶端通訊。

## <a name="minify-client-assets"></a>縮短用戶端資產

使用複雜的前端的 ASP.NET Core 應用程式經常會提供許多 JavaScript、 CSS 或影像檔。 初始載入要求的效能可改善：

* 合併，將多個檔案合併成一個。
* 壓縮，藉由移除空白字元和註解減少檔案大小。

建議：

* **請**使用 ASP.NET Core 的[內建支援](xref:client-side/bundling-and-minification)來包裝及縮小用戶端資產。
* **請**考慮其他協力廠商工具 (例如 [Webpack](https://webpack.js.org/)) 來進行複雜的用戶端資產管理。

## <a name="compress-responses"></a>壓縮回應

 減少回應的大小通常可大幅提升應用程式的回應速度。 減少承載大小的其中一個方法是壓縮應用程式的回應。 如需詳細資訊，請參閱 <c0> [ 回應壓縮](xref:performance/response-compression)。

## <a name="use-the-latest-aspnet-core-release"></a>使用最新的 ASP.NET Core 版本

ASP.NET Core 的每個新版本包含效能改進。 .NET Core 與 ASP.NET Core 中的最佳化表示較新版本通常勝過較舊版本。 例如，.NET Core 2.1 新增支援編譯的規則運算式，並受惠[ `Span<T>` ](https://msdn.microsoft.com/magazine/mt814808.aspx)。 新增 ASP.NET Core 2.2 支援 HTTP/2。 如果效能是優先考量，請考慮升級至目前版本的 ASP.NET Core。

<!-- TODO review link and taking advantage of new [performance features](#TBD)
Maybe skip this TBD link as each version will have perf improvements -->

## <a name="minimize-exceptions"></a>最小化例外狀況

例外狀況應該很少見。 擲回和攔截例外狀況是相對於其他程式碼流程模式變慢。 因為這個緣故，例外狀況，不應該用來控制一般程式流程中。

建議：

* **請勿**使用擲回或攔截例外狀況做為一般程式流程，尤其是在[經常性存取程式碼路徑](#hot)中。
* **請**在應用程式中納入邏輯，以偵測並處理會造成例外狀況的情況。
* **請**擲回或攔截異常或非預期狀況的例外狀況。

應用程式診斷工具，例如 Application Insights 可協助識別可能會影響效能的應用程式中的常見例外狀況。
