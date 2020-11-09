---
ms.openlocfilehash: 728138b5d298f9469a566defed6ee2e457896c0b
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822555"
---
<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você concluirá a implementação da função do Azure `GetMyNewestMessage` e atualizará o cliente de teste para chamar a função.

A função do Azure usa o [fluxo em nome de](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow). A ordem básica de eventos nesse fluxo são:

- O aplicativo de teste usa um fluxo de autenticação interativa para permitir que o usuário entre e conceda o consentimento. Ele obtém um token com escopo para a função do Azure. O token não **contém** escopos do Microsoft Graph.
- O aplicativo de teste invoca a função do Azure, enviando seu token de acesso no `Authorization` cabeçalho.
- A função do Azure valida o token e, em seguida, troca esse token por um segundo token de acesso que contenha escopos do Microsoft Graph.
- A função do Azure chama o Microsoft Graph no nome do usuário usando o segundo token de acesso.

> [!IMPORTANT]
> Para evitar o armazenamento da ID do aplicativo e o segredo na fonte, você usará o [Gerenciador de segredo do .net](https://docs.microsoft.com/aspnet/core/security/app-secrets) para armazenar esses valores. O gerente secreto é apenas para fins de desenvolvimento, os aplicativos de produção devem usar um Gerenciador de segredo confiável para armazenar segredos.

## <a name="add-authentication-to-the-single-page-application"></a>Adicionar autenticação ao aplicativo de página única

Comece adicionando autenticação ao SPA. Isso permitirá que o aplicativo obtenha um token de acesso concedendo acesso para chamar a função do Azure. Como este é um SPA, ele usará o [fluxo de código de autorização com o PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).

1. Crie um novo arquivo no diretório **TestClient** chamado **config.js** e adicione o código a seguir.

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    Substitua `YOUR_TEST_APP_APP_ID_HERE` pela ID do aplicativo que você criou no portal do Azure para o **aplicativo de teste de função do Azure Graph**. Substitua `YOUR_TENANT_ID_HERE` pelo valor da **ID de diretório (locatário)** que você copiou do portal do Azure. Substitua `YOUR_AZURE_FUNCTION_APP_ID_HERE` pela ID do aplicativo para a **função Graph do Azure**.

    > [!IMPORTANT]
    > Se você estiver usando o controle de origem como o Git, agora seria uma boa hora para excluir o arquivo **config.js** do controle de origem para evitar vazar inadvertidamente suas IDs de aplicativo e ID de locatário.

1. Crie um novo arquivo no diretório **TestClient** chamado **auth.js** e adicione o código a seguir.

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    Considere o que esse código faz.

    - Ele Inicializa um `PublicClientApplication` usando os valores armazenados em **config.js**.
    - Ele usa `loginPopup` para assinar o usuário no, usando o escopo de permissão para a função do Azure.
    - Ele armazena o nome de usuário do usuário na sessão.

    > [!IMPORTANT]
    > Como o aplicativo usa `loginPopup` , talvez seja necessário alterar o bloqueador de pop-ups do navegador para permitir pop-ups do `http://localhost:8080` .

1. Atualize a página e entre. A página deve ser atualizada com o nome de usuário, indicando que o logon foi bem-sucedido.

> [!TIP]
> Você pode analisar o token de acesso em [https://jwt.ms](https://jwt.ms) e confirmar que a `aud` declaração é a ID do aplicativo para a função do Azure e que a `scp` declaração contém o escopo de permissão da função do Azure, e não o Microsoft Graph.

## <a name="add-authentication-to-the-azure-function"></a>Adicionar autenticação à função do Azure

Nesta seção, você implementará o fluxo em nome de na `GetMyNewestMessage` função do Azure para obter um token de acesso compatível com o Microsoft Graph.

1. Inicialize o repositório de segredo de desenvolvimento do .NET abrindo sua CLI no diretório que contém **GraphTutorial. csproj** e executando o comando a seguir.

    ```Shell
    dotnet user-secrets init
    ```

1. Adicione a ID do aplicativo, o segredo e a ID do locatário ao repositório secreto usando os seguintes comandos. Substitua `YOUR_API_FUNCTION_APP_ID_HERE` pela ID do aplicativo para a **função Graph do Azure**. Substitua `YOUR_API_FUNCTION_APP_SECRET_HERE` pelo segredo do aplicativo que você criou no portal do Azure para a **função Graph do Azure**. Substitua `YOUR_TENANT_ID_HERE` pelo valor da **ID de diretório (locatário)** que você copiou do portal do Azure.

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a>Processar o token de portador de entrada

Nesta seção, você implementará uma classe para validar e processar o token de portador enviado do SPA para a função do Azure.

1. Crie um novo diretório no diretório **GraphTutorial** chamado **autenticação**.

1. Crie um novo arquivo denominado **TokenValidationResult.cs** na pasta **./GraphTutorial/Authentication** e adicione o código a seguir.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. Crie um novo arquivo denominado **TokenValidation.cs** na pasta **./GraphTutorial/Authentication** e adicione o código a seguir.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

Considere o que esse código faz.

- Ele garante que há um token de portador no `Authorization` cabeçalho.
- Ele verifica a assinatura e o emissor da configuração de OpenID publicada do Azure.
- Ele verifica se a audiência ( `aud` declaração) corresponde à ID do aplicativo da função do Azure.
- Ele analisa o token e gera uma ID de conta MSAL, que será necessária para aproveitar o cache de token.

### <a name="create-an-on-behalf-of-authentication-provider"></a>Criar um provedor de autenticação em nome de

1. Crie um novo arquivo no diretório de **autenticação** chamado **OnBehalfOfAuthProvider.cs** e adicione o código a seguir ao arquivo.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

Reserve um tempo para considerar o que o código no **OnBehalfOfAuthProvider.cs** faz.

- Na `GetAccessToken` função, ele primeiro tenta obter um token de usuário do cache de token usando `AcquireTokenSilent` . Se isso falhar, ele usará o token de portador enviado pelo aplicativo de teste para a função do Azure para gerar uma asserção do usuário. Em seguida, ele usa essa declaração de usuário para obter um token compatível com gráfico usando o `AcquireTokenOnBehalfOf` .
- Ele implementa a `Microsoft.Graph.IAuthenticationProvider` interface, permitindo que essa classe seja passada no construtor do `GraphServiceClient` para autenticar solicitações de saída.

### <a name="implement-a-graph-client-service"></a>Implementar um serviço de cliente do Graph

Nesta seção, você implementará um serviço que pode ser registrado para [injeção de dependência](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection). O serviço será usado para obter um cliente de gráfico autenticado.

1. Crie um novo diretório no diretório **GraphTutorial** chamado **Services**.

1. Crie um novo arquivo no diretório de **Serviços** chamado **IGraphClientService.cs** e adicione o código a seguir ao arquivo.

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. Crie um novo arquivo no diretório de **Serviços** chamado **GraphClientService.cs** e adicione o código a seguir ao arquivo.

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

1. Adicione as propriedades a seguir à `GraphClientService` classe.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. Adicione as seguintes funções à `GraphClientService` classe.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. Adicione uma implementação de espaço reservado para a `GetAppGraphClient` função. Você irá implementá-lo nas seções posteriores.

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    A `GetUserGraphClient` função utiliza os resultados da validação do token e cria um autenticado `GraphServiceClient` para o usuário.

1. Crie um novo arquivo no diretório **GraphTutorial** chamado **Startup.cs** e adicione o código a seguir ao arquivo.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="StartupSnippet":::

    Este código habilitará a [injeção de dependência](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) em suas funções do Azure, expondo o `IConfiguration` objeto e o `GraphClientService` serviço.

### <a name="implement-getmynewestmessage-function"></a>Implementar a função GetMyNewestMessage

1. Abra **./GraphTutorial/GetMyNewestMessage.cs** e substitua todo o conteúdo pelo seguinte.

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a>Revise o código no GetMyNewestMessage.cs

Reserve um tempo para considerar o que o código no **GetMyNewestMessage.cs** faz.

- No construtor, ele salva o `IConfiguration` objeto transmitido por meio da injeção de dependência.
- Na `Run` função, ele faz o seguinte:
  - Valida os valores de configuração necessários estão presentes no `IConfiguration` objeto.
  - Valida o token de portador e retorna um `401` código de status se o token for inválido.
  - Obtém um cliente de gráfico do `GraphClientService` para o usuário que fez essa solicitação.
  - Usa o SDK do Microsoft Graph para obter a mensagem mais recente da caixa de entrada do usuário e o retorna como um corpo JSON na resposta.

## <a name="call-the-azure-function-from-the-test-app"></a>Chamar a função do Azure no aplicativo de teste

1. Abra **auth.js** e adicione a seguinte função para obter um token de acesso.

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    Considere o que esse código faz.

    - Primeiro, ele tenta obter um token de acesso de forma silenciosa, sem interação do usuário. Como o usuário já deve estar conectado, MSAL deve ter tokens para o usuário em seu cache.
    - Se isso falhar com um erro que indica que o usuário precisa interagir, ele tentará obter um token interativamente.

1. Crie um novo arquivo no diretório **TestClient** chamado **azurefunctions.js** e adicione o código a seguir.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. Altere o diretório atual em sua CLI para o diretório **./GraphTutorial** e execute o seguinte comando para iniciar a função do Azure localmente.

    ```Shell
    func start
    ```

1. Se ainda não estiver servindo a SPA, abra uma segunda janela CLI e altere o diretório atual para o diretório **./TestClient** . Execute o comando a seguir para executar o aplicativo de teste.

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. Abra o navegador e vá até `http://localhost:8080`. Entre e selecione o item de navegação de **mensagens mais recente** . O aplicativo exibe informações sobre a mensagem mais recente na caixa de entrada do usuário.
