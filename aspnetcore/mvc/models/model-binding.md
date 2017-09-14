---
title: Associazione di modelli
author: rick-anderson
description: 
keywords: ASP.NET Core,
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: b355a48e-a15c-4d58-b69c-899763613a97
ms.technology: aspnet
ms.prod: asp.net-core
uid: mvc/models/model-binding
ms.openlocfilehash: 597d4058a410e0b5991b1d5a74c9fc7bfe8171b8
ms.sourcegitcommit: 9cdbfd0d670d70b9c354216aabee260c52dad5ee
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/12/2017
---
# <a name="model-binding"></a><span data-ttu-id="8af87-103">Associazione di modelli</span><span class="sxs-lookup"><span data-stu-id="8af87-103">Model Binding</span></span>

<span data-ttu-id="8af87-104">Da [Rachel Appel](https://github.com/rachelappel)</span><span class="sxs-lookup"><span data-stu-id="8af87-104">By [Rachel Appel](https://github.com/rachelappel)</span></span>

## <a name="introduction-to-model-binding"></a><span data-ttu-id="8af87-105">Introduzione al modello di associazione</span><span class="sxs-lookup"><span data-stu-id="8af87-105">Introduction to model binding</span></span>

<span data-ttu-id="8af87-106">Associazione di modelli in ASP.NET MVC di base esegue il mapping dei dati dalle richieste HTTP ai parametri di metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="8af87-106">Model binding in ASP.NET Core MVC maps data from HTTP requests to action method parameters.</span></span> <span data-ttu-id="8af87-107">I parametri possono essere tipi semplici, ad esempio stringhe, interi o valori float o possono essere tipi complessi.</span><span class="sxs-lookup"><span data-stu-id="8af87-107">The parameters may be simple types such as strings, integers, or floats, or they may be complex types.</span></span> <span data-ttu-id="8af87-108">In questo modo una grande funzionalità di MVC mapping di dati in ingresso a un equivalente è uno scenario spesso ripetuto, indipendentemente dalla dimensione o della complessità dei dati.</span><span class="sxs-lookup"><span data-stu-id="8af87-108">This is a great feature of MVC because mapping incoming data to a counterpart is an often repeated scenario, regardless of size or complexity of the data.</span></span> <span data-ttu-id="8af87-109">MVC risolve questo problema, astrarre associazione in modo gli sviluppatori non sono necessario mantenere la riscrittura di una versione leggermente diversa di tale codice stesso in tutte le applicazioni.</span><span class="sxs-lookup"><span data-stu-id="8af87-109">MVC solves this problem by abstracting binding away so developers don't have to keep rewriting a slightly different version of that same code in every app.</span></span> <span data-ttu-id="8af87-110">Scrittura del testo per digitare il codice del convertitore risulta difficile e soggetta a errori.</span><span class="sxs-lookup"><span data-stu-id="8af87-110">Writing your own text to type converter code is tedious, and error prone.</span></span>

## <a name="how-model-binding-works"></a><span data-ttu-id="8af87-111">Funzionamento di associazione del modello</span><span class="sxs-lookup"><span data-stu-id="8af87-111">How model binding works</span></span>

<span data-ttu-id="8af87-112">Quando MVC riceve una richiesta HTTP, viene instradato a un metodo di azione specifica di un controller.</span><span class="sxs-lookup"><span data-stu-id="8af87-112">When MVC receives an HTTP request, it routes it to a specific action method of a controller.</span></span> <span data-ttu-id="8af87-113">Determina il metodo di azione da eseguire in base alle quali è nei dati di route, quindi associa i valori dalla richiesta HTTP per i parametri del metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="8af87-113">It determines which action method to run based on what is in the route data, then it binds values from the HTTP request to that action method's parameters.</span></span> <span data-ttu-id="8af87-114">Si consideri ad esempio l'URL seguente:</span><span class="sxs-lookup"><span data-stu-id="8af87-114">For example, consider the following URL:</span></span>

`http://contoso.com/movies/edit/2`

<span data-ttu-id="8af87-115">Poiché il modello di route è simile al seguente, `{controller=Home}/{action=Index}/{id?}`, `movies/edit/2` instrada il `Movies` controller e il relativo `Edit` metodo di azione.</span><span class="sxs-lookup"><span data-stu-id="8af87-115">Since the route template looks like this, `{controller=Home}/{action=Index}/{id?}`, `movies/edit/2` routes to the `Movies` controller, and its `Edit` action method.</span></span> <span data-ttu-id="8af87-116">Tale metodo accetta inoltre un parametro facoltativo denominato `id`.</span><span class="sxs-lookup"><span data-stu-id="8af87-116">It also accepts an optional parameter called `id`.</span></span> <span data-ttu-id="8af87-117">Il codice per il metodo di azione dovrebbe essere simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="8af87-117">The code for the action method should look something like this:</span></span>

<!-- literal_block {"ids": [], "linenos": true, "xml:space": "preserve", "language": "csharp"} -->

```csharp
public IActionResult Edit(int? id)
   ```

<span data-ttu-id="8af87-118">Nota: Le stringhe della route dell'URL non sono tra maiuscole e minuscole.</span><span class="sxs-lookup"><span data-stu-id="8af87-118">Note: The strings in the URL route are not case sensitive.</span></span>

<span data-ttu-id="8af87-119">MVC tenterà di associare i dati di richiesta per i parametri dell'azione in base al nome.</span><span class="sxs-lookup"><span data-stu-id="8af87-119">MVC will try to bind request data to the action parameters by name.</span></span> <span data-ttu-id="8af87-120">MVC cercherà i valori per ogni parametro utilizzando il nome di parametro e i nomi delle proprietà impostabili pubbliche.</span><span class="sxs-lookup"><span data-stu-id="8af87-120">MVC will look for values for each parameter using the parameter name and the names of its public settable properties.</span></span> <span data-ttu-id="8af87-121">Nell'esempio precedente, il parametro sola azione è denominato `id`, MVC associa il valore con lo stesso nome nei valori di route.</span><span class="sxs-lookup"><span data-stu-id="8af87-121">In the above example, the only action parameter is named `id`, which MVC binds to the value with the same name in the route values.</span></span> <span data-ttu-id="8af87-122">Oltre ai valori di route MVC assocerà dati da varie parti della richiesta ed esegue l'operazione in un determinato ordine.</span><span class="sxs-lookup"><span data-stu-id="8af87-122">In addition to route values MVC will bind data from various parts of the request and it does so in a set order.</span></span> <span data-ttu-id="8af87-123">Di seguito è riportato un elenco delle origini dati nell'ordine in cui li esamina l'associazione di modelli:</span><span class="sxs-lookup"><span data-stu-id="8af87-123">Below is a list of the data sources in the order that model binding looks through them:</span></span>

1. <span data-ttu-id="8af87-124">`Form values`: Si tratta di valori del form che nella richiesta HTTP con il metodo POST.</span><span class="sxs-lookup"><span data-stu-id="8af87-124">`Form values`: These are form values that go in the HTTP request using the POST method.</span></span> <span data-ttu-id="8af87-125">(incluse le richieste POST jQuery).</span><span class="sxs-lookup"><span data-stu-id="8af87-125">(including jQuery POST requests).</span></span>

2. <span data-ttu-id="8af87-126">`Route values`: Il set di valori di route disponibili in [Routing](../../fundamentals/routing.md)</span><span class="sxs-lookup"><span data-stu-id="8af87-126">`Route values`: The set of route values provided by [Routing](../../fundamentals/routing.md)</span></span>

3. <span data-ttu-id="8af87-127">`Query strings`: La parte di stringa di query dell'URI.</span><span class="sxs-lookup"><span data-stu-id="8af87-127">`Query strings`: The query string part of the URI.</span></span>

<!-- DocFX BUG
The link works but generates an error when building with DocFX
@fundamentals/routing
[Routing](xref:fundamentals/routing)
-->

<span data-ttu-id="8af87-128">Nota: Modulo valori, i dati di route e query tutte le stringhe vengono archiviate come coppie nome-valore.</span><span class="sxs-lookup"><span data-stu-id="8af87-128">Note: Form values, route data, and query strings are all stored as name-value pairs.</span></span>

<span data-ttu-id="8af87-129">Poiché l'associazione del modello richiesto per una chiave denominata `id` Nessun elemento denominato `id` nei valori del form, spostato per i valori della route per tale chiave.</span><span class="sxs-lookup"><span data-stu-id="8af87-129">Since model binding asked for a key named `id` and there is nothing named `id` in the form values, it moved on to the route values looking for that key.</span></span> <span data-ttu-id="8af87-130">In questo esempio, è una corrispondenza.</span><span class="sxs-lookup"><span data-stu-id="8af87-130">In our example, it's a match.</span></span> <span data-ttu-id="8af87-131">Si verifichi l'associazione e il valore viene convertito all'intero 2.</span><span class="sxs-lookup"><span data-stu-id="8af87-131">Binding happens, and the value is converted to the integer 2.</span></span> <span data-ttu-id="8af87-132">La stessa richiesta mediante modifica (id di stringa) potrebbe convertire la stringa "2".</span><span class="sxs-lookup"><span data-stu-id="8af87-132">The same request using Edit(string id) would convert to the string "2".</span></span>

<span data-ttu-id="8af87-133">Nell'esempio viene utilizzato finora tipi semplici.</span><span class="sxs-lookup"><span data-stu-id="8af87-133">So far the example uses simple types.</span></span> <span data-ttu-id="8af87-134">In MVC tipi semplici sono qualsiasi tipo primitivo .NET o con un convertitore di tipi di stringa.</span><span class="sxs-lookup"><span data-stu-id="8af87-134">In MVC simple types are any .NET primitive type or type with a string type converter.</span></span> <span data-ttu-id="8af87-135">Se il parametro del metodo di azione fosse una classe, ad esempio il `Movie` tipo, che contiene i tipi semplici e complessi come proprietà, modello associazione continuerà del MVC gestire correttamente.</span><span class="sxs-lookup"><span data-stu-id="8af87-135">If the action method's parameter were a class such as the `Movie` type, which contains both simple and complex types as properties, MVC's model binding will still handle it nicely.</span></span> <span data-ttu-id="8af87-136">Usa la reflection e la ricorsione per attraversare le proprietà di ricerca di corrispondenze di tipi complessi.</span><span class="sxs-lookup"><span data-stu-id="8af87-136">It uses reflection and recursion to traverse the properties of complex types looking for matches.</span></span> <span data-ttu-id="8af87-137">Esegue la ricerca per il modello di associazione di modelli *parameter_name.property_name* per associare i valori alle proprietà.</span><span class="sxs-lookup"><span data-stu-id="8af87-137">Model binding looks for the pattern *parameter_name.property_name* to bind values to properties.</span></span> <span data-ttu-id="8af87-138">Se i valori corrispondenti del modulo non viene trovato, verrà effettuato un tentativo di eseguire il binding utilizzando solo il nome della proprietà.</span><span class="sxs-lookup"><span data-stu-id="8af87-138">If it doesn't find matching values of this form, it will attempt to bind using just the property name.</span></span> <span data-ttu-id="8af87-139">Per i tipi, ad esempio `Collection` , tipi di associazione del modello esegue la ricerca di corrispondenze da *parameter_name [index]* o semplicemente *[index]*.</span><span class="sxs-lookup"><span data-stu-id="8af87-139">For those types such as `Collection` types, model binding looks for matches to *parameter_name[index]* or just *[index]*.</span></span> <span data-ttu-id="8af87-140">Modello di associazione considera `Dictionary` tipi in modo analogo, che richiede il *parameter_name [key]* o semplicemente *[key]*, purché le chiavi sono tipi semplici.</span><span class="sxs-lookup"><span data-stu-id="8af87-140">Model binding treats  `Dictionary` types similarly, asking for *parameter_name[key]* or just *[key]*, as long as the keys are simple types.</span></span> <span data-ttu-id="8af87-141">Le chiavi supportate corrispondono ai nomi di campo HTML e gli helper di tag generati per lo stesso tipo di modello.</span><span class="sxs-lookup"><span data-stu-id="8af87-141">Keys that are supported match the field names HTML and tag helpers generated for the same model type.</span></span> <span data-ttu-id="8af87-142">In questo modo i valori di andata e ritorno in modo che i campi modulo rimangano riempiti con l'input dell'utente per i motivi di praticità, ad esempio, quando i dati associati dalla creazione o modifica non ha superato la convalida.</span><span class="sxs-lookup"><span data-stu-id="8af87-142">This enables round-tripping values so that the form fields remain filled with the user's input for their convenience, for example, when bound data from a create or edit did not pass validation.</span></span>

<span data-ttu-id="8af87-143">In ordine di associazione consente di eseguire la classe deve avere un costruttore predefinito pubblico e membro sia associato deve essere pubblica proprietà accessibile in scrittura.</span><span class="sxs-lookup"><span data-stu-id="8af87-143">In order for binding to happen the class must have a public default constructor and member to be bound must be public writable properties.</span></span> <span data-ttu-id="8af87-144">Quando si verifica l'associazione di modelli che della classe sarà possibile creare istanze solo utilizzando il costruttore predefinito pubblico, è possono impostare le proprietà.</span><span class="sxs-lookup"><span data-stu-id="8af87-144">When model binding happens the class will only be instantiated using the public default constructor, then the properties can be set.</span></span>

<span data-ttu-id="8af87-145">Quando è associato un parametro, l'associazione di modelli arresta cercando valori con lo stesso nome e lo sposta associare il parametro successivo.</span><span class="sxs-lookup"><span data-stu-id="8af87-145">When a parameter is bound, model binding stops looking for values with that name and it moves on to bind the next parameter.</span></span> <span data-ttu-id="8af87-146">Se l'associazione non riesce, MVC genera un errore.</span><span class="sxs-lookup"><span data-stu-id="8af87-146">If binding fails, MVC does not throw an error.</span></span> <span data-ttu-id="8af87-147">È possibile eseguire query per gli errori di stato modello controllando il `ModelState.IsValid` proprietà.</span><span class="sxs-lookup"><span data-stu-id="8af87-147">You can query for model state errors by checking the `ModelState.IsValid` property.</span></span>

<span data-ttu-id="8af87-148">Nota: Ogni voce del controller `ModelState` proprietà è un `ModelStateEntry` contenente un `Errors property`.</span><span class="sxs-lookup"><span data-stu-id="8af87-148">Note: Each entry in the controller's `ModelState` property is a `ModelStateEntry` containing an `Errors property`.</span></span> <span data-ttu-id="8af87-149">È raramente la necessità di eseguire query in questa raccolta manualmente.</span><span class="sxs-lookup"><span data-stu-id="8af87-149">It's rarely necessary to query this collection yourself.</span></span> <span data-ttu-id="8af87-150">In alternativa, usare `ModelState.IsValid` .</span><span class="sxs-lookup"><span data-stu-id="8af87-150">Use `ModelState.IsValid` instead.</span></span>

<span data-ttu-id="8af87-151">Esistono inoltre alcuni tipi di dati speciale che deve prendere in considerazione quando si esegue l'associazione del modello MVC:</span><span class="sxs-lookup"><span data-stu-id="8af87-151">Additionally, there are some special data types that MVC must consider when performing model binding:</span></span>

* <span data-ttu-id="8af87-152">`IFormFile`, `IEnumerable<IFormFile>`: Uno o più file caricati che fanno parte della richiesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="8af87-152">`IFormFile`, `IEnumerable<IFormFile>`: One or more uploaded files that are part of the HTTP request.</span></span>

* <span data-ttu-id="8af87-153">`CancelationToken`: Usato per annullare l'attività in un controller asincrono.</span><span class="sxs-lookup"><span data-stu-id="8af87-153">`CancelationToken`: Used to cancel activity in asynchronous controllers.</span></span>

<span data-ttu-id="8af87-154">Questi tipi possono essere associati ai parametri di azione o alle proprietà a un tipo di classe.</span><span class="sxs-lookup"><span data-stu-id="8af87-154">These types can be bound to action parameters or to properties on a class type.</span></span>

<span data-ttu-id="8af87-155">Dopo aver completato l'associazione di modelli [convalida](validation.md) si verifica.</span><span class="sxs-lookup"><span data-stu-id="8af87-155">Once model binding is complete, [Validation](validation.md) occurs.</span></span> <span data-ttu-id="8af87-156">Associazione di modelli predefinito prestazioni ottimali per la maggior parte degli scenari di sviluppo.</span><span class="sxs-lookup"><span data-stu-id="8af87-156">Default model binding works great for the vast majority of development scenarios.</span></span> <span data-ttu-id="8af87-157">È inoltre estendibile in modo se si hanno esigenze specifiche, è possibile personalizzare il comportamento predefinito.</span><span class="sxs-lookup"><span data-stu-id="8af87-157">It is also extensible so if you have unique needs you can customize the built-in behavior.</span></span>

## <a name="customize-model-binding-behavior-with-attributes"></a><span data-ttu-id="8af87-158">Personalizzare il comportamento di associazione del modello con gli attributi</span><span class="sxs-lookup"><span data-stu-id="8af87-158">Customize model binding behavior with attributes</span></span>

<span data-ttu-id="8af87-159">MVC contiene numerosi attributi che è possibile utilizzare per indirizzare il comportamento di associazione del modello predefinito per un'origine diversa.</span><span class="sxs-lookup"><span data-stu-id="8af87-159">MVC contains several attributes that you can use to direct its default model binding behavior to a different source.</span></span> <span data-ttu-id="8af87-160">Ad esempio, è possibile specificare se l'associazione è obbligatoria per una proprietà o se non deve mai accadere in qualsiasi tramite il `[BindRequired]` o `[BindNever]` attributi.</span><span class="sxs-lookup"><span data-stu-id="8af87-160">For example, you can specify whether binding is required for a property, or if it should never happen at all by using the `[BindRequired]` or `[BindNever]` attributes.</span></span> <span data-ttu-id="8af87-161">In alternativa, è possibile sostituire l'origine dati predefinita e quindi specificare l'origine dati del gestore di associazione del modello.</span><span class="sxs-lookup"><span data-stu-id="8af87-161">Alternatively, you can override the default data source, and specify the model binder's data source.</span></span> <span data-ttu-id="8af87-162">Di seguito è un elenco di attributi di associazione del modello:</span><span class="sxs-lookup"><span data-stu-id="8af87-162">Below is a list of model binding attributes:</span></span>

* <span data-ttu-id="8af87-163">`[BindRequired]`: Questo attributo viene aggiunto un errore di stato del modello se è Impossibile eseguire l'associazione.</span><span class="sxs-lookup"><span data-stu-id="8af87-163">`[BindRequired]`: This attribute adds a model state error if binding cannot occur.</span></span>

* <span data-ttu-id="8af87-164">`[BindNever]`: Indica il raccoglitore di modelli per non associare a questo parametro.</span><span class="sxs-lookup"><span data-stu-id="8af87-164">`[BindNever]`: Tells the model binder to never bind to this parameter.</span></span>

* <span data-ttu-id="8af87-165">`[FromHeader]`, `[FromQuery]`, `[FromRoute]`, `[FromForm]`: Utilizzarli per specificare l'origine di associazione esatto si desidera applicare.</span><span class="sxs-lookup"><span data-stu-id="8af87-165">`[FromHeader]`, `[FromQuery]`, `[FromRoute]`, `[FromForm]`: Use these to specify the exact binding source you want to apply.</span></span>

* <span data-ttu-id="8af87-166">`[FromServices]`: Questo attributo utilizza [inserimento di dipendenze](../../fundamentals/dependency-injection.md) per associare i parametri di servizi.</span><span class="sxs-lookup"><span data-stu-id="8af87-166">`[FromServices]`: This attribute uses [dependency injection](../../fundamentals/dependency-injection.md) to bind parameters from services.</span></span>

* <span data-ttu-id="8af87-167">`[FromBody]`: Utilizzare i formattatori configurati per associare i dati dal corpo della richiesta.</span><span class="sxs-lookup"><span data-stu-id="8af87-167">`[FromBody]`: Use the configured formatters to bind data from the request body.</span></span> <span data-ttu-id="8af87-168">Il formattatore è selezionato in base al tipo di contenuto della richiesta.</span><span class="sxs-lookup"><span data-stu-id="8af87-168">The formatter is selected based on content type of the request.</span></span>

* <span data-ttu-id="8af87-169">`[ModelBinder]`: Usato per sostituire il gestore di associazione del modello predefinito, origine dell'associazione e il nome.</span><span class="sxs-lookup"><span data-stu-id="8af87-169">`[ModelBinder]`: Used to override the default model binder, binding source and name.</span></span>

<span data-ttu-id="8af87-170">Gli attributi sono strumenti molto utili quando è necessario eseguire l'override del comportamento predefinito di associazione del modello.</span><span class="sxs-lookup"><span data-stu-id="8af87-170">Attributes are very helpful tools when you need to override the default behavior of model binding.</span></span>

## <a name="binding-formatted-data-from-the-request-body"></a><span data-ttu-id="8af87-171">Associazione dati formattati da un corpo della richiesta</span><span class="sxs-lookup"><span data-stu-id="8af87-171">Binding formatted data from the request body</span></span>

<span data-ttu-id="8af87-172">Dati della richiesta sono disponibili in un'ampia gamma di formati quali JSON, XML e molti altri.</span><span class="sxs-lookup"><span data-stu-id="8af87-172">Request data can come in a variety of formats including JSON, XML and many others.</span></span> <span data-ttu-id="8af87-173">Quando si utilizza l'attributo [FromBody] per indicare che si desidera associare un parametro per i dati nel corpo della richiesta, MVC utilizza un set di formattatori di cui è stato configurato per gestire i dati della richiesta in base al tipo di contenuto.</span><span class="sxs-lookup"><span data-stu-id="8af87-173">When you use the [FromBody] attribute to indicate that you want to bind a parameter to data in the request body, MVC uses a configured set of formatters to handle the request data based on its content type.</span></span> <span data-ttu-id="8af87-174">Per impostazione predefinita, MVC include un `JsonInputFormatter` classe per la gestione di dati JSON, ma è possibile aggiungere altri formattatori per la gestione di XML e altri formati personalizzati.</span><span class="sxs-lookup"><span data-stu-id="8af87-174">By default MVC includes a `JsonInputFormatter` class for handling JSON data, but you can add additional formatters for handling XML and other custom formats.</span></span>

> [!NOTE]
> <span data-ttu-id="8af87-175">Può esistere al massimo un parametro per ogni azione decorati con `[FromBody]`.</span><span class="sxs-lookup"><span data-stu-id="8af87-175">There can be at most one parameter per action decorated with `[FromBody]`.</span></span> <span data-ttu-id="8af87-176">Runtime di ASP.NET MVC Core delega la responsabilità di lettura del flusso di richiesta al formattatore.</span><span class="sxs-lookup"><span data-stu-id="8af87-176">The ASP.NET Core MVC run-time delegates the responsibility of reading the request stream to the formatter.</span></span> <span data-ttu-id="8af87-177">Una volta il flusso di richiesta è di lettura per un parametro, non è in genere possibile leggere il flusso di richiesta nuovamente per l'associazione altri `[FromBody]` parametri.</span><span class="sxs-lookup"><span data-stu-id="8af87-177">Once the request stream is read for a parameter, it's generally not possible to read the request stream again for binding other `[FromBody]` parameters.</span></span>

> [!NOTE]
> <span data-ttu-id="8af87-178">Il `JsonInputFormatter` è il formattatore predefinito e si basa su [Json.NET](https://www.newtonsoft.com/json).</span><span class="sxs-lookup"><span data-stu-id="8af87-178">The `JsonInputFormatter` is the default formatter and is based on [Json.NET](https://www.newtonsoft.com/json).</span></span>

<span data-ttu-id="8af87-179">Formattatori di input in base a viene selezionato il [Content-Type](https://www.w3.org/Protocols/rfc1341/4_Content-Type.html) intestazione e il tipo del parametro, a meno che non vi è un attributo applicato alla funzione che specifica in caso contrario.</span><span class="sxs-lookup"><span data-stu-id="8af87-179">ASP.NET selects input formatters based on the [Content-Type](https://www.w3.org/Protocols/rfc1341/4_Content-Type.html) header and the type of the parameter, unless there is an attribute applied to it specifying otherwise.</span></span> <span data-ttu-id="8af87-180">Se si desidera utilizzare il codice XML o un altro formato, è necessario configurare nel *Startup.cs* file, ma è prima necessario ottenere un riferimento a `Microsoft.AspNetCore.Mvc.Formatters.Xml` tramite NuGet.</span><span class="sxs-lookup"><span data-stu-id="8af87-180">If you'd like to use XML or another format you must configure it in the *Startup.cs* file, but you may first have to obtain a reference to `Microsoft.AspNetCore.Mvc.Formatters.Xml` using NuGet.</span></span> <span data-ttu-id="8af87-181">Il codice di avvio dovrebbe essere simile al seguente:</span><span class="sxs-lookup"><span data-stu-id="8af87-181">Your startup code should look something like this:</span></span>

<!-- literal_block {"ids": [], "linenos": true, "xml:space": "preserve", "language": "csharp"} -->

```csharp
public void ConfigureServices(IServiceCollection services)
   {
       services.AddMvc()
          .AddXmlSerializerFormatters();
   }
   ```

<span data-ttu-id="8af87-182">Codice il *Startup.cs* file contiene un `ConfigureServices` metodo con un `services` argomento è possibile utilizzare per compilare i servizi per l'applicazione ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="8af87-182">Code in the *Startup.cs* file contains a `ConfigureServices` method with a `services` argument you can use to build up services for your ASP.NET app.</span></span> <span data-ttu-id="8af87-183">Nell'esempio, si aggiungono un formattatore XML come un servizio che fornisce per questa applicazione MVC.</span><span class="sxs-lookup"><span data-stu-id="8af87-183">In the sample, we are adding an XML formatter as a service that MVC will provide for this app.</span></span> <span data-ttu-id="8af87-184">Il `options` argomento passato il `AddMvc` metodo consente di aggiungere e gestire i filtri, i formattatori e altre opzioni di sistema da MVC dopo l'avvio delle app.</span><span class="sxs-lookup"><span data-stu-id="8af87-184">The `options` argument passed into the `AddMvc` method allows you to add and manage filters, formatters, and other system options from MVC upon app startup.</span></span> <span data-ttu-id="8af87-185">Quindi applicare il `Consumes` attributo alle classi controller o ai metodi di azione per lavorare con il formato desiderato.</span><span class="sxs-lookup"><span data-stu-id="8af87-185">Then apply the `Consumes` attribute to controller classes or action methods to work with the format you want.</span></span>

### <a name="custom-model-binding"></a><span data-ttu-id="8af87-186">Associazione di modelli personalizzati</span><span class="sxs-lookup"><span data-stu-id="8af87-186">Custom Model Binding</span></span>

<span data-ttu-id="8af87-187">Scrivere la propria raccoglitori di modelli personalizzati, è possibile estendere l'associazione di modelli.</span><span class="sxs-lookup"><span data-stu-id="8af87-187">You can extend model binding by writing your own custom model binders.</span></span> <span data-ttu-id="8af87-188">Altre informazioni, vedere [l'associazione di modelli personalizzati](../advanced/custom-model-binding.md).</span><span class="sxs-lookup"><span data-stu-id="8af87-188">Learn more about [custom model binding](../advanced/custom-model-binding.md).</span></span>