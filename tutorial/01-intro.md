---
ms.openlocfilehash: 1e4267a5d8bdfe02dbc0170122113c35df822659
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822540"
---
<!-- markdownlint-disable MD002 MD041 -->

Este tutorial ensina como criar uma função do Azure que usa a API do Microsoft Graph para recuperar informações de calendário de um usuário.

> [!TIP]
> Se preferir baixar o tutorial concluído, você poderá baixar ou clonar o repositório do [GitHub](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp). Consulte o arquivo LEIAme na pasta **demonstração** para obter instruções sobre como configurar o aplicativo com uma ID de aplicativo e segredo.

## <a name="prerequisites"></a>Pré-requisitos

Antes de iniciar este tutorial, você deve ter as seguintes ferramentas instaladas em sua máquina de desenvolvimento.

- [SDK do .NET Core](https://dotnet.microsoft.com/download)
- [Ferramentas principais de funções do Azure](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [CLI do Azure](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [ngrok](https://ngrok.com/)

Você também deve ter uma conta corporativa ou de estudante da Microsoft, com acesso a uma conta de administrador global na mesma organização. Se você não tiver uma conta da Microsoft, poderá [se inscrever no programa para desenvolvedores do office 365](https://developer.microsoft.com/office/dev-program) para obter uma assinatura gratuita do Office 365.

> [!NOTE]
> Este tutorial foi escrito com as seguintes versões das ferramentas acima. As etapas deste guia podem funcionar com outras versões, mas que não foram testadas.
>
> - SDK do .NET Core 3.1.301
> - Ferramentas do Azure funções principais 3.0.2630
> - 2.8.0 da CLI do Azure
> - ngrok 2.3.35

## <a name="feedback"></a>Comentários

Forneça comentários sobre este tutorial no [repositório do GitHub](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).
