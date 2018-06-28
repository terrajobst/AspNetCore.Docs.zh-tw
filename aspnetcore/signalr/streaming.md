---
title: 使用資料流中 ASP.NET Core SignalR
author: rachelappel
description: ''
monikerRange: '>= aspnetcore-2.1'
ms.author: rachelap
ms.custom: mvc
ms.date: 06/07/2018
uid: signalr/streaming
ms.openlocfilehash: 08ddea4fb83150bab27a9e2685c75ff34565606b
ms.sourcegitcommit: 79b756ea03eae77a716f500ef88253ee9b1464d2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/22/2018
ms.locfileid: "36327489"
---
# <a name="use-streaming-in-aspnet-core-signalr"></a><span data-ttu-id="8801d-102">使用資料流中 ASP.NET Core SignalR</span><span class="sxs-lookup"><span data-stu-id="8801d-102">Use streaming in ASP.NET Core SignalR</span></span>

<span data-ttu-id="8801d-103">由[Brennan 瑜吉](https://github.com/BrennanConroy)</span><span class="sxs-lookup"><span data-stu-id="8801d-103">By [Brennan Conroy](https://github.com/BrennanConroy)</span></span>

<span data-ttu-id="8801d-104">ASP.NET Core SignalR 支援串流伺服器方法傳回值。</span><span class="sxs-lookup"><span data-stu-id="8801d-104">ASP.NET Core SignalR supports streaming return values of server methods.</span></span> <span data-ttu-id="8801d-105">這是適用於其中的資料片段會有一段時間的案例。</span><span class="sxs-lookup"><span data-stu-id="8801d-105">This is useful for scenarios where fragments of data will come in over time.</span></span> <span data-ttu-id="8801d-106">時傳回的值串流至用戶端，每個片段會傳送至用戶端時它會變成立即可用，而非等待變成可用的所有資料。</span><span class="sxs-lookup"><span data-stu-id="8801d-106">When a return value is streamed to the client, each fragment is sent to the client as soon as it becomes available, rather than waiting for all the data to become available.</span></span>

<span data-ttu-id="8801d-107">[檢視或下載範例程式碼](https://github.com/aspnet/Docs/tree/live/aspnetcore/signalr/streaming/sample) \(英文\) ([如何下載](xref:tutorials/index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="8801d-107">[View or download sample code](https://github.com/aspnet/Docs/tree/live/aspnetcore/signalr/streaming/sample) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

## <a name="set-up-the-hub"></a><span data-ttu-id="8801d-108">設定中樞</span><span class="sxs-lookup"><span data-stu-id="8801d-108">Set up the hub</span></span>

<span data-ttu-id="8801d-109">中樞方法會自動變成資料流處理的中樞方法，當它傳回`ChannelReader<T>`或`Task<ChannelReader<T>>`。</span><span class="sxs-lookup"><span data-stu-id="8801d-109">A hub method automatically becomes a streaming hub method when it returns a `ChannelReader<T>` or a `Task<ChannelReader<T>>`.</span></span> <span data-ttu-id="8801d-110">以下是示範串流到用戶端資料的基本概念的範例。</span><span class="sxs-lookup"><span data-stu-id="8801d-110">Below is a sample that shows the basics of streaming data to the client.</span></span> <span data-ttu-id="8801d-111">寫入物件是每當`ChannelReader`該物件會立即傳送至用戶端。</span><span class="sxs-lookup"><span data-stu-id="8801d-111">Whenever an object is written to the `ChannelReader` that object is immediately sent to the client.</span></span> <span data-ttu-id="8801d-112">在結束時，`ChannelReader`完成告訴用戶端的資料流已關閉。</span><span class="sxs-lookup"><span data-stu-id="8801d-112">At the end, the `ChannelReader` is completed to tell the client the stream is closed.</span></span>

> [!NOTE]
> <span data-ttu-id="8801d-113">寫入`ChannelReader`在背景執行緒，然後傳回`ChannelReader`儘速。</span><span class="sxs-lookup"><span data-stu-id="8801d-113">Write to the `ChannelReader` on a background thread and return the `ChannelReader` as soon as possible.</span></span> <span data-ttu-id="8801d-114">其他中樞引動過程將會封鎖直到`ChannelReader`傳回。</span><span class="sxs-lookup"><span data-stu-id="8801d-114">Other hub invocations will be blocked until a `ChannelReader` is returned.</span></span>

[!code-csharp[Streaming hub method](streaming/sample/Hubs/StreamHub.cs?range=10-34)]

## <a name="net-client"></a><span data-ttu-id="8801d-115">.NET 用戶端</span><span class="sxs-lookup"><span data-stu-id="8801d-115">.NET client</span></span>

<span data-ttu-id="8801d-116">`StreamAsChannelAsync`方法`HubConnection`用來叫用的資料流的方法。</span><span class="sxs-lookup"><span data-stu-id="8801d-116">The `StreamAsChannelAsync` method on `HubConnection` is used to invoke a streaming method.</span></span> <span data-ttu-id="8801d-117">將中樞方法的名稱和中樞方法中定義的引數傳遞`StreamAsChannelAsync`。</span><span class="sxs-lookup"><span data-stu-id="8801d-117">Pass the hub method name, and arguments defined in the hub method to `StreamAsChannelAsync`.</span></span> <span data-ttu-id="8801d-118">上的泛型參數`StreamAsChannelAsync<T>`指定的資料流的方法所傳回的物件類型。</span><span class="sxs-lookup"><span data-stu-id="8801d-118">The generic parameter on `StreamAsChannelAsync<T>` specifies the type of objects returned by the streaming method.</span></span> <span data-ttu-id="8801d-119">A`ChannelReader<T>`會傳回從資料流引動過程中，並代表用戶端上的資料流。</span><span class="sxs-lookup"><span data-stu-id="8801d-119">A `ChannelReader<T>` is returned from the stream invocation, and represents the stream on the client.</span></span> <span data-ttu-id="8801d-120">若要讀取資料，常見的模式是迴圈時`WaitToReadAsync`並呼叫`TryRead`資料時使用。</span><span class="sxs-lookup"><span data-stu-id="8801d-120">To read data, a common pattern is to loop over `WaitToReadAsync` and call `TryRead` when data is available.</span></span> <span data-ttu-id="8801d-121">在伺服器上，資料流已關閉或取消語彙基元傳遞至時，迴圈將結束`StreamAsChannelAsync`已取消。</span><span class="sxs-lookup"><span data-stu-id="8801d-121">The loop will end when the stream has been closed by the server, or the cancellation token passed to `StreamAsChannelAsync` is canceled.</span></span>

```csharp
var channel = await hubConnection.StreamAsChannelAsync<int>("Counter", 10, 500, CancellationToken.None);

// Wait asynchronously for data to become available
while (await channel.WaitToReadAsync())
{
    // Read all currently available data synchronously, before waiting for more data
    while (channel.TryRead(out var count))
    {
        Console.WriteLine($"{count}");
    }
}

Console.WriteLine("Streaming completed");
```

## <a name="javascript-client"></a><span data-ttu-id="8801d-122">JavaScript 用戶端</span><span class="sxs-lookup"><span data-stu-id="8801d-122">JavaScript client</span></span>

<span data-ttu-id="8801d-123">JavaScript 用戶端呼叫中樞上的資料流的方法，使用`connection.stream`。</span><span class="sxs-lookup"><span data-stu-id="8801d-123">JavaScript clients call streaming methods on hubs by using `connection.stream`.</span></span> <span data-ttu-id="8801d-124">`stream`方法接受兩個引數：</span><span class="sxs-lookup"><span data-stu-id="8801d-124">The `stream` method accepts two arguments:</span></span>

* <span data-ttu-id="8801d-125">中樞方法的名稱。</span><span class="sxs-lookup"><span data-stu-id="8801d-125">The name of the hub method.</span></span> <span data-ttu-id="8801d-126">下列範例中，在中樞方法的名稱是`Counter`。</span><span class="sxs-lookup"><span data-stu-id="8801d-126">In the following example, the hub method name is `Counter`.</span></span>
* <span data-ttu-id="8801d-127">定義在中樞方法的引數。</span><span class="sxs-lookup"><span data-stu-id="8801d-127">Arguments defined in the hub method.</span></span> <span data-ttu-id="8801d-128">在下列範例中，引數為： 接收的資料流項目和資料流的項目之間的延遲數的計數。</span><span class="sxs-lookup"><span data-stu-id="8801d-128">In the following example, the arguments are: a count for the number of stream items to receive, and the delay between stream items.</span></span>

<span data-ttu-id="8801d-129">`connection.stream` 傳回`IStreamResult`包含`subscribe`方法。</span><span class="sxs-lookup"><span data-stu-id="8801d-129">`connection.stream` returns an `IStreamResult` which contains a `subscribe` method.</span></span> <span data-ttu-id="8801d-130">傳遞`IStreamSubscriber`至`subscribe`並設定`next`， `error`，和`complete`回呼，以取得通知從`stream`引動過程。</span><span class="sxs-lookup"><span data-stu-id="8801d-130">Pass an `IStreamSubscriber` to `subscribe` and set the `next`, `error`, and `complete` callbacks to get notifications from the `stream` invocation.</span></span>

[!code-javascript[Streaming javascript](streaming/sample/wwwroot/js/stream.js?range=19-36)]

<span data-ttu-id="8801d-131">若要結束用戶端呼叫從資料流`dispose`方法`ISubscription`從傳回`subscribe`方法。</span><span class="sxs-lookup"><span data-stu-id="8801d-131">To end the stream from the client call the `dispose` method on the `ISubscription` that is returned from the `subscribe` method.</span></span>

## <a name="related-resources"></a><span data-ttu-id="8801d-132">相關資源</span><span class="sxs-lookup"><span data-stu-id="8801d-132">Related resources</span></span>

* [<span data-ttu-id="8801d-133">中樞</span><span class="sxs-lookup"><span data-stu-id="8801d-133">Hubs</span></span>](xref:signalr/hubs)
* [<span data-ttu-id="8801d-134">.NET 用戶端</span><span class="sxs-lookup"><span data-stu-id="8801d-134">.NET client</span></span>](xref:signalr/dotnet-client)
* [<span data-ttu-id="8801d-135">JavaScript 用戶端</span><span class="sxs-lookup"><span data-stu-id="8801d-135">JavaScript client</span></span>](xref:signalr/javascript-client)
* [<span data-ttu-id="8801d-136">發佈至 Azure</span><span class="sxs-lookup"><span data-stu-id="8801d-136">Publish to Azure</span></span>](xref:signalr/publish-to-azure-web-app)