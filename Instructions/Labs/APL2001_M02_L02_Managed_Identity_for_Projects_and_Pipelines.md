---
lab:
  title: Identidade gerenciada para projetos e pipelines
  module: 'Module 2: Manage identity for projects, pipelines, and agents'
---

# Identidade gerenciada para projetos e pipelines

As identidades gerenciadas oferecem um método seguro para controlar o acesso aos recursos do Azure. O Azure manipula essas identidades automaticamente, permitindo que você verifique o acesso a serviços compatíveis com a autenticação do Azure AD. Isso significa que você não precisa incorporar credenciais ao seu código, o que aumenta a segurança. No Azure DevOps, as identidades gerenciadas podem autenticar recursos do Azure em seus agentes auto-hospedados, simplificando o controle de acesso sem comprometer a segurança.

Neste laboratório, você criará uma identidade gerenciada para usar em seus pipelines YAML usando o Azure DevOps com agentes auto-hospedados e uma identidade gerenciada.

Este exercício levará aproximadamente **45** minutos.

## Antes de começar

Você precisará de uma assinatura do Azure, da organização do Azure DevOps e do aplicativo eShopOnWeb para acompanhar os laboratórios.

- Siga as etapas para [validar seu ambiente de laboratório](APL2001_M00_Validate_Lab_Environment.md).

- Verifique se você tem uma conta Microsoft ou uma conta do Azure AD com função de Proprietário ou Colaborador na assinatura do Azure. Para obter detalhes, veja [Listar atribuições de função do Azure usando o portal do Azure](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e atribuir funções de administrador no Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Instruções

### Exercício 1: Importar e executar pipelines de CI/CD

Neste exercício, você importará e executará o pipeline de CI, configurará a conexão de serviço com sua Assinatura do Azure e, em seguida, importará e executará o pipeline de CD.

#### Tarefa 1: (se concluído, ignorar) importar e executar o pipeline de CI

Vamos começar importando o pipeline de CI chamado [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb**.

1. Acesse **Pipelines > Pipelines**.

1. Selecione o botão **Novo pipeline** .

1. Selecione **Git do Azure Repos (Yaml)**.

1. Selecione o repositório **eShopOnWeb**.

1. Selecione **Arquivo YAML existente do Azure Pipelines**.

1. Selecione o arquivo **/.ado/eshoponweb-ci.yml** e clique em **Continuar**.

1. Clique no botão **Executar** para executar o pipeline.

    > [!NOTE]
    > Seu pipeline assumirá um nome com base no nome do projeto. Renomeie-o para identificar melhor o pipeline.

1. Vá para **Pipelines > Pipelines**, selecione o pipeline criado recentemente, selecione as reticências e selecione a opção **Renomear/mover**.

1. Nomeie-o **eshoponweb-ci** e selecione **Salvar**.

#### Exercício 2: gerenciar a conexão de serviço.

Você pode criar uma conexão do Azure Pipelines com serviços externos e remotos para executar tarefas em um trabalho.

Nesta tarefa, você criará uma entidade de serviço usando a CLI do Azure, que permitirá ao Azure DevOps:

- Implantar recursos na sua assinatura do Azure
- Implantar o aplicativo eShopOnWeb

> [!NOTE]
> Se você já tiver uma entidade de serviço e uma conexão de serviço com sua assinatura do Azure chamada **azure subs**, prossiga diretamente para a próxima tarefa.

Você precisará de uma entidade de serviço para implantar recursos do Azure a partir do Azure Pipelines.

Uma entidade de serviço é criada automaticamente pelo Azure Pipeline quando você se conecta a uma assinatura do Azure de dentro de uma definição de pipeline ou quando cria uma nova conexão de serviço na página de configurações do projeto (opção automática). Você também pode criar manualmente a entidade de serviço a partir do portal ou usando a CLI do Azure e reutilizá-la pelos projetos.

1. Abra o navegador da Web, navegue até o portal do Azure em `https://portal.azure.com` e entre com a conta de usuário que tenha a função de Proprietário na assinatura do Azure que você usará neste laboratório e a função de Administrador Global no locatário do Azure AD associada a essa assinatura.

1. No portal do Azure, clique no ícone do **Cloud Shell** localizado diretamente à direita da caixa de pesquisa na parte superior da página.

1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

   > [!NOTE]
   > Se esta for a primeira vez que você estiver iniciando o **Cloud Shell** e receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

1. No prompt do **Bash**, no painel **Cloud Shell**, execute os seguintes comandos para recuperar os valores do atributo ID de assinatura do Azure:

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > [!NOTE]
    > Copie ambos os valores para um arquivo de texto. Você precisará deles mais adiante neste laboratório.

1. No prompt do **Bash**, no painel **Cloud Shell**, execute o seguinte comando para criar uma entidade de serviço:

    ```sh
    az ad sp create-for-rbac --name sp-eshoponweb-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > [!NOTE]
    > O comando gerará uma saída JSON. Copie a saída no arquivo de texto. Você precisará disso em uma etapa posterior deste laboratório.

1. Em seguida, navegue até o projeto **eShopOnWeb** do Azure DevOps. Clique em **Configurações do Projeto > Conexões de Serviço (em Pipelines)** e **Nova Conexão de Serviço**.

1. Na folha **Nova conexão de serviço**, escolha **Azure Resource Manager** e **Avançar** (talvez seja necessário rolar para baixo).

1. Selecione **Entidade de serviço (manual)** e, em seguida, **Avançar**.

1. Preencha os campos vazios usando as informações coletadas durante as etapas anteriores:
    - ID e nome da assinatura.
    - ID da entidade de serviço (ou clientId), chave (ou senha) e TenantId.
    - Em **Nome da conexão de serviço**, digite **azure subs**. Esse nome será referenciado em pipelines YAML quando precisar de uma Conexão de Serviço do Azure DevOps para se comunicar com sua assinatura do Azure.

1. Selecione **Verificar e salvar**.

#### Exercício 3: importar e executar o pipeline de CD.

Agora, importe o pipeline de CD chamado [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Acesse **Pipelines > Pipelines**.

1. Clique no botão **Novo pipeline**.

1. Selecione **Git do Azure Repos (Yaml)**.

1. Selecione o repositório **eShopOnWeb**.

1. Selecione **Arquivo YAML existente do Azure Pipelines**.

1. Selecione o arquivo **/.ado/eshoponweb-cd-webapp-code.yml** e selecione **Continuar**.

1. Na definição do pipeline YAML, defina a seção de variáveis como:

    ```YAML
    variables:
      resource-group: 'AZ400-EWebShop-NAME'
      location: 'westeurope'
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: 'YOUR-SUBSCRIPTION-ID'
      azureserviceconnection: 'azure subs'
      webappname: 'az400-webapp-NAME'
    ```

1. Na seção de variáveis, substitua os espaços reservados pelos valores a seguir:

   - **AZ400-EWebShop-NAME** com o nome de sua preferência, por exemplo, **rg-eshoponweb**.
   - **local** com o nome da região do Azure que deseja implantar seus recursos, por exemplo, **southcentralus**.
   - **YOUR-SUBSCRIPTION-ID** com sua ID de assinatura do Azure.
   - **az400-webapp-NAME**, com um nome do aplicativo Web a ser implantado com um nome exclusivo global, por exemplo, **eshoponweb-lab-YOURNAME.**.

1. Se houver, na seção recursos, remova as seguintes entradas:

    ```YAML
    repositories:
      - repository: eShopSecurity
        type: git
        name: eShopSecurity/eShopSecurity #name of the project and repository
    ```

1. Selecione **Salvar e executar** para confirmar diretamente na ramificação principal ou crie uma ramificação para essa confirmação.

1. Selecione **Salvar e executar** novamente.

    > [!NOTE]
    > Se você optar por criar uma nova ramificação, será necessário criar uma solicitação de pull para mesclar as alterações na ramificação principal.

1. Abra o pipeline. Se você vir a mensagem "Este pipeline precisa de permissão para acessar um recurso antes que essa execução possa continuar a Implantar no WebApp", selecione **Exibir**, **Permitir** e **Permitir** novamente. Isso é necessário para permitir que o pipeline crie o recurso do Serviço de Aplicativo do Azure.

    ![Captura de tela do acesso de permissão do pipeline YAML.](media/pipeline-deploy-permit-resource.png)

1. A implantação pode levar alguns minutos para ser concluída, aguarde a execução do pipeline. A definição de CD consiste nas seguintes tarefas:
      - **Recursos**: está preparado para acionar automaticamente com base na conclusão do pipeline de CI. Ele também baixa o repositório para o arquivo do Bicep.
      - **AzureResourceManagerTemplateDeployment**: implanta o Aplicativo Web do Azure usando o modelo do Bicep.

1. Seu pipeline assumirá um nome com base no nome do projeto. Vamos **renomeá-lo** para identificar melhor o pipeline.

1. Vá para **Pipelines > Pipelines**, selecione o pipeline criado recentemente, selecione as reticências e selecione a opção **Renomear/mover**.

1. Nomeie-o **eshoponweb-cd-webapp-code** e selecione **Salvar**.

### Exercício 2: criar uma identidade gerenciada para a conexão de serviço

Neste exercício, você criará uma identidade gerenciada e, em seguida, criará uma nova conexão de serviço para usá-la nos pipelines de CI/CD.

#### Tarefa 1: criar uma identidade gerenciada

1. No seu navegador, abra o portal do Azure em `https://portal.azure.com`.

1. Na caixa **Pesquisar recursos, serviços e documentos (G+/)**, digite **Identidades gerenciadas** e selecione-o na lista suspensa.

    ![Captura de tela da opção Identidades gerenciadas no portal do Azure.](media/managed-identities.png)

1. Selecione o botão **Criar identidade gerenciada**.

1. No painel **Criar Identidade Gerenciada**, preencha as informações necessárias:
   - **Assinatura** com sua assinatura do Azure.
   - **Grupo de recursos** com um grupo de recursos novo ou já existente.
   - **Região** com a região próxima à sua localização ou disponível para seus recursos.
   - **Nome** com o nome da identidade gerenciada de sua preferência, por exemplo, **eshoponweb-mi**.

    ![Captura de tela do painel Criar identidade gerenciada.](media/create-managed-identity.png)

    > [!NOTE]
    > Se você não tiver um grupo de recursos, poderá criar um clicando no link **Criar novo**.

1. Selecione **Revisar + criar**, em seguida, **Criar**.

#### Tarefa 2: atribuir permissões à identidade gerenciada

Em seguida, você precisa atribuir as permissões da identidade gerenciada ao grupo de recursos e aos Serviços de Aplicativos.

1. No portal do Azure, navegue até a nova identidade gerenciada criada.

1. Selecione a guia **Atribuições de função do Azure** no menu lateral.

1. Selecione o botão **Adicionar atribuição de função** e execute as seguintes ações:

    | Configuração | Ação |
    | -- | -- |
    | Lista suspensa **Escopo**. | Selecione **Grupo de Recursos**. |
    | Lista suspensa **Assinatura** | Selecione sua assinatura do Azure. |
    | Lista suspensa **Grupo de recursos** | Selecione o grupo de recursos já existente. |
    | Lista suspensa **Função** | Selecione a função **Colaborador**. |

1. Selecione o botão **Salvar**.

    ![Captura de tela do painel Adicionar Atribuição](media/add-role-assignment.png)

### Exercício 3: criar uma nova máquina virtual do Azure usando o agente auto-hospedado e a identidade gerenciada e atualizar o pipeline de CI

Neste exercício, você criará uma nova máquina virtual do Azure usando o agente auto-hospedado e a identidade gerenciada criada no exercício anterior. Em seguida, você atualizará o pipeline de CI para usar a nova máquina virtual do Azure.

#### Tarefa 1: criar uma nova máquina virtual do Azure

1. No seu navegador, abra o portal do Azure em `https://portal.azure.com`.

1. Na caixa **Pesquisar recursos, serviços e documentos (G+/),** digite **Máquinas virtuais** e selecione na lista suspensa.

1. Selecione o botão **Criar**.

1. Selecione **Máquina virtual do Azure com uma configuração predefinida**.

    ![Captura de tela da criação de uma máquina virtual com uma configuração predefinida.](media/create-virtual-machine-preset.png)

1. Selecione **Desenvolvimento/Teste** como o ambiente de carga de trabalho e **Uso Geral** como o tipo de carga de trabalho.

1. Selecione o botão **Continuar criando uma VM**. Na guia **Básico**, execute as seguintes ações e selecione a guia **Gerenciamento**:

    | Configuração | Ação |
    | -- | -- |
    | Lista suspensa **Assinatura** | Selecione sua assinatura do Azure. |
    | Seção **Grupo de recursos** | Selecione o grupo de recursos já existente ou novo, por exemplo, **eshoponweb-resource**. |
    | Caixa de texto **Nome da máquina virtual**  | Insira o nome de sua preferência, por exemplo, **eshoponweb-vm**. |
    | Lista suspensa **Região** | Selecione a região próxima à sua localização ou disponível para seus recursos, por exemplo, **Centro-Sul dos EUA**. |
    | Lista suspensa **Opções de disponibilidade** | Selecione **Nenhuma redundância de infraestrutura necessária**. |
    | Lista suspensa **Tipos de segurança** | Selecione a opção **Máquinas virtuais de inicio confiável**. |
    | Lista suspensa **Imagem** | Selecione a imagem **Windows Server 2019 ou 2022 Datacenter**. |
    | Lista suspensa **Tamanho** | Selecione o tamanho**Padrão** mais econômico para fins de teste. |
    | Caixa de texto **Nome de usuário** | Digite o nome de usuário de sua preferência |
    | Caixa de texto **Senha** | Digite a senha de sua preferência |
    | Seção **Portas de entrada públicas** | Selecione **Permitir portas selecionadas**. |
    | Lista suspensa **Selecionar portas de entrada** | Selecione **RDP (3389)**. |

1. Na guia **Gerenciamento**, execute as seguintes ações e selecione **Examinar + criar**:
   
    | **Habilitar a seção identidade gerenciada atribuída pelo sistema** | marque a **caixa de seleção**. Isso permitirá que a VM use a identidade gerenciada que você criou. | | Seção **Endereço IP público** | Selecione **Criar novo**, insira um nome de sua preferência e selecione **Ok** |

    > [!IMPORTANT]
    > Não pule a etapa Exercício 5: remover os recursos de laboratório do Azure para evitar cobranças inesperadas.

1. Na guia **Revisar + criar**, selecione **Criar**.

1. Abra as configurações da máquina virtual, selecione a guia **Identidade** e clique no botão **Atribuições de função do Azure**.

1. Clique no botão **Adicionar atribuição de função**.

1. Selecione o escopo da assinatura, a assinatura e a função **Colaborador** .

1. Selecione o botão **Salvar**.

#### Tarefa 2: abrir a nova Máquina Virtual do Azure e instalar o agente auto-hospedado

1. Abra a nova Máquina Virtual do Azure que você criou anteriormente usando a conexão RDP. Você pode encontrar as informações de conexão na **Visão geral** verificando o botão **Conectar** .

2. Na VM do Azure, siga as etapas para instalar o agente na nova Máquina Virtual do Azure no [Exercício 1 do laboratório Configurar agentes e pools de agentes para pipelines seguros](APL2001_M03_L03_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md). Ao seguir as instruções, leve em conta as seguintes alterações:

   - Nomeie o pool de agentes como **eShopOnWebSelfPoolManaged** (em vez de **eShopOnWebSelfPool**) na etapa 5 da Tarefa 1.
   - Nomeie o agente como **eShopOnWebSelfAgentManaged** (em vez de **eShopOnWebSelfAgent**) na etapa 3 da Tarefa 4.
   - Selecione **NT AUTHORITY\NETWORK SERVICE** como a conta para executar o serviço durante a configuração da conta de usuário na etapa 3 da Tarefa 4.

3. Após a instalação do agente, abra seu pool de agentes no portal do Azure DevOps e veja que o novo agente está disponível.

    ![Captura de tela do novo agente no novo pool de agentes.](media/new-agent-pool.png)

### Exercício 4: criar uma nova conexão de serviço usando a identidade gerenciada e atualizar o pipeline de CD

Neste exercício, você criará uma nova conexão de serviço usando o método de autenticação de identidade gerenciada. Em seguida, você atualizará o pipeline de CD para usar a nova conexão de serviço.

#### Tarefa 1: criar uma nova conexão de serviço

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb** e navegue até **Configurações do projeto > Conexões de serviço**.

1. Selecione o botão **Nova conexão de serviço** e selecione **Azure Resource Manager**.

1. Selecione **Identidade Gerenciada** como o **Método de autenticação**.

1. Preencha os campos vazios usando as informações coletadas durante as etapas anteriores:
    - ID de assinatura, nome e ID do locatário (ou clientId).
    - Em **Nome da conexão do serviço**, digite **subs do Azure gerenciados**. Esse nome será referenciado em pipelines YAML quando precisar de uma Conexão de Serviço do Azure DevOps para se comunicar com sua assinatura do Azure.

1. Selecione **Verificar** e **Salvar**.

#### Tarefa 2: atualizar o pipeline de CD

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb** e navegue até **Pipelines > Pipelines**.

1. Selecione o pipeline **eshoponweb-cd-webapp-code** e selecione **Editar**.

1. Na seção de variáveis, atualize a variável **serviceConnection** com o nome da conexão de serviço que você criou na tarefa anterior, **subs do azure gerenciados**.

    ```YAML
          azureserviceconnection: 'azure subs managed'
    ```

1. Na subseção **trabalhos** da seção **estágios**, atualize o valor da propriedade **pool** para fazer referência ao pool de agentes auto-hospedados que você criou no exercício anterior, **eShopOnWebSelfPoolManaged**, de forma que ele fique no seguinte formato:

    ```YAML    
          jobs:
          - job: Deploy
            pool: eShopOnWebSelfPoolManaged
            steps:
            #download artifacts
            - download: eshoponweb-ci
    ```

1. Selecione **Salvar**, escolha confirmar diretamente na ramificação principal ou crie uma nova ramificação.

1. Selecione **Salvar** novamente.

    > [!NOTE]
    > Se você optar por criar uma nova ramificação, será necessário criar uma solicitação de pull para mesclar as alterações na ramificação principal.

1. Selecione **Executar** o pipeline e clique em **Executar** novamente.

1. Abra o pipeline. Se você vir a mensagem "Este pipeline precisa de permissão para acessar um recurso antes que essa execução possa continuar a Implantar no WebApp", selecione **Exibir**, **Permitir** e **Permitir** novamente. Isso é necessário para permitir que o pipeline crie o recurso do Serviço de Aplicativo do Azure.

1. A implantação pode levar alguns minutos para ser concluída, aguarde a execução do pipeline.

1. Você verá nos logs de pipeline que o pipeline está usando a identidade gerenciada.

    ![Captura de tela dos logs de pipeline usando a identidade gerenciada.](media/pipeline-logs-managed-identity.png)

Depois que o pipeline terminar, você poderá acessar o portal do Azure e conferir o novo recurso do Serviço de Aplicativo.

### Exercício 5: remover os recursos de laboratório do Azure

1. No portal do Azure, abra o grupo de recursos criado e selecione **Excluir grupo de recursos** para todos os recursos criados neste laboratório.

    ![Captura de tela do botão excluir grupo de recursos.](media/delete-resource-group.png)

    > [!WARNING]
    > Lembre-se sempre de remover todos os recursos do Azure que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

## Revisão

Neste laboratório, você aprendeu a habilitar dinamicamente a configuração e gerenciar sinalizadores de recursos.
