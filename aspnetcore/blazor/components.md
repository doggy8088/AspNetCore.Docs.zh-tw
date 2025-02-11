---
title: 建立和使用 ASP.NET Core Razor 元件
author: guardrex
description: 瞭解如何建立和使用 Razor 元件，包括如何系結至資料、處理事件，以及管理元件生命週期。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 09/30/2019
uid: blazor/components
ms.openlocfilehash: ea216e405e5be52b578e99a529d8c6a726ea9cdd
ms.sourcegitcommit: fe88748b762525cb490f7e39089a4760f6a73a24
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/30/2019
ms.locfileid: "71688035"
---
# <a name="create-and-use-aspnet-core-razor-components"></a>建立和使用 ASP.NET Core Razor 元件

By [Luke Latham](https://github.com/guardrex)和[Daniel Roth](https://github.com/danroth27)

[檢視或下載範例程式碼](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) \(英文\) ([如何下載](xref:index#how-to-download-a-sample))

Blazor 應用程式是使用*元件*所建立。 「元件」（component）是一種獨立的使用者介面（UI）區塊，例如頁面、對話方塊或表單。 元件包含 HTML 標籤，以及插入資料或回應 UI 事件所需的處理邏輯。 元件具有彈性且輕量。 它們可以在專案之間進行嵌套、重複使用及共用。

## <a name="component-classes"></a>元件類別

元件會使用C#和 HTML 標籤的組合，在[razor](xref:mvc/views/razor)元件檔案（*razor*）中執行。 Blazor 中的元件正式稱為*Razor 元件*。

元件的名稱必須以大寫字元開頭。 例如， *MyCoolComponent*有效，而*MyCoolComponent*則無效。

元件的 UI 是使用 HTML 定義的。 動態轉譯邏輯 (例如迴圈、條件、運算式) 是使用內嵌的 C# 語法 (稱為 [Razor](xref:mvc/views/razor)) 來新增的。 編譯應用程式時，會將 HTML 標籤和C#轉譯邏輯轉換成元件類別。 產生的類別名稱與檔案的名稱相符。

元件類別的成員均定義於 `@code` 區塊中。 `@code`在區塊中，會使用事件處理或定義其他元件邏輯的方法來指定元件狀態（屬性、欄位）。 允許一個以上的 `@code` 區塊。

> [!NOTE]
> 在 ASP.NET Core 3.0 的先前預覽中`@functions` ，區塊用於與 Razor 元件中的`@code`區塊相同的用途。 `@functions`區塊會繼續在 Razor 元件中運作，但我們建議使用`@code` ASP.NET Core 3.0 Preview 6 或更新版本中的區塊。

元件成員可以使用C#開頭為`@`的運算式，做為元件轉譯邏輯的一部分。 例如， C#欄位的呈現方式是在功能變數名稱`@`前面加上。 下列範例會評估並呈現：

* `_headingFontStyle`至的 CSS 屬性值`font-style`。
* `_headingText`至`<h1>`元素的內容。

```cshtml
<h1 style="font-style:@_headingFontStyle">@_headingText</h1>

@code {
    private string _headingFontStyle = "italic";
    private string _headingText = "Put on your new Blazor!";
}
```

一開始呈現元件之後，元件會重新產生其轉譯樹狀結構，以回應事件。 然後，Blazor 會比較新的轉譯樹狀結構與上一個，並將任何修改套用至瀏覽器的檔物件模型（DOM）。

元件是一般C#類別，可以放在專案內的任何位置。 產生網頁的元件通常會位於*Pages*資料夾中。 非頁面元件通常會放在*共用*資料夾中，或加入至專案的自訂資料夾中。 若要使用自訂資料夾，請將自訂資料夾的命名空間新增至父元件或應用程式的 *_Imports*檔案。 例如，下列命名空間會在應用程式的根命名空間為`WebApplication`時，讓元件資料夾中的元件可供使用：

```cshtml
@using WebApplication.Components
```

## <a name="integrate-components-into-razor-pages-and-mvc-apps"></a>將元件整合到 Razor Pages 和 MVC 應用程式

將元件與現有的 Razor Pages 和 MVC 應用程式搭配使用。 不需要重新撰寫現有的頁面或 views 就能使用 Razor 元件。 當頁面或視圖呈現時，會同時資源清單元件。

若要從頁面或視圖呈現元件，請使用`RenderComponentAsync<TComponent>` HTML helper 方法：

```cshtml
<div id="MyComponent">
    @(await Html.RenderComponentAsync<MyComponent>(RenderMode.ServerPrerendered))
</div>
```

雖然頁面和視圖可以使用元件，但相反的情況並非如此。 元件不能使用視圖和頁面特定的案例，例如部分視圖和區段。 若要在元件中使用部分視圖的邏輯，請將部分視圖邏輯分解成元件。

如需有關如何呈現元件，以及如何在 Blazor 伺服器應用程式中管理元件狀態的詳細<xref:blazor/hosting-models>資訊，請參閱一文。

## <a name="use-components"></a>使用元件

元件可以包含其他元件，方法是使用 HTML 專案語法來宣告它們。 使用元件的標記看起來像是 HTML 標籤，其中標籤名稱是元件類型。

屬性系結會區分大小寫。 例如， `@bind`是有效的，而且`@Bind`無效。

在*Index*中的下列標記會呈現`HeadingComponent`實例：

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/Index.razor?name=snippet_HeadingComponent)]

*Components/HeadingComponent. razor*：

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/HeadingComponent.razor)]

如果元件包含的 HTML 專案具有大寫的第一個字母，但不符合元件名稱，則會發出警告，指出該元素有未預期的名稱。 為元件的命名空間加入語句，可讓元件可用，這會移除警告。`@using`

## <a name="component-parameters"></a>元件參數

元件可以具有*元件參數*，其定義方式是在元件類別上使用具有`[Parameter]`屬性的公用屬性。 使用這些屬性來指定標記中元件的引數。

*Components/ChildComponent. razor*：

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/ChildComponent.razor?highlight=11-12)]

在下列範例中， `ParentComponent`會設定的`Title`屬性`ChildComponent`值。

*Pages/ParentComponent. razor*：

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/ParentComponent.razor?name=snippet_ParentComponent&highlight=5-6)]

## <a name="child-content"></a>子內容

元件可以設定另一個元件的內容。 指派元件會在指定接收元件的標記之間提供內容。

在下列範例中， `ChildComponent`有一個`ChildContent`代表`RenderFragment`的屬性，代表要呈現的 UI 區段。 的值`ChildContent`位於元件的標記中，應在其中呈現內容。 的值`ChildContent`會從父元件接收，並在啟動載入面板的`panel-body`內轉譯。

*Components/ChildComponent. razor*：

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/ChildComponent.razor?highlight=3,14-15)]

> [!NOTE]
> 接收`RenderFragment`內容的屬性必須依照慣例命名`ChildContent` 。

下列`ParentComponent`可以提供內容，將內容`<ChildComponent>`放`ChildComponent`在標籤內來呈現。

*Pages/ParentComponent. razor*：

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/ParentComponent.razor?name=snippet_ParentComponent&highlight=7-8)]

## <a name="attribute-splatting-and-arbitrary-parameters"></a>屬性展開和任意參數

除了元件的宣告參數之外，元件還可以捕捉和轉譯其他屬性。 您可以在字典中捕捉其他屬性，然後在使用[@attributes](xref:mvc/views/razor#attributes) Razor 指示詞轉譯元件時，splatted 至元素。 當定義的元件會產生支援各種自訂的標記專案時，這個案例就很有用。 例如，針對`<input>`支援許多參數的，分別定義屬性可能會很繁瑣。

在下列範例中，第一個`<input>`元素（`id="useIndividualParams"`）會使用個別的元件參數，而`<input>`第二`id="useAttributesDict"`個元素（）則使用屬性展開：

```cshtml
<input id="useIndividualParams"
       maxlength="@Maxlength"
       placeholder="@Placeholder"
       required="@Required"
       size="@Size" />

<input id="useAttributesDict"
       @attributes="InputAttributes" />

@code {
    [Parameter]
    public string Maxlength { get; set; } = "10";

    [Parameter]
    public string Placeholder { get; set; } = "Input placeholder text";

    [Parameter]
    public string Required { get; set; } = "required";

    [Parameter]
    public string Size { get; set; } = "50";

    [Parameter]
    public Dictionary<string, object> InputAttributes { get; set; } =
        new Dictionary<string, object>()
        {
            { "maxlength", "10" },
            { "placeholder", "Input placeholder text" },
            { "required", "required" },
            { "size", "50" }
        };
}
```

參數的類型必須使用字串索引`IEnumerable<KeyValuePair<string, object>>`鍵來執行。 在`IReadOnlyDictionary<string, object>`此案例中，使用也是一個選項。

使用這兩種方法的轉譯元素都相同：`<input>`

```html
<input id="useIndividualParams"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">

<input id="useAttributesDict"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">
```

若要接受任意屬性，請使用`[Parameter]` `CaptureUnmatchedValues`屬性設定為的屬性來`true`定義元件參數：

```cshtml
@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public Dictionary<string, object> InputAttributes { get; set; }
}
```

`CaptureUnmatchedValues` 上`[Parameter]`的屬性允許參數比對與任何其他參數不相符的所有屬性。 元件只能定義具有`CaptureUnmatchedValues`的單一參數。 搭配使用`CaptureUnmatchedValues`的屬性類型必須可從`Dictionary<string, object>`使用字串索引鍵來指派。 `IEnumerable<KeyValuePair<string, object>>`或`IReadOnlyDictionary<string, object>`也是此案例中的選項。

## <a name="data-binding"></a>資料繫結

元件和 DOM 元素的資料系結都是使用[@bind](xref:mvc/views/razor#bind)屬性來完成。 下列範例`_italicsCheck`會將欄位系結至核取方塊的已核取狀態：

```cshtml
<input type="checkbox" class="form-check-input" id="italicsCheck" 
    @bind="_italicsCheck" />
```

選取並清除核取方塊時，屬性的值會分別更新為`true`和。 `false`

只有在呈現元件時，才會在 UI 中更新此核取方塊，而不是回應變更屬性的值。 由於元件會在事件處理常式程式碼執行之後自行呈現，因此屬性更新通常會立即反映在 UI 中。

使用`@bind`搭配屬性（`<input @bind="CurrentValue" />`）基本上等同于下列內容： `CurrentValue`

```cshtml
<input value="@CurrentValue"
    @onchange="@((ChangeEventArgs __e) => CurrentValue = __e.Value)" />
```

當元件呈現時， `value`輸入專案的會`CurrentValue`來自屬性。 當使用者在文字方塊中輸入時， `onchange`就會引發事件， `CurrentValue`並將屬性設定為已變更的值。 事實上，程式碼產生會稍微複雜一點，因為`@bind`會處理一些執行型別轉換的情況。 在原則上`@bind` ，會將運算式的目前值`value`與屬性產生關聯，並使用已註冊的處理常式來處理變更。

`onchange`除了使用[@bind-value](xref:mvc/views/razor#bind) `event` [@bind-value:event](xref:mvc/views/razor#bind)語法來處理事件之外，也可以使用其他事件來系結屬性或欄位，方法是指定具有參數（）的屬性。 `@bind` 下列範例`CurrentValue`會系結`oninput`事件的屬性：

```cshtml
<input @bind-value="CurrentValue" @bind-value:event="oninput" />
```

不同`onchange`于當元素失去`oninput`焦點時引發的，會在文字方塊的值變更時引發。

**無法剖析的值**

當使用者將無法剖析的值提供給資料系結元素時，會在觸發系結事件時，自動將無法剖析的值還原成先前的值。

請考慮下列案例：

* 元素會系結`int`至具有初始值的`123`類型： `<input>`

  ```cshtml
  <input @bind="MyProperty" />

  @code {
      [Parameter]
      public int MyProperty { get; set; } = 123;
  }
  ```
* 使用者會將專案的值更新為`123.45`頁面中的，並變更元素的焦點。

在上述案例中，元素的值會還原成`123`。 當以的`123.45`原始`123`值拒絕值時，使用者瞭解其值不被接受。

根據預設，系結會套用至元素`onchange`的事件`@bind="{PROPERTY OR FIELD}"`（）。 使用`@bind-value="{PROPERTY OR FIELD}" @bind-value:event={EVENT}`來設定不同的事件。 對於事件（`@bind-value:event="oninput"`），回復會在引入無法剖析值的任何擊鍵之後發生。 `oninput` 以系結`oninput`型別為目標的事件時，使用者無法輸入`.`字元。 `int` 系統會立即移除字元，讓使用者收到只允許整數的立即回應。`.` 在某些情況下，在`oninput`事件上還原值不是理想的情況，例如，當使用者應允許清除無法解析`<input>`的值時。 替代方案包括：

* 請勿使用`oninput`事件。 使用預設`onchange`事件（`@bind="{PROPERTY OR FIELD}"`），在此情況下，除非元素失去焦點，否則不正確值不會還原。
* 系結至可為 null 的型`int?`別`string`（例如或），並提供自訂邏輯來處理不正確專案。
* 使用[表單驗證元件](xref:blazor/forms-validation)，例如`InputNumber`或`InputDate`。 表單驗證元件有內建支援，可管理不正確輸入。 表單驗證元件：
  * 允許使用者在相關聯`EditContext`的上提供不正確輸入和接收驗證錯誤。
  * 在 UI 中顯示驗證錯誤，而不幹擾使用者輸入其他 webform 資料。

**全球化**

`@bind`系統會使用目前文化特性的規則，將值格式化以顯示和剖析。

您可以從<xref:System.Globalization.CultureInfo.CurrentCulture?displayProperty=fullName>屬性存取目前的文化特性。

[InvariantCulture](xref:System.Globalization.CultureInfo.InvariantCulture)用於下欄欄位類型（`<input type="{TYPE}" />`）：

* `date`
* `number`

前面的欄位類型：

* 會使用其適當的瀏覽器格式規則來顯示。
* 不能包含自由格式的文字。
* 根據瀏覽器的執行方式提供使用者互動特性。

下欄欄位類型具有特定的格式需求，而且目前不受 Blazor 支援，因為並非所有主要瀏覽器都支援：

* `datetime-local`
* `month`
* `week`

`@bind`支援參數，以提供用於剖析和格式化值的。 <xref:System.Globalization.CultureInfo?displayProperty=fullName> `@bind:culture` 使用`date`和欄位類型時， `number`不建議指定文化特性。 `date`和`number`具有提供必要文化特性的內建 Blazor 支援。

如需如何設定使用者文化特性的詳細資訊，請參閱[當地語系化](#localization)一節。

**格式字串**

資料系結會使用來[@bind:format](xref:mvc/views/razor#bind)處理格式字串。<xref:System.DateTime> 目前無法使用其他格式運算式，例如貨幣或數位格式。

```cshtml
<input @bind="StartDate" @bind:format="yyyy-MM-dd" />

@code {
    [Parameter]
    public DateTime StartDate { get; set; } = new DateTime(2020, 1, 1);
}
```

在上述程式碼中， `<input>`元素的欄位類型（`type`）預設為`text`。 `@bind:format`支援系結下列 .NET 類型：

* <xref:System.DateTime?displayProperty=fullName>
* <xref:System.DateTime?displayProperty=fullName>?
* <xref:System.DateTimeOffset?displayProperty=fullName>
* <xref:System.DateTimeOffset?displayProperty=fullName>?

屬性會指定要套用`value`至`<input>`元素之的日期格式。 `@bind:format` 當發生`onchange`事件時，也會使用此格式來剖析值。

不建議指定`date`欄位類型的格式，因為 Blazor 具有格式日期的內建支援。

**元件參數**

Binding 可辨識元件參數， `@bind-{property}`其中可以跨元件系結屬性值。

下列子元件（`ChildComponent`） `Year`具有元件參數和`YearChanged`回呼：

```cshtml
<h2>Child Component</h2>

<p>Year: @Year</p>

@code {
    [Parameter]
    public int Year { get; set; }

    [Parameter]
    public EventCallback<int> YearChanged { get; set; }
}
```

`EventCallback<T>`[app eventcallback](#eventcallback)一節中會說明。

下列父元件會使用`ChildComponent` ，並`ParentYear`將參數從父系系結至`Year`子元件上的參數：

```cshtml
@page "/ParentComponent"

<h1>Parent Component</h1>

<p>ParentYear: @ParentYear</p>

<ChildComponent @bind-Year="ParentYear" />

<button class="btn btn-primary" @onclick="ChangeTheYear">
    Change Year to 1986
</button>

@code {
    [Parameter]
    public int ParentYear { get; set; } = 1978;

    private void ChangeTheYear()
    {
        ParentYear = 1986;
    }
}
```

載入會`ParentComponent`產生下列標記：

```html
<h1>Parent Component</h1>

<p>ParentYear: 1978</p>

<h2>Child Component</h2>

<p>Year: 1978</p>
```

`ParentYear`如果藉由選取`ParentComponent`中的按鈕來變更屬性的值`ChildComponent` ， `Year`則會更新的屬性。 當為重新顯示時`Year` ，的新值會在 UI 中呈現： `ParentComponent`

```html
<h1>Parent Component</h1>

<p>ParentYear: 1986</p>

<h2>Child Component</h2>

<p>Year: 1986</p>
```

參數`Year`是可系結的，因為它`YearChanged`有`Year`符合參數類型的伴隨事件。

依照慣例， `<ChildComponent @bind-Year="ParentYear" />`基本上等同于撰寫：

```cshtml
<ChildComponent @bind-Year="ParentYear" @bind-Year:event="YearChanged" />
```

一般來說，屬性可以使用`@bind-property:event`屬性系結至對應的事件處理常式。 例如，您可以使用`MyProp`下列兩個屬性`MyEventHandler` ，將屬性系結至：

```cshtml
<MyComponent @bind-MyProp="MyValue" @bind-MyProp:event="MyEventHandler" />
```

## <a name="event-handling"></a>事件處理

Razor 元件提供事件處理功能。 針對名為`on{event}`的 HTML 專案屬性（例如， `onclick`和`onsubmit`）與委派類型的值，Razor 元件會將屬性的值視為事件處理常式。 屬性的名稱一律會格式化[ @on{event}](xref:mvc/views/razor#onevent)。

下列程式碼會在`UpdateHeading` UI 中選取按鈕時呼叫方法：

```cshtml
<button class="btn btn-primary" @onclick="UpdateHeading">
    Update heading
</button>

@code {
    private void UpdateHeading(MouseEventArgs e)
    {
        ...
    }
}
```

下列程式碼會在`CheckChanged` UI 中變更核取方塊時呼叫方法：

```cshtml
<input type="checkbox" class="form-check-input" @onchange="CheckChanged" />

@code {
    private void CheckChanged()
    {
        ...
    }
}
```

事件處理常式也可以是非同步<xref:System.Threading.Tasks.Task>，並且會傳回。 不需要手動呼叫`StateHasChanged()`。 例外狀況會在發生時記錄。

在下列範例中， `UpdateHeading`當選取按鈕時，會以非同步方式呼叫：

```cshtml
<button class="btn btn-primary" @onclick="UpdateHeading">
    Update heading
</button>

@code {
    private async Task UpdateHeading(MouseEventArgs e)
    {
        ...
    }
}
```

### <a name="event-argument-types"></a>事件引數類型

對於某些事件，則允許事件引數類型。 如果不需要存取這些事件種類的其中一個，則在方法呼叫中不需要。

支援的 `EventArgs` 會顯示在下表中。

| Event - 事件 | 類別 |
| ----- | ----- |
| 剪貼簿        | `ClipboardEventArgs` |
| 拖放式             | `DragEventArgs`&ndash; 並按住`DataTransferItem`拖曳的專案資料。 `DataTransfer` |
| Error            | `ErrorEventArgs` |
| 焦點            | `FocusEventArgs`不包含的`relatedTarget`支援。 &ndash; |
| `<input>` 變更 | `ChangeEventArgs` |
| 鍵盤         | `KeyboardEventArgs` |
| 滑鼠            | `MouseEventArgs` |
| 滑鼠指標    | `PointerEventArgs` |
| 滑鼠滾輪      | `WheelEventArgs` |
| 進度         | `ProgressEventArgs` |
| 觸控            | `TouchEventArgs`&ndash; 代表觸控裝置上`TouchPoint`的單一連絡人點。 |

如需上表中事件的屬性和事件處理行為的詳細資訊，請參閱[參考來源中的 EventArgs 類別（aspnet/AspNetCore release/3.0 分支）](https://github.com/aspnet/AspNetCore/tree/release/3.0/src/Components/Web/src/Web)。

### <a name="lambda-expressions"></a>Lambda 運算式

您也可以使用 Lambda 運算式：

```cshtml
<button @onclick="@(e => Console.WriteLine("Hello, world!"))">Say hello</button>
```

關閉其他值（例如反覆運算一組專案時）通常會很方便。 下列範例會建立三個按鈕，在 UI 中`UpdateHeading`選取時，每個`MouseEventArgs`都會呼叫傳遞事件引數`buttonNumber`（）和其按鈕編號（）：

```cshtml
<h2>@message</h2>

@for (var i = 1; i < 4; i++)
{
    var buttonNumber = i;

    <button class="btn btn-primary"
            @onclick="@(e => UpdateHeading(e, buttonNumber))">
        Button #@i
    </button>
}

@code {
    private string message = "Select a button to learn its position.";

    private void UpdateHeading(MouseEventArgs e, int buttonNumber)
    {
        message = $"You selected Button #{buttonNumber} at " +
            $"mouse position: {e.ClientX} X {e.ClientY}.";
    }
}
```

> [!NOTE]
> 請**不要在**lambda 運算式中直接`i`使用`for`迴圈中的迴圈變數（）。 否則，所有 lambda 運算式都會使用相同的變數， `i`使的值在所有 lambda 中都相同。 請一律在本機變數中捕捉其值`buttonNumber` （在上述範例中為），然後使用它。

### <a name="eventcallback"></a>App eventcallback

有一個常見的嵌套元件案例，就是在子元件事件發生&mdash;時（例如， `onclick`當子系中發生事件時），想要執行父元件的方法。 若要在元件之間公開事件， `EventCallback`請使用。 父元件可以將回呼方法指派給子元件的`EventCallback`。

範例`ChildComponent`應用程式中的會示範如何`EventCallback`設定按鈕`onclick`的處理常式，以接收來自範例的`ParentComponent`委派。 會使用`MouseEventArgs`輸入，`onclick`這適用于來自週邊裝置的事件： `EventCallback`

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/ChildComponent.razor?highlight=5-7,17-18)]

會將子系`ShowMessage`設定為其方法：`EventCallback<T>` `ParentComponent`

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/ParentComponent.razor?name=snippet_ParentComponent&highlight=6,16-19)]

當您在中`ChildComponent`選取按鈕時：

* 會呼叫`ShowMessage`的方法。 `ParentComponent` `messageText`會更新並顯示在中`ParentComponent`。
* 回呼的方法`StateHasChanged` （`ShowMessage`）中不需要呼叫。 `StateHasChanged`會自動呼叫來 rerender `ParentComponent`，就像子事件會觸發元件 rerendering 在子系內執行的事件處理常式一樣。

`EventCallback`和`EventCallback<T>`允許非同步委派。 `EventCallback<T>`是強型別，而且需要特定的引數類型。 `EventCallback`是弱型別，並允許任何引數類型。

```cshtml
<p><b>@messageText</b></p>

@{ var message = "Default Text"; }

<ChildComponent 
    OnClick="@(async () => { await Task.Yield(); messageText = "Blaze It!"; })" />

@code {
    private string messageText;
}
```

使用叫用`EventCallback<T>` <xref:System.Threading.Tasks.Task>或，並等待： `EventCallback` `InvokeAsync`

```csharp
await callback.InvokeAsync(arg);
```

針對`EventCallback`事件`EventCallback<T>`處理和系結元件參數使用和。

慣用強`EventCallback<T>` `EventCallback`型別。 `EventCallback<T>`為元件的使用者提供更好的錯誤意見反應。 與其他 UI 事件處理常式類似，指定事件參數是選擇性的。 當`EventCallback`沒有任何值傳遞給回呼時，請使用。

## <a name="chained-bind"></a>連鎖系結

常見的案例是將資料系結參數連結至元件輸出中的 page 元素。 此案例稱為*連鎖*系結，因為會同時發生多個層級的系結。

無法在頁面的元素中使用`@bind`語法來執行連鎖系結。 事件處理常式和值必須分別指定。 不過，父系元件可以使用`@bind`語法搭配元件的參數。

下列`PasswordField`元件（*self.passwordfield.text*）：

* 將元素的值設定`Password`為屬性。 `<input>`
* 將`Password`屬性的變更公開至具有[app eventcallback](#eventcallback)的父元件。

```cshtml
Password: 

<input @oninput="OnPasswordChanged" 
       required 
       type="@(showPassword ? "text" : "password")" 
       value="@Password" />

<button class="btn btn-primary" @onclick="ToggleShowPassword">
    Show password
</button>

@code {
    private bool showPassword;

    [Parameter]
    public string Password { get; set; }

    [Parameter]
    public EventCallback<string> PasswordChanged { get; set; }

    private Task OnPasswordChanged(ChangeEventArgs e)
    {
        Password = e.Value.ToString();

        return PasswordChanged.InvokeAsync(Password);
    }

    private void ToggleShowPassword()
    {
        showPassword = !showPassword;
    }
}
```

`PasswordField`元件會在另一個元件中使用：

```cshtml
<PasswordField @bind-Password="password" />

@code {
    private string password;
}
```

若要對上述範例中的密碼執行檢查或陷印錯誤：

* 建立的支援欄位（ `Password` `password`在下列範例程式碼中）。
* 執行`Password` setter 中的檢查或陷阱錯誤。

如果密碼的值中使用了空格，下列範例會向使用者提供立即的意見反應：

```cshtml
Password: 

<input @oninput="OnPasswordChanged" 
       required 
       type="@(showPassword ? "text" : "password")" 
       value="@Password" />

<button class="btn btn-primary" @onclick="ToggleShowPassword">
    Show password
</button>

<span class="text-danger">@validationMessage</span>

@code {
    private bool showPassword;
    private string password;
    private string validationMessage;

    [Parameter]
    public string Password
    {
        get { return password ?? string.Empty; }
        set
        {
            if (password != value)
            {
                if (value.Contains(' '))
                {
                    validationMessage = "Spaces not allowed!";
                }
                else
                {
                    password = value;
                    validationMessage = string.Empty;
                }
            }
        }
    }

    [Parameter]
    public EventCallback<string> PasswordChanged { get; set; }

    private Task OnPasswordChanged(ChangeEventArgs e)
    {
        Password = e.Value.ToString();

        return PasswordChanged.InvokeAsync(Password);
    }

    private void ToggleShowPassword()
    {
        showPassword = !showPassword;
    }
}
```

## <a name="capture-references-to-components"></a>捕獲元件的參考

元件參考提供參考元件實例的方法，讓您可以對該實例發出命令，例如`Show`或。 `Reset` 若要捕捉元件參考：

* [@ref](xref:mvc/views/razor#ref)將屬性加入至子元件。
* 定義與子元件類型相同的欄位。

```cshtml
<MyLoginDialog @ref="loginDialog" ... />

@code {
    private MyLoginDialog loginDialog;

    private void OnSomething()
    {
        loginDialog.Show();
    }
}
```

當元件呈現時， `loginDialog`欄位會填入`MyLoginDialog`子元件實例。 接著，您可以在元件實例上叫用 .NET 方法。

> [!IMPORTANT]
> 只有在轉譯元件之後才會填入`MyLoginDialog` 變數，而且其輸出會包含元素。`loginDialog` 直到該點為止，沒有任何可參考的內容。 若要在元件完成呈現之後操作元件參考，請使用`OnAfterRenderAsync`或`OnAfterRender`方法。

雖然捕捉元件參考使用類似的語法來[捕捉元素參考](xref:blazor/javascript-interop#capture-references-to-elements)，但它並不是[JavaScript interop](xref:blazor/javascript-interop)功能。 元件參考不會傳遞至 JavaScript&mdash;程式碼，而只會在 .net 程式碼中使用。

> [!NOTE]
> 請勿**使用元件**參考來改變子元件的狀態。 請改用一般宣告式參數，將資料傳遞至子元件。 使用一般宣告式參數會導致子元件自動 rerender 正確的時間。

## <a name="invoke-component-methods-externally-to-update-state"></a>在外部叫用元件方法來更新狀態

Blazor 會使用`SynchronizationContext`來強制執行單一邏輯執行緒。 元件的生命週期方法以及 Blazor 所引發的任何事件回呼都會在這個`SynchronizationContext`上執行。 在事件中，必須根據外來事件（例如計時器或其他通知）來更新元件，請使用`InvokeAsync`方法，這會分派給 Blazor 的。 `SynchronizationContext`

例如，假設有一個通知程式*服務*可通知任何處于已更新狀態的「接聽」元件：

```csharp
public class NotifierService
{
    // Can be called from anywhere
    public async Task Update(string key, int value)
    {
        if (Notify != null)
        {
            await Notify.Invoke(key, value);
        }
    }

    public event Func<string, int, Task> Notify;
}
```

`NotifierService`使用來更新元件：

```cshtml
@page "/"
@inject NotifierService Notifier
@implements IDisposable

<p>Last update: @lastNotification.key = @lastNotification.value</p>

@code {
    private (string key, int value) lastNotification;

    protected override void OnInitialized()
    {
        Notifier.Notify += OnNotify;
    }

    public async Task OnNotify(string key, int value)
    {
        await InvokeAsync(() =>
        {
            lastNotification = (key, value);
            StateHasChanged();
        });
    }

    public void Dispose()
    {
        Notifier.Notify -= OnNotify;
    }
}
```

在上述範例中， `NotifierService`會叫用 Blazor `OnNotify`的`SynchronizationContext`之外的元件方法。 `InvokeAsync`用來切換至正確的內容，並將轉譯排入佇列。

## <a name="use-key-to-control-the-preservation-of-elements-and-components"></a>使用\@金鑰來控制元素和元件的保留

當轉譯專案或元件的清單，以及後續變更的專案或元件時，Blazor 的比較演算法必須決定哪些先前的專案或元件可以保留，以及模型物件應如何對應至這些專案。 一般來說，此程式是自動的，可以忽略，但在某些情況下，您可能會想要控制進程。

參考下列範例：

```csharp
@foreach (var person in People)
{
    <DetailsEditor Details="person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

`People`集合的內容可能會隨著插入、刪除或重新排序的專案而變更。 當元件 rerenders 時， `<DetailsEditor>`元件可能會變更以接收不同`Details`的參數值。 這可能會導致比預期更複雜的 rerendering。 在某些情況下，rerendering 可能會導致可見的行為差異，例如失去元素的焦點。

您可以使用`@key`指示詞屬性來控制對應進程。 `@key`導致比較演算法根據索引鍵的值，保證保留元素或元件：

```csharp
@foreach (var person in People)
{
    <DetailsEditor @key="person" Details="person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

當集合變更時，比較演算法會保留實例和`person`實例`<DetailsEditor>`之間的關聯： `People`

* 如果從清單中刪除，則只會從 UI `<DetailsEditor>`移除對應的實例。 `People` `Person` 其他實例則保持不變。
* 如果在清單中的某個位置插入，則會在對應`<DetailsEditor>`的位置插入一個新的實例。 `Person` 其他實例則保持不變。
* 如果`Person`重新排序專案，則會保留對應`<DetailsEditor>`的實例，並在 UI 中重新排序。

在某些情況下，使用`@key`可將 rerendering 的複雜性降到最低，並避免 DOM 的具狀態部分可能發生的問題，例如焦點位置。

> [!IMPORTANT]
> 索引鍵在每個容器元素或元件的本機。 金鑰不會在檔之間進行全域比較。

### <a name="when-to-use-key"></a>使用\@金鑰的時機

一般來說，每當轉譯清單（例如`@key` ， `@foreach`在區塊中），而且有適合的值來定義時，就有合理的`@key`使用方式。

當物件變更時`@key` ，您也可以使用來防止 Blazor 保留元素或元件子樹：

```cshtml
<div @key="currentPerson">
    ... content that depends on currentPerson ...
</div>
```

如果`@currentPerson`變更`<div>` ，attribute 指示詞會強制 Blazor 捨棄整個及其下階，並使用新的元素和元件重建 UI 中的子樹。 `@key` 如果您需要保證變更時`@currentPerson`不會保留任何 UI 狀態，這會很有用。

### <a name="when-not-to-use-key"></a>不使用\@金鑰的時機

與`@key`比較時，會產生效能成本。 效能成本並不大，但只會`@key`指定控制元素或元件保留規則是否能讓應用程式受益。

`@key`即使未使用，Blazor 也會盡可能保留子項目和元件實例。 使用的唯一優點是`@key`控制模型實例*如何*對應至保留的元件實例，而不是用來選取對應的比較演算法。

### <a name="what-values-to-use-for-key"></a>要用於金鑰的\@值

一般來說，提供下列其中一種類型的值是合理的`@key`：

* 模型物件實例（例如，如先前`Person`範例所示的實例）。 這可確保根據物件參考的相等性進行保留。
* 唯一識別碼（例如，、或`int` `Guid`類型`string`的主要索引鍵值）。

請確定用於`@key`的值不會造成衝突。 如果在相同的父元素中偵測到衝突值，Blazor 會擲回例外狀況，因為它無法以決定性的方式將舊專案或元件對應到新的專案或元件。 只使用不同的值，例如物件實例或主鍵值。

## <a name="lifecycle-methods"></a>生命週期方法

`OnInitializedAsync`並`OnInitialized`執行程式碼，以初始化元件。 若要執行非同步作業，請`OnInitializedAsync`在作業`await`上使用和關鍵字：

```csharp
protected override async Task OnInitializedAsync()
{
    await ...
}
```

如需同步操作，請`OnInitialized`使用：

```csharp
protected override void OnInitialized()
{
    ...
}
```

`OnParametersSetAsync`當`OnParametersSet`元件已從其父系接收參數，並將值指派給屬性時，會呼叫和。 這些方法會在元件初始化之後和每次呈現元件時執行：

```csharp
protected override async Task OnParametersSetAsync()
{
    await ...
}
```

```csharp
protected override void OnParametersSet()
{
    ...
}
```

`OnAfterRenderAsync`在`OnAfterRender`元件完成呈現之後，會呼叫和。 此時會填入元素和元件參考。 使用此階段來執行使用轉譯內容的其他初始化步驟，例如啟用在轉譯的 DOM 元素上操作的協力廠商 JavaScript 程式庫。

`OnAfterRender`在*伺服器上進行預呈現時，不會呼叫。*

`OnAfterRenderAsync`和`firstRender` 的`OnAfterRender`參數為：

* 當第`true`一次叫用元件實例時，將設為。
* 確保初始化工作只會執行一次。

```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        await ...
    }
}
```

```csharp
protected override void OnAfterRender(bool firstRender)
{
    if (firstRender)
    {
        ...
    }
}
```

### <a name="handle-incomplete-async-actions-at-render"></a>處理轉譯時的未完成非同步動作

在呈現元件之前，在生命週期事件中執行的非同步動作可能尚未完成。 當生命週期`null`方法正在執行時，物件可能會或未完全填入資料。 提供轉譯邏輯，以確認物件已初始化。 當物件為`null`時，呈現預留位置 UI 專案（例如，載入訊息）。

在 Blazor 範本的`OnInitializedAsync` `forecasts`元件中，會覆寫為 asychronously 接收預測資料（）。 `FetchData` 當`forecasts` 為`null`時，會向使用者顯示載入訊息。 在所`Task` `OnInitializedAsync`傳回的完成之後，元件會以更新的狀態重新顯示。

*Pages/FetchData.razor*：

[!code-cshtml[](components/samples_snapshot/3.x/FetchData.razor?highlight=9)]

### <a name="execute-code-before-parameters-are-set"></a>在設定參數之前執行程式碼

`SetParameters`在設定參數之前，可以覆寫以執行程式碼：

```csharp
public override void SetParameters(ParameterView parameters)
{
    ...

    base.SetParameters(parameters);
}
```

如果`base.SetParameters`未叫用，自訂程式碼就可以任何需要的方式解讀傳入的參數值。 例如，傳入的參數不需要指派給類別的屬性。

### <a name="suppress-refreshing-of-the-ui"></a>隱藏 UI 的重新整理

`ShouldRender`可以覆寫以隱藏 UI 的重新整理。 如果執行`true`傳回，則會重新整理 UI。 `ShouldRender`即使已覆寫，元件一律會一開始呈現。

```csharp
protected override bool ShouldRender()
{
    var renderUI = true;

    return renderUI;
}
```

## <a name="component-disposal-with-idisposable"></a>使用 IDisposable 的元件處置

如果元件<xref:System.IDisposable>會執行，則會在從 UI 中移除元件時呼叫[Dispose 方法](/dotnet/standard/garbage-collection/implementing-dispose)。 下列元件會使用`@implements IDisposable` `Dispose`和方法：

```csharp
@using System
@implements IDisposable

...

@code {
    public void Dispose()
    {
        ...
    }
}
```

## <a name="routing"></a>路由

Blazor 中的路由是藉由將路由範本提供給應用程式中每個可存取的元件來達成。

編譯含有`@page`指示詞的 Razor 檔案時，系統會<xref:Microsoft.AspNetCore.Mvc.RouteAttribute>指定路由範本給產生的類別。 在執行時間，路由器會尋找具有的`RouteAttribute`元件類別，並轉譯哪個元件具有符合所要求 URL 的路由範本。

多個路由範本可以套用至元件。 下列元件會回應和`/BlazorRoute` `/DifferentBlazorRoute`的要求：

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/BlazorRoute.razor?name=snippet_BlazorRoute)]

## <a name="route-parameters"></a>路由參數

元件可以從指示詞中`@page`提供的路由範本接收路由參數。 路由器會使用路由參數來填入對應的元件參數。

*路由參數元件*：

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/RouteParameter.razor?name=snippet_RouteParameter)]

不支援選擇性參數，因此上述`@page`範例中會套用兩個指示詞。 第一個則允許不使用參數導覽至元件。 第二`@page`個指示詞`{text}`會採用 route 參數，並將值`Text`指派給屬性。

## <a name="base-class-inheritance-for-a-code-behind-experience"></a>「程式碼後置」體驗的基類繼承

元件檔案會將 HTML 標籤C#和處理常式代碼混合在同一個檔案中。 `@inherits`指示詞可用於提供具有「程式碼後置」體驗的 Blazor apps，以分隔元件標記與處理常式代碼。

[範例應用程式](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/)會顯示元件如何繼承基類， `BlazorRocksBase`以提供元件的屬性和方法。

*Pages/BlazorRocks. razor*：

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/BlazorRocks.razor?name=snippet_BlazorRocks)]

*BlazorRocksBase.cs*：

[!code-csharp[](common/samples/3.x/BlazorSample/Pages/BlazorRocksBase.cs)]

基類應該衍生自`ComponentBase`。

## <a name="import-components"></a>匯入元件

以 Razor 撰寫之元件的命名空間是以為基礎：

* 專案的`RootNamespace`。
* 從專案根到元件的路徑。 例如， `ComponentsSample/Pages/Index.razor`位於命名空間`ComponentsSample.Pages`中。 元件會C#遵循名稱系結規則。 在*ComponentsSample* *的情況下，相同*資料夾、*分頁*和父資料夾中的所有元件都在範圍內。

您可以使用 Razor 的[ \@using](xref:mvc/views/razor#using)指示詞，將不同命名空間中定義的元件帶入範圍中。

`NavMenu.razor`如果資料夾`ComponentsSample/Shared/`中有另一個元件，則可以在中`Index.razor`使用此元件，並搭配下列`@using`語句：

```cshtml
@using ComponentsSample.Shared

This is the Index page.

<NavMenu></NavMenu>
```

元件也可以使用其完整名稱來參考，這樣就不再需要[ \@using](xref:mvc/views/razor#using)指示詞：

```cshtml
This is the Index page.

<ComponentsSample.Shared.NavMenu></ComponentsSample.Shared.NavMenu>
```

> [!NOTE]
> 不`global::`支援該限定性。
>
> 不支援使用具有`using`別名的語句來`@using Foo = Bar`匯入元件（例如）。
>
> 不支援部分限定的名稱。 例如， `<Shared.NavMenu></Shared.NavMenu>`不支援`@using ComponentsSample`新增和`NavMenu.razor`參考。

## <a name="conditional-html-element-attributes"></a>條件式 HTML 元素屬性

HTML 專案屬性會根據 .NET 值有條件地呈現。 如果值為`false`或`null`，則不會呈現屬性。 如果值為`true`，則會以最小化的方式呈現屬性。

在下列範例中， `IsCompleted` `checked`會判斷是否呈現在專案的標記中：

```cshtml
<input type="checkbox" checked="@IsCompleted" />

@code {
    [Parameter]
    public bool IsCompleted { get; set; }
}
```

如果`IsCompleted` 為`true`，則會將核取方塊轉譯為：

```html
<input type="checkbox" checked />
```

如果`IsCompleted` 為`false`，則會將核取方塊轉譯為：

```html
<input type="checkbox" />
```

如需詳細資訊，請參閱<xref:mvc/views/razor>。

> [!WARNING]
> 當 .NET 類型為`bool`時，某些 HTML 屬性（例如，[按下的 aria](https://developer.mozilla.org/docs/Web/Accessibility/ARIA/Roles/button_role#Toggle_buttons)）無法正常運作。 在這些情況下，請`string`使用型別， `bool`而不是。

## <a name="raw-html"></a>原始 HTML

字串通常會使用 DOM 文位元組點來呈現，這表示它們可能包含的任何標記都會被忽略，並視為常值。 若要轉譯原始 HTML，請將 HTML 內容包裝`MarkupString`在值中。 此值會剖析為 HTML 或 SVG，並插入 DOM 中。

> [!WARNING]
> 轉譯從任何未受信任來源所建立的原始 HTML 會有**安全性風險**，應予以避免！

下列範例顯示如何使用`MarkupString`類型，將靜態 HTML 內容的區塊新增至元件的轉譯輸出：

```html
@((MarkupString)myMarkup)

@code {
    private string myMarkup = 
        "<p class='markup'>This is a <em>markup string</em>.</p>";
}
```

## <a name="templated-components"></a>樣板化元件

樣板化元件是接受一或多個 UI 範本做為參數的元件，然後可以用來做為元件轉譯邏輯的一部分。 樣板化元件可讓您撰寫比一般元件更容易重複使用的較高層級元件。 其中有幾個範例包括：

* 資料表元件，可讓使用者指定資料表標頭、資料列和頁尾的範本。
* 清單元件，可讓使用者指定範本來轉譯清單中的專案。

### <a name="template-parameters"></a>範本參數

樣板化元件是藉由指定一或多個類型`RenderFragment`為或`RenderFragment<T>`的元件參數所定義。 呈現片段代表要呈現的 UI 區段。 `RenderFragment<T>`採用可在叫用轉譯片段時指定的類型參數。

`TableTemplate`成分

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/TableTemplate.razor)]

使用樣板化元件時，可以使用符合參數名稱的子項目來指定範本參數（`TableHeader` `RowTemplate`在下列範例中為）：

```cshtml
<TableTemplate Items="pets">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate>
        <td>@context.PetId</td>
        <td>@context.Name</td>
    </RowTemplate>
</TableTemplate>
```

### <a name="template-context-parameters"></a>範本內容參數

當做元素傳遞之`RenderFragment<T>`類型的元件引數具有名為`context`的隱含參數（例如， `@context.PetId`從上述程式碼範例中），但您可以使用子系`Context`上的屬性來變更參數名稱。元素. 在下列範例中， `RowTemplate`元素的`Context`屬性會指定`pet`參數：

```cshtml
<TableTemplate Items="pets">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate Context="pet">
        <td>@pet.PetId</td>
        <td>@pet.Name</td>
    </RowTemplate>
</TableTemplate>
```

或者，您也可以在`Context` component 元素上指定屬性。 指定`Context`的屬性會套用至所有指定的範本參數。 當您想要指定隱含子內容的內容參數名稱時（不含任何換行的子項目），這會很有用。 在下列範例中， `Context`屬性會出現`TableTemplate`在元素上，並套用至所有範本參數：

```cshtml
<TableTemplate Items="pets" Context="pet">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate>
        <td>@pet.PetId</td>
        <td>@pet.Name</td>
    </RowTemplate>
</TableTemplate>
```

### <a name="generic-typed-components"></a>泛型型別元件

樣板化元件通常會以一般方式輸入。 例如，泛型`ListViewTemplate`元件可以用來呈現`IEnumerable<T>`值。 若要定義泛型元件，請使用[@typeparam](xref:mvc/views/razor#typeparam)指示詞來指定類型參數：

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/ListViewTemplate.razor)]

使用泛型型別元件時，會在可能的情況下推斷型別參數：

```cshtml
<ListViewTemplate Items="pets">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>
```

否則，必須使用符合型別參數名稱的屬性來明確指定型別參數。 在下列範例中， `TItem="Pet"`會指定類型：

```cshtml
<ListViewTemplate Items="pets" TItem="Pet">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>
```

## <a name="cascading-values-and-parameters"></a>級聯的值和參數

在某些情況下，使用[元件參數](#component-parameters)將資料從上階元件傳送到子元件是不方便的，特別是在有數個元件層時。 串聯的值和參數可讓上階元件提供一個值給其所有子系元件，藉此解決這個問題。 級聯的值和參數也會提供一種方法來協調元件。

### <a name="theme-example"></a>主題範例

在範例應用程式的下列範例中， `ThemeInfo`類別會指定主題資訊以向下流動元件階層，讓應用程式中指定部分內的所有按鈕共用相同的樣式。

*UIThemeClasses/ThemeInfo .cs*：

```csharp
public class ThemeInfo
{
    public string ButtonClass { get; set; }
}
```

祖系元件可以使用串聯值元件來提供串聯值。 此`CascadingValue`元件會包裝元件階層的子樹，並提供單一值給該子樹內的所有元件。

例如，範例應用程式會在其中一個應用`ThemeInfo`程式的配置中，將主題資訊（）指定為構成`@Body`屬性版面配置主體之所有元件的串聯參數。 `ButtonClass`在版面配置元件中`btn-success` ，會指派的值。 任何子代元件都可以透過`ThemeInfo`串聯物件使用此屬性。

`CascadingValuesParametersLayout`成分

```cshtml
@inherits LayoutComponentBase
@using BlazorSample.UIThemeClasses

<div class="container-fluid">
    <div class="row">
        <div class="col-sm-3">
            <NavMenu />
        </div>
        <div class="col-sm-9">
            <CascadingValue Value="theme">
                <div class="content px-4">
                    @Body
                </div>
            </CascadingValue>
        </div>
    </div>
</div>

@code {
    private ThemeInfo theme = new ThemeInfo { ButtonClass = "btn-success" };
}
```

為了利用串聯值，元件會使用`[CascadingParameter]`屬性宣告串聯式參數。 串聯式值會依類型系結至串聯式參數。

在範例應用程式中， `CascadingValuesParametersTheme`元件`ThemeInfo`會將串聯值系結至串聯式參數。 參數是用來為元件所顯示的其中一個按鈕設定 CSS 類別。

`CascadingValuesParametersTheme`成分

```cshtml
@page "/cascadingvaluesparameterstheme"
@layout CascadingValuesParametersLayout
@using BlazorSample.UIThemeClasses

<h1>Cascading Values & Parameters</h1>

<p>Current count: @currentCount</p>

<p>
    <button class="btn" @onclick="IncrementCount">
        Increment Counter (Unthemed)
    </button>
</p>

<p>
    <button class="btn @ThemeInfo.ButtonClass" @onclick="IncrementCount">
        Increment Counter (Themed)
    </button>
</p>

@code {
    private int currentCount = 0;

    [CascadingParameter]
    protected ThemeInfo ThemeInfo { get; set; }

    private void IncrementCount()
    {
        currentCount++;
    }
}
```

若要在相同的子樹中串聯多個相同類型的值， `Name`請提供唯一`CascadingValue`的字串給每`CascadingParameter`個元件及其對應的。 在下列範例中，兩`CascadingValue`個元件依名稱串聯`MyCascadingType`不同的實例：

```cshtml
<CascadingValue Value=@ParentCascadeParameter1 Name="CascadeParam1">
    <CascadingValue Value=@ParentCascadeParameter2 Name="CascadeParam2">
        ...
    </CascadingValue>
</CascadingValue>

@code {
    private MyCascadingType ParentCascadeParameter1;

    [Parameter]
    public MyCascadingType ParentCascadeParameter2 { get; set; }

    ...
}
```

在子系元件中，串聯的參數會以名稱從上階元件中對應的串聯值接收其值：

```cshtml
...

@code {
    [CascadingParameter(Name = "CascadeParam1")]
    protected MyCascadingType ChildCascadeParameter1 { get; set; }
    
    [CascadingParameter(Name = "CascadeParam2")]
    protected MyCascadingType ChildCascadeParameter2 { get; set; }
}
```

### <a name="tabset-example"></a>TabSet 範例

串聯式參數也可以讓元件在元件階層之間共同作業。 例如，請考慮範例應用程式中的下列*TabSet*範例。

範例應用程式具有`ITab`可執行 tab 鍵的介面：

[!code-csharp[](common/samples/3.x/BlazorSample/UIInterfaces/ITab.cs)]

元件會使用元件，其中包含數個`Tab`元件： `TabSet` `CascadingValuesParametersTabSet`

[!code-cshtml[](common/samples/3.x/BlazorSample/Pages/CascadingValuesParametersTabSet.razor?name=snippet_TabSet)]

子`Tab`元件不會明確地當做參數傳遞`TabSet`至。 相反地，子`Tab`元件是的子內容`TabSet`的一部分。 不過， `TabSet`仍然需要知道每個`Tab`元件，使其可以呈現標頭和使用中的索引標籤。若要啟用這項協調而不需要額外`TabSet`的程式碼，元件*可以提供本身作為*串聯的值，然後由子代`Tab`元件挑選。

`TabSet`成分

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/TabSet.razor)]

子系`TabSet` `Tab` `TabSet`元件會以串聯式參數的形式捕捉包含的，因此元件會將自己加入至索引標籤作用中的和座標。 `Tab`

`Tab`成分

[!code-cshtml[](common/samples/3.x/BlazorSample/Components/Tab.razor)]

## <a name="razor-templates"></a>Razor 範本

轉譯片段可以使用 Razor 範本語法來定義。 Razor 範本是定義 UI 程式碼片段並採用下列格式的方式：

```cshtml
@<{HTML tag}>...</{HTML tag}>
```

下列範例說明如何指定`RenderFragment`和`RenderFragment<T>`值，並直接在元件中呈現範本。 轉譯片段也可以當做引數傳遞至樣板[化元件](#templated-components)。

```cshtml
@timeTemplate

@petTemplate(new Pet { Name = "Rex" })

@code {
    private RenderFragment timeTemplate = @<p>The time is @DateTime.Now.</p>;
    private RenderFragment<Pet> petTemplate = 
        (pet) => @<p>Your pet's name is @pet.Name.</p>;

    private class Pet
    {
        public string Name { get; set; }
    }
}
```

上述程式碼的轉譯輸出：

```html
<p>The time is 10/04/2018 01:26:52.</p>

<p>Your pet's name is Rex.</p>
```

## <a name="manual-rendertreebuilder-logic"></a>手動 RenderTreeBuilder 邏輯

`Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder`提供操作元件和元素的方法，包括在程式碼中C#手動建立元件。

> [!NOTE]
> 使用來`RenderTreeBuilder`建立元件是一個先進的案例。 格式不正確的元件（例如，未封閉的標記標記）可能會導致未定義的行為。

請考慮下列`PetDetails`元件，它可以手動內建在另一個元件中：

```cshtml
<h2>Pet Details Component</h2>

<p>@PetDetailsQuote</p>

@code
{
    [Parameter]
    public string PetDetailsQuote { get; set; }
}
```

在下列範例中， `CreateComponent`方法中的迴圈會產生三個`PetDetails`元件。 呼叫`RenderTreeBuilder`方法來建立元件（`OpenComponent`和`AddAttribute`）時，序號是源程式碼號。 Blazor 差異演算法依賴對應于不同程式程式碼的序號，而不是相異的呼叫調用。 使用`RenderTreeBuilder`方法建立元件時，硬式編碼序號的引數。 **使用計算或計數器來產生序號可能會導致效能不佳。** 如需詳細資訊，請參閱[序號與程式程式碼號，而不是執行順序](#sequence-numbers-relate-to-code-line-numbers-and-not-execution-order)一節。

`BuiltContent`成分

```cshtml
@page "/BuiltContent"

<h1>Build a component</h1>

@CustomRender

<button type="button" @onclick="RenderComponent">
    Create three Pet Details components
</button>

@code {
    private RenderFragment CustomRender { get; set; }
    
    private RenderFragment CreateComponent() => builder =>
    {
        for (var i = 0; i < 3; i++) 
        {
            builder.OpenComponent(0, typeof(PetDetails));
            builder.AddAttribute(1, "PetDetailsQuote", "Someone's best friend!");
            builder.CloseComponent();
        }
    };    
    
    private void RenderComponent()
    {
        CustomRender = CreateComponent();
    }
}
```

> !WARNING@No__t-0 中的類型允許處理轉譯作業的*結果*。 這些是 Blazor framework 執行的內部詳細資料。 這些類型應該被視為不*穩定*，未來的版本可能會變更。

### <a name="sequence-numbers-relate-to-code-line-numbers-and-not-execution-order"></a>序號與程式程式碼號相關，而不是執行順序

一律`.razor`會編譯 Blazor 檔案。 這可能是的絕佳優點`.razor` ，因為編譯步驟可以用來插入可在執行時間改善應用程式效能的資訊。

這些改良功能的重要範例包括*序號*。 序號會向運行時程表示輸出來自哪些不同和已排序的程式程式碼。 執行時間會使用這項資訊，以線性時間產生有效率的樹狀差異，這比一般樹狀結構的差異演算法通常還能快得多。

請考慮下列簡單`.razor`的檔案：

```cshtml
@if (someFlag)
{
    <text>First</text>
}

Second
```

上述程式碼會編譯成如下所示的內容：

```csharp
if (someFlag)
{
    builder.AddContent(0, "First");
}

builder.AddContent(1, "Second");
```

當程式碼第一次執行時，如果`someFlag`是`true`，則產生器會接收：

| 序列 | Type      | Data   |
| :------: | --------- | :----: |
| 0        | Text node | 第一個  |
| 1        | Text node | 第二個 |

想像一下， `false`會變成，然後再次呈現標記。 `someFlag` 這次，產生器會接收：

| 序列 | Type       | Data   |
| :------: | ---------- | :----: |
| 1        | Text node  | 第二個 |

當執行時間執行 diff 時，會看到順序`0`中的專案已移除，因此它會產生下列簡單的*編輯腳本*：

* 移除第一個文位元組點。

#### <a name="what-goes-wrong-if-you-generate-sequence-numbers-programmatically"></a>當您以程式設計方式產生序號時，會發生什麼錯誤

想像一下，您會改為撰寫下列轉譯樹產生器邏輯：

```csharp
var seq = 0;

if (someFlag)
{
    builder.AddContent(seq++, "First");
}

builder.AddContent(seq++, "Second");
```

現在，第一個輸出是：

| 序列 | Type      | Data   |
| :------: | --------- | :----: |
| 0        | Text node | 第一個  |
| 1        | Text node | 第二個 |

此結果與先前的案例相同，因此不會有負面問題存在。 `someFlag``false`在第二個轉譯上，輸出為：

| 序列 | Type      | Data   |
| :------: | --------- | ------ |
| 0        | Text node | 第二個 |

這次，diff 演算法發現發生了*兩*項變更，而演算法會產生下列編輯腳本：

* 將第一個文位元組點的值變更為`Second`。
* 移除第二個文位元組點。

產生序號已遺失關於`if/else`分支和迴圈在原始程式碼中出現位置的所有實用資訊。 這會導致差異**兩倍，但前提**是之前。

這是一個簡單的範例。 在具有複雜和深層嵌套結構的更真實案例中，尤其是使用迴圈時，效能成本會更嚴重。 Diff 演算法不會立即識別已插入或移除的迴圈區塊或分支，而是必須在轉譯樹狀結構中進行深入的遞迴，而且通常會建立更長的編輯腳本，因為它 misinformed 了新舊結構的方式。相互關聯。

#### <a name="guidance-and-conclusions"></a>指引和結論

* 如果序號是動態產生的，應用程式效能會受到影響。
* 架構無法在執行時間自動建立自己的序號，因為必要的資訊不存在，除非是在編譯時期加以捕捉。
* 請勿撰寫長時間區塊的手動執行`RenderTreeBuilder`邏輯。 偏好`.razor`使用檔案，並允許編譯器處理序號。 如果您`RenderTreeBuilder`無法避免手動邏輯，請將長塊的程式碼分割成較小的片段，以`OpenRegion` / `CloseRegion`呼叫。 每個區域都有自己的序號個別空間，因此您可以在每個區域內從零（或任何其他任一數字）重新開機。
* 如果序號已硬式編碼，則 diff 演算法只會要求序號增加值。 起始值和間距無關。 一個合法的選項是使用程式程式碼號做為序號，或從零開始，並以一個或數百個（或任何慣用的間隔）來增加。 
* Blazor 會使用序號，而其他樹狀結構比較的 UI 架構則不會使用它們。 使用序號時，比較速度會更快，而且 Blazor 具有可自動處理序號的編譯步驟，讓開發人員撰寫`.razor`檔案。

## <a name="localization"></a>當地語系化

Blazor 伺服器應用程式是使用[當地語系化中介軟體](xref:fundamentals/localization#localization-middleware)來當地語系化。 中介軟體會為從應用程式要求資源的使用者選取適當的文化特性。

您可以使用下列其中一種方法來設定文化特性：

* [Cookie](#cookies)
* [提供 UI 以選擇文化特性](#provide-ui-to-choose-the-culture)

如需詳細資訊與範例，請參閱<xref:fundamentals/localization>。

### <a name="cookies"></a>Cookie

當地語系化文化特性 cookie 可以保存使用者的文化特性。 Cookie 是由`OnGet`應用程式的主機頁面（*Pages/主機. cshtml .cs*）的方法所建立。 當地語系化中介軟體會在後續要求中讀取 cookie，以設定使用者的文化特性。 

使用 cookie 可確保 WebSocket 連接可以正確地傳播文化特性。 如果當地語系化架構是以 URL 路徑或查詢字串為基礎，配置可能無法使用 Websocket，因而無法保存文化特性。 因此，建議的方法是使用當地語系化文化特性 cookie。

如果文化特性保存在當地語系化 cookie 中，任何技術都可以用來指派文化特性。 如果應用程式已建立伺服器端 ASP.NET Core 的當地語系化配置，請繼續使用應用程式的現有當地語系化基礎結構，並在應用程式的配置內設定當地語系化文化特性 cookie。

下列範例顯示如何在可由當地語系化中介軟體讀取的 cookie 中設定目前的文化特性。 在 Blazor 伺服器應用程式中，使用下列內容建立*Pages/主機. cshtml*檔案：

```csharp
public class HostModel : PageModel
{
    public void OnGet()
    {
        HttpContext.Response.Cookies.Append(
            CookieRequestCultureProvider.DefaultCookieName,
            CookieRequestCultureProvider.MakeCookieValue(
                new RequestCulture(
                    CultureInfo.CurrentCulture,
                    CultureInfo.CurrentUICulture)));
    }
}
```

當地語系化會在應用程式中處理：

1. 瀏覽器會將初始 HTTP 要求傳送至應用程式。
1. 文化特性是由當地語系化中介軟體所指派。
1. *_Host*中的 `OnGet` 方法會在 cookie 中保存文化特性，做為回應的一部分。
1. 瀏覽器會開啟 WebSocket 連線，以建立互動式 Blazor 伺服器會話。
1. 當地語系化中介軟體會讀取 cookie 並指派文化特性。
1. Blazor 伺服器會話的開頭是正確的文化特性。

## <a name="provide-ui-to-choose-the-culture"></a>提供 UI 以選擇文化特性

為了提供允許使用者選取文化特性的 UI，建議使用以重新*導向為基礎的方法*。 此程式類似于在使用者嘗試存取安全資源&mdash;時，會將使用者重新導向至登入頁面，然後重新導向至原始資源的情況下，在 web 應用程式中發生的情況。 

應用程式會透過重新導向至控制器的方式，來保存使用者選取的文化特性。 控制器會將使用者選取的文化特性設定為 cookie，並將使用者重新導向回到原始 URI。

在伺服器上建立 HTTP 端點，以在 cookie 中設定使用者選取的文化特性，並執行重新導向回到原始 URI：

```csharp
[Route("[controller]/[action]")]
public class CultureController : Controller
{
    public IActionResult SetCulture(string culture, string redirectUri)
    {
        if (culture != null)
        {
            HttpContext.Response.Cookies.Append(
                CookieRequestCultureProvider.DefaultCookieName,
                CookieRequestCultureProvider.MakeCookieValue(
                    new RequestCulture(culture)));
        }

        return LocalRedirect(redirectUri);
    }
}
```

> [!WARNING]
> `LocalRedirect`使用動作結果來防止開啟重新導向攻擊。 如需詳細資訊，請參閱<xref:security/preventing-open-redirects>。

下列元件顯示當使用者選取文化特性時，如何執行初始重新導向的範例：

```cshtml
@inject NavigationManager NavigationManager

<h3>Select your language</h3>

<select @onchange="OnSelected">
    <option>Select...</option>
    <option value="en-US">English</option>
    <option value="fr-FR">Français</option>
</select>

@code {
    private double textNumber;

    private void OnSelected(ChangeEventArgs e)
    {
        var culture = (string)e.Value;
        var uri = new Uri(NavigationManager.Uri())
            .GetComponents(UriComponents.PathAndQuery, UriFormat.Unescaped);
        var query = $"?culture={Uri.EscapeDataString(culture)}&" +
            $"redirectUri={Uri.EscapeDataString(uri)}";

        NavigationManager.NavigateTo("/Culture/SetCulture" + query, forceLoad: true);
    }
}
```

### <a name="use-net-localization-scenarios-in-blazor-apps"></a>在 Blazor apps 中使用 .NET 當地語系化案例

在 Blazor apps 中，可以使用下列 .NET 當地語系化和全球化案例：

* .NET 的資源系統
* 特定文化特性的數位和日期格式

Blazor 的`@bind`功能會根據使用者目前的文化特性來執行全球化。 如需詳細資訊，請參閱[資料](#data-binding)系結一節。

目前支援一組有限的 ASP.NET Core 當地語系化案例：

* `IStringLocalizer<>`Blazor 應用程式*支援*。
* `IHtmlLocalizer<>`、 `IViewLocalizer<>`和資料批註當地語系化 ASP.NET Core MVC 案例中，而且**不支援**Blazor 應用程式。

如需詳細資訊，請參閱<xref:fundamentals/localization>。

## <a name="scalable-vector-graphics-svg-images"></a>可擴充向量圖形（SVG）影像

由於 Blazor 會轉譯 HTML，因此可透過`<img>`標記支援瀏覽器支援的影像，包括可擴充的向量圖形（svg）影像（*svg*）：

```html
<img alt="Example image" src="some-image.svg" />
```

同樣地，樣式表單檔案（ *.css*）的 CSS 規則也支援 SVG 影像：

```css
.my-element {
    background-image: url("some-image.svg");
}
```

不過，在所有案例中不支援內嵌 SVG 標記。 如果您將`<svg>`標記直接放入元件檔案（*razor*），則會支援基本映射轉譯，但尚不支援許多先進的案例。 例如， `<use>`目前未遵守標記，而且`@bind`無法與某些 SVG 標記搭配使用。 我們希望在未來的版本中解決這些限制。

## <a name="additional-resources"></a>其他資源

* <xref:security/blazor/server>&ndash;包含有關建立 Blazor 伺服器應用程式的指導方針，必須對抗資源耗盡。
