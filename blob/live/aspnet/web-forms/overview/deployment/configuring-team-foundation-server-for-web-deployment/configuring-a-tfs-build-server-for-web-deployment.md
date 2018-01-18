---
uid: web-forms/overview/deployment/configuring-team-foundation-server-for-web-deployment/configuring-a-tfs-build-server-for-web-deployment
title: Configurazione di un TFS Server di compilazione per la distribuzione Web | Documenti Microsoft
author: jrjlee
description: In questo argomento viene descritto come preparare un server di compilazione di Team Foundation Server (TFS) per compilare e distribuire le soluzioni con Team Build e le informazioni di Internet...
ms.author: aspnetcontent
manager: wpickett
ms.date: 05/04/2012
ms.topic: article
ms.assetid: f8400241-4f4b-4bbd-9994-54fb64909e6e
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/deployment/configuring-team-foundation-server-for-web-deployment/configuring-a-tfs-build-server-for-web-deployment
msc.type: authoredcontent
ms.openlocfilehash: 505cca303b5569b2f676adab767d742cb5cd21a7
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 11/10/2017
---
<a name="configuring-a-tfs-build-server-for-web-deployment"></a><span data-ttu-id="8c098-103">Configurazione di un Server di compilazione TFS per la distribuzione Web</span><span class="sxs-lookup"><span data-stu-id="8c098-103">Configuring a TFS Build Server for Web Deployment</span></span>
====================
<span data-ttu-id="8c098-104">da [Jason Lee](https://github.com/jrjlee)</span><span class="sxs-lookup"><span data-stu-id="8c098-104">by [Jason Lee](https://github.com/jrjlee)</span></span>

[<span data-ttu-id="8c098-105">Scarica il PDF</span><span class="sxs-lookup"><span data-stu-id="8c098-105">Download PDF</span></span>](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/63/56/8130.DeployingWebAppsInEnterpriseScenarios.pdf)

> <span data-ttu-id="8c098-106">In questo argomento viene descritto come preparare un server di compilazione di Team Foundation Server (TFS) per compilare e distribuire le soluzioni con Team Build e lo strumento di distribuzione Web di Internet Information Services (IIS) (distribuzione Web).</span><span class="sxs-lookup"><span data-stu-id="8c098-106">This topic describes how to prepare a Team Foundation Server (TFS) build server to build and deploy your solutions using Team Build and the Internet Information Services (IIS) Web Deployment Tool (Web Deploy).</span></span>


<span data-ttu-id="8c098-107">In questo argomento fa parte di una serie di esercitazioni basate su requisiti di distribuzione dell'organizzazione di una società fittizia denominata Fabrikam, Inc. Questa serie di esercitazioni utilizza una soluzione di esempio & #x 2014; il [soluzione responsabile contatto](../web-deployment-in-the-enterprise/the-contact-manager-solution.md)& #x 2014; per rappresentare un'applicazione web con un livello di complessità, tra cui un'applicazione ASP.NET MVC 3, Windows realistico Servizio di Communication Foundation (WCF) e un progetto di database.</span><span class="sxs-lookup"><span data-stu-id="8c098-107">This topic forms part of a series of tutorials based around the enterprise deployment requirements of a fictional company named Fabrikam, Inc. This tutorial series uses a sample solution&#x2014;the [Contact Manager solution](../web-deployment-in-the-enterprise/the-contact-manager-solution.md)&#x2014;to represent a web application with a realistic level of complexity, including an ASP.NET MVC 3 application, a Windows Communication Foundation (WCF) service, and a database project.</span></span>

<span data-ttu-id="8c098-108">Il metodo di distribuzione il fulcro di queste esercitazioni si basa sul progetto file split approccio descritto in [informazioni sui File di progetto](../web-deployment-in-the-enterprise/understanding-the-project-file.md), in cui il processo di compilazione è controllato da due progetti #x 2014; & file contenente una compilare le istruzioni che si applicano a ogni ambiente di destinazione e quella contenente impostazioni specifiche dell'ambiente di compilazione e distribuzione.</span><span class="sxs-lookup"><span data-stu-id="8c098-108">The deployment method at the heart of these tutorials is based on the split project file approach described in [Understanding the Project File](../web-deployment-in-the-enterprise/understanding-the-project-file.md), in which the build process is controlled by two project files&#x2014;one containing build instructions that apply to every destination environment, and one containing environment-specific build and deployment settings.</span></span> <span data-ttu-id="8c098-109">In fase di compilazione, il file di progetto specifici dell'ambiente viene unito nel file di progetto indipendenti dall'ambiente in modo da formare un set completo di istruzioni di compilazione.</span><span class="sxs-lookup"><span data-stu-id="8c098-109">At build time, the environment-specific project file is merged into the environment-agnostic project file to form a complete set of build instructions.</span></span>

## <a name="task-overview"></a><span data-ttu-id="8c098-110">Panoramica di Task</span><span class="sxs-lookup"><span data-stu-id="8c098-110">Task Overview</span></span>

<span data-ttu-id="8c098-111">Per preparare un server di compilazione per compilare e distribuire le soluzioni, è necessario:</span><span class="sxs-lookup"><span data-stu-id="8c098-111">To prepare a build server to build and deploy your solutions, you'll need to:</span></span>

- <span data-ttu-id="8c098-112">Installare e configurare il servizio di compilazione TFS.</span><span class="sxs-lookup"><span data-stu-id="8c098-112">Install and configure the TFS build service.</span></span>
- <span data-ttu-id="8c098-113">Installare Visual Studio 2010.</span><span class="sxs-lookup"><span data-stu-id="8c098-113">Install Visual Studio 2010.</span></span>
- <span data-ttu-id="8c098-114">Installare eventuali prodotti o i componenti richiesti per compilare la soluzione, come le versioni di .NET Framework o di ASP.NET MVC.</span><span class="sxs-lookup"><span data-stu-id="8c098-114">Install any products or components that are required to build your solution, like versions of the .NET Framework or ASP.NET MVC.</span></span>
- <span data-ttu-id="8c098-115">Installare distribuzione Web 2.0 o versione successiva.</span><span class="sxs-lookup"><span data-stu-id="8c098-115">Install Web Deploy 2.0 or later.</span></span>

<span data-ttu-id="8c098-116">Questo argomento viene illustrato come eseguire queste procedure o puntino ad altre risorse, qualora esistano.</span><span class="sxs-lookup"><span data-stu-id="8c098-116">This topic will show you how to perform these procedures or point to other resources where they exist.</span></span> <span data-ttu-id="8c098-117">Le attività e procedure dettagliate in questo argomento si presuppongono che:</span><span class="sxs-lookup"><span data-stu-id="8c098-117">The tasks and walkthroughs in this topic assume that:</span></span>

- <span data-ttu-id="8c098-118">Si inizia con una compilazione pulita server che esegue Windows Server 2008 R2 Service Pack 1.</span><span class="sxs-lookup"><span data-stu-id="8c098-118">You're starting with a clean server build running Windows Server 2008 R2 Service Pack 1.</span></span>
- <span data-ttu-id="8c098-119">Il server è aggiunto al dominio con un indirizzo IP statico.</span><span class="sxs-lookup"><span data-stu-id="8c098-119">The server is domain-joined with a static IP address.</span></span>
- <span data-ttu-id="8c098-120">È installato il livello di applicazione di TFS in un server separato, come descritto in [distribuzione Web aziendale: panoramica dello Scenario](../deploying-web-applications-in-enterprise-scenarios/enterprise-web-deployment-scenario-overview.md).</span><span class="sxs-lookup"><span data-stu-id="8c098-120">You've installed the TFS application tier on a separate server, as described in [Enterprise Web Deployment: Scenario Overview](../deploying-web-applications-in-enterprise-scenarios/enterprise-web-deployment-scenario-overview.md).</span></span>

### <a name="who-performs-these-procedures"></a><span data-ttu-id="8c098-121">Che consente di eseguire queste procedure?</span><span class="sxs-lookup"><span data-stu-id="8c098-121">Who Performs These Procedures?</span></span>

<span data-ttu-id="8c098-122">Nella maggior parte dei casi, un amministratore TFS sarà responsabile per la configurazione del server di compilazione.</span><span class="sxs-lookup"><span data-stu-id="8c098-122">In most cases, a TFS administrator will be responsible for configuring build servers.</span></span> <span data-ttu-id="8c098-123">In alcuni casi, il team di sviluppo può assumere la proprietà del server di compilazione specifica.</span><span class="sxs-lookup"><span data-stu-id="8c098-123">In some cases, the developer team may take ownership of specific build servers.</span></span>

## <a name="install-and-configure-the-tfs-build-service"></a><span data-ttu-id="8c098-124">Installare e configurare il servizio di compilazione TFS</span><span class="sxs-lookup"><span data-stu-id="8c098-124">Install and Configure the TFS Build Service</span></span>

<span data-ttu-id="8c098-125">Quando si configura un server di compilazione, la prima attività è di installare e configurare il servizio di compilazione TFS.</span><span class="sxs-lookup"><span data-stu-id="8c098-125">When you configure a build server, your first task is to install and configure the TFS build service.</span></span> <span data-ttu-id="8c098-126">Come parte di questo processo, è necessario:</span><span class="sxs-lookup"><span data-stu-id="8c098-126">As part of this process, you'll need to:</span></span>

- <span data-ttu-id="8c098-127">Installare il servizio di compilazione TFS e configurare un account del servizio.</span><span class="sxs-lookup"><span data-stu-id="8c098-127">Install the TFS build service and configure a service account.</span></span> <span data-ttu-id="8c098-128">Le attività di compilazione, inclusa la distribuzione, verranno eseguito utilizzando l'identità dell'account del servizio di compilazione.</span><span class="sxs-lookup"><span data-stu-id="8c098-128">Any build tasks, including deployment, will run using the identity of the build service account.</span></span>
- <span data-ttu-id="8c098-129">Creare un *controller di compilazione* e uno o più *gli agenti di compilazione*.</span><span class="sxs-lookup"><span data-stu-id="8c098-129">Create a *build controller* and one or more *build agents*.</span></span> <span data-ttu-id="8c098-130">Ogni controller di compilazione gestisce un set di agenti di compilazione.</span><span class="sxs-lookup"><span data-stu-id="8c098-130">Each build controller manages a set of build agents.</span></span> <span data-ttu-id="8c098-131">Quando si accoda una compilazione, il controller di compilazione assegna le attività di compilazione per un agente di compilazione disponibili.</span><span class="sxs-lookup"><span data-stu-id="8c098-131">When you queue a build, the build controller assigns the build task to an available build agent.</span></span> <span data-ttu-id="8c098-132">Ogni raccolta di progetti team in TFS è mappato a un solo controller di compilazione.</span><span class="sxs-lookup"><span data-stu-id="8c098-132">Each team project collection in TFS is mapped to a single build controller.</span></span>
- <span data-ttu-id="8c098-133">Configurare una cartella di ricezione per l'output di compilazione.</span><span class="sxs-lookup"><span data-stu-id="8c098-133">Configure a drop folder for your build outputs.</span></span> <span data-ttu-id="8c098-134">Si tratta di una condivisione di rete.</span><span class="sxs-lookup"><span data-stu-id="8c098-134">This is a network share.</span></span> <span data-ttu-id="8c098-135">Qualsiasi compilazione output, ad esempio pacchetti di distribuzione web, vengono inviati alla cartella di ricezione.</span><span class="sxs-lookup"><span data-stu-id="8c098-135">Any build outputs, like web deployment packages, are sent to the drop folder.</span></span>

<span data-ttu-id="8c098-136">Il [amministrazione di Team Foundation Build](https://msdn.microsoft.com/en-us/library/ms252495.aspx) capitolo in MSDN contiene tutte le risorse necessarie per eseguire queste attività:</span><span class="sxs-lookup"><span data-stu-id="8c098-136">The [Administering Team Foundation Build](https://msdn.microsoft.com/en-us/library/ms252495.aspx) chapter on MSDN contains all the resources you need in order to perform these tasks:</span></span>

- <span data-ttu-id="8c098-137">Per una panoramica concettuale di Team Foundation Build, tra cui il servizio di compilazione, controller di compilazione e agenti di compilazione, vedere [la comprensione di Team Foundation Build System](https://msdn.microsoft.com/en-us/library/dd793166.aspx).</span><span class="sxs-lookup"><span data-stu-id="8c098-137">For a conceptual overview of Team Foundation Build, including the build service, build controllers, and build agents, see [Understanding a Team Foundation Build System](https://msdn.microsoft.com/en-us/library/dd793166.aspx).</span></span>
- <span data-ttu-id="8c098-138">Per informazioni sull'installazione e configurazione del servizio di compilazione, vedere [configurare un computer di compilazione](https://msdn.microsoft.com/en-us/library/ms181712.aspx).</span><span class="sxs-lookup"><span data-stu-id="8c098-138">For information on installing and configuring the build service, see [Configure a Build Machine](https://msdn.microsoft.com/en-us/library/ms181712.aspx).</span></span>
- <span data-ttu-id="8c098-139">Per informazioni sulla creazione di controller di compilazione, vedere [creare e usare un Controller di compilazione](https://msdn.microsoft.com/en-us/library/ee330987.aspx).</span><span class="sxs-lookup"><span data-stu-id="8c098-139">For information on creating build controllers, see [Create and Work with a Build Controller](https://msdn.microsoft.com/en-us/library/ee330987.aspx).</span></span>
- <span data-ttu-id="8c098-140">Per informazioni sulla creazione di agenti di compilazione, vedere [creare e usare agenti di compilazione](https://msdn.microsoft.com/en-us/library/bb399135.aspx).</span><span class="sxs-lookup"><span data-stu-id="8c098-140">For information on creating build agents, see [Create and Work with Build Agents](https://msdn.microsoft.com/en-us/library/bb399135.aspx).</span></span>
- <span data-ttu-id="8c098-141">Per informazioni sulla creazione e configurazione di cartelle di ricezione, vedere [configurare cartelle di ricezione](https://msdn.microsoft.com/en-us/library/bb778394.aspx).</span><span class="sxs-lookup"><span data-stu-id="8c098-141">For information on creating and configuring drop folders, see [Set Up Drop Folders](https://msdn.microsoft.com/en-us/library/bb778394.aspx).</span></span>

## <a name="install-required-products-and-components"></a><span data-ttu-id="8c098-142">Installare i componenti e i prodotti necessari</span><span class="sxs-lookup"><span data-stu-id="8c098-142">Install Required Products and Components</span></span>

<span data-ttu-id="8c098-143">Per abilitare il server di compilazione compilare le soluzioni, è necessario installare tutti i prodotti, componenti o gli assembly che la soluzione richiede.</span><span class="sxs-lookup"><span data-stu-id="8c098-143">To enable the build server to build your solutions, you must install any products, components, or assemblies that your solution requires.</span></span> <span data-ttu-id="8c098-144">Prima di installare i componenti della piattaforma web, è necessario installare Visual Studio 2010 (qualsiasi versione) nel server di compilazione.</span><span class="sxs-lookup"><span data-stu-id="8c098-144">Before you install any web platform components, you should install Visual Studio 2010 (any version) on the build server.</span></span> <span data-ttu-id="8c098-145">In questo modo si garantisce che i file di destinazione di base Microsoft Build Engine (MSBuild) e i file di destinazione Pipeline pubblicazione sul Web (WPP) sono disponibili per il servizio di compilazione.</span><span class="sxs-lookup"><span data-stu-id="8c098-145">This ensures that the core Microsoft Build Engine (MSBuild) target files and the Web Publishing Pipeline (WPP) target files are available to the build service.</span></span> <span data-ttu-id="8c098-146">Il programma di installazione di Visual Studio è necessario installare anche distribuzione Web, è necessario se si prevede di distribuire i pacchetti web come parte del processo di compilazione.</span><span class="sxs-lookup"><span data-stu-id="8c098-146">The Visual Studio installer should also install Web Deploy, which you'll need if you plan to deploy web packages as part of your build process.</span></span>

<span data-ttu-id="8c098-147">Il modo migliore per installare i componenti della piattaforma web comune è utilizzare il [installazione guidata piattaforma Web](https://go.microsoft.com/?linkid=9805118).</span><span class="sxs-lookup"><span data-stu-id="8c098-147">The best way to install common web platform components is to use the [Web Platform Installer](https://go.microsoft.com/?linkid=9805118).</span></span> <span data-ttu-id="8c098-148">Ciò garantisce che si sta installando la versione più recente di ogni prodotto e inoltre rileva e installa automaticamente tutti i prerequisiti per ogni prodotto.</span><span class="sxs-lookup"><span data-stu-id="8c098-148">This ensures that you're installing the latest version of each product, and it also automatically detects and installs any prerequisites for each product.</span></span> <span data-ttu-id="8c098-149">In caso del [Contact Manager](../web-deployment-in-the-enterprise/the-contact-manager-solution.md) soluzione, utilizzare l'installazione guidata piattaforma Web per installare questi prodotti e componenti:</span><span class="sxs-lookup"><span data-stu-id="8c098-149">In the case of the [Contact Manager](../web-deployment-in-the-enterprise/the-contact-manager-solution.md) solution, you should use the Web Platform Installer to install these products and components:</span></span>

- <span data-ttu-id="8c098-150">**.NET framework 4.0**.</span><span class="sxs-lookup"><span data-stu-id="8c098-150">**.NET Framework 4.0**.</span></span> <span data-ttu-id="8c098-151">Ciò è necessario per eseguire applicazioni che sono state compilate in questa versione di .NET Framework.</span><span class="sxs-lookup"><span data-stu-id="8c098-151">This is required to run applications that were built on this version of the .NET Framework.</span></span>
- <span data-ttu-id="8c098-152">**2.1 o successivo dello strumento di distribuzione Web**.</span><span class="sxs-lookup"><span data-stu-id="8c098-152">**Web Deployment Tool 2.1 or later**.</span></span> <span data-ttu-id="8c098-153">Questo modo vengono installati sul server di distribuzione Web (e il relativo file eseguibile sottostante, MSDeploy.exe).</span><span class="sxs-lookup"><span data-stu-id="8c098-153">This installs Web Deploy (and its underlying executable, MSDeploy.exe) on your server.</span></span> <span data-ttu-id="8c098-154">Come parte di questo processo, installa e avvia il servizio agente di distribuzione Web.</span><span class="sxs-lookup"><span data-stu-id="8c098-154">As part of this process, it installs and starts the Web Deployment Agent Service.</span></span> <span data-ttu-id="8c098-155">Questo servizio consente di distribuire i pacchetti web da un computer remoto.</span><span class="sxs-lookup"><span data-stu-id="8c098-155">This service lets you deploy web packages from a remote computer.</span></span>
- <span data-ttu-id="8c098-156">**ASP.NET MVC 3**.</span><span class="sxs-lookup"><span data-stu-id="8c098-156">**ASP.NET MVC 3**.</span></span> <span data-ttu-id="8c098-157">Consente di installare gli assembly che necessari per eseguire applicazioni ASP.NET MVC 3.</span><span class="sxs-lookup"><span data-stu-id="8c098-157">This installs the assemblies you need to run ASP.NET MVC 3 applications.</span></span>

<span data-ttu-id="8c098-158">**Per installare i componenti e prodotti necessari**</span><span class="sxs-lookup"><span data-stu-id="8c098-158">**To install the required products and components**</span></span>

1. <span data-ttu-id="8c098-159">Installare Visual Studio 2010.</span><span class="sxs-lookup"><span data-stu-id="8c098-159">Install Visual Studio 2010.</span></span> <span data-ttu-id="8c098-160">Quando viene richiesto di selezionare le funzionalità da installare, è necessario includere:</span><span class="sxs-lookup"><span data-stu-id="8c098-160">When prompted to select features to install, you should include:</span></span>

    1. <span data-ttu-id="8c098-161">Qualsiasi linguaggi di programmazione che si devono compilare.</span><span class="sxs-lookup"><span data-stu-id="8c098-161">Any programming languages that you need to compile.</span></span>
    2. <span data-ttu-id="8c098-162">Visual Web Developer.</span><span class="sxs-lookup"><span data-stu-id="8c098-162">Visual Web Developer.</span></span> <span data-ttu-id="8c098-163">In questo modo si garantisce che le destinazioni del sito vengono aggiunti al server di compilazione.</span><span class="sxs-lookup"><span data-stu-id="8c098-163">This ensures that the WPP targets are added to your build server.</span></span>

        ![](configuring-a-tfs-build-server-for-web-deployment/_static/image1.png)
2. <span data-ttu-id="8c098-164">Una volta completata l'installazione di Visual Studio 2010, scaricare e installare [Visual Studio 2010 Service Pack 1](https://go.microsoft.com/?linkid=9805133) (se è non è già incluso nel supporto di installazione).</span><span class="sxs-lookup"><span data-stu-id="8c098-164">When the installation of Visual Studio 2010 is complete, download and install [Visual Studio 2010 Service Pack 1](https://go.microsoft.com/?linkid=9805133) (if it's not already included in your installation media).</span></span>

    > [!NOTE]
    > <span data-ttu-id="8c098-165">Visual Studio 2010 Service Pack 1 risolve un bug che può impedire l'individuazione di MSDeploy eseguibile di MSBuild.</span><span class="sxs-lookup"><span data-stu-id="8c098-165">Visual Studio 2010 Service Pack 1 resolves a bug that can prevent MSBuild from locating the MSDeploy executable.</span></span>
3. <span data-ttu-id="8c098-166">Scaricare e avviare il [installazione guidata piattaforma Web](https://go.microsoft.com/?linkid=9805118).</span><span class="sxs-lookup"><span data-stu-id="8c098-166">Download and launch the [Web Platform Installer](https://go.microsoft.com/?linkid=9805118).</span></span>
4. <span data-ttu-id="8c098-167">Nella parte superiore del **installazione guidata piattaforma Web 3.0** finestra, fare clic su **prodotti**.</span><span class="sxs-lookup"><span data-stu-id="8c098-167">At the top of the **Web Platform Installer 3.0** window, click **Products**.</span></span>
5. <span data-ttu-id="8c098-168">Sul lato sinistro della finestra, nel riquadro di spostamento, fare clic su **Framework**.</span><span class="sxs-lookup"><span data-stu-id="8c098-168">On the left side of the window, in the navigation pane, click **Frameworks**.</span></span>
6. <span data-ttu-id="8c098-169">Nel **Microsoft .NET Framework 4** riga se .NET Framework non è già installato, fare clic su **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="8c098-169">In the **Microsoft .NET Framework 4** row, if the .NET Framework is not already installed, click **Add**.</span></span>

    > [!NOTE]
    > <span data-ttu-id="8c098-170">Si è già installato .NET Framework 4.0 in Windows Update.</span><span class="sxs-lookup"><span data-stu-id="8c098-170">You may have already installed the .NET Framework 4.0 through Windows Update.</span></span> <span data-ttu-id="8c098-171">Se è già installato un prodotto o componente, l'installazione guidata piattaforma Web indicherà questo sostituendo il **Aggiungi** pulsante con il testo **installato**.</span><span class="sxs-lookup"><span data-stu-id="8c098-171">If a product or component is already installed, the Web Platform Installer will indicate this by replacing the **Add** button with the text **Installed**.</span></span>

    ![](configuring-a-tfs-build-server-for-web-deployment/_static/image2.png)
7. <span data-ttu-id="8c098-172">Nel **ASP.NET MVC 3 (Visual Studio 2010)** di riga, fare clic su **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="8c098-172">In the **ASP.NET MVC 3 (Visual Studio 2010)** row, click **Add**.</span></span>
8. <span data-ttu-id="8c098-173">Nel riquadro di spostamento, fare clic su **Server**.</span><span class="sxs-lookup"><span data-stu-id="8c098-173">In the navigation pane, click **Server**.</span></span>
9. <span data-ttu-id="8c098-174">Nel **2.1 dello strumento di distribuzione Web** di riga, fare clic su **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="8c098-174">In the **Web Deployment Tool 2.1** row, click **Add**.</span></span>
10. <span data-ttu-id="8c098-175">Fare clic su **Installa**.</span><span class="sxs-lookup"><span data-stu-id="8c098-175">Click **Install**.</span></span> <span data-ttu-id="8c098-176">L'installazione guidata piattaforma Web verrà visualizzato un elenco di prodotti & #x 2014; insieme a tutte le dipendenze associate & #x 2014; sia installato e verrà richiesto di accettare le condizioni di licenza.</span><span class="sxs-lookup"><span data-stu-id="8c098-176">The Web Platform Installer will show you a list of products&#x2014;together with any associated dependencies&#x2014;to be installed and will prompt you to accept the license terms.</span></span>
11. <span data-ttu-id="8c098-177">Esaminare le condizioni di licenza e, se accettano le condizioni, fare clic su **accetto**.</span><span class="sxs-lookup"><span data-stu-id="8c098-177">Review the license terms, and if you consent to the terms, click **I Accept**.</span></span>
12. <span data-ttu-id="8c098-178">Una volta completato l'installazione, fare clic su **fine**, quindi chiudere il **installazione guidata piattaforma Web 3.0** finestra.</span><span class="sxs-lookup"><span data-stu-id="8c098-178">When the installation is complete, click **Finish**, and then close the **Web Platform Installer 3.0** window.</span></span>

> [!NOTE]
> <span data-ttu-id="8c098-179">Se il processo di distribuzione include l'utilizzo di strumenti come VSDBCMD.exe o SQLCMD.exe, è necessario assicurarsi che questi siano installati nel server di compilazione.</span><span class="sxs-lookup"><span data-stu-id="8c098-179">If your deployment process includes the use of tools like VSDBCMD.exe or SQLCMD.exe, you'll need to ensure that these are installed on your build server.</span></span> <span data-ttu-id="8c098-180">VSDBCMD.exe è uno strumento di Visual Studio e viene in genere aggiunto al server durante l'installazione di Team Foundation Build.</span><span class="sxs-lookup"><span data-stu-id="8c098-180">VSDBCMD.exe is a Visual Studio tool and is typically added to the server when you install Team Foundation Build.</span></span> <span data-ttu-id="8c098-181">SQLCMD.exe è uno strumento di SQL Server.</span><span class="sxs-lookup"><span data-stu-id="8c098-181">SQLCMD.exe is a SQL Server tool.</span></span> <span data-ttu-id="8c098-182">È possibile scaricare una versione autonoma di SQLCMD.exe dal [Microsoft SQL Server 2008 R2 Feature Pack](https://go.microsoft.com/?linkid=9805134) pagina.</span><span class="sxs-lookup"><span data-stu-id="8c098-182">You can download a stand-alone version of SQLCMD.exe from the [Microsoft SQL Server 2008 R2 Feature Pack](https://go.microsoft.com/?linkid=9805134) page.</span></span>


## <a name="conclusion"></a><span data-ttu-id="8c098-183">Conclusione</span><span class="sxs-lookup"><span data-stu-id="8c098-183">Conclusion</span></span>

<span data-ttu-id="8c098-184">A questo punto, il server di compilazione è possibile avviare la creazione e la distribuzione di progetti di applicazione web.</span><span class="sxs-lookup"><span data-stu-id="8c098-184">At this point, your build server is ready to start building and deploying your web application projects.</span></span> <span data-ttu-id="8c098-185">L'argomento successivo, [la creazione di una compilazione che supporta la distribuzione della definizione](creating-a-build-definition-that-supports-deployment.md), viene descritto come creare e configurare una definizione di compilazione per controllare quando e come i progetti sono compilati e distribuiti.</span><span class="sxs-lookup"><span data-stu-id="8c098-185">The next topic, [Creating a Build Definition That Supports Deployment](creating-a-build-definition-that-supports-deployment.md), describes how to create and configure a build definition to control when and how your projects are built and deployed.</span></span>

## <a name="further-reading"></a><span data-ttu-id="8c098-186">Ulteriori informazioni</span><span class="sxs-lookup"><span data-stu-id="8c098-186">Further Reading</span></span>

<span data-ttu-id="8c098-187">Per istruzioni generali sull'uso di Team Build, vedere [amministrazione di Team Foundation Build](https://msdn.microsoft.com/en-us/library/ms252495.aspx).</span><span class="sxs-lookup"><span data-stu-id="8c098-187">For more general guidance on working with Team Build, see [Administering Team Foundation Build](https://msdn.microsoft.com/en-us/library/ms252495.aspx).</span></span>

>[!div class="step-by-step"]
<span data-ttu-id="8c098-188">[Precedente](adding-content-to-source-control.md)
[Successivo](creating-a-build-definition-that-supports-deployment.md)</span><span class="sxs-lookup"><span data-stu-id="8c098-188">[Previous](adding-content-to-source-control.md)
[Next](creating-a-build-definition-that-supports-deployment.md)</span></span>