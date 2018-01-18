---
title: Migrazione da ASP.NET ad ASP.NET Core 2.0
author: isaac2004
description: "Questo documento di riferimento contiene indicazioni sulla migrazione di applicazioni già esistenti in ASP.NET MVC o Web API ad ASP.NET Core 2.0."
keywords: ASP.NET Core, MVC, migrazione
ms.author: scaddie
manager: wpickett
ms.date: 08/27/2017
ms.topic: article
ms.assetid: 3155cc9e-d0c9-424b-886c-35c0ec6f9f4e
ms.technology: aspnet
ms.prod: asp.net-core
uid: migration/proper-to-2x/index
ms.openlocfilehash: 174ce7c99b8b1b65544f931dbb373a4829a64539
ms.sourcegitcommit: 8f42ab93402c1b8044815e1e48d0bb84c81f8b59
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/29/2017
---
# <a name="migrating-from-aspnet-to-aspnet-core-20"></a><span data-ttu-id="f8ba3-104">Migrazione da ASP.NET ad ASP.NET Core 2.0</span><span class="sxs-lookup"><span data-stu-id="f8ba3-104">Migrating from ASP.NET to ASP.NET Core 2.0</span></span>

<span data-ttu-id="f8ba3-105">Di [Isaac Levin](https://isaaclevin.com)</span><span class="sxs-lookup"><span data-stu-id="f8ba3-105">By [Isaac Levin](https://isaaclevin.com)</span></span>

<span data-ttu-id="f8ba3-106">Questo articolo offre una guida di riferimento per la migrazione delle applicazioni ASP.NET ad ASP.NET Core 2.0.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-106">This article serves as a reference guide for migrating ASP.NET applications to ASP.NET Core 2.0.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="f8ba3-107">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="f8ba3-107">Prerequisites</span></span>

* <span data-ttu-id="f8ba3-108">[.NET Core 2.0.0 SDK](https://dot.net/core) o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-108">[.NET Core 2.0.0 SDK](https://dot.net/core) or later.</span></span>

## <a name="target-frameworks"></a><span data-ttu-id="f8ba3-109">Framework di destinazione</span><span class="sxs-lookup"><span data-stu-id="f8ba3-109">Target Frameworks</span></span>
<span data-ttu-id="f8ba3-110">I progetti di ASP.NET Core 2.0 offrono agli sviluppatori la flessibilità necessaria per scegliere .NET Core, .NET Framework o entrambi come destinazione.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-110">ASP.NET Core 2.0 projects offer developers the flexibility of targeting .NET Core, .NET Framework, or both.</span></span> <span data-ttu-id="f8ba3-111">Vedere [Scelta di .NET Core o .NET Framework per le app server](https://docs.microsoft.com/dotnet/standard/choosing-core-framework-server) per determinare quale framework di destinazione è più appropriato.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-111">See [Choosing between .NET Core and .NET Framework for server apps](https://docs.microsoft.com/dotnet/standard/choosing-core-framework-server) to determine which target framework is most appropriate.</span></span>

<span data-ttu-id="f8ba3-112">Quando la destinazione è .NET Framework, i progetti devono fare riferimento a singoli pacchetti NuGet.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-112">When targeting .NET Framework, projects need to reference individual NuGet packages.</span></span>

<span data-ttu-id="f8ba3-113">La scelta di .NET Core come destinazione consente di eliminare numerosi riferimenti espliciti ai pacchetti, grazie al [metapacchetto](xref:fundamentals/metapackage) di ASP.NET 2.0 Core.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-113">Targeting .NET Core allows you to eliminate numerous explicit package references, thanks to the ASP.NET Core 2.0 [metapackage](xref:fundamentals/metapackage).</span></span> <span data-ttu-id="f8ba3-114">Installare il metapacchetto `Microsoft.AspNetCore.All` nel progetto:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-114">Install the `Microsoft.AspNetCore.All` metapackage in your project:</span></span>

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.0" />
</ItemGroup>
```

<span data-ttu-id="f8ba3-115">Quando si usa il metapacchetto, con l'app non viene distribuito alcun pacchetto a cui si fa riferimento nel metapacchetto.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-115">When the metapackage is used, no packages referenced in the metapackage are deployed with the app.</span></span> <span data-ttu-id="f8ba3-116">L'archivio di runtime di .NET Core include questi asset, che vengono precompilati per migliorare le prestazioni.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-116">The .NET Core Runtime Store includes these assets, and they are precompiled to improve performance.</span></span> <span data-ttu-id="f8ba3-117">Vedere le informazioni sul [metapacchetto Microsoft.AspNetCore.All per ASP.NET Core 2. x](xref:fundamentals/metapackage) per maggiori dettagli.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-117">See [Microsoft.AspNetCore.All metapackage for ASP.NET Core 2.x](xref:fundamentals/metapackage) for more detail.</span></span>

## <a name="project-structure-differences"></a><span data-ttu-id="f8ba3-118">Differenze di struttura del progetto</span><span class="sxs-lookup"><span data-stu-id="f8ba3-118">Project structure differences</span></span>
<span data-ttu-id="f8ba3-119">Il formato di file *CSPROJ* è stato semplificato in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-119">The *.csproj* file format has been simplified in ASP.NET Core.</span></span> <span data-ttu-id="f8ba3-120">Alcune modifiche importanti includono:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-120">Some notable changes include:</span></span>
- <span data-ttu-id="f8ba3-121">L'inclusione esplicita dei file non è necessaria affinché i file vengano considerati parte del progetto.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-121">Explicit inclusion of files is not necessary for them to be considered part of the project.</span></span> <span data-ttu-id="f8ba3-122">In questo modo si riduce il rischio di conflitti di merge XML quando si lavora con team di grandi dimensioni.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-122">This reduces the risk of XML merge conflicts when working on large teams.</span></span>
- <span data-ttu-id="f8ba3-123">Non sono presenti riferimenti basati su GUID ad altri progetti e questo migliora la leggibilità dei file.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-123">There are no GUID-based references to other projects, which improves file readability.</span></span>
- <span data-ttu-id="f8ba3-124">Il file può essere modificato senza scaricarlo in Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-124">The file can be edited without unloading it in Visual Studio:</span></span>

    ![Modificare l'opzione CSPROJ del menu di scelta rapida in Visual Studio 2017](_static/EditProjectVs2017.png)

## <a name="globalasax-file-replacement"></a><span data-ttu-id="f8ba3-126">Sostituzione di file Global.asax</span><span class="sxs-lookup"><span data-stu-id="f8ba3-126">Global.asax file replacement</span></span>
<span data-ttu-id="f8ba3-127">In ASP.NET Core è stato introdotto un nuovo meccanismo per l'avvio automatico delle app.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-127">ASP.NET Core introduced a new mechanism for bootstrapping an app.</span></span> <span data-ttu-id="f8ba3-128">Il punto di ingresso per le applicazioni ASP.NET è il file *Global.asax*.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-128">The entry point for ASP.NET applications is the *Global.asax* file.</span></span> <span data-ttu-id="f8ba3-129">Attività quali la configurazione della route e le registrazioni di area e filtro vengono gestite nel file *Global.asax*.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-129">Tasks such as route configuration and filter and area registrations are handled in the *Global.asax* file.</span></span>

[!code-csharp[Main](samples/globalasax-sample.cs)]

<span data-ttu-id="f8ba3-130">Con questo approccio l'applicazione e il server a cui viene distribuita vengono accoppiati in un modo che interferisce con l'implementazione.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-130">This approach couples the application and the server to which it's deployed in a way that interferes with the implementation.</span></span> <span data-ttu-id="f8ba3-131">Al fine di disaccoppiare gli elementi, è stata introdotta la funzionalità [OWIN](http://owin.org/) che offre un modo più semplice di usare più framework insieme.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-131">In an effort to decouple, [OWIN](http://owin.org/) was introduced to provide a cleaner way to use multiple frameworks together.</span></span> <span data-ttu-id="f8ba3-132">OWIN offre una pipeline per aggiungere solo i moduli necessari.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-132">OWIN provides a pipeline to add only the modules needed.</span></span> <span data-ttu-id="f8ba3-133">L'ambiente host accetta una funzione di [avvio](xref:fundamentals/startup) per configurare i servizi e la pipeline delle richieste dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-133">The hosting environment takes a [Startup](xref:fundamentals/startup) function to configure services and the app's request pipeline.</span></span> <span data-ttu-id="f8ba3-134">`Startup` registra un set di middleware con l'applicazione.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-134">`Startup` registers a set of middleware with the application.</span></span> <span data-ttu-id="f8ba3-135">Per ogni richiesta, l'applicazione chiama ognuno dei componenti middleware con il puntatore iniziale di un elenco collegato a un set esistente di gestori.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-135">For each request, the application calls each of the the middleware components with the head pointer of a linked list to an existing set of handlers.</span></span> <span data-ttu-id="f8ba3-136">Ogni componente middleware può aggiungere uno o più gestori alla pipeline di gestione delle richieste.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-136">Each middleware component can add one or more handlers to the request handling pipeline.</span></span> <span data-ttu-id="f8ba3-137">Questa operazione viene eseguita restituendo un riferimento al gestore che rappresenta il nuovo inizio dell'elenco.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-137">This is accomplished by returning a reference to the handler that is the new head of the list.</span></span> <span data-ttu-id="f8ba3-138">Ogni gestore è responsabile della memorizzazione e della chiamata del gestore successivo nell'elenco.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-138">Each handler is responsible for remembering and invoking the next handler in the list.</span></span> <span data-ttu-id="f8ba3-139">Con ASP.NET Core, il punto di ingresso a un'applicazione è `Startup` e non esiste più una dipendenza da *Global.asax*.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-139">With ASP.NET Core, the entry point to an application is `Startup`, and you no longer have a dependency on *Global.asax*.</span></span> <span data-ttu-id="f8ba3-140">Quando si usa OWIN con .NET Framework, usare una pipeline simile alla seguente:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-140">When using OWIN with .NET Framework, use something like the following as a pipeline:</span></span>

[!code-csharp[Main](samples/webapi-owin.cs)]

<span data-ttu-id="f8ba3-141">Ciò consente di configurare le route predefinite e impostare come predefinito XmlSerialization per Json.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-141">This configures your default routes, and defaults to XmlSerialization over Json.</span></span> <span data-ttu-id="f8ba3-142">Aggiungere altro middleware a questa pipeline in base alle esigenze, ad esempio caricamento di servizi, impostazioni di configurazione, file statici e così via.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-142">Add other Middleware to this pipeline as needed (loading services, configuration settings, static files, etc.).</span></span>

<span data-ttu-id="f8ba3-143">ASP.NET Core usa un approccio simile, ma non si basa su OWIN per gestire la voce.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-143">ASP.NET Core uses a similar approach, but doesn't rely on OWIN to handle the entry.</span></span> <span data-ttu-id="f8ba3-144">L'operazione viene invece eseguita usando il metodo *Program.cs* `Main` (simile alle applicazioni console) e `Startup` viene caricato da tale posizione.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-144">Instead, that is done through the *Program.cs* `Main` method (similar to console applications) and `Startup` is loaded through there.</span></span>

[!code-csharp[Main](samples/program.cs)]

<span data-ttu-id="f8ba3-145">`Startup` deve includere un metodo `Configure`.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-145">`Startup` must include a `Configure` method.</span></span> <span data-ttu-id="f8ba3-146">In `Configure` aggiungere il middleware necessario alla pipeline.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-146">In `Configure`, add the necessary middleware to the pipeline.</span></span> <span data-ttu-id="f8ba3-147">Nell'esempio seguente, estratto dal modello di sito Web predefinito, vengono usati vari metodi di estensione per configurare la pipeline con il supporto per:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-147">In the following example (from the default web site template), several extension methods are used to configure the pipeline with support for:</span></span>

* [<span data-ttu-id="f8ba3-148">BrowserLink</span><span class="sxs-lookup"><span data-stu-id="f8ba3-148">BrowserLink</span></span>](http://vswebessentials.com/features/browserlink)
* <span data-ttu-id="f8ba3-149">Pagine di errore</span><span class="sxs-lookup"><span data-stu-id="f8ba3-149">Error pages</span></span>
* <span data-ttu-id="f8ba3-150">File statici</span><span class="sxs-lookup"><span data-stu-id="f8ba3-150">Static files</span></span>
* <span data-ttu-id="f8ba3-151">ASP.NET Core MVC</span><span class="sxs-lookup"><span data-stu-id="f8ba3-151">ASP.NET Core MVC</span></span>
* <span data-ttu-id="f8ba3-152">Identità</span><span class="sxs-lookup"><span data-stu-id="f8ba3-152">Identity</span></span>

[!code-csharp[Main](../../common/samples/WebApplication1/Startup.cs?highlight=8,9,10,14,17,19,21&start=58&end=84)]

<span data-ttu-id="f8ba3-153">L'host e applicazione sono stati disaccoppiati e questo offre la possibilità di passare a una piattaforma diversa in futuro.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-153">The host and application have been decoupled, which provides the flexibility of moving to a different platform in the future.</span></span>

<span data-ttu-id="f8ba3-154">**Nota:** per un riferimento più dettaglio all'avvio e al middleware di ASP.NET Core, vedere l'articolo sull'[avvio in ASP.NET Core](xref:fundamentals/startup)</span><span class="sxs-lookup"><span data-stu-id="f8ba3-154">**Note:** For a more in-depth reference to ASP.NET Core Startup and Middleware, see [Startup in ASP.NET Core](xref:fundamentals/startup)</span></span>

## <a name="storing-configurations"></a><span data-ttu-id="f8ba3-155">Archiviazione delle configurazioni</span><span class="sxs-lookup"><span data-stu-id="f8ba3-155">Storing Configurations</span></span>
<span data-ttu-id="f8ba3-156">ASP.NET supporta le impostazioni di archiviazione.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-156">ASP.NET supports storing settings.</span></span> <span data-ttu-id="f8ba3-157">Tali impostazioni vengono usate, ad esempio, per supportare l'ambiente in cui vengono distribuite le applicazioni.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-157">These setting are used, for example, to support the environment to which the applications were deployed.</span></span> <span data-ttu-id="f8ba3-158">Una prassi comune era archiviare tutte le coppie chiave-valore personalizzate nella sezione `<appSettings>` del file *Web.config*:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-158">A common practice was to store all custom key-value pairs in the `<appSettings>` section of the *Web.config* file:</span></span>

[!code-xml[Main](samples/webconfig-sample.xml)]

<span data-ttu-id="f8ba3-159">Le applicazioni leggono queste impostazioni usando la raccolta `ConfigurationManager.AppSettings` nello spazio dei nomi `System.Configuration`:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-159">Applications read these settings using the `ConfigurationManager.AppSettings` collection in the `System.Configuration` namespace:</span></span>

[!code-csharp[Main](samples/read-webconfig.cs)]

<span data-ttu-id="f8ba3-160">ASP.NET Core è in grado di archiviare i dati di configurazione per l'applicazione in tutti i file e di caricarli come parte dell'avvio automatico del middleware.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-160">ASP.NET Core can store configuration data for the application in any file and load them as part of middleware bootstrapping.</span></span> <span data-ttu-id="f8ba3-161">Il file predefinito usato nei modelli di progetto è *appSettings.json*:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-161">The default file used in the project templates is *appsettings.json*:</span></span>

[!code-json[Main](samples/appsettings-sample.json)]

<span data-ttu-id="f8ba3-162">Il caricamento del file in un'istanza di `IConfiguration` all'interno dell'applicazione viene eseguito in *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-162">Loading this file into an instance of `IConfiguration` inside your application is done in *Startup.cs*:</span></span>

[!code-csharp[Main](samples/startup-builder.cs)]

<span data-ttu-id="f8ba3-163">L'app legge da `Configuration` per ottenere le impostazioni:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-163">The app reads from `Configuration` to get the settings:</span></span>

[!code-csharp[Main](samples/read-appsettings.cs)]

<span data-ttu-id="f8ba3-164">Sono disponibili estensioni di questo approccio che aumentano l'efficacia del processo, ad esempio l'uso dell'[inserimento delle dipendenze](xref:fundamentals/dependency-injection) per caricare un servizio con questi valori.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-164">There are extensions to this approach to make the process more robust, such as using [Dependency Injection](xref:fundamentals/dependency-injection) (DI) to load a service with these values.</span></span> <span data-ttu-id="f8ba3-165">L'approccio con inserimento delle dipendenze offre un set fortemente tipizzato di oggetti di configurazione.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-165">The DI approach provides a strongly-typed set of configuration objects.</span></span>

````csharp
// Assume AppConfiguration is a class representing a strongly-typed version of AppConfiguration section
services.Configure<AppConfiguration>(Configuration.GetSection("AppConfiguration"));
````

<span data-ttu-id="f8ba3-166">**Nota:** per informazioni più dettagliate sulla configurazione di ASP.NET Core, vedere l'articolo sulla [configurazione in ASP.NET Core](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="f8ba3-166">**Note:** For a more in-depth reference to ASP.NET Core configuration, see [Configuration in ASP.NET Core](xref:fundamentals/configuration/index).</span></span>

## <a name="native-dependency-injection"></a><span data-ttu-id="f8ba3-167">Inserimento delle dipendenze nativo</span><span class="sxs-lookup"><span data-stu-id="f8ba3-167">Native Dependency Injection</span></span>
<span data-ttu-id="f8ba3-168">Un obiettivo importante nella compilazione di applicazioni scalabili di grandi dimensioni è l'accoppiamento libero di componenti e servizi.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-168">An important goal when building large, scalable applications is the loose coupling of components and services.</span></span> <span data-ttu-id="f8ba3-169">L'[inserimento delle dipendenze](xref:fundamentals/dependency-injection) è una tecnica comune che consente di raggiungerlo ed è un componente nativo di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-169">[Dependency Injection](xref:fundamentals/dependency-injection) is a popular technique for achieving this, and it is a native component of ASP.NET Core.</span></span>

<span data-ttu-id="f8ba3-170">Nelle applicazioni ASP.NET, gli sviluppatori si affidano a una libreria di terze parti per implementare l'inserimento delle dipendenze.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-170">In ASP.NET applications, developers rely on a third-party library to implement Dependency Injection.</span></span> <span data-ttu-id="f8ba3-171">Una di queste librerie è [Unity](https://github.com/unitycontainer/unity) e fa parte di Modelli e procedure Microsoft.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-171">One such library is [Unity](https://github.com/unitycontainer/unity), provided by Microsoft Patterns & Practices.</span></span> 

<span data-ttu-id="f8ba3-172">Un esempio di configurazione dell'inserimento delle dipendenze con Unity è l'implementazione di `IDependencyResolver` che esegue il wrapping di un oggetto `UnityContainer`:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-172">An example of setting up Dependency Injection with Unity is implementing `IDependencyResolver` that wraps a `UnityContainer`:</span></span>

[!code-csharp[Main](../../../aspnet/web-api/overview/advanced/dependency-injection/samples/sample8.cs)]

<span data-ttu-id="f8ba3-173">Creare un'istanza di `UnityContainer`, registrare il servizio e impostare il sistema di risoluzione delle dipendenze di `HttpConfiguration` sulla nuova istanza di `UnityResolver` per il contenitore:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-173">Create an instance of your `UnityContainer`, register your service, and set the dependency resolver of `HttpConfiguration` to the new instance of `UnityResolver` for your container:</span></span>

[!code-csharp[Main](../../../aspnet/web-api/overview/advanced/dependency-injection/samples/sample9.cs)]

<span data-ttu-id="f8ba3-174">Inserire `IProductRepository` dove necessario:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-174">Inject `IProductRepository` where needed:</span></span>

[!code-csharp[Main](../../../aspnet/web-api/overview/advanced/dependency-injection/samples/sample5.cs)]

<span data-ttu-id="f8ba3-175">Poiché l'inserimento delle dipendenze fa parte di ASP.NET Core, è possibile aggiungere il servizio nel metodo `ConfigureServices` di *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-175">Because Dependency Injection is part of ASP.NET Core, you can add your service in the `ConfigureServices` method of *Startup.cs*:</span></span>

[!code-csharp[Main](samples/configure-services.cs)]

<span data-ttu-id="f8ba3-176">Il repository può essere inserito in qualsiasi posizione, analogamente a Unity.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-176">The repository can be injected anywhere, as was true with Unity.</span></span>

<span data-ttu-id="f8ba3-177">**Nota:** per un riferimento dettagliato all'inserimento delle dipendenze in ASP.NET Core, vedere l'introduzione all'[inserimento delle dipendenze in ASP.NET Core](xref:fundamentals/dependency-injection#replacing-the-default-services-container)</span><span class="sxs-lookup"><span data-stu-id="f8ba3-177">**Note:** For an in-depth reference to dependency injection in ASP.NET Core, see [Dependency Injection in ASP.NET Core](xref:fundamentals/dependency-injection#replacing-the-default-services-container)</span></span>

## <a name="serving-static-files"></a><span data-ttu-id="f8ba3-178">Gestione dei file statici</span><span class="sxs-lookup"><span data-stu-id="f8ba3-178">Serving Static Files</span></span>
<span data-ttu-id="f8ba3-179">Una parte importante dello sviluppo Web è la possibilità di distribuire asset statici sul lato client.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-179">An important part of web development is the ability to serve static, client-side assets.</span></span> <span data-ttu-id="f8ba3-180">Gli esempi più comuni di file statici sono HTML, CSS, Javascript e immagini.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-180">The most common examples of static files are HTML, CSS, Javascript, and images.</span></span> <span data-ttu-id="f8ba3-181">Questi file devono essere salvati nella posizione di pubblicazione dell'app (o della rete CDN) con riferimenti che ne consentano il caricamento da parte di una richiesta.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-181">These files need to be saved in the published location of the app (or CDN) and referenced so they can be loaded by a request.</span></span> <span data-ttu-id="f8ba3-182">Questo processo è stato modificato in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-182">This process has changed in ASP.NET Core.</span></span>

<span data-ttu-id="f8ba3-183">In ASP.NET i file statici vengono archiviati in directory diverse e viene fatto riferimento ai file nelle viste.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-183">In ASP.NET, static files are stored in various directories and referenced in the views.</span></span>

<span data-ttu-id="f8ba3-184">In ASP.NET Core i file statici vengono archiviati nella "radice Web" (*&lt;radice contenuto&gt;/wwwroot*), a meno che la configurazione non sia diversa.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-184">In ASP.NET Core, static files are stored in the "web root" (*&lt;content root&gt;/wwwroot*), unless configured otherwise.</span></span> <span data-ttu-id="f8ba3-185">I file vengono caricati nella pipeline delle richieste chiamando il metodo di estensione `UseStaticFiles` da `Startup.Configure`:</span><span class="sxs-lookup"><span data-stu-id="f8ba3-185">The files are loaded into the request pipeline by invoking the `UseStaticFiles` extension method from `Startup.Configure`:</span></span>

[!code-csharp[Main](../../fundamentals/static-files/sample/StartupStaticFiles.cs?highlight=3&name=snippet1)]

<span data-ttu-id="f8ba3-186">**Nota:** se la destinazione è .NET Framework, installare il pacchetto NuGet `Microsoft.AspNetCore.StaticFiles`.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-186">**Note:** If targeting .NET Framework, install the NuGet package `Microsoft.AspNetCore.StaticFiles`.</span></span>

<span data-ttu-id="f8ba3-187">Ad esempio, un asset immagine nella cartella *wwwroot/images* è accessibile al browser in corrispondenza di una posizione come `http://<app>/images/<imageFileName>`.</span><span class="sxs-lookup"><span data-stu-id="f8ba3-187">For example, an image asset in the *wwwroot/images* folder is accessible to the browser at a location such as `http://<app>/images/<imageFileName>`.</span></span>

<span data-ttu-id="f8ba3-188">**Nota:** per informazioni più dettagliate sulla gestione dei file statici in ASP.NET Core, vedere l'[introduzione all'uso dei file statici in ASP.NET Core](xref:fundamentals/static-files).</span><span class="sxs-lookup"><span data-stu-id="f8ba3-188">**Note:** For a more in-depth reference to serving static files in ASP.NET Core, see [Introduction to working with static files in ASP.NET Core](xref:fundamentals/static-files).</span></span>

## <a name="additional-resources"></a><span data-ttu-id="f8ba3-189">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="f8ba3-189">Additional Resources</span></span>
* [<span data-ttu-id="f8ba3-190">Portabilità in .NET Core - Librerie</span><span class="sxs-lookup"><span data-stu-id="f8ba3-190">Porting Libraries to .NET Core</span></span>](https://docs.microsoft.com/dotnet/core/porting/libraries)