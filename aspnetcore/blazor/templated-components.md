---
title: ASP.NET Core Blazor 樣板化元件
author: guardrex
description: 瞭解樣板化元件可以如何接受一或多個 UI 範本做為參數，然後用來做為元件轉譯邏輯的一部分。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/18/2020
no-loc:
- Blazor
- SignalR
uid: blazor/templated-components
ms.openlocfilehash: b57e3fe186402723607e90b1628062f602c77632
ms.sourcegitcommit: 9b6e7f421c243963d5e419bdcfc5c4bde71499aa
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/21/2020
ms.locfileid: "79989491"
---
# <a name="aspnet-core-opno-locblazor-templated-components"></a>ASP.NET Core Blazor 樣板化元件

By [Luke Latham](https://github.com/guardrex)和[Daniel Roth](https://github.com/danroth27)

樣板化元件是接受一或多個 UI 範本做為參數的元件，然後可以用來做為元件轉譯邏輯的一部分。 樣板化元件可讓您撰寫比一般元件更容易重複使用的較高層級元件。 其中有幾個範例包括：

* 資料表元件，可讓使用者指定資料表標頭、資料列和頁尾的範本。
* 清單元件，可讓使用者指定範本來轉譯清單中的專案。

## <a name="template-parameters"></a>範本參數

樣板化元件是藉由指定 `RenderFragment` 或 `RenderFragment<T>`類型的一或多個元件參數來定義。 呈現片段代表要呈現的 UI 區段。 `RenderFragment<T>` 會採用可在叫用轉譯片段時指定的類型參數。

`TableTemplate` 元件：

[!code-razor[](common/samples/3.x/BlazorWebAssemblySample/Components/TableTemplate.razor)]

使用樣板化元件時，可以使用符合參數名稱的子專案來指定範本參數（`TableHeader`，並在下列範例中 `RowTemplate`）：

```razor
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

> [!NOTE]
> 在未來的版本中將會支援泛型型別條件約束。 如需詳細資訊，請參閱[允許泛型型別條件約束（dotnet/aspnetcore #8433）](https://github.com/dotnet/aspnetcore/issues/8433)。

## <a name="template-context-parameters"></a>範本內容參數

當做元素傳遞 `RenderFragment<T>` 類型的元件引數具有名為 `context` 的隱含參數（例如，從前面的程式碼範例，`@context.PetId`），但您可以使用子專案上的 `Context` 屬性來變更參數名稱。 在下列範例中，`RowTemplate` 元素的 `Context` 屬性會指定 `pet` 參數：

```razor
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

或者，您也可以指定 component 元素上的 `Context` 屬性。 指定的 `Context` 屬性會套用至所有指定的範本參數。 當您想要指定隱含子內容的內容參數名稱時（不含任何換行的子項目），這會很有用。 在下列範例中，`Context` 屬性會出現在 `TableTemplate` 元素上，並套用至所有範本參數：

```razor
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

## <a name="generic-typed-components"></a>泛型型別元件

樣板化元件通常會以一般方式輸入。 例如，泛型 `ListViewTemplate` 元件可以用來呈現 `IEnumerable<T>` 值。 若要定義泛型元件，請使用[`@typeparam`](xref:mvc/views/razor#typeparam)指示詞來指定類型參數：

[!code-razor[](common/samples/3.x/BlazorWebAssemblySample/Components/ListViewTemplate.razor)]

使用泛型型別元件時，會在可能的情況下推斷型別參數：

```razor
<ListViewTemplate Items="pets">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>
```

否則，必須使用符合型別參數名稱的屬性來明確指定型別參數。 在下列範例中，`TItem="Pet"` 會指定類型：

```razor
<ListViewTemplate Items="pets" TItem="Pet">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>
```
