---
title: 在 ASP.NET Core 中使用託管服務的背景工作
author: rick-anderson
description: 了解如何在 ASP.NET Core 中使用託管服務實作背景工作。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/10/2020
uid: fundamentals/host/hosted-services
ms.openlocfilehash: d3f409170eedd281fd7608c4b9835bf9443c49b0
ms.sourcegitcommit: 9a129f5f3e31cc449742b164d5004894bfca90aa
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/06/2020
ms.locfileid: "78666198"
---
# <a name="background-tasks-with-hosted-services-in-aspnet-core"></a>在 ASP.NET Core 中使用託管服務的背景工作

[Jeow Li Huan](https://github.com/huan086)

::: moniker range=">= aspnetcore-3.0"

在 ASP.NET Core 中，背景工作可實作為「託管服務」。 託管服務是具有背景工作邏輯的類別，可實作 <xref:Microsoft.Extensions.Hosting.IHostedService> 介面。 本主題提供三個託管服務範例：

* 在計時器上執行的背景工作。
* 啟動[具範圍服務](xref:fundamentals/dependency-injection#service-lifetimes)的託管服務。 已設定範圍的服務可以使用相依性[插入（DI）](xref:fundamentals/dependency-injection)。
* 以循序方式執行的排入佇列背景工作。

[檢視或下載範例程式碼](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/host/hosted-services/samples/) \(英文\) ([如何下載](xref:index#how-to-download-a-sample))

## <a name="worker-service-template"></a>背景工作服務範本

ASP.NET Core 背景工作服務範本提供撰寫長期執行服務應用程式的起點。 從背景工作角色服務範本建立的應用程式會在其專案檔中指定背景工作角色 SDK：

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

使用範本作為裝載服務應用程式的基礎：

[!INCLUDE[](~/includes/worker-template-instructions.md)]

## <a name="package"></a>Package

以背景工作角色服務範本為基礎的應用程式會使用 `Microsoft.NET.Sdk.Worker` SDK，並具有對[Microsoft Extensions. 裝載](https://www.nuget.org/packages/Microsoft.Extensions.Hosting)封裝的明確套件參考。 例如，請參閱範例應用程式的專案檔（*BackgroundTasksSample .csproj*）。

若是使用 `Microsoft.NET.Sdk.Web` SDK 的 web 應用程式，則會隱含地從共用架構參考[Microsoft Extensions. 裝載](https://www.nuget.org/packages/Microsoft.Extensions.Hosting)套件。 應用程式的專案檔中不需要明確的套件參考。

## <a name="ihostedservice-interface"></a>IHostedService 介面

<xref:Microsoft.Extensions.Hosting.IHostedService> 介面會針對主機所管理的物件定義兩種方法：

* [StartAsync （CancellationToken）](xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*) &ndash; `StartAsync` 包含啟動背景工作的邏輯。 *在之前*呼叫 `StartAsync`：

  * 應用程式的要求處理管線已設定（`Startup.Configure`）。
  * 伺服器已啟動並[IApplicationLifetime。 ApplicationStarted](xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime.ApplicationStarted*)會觸發。

  您可以變更預設行為，以便在應用程式的管線已設定且呼叫 `ApplicationStarted` 之後，託管服務的 `StartAsync` 執行。 若要變更預設行為，請在呼叫 `ConfigureWebHostDefaults`之後，新增託管服務（在下列範例中`VideosWatcher`）：

  ```csharp
  using Microsoft.AspNetCore.Hosting;
  using Microsoft.Extensions.DependencyInjection;
  using Microsoft.Extensions.Hosting;

  public class Program
  {
      public static void Main(string[] args)
      {
          CreateHostBuilder(args).Build().Run();
      }

      public static IHostBuilder CreateHostBuilder(string[] args) =>
          Host.CreateDefaultBuilder(args)
              .ConfigureWebHostDefaults(webBuilder =>
              {
                  webBuilder.UseStartup<Startup>();
              })
              .ConfigureServices(services =>
              {
                  services.AddHostedService<VideosWatcher>();
              });
  }
  ```

* 當主機執行正常關機時，會觸發[StopAsync （CancellationToken）](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*) &ndash;。 `StopAsync` 包含用來結束背景工作的邏輯。 實作 <xref:System.IDisposable> 和 [完成項 (解構函式)](/dotnet/csharp/programming-guide/classes-and-structs/destructors) 以處置任何非受控的資源。

  取消權杖有五秒的逾時預設值，以表示關機程序應該不再順利。 在權杖上要求取消時：

  * 應終止應用程式正在執行的任何剩餘背景作業。
  * 在 `StopAsync` 中呼叫的任何方法應立即傳回。

  不過，不會在要求取消後直接放棄工作&mdash;呼叫者會等待所有工作完成。

  如果應用程式意外關閉 (例如，應用程式的處理序失敗)，可能不會呼叫 `StopAsync`。 因此，任何在 `StopAsync` 中所呼叫方法或所執行作業可能不會發生。

  若要延長預設的五秒鐘關機逾時，請設定：

  * <xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> (使用泛型主機時)。 如需詳細資訊，請參閱 <xref:fundamentals/host/generic-host#shutdown-timeout>。
  * 使用 Web 主機時，關機逾時會裝載組態設定。 如需詳細資訊，請參閱 <xref:fundamentals/host/web-host#shutdown-timeout>。

託管服務會在應用程式啟動時隨即啟動，然後在應用程式關閉時正常關閉。 如果在背景工作執行期間擲回錯誤，即使未呼叫 `Dispose`，也應該呼叫 `StopAsync`。

## <a name="backgroundservice-base-class"></a>BackgroundService 基類

<xref:Microsoft.Extensions.Hosting.BackgroundService> 是用來執行長時間執行 <xref:Microsoft.Extensions.Hosting.IHostedService>的基類。

呼叫[ExecuteAsync （CancellationToken）](xref:Microsoft.Extensions.Hosting.BackgroundService.ExecuteAsync*)以執行背景服務。 此實作為傳回的 <xref:System.Threading.Tasks.Task>，代表背景服務的整個存留期。 在[ExecuteAsync 變成非同步](https://github.com/dotnet/extensions/issues/2149)（例如藉由呼叫 `await`）之前，不會再啟動任何進一步的服務。 請避免在 `ExecuteAsync`中執行長時間的封鎖初始化工作。 [StopAsync （CancellationToken）](xref:Microsoft.Extensions.Hosting.BackgroundService.StopAsync*)中的主機區塊，等待 `ExecuteAsync` 完成。

呼叫[IHostedService. StopAsync](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*)時，會觸發解除標記。 您的 `ExecuteAsync` 的執行應該會在引發解除標記時立即完成，以便正常地關閉服務。 否則，服務強制會在關機時間關閉。 如需詳細資訊，請參閱[IHostedService 介面](#ihostedservice-interface)一節。

## <a name="timed-background-tasks"></a>計時背景工作

計時背景工作使用 [System.Threading.Timer](xref:System.Threading.Timer) 類別。 此計時器會觸發工作的 `DoWork` 方法。 計時器已在 `StopAsync` 停用，並會在處置服務容器時於 `Dispose` 上進行處置：

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/TimedHostedService.cs?name=snippet1&highlight=16-17,34,41)]

<xref:System.Threading.Timer> 不會等待先前執行的 `DoWork` 完成，因此所顯示的方法可能不適用於每個案例。 [連鎖：](xref:System.Threading.Interlocked.Increment*)用來將執行計數器遞增為不可部分完成的作業，以確保多個執行緒不會同時更新 `executionCount`。

服務會在 `IHostBuilder.ConfigureServices` （*Program.cs*）中以 `AddHostedService` 擴充方法註冊：

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet1)]

## <a name="consuming-a-scoped-service-in-a-background-task"></a>在背景工作中使用範圍服務

若要使用[BackgroundService](#backgroundservice-base-class)內的[範圍服務](xref:fundamentals/dependency-injection#service-lifetimes)，請建立範圍。 根據預設，不會針對託管服務建立任何範圍。

範圍背景工作服務包含背景工作的邏輯。 在下例中︰

* 服務是非同步。 `DoWork` 方法會傳回 `Task`。 基於示範目的，`DoWork` 方法會等待10秒的延遲。
* <xref:Microsoft.Extensions.Logging.ILogger> 會插入服務中。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/ScopedProcessingService.cs?name=snippet1)]

託管服務會建立範圍來解析已設定範圍的背景工作服務，以呼叫其 `DoWork` 方法。 `DoWork` 會傳回 `Task`，在 `ExecuteAsync`中等待：

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/ConsumeScopedServiceHostedService.cs?name=snippet1&highlight=19,22-35)]

服務會在 `IHostBuilder.ConfigureServices` （*Program.cs*）中註冊。 託管服務會向 `AddHostedService` 擴充方法註冊：

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet2)]

## <a name="queued-background-tasks"></a>排入佇列背景工作

背景工作佇列是以 .NET 4.x <xref:System.Web.Hosting.HostingEnvironment.QueueBackgroundWorkItem*> （[暫時排程為 ASP.NET Core 的內建](https://github.com/aspnet/Hosting/issues/1280)）為基礎：

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/BackgroundTaskQueue.cs?name=snippet1)]

在下列 `QueueHostedService` 範例中：

* `BackgroundProcessing` 方法會傳回 `Task`，在 `ExecuteAsync`中等待。
* 佇列中的背景工作會在 `BackgroundProcessing`中進行清除作業並執行。
* 在 `StopAsync`中停止服務之前，會等待工作專案。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/QueuedHostedService.cs?name=snippet1&highlight=28-29,33)]

每當在輸入裝置上選取 `w` 金鑰時，`MonitorLoop` 服務就會處理託管服務的佇列工作：

* `IBackgroundTaskQueue` 會插入 `MonitorLoop` 服務中。
* 呼叫 `IBackgroundTaskQueue.QueueBackgroundWorkItem` 以將工作專案排入佇列。
* 工作專案會模擬長時間執行的背景工作：
  * 會執行三個5秒的延遲（`Task.Delay`）。
  * 如果工作已取消，`try-catch` 語句會 <xref:System.OperationCanceledException> 補漏白。

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/MonitorLoop.cs?name=snippet_Monitor&highlight=7,33)]

服務會在 `IHostBuilder.ConfigureServices` （*Program.cs*）中註冊。 託管服務會向 `AddHostedService` 擴充方法註冊：

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet3)]

`MontiorLoop` 會在 `Program.Main`中啟動：

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet4)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

在 ASP.NET Core 中，背景工作可實作為「託管服務」。 託管服務是具有背景工作邏輯的類別，可實作 <xref:Microsoft.Extensions.Hosting.IHostedService> 介面。 本主題提供三個託管服務範例：

* 在計時器上執行的背景工作。
* 啟動[具範圍服務](xref:fundamentals/dependency-injection#service-lifetimes)的託管服務。 已設定範圍的服務可以使用相依性[插入（DI）](xref:fundamentals/dependency-injection)
* 以循序方式執行的排入佇列背景工作。

[檢視或下載範例程式碼](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/host/hosted-services/samples/) \(英文\) ([如何下載](xref:index#how-to-download-a-sample))

## <a name="package"></a>Package

參考 [Microsoft.AspNetCore.App 中繼套件](xref:fundamentals/metapackage-app)，或新增 [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) 套件的套件參考。

## <a name="ihostedservice-interface"></a>IHostedService 介面

託管服務會實作 <xref:Microsoft.Extensions.Hosting.IHostedService> 介面。 此介面針對主機所管理的物件定義兩種方法：

* [StartAsync （CancellationToken）](xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*) &ndash; `StartAsync` 包含啟動背景工作的邏輯。 使用 [Web 主機](xref:fundamentals/host/web-host)時，是在啟動伺服器和觸發 `StartAsync`IApplicationLifetime.ApplicationStarted[ 之後才呼叫 ](xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime.ApplicationStarted*)。 使用 [泛型主機](xref:fundamentals/host/generic-host)時，是在觸發 `StartAsync` 之前呼叫 `ApplicationStarted`。

* 當主機執行正常關機時，會觸發[StopAsync （CancellationToken）](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*) &ndash;。 `StopAsync` 包含用來結束背景工作的邏輯。 實作 <xref:System.IDisposable> 和 [完成項 (解構函式)](/dotnet/csharp/programming-guide/classes-and-structs/destructors) 以處置任何非受控的資源。

  取消權杖有五秒的逾時預設值，以表示關機程序應該不再順利。 在權杖上要求取消時：

  * 應終止應用程式正在執行的任何剩餘背景作業。
  * 在 `StopAsync` 中呼叫的任何方法應立即傳回。

  不過，不會在要求取消後直接放棄工作&mdash;呼叫者會等待所有工作完成。

  如果應用程式意外關閉 (例如，應用程式的處理序失敗)，可能不會呼叫 `StopAsync`。 因此，任何在 `StopAsync` 中所呼叫方法或所執行作業可能不會發生。

  若要延長預設的五秒鐘關機逾時，請設定：

  * <xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> (使用泛型主機時)。 如需詳細資訊，請參閱 <xref:fundamentals/host/generic-host#shutdown-timeout>。
  * 使用 Web 主機時，關機逾時會裝載組態設定。 如需詳細資訊，請參閱 <xref:fundamentals/host/web-host#shutdown-timeout>。

託管服務會在應用程式啟動時隨即啟動，然後在應用程式關閉時正常關閉。 如果在背景工作執行期間擲回錯誤，即使未呼叫 `Dispose`，也應該呼叫 `StopAsync`。

## <a name="timed-background-tasks"></a>計時背景工作

計時背景工作使用 [System.Threading.Timer](xref:System.Threading.Timer) 類別。 此計時器會觸發工作的 `DoWork` 方法。 計時器已在 `StopAsync` 停用，並會在處置服務容器時於 `Dispose` 上進行處置：

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/TimedHostedService.cs?name=snippet1&highlight=15-16,30,37)]

<xref:System.Threading.Timer> 不會等待先前執行的 `DoWork` 完成，因此所顯示的方法可能不適用於每個案例。

服務是在 `Startup.ConfigureServices` 中使用 `AddHostedService` 擴充方法註冊：

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet1)]

## <a name="consuming-a-scoped-service-in-a-background-task"></a>在背景工作中使用範圍服務

若要在 [ 內使用](xref:fundamentals/dependency-injection#service-lifetimes)具範圍服務`IHostedService`，請建立一個範圍。 根據預設，不會針對託管服務建立任何範圍。

範圍背景工作服務包含背景工作的邏輯。 在下列範例中，<xref:Microsoft.Extensions.Logging.ILogger> 會插入至服務：

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/ScopedProcessingService.cs?name=snippet1)]

託管服務會建立範圍來解析範圍背景工作服務，以呼叫其 `DoWork` 方法：

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/ConsumeScopedServiceHostedService.cs?name=snippet1&highlight=29-36)]

這些服務會在 `Startup.ConfigureServices` 中註冊。 `IHostedService` 實作是以 `AddHostedService` 擴充方法註冊：

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet2)]

## <a name="queued-background-tasks"></a>排入佇列背景工作

背景工作佇列是根據 .NET Framework 4.x <xref:System.Web.Hosting.HostingEnvironment.QueueBackgroundWorkItem*> （[暫時排程為 ASP.NET Core 的內建](https://github.com/aspnet/Hosting/issues/1280)）：

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/BackgroundTaskQueue.cs?name=snippet1)]

在 `QueueHostedService` 中，佇列中的背景工作會從佇列中清除，並作為 [BackgroundService](#backgroundservice-base-class) 執行，這是實作長時間執行 `IHostedService` 的基底類別：

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/QueuedHostedService.cs?name=snippet1&highlight=21,25)]

這些服務會在 `Startup.ConfigureServices` 中註冊。 `IHostedService` 實作是以 `AddHostedService` 擴充方法註冊：

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet3)]

在索引頁面模型類別中：

* 將 `IBackgroundTaskQueue` 插入至建構函式並指派給 `Queue`。
* 插入 <xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory> 並指派給 `_serviceScopeFactory`。 處理站會用來建立 <xref:Microsoft.Extensions.DependencyInjection.IServiceScope> 的執行個體，可用來在範圍內建立服務。 建立範圍，以便使用應用程式的 `AppDbContext` ([具範圍服務](xref:fundamentals/dependency-injection#service-lifetimes))，在 `IBackgroundTaskQueue` (單一服務) 中寫入資料庫記錄。

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Pages/Index.cshtml.cs?name=snippet1)]

在索引頁面上選取 [新增工作] 按鈕時，就會執行 `OnPostAddTask` 方法。 呼叫 `QueueBackgroundWorkItem` 以將工作專案排入佇列：

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Pages/Index.cshtml.cs?name=snippet2)]

::: moniker-end

## <a name="additional-resources"></a>其他資源

* [在微服務中使用 IHostedService 和 BackgroundService 類別實作背景工作](/dotnet/standard/microservices-architecture/multi-container-microservice-net-applications/background-tasks-with-ihostedservice)
* [在 Azure App Service 中使用 Webjob 執行背景工作](/azure/app-service/webjobs-create)
* <xref:System.Threading.Timer>
