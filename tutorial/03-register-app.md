---
ms.openlocfilehash: 0c9c6703a54e50f576a171a4a161305c9b9ae6ff
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822553"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="0cfa6-101">Neste exercício, você criará três novos aplicativos do Azure AD usando o centro de administração do Azure Active Directory:</span><span class="sxs-lookup"><span data-stu-id="0cfa6-101">In this exercise you will create three new Azure AD applications using the Azure Active Directory admin center:</span></span>

- <span data-ttu-id="0cfa6-102">Um registro de aplicativo para o aplicativo de página única para que ele possa entrar em usuários e obter tokens permitindo que o aplicativo chame a função do Azure.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-102">An app registration for the single-page application so that it can sign in users and get tokens allowing the application to call the Azure Function.</span></span>
- <span data-ttu-id="0cfa6-103">Um registro de aplicativo para a função do Azure que permite que ele use o [fluxo em nome de](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) para trocar o token enviado pelo Spa para um token que lhe permitirá chamar o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-103">An app registration for the Azure Function that allows it to use the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) to exchange the token sent by the SPA for a token that will allow it to call Microsoft Graph.</span></span>
- <span data-ttu-id="0cfa6-104">Um registro de aplicativo para o webhook de função do Azure que permite que ele use o [fluxo de credenciais do cliente](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) para chamar o Microsoft Graph sem um usuário.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-104">An app registration for the Azure Function webhook that allows it to use the [client credential flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) to call Microsoft Graph without a user.</span></span>

> [!NOTE]
> <span data-ttu-id="0cfa6-105">Este exemplo requer três registros de aplicativo, pois está implementando o fluxo em nome de e o fluxo de credenciais do cliente.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-105">This example requires three app registrations because it is implementing both the on-behalf-of flow and the client credential flow.</span></span> <span data-ttu-id="0cfa6-106">Se sua função do Azure usar apenas um desses fluxos, você só precisará criar os registros de aplicativo que correspondem a esse fluxo.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-106">If your Azure Function only uses one of these flows, you would only need to create the app registrations that correspond to that flow.</span></span>

1. <span data-ttu-id="0cfa6-107">Abra um navegador e navegue até o [centro de administração do Azure Active Directory](https://aad.portal.azure.com) e faça logon usando um administrador da organização de locatários do Microsoft 365.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-107">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com) and login using an Microsoft 365 tenant organization admin.</span></span>

1. <span data-ttu-id="0cfa6-108">Selecione **Azure Active Directory** na navegação esquerda e selecione **Registros de aplicativos** em **Gerenciar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-108">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="0cfa6-109">Uma captura de tela dos registros de aplicativo</span><span class="sxs-lookup"><span data-stu-id="0cfa6-109">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

## <a name="register-an-app-for-the-single-page-application"></a><span data-ttu-id="0cfa6-110">Registrar um aplicativo para o aplicativo de página única</span><span class="sxs-lookup"><span data-stu-id="0cfa6-110">Register an app for the single-page application</span></span>

1. <span data-ttu-id="0cfa6-111">Selecione **Novo registro**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-111">Select **New registration**.</span></span> <span data-ttu-id="0cfa6-112">Na página **Registrar um aplicativo** , defina os valores da seguinte forma.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-112">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="0cfa6-113">Defina **Nome** para `Graph Azure Function Test App`.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-113">Set **Name** to `Graph Azure Function Test App`.</span></span>
    - <span data-ttu-id="0cfa6-114">Defina os **tipos de conta com suporte** para **contas nesse diretório organizacional apenas**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-114">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="0cfa6-115">Em **URI de redirecionamento** , altere o menu suspenso para o **aplicativo de página única (Spa)** e defina o valor como `http://localhost:8080` .</span><span class="sxs-lookup"><span data-stu-id="0cfa6-115">Under **Redirect URI** , change the dropdown to **Single-page application (SPA)** and set the value to `http://localhost:8080`.</span></span>

    ![Uma captura de tela da página registrar um aplicativo](./images/register-command-line-app.png)

1. <span data-ttu-id="0cfa6-117">Selecione **Registrar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-117">Select **Register**.</span></span> <span data-ttu-id="0cfa6-118">Na página do **aplicativo de teste de função do Azure Graph** , copie os valores da ID do **aplicativo (cliente)** e da ID do **diretório (locatário)** e salve-os, você precisará deles nas etapas posteriores.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-118">On the **Graph Azure Function Test App** page, copy the values of the **Application (client) ID** and **Directory (tenant) ID** and save them, you will need them in the later steps.</span></span>

    ![Uma captura de tela da ID do aplicativo do novo registro de aplicativo](./images/aad-application-id.png)

## <a name="register-an-app-for-the-azure-function"></a><span data-ttu-id="0cfa6-120">Registrar um aplicativo para a função do Azure</span><span class="sxs-lookup"><span data-stu-id="0cfa6-120">Register an app for the Azure Function</span></span>

1. <span data-ttu-id="0cfa6-121">Retorne a **registros de aplicativos** e selecione **novo registro**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-121">Return to **App Registrations** , and select **New registration**.</span></span> <span data-ttu-id="0cfa6-122">Na página **Registrar um aplicativo** , defina os valores da seguinte forma.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-122">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="0cfa6-123">Defina **Nome** para `Graph Azure Function`.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-123">Set **Name** to `Graph Azure Function`.</span></span>
    - <span data-ttu-id="0cfa6-124">Defina os **tipos de conta com suporte** para **contas nesse diretório organizacional apenas**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-124">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="0cfa6-125">Deixe **URI de redirecionamento** em branco.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-125">Leave **Redirect URI** blank.</span></span>

1. <span data-ttu-id="0cfa6-126">Selecione **Registrar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-126">Select **Register**.</span></span> <span data-ttu-id="0cfa6-127">Na página de **função do Azure Graph** , copie o valor da **ID do aplicativo (cliente)** e salve-o, você precisará dele na próxima etapa.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-127">On the **Graph Azure Function** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

1. <span data-ttu-id="0cfa6-128">Selecione **Certificados e segredos** sob **Gerenciar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-128">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="0cfa6-129">Selecione o botão **Novo segredo do cliente**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-129">Select the **New client secret** button.</span></span> <span data-ttu-id="0cfa6-130">Insira um valor em **Descrição** e selecione uma das opções para **expirar** e selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-130">Enter a value in **Description** and select one of the options for **Expires** and select **Add**.</span></span>

    ![Uma captura de tela da caixa de diálogo Adicionar um segredo do cliente](./images/aad-new-client-secret.png)

1. <span data-ttu-id="0cfa6-132">Copie o valor secreto do cliente antes de sair desta página.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-132">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="0cfa6-133">Você precisará dele na próxima etapa.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-133">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="0cfa6-134">Este segredo do cliente nunca é mostrado novamente, portanto, copie-o agora.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-134">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![Uma captura de tela do novo segredo do cliente recentemente adicionado](./images/aad-copy-client-secret.png)

1. <span data-ttu-id="0cfa6-136">Selecione **permissões de API** em **gerenciar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-136">Select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="0cfa6-137">Escolha **Adicionar uma permissão**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-137">Choose **Add a permission**.</span></span>

1. <span data-ttu-id="0cfa6-138">Selecione **Microsoft Graph** e, em seguida, **permissões delegadas**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-138">Select **Microsoft Graph** , then **Delegated Permissions**.</span></span> <span data-ttu-id="0cfa6-139">Adicione **mail. Read** e selecione **Add Permissions**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-139">Add **Mail.Read** and select **Add permissions**.</span></span>

    ![Uma captura de tela das permissões configuradas para o registro de aplicativo de função do Azure](./images/web-api-configured-permissions.png)

1. <span data-ttu-id="0cfa6-141">Selecione **expor uma API** em **gerenciar** e, em seguida, escolha **Adicionar um escopo**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-141">Select **Expose an API** under **Manage** , then choose **Add a scope**.</span></span>

1. <span data-ttu-id="0cfa6-142">Aceite o **URI da ID do aplicativo** padrão e escolha **salvar e continuar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-142">Accept the default **Application ID URI** and choose **Save and continue**.</span></span>

1. <span data-ttu-id="0cfa6-143">Preencha o formulário **Adicionar um escopo** da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="0cfa6-143">Fill in the **Add a scope** form as follows:</span></span>

    - <span data-ttu-id="0cfa6-144">**Nome do escopo:** Mail. Read</span><span class="sxs-lookup"><span data-stu-id="0cfa6-144">**Scope name:** Mail.Read</span></span>
    - <span data-ttu-id="0cfa6-145">**Quem pode consenter?:** Administradores e usuários</span><span class="sxs-lookup"><span data-stu-id="0cfa6-145">**Who can consent?:** Admins and users</span></span>
    - <span data-ttu-id="0cfa6-146">**Nome para exibição do consentimento do administrador:** Ler caixas de entrada de todos os usuários</span><span class="sxs-lookup"><span data-stu-id="0cfa6-146">**Admin consent display name:** Read all users' inboxes</span></span>
    - <span data-ttu-id="0cfa6-147">**Descrição do consentimento do administrador:** Permite que o aplicativo Leia as caixas de entrada de todos os usuários</span><span class="sxs-lookup"><span data-stu-id="0cfa6-147">**Admin consent description:** Allows the app to read all users' inboxes</span></span>
    - <span data-ttu-id="0cfa6-148">**Nome para exibição do consentimento do usuário:** Ler sua caixa de entrada</span><span class="sxs-lookup"><span data-stu-id="0cfa6-148">**User consent display name:** Read your inbox</span></span>
    - <span data-ttu-id="0cfa6-149">**Descrição do consentimento do usuário:** Permite que o aplicativo Leia sua caixa de entrada</span><span class="sxs-lookup"><span data-stu-id="0cfa6-149">**User consent description:** Allows the app to read your inbox</span></span>
    - <span data-ttu-id="0cfa6-150">**Estado:** Permiti</span><span class="sxs-lookup"><span data-stu-id="0cfa6-150">**State:** Enabled</span></span>

1. <span data-ttu-id="0cfa6-151">Selecione **Adicionar escopo**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-151">Select **Add scope**.</span></span>

1. <span data-ttu-id="0cfa6-152">Copie o novo escopo, você precisará dele nas etapas posteriores.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-152">Copy the new scope, you'll need it in later steps.</span></span>

    ![Uma captura de tela dos escopos definidos para o registro de aplicativo de função do Azure](./images/web-api-defined-scopes.png)

1. <span data-ttu-id="0cfa6-154">Selecione **manifesto** em **gerenciar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-154">Select **Manifest** under **Manage**.</span></span>

1. <span data-ttu-id="0cfa6-155">Localize `knownClientApplications` no manifesto e substitua o valor atual `[]` por `[TEST_APP_ID]` , em que `TEST_APP_ID` é a ID do aplicativo do registro do aplicativo de **aplicativo de teste de função do Azure** .</span><span class="sxs-lookup"><span data-stu-id="0cfa6-155">Locate `knownClientApplications` in the manifest, and replace it's current value of `[]` with `[TEST_APP_ID]`, where `TEST_APP_ID` is the application ID of the **Graph Azure Function Test App** app registration.</span></span> <span data-ttu-id="0cfa6-156">Selecione **Salvar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-156">Select **Save**.</span></span>

> [!NOTE]
> <span data-ttu-id="0cfa6-157">A adição da ID de aplicativo do aplicativo de teste à `knownClientApplications` propriedade no manifesto da função do Azure permite que o aplicativo de teste dispare um [fluxo de consentimento combinado](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow#default-and-combined-consent).</span><span class="sxs-lookup"><span data-stu-id="0cfa6-157">Adding the test application's app ID to the `knownClientApplications` property in the Azure Function's manifest allows the test application to trigger a [combined consent flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow#default-and-combined-consent).</span></span> <span data-ttu-id="0cfa6-158">Isso é necessário para o fluxo em nome de para funcionar.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-158">This is necessary for the on-behalf-of flow to work.</span></span>

## <a name="add-azure-function-scope-to-test-application-registration"></a><span data-ttu-id="0cfa6-159">Adicionar o escopo de função do Azure para testar o registro do aplicativo</span><span class="sxs-lookup"><span data-stu-id="0cfa6-159">Add Azure Function scope to test application registration</span></span>

1. <span data-ttu-id="0cfa6-160">Retorne ao registro de **aplicativo de teste de função do Azure Graph** e selecione **permissões de API** em **gerenciar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-160">Return to the **Graph Azure Function Test App** registration, and select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="0cfa6-161">Selecione **Adicionar uma permissão**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-161">Select **Add a permission**.</span></span>

1. <span data-ttu-id="0cfa6-162">Selecione **minhas APIs** e, em seguida, selecione **carregar mais**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-162">Select **My APIs** , then select **Load more**.</span></span> <span data-ttu-id="0cfa6-163">Selecione a **função Graph do Azure**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-163">Select **Graph Azure Function**.</span></span>

    ![Uma captura de tela da caixa de diálogo solicitar permissões de API](./images/test-app-add-permissions.png)

1. <span data-ttu-id="0cfa6-165">Selecione a permissão **mail. Read** e, em seguida, selecione **adicionar permissões**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-165">Select the **Mail.Read** permission, then select **Add permissions**.</span></span>

1. <span data-ttu-id="0cfa6-166">Nas **permissões configuradas** , remova a permissão **User. Read** no **Microsoft Graph** selecionando o **...** à direita da permissão e selecionando **remover permissão**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-166">In the **Configured permissions** , remove the **User.Read** permission under **Microsoft Graph** by selecting the **...** to the right of the permission and selecting **Remove permission**.</span></span> <span data-ttu-id="0cfa6-167">Selecione **Sim, remover** para confirmar.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-167">Select **Yes, remove** to confirm.</span></span>

    ![Uma captura de tela das permissões configuradas para o registro do aplicativo de aplicativo de teste](./images/test-app-configured-permissions.png)

## <a name="register-an-app-for-the-azure-function-webhook"></a><span data-ttu-id="0cfa6-169">Registrar um aplicativo para o webhook de função do Azure</span><span class="sxs-lookup"><span data-stu-id="0cfa6-169">Register an app for the Azure Function webhook</span></span>

1. <span data-ttu-id="0cfa6-170">Retorne a **registros de aplicativos** e selecione **novo registro**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-170">Return to **App Registrations** , and select **New registration**.</span></span> <span data-ttu-id="0cfa6-171">Na página **Registrar um aplicativo** , defina os valores da seguinte forma.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-171">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="0cfa6-172">Defina **Nome** para `Graph Azure Function Webhook`.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-172">Set **Name** to `Graph Azure Function Webhook`.</span></span>
    - <span data-ttu-id="0cfa6-173">Defina os **tipos de conta com suporte** para **contas nesse diretório organizacional apenas**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-173">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="0cfa6-174">Deixe **URI de redirecionamento** em branco.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-174">Leave **Redirect URI** blank.</span></span>

1. <span data-ttu-id="0cfa6-175">Selecione **Registrar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-175">Select **Register**.</span></span> <span data-ttu-id="0cfa6-176">Na página **webhook da função de gráfico do Azure** , copie o valor da **ID do aplicativo (cliente)** e salve-o, você precisará dele na próxima etapa.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-176">On the **Graph Azure Function webhook** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

1. <span data-ttu-id="0cfa6-177">Selecione **Certificados e segredos** sob **Gerenciar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-177">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="0cfa6-178">Selecione o botão **Novo segredo do cliente**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-178">Select the **New client secret** button.</span></span> <span data-ttu-id="0cfa6-179">Insira um valor em **Descrição** e selecione uma das opções para **expirar** e selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-179">Enter a value in **Description** and select one of the options for **Expires** and select **Add**.</span></span>

1. <span data-ttu-id="0cfa6-180">Copie o valor secreto do cliente antes de sair desta página.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-180">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="0cfa6-181">Você precisará dele na próxima etapa.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-181">You will need it in the next step.</span></span>

1. <span data-ttu-id="0cfa6-182">Selecione **permissões de API** em **gerenciar**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-182">Select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="0cfa6-183">Escolha **Adicionar uma permissão**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-183">Choose **Add a permission**.</span></span>

1. <span data-ttu-id="0cfa6-184">Selecione **Microsoft Graph** e, em seguida, **permissões de aplicativo**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-184">Select **Microsoft Graph** , then **Application Permissions**.</span></span> <span data-ttu-id="0cfa6-185">Adicione **User. Read. All** e **mail. Read** e, em seguida, selecione **adicionar permissões**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-185">Add **User.Read.All** and **Mail.Read** , then select **Add permissions**.</span></span>

1. <span data-ttu-id="0cfa6-186">Nas **permissões configuradas** , remova a permissão **usuário delegado. Read** no **Microsoft Graph** selecionando o **...** à direita da permissão e selecionando **remover permissão**.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-186">In the **Configured permissions** , remove the delegated **User.Read** permission under **Microsoft Graph** by selecting the **...** to the right of the permission and selecting **Remove permission**.</span></span> <span data-ttu-id="0cfa6-187">Selecione **Sim, remover** para confirmar.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-187">Select **Yes, remove** to confirm.</span></span>

1. <span data-ttu-id="0cfa6-188">Selecione o botão **conceder permissão de administrador para...** e, em seguida, selecione **Sim** para conceder consentimento de administrador para as permissões de aplicativo configuradas.</span><span class="sxs-lookup"><span data-stu-id="0cfa6-188">Select the **Grant admin consent for...** button, then select **Yes** to grant admin consent for the configured application permissions.</span></span> <span data-ttu-id="0cfa6-189">A coluna **status** na tabela de **permissões configurada** é alterada para **concedido para..**..</span><span class="sxs-lookup"><span data-stu-id="0cfa6-189">The **Status** column in the **Configured permissions** table changes to **Granted for ...**.</span></span>

    ![Uma captura de tela das permissões configuradas para o webhook com consentimento de administrador concedido](./images/webhook-configured-permissions.png)
