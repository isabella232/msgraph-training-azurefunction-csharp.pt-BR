---
ms.openlocfilehash: fbd43b4708cd1aeaad93b70fcf151869bd9e543e
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822547"
---
<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você aprenderá sobre as alterações necessárias para a função do Azure de amostra preparar a [publicação para um aplicativo Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).

## <a name="update-code"></a>Código de atualização

A configuração é lida do repositório de segredo do usuário, que só se aplica à sua máquina de desenvolvimento. Antes de publicar no Azure, você precisará alterar onde armazenou sua configuração e atualizar o código no **Startup.cs** de acordo.

Os segredos do aplicativo devem ser armazenados no armazenamento seguro, como o [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).

## <a name="update-cors-setting-for-azure-function"></a>Atualizar a configuração CORS para a função do Azure

Neste exemplo, configuramos o CORS no **local.settings.jsativado** para permitir que o aplicativo de teste chame a função. Você precisará configurar sua função publicada para permitir qualquer aplicativo de SPA que o chamará.

## <a name="update-app-registrations"></a>Atualizar registros de aplicativo

A  `knownClientApplications` propriedade no manifesto do registro de aplicativo de **função do Azure para Graph** precisará ser atualizada com as identificações de aplicativos de qualquer aplicativo que chamará a função do Azure.

## <a name="recreate-existing-subscriptions"></a>Recriar assinaturas existentes

Todas as assinaturas criadas usando a URL de webhook na sua máquina local ou ngrok devem ser recriadas usando a URL de produção da `Notify` função do Azure.
