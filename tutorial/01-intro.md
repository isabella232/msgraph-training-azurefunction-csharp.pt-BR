---
ms.openlocfilehash: 1e4267a5d8bdfe02dbc0170122113c35df822659
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822540"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="ae460-101">Este tutorial ensina como criar uma função do Azure que usa a API do Microsoft Graph para recuperar informações de calendário de um usuário.</span><span class="sxs-lookup"><span data-stu-id="ae460-101">This tutorial teaches you how to build an Azure Function that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="ae460-102">Se preferir baixar o tutorial concluído, você poderá baixar ou clonar o repositório do [GitHub](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span><span class="sxs-lookup"><span data-stu-id="ae460-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span> <span data-ttu-id="ae460-103">Consulte o arquivo LEIAme na pasta **demonstração** para obter instruções sobre como configurar o aplicativo com uma ID de aplicativo e segredo.</span><span class="sxs-lookup"><span data-stu-id="ae460-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="ae460-104">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="ae460-104">Prerequisites</span></span>

<span data-ttu-id="ae460-105">Antes de iniciar este tutorial, você deve ter as seguintes ferramentas instaladas em sua máquina de desenvolvimento.</span><span class="sxs-lookup"><span data-stu-id="ae460-105">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="ae460-106">SDK do .NET Core</span><span class="sxs-lookup"><span data-stu-id="ae460-106">.NET Core SDK</span></span>](https://dotnet.microsoft.com/download)
- [<span data-ttu-id="ae460-107">Ferramentas principais de funções do Azure</span><span class="sxs-lookup"><span data-stu-id="ae460-107">Azure Functions Core Tools</span></span>](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [<span data-ttu-id="ae460-108">CLI do Azure</span><span class="sxs-lookup"><span data-stu-id="ae460-108">Azure CLI</span></span>](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [<span data-ttu-id="ae460-109">ngrok</span><span class="sxs-lookup"><span data-stu-id="ae460-109">ngrok</span></span>](https://ngrok.com/)

<span data-ttu-id="ae460-110">Você também deve ter uma conta corporativa ou de estudante da Microsoft, com acesso a uma conta de administrador global na mesma organização.</span><span class="sxs-lookup"><span data-stu-id="ae460-110">You should also have a Microsoft work or school account, with access to a global administrator account in the same organization.</span></span> <span data-ttu-id="ae460-111">Se você não tiver uma conta da Microsoft, poderá [se inscrever no programa para desenvolvedores do office 365](https://developer.microsoft.com/office/dev-program) para obter uma assinatura gratuita do Office 365.</span><span class="sxs-lookup"><span data-stu-id="ae460-111">If you don't have a Microsoft account, you can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="ae460-112">Este tutorial foi escrito com as seguintes versões das ferramentas acima.</span><span class="sxs-lookup"><span data-stu-id="ae460-112">This tutorial was written with the following versions of the above tools.</span></span> <span data-ttu-id="ae460-113">As etapas deste guia podem funcionar com outras versões, mas que não foram testadas.</span><span class="sxs-lookup"><span data-stu-id="ae460-113">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="ae460-114">SDK do .NET Core 3.1.301</span><span class="sxs-lookup"><span data-stu-id="ae460-114">.NET Core SDK 3.1.301</span></span>
> - <span data-ttu-id="ae460-115">Ferramentas do Azure funções principais 3.0.2630</span><span class="sxs-lookup"><span data-stu-id="ae460-115">Azure Functions Core Tools 3.0.2630</span></span>
> - <span data-ttu-id="ae460-116">2.8.0 da CLI do Azure</span><span class="sxs-lookup"><span data-stu-id="ae460-116">Azure CLI 2.8.0</span></span>
> - <span data-ttu-id="ae460-117">ngrok 2.3.35</span><span class="sxs-lookup"><span data-stu-id="ae460-117">ngrok 2.3.35</span></span>

## <a name="feedback"></a><span data-ttu-id="ae460-118">Comentários</span><span class="sxs-lookup"><span data-stu-id="ae460-118">Feedback</span></span>

<span data-ttu-id="ae460-119">Forneça comentários sobre este tutorial no [repositório do GitHub](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span><span class="sxs-lookup"><span data-stu-id="ae460-119">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span>
