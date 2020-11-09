---
ms.openlocfilehash: 728138b5d298f9469a566defed6ee2e457896c0b
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822555"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f3d29-101">Neste exercício, você concluirá a implementação da função do Azure `GetMyNewestMessage` e atualizará o cliente de teste para chamar a função.</span><span class="sxs-lookup"><span data-stu-id="f3d29-101">In this exercise you will finish implementing the Azure Function `GetMyNewestMessage` and update the test client to call the function.</span></span>

<span data-ttu-id="f3d29-102">A função do Azure usa o [fluxo em nome de](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow).</span><span class="sxs-lookup"><span data-stu-id="f3d29-102">The Azure Function uses the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow).</span></span> <span data-ttu-id="f3d29-103">A ordem básica de eventos nesse fluxo são:</span><span class="sxs-lookup"><span data-stu-id="f3d29-103">The basic order of events in this flow are:</span></span>

- <span data-ttu-id="f3d29-104">O aplicativo de teste usa um fluxo de autenticação interativa para permitir que o usuário entre e conceda o consentimento.</span><span class="sxs-lookup"><span data-stu-id="f3d29-104">The test application uses an interactive auth flow to allow the user to sign in and grant consent.</span></span> <span data-ttu-id="f3d29-105">Ele obtém um token com escopo para a função do Azure.</span><span class="sxs-lookup"><span data-stu-id="f3d29-105">It gets back a token that is scoped to the Azure Function.</span></span> <span data-ttu-id="f3d29-106">O token não **contém** escopos do Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="f3d29-106">The token does **NOT** contain any Microsoft Graph scopes.</span></span>
- <span data-ttu-id="f3d29-107">O aplicativo de teste invoca a função do Azure, enviando seu token de acesso no `Authorization` cabeçalho.</span><span class="sxs-lookup"><span data-stu-id="f3d29-107">The test application invokes the Azure Function, sending its access token in the `Authorization` header.</span></span>
- <span data-ttu-id="f3d29-108">A função do Azure valida o token e, em seguida, troca esse token por um segundo token de acesso que contenha escopos do Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="f3d29-108">The Azure Function validates the token, then exchanges that token for a second access token that contains Microsoft Graph scopes.</span></span>
- <span data-ttu-id="f3d29-109">A função do Azure chama o Microsoft Graph no nome do usuário usando o segundo token de acesso.</span><span class="sxs-lookup"><span data-stu-id="f3d29-109">The Azure Function calls Microsoft Graph on the user's behalf using the second access token.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="f3d29-110">Para evitar o armazenamento da ID do aplicativo e o segredo na fonte, você usará o [Gerenciador de segredo do .net](https://docs.microsoft.com/aspnet/core/security/app-secrets) para armazenar esses valores.</span><span class="sxs-lookup"><span data-stu-id="f3d29-110">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="f3d29-111">O gerente secreto é apenas para fins de desenvolvimento, os aplicativos de produção devem usar um Gerenciador de segredo confiável para armazenar segredos.</span><span class="sxs-lookup"><span data-stu-id="f3d29-111">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

## <a name="add-authentication-to-the-single-page-application"></a><span data-ttu-id="f3d29-112">Adicionar autenticação ao aplicativo de página única</span><span class="sxs-lookup"><span data-stu-id="f3d29-112">Add authentication to the single page application</span></span>

<span data-ttu-id="f3d29-113">Comece adicionando autenticação ao SPA.</span><span class="sxs-lookup"><span data-stu-id="f3d29-113">Start by adding authentication to the SPA.</span></span> <span data-ttu-id="f3d29-114">Isso permitirá que o aplicativo obtenha um token de acesso concedendo acesso para chamar a função do Azure.</span><span class="sxs-lookup"><span data-stu-id="f3d29-114">This will allow the application to get an access token granting access to call the Azure Function.</span></span> <span data-ttu-id="f3d29-115">Como este é um SPA, ele usará o [fluxo de código de autorização com o PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span><span class="sxs-lookup"><span data-stu-id="f3d29-115">Because this is a SPA, it will use the [authorization code flow with PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span></span>

1. <span data-ttu-id="f3d29-116">Crie um novo arquivo no diretório **TestClient** chamado **config.js** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="f3d29-116">Create a new file in the **TestClient** directory named **config.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    <span data-ttu-id="f3d29-117">Substitua `YOUR_TEST_APP_APP_ID_HERE` pela ID do aplicativo que você criou no portal do Azure para o **aplicativo de teste de função do Azure Graph**.</span><span class="sxs-lookup"><span data-stu-id="f3d29-117">Replace `YOUR_TEST_APP_APP_ID_HERE` with the application ID you created in the Azure portal for the **Graph Azure Function Test App**.</span></span> <span data-ttu-id="f3d29-118">Substitua `YOUR_TENANT_ID_HERE` pelo valor da **ID de diretório (locatário)** que você copiou do portal do Azure.</span><span class="sxs-lookup"><span data-stu-id="f3d29-118">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span> <span data-ttu-id="f3d29-119">Substitua `YOUR_AZURE_FUNCTION_APP_ID_HERE` pela ID do aplicativo para a **função Graph do Azure**.</span><span class="sxs-lookup"><span data-stu-id="f3d29-119">Replace `YOUR_AZURE_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="f3d29-120">Se você estiver usando o controle de origem como o Git, agora seria uma boa hora para excluir o arquivo **config.js** do controle de origem para evitar vazar inadvertidamente suas IDs de aplicativo e ID de locatário.</span><span class="sxs-lookup"><span data-stu-id="f3d29-120">If you're using source control such as git, now would be a good time to exclude the **config.js** file from source control to avoid inadvertently leaking your app IDs and tenant ID.</span></span>

1. <span data-ttu-id="f3d29-121">Crie um novo arquivo no diretório **TestClient** chamado **auth.js** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="f3d29-121">Create a new file in the **TestClient** directory named **auth.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    <span data-ttu-id="f3d29-122">Considere o que esse código faz.</span><span class="sxs-lookup"><span data-stu-id="f3d29-122">Consider what this code does.</span></span>

    - <span data-ttu-id="f3d29-123">Ele Inicializa um `PublicClientApplication` usando os valores armazenados em **config.js**.</span><span class="sxs-lookup"><span data-stu-id="f3d29-123">It initializes a `PublicClientApplication` using the values stored in **config.js**.</span></span>
    - <span data-ttu-id="f3d29-124">Ele usa `loginPopup` para assinar o usuário no, usando o escopo de permissão para a função do Azure.</span><span class="sxs-lookup"><span data-stu-id="f3d29-124">It uses `loginPopup` to sign the user in, using the permission scope for the Azure Function.</span></span>
    - <span data-ttu-id="f3d29-125">Ele armazena o nome de usuário do usuário na sessão.</span><span class="sxs-lookup"><span data-stu-id="f3d29-125">It stores the user's username in the session.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="f3d29-126">Como o aplicativo usa `loginPopup` , talvez seja necessário alterar o bloqueador de pop-ups do navegador para permitir pop-ups do `http://localhost:8080` .</span><span class="sxs-lookup"><span data-stu-id="f3d29-126">Since the app uses `loginPopup`, you may need to change your browser's pop-up blocker to allow pop-ups from `http://localhost:8080`.</span></span>

1. <span data-ttu-id="f3d29-127">Atualize a página e entre.</span><span class="sxs-lookup"><span data-stu-id="f3d29-127">Refresh the page and sign in.</span></span> <span data-ttu-id="f3d29-128">A página deve ser atualizada com o nome de usuário, indicando que o logon foi bem-sucedido.</span><span class="sxs-lookup"><span data-stu-id="f3d29-128">The page should update with the user name, indicating that the sign in was successful.</span></span>

> [!TIP]
> <span data-ttu-id="f3d29-129">Você pode analisar o token de acesso em [https://jwt.ms](https://jwt.ms) e confirmar que a `aud` declaração é a ID do aplicativo para a função do Azure e que a `scp` declaração contém o escopo de permissão da função do Azure, e não o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="f3d29-129">You can parse the access token at [https://jwt.ms](https://jwt.ms) and confirm that the `aud` claim is the app ID for the Azure Function, and that the `scp` claim contains the Azure Function's permission scope, not Microsoft Graph.</span></span>

## <a name="add-authentication-to-the-azure-function"></a><span data-ttu-id="f3d29-130">Adicionar autenticação à função do Azure</span><span class="sxs-lookup"><span data-stu-id="f3d29-130">Add authentication to the Azure Function</span></span>

<span data-ttu-id="f3d29-131">Nesta seção, você implementará o fluxo em nome de na `GetMyNewestMessage` função do Azure para obter um token de acesso compatível com o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="f3d29-131">In this section you'll implement the on-behalf-of flow in the `GetMyNewestMessage` Azure Function to get an access token compatible with Microsoft Graph.</span></span>

1. <span data-ttu-id="f3d29-132">Inicialize o repositório de segredo de desenvolvimento do .NET abrindo sua CLI no diretório que contém **GraphTutorial. csproj** e executando o comando a seguir.</span><span class="sxs-lookup"><span data-stu-id="f3d29-132">Initialize the .NET development secret store by opening your CLI in the directory that contains **GraphTutorial.csproj** and running the following command.</span></span>

    ```Shell
    dotnet user-secrets init
    ```

1. <span data-ttu-id="f3d29-133">Adicione a ID do aplicativo, o segredo e a ID do locatário ao repositório secreto usando os seguintes comandos.</span><span class="sxs-lookup"><span data-stu-id="f3d29-133">Add your application ID, secret, and tenant ID to the secret store using the following commands.</span></span> <span data-ttu-id="f3d29-134">Substitua `YOUR_API_FUNCTION_APP_ID_HERE` pela ID do aplicativo para a **função Graph do Azure**.</span><span class="sxs-lookup"><span data-stu-id="f3d29-134">Replace `YOUR_API_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span> <span data-ttu-id="f3d29-135">Substitua `YOUR_API_FUNCTION_APP_SECRET_HERE` pelo segredo do aplicativo que você criou no portal do Azure para a **função Graph do Azure**.</span><span class="sxs-lookup"><span data-stu-id="f3d29-135">Replace `YOUR_API_FUNCTION_APP_SECRET_HERE` with the application secret you created in the Azure portal for the **Graph Azure Function**.</span></span> <span data-ttu-id="f3d29-136">Substitua `YOUR_TENANT_ID_HERE` pelo valor da **ID de diretório (locatário)** que você copiou do portal do Azure.</span><span class="sxs-lookup"><span data-stu-id="f3d29-136">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span>

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a><span data-ttu-id="f3d29-137">Processar o token de portador de entrada</span><span class="sxs-lookup"><span data-stu-id="f3d29-137">Process the incoming bearer token</span></span>

<span data-ttu-id="f3d29-138">Nesta seção, você implementará uma classe para validar e processar o token de portador enviado do SPA para a função do Azure.</span><span class="sxs-lookup"><span data-stu-id="f3d29-138">In this section you'll implement a class to validate and process the bearer token sent from the SPA to the Azure Function.</span></span>

1. <span data-ttu-id="f3d29-139">Crie um novo diretório no diretório **GraphTutorial** chamado **autenticação**.</span><span class="sxs-lookup"><span data-stu-id="f3d29-139">Create a new directory in the **GraphTutorial** directory named **Authentication**.</span></span>

1. <span data-ttu-id="f3d29-140">Crie um novo arquivo denominado **TokenValidationResult.cs** na pasta **./GraphTutorial/Authentication** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="f3d29-140">Create a new file named **TokenValidationResult.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. <span data-ttu-id="f3d29-141">Crie um novo arquivo denominado **TokenValidation.cs** na pasta **./GraphTutorial/Authentication** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="f3d29-141">Create a new file named **TokenValidation.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

<span data-ttu-id="f3d29-142">Considere o que esse código faz.</span><span class="sxs-lookup"><span data-stu-id="f3d29-142">Consider what this code does.</span></span>

- <span data-ttu-id="f3d29-143">Ele garante que há um token de portador no `Authorization` cabeçalho.</span><span class="sxs-lookup"><span data-stu-id="f3d29-143">It ensure there is a bearer token in the `Authorization` header.</span></span>
- <span data-ttu-id="f3d29-144">Ele verifica a assinatura e o emissor da configuração de OpenID publicada do Azure.</span><span class="sxs-lookup"><span data-stu-id="f3d29-144">It verifies the signature and issuer from Azure's published OpenID configuration.</span></span>
- <span data-ttu-id="f3d29-145">Ele verifica se a audiência ( `aud` declaração) corresponde à ID do aplicativo da função do Azure.</span><span class="sxs-lookup"><span data-stu-id="f3d29-145">It verifies that the audience (`aud` claim) matches the Azure Function's application ID.</span></span>
- <span data-ttu-id="f3d29-146">Ele analisa o token e gera uma ID de conta MSAL, que será necessária para aproveitar o cache de token.</span><span class="sxs-lookup"><span data-stu-id="f3d29-146">It parses the token and generates an MSAL account ID, which will be needed to take advantage of token caching.</span></span>

### <a name="create-an-on-behalf-of-authentication-provider"></a><span data-ttu-id="f3d29-147">Criar um provedor de autenticação em nome de</span><span class="sxs-lookup"><span data-stu-id="f3d29-147">Create an on-behalf-of authentication provider</span></span>

1. <span data-ttu-id="f3d29-148">Crie um novo arquivo no diretório de **autenticação** chamado **OnBehalfOfAuthProvider.cs** e adicione o código a seguir ao arquivo.</span><span class="sxs-lookup"><span data-stu-id="f3d29-148">Create a new file in the **Authentication** directory named **OnBehalfOfAuthProvider.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

<span data-ttu-id="f3d29-149">Reserve um tempo para considerar o que o código no **OnBehalfOfAuthProvider.cs** faz.</span><span class="sxs-lookup"><span data-stu-id="f3d29-149">Take a moment to consider what the code in **OnBehalfOfAuthProvider.cs** does.</span></span>

- <span data-ttu-id="f3d29-150">Na `GetAccessToken` função, ele primeiro tenta obter um token de usuário do cache de token usando `AcquireTokenSilent` .</span><span class="sxs-lookup"><span data-stu-id="f3d29-150">In the `GetAccessToken` function, it first attempts to get a user token from the token cache using `AcquireTokenSilent`.</span></span> <span data-ttu-id="f3d29-151">Se isso falhar, ele usará o token de portador enviado pelo aplicativo de teste para a função do Azure para gerar uma asserção do usuário.</span><span class="sxs-lookup"><span data-stu-id="f3d29-151">If this fails, it uses the bearer token sent by the test app to the Azure Function to generate a user assertion.</span></span> <span data-ttu-id="f3d29-152">Em seguida, ele usa essa declaração de usuário para obter um token compatível com gráfico usando o `AcquireTokenOnBehalfOf` .</span><span class="sxs-lookup"><span data-stu-id="f3d29-152">It then uses that user assertion to get a Graph-compatible token using `AcquireTokenOnBehalfOf`.</span></span>
- <span data-ttu-id="f3d29-153">Ele implementa a `Microsoft.Graph.IAuthenticationProvider` interface, permitindo que essa classe seja passada no construtor do `GraphServiceClient` para autenticar solicitações de saída.</span><span class="sxs-lookup"><span data-stu-id="f3d29-153">It implements the `Microsoft.Graph.IAuthenticationProvider` interface, allowing this class to be passed in the constructor of the `GraphServiceClient` to authenticate outgoing requests.</span></span>

### <a name="implement-a-graph-client-service"></a><span data-ttu-id="f3d29-154">Implementar um serviço de cliente do Graph</span><span class="sxs-lookup"><span data-stu-id="f3d29-154">Implement a Graph client service</span></span>

<span data-ttu-id="f3d29-155">Nesta seção, você implementará um serviço que pode ser registrado para [injeção de dependência](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection).</span><span class="sxs-lookup"><span data-stu-id="f3d29-155">In this section you'll implement a service that can be registered for [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection).</span></span> <span data-ttu-id="f3d29-156">O serviço será usado para obter um cliente de gráfico autenticado.</span><span class="sxs-lookup"><span data-stu-id="f3d29-156">The service will be used to get an authenticated Graph client.</span></span>

1. <span data-ttu-id="f3d29-157">Crie um novo diretório no diretório **GraphTutorial** chamado **Services**.</span><span class="sxs-lookup"><span data-stu-id="f3d29-157">Create a new directory in the **GraphTutorial** directory named **Services**.</span></span>

1. <span data-ttu-id="f3d29-158">Crie um novo arquivo no diretório de **Serviços** chamado **IGraphClientService.cs** e adicione o código a seguir ao arquivo.</span><span class="sxs-lookup"><span data-stu-id="f3d29-158">Create a new file in the **Services** directory named **IGraphClientService.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. <span data-ttu-id="f3d29-159">Crie um novo arquivo no diretório de **Serviços** chamado **GraphClientService.cs** e adicione o código a seguir ao arquivo.</span><span class="sxs-lookup"><span data-stu-id="f3d29-159">Create a new file in the **Services** directory named **GraphClientService.cs** and add the following code to that file.</span></span>

    ```csharp
    using GraphTutorial.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;

    namespace GraphTutorial.Services
    {
        // Service added via dependency injection
        // Used to get an authenticated Graph client
        public class GraphClientService : IGraphClientService
        {
        }
    }
    ```

1. <span data-ttu-id="f3d29-160">Adicione as propriedades a seguir à `GraphClientService` classe.</span><span class="sxs-lookup"><span data-stu-id="f3d29-160">Add the following properties to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. <span data-ttu-id="f3d29-161">Adicione as seguintes funções à `GraphClientService` classe.</span><span class="sxs-lookup"><span data-stu-id="f3d29-161">Add the following functions to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. <span data-ttu-id="f3d29-162">Adicione uma implementação de espaço reservado para a `GetAppGraphClient` função.</span><span class="sxs-lookup"><span data-stu-id="f3d29-162">Add a placeholder implementation for the `GetAppGraphClient` function.</span></span> <span data-ttu-id="f3d29-163">Você irá implementá-lo nas seções posteriores.</span><span class="sxs-lookup"><span data-stu-id="f3d29-163">You will implement that in later sections.</span></span>

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    <span data-ttu-id="f3d29-164">A `GetUserGraphClient` função utiliza os resultados da validação do token e cria um autenticado `GraphServiceClient` para o usuário.</span><span class="sxs-lookup"><span data-stu-id="f3d29-164">The `GetUserGraphClient` function takes the results of token validation and builds an authenticated `GraphServiceClient` for the user.</span></span>

1. <span data-ttu-id="f3d29-165">Crie um novo arquivo no diretório **GraphTutorial** chamado **Startup.cs** e adicione o código a seguir ao arquivo.</span><span class="sxs-lookup"><span data-stu-id="f3d29-165">Create a new file in the **GraphTutorial** directory named **Startup.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="StartupSnippet":::

    <span data-ttu-id="f3d29-166">Este código habilitará a [injeção de dependência](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) em suas funções do Azure, expondo o `IConfiguration` objeto e o `GraphClientService` serviço.</span><span class="sxs-lookup"><span data-stu-id="f3d29-166">This code will enable [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in your Azure Functions, exposing the `IConfiguration` object and the `GraphClientService` service.</span></span>

### <a name="implement-getmynewestmessage-function"></a><span data-ttu-id="f3d29-167">Implementar a função GetMyNewestMessage</span><span class="sxs-lookup"><span data-stu-id="f3d29-167">Implement GetMyNewestMessage function</span></span>

1. <span data-ttu-id="f3d29-168">Abra **./GraphTutorial/GetMyNewestMessage.cs** e substitua todo o conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="f3d29-168">Open **./GraphTutorial/GetMyNewestMessage.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a><span data-ttu-id="f3d29-169">Revise o código no GetMyNewestMessage.cs</span><span class="sxs-lookup"><span data-stu-id="f3d29-169">Review the code in GetMyNewestMessage.cs</span></span>

<span data-ttu-id="f3d29-170">Reserve um tempo para considerar o que o código no **GetMyNewestMessage.cs** faz.</span><span class="sxs-lookup"><span data-stu-id="f3d29-170">Take a moment to consider what the code in **GetMyNewestMessage.cs** does.</span></span>

- <span data-ttu-id="f3d29-171">No construtor, ele salva o `IConfiguration` objeto transmitido por meio da injeção de dependência.</span><span class="sxs-lookup"><span data-stu-id="f3d29-171">In the constructor, it saves the `IConfiguration` object passed in via dependency injection.</span></span>
- <span data-ttu-id="f3d29-172">Na `Run` função, ele faz o seguinte:</span><span class="sxs-lookup"><span data-stu-id="f3d29-172">In the `Run` function, it does the following:</span></span>
  - <span data-ttu-id="f3d29-173">Valida os valores de configuração necessários estão presentes no `IConfiguration` objeto.</span><span class="sxs-lookup"><span data-stu-id="f3d29-173">Validates the required configuration values are present in the `IConfiguration` object.</span></span>
  - <span data-ttu-id="f3d29-174">Valida o token de portador e retorna um `401` código de status se o token for inválido.</span><span class="sxs-lookup"><span data-stu-id="f3d29-174">Validates the bearer token and returns a `401` status code if the token is invalid.</span></span>
  - <span data-ttu-id="f3d29-175">Obtém um cliente de gráfico do `GraphClientService` para o usuário que fez essa solicitação.</span><span class="sxs-lookup"><span data-stu-id="f3d29-175">Gets a Graph client from the `GraphClientService` for the user that made this request.</span></span>
  - <span data-ttu-id="f3d29-176">Usa o SDK do Microsoft Graph para obter a mensagem mais recente da caixa de entrada do usuário e o retorna como um corpo JSON na resposta.</span><span class="sxs-lookup"><span data-stu-id="f3d29-176">Uses the Microsoft Graph SDK to get the newest message from the user's inbox and returns it as a JSON body in the response.</span></span>

## <a name="call-the-azure-function-from-the-test-app"></a><span data-ttu-id="f3d29-177">Chamar a função do Azure no aplicativo de teste</span><span class="sxs-lookup"><span data-stu-id="f3d29-177">Call the Azure Function from the test app</span></span>

1. <span data-ttu-id="f3d29-178">Abra **auth.js** e adicione a seguinte função para obter um token de acesso.</span><span class="sxs-lookup"><span data-stu-id="f3d29-178">Open **auth.js** and add the following function to get an access token.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    <span data-ttu-id="f3d29-179">Considere o que esse código faz.</span><span class="sxs-lookup"><span data-stu-id="f3d29-179">Consider what this code does.</span></span>

    - <span data-ttu-id="f3d29-180">Primeiro, ele tenta obter um token de acesso de forma silenciosa, sem interação do usuário.</span><span class="sxs-lookup"><span data-stu-id="f3d29-180">It first attempts to get an access token silently, without user interaction.</span></span> <span data-ttu-id="f3d29-181">Como o usuário já deve estar conectado, MSAL deve ter tokens para o usuário em seu cache.</span><span class="sxs-lookup"><span data-stu-id="f3d29-181">Since the user should already be signed in, MSAL should have tokens for the user in its cache.</span></span>
    - <span data-ttu-id="f3d29-182">Se isso falhar com um erro que indica que o usuário precisa interagir, ele tentará obter um token interativamente.</span><span class="sxs-lookup"><span data-stu-id="f3d29-182">If that fails with an error that indicates the user needs to interact, it attempts to get a token interactively.</span></span>

1. <span data-ttu-id="f3d29-183">Crie um novo arquivo no diretório **TestClient** chamado **azurefunctions.js** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="f3d29-183">Create a new file in the **TestClient** directory named **azurefunctions.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. <span data-ttu-id="f3d29-184">Altere o diretório atual em sua CLI para o diretório **./GraphTutorial** e execute o seguinte comando para iniciar a função do Azure localmente.</span><span class="sxs-lookup"><span data-stu-id="f3d29-184">Change the current directory in your CLI to the **./GraphTutorial** directory and run the following command to start the Azure Function locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="f3d29-185">Se ainda não estiver servindo a SPA, abra uma segunda janela CLI e altere o diretório atual para o diretório **./TestClient** .</span><span class="sxs-lookup"><span data-stu-id="f3d29-185">If not already serving the SPA, open a second CLI window and change the current directory to the **./TestClient** directory.</span></span> <span data-ttu-id="f3d29-186">Execute o comando a seguir para executar o aplicativo de teste.</span><span class="sxs-lookup"><span data-stu-id="f3d29-186">Run the following command to run the test application.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. <span data-ttu-id="f3d29-187">Abra o navegador e vá até `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="f3d29-187">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="f3d29-188">Entre e selecione o item de navegação de **mensagens mais recente** .</span><span class="sxs-lookup"><span data-stu-id="f3d29-188">Sign in and select the **Latest Message** navigation item.</span></span> <span data-ttu-id="f3d29-189">O aplicativo exibe informações sobre a mensagem mais recente na caixa de entrada do usuário.</span><span class="sxs-lookup"><span data-stu-id="f3d29-189">The app displays information about the newest message in the user's inbox.</span></span>
