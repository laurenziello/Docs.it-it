---
title: Crittografia chiave
author: rick-anderson
description: 
keywords: ASP.NET Core,
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: f2bbbf4e-0945-43ce-be59-8bf19e448798
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/implementation/key-encryption-at-rest
ms.openlocfilehash: 16a9385630d88c4c9f33954f83fce2bbce5be719
ms.sourcegitcommit: 9cdbfd0d670d70b9c354216aabee260c52dad5ee
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 09/12/2017
---
# <a name="key-encryption-at-rest"></a><span data-ttu-id="6373e-103">Crittografia chiave</span><span class="sxs-lookup"><span data-stu-id="6373e-103">Key Encryption At Rest</span></span>

<a name=data-protection-implementation-key-encryption-at-rest></a>

<span data-ttu-id="6373e-104">Per impostazione predefinita il sistema di protezione dati [utilizza un approccio euristico](../configuration/default-settings.md#data-protection-default-settings) per determinare la modalità crittografia materiale della chiave devono essere crittografati a riposo.</span><span class="sxs-lookup"><span data-stu-id="6373e-104">By default the data protection system [employs a heuristic](../configuration/default-settings.md#data-protection-default-settings) to determine how cryptographic key material should be encrypted at rest.</span></span> <span data-ttu-id="6373e-105">Lo sviluppatore può eseguire l'override l'euristica e specificare manualmente la modalità di crittografia chiavi inattivi.</span><span class="sxs-lookup"><span data-stu-id="6373e-105">The developer can override the heuristic and manually specify how keys should be encrypted at rest.</span></span>

> [!NOTE]
> <span data-ttu-id="6373e-106">Se si specifica una crittografia a chiave esplicita al meccanismo di rest, il sistema di protezione dati verrà annullare la registrazione il meccanismo di archiviazione chiavi predefinito che ha fornito l'euristica indicata.</span><span class="sxs-lookup"><span data-stu-id="6373e-106">If you specify an explicit key encryption at rest mechanism, the data protection system will deregister the default key storage mechanism that the heuristic provided.</span></span> <span data-ttu-id="6373e-107">È necessario [specifica un meccanismo di archiviazione chiavi esplicita](key-storage-providers.md#data-protection-implementation-key-storage-providers), in caso contrario il sistema di protezione dati non verrà avviati.</span><span class="sxs-lookup"><span data-stu-id="6373e-107">You must [specify an explicit key storage mechanism](key-storage-providers.md#data-protection-implementation-key-storage-providers), otherwise the data protection system will fail to start.</span></span>

<a name=data-protection-implementation-key-encryption-at-rest-providers></a>

<span data-ttu-id="6373e-108">Il sistema di protezione dati viene fornito con tre meccanismi di crittografia della chiave nella casella.</span><span class="sxs-lookup"><span data-stu-id="6373e-108">The data protection system ships with three in-box key encryption mechanisms.</span></span>

## <a name="windows-dpapi"></a><span data-ttu-id="6373e-109">DPAPI di Windows</span><span class="sxs-lookup"><span data-stu-id="6373e-109">Windows DPAPI</span></span>

<span data-ttu-id="6373e-110">*Questo meccanismo è disponibile solo in Windows.*</span><span class="sxs-lookup"><span data-stu-id="6373e-110">*This mechanism is available only on Windows.*</span></span>

<span data-ttu-id="6373e-111">Quando viene utilizzato DPAPI di Windows, il materiale della chiave viene crittografato tramite [CryptProtectData](https://msdn.microsoft.com/library/windows/desktop/aa380261(v=vs.85).aspx) prima di essere resa persistente nell'archiviazione.</span><span class="sxs-lookup"><span data-stu-id="6373e-111">When Windows DPAPI is used, key material will be encrypted via [CryptProtectData](https://msdn.microsoft.com/library/windows/desktop/aa380261(v=vs.85).aspx) before being persisted to storage.</span></span> <span data-ttu-id="6373e-112">DPAPI è un meccanismo di crittografia appropriati per i dati non verranno letti mai di fuori di computer in uso (tuttavia è possibile eseguire il backup di queste chiavi fino a Active Directory, vedere [DPAPI e i profili](https://support.microsoft.com/kb/309408/#6)).</span><span class="sxs-lookup"><span data-stu-id="6373e-112">DPAPI is an appropriate encryption mechanism for data that will never be read outside of the current machine (though it is possible to back these keys up to Active Directory; see [DPAPI and Roaming Profiles](https://support.microsoft.com/kb/309408/#6)).</span></span> <span data-ttu-id="6373e-113">Ad esempio per configurare la crittografia della chiave a riposo DPAPI.</span><span class="sxs-lookup"><span data-stu-id="6373e-113">For example to configure DPAPI key-at-rest encryption.</span></span>

```csharp
sc.AddDataProtection()
       // only the local user account can decrypt the keys
       .ProtectKeysWithDpapi();
   ```

<span data-ttu-id="6373e-114">Se ProtectKeysWithDpapi viene chiamata senza parametri, solo l'account di utente di Windows corrente può decrittografare il materiale della chiave permanente.</span><span class="sxs-lookup"><span data-stu-id="6373e-114">If ProtectKeysWithDpapi is called with no parameters, only the current Windows user account can decipher the persisted key material.</span></span> <span data-ttu-id="6373e-115">È facoltativamente possibile specificare che tutti gli account utente del computer (non solo l'account utente corrente) deve essere in grado di decifrare il materiale della chiave, come illustrato nell'esempio seguente.</span><span class="sxs-lookup"><span data-stu-id="6373e-115">You can optionally specify that any user account on the machine (not just the current user account) should be able to decipher the key material, as shown in the below example.</span></span>

```csharp
sc.AddDataProtection()
       // all user accounts on the machine can decrypt the keys
       .ProtectKeysWithDpapi(protectToLocalMachine: true);
   ```

## <a name="x509-certificate"></a><span data-ttu-id="6373e-116">Certificato x. 509</span><span class="sxs-lookup"><span data-stu-id="6373e-116">X.509 certificate</span></span>

<span data-ttu-id="6373e-117">*Questo meccanismo non è disponibile in `.NET Core 1.0` o `1.1`.*</span><span class="sxs-lookup"><span data-stu-id="6373e-117">*This mechanism is not available on `.NET Core 1.0` or `1.1`.*</span></span>

<span data-ttu-id="6373e-118">Se l'applicazione è suddiviso in più computer, potrebbe essere opportuno distribuire un certificato x. 509 condiviso tra il computer e configurare le applicazioni di utilizzare il certificato per la crittografia delle chiavi inattivi.</span><span class="sxs-lookup"><span data-stu-id="6373e-118">If your application is spread across multiple machines, it may be convenient to distribute a shared X.509 certificate across the machines and to configure applications to use this certificate for encryption of keys at rest.</span></span> <span data-ttu-id="6373e-119">Per un esempio, vedere di seguito.</span><span class="sxs-lookup"><span data-stu-id="6373e-119">See below for an example.</span></span>

```csharp
sc.AddDataProtection()
       // searches the cert store for the cert with this thumbprint
       .ProtectKeysWithCertificate("3BCE558E2AD3E0E34A7743EAB5AEA2A9BD2575A0");
   ```

<span data-ttu-id="6373e-120">A causa di limitazioni di .NET Framework sono supportati solo i certificati con chiavi private CryptoAPI.</span><span class="sxs-lookup"><span data-stu-id="6373e-120">Due to .NET Framework limitations only certificates with CAPI private keys are supported.</span></span> <span data-ttu-id="6373e-121">Vedere [crittografia basata su certificati con Windows DPAPI-NG](#data-protection-implementation-key-encryption-at-rest-dpapi-ng) di sotto delle possibili soluzioni alternative per queste limitazioni.</span><span class="sxs-lookup"><span data-stu-id="6373e-121">See [Certificate-based encryption with Windows DPAPI-NG](#data-protection-implementation-key-encryption-at-rest-dpapi-ng) below for possible workarounds to these limitations.</span></span>

<a name=data-protection-implementation-key-encryption-at-rest-dpapi-ng></a>

## <a name="windows-dpapi-ng"></a><span data-ttu-id="6373e-122">DPAPI di Windows-NG</span><span class="sxs-lookup"><span data-stu-id="6373e-122">Windows DPAPI-NG</span></span>

<span data-ttu-id="6373e-123">*Questo meccanismo è disponibile solo in Windows 8 o Windows Server 2012 e versioni successive.*</span><span class="sxs-lookup"><span data-stu-id="6373e-123">*This mechanism is available only on Windows 8 / Windows Server 2012 and later.*</span></span>

<span data-ttu-id="6373e-124">A partire da Windows 8, il sistema operativo supporta DPAPI-NG (detto anche DPAPI CNG).</span><span class="sxs-lookup"><span data-stu-id="6373e-124">Beginning with Windows 8, the operating system supports DPAPI-NG (also called CNG DPAPI).</span></span> <span data-ttu-id="6373e-125">Microsoft disposto lo scenario di utilizzo come indicato di seguito.</span><span class="sxs-lookup"><span data-stu-id="6373e-125">Microsoft lays out its usage scenario as follows.</span></span>

   <span data-ttu-id="6373e-126">Il cloud computing, tuttavia, richiede spesso che essere decrittografato in un altro contenuto crittografato in un solo computer.</span><span class="sxs-lookup"><span data-stu-id="6373e-126">Cloud computing, however, often requires that content encrypted on one computer be decrypted on another.</span></span> <span data-ttu-id="6373e-127">Pertanto, a partire da Windows 8, esteso il concetto di utilizza un'API relativamente semplice da contenere scenari basati su cloud di Microsoft.</span><span class="sxs-lookup"><span data-stu-id="6373e-127">Therefore, beginning with Windows 8, Microsoft extended the idea of using a relatively straightforward API to encompass cloud scenarios.</span></span> <span data-ttu-id="6373e-128">Questa nuova API, chiamata DPAPI-NG, consente di condividere in modo sicuro i segreti (chiavi, le password, il materiale della chiave) e i messaggi da di protezione a un set di entità che può essere usato per rimuovere la protezione di essi in computer diversi, dopo la corretta autenticazione e autorizzazione.</span><span class="sxs-lookup"><span data-stu-id="6373e-128">This new API, called DPAPI-NG, enables you to securely share secrets (keys, passwords, key material) and messages by protecting them to a set of principals that can be used to unprotect them on different computers after proper authentication and authorization.</span></span>

   <span data-ttu-id="6373e-129">Da [su DPAPI CNG](https://msdn.microsoft.com/library/windows/desktop/hh706794(v=vs.85).aspx)</span><span class="sxs-lookup"><span data-stu-id="6373e-129">From [About CNG DPAPI](https://msdn.microsoft.com/library/windows/desktop/hh706794(v=vs.85).aspx)</span></span>

<span data-ttu-id="6373e-130">L'entità viene codificato come una regola del descrittore di protezione.</span><span class="sxs-lookup"><span data-stu-id="6373e-130">The principal is encoded as a protection descriptor rule.</span></span> <span data-ttu-id="6373e-131">Si consideri l'esempio seguente che crittografa il materiale della chiave in modo che solo l'utente aggiunto al dominio con il SID specificato è in grado di decrittografare il materiale della chiave.</span><span class="sxs-lookup"><span data-stu-id="6373e-131">Consider the below example, which encrypts key material such that only the domain-joined user with the specified SID can decrypt the key material.</span></span>

```csharp
sc.AddDataProtection()
     // uses the descriptor rule "SID=S-1-5-21-..."
     .ProtectKeysWithDpapiNG("SID=S-1-5-21-...",
       flags: DpapiNGProtectionDescriptorFlags.None);
   ```

<span data-ttu-id="6373e-132">È inoltre disponibile un overload senza parametri di ProtectKeysWithDpapiNG.</span><span class="sxs-lookup"><span data-stu-id="6373e-132">There is also a parameterless overload of ProtectKeysWithDpapiNG.</span></span> <span data-ttu-id="6373e-133">Si tratta di un metodo pratico per specificare la regola "SID = miei", dove il mio è il SID dell'account utente Windows corrente.</span><span class="sxs-lookup"><span data-stu-id="6373e-133">This is a convenience method for specifying the rule "SID=mine", where mine is the SID of the current Windows user account.</span></span>

```csharp
sc.AddDataProtection()
     // uses the descriptor rule "SID={current account SID}"
     .ProtectKeysWithDpapiNG();
   ```

<span data-ttu-id="6373e-134">In questo scenario, il controller di dominio Active Directory è responsabile per la distribuzione di chiavi di crittografia utilizzate dalle operazioni di DPAPI NG.</span><span class="sxs-lookup"><span data-stu-id="6373e-134">In this scenario, the AD domain controller is responsible for distributing the encryption keys used by the DPAPI-NG operations.</span></span> <span data-ttu-id="6373e-135">L'utente di destinazione sarà in grado di decifrare il payload crittografato da qualsiasi computer appartenenti a un dominio (a condizione che il processo viene eseguito con la propria identità).</span><span class="sxs-lookup"><span data-stu-id="6373e-135">The target user will be able to decipher the encrypted payload from any domain-joined machine (provided that the process is running under their identity).</span></span>

## <a name="certificate-based-encryption-with-windows-dpapi-ng"></a><span data-ttu-id="6373e-136">Crittografia basata su certificati con Windows DPAPI-NG.</span><span class="sxs-lookup"><span data-stu-id="6373e-136">Certificate-based encryption with Windows DPAPI-NG</span></span>

<span data-ttu-id="6373e-137">Se si esegue in Windows 8.1 / Windows Server 2012 R2 o versioni successive, è possibile utilizzare Windows DPAPI-NG per eseguire la crittografia basata sui certificati, anche se l'applicazione è in esecuzione [.NET Core](https://www.microsoft.com/net/core).</span><span class="sxs-lookup"><span data-stu-id="6373e-137">If you're running on Windows 8.1 / Windows Server 2012 R2 or later, you can use Windows DPAPI-NG to perform certificate-based encryption, even if the application is running on [.NET Core](https://www.microsoft.com/net/core).</span></span> <span data-ttu-id="6373e-138">Per sfruttare i vantaggi di questo, utilizzare la stringa di descrizione regola "certificato = HashId:thumbprint", dove identificazione personale è l'identificazione digitale SHA1 con codifica esadecimale del certificato da utilizzare.</span><span class="sxs-lookup"><span data-stu-id="6373e-138">To take advantage of this, use the rule descriptor string "CERTIFICATE=HashId:thumbprint", where thumbprint is the hex-encoded SHA1 thumbprint of the certificate to use.</span></span> <span data-ttu-id="6373e-139">Per un esempio, vedere di seguito.</span><span class="sxs-lookup"><span data-stu-id="6373e-139">See below for an example.</span></span>

```csharp
sc.AddDataProtection()
       // searches the cert store for the cert with this thumbprint
       .ProtectKeysWithDpapiNG("CERTIFICATE=HashId:3BCE558E2AD3E0E34A7743EAB5AEA2A9BD2575A0",
           flags: DpapiNGProtectionDescriptorFlags.None);
   ```

<span data-ttu-id="6373e-140">Qualsiasi applicazione che fa riferimento a questo repository deve essere in esecuzione in Windows 8.1 / Windows Server 2012 R2 o versioni successive in grado di decifrare la chiave.</span><span class="sxs-lookup"><span data-stu-id="6373e-140">Any application which is pointed at this repository must be running on Windows 8.1 / Windows Server 2012 R2 or later to be able to decipher this key.</span></span>

## <a name="custom-key-encryption"></a><span data-ttu-id="6373e-141">Chiave di crittografia personalizzato</span><span class="sxs-lookup"><span data-stu-id="6373e-141">Custom key encryption</span></span>

<span data-ttu-id="6373e-142">Se i meccanismi di casella in non sono appropriati, lo sviluppatore può specificare il proprio meccanismo di crittografia della chiave, fornendo un IXmlEncryptor personalizzato.</span><span class="sxs-lookup"><span data-stu-id="6373e-142">If the in-box mechanisms are not appropriate, the developer can specify their own key encryption mechanism by providing a custom IXmlEncryptor.</span></span>