---
title: La migrazione di gestori HTTP e moduli al middleware di ASP.NET Core.
author: rick-anderson
description: 
keywords: ASP.NET Core,
ms.author: tdykstra
manager: wpickett
ms.date: 12/07/2016
ms.topic: article
ms.assetid: 9c826a76-fbd2-46b5-978d-6ca6df53531a
ms.technology: aspnet
ms.prod: asp.net-core
uid: migration/http-modules
ms.openlocfilehash: e14664133abf010b80374036e4855fdff71d1d5f
ms.sourcegitcommit: 9cdbfd0d670d70b9c354216aabee260c52dad5ee
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/12/2017
---
# <a name="migrating-http-handlers-and-modules-to-aspnet-core-middleware"></a><span data-ttu-id="262c3-103">La migrazione di gestori HTTP e moduli al middleware di ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="262c3-103">Migrating HTTP handlers and modules to ASP.NET Core middleware</span></span> 

<span data-ttu-id="262c3-104">Da [Matt Perdeck](https://www.linkedin.com/in/mattperdeck)</span><span class="sxs-lookup"><span data-stu-id="262c3-104">By [Matt Perdeck](https://www.linkedin.com/in/mattperdeck)</span></span>

<span data-ttu-id="262c3-105">In questo articolo viene illustrato come eseguire la migrazione di ASP.NET esistente [moduli e i gestori da System. webServer](https://docs.microsoft.com/iis/configuration/system.webserver/) per ASP.NET Core [middleware](../fundamentals/middleware.md).</span><span class="sxs-lookup"><span data-stu-id="262c3-105">This article shows how to migrate existing ASP.NET [HTTP modules and handlers from system.webserver](https://docs.microsoft.com/iis/configuration/system.webserver/) to ASP.NET Core [middleware](../fundamentals/middleware.md).</span></span>

## <a name="modules-and-handlers-revisited"></a><span data-ttu-id="262c3-106">I moduli e i gestori aggiornamento</span><span class="sxs-lookup"><span data-stu-id="262c3-106">Modules and handlers revisited</span></span>

<span data-ttu-id="262c3-107">Prima di procedere al middleware di ASP.NET Core, verrà innanzitutto riepilogo funzionano dei gestori e moduli HTTP:</span><span class="sxs-lookup"><span data-stu-id="262c3-107">Before proceeding to ASP.NET Core middleware, let's first recap how HTTP modules and handlers work:</span></span>

![Gestore di moduli](http-modules/_static/moduleshandlers.png)

<span data-ttu-id="262c3-109">**I gestori sono:**</span><span class="sxs-lookup"><span data-stu-id="262c3-109">**Handlers are:**</span></span>

   * <span data-ttu-id="262c3-110">Le classi che implementano [IHttpHandler](https://docs.microsoft.com/dotnet/api/system.web.ihttphandler)</span><span class="sxs-lookup"><span data-stu-id="262c3-110">Classes that implement [IHttpHandler](https://docs.microsoft.com/dotnet/api/system.web.ihttphandler)</span></span>

   * <span data-ttu-id="262c3-111">Utilizzato per gestire le richieste con un nome file specificato o l'estensione, ad esempio *.report*</span><span class="sxs-lookup"><span data-stu-id="262c3-111">Used to handle requests with a given file name or extension, such as *.report*</span></span>

   * <span data-ttu-id="262c3-112">[Configurato](https://docs.microsoft.com//iis/configuration/system.webserver/handlers/) in *Web. config*</span><span class="sxs-lookup"><span data-stu-id="262c3-112">[Configured](https://docs.microsoft.com//iis/configuration/system.webserver/handlers/) in *Web.config*</span></span>

<span data-ttu-id="262c3-113">**I moduli sono:**</span><span class="sxs-lookup"><span data-stu-id="262c3-113">**Modules are:**</span></span>

   * <span data-ttu-id="262c3-114">Le classi che implementano [IHttpModule](https://docs.microsoft.com/dotnet/api/system.web.ihttpmodule)</span><span class="sxs-lookup"><span data-stu-id="262c3-114">Classes that implement [IHttpModule](https://docs.microsoft.com/dotnet/api/system.web.ihttpmodule)</span></span>

   * <span data-ttu-id="262c3-115">Viene richiamata per ogni richiesta</span><span class="sxs-lookup"><span data-stu-id="262c3-115">Invoked for every request</span></span>

   * <span data-ttu-id="262c3-116">In grado di corto circuito (interrompere ulteriore elaborazione di una richiesta)</span><span class="sxs-lookup"><span data-stu-id="262c3-116">Able to short-circuit (stop further processing of a request)</span></span>

   * <span data-ttu-id="262c3-117">In grado di aggiungere alla risposta HTTP o crearne di propri</span><span class="sxs-lookup"><span data-stu-id="262c3-117">Able to add to the HTTP response, or create their own</span></span>

   * <span data-ttu-id="262c3-118">[Configurato](https://docs.microsoft.com//iis/configuration/system.webserver/modules/) in *Web. config*</span><span class="sxs-lookup"><span data-stu-id="262c3-118">[Configured](https://docs.microsoft.com//iis/configuration/system.webserver/modules/) in *Web.config*</span></span>

<span data-ttu-id="262c3-119">**L'ordine in cui i moduli di elaborare le richieste in ingresso è determinato da:**</span><span class="sxs-lookup"><span data-stu-id="262c3-119">**The order in which modules process incoming requests is determined by:**</span></span>

   1. <span data-ttu-id="262c3-120">Il [ciclo di vita dell'applicazione](https://msdn.microsoft.com/library/ms227673.aspx), ovvero gli eventi una serie viene generato da ASP.NET: [BeginRequest](https://docs.microsoft.com/dotnet/api/system.web.httpapplication.beginrequest), [AuthenticateRequest](https://docs.microsoft.com/dotnet/api/system.web.httpapplication.authenticaterequest)e così via. Ogni modulo è possibile creare un gestore per uno o più eventi.</span><span class="sxs-lookup"><span data-stu-id="262c3-120">The [application life cycle](https://msdn.microsoft.com/library/ms227673.aspx), which is a series events fired by ASP.NET: [BeginRequest](https://docs.microsoft.com/dotnet/api/system.web.httpapplication.beginrequest), [AuthenticateRequest](https://docs.microsoft.com/dotnet/api/system.web.httpapplication.authenticaterequest), etc. Each module can create a handler for one or more events.</span></span>

   2. <span data-ttu-id="262c3-121">Per lo stesso evento, l'ordine in cui sono configurati in *Web. config*.</span><span class="sxs-lookup"><span data-stu-id="262c3-121">For the same event, the order in which they are configured in *Web.config*.</span></span>

<span data-ttu-id="262c3-122">Oltre a moduli, è possibile aggiungere i gestori per gli eventi del ciclo di vita per il *Global.asax.cs* file.</span><span class="sxs-lookup"><span data-stu-id="262c3-122">In addition to modules, you can add handlers for the life cycle events to your *Global.asax.cs* file.</span></span> <span data-ttu-id="262c3-123">Questi gestori vengono eseguiti dopo i gestori di moduli configurati.</span><span class="sxs-lookup"><span data-stu-id="262c3-123">These handlers run after the handlers in the configured modules.</span></span>

## <a name="from-handlers-and-modules-to-middleware"></a><span data-ttu-id="262c3-124">Da gestori e i moduli al middleware.</span><span class="sxs-lookup"><span data-stu-id="262c3-124">From handlers and modules to middleware</span></span>

<span data-ttu-id="262c3-125">**Middleware sono più semplici rispetto a gestori e i moduli HTTP:**</span><span class="sxs-lookup"><span data-stu-id="262c3-125">**Middleware are simpler than HTTP modules and handlers:**</span></span>

   * <span data-ttu-id="262c3-126">I moduli, i gestori, *Global.asax.cs*, *Web. config* (ad eccezione di configurazione di IIS) e il ciclo di vita dell'applicazione sono stati eliminati</span><span class="sxs-lookup"><span data-stu-id="262c3-126">Modules, handlers, *Global.asax.cs*, *Web.config* (except for IIS configuration) and the application life cycle are gone</span></span>

   * <span data-ttu-id="262c3-127">I ruoli di moduli e i gestori sono stati eseguiti dai middleware</span><span class="sxs-lookup"><span data-stu-id="262c3-127">The roles of both modules and handlers have been taken over by middleware</span></span>

   * <span data-ttu-id="262c3-128">Middleware vengono configurati utilizzando codice anziché in *Web. config*</span><span class="sxs-lookup"><span data-stu-id="262c3-128">Middleware are configured using code rather than in *Web.config*</span></span>

   * <span data-ttu-id="262c3-129">[Creazione di rami pipeline](../fundamentals/middleware.md#middleware-run-map-use) consente di inviare richieste al middleware specifico, dipende non solo l'URL ma anche delle intestazioni di richiesta, le stringhe di query, ecc.</span><span class="sxs-lookup"><span data-stu-id="262c3-129">[Pipeline branching](../fundamentals/middleware.md#middleware-run-map-use) lets you send requests to specific middleware, based on not only the URL but also on request headers, query strings, etc.</span></span>

<span data-ttu-id="262c3-130">**Middleware sono molto simili ai moduli:**</span><span class="sxs-lookup"><span data-stu-id="262c3-130">**Middleware are very similar to modules:**</span></span>

   * <span data-ttu-id="262c3-131">Richiamato in linea di massima per ogni richiesta</span><span class="sxs-lookup"><span data-stu-id="262c3-131">Invoked in principle for every request</span></span>

   * <span data-ttu-id="262c3-132">In grado di corto circuito, una richiesta da [non passare la richiesta al middleware successivo](#http-modules-shortcircuiting-middleware)</span><span class="sxs-lookup"><span data-stu-id="262c3-132">Able to short-circuit a request, by [not passing the request to the next middleware](#http-modules-shortcircuiting-middleware)</span></span>

   * <span data-ttu-id="262c3-133">In grado di creare i propri risposta HTTP</span><span class="sxs-lookup"><span data-stu-id="262c3-133">Able to create their own HTTP response</span></span>

<span data-ttu-id="262c3-134">**Middleware e i moduli vengono elaborati in un ordine diverso:**</span><span class="sxs-lookup"><span data-stu-id="262c3-134">**Middleware and modules are processed in a different order:**</span></span>

   * <span data-ttu-id="262c3-135">Ordine di middleware è in base all'ordine in cui sono vengono inserite nella pipeline delle richieste, mentre l'ordine dei moduli si basa principalmente sulla [ciclo di vita dell'applicazione](https://msdn.microsoft.com/library/ms227673.aspx) eventi</span><span class="sxs-lookup"><span data-stu-id="262c3-135">Order of middleware is based on the order in which they are inserted into the request pipeline, while order of modules is mainly based on [application life cycle](https://msdn.microsoft.com/library/ms227673.aspx) events</span></span>

   * <span data-ttu-id="262c3-136">Ordine del middleware per le risposte è l'opposto rispetto a quello per le richieste, mentre l'ordine dei moduli è lo stesso per le richieste e risposte</span><span class="sxs-lookup"><span data-stu-id="262c3-136">Order of middleware for responses is the reverse from that for requests, while order of modules is the same for requests and responses</span></span>

   * <span data-ttu-id="262c3-137">Vedere [creazione di una pipeline middleware con IApplicationBuilder](../fundamentals/middleware.md#creating-a-middleware-pipeline-with-iapplicationbuilder)</span><span class="sxs-lookup"><span data-stu-id="262c3-137">See [Creating a middleware pipeline with IApplicationBuilder](../fundamentals/middleware.md#creating-a-middleware-pipeline-with-iapplicationbuilder)</span></span>

![Middleware](http-modules/_static/middleware.png)

<span data-ttu-id="262c3-139">Si noti come nell'immagine precedente, il middleware di autenticazione short-circuited la richiesta.</span><span class="sxs-lookup"><span data-stu-id="262c3-139">Note how in the image above, the authentication middleware short-circuited the request.</span></span>

## <a name="migrating-module-code-to-middleware"></a><span data-ttu-id="262c3-140">Migrazione del codice modulo al middleware.</span><span class="sxs-lookup"><span data-stu-id="262c3-140">Migrating module code to middleware</span></span>

<span data-ttu-id="262c3-141">Un modulo HTTP esistente avrà un aspetto simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="262c3-141">An existing HTTP module will look similar to this:</span></span>

<span data-ttu-id="262c3-142">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Modules/MyModule.cs?highlight=6,8,24,31)]</span><span class="sxs-lookup"><span data-stu-id="262c3-142">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Modules/MyModule.cs?highlight=6,8,24,31)]</span></span>

<span data-ttu-id="262c3-143">Come illustrato nel [Middleware](../fundamentals/middleware.md) pagina, un middleware di ASP.NET Core è una classe che espone un `Invoke` acquisire metodo un `HttpContext` e restituendo un `Task`.</span><span class="sxs-lookup"><span data-stu-id="262c3-143">As shown in the [Middleware](../fundamentals/middleware.md) page, an ASP.NET Core middleware is a class that exposes an `Invoke` method taking an `HttpContext` and returning a `Task`.</span></span> <span data-ttu-id="262c3-144">Il middleware nuovo sarà simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="262c3-144">Your new middleware will look like this:</span></span>

<a name=http-modules-usemiddleware></a>

<span data-ttu-id="262c3-145">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddleware.cs?highlight=9,13,20,24,28,30,32)]</span><span class="sxs-lookup"><span data-stu-id="262c3-145">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddleware.cs?highlight=9,13,20,24,28,30,32)]</span></span>

<span data-ttu-id="262c3-146">Il modello di middleware precedente è stato effettuato dalla sezione su [scrittura middleware](../fundamentals/middleware.md#middleware-writing-middleware).</span><span class="sxs-lookup"><span data-stu-id="262c3-146">The above middleware template was taken from the section on [writing middleware](../fundamentals/middleware.md#middleware-writing-middleware).</span></span>

<span data-ttu-id="262c3-147">Il *MyMiddlewareExtensions* classe helper rende più semplice configurare il middleware del `Startup` classe.</span><span class="sxs-lookup"><span data-stu-id="262c3-147">The *MyMiddlewareExtensions* helper class makes it easier to configure your middleware in your `Startup` class.</span></span> <span data-ttu-id="262c3-148">Il `UseMyMiddleware` metodo aggiunge la classe middleware alla pipeline delle richieste.</span><span class="sxs-lookup"><span data-stu-id="262c3-148">The `UseMyMiddleware` method adds your middleware class to the request pipeline.</span></span> <span data-ttu-id="262c3-149">Servizi richiesti dal middleware ottengano inseriti nel costruttore del middleware.</span><span class="sxs-lookup"><span data-stu-id="262c3-149">Services required by the middleware get injected in the middleware's constructor.</span></span>

<a name=http-modules-shortcircuiting-middleware></a>

<span data-ttu-id="262c3-150">Il modulo potrebbe terminare una richiesta, ad esempio se l'utente non è autorizzato:</span><span class="sxs-lookup"><span data-stu-id="262c3-150">Your module might terminate a request, for example if the user is not authorized:</span></span>

<span data-ttu-id="262c3-151">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Modules/MyTerminatingModule.cs?highlight=9,10,11,12,13&name=snippet_Terminate)]</span><span class="sxs-lookup"><span data-stu-id="262c3-151">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Modules/MyTerminatingModule.cs?highlight=9,10,11,12,13&name=snippet_Terminate)]</span></span>

<span data-ttu-id="262c3-152">Un tipo di middleware viene gestita tramite la chiamata non `Invoke` nel middleware successivo nella pipeline.</span><span class="sxs-lookup"><span data-stu-id="262c3-152">A middleware handles this by not calling `Invoke` on the next middleware in the pipeline.</span></span> <span data-ttu-id="262c3-153">Tenere presente che questo non viene completamente terminato la richiesta, in quanto middlewares precedente ancora da richiamare quando la risposta procede attraverso la pipeline.</span><span class="sxs-lookup"><span data-stu-id="262c3-153">Keep in mind that this does not fully terminate the request, because previous middlewares will still be invoked when the response makes its way back through the pipeline.</span></span>

<span data-ttu-id="262c3-154">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyTerminatingMiddleware.cs?highlight=7,8&name=snippet_Terminate)]</span><span class="sxs-lookup"><span data-stu-id="262c3-154">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyTerminatingMiddleware.cs?highlight=7,8&name=snippet_Terminate)]</span></span>

<span data-ttu-id="262c3-155">Quando si esegue la migrazione di funzionalità del modulo per il middleware di nuovo, è possibile che il codice non viene compilato perché la `HttpContext` classe è stata modificata in modo significativo in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="262c3-155">When you migrate your module's functionality to your new middleware, you may find that your code doesn't compile because the `HttpContext` class has significantly changed in ASP.NET Core.</span></span> <span data-ttu-id="262c3-156">[In un secondo momento](#migrating-to-the-new-httpcontext), si vedrà come eseguire la migrazione alla nuova ASP.NET Core HttpContext.</span><span class="sxs-lookup"><span data-stu-id="262c3-156">[Later on](#migrating-to-the-new-httpcontext), you'll see how to migrate to the new ASP.NET Core HttpContext.</span></span>

## <a name="migrating-module-insertion-into-the-request-pipeline"></a><span data-ttu-id="262c3-157">Inserimento di modulo migrazione nella pipeline delle richieste</span><span class="sxs-lookup"><span data-stu-id="262c3-157">Migrating module insertion into the request pipeline</span></span>

<span data-ttu-id="262c3-158">I moduli HTTP vengono in genere aggiunti per la richiesta di pipeline utilizzando *Web. config*:</span><span class="sxs-lookup"><span data-stu-id="262c3-158">HTTP modules are typically added to the request pipeline using *Web.config*:</span></span>

<span data-ttu-id="262c3-159">[!code-xml[Principale](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Web.config?highlight=6&range=1-3,32-33,36,43,50,101)]</span><span class="sxs-lookup"><span data-stu-id="262c3-159">[!code-xml[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Web.config?highlight=6&range=1-3,32-33,36,43,50,101)]</span></span>

<span data-ttu-id="262c3-160">Il problema, convertire [aggiunta del nuovo middleware](../fundamentals/middleware.md#creating-a-middleware-pipeline-with-iapplicationbuilder) alla pipeline delle richieste nel `Startup` classe:</span><span class="sxs-lookup"><span data-stu-id="262c3-160">Convert this by [adding your new middleware](../fundamentals/middleware.md#creating-a-middleware-pipeline-with-iapplicationbuilder) to the request pipeline in your `Startup` class:</span></span>

<span data-ttu-id="262c3-161">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=16)]</span><span class="sxs-lookup"><span data-stu-id="262c3-161">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=16)]</span></span>

<span data-ttu-id="262c3-162">La posizione esatta nella pipeline di punto di inserimento del nuovo middleware dipende dall'evento che gestito come modulo (`BeginRequest`, `EndRequest`e così via) e il relativo ordine nell'elenco dei moduli in *Web. config*.</span><span class="sxs-lookup"><span data-stu-id="262c3-162">The exact spot in the pipeline where you insert your new middleware depends on the event that it handled as a module (`BeginRequest`, `EndRequest`, etc.) and its order in your list of modules in *Web.config*.</span></span>

<span data-ttu-id="262c3-163">Come già non indicato, vi è alcun ciclo di vita delle applicazioni in ASP.NET Core e l'ordine in cui vengono elaborate le risposte dal middleware è diverso da quello usato dai moduli.</span><span class="sxs-lookup"><span data-stu-id="262c3-163">As previously stated, there is no application life cycle in ASP.NET Core and the order in which responses are processed by middleware differs from the order used by modules.</span></span> <span data-ttu-id="262c3-164">Questo potrebbe rendere la decisione di ordinamento più complessa.</span><span class="sxs-lookup"><span data-stu-id="262c3-164">This could make your ordering decision more challenging.</span></span>

<span data-ttu-id="262c3-165">Se l'ordinamento diventa un problema, è possibile suddividere un modulo in più componenti middleware che possono essere ordinati in modo indipendente.</span><span class="sxs-lookup"><span data-stu-id="262c3-165">If ordering becomes a problem, you could split your module into multiple middleware components that can be ordered independently.</span></span>

## <a name="migrating-handler-code-to-middleware"></a><span data-ttu-id="262c3-166">Migrazione del codice del gestore al middleware.</span><span class="sxs-lookup"><span data-stu-id="262c3-166">Migrating handler code to middleware</span></span>

<span data-ttu-id="262c3-167">Un gestore HTTP è simile alla seguente:</span><span class="sxs-lookup"><span data-stu-id="262c3-167">An HTTP handler looks something like this:</span></span>

<span data-ttu-id="262c3-168">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net4/Asp.Net4/HttpHandlers/ReportHandler.cs?highlight=5,7,13,14,15,16)]</span><span class="sxs-lookup"><span data-stu-id="262c3-168">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/HttpHandlers/ReportHandler.cs?highlight=5,7,13,14,15,16)]</span></span>

<span data-ttu-id="262c3-169">Nel progetto ASP.NET Core, è necessario convertire in un tipo di middleware simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="262c3-169">In your ASP.NET Core project, you would translate this to a middleware similar to this:</span></span>

<span data-ttu-id="262c3-170">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net.Core/Middleware/ReportHandlerMiddleware.cs?highlight=7,9,13,20,21,22,23,40,42,44)]</span><span class="sxs-lookup"><span data-stu-id="262c3-170">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/ReportHandlerMiddleware.cs?highlight=7,9,13,20,21,22,23,40,42,44)]</span></span>

<span data-ttu-id="262c3-171">Questo middleware è molto simile per il middleware corrispondente ai moduli.</span><span class="sxs-lookup"><span data-stu-id="262c3-171">This middleware is very similar to the middleware corresponding to modules.</span></span> <span data-ttu-id="262c3-172">L'unica differenza è che qui non è presente alcuna chiamata a `_next.Invoke(context)`.</span><span class="sxs-lookup"><span data-stu-id="262c3-172">The only real difference is that here there is no call to `_next.Invoke(context)`.</span></span> <span data-ttu-id="262c3-173">Che è utile, perché il gestore non è alla fine della pipeline di richieste, in modo che vi sia alcun middleware successivo da richiamare.</span><span class="sxs-lookup"><span data-stu-id="262c3-173">That makes sense, because the handler is at the end of the request pipeline, so there will be no next middleware to invoke.</span></span>

## <a name="migrating-handler-insertion-into-the-request-pipeline"></a><span data-ttu-id="262c3-174">La migrazione di inserimento gestore nella pipeline delle richieste</span><span class="sxs-lookup"><span data-stu-id="262c3-174">Migrating handler insertion into the request pipeline</span></span>

<span data-ttu-id="262c3-175">Configurazione di un gestore HTTP viene eseguita in *Web. config* e simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="262c3-175">Configuring an HTTP handler is done in *Web.config* and looks something like this:</span></span>

<span data-ttu-id="262c3-176">[!code-xml[Principale](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Web.config?highlight=6&range=1-3,32,46-48,50,101)]</span><span class="sxs-lookup"><span data-stu-id="262c3-176">[!code-xml[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Web.config?highlight=6&range=1-3,32,46-48,50,101)]</span></span>

<span data-ttu-id="262c3-177">È possibile convertire questo valore tramite l'aggiunta del nuovo middleware gestore alla pipeline delle richieste nel `Startup` (classe), simile al middleware convertito da moduli.</span><span class="sxs-lookup"><span data-stu-id="262c3-177">You could convert this by adding your new handler middleware to the request pipeline in your `Startup` class, similar to middleware converted from modules.</span></span> <span data-ttu-id="262c3-178">Il problema con questo approccio è che invierebbe tutte le richieste per il nuovo middleware gestore.</span><span class="sxs-lookup"><span data-stu-id="262c3-178">The problem with that approach is that it would send all requests to your new handler middleware.</span></span> <span data-ttu-id="262c3-179">Tuttavia, si desidera solo le richieste con una determinata estensione per raggiungere il middleware.</span><span class="sxs-lookup"><span data-stu-id="262c3-179">However, you only want requests with a given extension to reach your middleware.</span></span> <span data-ttu-id="262c3-180">Che fornisce la stessa funzionalità che si aveva con il gestore HTTP.</span><span class="sxs-lookup"><span data-stu-id="262c3-180">That would give you the same functionality you had with your HTTP handler.</span></span>

<span data-ttu-id="262c3-181">Una soluzione consiste nel creare un ramo della pipeline per le richieste con una determinata estensione utilizzando il `MapWhen` metodo di estensione.</span><span class="sxs-lookup"><span data-stu-id="262c3-181">One solution is to branch the pipeline for requests with a given extension, using the `MapWhen` extension method.</span></span> <span data-ttu-id="262c3-182">A tale scopo, lo stesso `Configure` metodo in cui si aggiunge il middleware di altri:</span><span class="sxs-lookup"><span data-stu-id="262c3-182">You do this in the same `Configure` method where you add the other middleware:</span></span>

<span data-ttu-id="262c3-183">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=27-34)]</span><span class="sxs-lookup"><span data-stu-id="262c3-183">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=27-34)]</span></span>

<span data-ttu-id="262c3-184">`MapWhen`utilizza questi parametri:</span><span class="sxs-lookup"><span data-stu-id="262c3-184">`MapWhen` takes these parameters:</span></span>

1. <span data-ttu-id="262c3-185">Un'espressione lambda che accetta il `HttpContext` e restituisce `true` se la richiesta deve essere inoltrato al ramo.</span><span class="sxs-lookup"><span data-stu-id="262c3-185">A lambda that takes the `HttpContext` and returns `true` if the request should go down the branch.</span></span> <span data-ttu-id="262c3-186">Ciò significa che è possibile creare rami di richieste di base non solo l'estensione, ma anche le intestazioni di richiesta, i parametri di stringa di query e così via.</span><span class="sxs-lookup"><span data-stu-id="262c3-186">This means you can branch requests not just based on their extension, but also on request headers, query string parameters, etc.</span></span>

2. <span data-ttu-id="262c3-187">Un'espressione lambda che accetta un `IApplicationBuilder` e aggiunge il middleware di tutti per il ramo.</span><span class="sxs-lookup"><span data-stu-id="262c3-187">A lambda that takes an `IApplicationBuilder` and adds all the middleware for the branch.</span></span> <span data-ttu-id="262c3-188">Ciò significa che è possibile aggiungere altro middleware per il ramo davanti del middleware di gestore.</span><span class="sxs-lookup"><span data-stu-id="262c3-188">This means you can add additional middleware to the branch in front of your handler middleware.</span></span>

<span data-ttu-id="262c3-189">Middleware aggiunto alla pipeline prima che il ramo verrà richiamato in tutte le richieste; il ramo avrà alcun effetto su di essi.</span><span class="sxs-lookup"><span data-stu-id="262c3-189">Middleware added to the pipeline before the branch will be invoked on all requests; the branch will have no impact on them.</span></span>

## <a name="loading-middleware-options-using-the-options-pattern"></a><span data-ttu-id="262c3-190">Il caricamento di opzioni del middleware usando il modello di opzioni</span><span class="sxs-lookup"><span data-stu-id="262c3-190">Loading middleware options using the options pattern</span></span>

<span data-ttu-id="262c3-191">Alcuni gestori e i moduli sono disponibili opzioni di configurazione che vengono archiviate in *Web. config*. Tuttavia, in ASP.NET Core viene utilizzato un nuovo modello di configurazione al posto di *Web. config*.</span><span class="sxs-lookup"><span data-stu-id="262c3-191">Some modules and handlers have configuration options that are stored in *Web.config*. However, in ASP.NET Core a new configuration model is used in place of *Web.config*.</span></span>

<span data-ttu-id="262c3-192">Il nuovo [sistema di configurazione](../fundamentals/configuration.md) fornisce le seguenti opzioni per risolvere il problema:</span><span class="sxs-lookup"><span data-stu-id="262c3-192">The new [configuration system](../fundamentals/configuration.md) gives you these options to solve this:</span></span>

* <span data-ttu-id="262c3-193">Inserire direttamente le opzioni in middleware, come illustrato nel [nella sezione successiva](#loading-middleware-options-through-direct-injection).</span><span class="sxs-lookup"><span data-stu-id="262c3-193">Directly inject the options into the middleware, as shown in the [next section](#loading-middleware-options-through-direct-injection).</span></span>

* <span data-ttu-id="262c3-194">Utilizzare il [modello opzioni](../fundamentals/configuration.md#options-config-objects):</span><span class="sxs-lookup"><span data-stu-id="262c3-194">Use the [options pattern](../fundamentals/configuration.md#options-config-objects):</span></span>

1.  <span data-ttu-id="262c3-195">Creare una classe che contenga le opzioni del middleware, ad esempio:</span><span class="sxs-lookup"><span data-stu-id="262c3-195">Create a class to hold your middleware options, for example:</span></span>

    <span data-ttu-id="262c3-196">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_Options)]</span><span class="sxs-lookup"><span data-stu-id="262c3-196">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_Options)]</span></span>

2.  <span data-ttu-id="262c3-197">Archiviare i valori di opzione</span><span class="sxs-lookup"><span data-stu-id="262c3-197">Store the option values</span></span>

    <span data-ttu-id="262c3-198">Il sistema di configurazione consente di archiviare i valori di opzione in qualsiasi punto desiderato.</span><span class="sxs-lookup"><span data-stu-id="262c3-198">The configuration system allows you to store option values anywhere you want.</span></span> <span data-ttu-id="262c3-199">Tuttavia, più siti utilizzare *appSettings. JSON*, pertanto l'utente verrà reindirizzato a tale approccio:</span><span class="sxs-lookup"><span data-stu-id="262c3-199">However, most sites use *appsettings.json*, so we'll take that approach:</span></span>

    <span data-ttu-id="262c3-200">[!code-json[Principale](http-modules/sample/Asp.Net.Core/appsettings.json?range=1,14-18)]</span><span class="sxs-lookup"><span data-stu-id="262c3-200">[!code-json[Main](http-modules/sample/Asp.Net.Core/appsettings.json?range=1,14-18)]</span></span>

    <span data-ttu-id="262c3-201">*MyMiddlewareOptionsSection* qui è un nome di sezione.</span><span class="sxs-lookup"><span data-stu-id="262c3-201">*MyMiddlewareOptionsSection* here is a section name.</span></span> <span data-ttu-id="262c3-202">Non deve essere identico al nome della classe di opzioni.</span><span class="sxs-lookup"><span data-stu-id="262c3-202">It doesn't have to be the same as the name of your options class.</span></span>

3. <span data-ttu-id="262c3-203">Associare i valori dell'opzione con la classe di opzioni</span><span class="sxs-lookup"><span data-stu-id="262c3-203">Associate the option values with the options class</span></span>

    <span data-ttu-id="262c3-204">Il modello di opzioni Usa framework della ASP.NET Core dipendenza attacco intrusivo nel codice per associare il tipo di opzioni (ad esempio `MyMiddlewareOptions`) con un `MyMiddlewareOptions` oggetto con le opzioni.</span><span class="sxs-lookup"><span data-stu-id="262c3-204">The options pattern uses ASP.NET Core's dependency injection framework to associate the options type (such as `MyMiddlewareOptions`) with a `MyMiddlewareOptions` object that has the actual options.</span></span>

    <span data-ttu-id="262c3-205">Aggiornamento del `Startup` classe:</span><span class="sxs-lookup"><span data-stu-id="262c3-205">Update your `Startup` class:</span></span>

    1.  <span data-ttu-id="262c3-206">Se si utilizza *appSettings. JSON*, aggiungerlo al generatore di configurazione nel `Startup` costruttore:</span><span class="sxs-lookup"><span data-stu-id="262c3-206">If you're using *appsettings.json*, add it to the configuration builder in the `Startup` constructor:</span></span>

      <span data-ttu-id="262c3-207">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Ctor&highlight=5-6)]</span><span class="sxs-lookup"><span data-stu-id="262c3-207">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Ctor&highlight=5-6)]</span></span>

    2.  <span data-ttu-id="262c3-208">Configurare il servizio di opzioni:</span><span class="sxs-lookup"><span data-stu-id="262c3-208">Configure the options service:</span></span>

      <span data-ttu-id="262c3-209">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_ConfigureServices&highlight=4)]</span><span class="sxs-lookup"><span data-stu-id="262c3-209">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_ConfigureServices&highlight=4)]</span></span>

    3.  <span data-ttu-id="262c3-210">Associare le opzioni disponibili con la classe di opzioni:</span><span class="sxs-lookup"><span data-stu-id="262c3-210">Associate your options with your options class:</span></span>

      <span data-ttu-id="262c3-211">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_ConfigureServices&highlight=6-8)]</span><span class="sxs-lookup"><span data-stu-id="262c3-211">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_ConfigureServices&highlight=6-8)]</span></span>

4.  <span data-ttu-id="262c3-212">Inserire le opzioni nel costruttore del middleware.</span><span class="sxs-lookup"><span data-stu-id="262c3-212">Inject the options into your middleware constructor.</span></span> <span data-ttu-id="262c3-213">È simile per l'inserimento di opzioni in un controller.</span><span class="sxs-lookup"><span data-stu-id="262c3-213">This is similar to injecting options into a controller.</span></span>

  <span data-ttu-id="262c3-214">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_MiddlewareWithParams&highlight=4,7,10,15-16)]</span><span class="sxs-lookup"><span data-stu-id="262c3-214">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_MiddlewareWithParams&highlight=4,7,10,15-16)]</span></span>

  <span data-ttu-id="262c3-215">Il [UseMiddleware](#http-modules-usemiddleware) metodo di estensione che aggiunge il middleware per la `IApplicationBuilder` si occupa di inserimento di dipendenze.</span><span class="sxs-lookup"><span data-stu-id="262c3-215">The [UseMiddleware](#http-modules-usemiddleware) extension method that adds your middleware to the `IApplicationBuilder` takes care of dependency injection.</span></span>

  <span data-ttu-id="262c3-216">Questo non è limitato a `IOptions` oggetti.</span><span class="sxs-lookup"><span data-stu-id="262c3-216">This is not limited to `IOptions` objects.</span></span> <span data-ttu-id="262c3-217">In questo modo può essere inserito da qualsiasi altro oggetto che richiede il middleware.</span><span class="sxs-lookup"><span data-stu-id="262c3-217">Any other object that your middleware requires can be injected this way.</span></span>

## <a name="loading-middleware-options-through-direct-injection"></a><span data-ttu-id="262c3-218">Il caricamento di opzioni del middleware tramite inserimento diretto</span><span class="sxs-lookup"><span data-stu-id="262c3-218">Loading middleware options through direct injection</span></span>

<span data-ttu-id="262c3-219">Il modello di opzioni presenta il vantaggio che crea separato accoppiamento tra i valori delle opzioni e utenti.</span><span class="sxs-lookup"><span data-stu-id="262c3-219">The options pattern has the advantage that it creates loose coupling between options values and their consumers.</span></span> <span data-ttu-id="262c3-220">Una volta associato a una classe di opzioni i valori delle opzioni effettivo, qualsiasi altra classe può accedere alle opzioni tramite il framework di inserimento di dipendenza.</span><span class="sxs-lookup"><span data-stu-id="262c3-220">Once you've associated an options class with the actual options values, any other class can get access to the options through the dependency injection framework.</span></span> <span data-ttu-id="262c3-221">Non è necessario passare intorno ai valori di opzioni.</span><span class="sxs-lookup"><span data-stu-id="262c3-221">There is no need to pass around options values.</span></span>

<span data-ttu-id="262c3-222">Questo modo si ottengono tuttavia se si desidera utilizzare il middleware stesso due volte, con opzioni diverse.</span><span class="sxs-lookup"><span data-stu-id="262c3-222">This breaks down though if you want to use the same middleware twice, with different options.</span></span> <span data-ttu-id="262c3-223">Ad esempio un'autorizzazione middleware utilizzato in diversi rami che consente di ruoli diversi.</span><span class="sxs-lookup"><span data-stu-id="262c3-223">For example an authorization middleware used in different branches allowing different roles.</span></span> <span data-ttu-id="262c3-224">È possibile associare i due oggetti diverse opzioni con la classe di opzioni di uno.</span><span class="sxs-lookup"><span data-stu-id="262c3-224">You can't associate two different options objects with the one options class.</span></span>

<span data-ttu-id="262c3-225">La soluzione consiste nell'ottenere gli oggetti di opzioni con i valori effettivi opzioni la `Startup` classe e passare direttamente in ogni istanza del middleware.</span><span class="sxs-lookup"><span data-stu-id="262c3-225">The solution is to get the options objects with the actual options values in your `Startup` class and pass those directly to each instance of your middleware.</span></span>

1.  <span data-ttu-id="262c3-226">Aggiungere una seconda chiave da *appSettings. JSON*</span><span class="sxs-lookup"><span data-stu-id="262c3-226">Add a second key to *appsettings.json*</span></span>

    <span data-ttu-id="262c3-227">Per aggiungere un secondo set di opzioni per il *appSettings. JSON* file, utilizzare una nuova chiave per identificarla in modo univoco:</span><span class="sxs-lookup"><span data-stu-id="262c3-227">To add a second set of options to the *appsettings.json* file, use a new key to uniquely identify it:</span></span>

    <span data-ttu-id="262c3-228">[!code-json[Principale](http-modules/sample/Asp.Net.Core/appsettings.json?range=1,10-18&highlight=2-5)]</span><span class="sxs-lookup"><span data-stu-id="262c3-228">[!code-json[Main](http-modules/sample/Asp.Net.Core/appsettings.json?range=1,10-18&highlight=2-5)]</span></span>

2.  <span data-ttu-id="262c3-229">Recuperare i valori delle opzioni e passarle al middleware.</span><span class="sxs-lookup"><span data-stu-id="262c3-229">Retrieve options values and pass them to middleware.</span></span> <span data-ttu-id="262c3-230">Il `Use...` metodo di estensione (che aggiunge il middleware alla pipeline) è un punto logico per passare i valori di opzione:</span><span class="sxs-lookup"><span data-stu-id="262c3-230">The `Use...` extension method (which adds your middleware to the pipeline) is a logical place to pass in the option values:</span></span> 

    <span data-ttu-id="262c3-231">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=20-23)]</span><span class="sxs-lookup"><span data-stu-id="262c3-231">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=20-23)]</span></span>

4.  <span data-ttu-id="262c3-232">Abilitare il middleware accettare un parametro di opzioni.</span><span class="sxs-lookup"><span data-stu-id="262c3-232">Enable middleware to take an options parameter.</span></span> <span data-ttu-id="262c3-233">Fornire un overload di `Use...` metodo di estensione (che accetta il parametro di opzioni e lo passa al `UseMiddleware`).</span><span class="sxs-lookup"><span data-stu-id="262c3-233">Provide an overload of the `Use...` extension method (that takes the options parameter and passes it to `UseMiddleware`).</span></span> <span data-ttu-id="262c3-234">Quando `UseMiddleware` viene chiamato con parametri, passa i parametri del costruttore del middleware quando viene creata un'istanza dell'oggetto middleware.</span><span class="sxs-lookup"><span data-stu-id="262c3-234">When `UseMiddleware` is called with parameters, it passes the parameters to your middleware constructor when it instantiates the middleware object.</span></span>

    <span data-ttu-id="262c3-235">[!code-csharp[Principale](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_Extensions&highlight=9-14)]</span><span class="sxs-lookup"><span data-stu-id="262c3-235">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_Extensions&highlight=9-14)]</span></span>

    <span data-ttu-id="262c3-236">Si noti come questo include l'oggetto di opzioni in un `OptionsWrapper` oggetto.</span><span class="sxs-lookup"><span data-stu-id="262c3-236">Note how this wraps the options object in an `OptionsWrapper` object.</span></span> <span data-ttu-id="262c3-237">Implementa `IOptions`, come previsto dal costruttore del middleware.</span><span class="sxs-lookup"><span data-stu-id="262c3-237">This implements `IOptions`, as expected by the middleware constructor.</span></span>

## <a name="migrating-to-the-new-httpcontext"></a><span data-ttu-id="262c3-238">La migrazione a nuovi HttpContext</span><span class="sxs-lookup"><span data-stu-id="262c3-238">Migrating to the new HttpContext</span></span>

<span data-ttu-id="262c3-239">Illustrato in precedenza che il `Invoke` del middleware metodo accetta un parametro di tipo `HttpContext`:</span><span class="sxs-lookup"><span data-stu-id="262c3-239">You saw earlier that the `Invoke` method in your middleware takes a parameter of type `HttpContext`:</span></span>

```csharp
public async Task Invoke(HttpContext context)
```

<span data-ttu-id="262c3-240">`HttpContext`è stato modificato in modo significativo in ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="262c3-240">`HttpContext` has significantly changed in ASP.NET Core.</span></span> <span data-ttu-id="262c3-241">In questa sezione viene illustrato come convertire le proprietà utilizzate più di frequente di [System.Web.HttpContext](https://docs.microsoft.com/dotnet/api/system.web.httpcontext) al nuovo `Microsoft.AspNetCore.Http.HttpContext`.</span><span class="sxs-lookup"><span data-stu-id="262c3-241">This section shows how to translate the most commonly used properties of [System.Web.HttpContext](https://docs.microsoft.com/dotnet/api/system.web.httpcontext) to the new `Microsoft.AspNetCore.Http.HttpContext`.</span></span>

### <a name="httpcontext"></a><span data-ttu-id="262c3-242">HttpContext</span><span class="sxs-lookup"><span data-stu-id="262c3-242">HttpContext</span></span>

<span data-ttu-id="262c3-243">**HttpContext. Items** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-243">**HttpContext.Items** translates to:</span></span>

<span data-ttu-id="262c3-244">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Items)]</span><span class="sxs-lookup"><span data-stu-id="262c3-244">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Items)]</span></span>

<span data-ttu-id="262c3-245">**ID richiesta univoco (non controparte System.Web.HttpContext)**</span><span class="sxs-lookup"><span data-stu-id="262c3-245">**Unique request ID (no System.Web.HttpContext counterpart)**</span></span>

<span data-ttu-id="262c3-246">Fornisce un id univoco per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="262c3-246">Gives you a unique id for each request.</span></span> <span data-ttu-id="262c3-247">Molto utile da includere nei file registro.</span><span class="sxs-lookup"><span data-stu-id="262c3-247">Very useful to include in your logs.</span></span>

<span data-ttu-id="262c3-248">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Trace)]</span><span class="sxs-lookup"><span data-stu-id="262c3-248">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Trace)]</span></span>

### <a name="httpcontextrequest"></a><span data-ttu-id="262c3-249">HttpContext. Request</span><span class="sxs-lookup"><span data-stu-id="262c3-249">HttpContext.Request</span></span>

<span data-ttu-id="262c3-250">**HttpContext.Request.HttpMethod** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-250">**HttpContext.Request.HttpMethod** translates to:</span></span>

<span data-ttu-id="262c3-251">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Method)]</span><span class="sxs-lookup"><span data-stu-id="262c3-251">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Method)]</span></span>

<span data-ttu-id="262c3-252">**HttpContext.Request.QueryString** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-252">**HttpContext.Request.QueryString** translates to:</span></span>

<span data-ttu-id="262c3-253">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Query)]</span><span class="sxs-lookup"><span data-stu-id="262c3-253">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Query)]</span></span>

<span data-ttu-id="262c3-254">**HttpContext.Request.Url** e **HttpContext.Request.RawUrl** Traduci in:</span><span class="sxs-lookup"><span data-stu-id="262c3-254">**HttpContext.Request.Url** and **HttpContext.Request.RawUrl** translate to:</span></span>

<span data-ttu-id="262c3-255">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Url)]</span><span class="sxs-lookup"><span data-stu-id="262c3-255">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Url)]</span></span>

<span data-ttu-id="262c3-256">**HttpContext.Request.IsSecureConnection** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-256">**HttpContext.Request.IsSecureConnection** translates to:</span></span>

<span data-ttu-id="262c3-257">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Secure)]</span><span class="sxs-lookup"><span data-stu-id="262c3-257">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Secure)]</span></span>

<span data-ttu-id="262c3-258">**HttpContext.Request.UserHostAddress** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-258">**HttpContext.Request.UserHostAddress** translates to:</span></span>

<span data-ttu-id="262c3-259">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Host)]</span><span class="sxs-lookup"><span data-stu-id="262c3-259">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Host)]</span></span>

<span data-ttu-id="262c3-260">**HttpContext.Request.Cookies** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-260">**HttpContext.Request.Cookies** translates to:</span></span>

<span data-ttu-id="262c3-261">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Cookies)]</span><span class="sxs-lookup"><span data-stu-id="262c3-261">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Cookies)]</span></span>

<span data-ttu-id="262c3-262">**HttpContext.Request.RequestContext.RouteData** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-262">**HttpContext.Request.RequestContext.RouteData** translates to:</span></span>

<span data-ttu-id="262c3-263">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Route)]</span><span class="sxs-lookup"><span data-stu-id="262c3-263">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Route)]</span></span>

<span data-ttu-id="262c3-264">**HttpContext.Request.Headers** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-264">**HttpContext.Request.Headers** translates to:</span></span>

<span data-ttu-id="262c3-265">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Headers)]</span><span class="sxs-lookup"><span data-stu-id="262c3-265">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Headers)]</span></span>

<span data-ttu-id="262c3-266">**HttpContext.Request.UserAgent** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-266">**HttpContext.Request.UserAgent** translates to:</span></span>

<span data-ttu-id="262c3-267">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Agent)]</span><span class="sxs-lookup"><span data-stu-id="262c3-267">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Agent)]</span></span>

<span data-ttu-id="262c3-268">**HttpContext.Request.UrlReferrer** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-268">**HttpContext.Request.UrlReferrer** translates to:</span></span>

<span data-ttu-id="262c3-269">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Referrer)]</span><span class="sxs-lookup"><span data-stu-id="262c3-269">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Referrer)]</span></span>

<span data-ttu-id="262c3-270">**HttpContext.Request.ContentType** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-270">**HttpContext.Request.ContentType** translates to:</span></span>

<span data-ttu-id="262c3-271">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Type)]</span><span class="sxs-lookup"><span data-stu-id="262c3-271">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Type)]</span></span>

<span data-ttu-id="262c3-272">**HttpContext.Request.Form** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-272">**HttpContext.Request.Form** translates to:</span></span>

<span data-ttu-id="262c3-273">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Form)]</span><span class="sxs-lookup"><span data-stu-id="262c3-273">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Form)]</span></span>

> [!WARNING]
> <span data-ttu-id="262c3-274">Leggere i valori del form solo se il tipo di contenuto sub *x-www-form-urlencoded* o *form-data*.</span><span class="sxs-lookup"><span data-stu-id="262c3-274">Read form values only if the content sub type is *x-www-form-urlencoded* or *form-data*.</span></span>

<span data-ttu-id="262c3-275">**HttpContext.Request.InputStream** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-275">**HttpContext.Request.InputStream** translates to:</span></span>

<span data-ttu-id="262c3-276">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Input)]</span><span class="sxs-lookup"><span data-stu-id="262c3-276">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Input)]</span></span>

> [!WARNING]
> <span data-ttu-id="262c3-277">Utilizzare questo codice solo in un middleware di tipo di gestore, alla fine di una pipeline.</span><span class="sxs-lookup"><span data-stu-id="262c3-277">Use this code only in a handler type middleware, at the end of a pipeline.</span></span>
>
><span data-ttu-id="262c3-278">È possibile leggere il corpo non elaborato, come illustrato in precedenza una sola volta per ogni richiesta.</span><span class="sxs-lookup"><span data-stu-id="262c3-278">You can read the raw body as shown above only once per request.</span></span> <span data-ttu-id="262c3-279">Tentativo di leggere il corpo dopo la prima lettura middleware leggerà un corpo vuoto.</span><span class="sxs-lookup"><span data-stu-id="262c3-279">Middleware trying to read the body after the first read will read an empty body.</span></span>
>
><span data-ttu-id="262c3-280">Questa opzione non viene applicata alla lettura di un modulo come illustrato in precedenza, perché questa operazione viene effettuata da un buffer.</span><span class="sxs-lookup"><span data-stu-id="262c3-280">This does not apply to reading a form as shown earlier, because that is done from a buffer.</span></span>

### <a name="httpcontextresponse"></a><span data-ttu-id="262c3-281">HttpContext.Response</span><span class="sxs-lookup"><span data-stu-id="262c3-281">HttpContext.Response</span></span>

<span data-ttu-id="262c3-282">**HttpContext.Response.Status** e **HttpContext.Response.StatusDescription** Traduci in:</span><span class="sxs-lookup"><span data-stu-id="262c3-282">**HttpContext.Response.Status** and **HttpContext.Response.StatusDescription** translate to:</span></span>

<span data-ttu-id="262c3-283">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Status)]</span><span class="sxs-lookup"><span data-stu-id="262c3-283">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Status)]</span></span>

<span data-ttu-id="262c3-284">**HttpContext.Response.ContentEncoding** e **HttpContext.Response.ContentType** Traduci in:</span><span class="sxs-lookup"><span data-stu-id="262c3-284">**HttpContext.Response.ContentEncoding** and **HttpContext.Response.ContentType** translate to:</span></span>

<span data-ttu-id="262c3-285">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_RespType)]</span><span class="sxs-lookup"><span data-stu-id="262c3-285">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_RespType)]</span></span>

<span data-ttu-id="262c3-286">**HttpContext.Response.ContentType** sul proprio anche viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-286">**HttpContext.Response.ContentType** on its own also translates to:</span></span>

<span data-ttu-id="262c3-287">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_RespTypeOnly)]</span><span class="sxs-lookup"><span data-stu-id="262c3-287">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_RespTypeOnly)]</span></span>

<span data-ttu-id="262c3-288">**HttpContext.Response.Output** viene convertito in:</span><span class="sxs-lookup"><span data-stu-id="262c3-288">**HttpContext.Response.Output** translates to:</span></span>

<span data-ttu-id="262c3-289">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Output)]</span><span class="sxs-lookup"><span data-stu-id="262c3-289">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Output)]</span></span>

<span data-ttu-id="262c3-290">**HttpContext.Response.TransmitFile**</span><span class="sxs-lookup"><span data-stu-id="262c3-290">**HttpContext.Response.TransmitFile**</span></span>

<span data-ttu-id="262c3-291">Disporre di un file è descritto [qui](../fundamentals/request-features.md#middleware-and-request-features).</span><span class="sxs-lookup"><span data-stu-id="262c3-291">Serving up a file is discussed [here](../fundamentals/request-features.md#middleware-and-request-features).</span></span>

<span data-ttu-id="262c3-292">**HttpContext.Response.Headers**</span><span class="sxs-lookup"><span data-stu-id="262c3-292">**HttpContext.Response.Headers**</span></span>

<span data-ttu-id="262c3-293">L'invio delle intestazioni di risposta è complicato dal fatto che se si imposta dopo qualsiasi elemento è stato scritto per il corpo della risposta, non verrà inviati.</span><span class="sxs-lookup"><span data-stu-id="262c3-293">Sending response headers is complicated by the fact that if you set them after anything has been written to the response body, they will not be sent.</span></span>

<span data-ttu-id="262c3-294">La soluzione consiste nell'impostare un metodo di callback che verrà chiamato destra prima della scrittura dell'inizio della risposta.</span><span class="sxs-lookup"><span data-stu-id="262c3-294">The solution is to set a callback method that will be called right before writing to the response starts.</span></span> <span data-ttu-id="262c3-295">Questo è più adatto all'inizio del `Invoke` metodo del middleware.</span><span class="sxs-lookup"><span data-stu-id="262c3-295">This is best done at the start of the `Invoke` method in your middleware.</span></span> <span data-ttu-id="262c3-296">È il metodo di callback che imposta le intestazioni di risposta.</span><span class="sxs-lookup"><span data-stu-id="262c3-296">It is this callback method that sets your response headers.</span></span>

<span data-ttu-id="262c3-297">Il codice seguente imposta un metodo di callback chiamato `SetHeaders`:</span><span class="sxs-lookup"><span data-stu-id="262c3-297">The following code sets a callback method called `SetHeaders`:</span></span>

```csharp
public async Task Invoke(HttpContext httpContext)
{
    // ...
    httpContext.Response.OnStarting(SetHeaders, state: httpContext);
```

<span data-ttu-id="262c3-298">Il `SetHeaders` metodo di callback è analogo al seguente:</span><span class="sxs-lookup"><span data-stu-id="262c3-298">The `SetHeaders` callback method would look like this:</span></span>

<span data-ttu-id="262c3-299">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_SetHeaders)]</span><span class="sxs-lookup"><span data-stu-id="262c3-299">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_SetHeaders)]</span></span>

<span data-ttu-id="262c3-300">**HttpContext.Response.Cookies**</span><span class="sxs-lookup"><span data-stu-id="262c3-300">**HttpContext.Response.Cookies**</span></span>

<span data-ttu-id="262c3-301">Si spostano i cookie nel browser in un *Set-Cookie* intestazione della risposta.</span><span class="sxs-lookup"><span data-stu-id="262c3-301">Cookies travel to the browser in a *Set-Cookie* response header.</span></span> <span data-ttu-id="262c3-302">Di conseguenza, inviare cookie richiede il callback stesso utilizzato per l'invio delle intestazioni di risposta:</span><span class="sxs-lookup"><span data-stu-id="262c3-302">As a result, sending cookies requires the same callback as used for sending response headers:</span></span>

```csharp
public async Task Invoke(HttpContext httpContext)
{
    // ...
    httpContext.Response.OnStarting(SetCookies, state: httpContext);
    httpContext.Response.OnStarting(SetHeaders, state: httpContext);
```

<span data-ttu-id="262c3-303">Il `SetCookies` metodo di callback avrà un aspetto simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="262c3-303">The `SetCookies` callback method would look like the following:</span></span>

<span data-ttu-id="262c3-304">[!code-csharp[Principale](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_SetCookies)]</span><span class="sxs-lookup"><span data-stu-id="262c3-304">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_SetCookies)]</span></span>

## <a name="additional-resources"></a><span data-ttu-id="262c3-305">Risorse aggiuntive</span><span class="sxs-lookup"><span data-stu-id="262c3-305">Additional Resources</span></span>

* [<span data-ttu-id="262c3-306">Panoramica di moduli HTTP e i gestori HTTP</span><span class="sxs-lookup"><span data-stu-id="262c3-306">HTTP Handlers and HTTP Modules Overview</span></span>](https://docs.microsoft.com/iis/configuration/system.webserver/)

* [<span data-ttu-id="262c3-307">Configurazione</span><span class="sxs-lookup"><span data-stu-id="262c3-307">Configuration</span></span>](../fundamentals/configuration.md)

* [<span data-ttu-id="262c3-308">Avvio dell'applicazione</span><span class="sxs-lookup"><span data-stu-id="262c3-308">Application Startup</span></span>](../fundamentals/startup.md)

* [<span data-ttu-id="262c3-309">Middleware</span><span class="sxs-lookup"><span data-stu-id="262c3-309">Middleware</span></span>](../fundamentals/middleware.md)