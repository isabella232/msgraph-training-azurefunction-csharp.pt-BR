---
ms.openlocfilehash: fbd43b4708cd1aeaad93b70fcf151869bd9e543e
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822547"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="8568c-101">Neste exercício, você aprenderá sobre as alterações necessárias para a função do Azure de amostra preparar a [publicação para um aplicativo Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).</span><span class="sxs-lookup"><span data-stu-id="8568c-101">In this exercise you'll learn about what changes are needed to the sample Azure Function to prepare for [publishing to an Azure Functions app](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).</span></span>

## <a name="update-code"></a><span data-ttu-id="8568c-102">Código de atualização</span><span class="sxs-lookup"><span data-stu-id="8568c-102">Update code</span></span>

<span data-ttu-id="8568c-103">A configuração é lida do repositório de segredo do usuário, que só se aplica à sua máquina de desenvolvimento.</span><span class="sxs-lookup"><span data-stu-id="8568c-103">Configuration is read from the user secret store, which only applies to your development machine.</span></span> <span data-ttu-id="8568c-104">Antes de publicar no Azure, você precisará alterar onde armazenou sua configuração e atualizar o código no **Startup.cs** de acordo.</span><span class="sxs-lookup"><span data-stu-id="8568c-104">Before you publish to Azure, you'll need to change where you store your configuration, and update the code in **Startup.cs** accordingly.</span></span>

<span data-ttu-id="8568c-105">Os segredos do aplicativo devem ser armazenados no armazenamento seguro, como o [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).</span><span class="sxs-lookup"><span data-stu-id="8568c-105">Application secrets should be stored in secure storage, such as [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).</span></span>

## <a name="update-cors-setting-for-azure-function"></a><span data-ttu-id="8568c-106">Atualizar a configuração CORS para a função do Azure</span><span class="sxs-lookup"><span data-stu-id="8568c-106">Update CORS setting for Azure Function</span></span>

<span data-ttu-id="8568c-107">Neste exemplo, configuramos o CORS no **local.settings.jsativado** para permitir que o aplicativo de teste chame a função.</span><span class="sxs-lookup"><span data-stu-id="8568c-107">In this sample we configured CORS in **local.settings.json** to allow the test application to call the function.</span></span> <span data-ttu-id="8568c-108">Você precisará configurar sua função publicada para permitir qualquer aplicativo de SPA que o chamará.</span><span class="sxs-lookup"><span data-stu-id="8568c-108">You'll need to configure your published function to allow any SPA apps that will call it.</span></span>

## <a name="update-app-registrations"></a><span data-ttu-id="8568c-109">Atualizar registros de aplicativo</span><span class="sxs-lookup"><span data-stu-id="8568c-109">Update app registrations</span></span>

<span data-ttu-id="8568c-110">A  `knownClientApplications` propriedade no manifesto do registro de aplicativo de **função do Azure para Graph** precisará ser atualizada com as identificações de aplicativos de qualquer aplicativo que chamará a função do Azure.</span><span class="sxs-lookup"><span data-stu-id="8568c-110">The  `knownClientApplications` property in the manifest for the **Graph Azure Function** app registration will need to be updated with the application IDs of any apps that will be calling the Azure Function.</span></span>

## <a name="recreate-existing-subscriptions"></a><span data-ttu-id="8568c-111">Recriar assinaturas existentes</span><span class="sxs-lookup"><span data-stu-id="8568c-111">Recreate existing subscriptions</span></span>

<span data-ttu-id="8568c-112">Todas as assinaturas criadas usando a URL de webhook na sua máquina local ou ngrok devem ser recriadas usando a URL de produção da `Notify` função do Azure.</span><span class="sxs-lookup"><span data-stu-id="8568c-112">Any subscriptions created using the webhook URL on your local machine or ngrok should be recreated using the production URL of the `Notify` Azure Function.</span></span>
