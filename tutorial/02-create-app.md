---
ms.openlocfilehash: 003d7f382a0c950d31bb84176feb1e83fbd5cf19
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822543"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="bef48-101">Neste tutorial, você criará uma função simples do Azure que implementa funções de gatilho HTTP que chamam o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="bef48-101">In this tutorial, you will create a simple Azure Function that implements HTTP trigger functions that call Microsoft Graph.</span></span> <span data-ttu-id="bef48-102">Essas funções abrangerão os seguintes cenários:</span><span class="sxs-lookup"><span data-stu-id="bef48-102">These functions will cover the following scenarios:</span></span>

- <span data-ttu-id="bef48-103">Implementa uma API para acessar a caixa de entrada de um usuário usando a autenticação de [fluxo em nome de](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) .</span><span class="sxs-lookup"><span data-stu-id="bef48-103">Implements an API to access a user's inbox using [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) authentication.</span></span>
- <span data-ttu-id="bef48-104">Implementa uma API para assinar e cancelar a assinatura de notificações na caixa de entrada de um usuário, usando o uso de [credenciais de cliente conceder](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) autenticação de fluxo.</span><span class="sxs-lookup"><span data-stu-id="bef48-104">Implements an API to subscribe and unsubscribe for notifications on a user's inbox, using using [client credentials grant flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) authentication.</span></span>
- <span data-ttu-id="bef48-105">Implementa um webhook para receber [notificações de alteração](https://docs.microsoft.com/graph/webhooks) do Microsoft Graph e acessar dados usando o fluxo de concessão de credenciais do cliente.</span><span class="sxs-lookup"><span data-stu-id="bef48-105">Implements a webhook to receive [change notifications](https://docs.microsoft.com/graph/webhooks) from Microsoft Graph and access data using client credentials grant flow.</span></span>

<span data-ttu-id="bef48-106">Você também criará um aplicativo simples de página única (SPA) JavaScript para chamar as APIs implementadas na função do Azure.</span><span class="sxs-lookup"><span data-stu-id="bef48-106">You will also create a simple JavaScript single-page application (SPA) to call the APIs implemented in the Azure Function.</span></span>

## <a name="create-azure-functions-project"></a><span data-ttu-id="bef48-107">Criar projeto de funções do Azure</span><span class="sxs-lookup"><span data-stu-id="bef48-107">Create Azure Functions project</span></span>

1. <span data-ttu-id="bef48-108">Abra a interface de linha de comando (CLI) em um diretório onde você deseja criar o projeto.</span><span class="sxs-lookup"><span data-stu-id="bef48-108">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="bef48-109">Execute o seguinte comando.</span><span class="sxs-lookup"><span data-stu-id="bef48-109">Run the following command.</span></span>

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. <span data-ttu-id="bef48-110">Altere o diretório atual em sua CLI para o diretório **GraphTutorial** e execute os seguintes comandos para criar três funções no projeto.</span><span class="sxs-lookup"><span data-stu-id="bef48-110">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands to create three functions in the project.</span></span>

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. <span data-ttu-id="bef48-111">Abra **local.settings.jsem** e adicione o seguinte ao arquivo para permitir CORS de `http://localhost:8080` , a URL para o aplicativo de teste.</span><span class="sxs-lookup"><span data-stu-id="bef48-111">Open **local.settings.json** and add the following to the file to allow CORS from `http://localhost:8080`, the URL for the test application.</span></span>

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. <span data-ttu-id="bef48-112">Execute o comando a seguir para executar o projeto localmente.</span><span class="sxs-lookup"><span data-stu-id="bef48-112">Run the following command to run the project locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="bef48-113">Se tudo estiver funcionando, você verá o seguinte resultado:</span><span class="sxs-lookup"><span data-stu-id="bef48-113">If everything is working, you will see the following output:</span></span>

    ```Shell
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. <span data-ttu-id="bef48-114">Verifique se as funções estão funcionando corretamente abrindo o navegador e navegando até as URLs de função mostradas na saída.</span><span class="sxs-lookup"><span data-stu-id="bef48-114">Verify that the functions are working correctly by opening your browser and browsing to the function URLs shown in the output.</span></span> <span data-ttu-id="bef48-115">Você deverá ver a seguinte mensagem em seu navegador: `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` .</span><span class="sxs-lookup"><span data-stu-id="bef48-115">You should see the following message in your browser: `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.`.</span></span>

## <a name="create-single-page-application"></a><span data-ttu-id="bef48-116">Criar um aplicativo de página única</span><span class="sxs-lookup"><span data-stu-id="bef48-116">Create single-page application</span></span>

1. <span data-ttu-id="bef48-117">Abra a CLI em um diretório onde você deseja criar o projeto.</span><span class="sxs-lookup"><span data-stu-id="bef48-117">Open your CLI in a directory where you want to create the project.</span></span> <span data-ttu-id="bef48-118">Crie um diretório chamado **TestClient** para manter seus arquivos HTML e JavaScript.</span><span class="sxs-lookup"><span data-stu-id="bef48-118">Create a directory named **TestClient** to hold your HTML and JavaScript files.</span></span>

1. <span data-ttu-id="bef48-119">Crie um novo arquivo chamado **index.html** no diretório **TestClient** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="bef48-119">Create a new file named **index.html** in the **TestClient** directory and add the following code.</span></span>

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    <span data-ttu-id="bef48-120">Isso define o layout básico do aplicativo, incluindo uma barra de navegação.</span><span class="sxs-lookup"><span data-stu-id="bef48-120">This defines the basic layout of the app, including a navigation bar.</span></span> <span data-ttu-id="bef48-121">Ele também adiciona o seguinte:</span><span class="sxs-lookup"><span data-stu-id="bef48-121">It also adds the following:</span></span>

    - <span data-ttu-id="bef48-122">[Bootstrap](https://getbootstrap.com/) e seu JavaScript de suporte</span><span class="sxs-lookup"><span data-stu-id="bef48-122">[Bootstrap](https://getbootstrap.com/) and its supporting JavaScript</span></span>
    - [<span data-ttu-id="bef48-123">FontAwesome</span><span class="sxs-lookup"><span data-stu-id="bef48-123">FontAwesome</span></span>](https://fontawesome.com/)
    - [<span data-ttu-id="bef48-124">Biblioteca de autenticação da Microsoft para JavaScript (MSAL.js) 2,0</span><span class="sxs-lookup"><span data-stu-id="bef48-124">Microsoft Authentication Library for JavaScript (MSAL.js) 2.0</span></span>](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > <span data-ttu-id="bef48-125">A página inclui um favicon ( `<link rel="shortcut icon" href="g-raph.png">` ).</span><span class="sxs-lookup"><span data-stu-id="bef48-125">The page includes a favicon, (`<link rel="shortcut icon" href="g-raph.png">`).</span></span> <span data-ttu-id="bef48-126">Você pode remover essa linha ou pode baixar o arquivo de **g-raph.png** do [GitHub](https://github.com/microsoftgraph/g-raph).</span><span class="sxs-lookup"><span data-stu-id="bef48-126">You can remove this line, or you can download the **g-raph.png** file from [GitHub](https://github.com/microsoftgraph/g-raph).</span></span>

1. <span data-ttu-id="bef48-127">Crie um novo arquivo chamado **Style. css** no diretório **TestClient** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="bef48-127">Create a new file named **style.css** in the **TestClient** directory and add the following code.</span></span>

    :::code language="css" source="../demo/TestClient/style.css":::

1. <span data-ttu-id="bef48-128">Crie um novo arquivo chamado **ui.js** no diretório **TestClient** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="bef48-128">Create a new file named **ui.js** in the **TestClient** directory and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    <span data-ttu-id="bef48-129">Este código usa JavaScript para renderizar a página atual com base no modo de exibição selecionado.</span><span class="sxs-lookup"><span data-stu-id="bef48-129">This code uses JavaScript to render the current page based on the selected view.</span></span>

### <a name="test-the-single-page-application"></a><span data-ttu-id="bef48-130">Testar o aplicativo de página única</span><span class="sxs-lookup"><span data-stu-id="bef48-130">Test the single-page application</span></span>

> [!NOTE]
> <span data-ttu-id="bef48-131">Esta seção inclui instruções para usar o [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) para executar um servidor http de teste simples em sua máquina de desenvolvimento.</span><span class="sxs-lookup"><span data-stu-id="bef48-131">This section includes instructions for using [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) to run a simple testing HTTP server on your development machine.</span></span> <span data-ttu-id="bef48-132">Não é necessário usar essa ferramenta específica.</span><span class="sxs-lookup"><span data-stu-id="bef48-132">Using this specific tool is not required.</span></span> <span data-ttu-id="bef48-133">Você pode usar qualquer servidor de teste que preferir para servir o diretório **TestClient**</span><span class="sxs-lookup"><span data-stu-id="bef48-133">You can use any testing server you prefer to serve the **TestClient** directory.</span></span>

1. <span data-ttu-id="bef48-134">Execute o seguinte comando em sua CLI para instalar o **dotnet-atendimento**.</span><span class="sxs-lookup"><span data-stu-id="bef48-134">Run the following command in your CLI to install **dotnet-serve**.</span></span>

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. <span data-ttu-id="bef48-135">Altere o diretório atual em sua CLI para o diretório **TestClient** e execute o seguinte comando para iniciar um servidor http.</span><span class="sxs-lookup"><span data-stu-id="bef48-135">Change the current directory in your CLI to the **TestClient** directory and run the following command to start an HTTP server.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. <span data-ttu-id="bef48-136">Abra o navegador e vá até `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="bef48-136">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="bef48-137">A página deve renderizar, mas nenhum dos botões funciona no momento.</span><span class="sxs-lookup"><span data-stu-id="bef48-137">The page should render, but none of the buttons currently work.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="bef48-138">Adicionar pacotes NuGet</span><span class="sxs-lookup"><span data-stu-id="bef48-138">Add NuGet packages</span></span>

<span data-ttu-id="bef48-139">Antes de prosseguir, instale alguns pacotes NuGet adicionais que serão usados posteriormente.</span><span class="sxs-lookup"><span data-stu-id="bef48-139">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="bef48-140">[Microsoft. Azure. Functions. Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) para habilitar a injeção de dependência no projeto de funções do Azure.</span><span class="sxs-lookup"><span data-stu-id="bef48-140">[Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) to enable dependency injection in the Azure Functions project.</span></span>
- <span data-ttu-id="bef48-141">[Microsoft.Extensions.Configuration. Usersecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) para ler a configuração do aplicativo do [repositório de segredo de desenvolvimento do .net](https://docs.microsoft.com/aspnet/core/security/app-secrets).</span><span class="sxs-lookup"><span data-stu-id="bef48-141">[Microsoft.Extensions.Configuration.UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) to read application configuration from the [.NET development secret store](https://docs.microsoft.com/aspnet/core/security/app-secrets).</span></span>
- <span data-ttu-id="bef48-142">[Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) para fazer chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="bef48-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="bef48-143">[Microsoft. Identity. Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) para autenticação e gerenciamento de tokens.</span><span class="sxs-lookup"><span data-stu-id="bef48-143">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) for authenticating and managing tokens.</span></span>
- <span data-ttu-id="bef48-144">[Microsoft. IdentityModel. Protocols. OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) para recuperar a configuração de OpenID para validação de token.</span><span class="sxs-lookup"><span data-stu-id="bef48-144">[Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) for retrieving OpenID configuration for token validation.</span></span>
- <span data-ttu-id="bef48-145">[System. IdentityModel. Tokens. JWT](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) para validar tokens enviados para a API Web.</span><span class="sxs-lookup"><span data-stu-id="bef48-145">[System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) for validating tokens sent to the web API.</span></span>

1. <span data-ttu-id="bef48-146">Altere o diretório atual em sua CLI para o diretório **GraphTutorial** e execute os seguintes comandos.</span><span class="sxs-lookup"><span data-stu-id="bef48-146">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands.</span></span>

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```
