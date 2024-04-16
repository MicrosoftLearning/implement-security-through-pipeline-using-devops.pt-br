---
lab:
  title: Validar o ambiente de laboratório
  module: 'Module 0: Welcome'
---

# Validar o ambiente de laboratório

Para se preparar para os laboratórios, é crucial ter seu ambiente configurado corretamente. Esta página irá guiar você no processo de configuração, garantindo que todos os pré-requisitos sejam atendidos.

- Os laboratórios requerem o **Microsoft Edge** ou um [navegador com suporte do Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)

- **Configurar uma Assinatura do Azure:** se você ainda não tiver uma assinatura do Azure, crie uma seguindo as instruções nesta página ou visite [https://azure.microsoft.com/free](https://azure.microsoft.com/free) para se inscrever gratuitamente.

- **Configurar uma organização do Azure DevOps:** se você ainda não tiver uma organização do Azure DevOps que possa usar para os laboratório, crie uma seguindo as instruções disponíveis nesta página ou em [Criar uma organização ou coleção de projetos](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).
  
- [Página de download do Git for Windows](https://gitforwindows.org/). Ele será instalado como parte dos pré-requisitos deste laboratório.

- [Visual Studio Code](https://code.visualstudio.com/). Ele será instalado como parte dos pré-requisitos deste laboratório.

- [CLI do Azure](https://learn.microsoft.com/cli/azure/install-azure-cli). Instale a CLI do Azure nos computadores de agente auto-hospedados.

## Instruções para criar uma Organização do DevOps do Azure (você só precisa fazer isso uma vez)

> **Observação**: comece na etapa 3 se você já tiver uma configuração de **Conta Microsoft pessoal** e uma Assinatura do Azure ativa vinculada a essa conta.

1. Use uma sessão privada do navegador para obter uma nova **conta Microsoft (MSA) pessoal** em `https://account.microsoft.com`.

1. Usando a mesma sessão do navegador, inscreva-se para obter uma assinatura gratuita do Azure em `https://azure.microsoft.com/free`.

1. Abra um navegador e navegue até o portal do Azure em `https://portal.azure.com`, em seguida, pesquise na parte superior da tela do portal do Azure por **Azure DevOps**. Na página resultante, clique em **Organizações do Azure DevOps**.

1. Em seguida, clique no link rotulado **Minhas Organizações do Azure DevOps** ou navegue diretamente para `https://aex.dev.azure.com`.

1. Na página **Precisamos de mais alguns detalhes**, selecione **Continuar**.

1. Na caixa suspensa à esquerda, escolha **Diretório padrão** em vez de **Conta Microsoft**.

1. Se solicitado (*"Precisamos de mais alguns detalhes"*), forneça seu nome, endereço de email e local e clique em **Continuar**.

1. De volta a `https://aex.dev.azure.com` com **Diretório padrão** selecionado, clique no botão azul **Criar nova organização**.

1. Aceite os *Termos de Serviço* clicando em **Continuar**.

1. Se solicitado *("Quase concluído")*, deixe o nome da organização do Azure DevOps como padrão (precisa ser um nome globalmente exclusivo) e escolha um local de hospedagem próximo a você na lista.

1. Quando a organização recém-criada for aberta no **Azure DevOps**, selecione **Configurações da organização** no canto inferior esquerdo.

1. Na tela **Configurações da organização**, selecione **Cobrança** (levará alguns segundos para abrir esta tela).

1. Selecione **Configurar cobrança** e, no lado direito da tela, selecione sua **Assinatura do Azure** e selecione **Salvar** para vincular a assinatura à organização.

1. Depois que a tela mostrar a ID de Assinatura do Azure vinculada na parte superior, altere o número de **Trabalhos paralelos pagos** de **CI/CD hospedado pela MS** de 0 para **1**. Depois selecione o botão **SALVAR** na parte inferior.

1. Talvez você **aguarde pelo menos 3 horas antes de usar os recursos de CI/CD** para que as novas configurações sejam refletidas no back-end. Caso contrário, você ainda verá a mensagem *"Nenhum paralelismo hospedado foi comprado ou concedido"*.

## Instruções para criar e configurar o projeto do Azure DevOps (você só precisa fazer isso uma vez)

> **Observação**: certifique-se de concluir as etapas para criar sua Organização do Azure DevOps antes de continuar com essas etapas.

Para seguir todas as instruções do laboratório, você precisará configurar um novo projeto do Azure DevOps, criar um repositório baseado no aplicativo [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) e criar uma conexão de serviço com sua assinatura do Azure.

### Criar o Projeto de Equipe

Primeiro, você criará um projeto **eShopOnWeb** do Azure DevOps para ser usado por vários laboratórios.

1. Abra o navegador e navegue até a organização do Azure DevOps.

1. Selecione a opção **Novo projeto** e use as seguintes configurações:
   - nome: **eShopOnWeb**
   - visibilidade: **Particular**
   - Avançado: Controle de Versão: **Git**
   - Avançado: Processo de Item de Trabalho: **Scrum**

1. Selecione **Criar**.

   ![Criar Projeto](media/create-project.png)

### Importar repositório git eShopOnWeb

Agora, você importará o eShopOnWeb para seu repositório do git.

1. Abra o navegador e navegue até a organização do Azure DevOps.

1. Abra o projeto **eShopOnWeb** criado anteriormente.

1. Selecione os **Repositórios > Arquivos**, **Importar um repositório** e selecione **Importar**.

1. Na janela **Importar um repositório do Git**, cole a seguinte URL `https://github.com/MicrosoftLearning/eShopOnWeb` e selecione **Importar**:

   ![Importar repositório](media/import-repo.png)

1. O repositório está organizado da seguinte forma:

   - A pasta **.ado** contém os pipelines YAML do Azure DevOps.
   - O contêiner da pasta **.devcontainer** está configurado para o desenvolvimento usando contêineres (localmente no VS Code ou no GitHub Codespaces).
   - A pasta **.azure** contém modelos da infraestrutura como código do Bicep e do ARM.
   - A pasta **.github** contém definições de fluxo de trabalho YAML do GitHub.
   - A pasta **src** contém o site do .NET 6 usado nos cenários do laboratório. 

1. Deixe a janela do navegador da Web aberta.  

### Criar uma entidade de serviço e uma conexão de serviço para acessar recursos do Azure

Em seguida, você criará uma entidade de serviço usando a CLI do Azure e uma conexão de serviço no Azure DevOps que permitirá implantar e acessar recursos em sua assinatura do Azure.

1. Inicie um navegador da Web, navegue até o Portal do Azure em `https://portal.azure.com` e entre com a conta de usuário que tem a função proprietário na assinatura do Azure que você usará nos laboratórios deste curso e tenha a função de Administrador Global no locatário do Microsoft Entra associado a essa assinatura.

1. No portal do Azure, selecione o ícone **Cloud Shell** na parte superior da página à direita da caixa de pesquisa.

1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

   > [!NOTE]
   > Se esta for a primeira vez que você estiver iniciando o **Cloud Shell** e receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

1. No prompt **Bash**, no painel **Cloud Shell**, execute os seguintes comandos para recuperar os valores da ID de assinatura do Azure e dos atributos de nome de assinatura:

   ```bash
   subscriptionName=$(az account show --query name --output tsv)
   subscriptionId=$(az account show --query id --output tsv)
   echo $subscriptionName
   echo $subscriptionId
   ```

   > [!NOTE]
   > Copie ambos os valores para um arquivo de texto. Você precisará delas nos laboratórios deste curso.

1. No prompt **Bash**, no painel **Cloud Shell**, execute o seguinte comando para criar uma entidade de serviço:

   ```bash
   az ad sp create-for-rbac --name sp-eshoponweb-azdo --role contributor --scopes /subscriptions/$subscriptionId
   ```

   > [!NOTE]
   > O comando gerará uma saída JSON. Copie a saída no arquivo de texto. Você precisará dele em breve.

   > [!NOTE]
   > Registre o valor, nome da entidade de segurança, sua ID e ID de locatário incluídas na saída JSON. Você precisará delas nos laboratórios deste curso.

1. Volte para a janela do navegador da Web exibindo o portal do Azure DevOps com o projeto **eShopOnWeb** aberto e selecione **Configurações de Projeto** no canto inferior esquerdo do portal.

1. Em Pipelines, selecione **Conexões de serviço** e, em seguida, selecione o botão **Criar conexão de serviço**.

   ![Captura de tela do botão de criação da nova conexão de serviço.](media/new-service-connection.png)

1. Na folha **Nova conexão de serviço**, escolha **Azure Resource Manager** e **Avançar** (talvez seja necessário rolar para baixo).

1. Depois selecione **Entidade de serviço (manual)** e **Avançar**.

1. Preencha os campos vazios usando as informações coletadas durante as etapas anteriores:

   - ID e nome da assinatura.
   - ID da entidade de serviço (ou clientId/AppId), chave da entidade de serviço (ou senha) e TenantId.
   - Em **Nome da conexão de serviço**, digite **azure subs**. Esse nome será referenciado em pipelines YAML para fazer referência à conexão de serviço para acessar sua assinatura do Azure.

   ![Captura de tela da configuração da conexão de serviço do Azure.](media/azure-service-connection.png)

1. Não marque **Conceder permissão de acesso a todos os pipelines**. Selecione **Verificar e salvar**.

   > [!NOTE]
   > A opção **Conceder permissão de acesso a todos os pipelines** não é recomendada para ambientes de produção. Ela só é usada neste laboratório para simplificar a configuração do pipeline.

Agora você concluiu as etapas necessárias para continuar com os laboratórios.
