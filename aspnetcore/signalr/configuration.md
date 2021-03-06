---
title: Configurazione di ASP.NET Core SignalR
author: tdykstra
description: Informazioni su come configurare le app ASP.NET Core SignalR.
monikerRange: '>= aspnetcore-2.1'
ms.author: tdykstra
ms.custom: mvc
ms.date: 09/06/2018
uid: signalr/configuration
ms.openlocfilehash: 72fc53cad7caf55e85d0668c9dbea1a70fc8674b
ms.sourcegitcommit: c684eb6c0999d11d19e15e65939e5c7f99ba47df
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/19/2018
ms.locfileid: "46292336"
---
# <a name="aspnet-core-signalr-configuration"></a>Configurazione di ASP.NET Core SignalR

## <a name="jsonmessagepack-serialization-options"></a>Opzioni di serializzazione JSON/MessagePack

ASP.NET Core SignalR supporta due protocolli per la codifica messaggi: [JSON](https://www.json.org/) e [MessagePack](https://msgpack.org/index.html). Ogni protocollo presenta le opzioni di configurazione della serializzazione.

Serializzazione JSON può essere configurata nel server usando il [AddJsonProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.jsonprotocoldependencyinjectionextensions.addjsonprotocol) metodo di estensione, che può essere aggiunto dopo [AddSignalR](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr) nel `Startup.ConfigureServices` (metodo). Il `AddJsonProtocol` metodo accetta un delegato che riceve un `options` oggetto. Il [PayloadSerializerSettings](/dotnet/api/microsoft.aspnetcore.signalr.jsonhubprotocoloptions.payloadserializersettings) proprietà su tale oggetto è un JSON.NET `JsonSerializerSettings` oggetto che può essere utilizzato per configurare la serializzazione di argomenti e valori restituiti. Vedere le [documentazione di JSON.NET](https://www.newtonsoft.com/json/help/html/Introduction.htm) per altri dettagli.

Ad esempio, per configurare il serializzatore per usare i nomi delle proprietà "PascalCase", anziché i nomi predefiniti "camelCase", usare il codice seguente:

```csharp
services.AddSignalR()
    .AddJsonProtocol(options => {
        options.PayloadSerializerSettings.ContractResolver = 
        new DefaultContractResolver();
    });
```

Nel client .NET, lo stesso `AddJsonProtocol` metodo di estensione esiste sul [HubConnectionBuilder](/dotnet/api/microsoft.aspnetcore.signalr.client.hubconnectionbuilder). Il `Microsoft.Extensions.DependencyInjection` dello spazio dei nomi deve essere importato per risolvere il metodo di estensione:

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
> Non è possibile configurare la serializzazione JSON nel client JavaScript in questo momento.

### <a name="messagepack-serialization-options"></a>Opzioni di serializzazione MessagePack

MessagePack serializzazione può essere configurata specificando un delegato per il [AddMessagePackProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.msgpackprotocoldependencyinjectionextensions.addmessagepackprotocol) chiamare. Visualizzare [MessagePack in SignalR](xref:signalr/messagepackhubprotocol) per altri dettagli.

> [!NOTE]
> Non è possibile configurare la serializzazione MessagePack nel client JavaScript in questo momento.

## <a name="configure-server-options"></a>Configurare le opzioni server

La tabella seguente descrive le opzioni per la configurazione degli hub SignalR:

| Opzione | Valore predefinito | Descrizione |
| ------ | ------------- | ----------- |
| `HandshakeTimeout` | 15 secondi | Se il client non invia un messaggio di handshake iniziale all'interno di questo intervallo di tempo, la connessione viene chiusa. Si tratta di un'impostazione avanzata che deve essere modificata solo se si verificano a causa della latenza di rete gravi errori di timeout di handshake. Per informazioni dettagliate sul processo di handshake, vedere la [specifica del protocollo dell'Hub SignalR](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md). |
| `KeepAliveInterval` | 15 secondi | Se il server non ha inviato un messaggio all'interno di questo intervallo, un messaggio ping viene inviato automaticamente a mantenere aperta la connessione. Quando si modificano `KeepAliveInterval`, modificare il `ServerTimeout` / `serverTimeoutInMilliseconds` impostazione sul client. L'elemento consigliato `ServerTimeout` / `serverTimeoutInMilliseconds` valore è double il `KeepAliveInterval` valore.  |
| `SupportedProtocols` | Tutti i protocolli installati | Protocolli supportati da questo hub. Per impostazione predefinita, sono consentiti tutti i protocolli registrati nel server, ma i protocolli possono essere rimossi da questo elenco per disabilitare i protocolli specifici per singoli hub. |
| `EnableDetailedErrors` | `false` | Se `true`dettagliati vengono restituiti i messaggi di eccezione ai client quando viene generata un'eccezione in un metodo dell'Hub. Il valore predefinito è `false`, come i messaggi di eccezione possono contenere informazioni sensibili. |

Opzioni possono essere configurate per tutti gli hub, fornendo un delegato di opzioni per la `AddSignalR` chiamare in `Startup.ConfigureServices`.

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

Le opzioni per un singolo hub ignorare le opzioni globali disponibili in `AddSignalR` e possono essere configurati usando [AddHubOptions\<T >](/dotnet/api/microsoft.extensions.dependencyinjection.huboptionsdependencyinjectionextensions.addhuboptions):

```csharp
services.AddSignalR().AddHubOptions<MyHub>(options =>
{
    options.EnableDetailedErrors = true;
});
```

Usare `HttpConnectionDispatcherOptions` per configurare impostazioni avanzate per i trasporti e gestione della memoria buffer. Queste opzioni vengono configurate passando un delegato [MapHub\<T >](/dotnet/api/microsoft.aspnetcore.signalr.hubroutebuilder.maphub).

| Opzione | Valore predefinito | Descrizione |
| ------ | ------------- | ----------- |
| `ApplicationMaxBufferSize` | 32 KB | Il numero massimo di byte ricevuti dal client che il buffer del server. Se si aumenta questo valore consente al server ricevere messaggi di grandi dimensioni, ma può influire negativamente sul consumo di memoria. |
| `AuthorizationData` | Dati raccolti automaticamente dal `Authorize` gli attributi applicati alla classe dell'Hub. | Un elenco delle [IAuthorizeData](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizedata) oggetti utilizzati per determinare se un client è autorizzato a connettersi all'hub. |
| `TransportMaxBufferSize` | 32 KB | Il numero massimo di byte inviati dall'app che il buffer del server. Se si aumenta questo valore consente al server di inviare messaggi di grandi dimensioni, ma può influire negativamente sul consumo di memoria. |
| `Transports` | Tutti i trasporti sono abilitati. | Maschera di bit del `HttpTransportType` valori che è possono limitare i trasporti un client può usare per la connessione. |
| `LongPolling` | come illustrato più avanti. | Opzioni aggiuntive specifiche per il trasporto di Polling lungo. |
| `WebSockets` | come illustrato più avanti. | Opzioni aggiuntive specifiche per il trasporto WebSocket. |

Il trasporto di Polling lungo è presenti opzioni aggiuntive che possono essere configurate usando il `LongPolling` proprietà:

| Opzione | Valore predefinito | Descrizione |
| ------ | ------------- | ----------- |
| `PollTimeout` | 90 secondi | La quantità massima di tempo il server attende un messaggio da inviare al client prima di terminare una richiesta di poll singolo. Riduzione di questo valore fa sì che il client inviare nuove richieste di polling più frequente. |

Il trasporto WebSocket è presenti opzioni aggiuntive che possono essere configurate usando il `WebSockets` proprietà:

| Opzione | Valore predefinito | Descrizione |
| ------ | ------------- | ----------- |
| `CloseTimeout` | 5 secondi | Dopo aver chiuso il server, se il client non riesce a chiudere entro questo intervallo di tempo, la connessione viene terminata. |
| `SubProtocolSelector` | `null` | Un delegato che può essere utilizzato per impostare il `Sec-WebSocket-Protocol` intestazione su un valore personalizzato. Il delegato riceve i valori richiesti dal client come input e deve restituire il valore desiderato. |

## <a name="configure-client-options"></a>Configurare le opzioni client

Opzioni client possono essere configurate nella `HubConnectionBuilder` tipo, disponibile nel client sia .NET sia JavaScript, nonché nel `HubConnection` stesso.

### <a name="configure-logging"></a>Configurare la registrazione

La registrazione è configurata nel Client .NET usando il `ConfigureLogging` (metodo). Registrazione provider e i filtri possono essere registrata nello stesso modo come sono nel server. Vedere le [registrazione in ASP.NET Core](xref:fundamentals/logging/index#how-to-add-providers) per altre informazioni.

> [!NOTE]
> Per registrare i provider di registrazione, è necessario installare i pacchetti necessari. Vedere le [provider di registrazione predefiniti](xref:fundamentals/logging/index#built-in-logging-providers) sezione della documentazione per un elenco completo.

Ad esempio, per abilitare la registrazione della Console, installare il `Microsoft.Extensions.Logging.Console` pacchetto NuGet. Chiamare il `AddConsole` metodo di estensione:

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub")
    .ConfigureLogging(logging => {
        logging.SetMinimumLevel(LogLevel.Information);
        logging.AddConsole();
    })
    .Build();
```

Nel client JavaScript, un simile `configureLogging` esiste alcun metodo. Fornire un `LogLevel` valore che indica il livello minimo di messaggi di log da produrre. I log vengono scritti nella finestra della console del browser.

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub")
    .configureLogging(signalR.LogLevel.Information)
    .build();
```

> [!NOTE]
> Per disabilitare completamente la registrazione, specificare `signalR.LogLevel.None` nella `configureLogging` (metodo).

Livelli di log disponibili per il client JavaScript sono elencati di seguito. Impostazione del livello di log a uno di questi valori consente la registrazione di messaggi **o versione successiva** tale livello.

| Livello | Descrizione |
| ----- | ----------- |
| `None` | Viene registrato alcun messaggio. |
| `Critical` | Messaggi che indicano un errore nell'intera app. |
| `Error` | Messaggi che indicano un errore dell'operazione corrente. |
| `Warning` | Messaggi che indicano un problema non grave. |
| `Information` | Messaggi informativi. |
| `Debug` | Messaggi di diagnostica utili per il debug. |
| `Trace` | Messaggi di diagnostica molto dettagliati progettati per la diagnosi dei problemi specifici. |

### <a name="configure-allowed-transports"></a>Configurare i trasporti consentiti

È possibile configurare i trasporti utilizzati da SignalR nel `WithUrl` chiamare (`withUrl` in JavaScript). Un OR bit per bit dei valori di `HttpTransportType` può essere utilizzato per limitare il client per usare solo i tipi di trasporto specificati. Tutti i trasporti sono abilitati per impostazione predefinita.

Ad esempio, per disabilitare il trasporto di eventi Server-Sent, ma consentire le connessioni WebSocket e di Polling lungo:

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub", HttpTransportType.WebSockets | HttpTransportType.LongPolling)
    .Build();
```

Nel client JavaScript, i trasporti vengono configurati impostando il `transport` campo per l'oggetto di opzioni fornita a `withUrl`:

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub", { transport: signalR.HttpTransportType.WebSockets | signalR.HttpTransportType.LongPolling })
    .build();
```

### <a name="configure-bearer-authentication"></a>Configurare l'autenticazione della connessione

Per fornire i dati di autenticazione con le richieste di SignalR, usare il `AccessTokenProvider` opzione (`accessTokenFactory` in JavaScript) per specificare una funzione che restituisce il token di accesso desiderato. Nel Client .NET, questo token di accesso viene passato come un HTTP "Autenticazione della connessione" token (tramite il `Authorization` intestazione con un tipo di `Bearer`). Nel client JavaScript, il token di accesso viene utilizzato come un token di connessione **eccetto** in alcuni casi in cui browser API limitare la possibilità di applicare le intestazioni (in particolare, in richieste Server-Sent eventi e WebSockets). In questi casi, il token di accesso viene fornito come valore stringa di query `access_token`.

Nel client .NET, il `AccessTokenProvider` opzione può essere specificata tramite il delegato di opzioni in `WithUrl`:

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub", options => {
        options.AccessTokenProvider = async () => {
            // Get and return the access token.
        };
    })
    .Build();
```

Nel client JavaScript, il token di accesso viene configurato impostando la `accessTokenFactory` campo per l'oggetto di opzioni in `withUrl`:

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

### <a name="configure-timeout-and-keep-alive-options"></a>Configurare i timeout e opzioni keep-alive

Sono disponibili in opzioni aggiuntive per la configurazione di timeout e il comportamento keep-alive di `HubConnection` oggetto stesso:

| Opzione di .NET | Opzione di JavaScript | Valore predefinito | Descrizione |
| ----------- | ----------------- | ------------- | ----------- |
| `ServerTimeout` | `serverTimeoutInMilliseconds` | 30 secondi (30.000 millisecondi) | Timeout per l'attività del server. Se il server non ha inviato un messaggio in questo intervallo, il client considera il trigger server disconnessa la `Closed` evento (`onclose` in JavaScript). Questo valore deve essere sufficientemente grande essere inviati dal server di un messaggio ping **e** ricevuti dal client entro l'intervallo di timeout. Il valore consigliato è un numero almeno il doppio server `KeepAliveInterval` valore, per consentire tempo perché ping in arrivo. |
| `HandshakeTimeout` | Non è configurabile | 15 secondi | Timeout per l'handshake iniziale del server. Se il server non invia una risposta di handshake in questo intervallo, il client viene annullata l'handshake e trigger la `Closed` evento (`onclose` in JavaScript). Si tratta di un'impostazione avanzata che deve essere modificata solo se si verificano a causa della latenza di rete gravi errori di timeout di handshake. Per informazioni dettagliate sul processo di Handshake, vedere la [specifica del protocollo dell'Hub SignalR](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md). |

Nel Client .NET, i valori di timeout vengono specificati come `TimeSpan` valori. Nel client JavaScript, i valori di timeout vengono specificati come un numero che indica la durata in millisecondi.

### <a name="configure-additional-options"></a>Configurare le opzioni aggiuntive

Opzioni aggiuntive possono essere configurate nel `WithUrl` (`withUrl` in JavaScript) metodo `HubConnectionBuilder`:

| Opzione di .NET | Opzione di JavaScript | Valore predefinito | Descrizione |
| ----------- | ----------------- | ------------- | ----------- |
| `AccessTokenProvider` | `accessTokenFactory` | `null` | Una funzione che restituisce una stringa che viene fornita come un token di autenticazione nelle richieste HTTP. |
| `SkipNegotiation` | `skipNegotiation` | `false` | Impostare questa proprietà su `true` per ignorare la fase di negoziazione. **Supportato solo quando il trasporto abilitato solo il trasporto WebSocket**. Questa impostazione non può essere abilitata quando si usa il servizio Azure SignalR. |
| `ClientCertificates` | Non è configurabile * | Empty | Raccolta di certificati TLS da inviare autenticare le richieste. |
| `Cookies` | Non è configurabile * | Empty | Una raccolta di cookie HTTP da inviare con ogni richiesta HTTP. |
| `Credentials` | Non è configurabile * | Empty | Credenziali da inviare con ogni richiesta HTTP. |
| `CloseTimeout` | Non è configurabile * | 5 secondi | Solo WebSocket. La quantità massima di tempo è in attesa dopo la chiusura per il server confermare la richiesta di chiusura al client. Se il server non riconosciuto alla chiusura entro questo intervallo, il client si disconnette. |
| `Headers` | Non è configurabile * | Empty | Dizionario di intestazioni HTTP aggiuntive da inviare con ogni richiesta HTTP. |
| `HttpMessageHandlerFactory` | Non è configurabile * | `null` | Un delegato che può essere utilizzato per configurare o sostituire il `HttpMessageHandler` usato per inviare richieste HTTP. Non è utilizzato per le connessioni di WebSocket. Questo delegato deve restituire un valore diverso da null, e riceve il valore predefinito come parametro. Modificare le impostazioni del valore predefinito e restituirlo o restituire un nuovo `HttpMessageHandler` istanza. **Quando si sostituisce il gestore assicurarsi di copiare le impostazioni desiderate per impedire che il gestore fornito, in caso contrario, le opzioni configurate (ad esempio cookie e intestazioni) non si applicano al nuovo gestore.** |
| `Proxy` | Non è configurabile * | `null` | Un proxy HTTP da usare quando si inviano richieste HTTP. |
| `UseDefaultCredentials` | Non è configurabile * | `false` | Impostare il valore booleano per inviare le credenziali predefinite per le richieste HTTP e WebSocket. In questo modo l'uso dell'autenticazione di Windows. |
| `WebSocketConfiguration` | Non è configurabile * | `null` | Delegato che può essere utilizzato per configurare le opzioni aggiuntive di WebSocket. Riceve un'istanza di [ClientWebSocketOptions](/dotnet/api/system.net.websockets.clientwebsocketoptions) che può essere utilizzato per configurare le opzioni. |

Opzioni contrassegnate con un asterisco (*) non sono configurabili nel client JavaScript, a causa delle limitazioni nel browser API.

Nel Client .NET, queste opzioni possono essere modificate dal delegato opzioni fornito a `WithUrl`:

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub", options => {
        options.Headers["Foo"] = "Bar";
        options.Cookies.Add(new Cookie(/* ... */);
        options.ClientCertificates.Add(/* ... */);
    })
    .Build();
```

Nel JavaScript Client, queste opzioni possono essere fornite in un oggetto JavaScript fornito a `withUrl`:

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/myhub", {
        skipNegotiation: true,
        transport: signalR.HttpTransportType.WebSockets
    })
    .build();
```

## <a name="additional-resources"></a>Risorse aggiuntive

* <xref:tutorials/signalr>
* <xref:signalr/hubs>
* <xref:signalr/javascript-client>
* <xref:signalr/dotnet-client>
* <xref:signalr/messagepackhubprotocol>
* <xref:signalr/supported-platforms>
