---
lab:
  title: Configurar uma estrutura repositório e projeto para dar suporte a pipelines seguros
  module: 'Module 1: Configure a project and repository structure to support secure pipelines'
---

# Configurar uma estrutura repositório e projeto para dar suporte a pipelines seguros

Neste laboratório, você vai aprender a configurar uma estrutura de repositório e projeto no Azure DevOps para dar suporte a pipelines seguros. Este laboratório aborda as práticas recomendadas para organizar projetos e repositórios, atribuir permissões e gerenciar arquivos seguros.

Estes exercícios levam aproximadamente **30** minutos.

## Antes de começar

Você precisará de uma assinatura do Azure, da organização do Azure DevOps e do aplicativo eShopOnWeb para acompanhar os laboratórios.

- Siga as etapas para [validar seu ambiente de laboratório](APL2001_M00_Validate_Lab_Environment.md).

## Instruções

### Exercício 1: configurar uma estrutura de projeto segura

Neste exercício, você configurará uma estrutura de projeto segura criando um novo projeto e atribuindo permissões de projeto à estrutura. Separar responsabilidades e recursos em diferentes projetos ou repositórios com permissões específicas oferece suporte à segurança.

#### Tarefa 1: criar uma nova equipe de projeto

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra as **configurações da organização** no canto inferior esquerdo do portal e, em seguida, **Projetos** na seção Geral.

1. Selecione a opção **Novo projeto** e use as seguintes configurações:
   - nome: **eShopSecurity**
   - visibilidade: **Particular**
   - Avançado: Controle de Versão: **Git**
   - Avançado: Processo de Item de Trabalho: **Scrum**

    ![Captura de tela da caixa de diálogo do novo projeto com as configurações especificadas.](media/new-team-project.png)

1. Selecione **Criar** para criar o projeto.

1. Agora é possível alternar entre os diferentes projetos clicando no ícone do Azure DevOps no canto superior esquerdo do portal do Azure DevOps.

    ![Captura de tela dos projetos eShopOnWeb e eShopSecurity da equipe do Azure DevOps.](media/azure-devops-projects.png)

É possível gerenciar permissões e configurações para cada projeto separadamente acessando o menu Configurações do projeto e selecionando o projeto de equipe apropriado. Se você tiver vários usuários ou equipes trabalhando em projetos diferentes, também poderá atribuir permissões a cada projeto separadamente.

#### Tarefa 2: criar um novo repositório e atribuir permissões de projeto

1. Selecione o nome da organização no canto superior esquerdo do portal Azure DevOps e selecione o novo projeto **eShopSecurity**.

1. Selecione o menu **Repos**.

1. Selecione o botão **Inicializar** para inicializar o novo repositório adicionando o arquivo README.md.

1. Abra o menu **Configurações do projeto** no canto inferior esquerdo do portal e selecione **Repositórios** na seção Repos.

1. Selecione o novo repositório **eShopSecurity** e selecione a guia **Segurança**.

1. Remova as permissões Herdar do pai desmarcando o botão de alternância **Herança**.

1. Selecione o grupo **Colaboradores** e selecione o menu suspenso **Negar** para todas as permissões, exceto **Leitura**. Isso impedirá que todos os usuários do grupo de Colaboradores acessem o repositório.

1. Selecione seu usuário em Usuários e selecione o botão **Permitir** para permitir todas as permissões.

    ![Captura de tela das configurações de segurança do repositório com permitir para leitura e negar para todas as outras permissões.](media/repository-security.png)

1. (Opcional) Adicione um grupo específico de usuários ou usuário que você deseja conceder acesso ao repositório e execute pipelines a partir do projeto eShopOnWeb. Clique na caixa de pesquisa, digite o nome do grupo, selecione-o e defina as permissões que deseja permitir ou negar para o grupo ou usuário.

    > [!NOTE]
    > Certifique-se de ter o mesmo grupo em seu projeto eShopOnWeb. Isso permitirá que você execute pipelines do projeto eShopOnWeb e acesse o repositório no projeto eShopSecurity.

1. As alterações vão ser salvas automaticamente.

Agora, apenas o usuário que você atribuiu permissões e os administradores podem acessar o repositório. Isso é útil quando você deseja permitir que usuários específicos acessem o repositório e executem pipelines a partir do projeto eShopOnWeb.

### Exercício 2: configurar uma estrutura de pipeline e modelo para oferecer suporte a pipelines seguros

#### Tarefa 1: (se concluído, ignorar) importar e executar o pipeline de CI

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb** no Azure DevOps.

1. Acesse **Pipelines > Pipelines**.

1. Selecione o botão **Criar pipeline**.

1. Selecione **Git do Azure Repos (Yaml)**.

1. Selecione o repositório **eShopOnWeb**.

1. Selecione **Arquivo YAML existente do Azure Pipelines**.

1. Selecione o arquivo **/.ado/eshoponweb-ci.yml** e selecione **Continuar**.

1. Clique no botão **Executar** para executar o pipeline.

1. Seu pipeline assumirá um nome com base no nome do projeto. Renomeie-o para identificar melhor o pipeline.

1. Vá para **Pipelines > Pipelines** e selecione o pipeline criado recentemente. Selecione as reticências e, em seguida, selecione a opção **Renomear/mover**.

1. Nomeie-o **eshoponweb-ci** e selecione **Salvar**.

#### Tarefa 2: criar uma entidade de serviço e uma conexão de serviço para acessar os recursos do Azure

Nesta tarefa, você criará uma entidade de serviço usando a CLI do Azure e a Conexão de Serviço no Azure DevOps, o que permitirá implantar recursos em sua assinatura do Azure.

1. Abra o navegador da Web, navegue até o portal do Azure em `https://portal.azure.com` e entre com a conta de usuário que tenha a função de Proprietário na assinatura do Azure que você usará neste laboratório e a função de Administrador Global no locatário do Azure AD associada a essa assinatura.

1. No portal do Azure, selecione o ícone **Cloud Shell** na parte superior da página à direita da caixa de pesquisa.

1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**.

   > [!NOTE]
   > Se esta for a primeira vez que você estiver iniciando o **Cloud Shell** e receber a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que você está usando no laboratório e selecione **Criar armazenamento**.

1. No prompt **Bash**, no painel **Cloud Shell**, execute os seguintes comandos para recuperar os valores da ID de assinatura do Azure e dos atributos de nome de assinatura:

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > [!NOTE]
    > Copie ambos os valores para um arquivo de texto. Você precisará deles mais adiante neste laboratório.

1. No prompt **Bash**, no painel **Cloud Shell**, execute o seguinte comando para criar uma entidade de serviço:

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > [!NOTE]
    > Substitua **myServicePrincipalName** por qualquer cadeia de caracteres exclusiva que consista em letras e dígitos, por exemplo, **AzureDevOpsSP** e **mySubscriptionID** por sua subscriptionId do Azure.

    > [!NOTE]
    > O comando gerará uma saída JSON. Copie a saída no arquivo de texto. Você precisará disso em uma etapa posterior deste laboratório.

1. Em seguida, navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb** e selecione **Configurações do projeto** no canto inferior esquerdo do portal.

1. Em Pipelines, selecione **Conexões de serviço** e, em seguida, selecione o botão **Criar conexão de serviço**.

    ![Captura de tela do botão de criação da nova conexão de serviço.](media/new-service-connection.png)

1. Na folha **Nova conexão de serviço**, escolha **Azure Resource Manager** e **Avançar** (talvez seja necessário rolar para baixo).

1. Depois selecione **Entidade de serviço (manual)** e **Avançar**.

1. Preencha os campos vazios usando as informações coletadas durante as etapas anteriores:
    - ID e nome da assinatura.
    - ID da entidade de serviço (ou clientId/AppId), chave da entidade de serviço (ou senha) e TenantId.
    - Em **Nome da conexão de serviço**, digite **azure subs**. Esse nome será referenciado em pipelines YAML quando precisar de uma Conexão de Serviço do Azure DevOps para se comunicar com sua assinatura do Azure.

        ![Captura de tela da configuração da conexão de serviço do Azure.](media/azure-service-connection.png)

1. Não marque **Conceder permissão de acesso a todos os pipelines**. Selecione **Verificar e salvar**.

    > [!NOTE]
    > A opção **Conceder permissão de acesso a todos os pipelines** não é recomendada para ambientes de produção. Ela só é usada neste laboratório para simplificar a configuração do pipeline.

#### Tarefa 3: (se concluída, pular) importar e executar o pipeline de CD

1. Acesse **Pipelines > Pipelines**.

1. Selecione o botão **Novo pipeline**.

1. Selecione **Git do Azure Repos (Yaml)**.

1. Selecione o repositório **eShopOnWeb**.

1. Selecione **Arquivo YAML existente do Azure Pipelines**.

1. Selecione o arquivo **/.ado/eshoponweb-cd-webapp-code.yml** e selecione **Continuar**.

1. Na definição de pipeline YAML na seção variáveis, personalize:
   - **AZ400-EWebShop-NAME** com o nome de sua preferência, por exemplo, **rg-eshoponweb-secure**.
   - **Local** com o nome da região do Azure que você deseja implantar seus recursos, por exemplo, **southcentralus**.
   - **YOUR-SUBSCRIPTION-ID** com sua ID de assinatura do Azure.
   - **az400eshop-NAME**, com um nome de aplicativo Web a ser implantado com um nome exclusivo global, por exemplo, **eshoponweb-lab-secure**.

1. Selecione **Salvar e executar** para confirmar diretamente na ramificação principal ou crie uma ramificação para essa confirmação.

1. Selecione **Salvar e executar** novamente.

    > [!NOTE]
    > Se você optar por criar uma nova ramificação, será necessário criar uma solicitação de pull para mesclar as alterações na ramificação principal.

1. Abra o pipeline. Se você vir a mensagem "Este pipeline precisa de permissão para acessar um recurso antes que essa execução possa continuar a Implantar no WebApp", selecione **Exibir**, **Permitir** e **Permitir** novamente. Isso é necessário para permitir que o pipeline crie o recurso do Serviço de Aplicativo do Azure.

    ![Captura de tela do acesso de permissão do pipeline YAML.](media/pipeline-deploy-permit-resource.png)

1. A implantação pode levar alguns minutos para ser concluída, aguarde a execução do pipeline. A definição de CD consiste nas seguintes tarefas:
      - **Recursos**: está preparado para acionar automaticamente com base na conclusão do pipeline de CI. Ele também baixa o repositório para o arquivo do Bicep.
      - **AzureResourceManagerTemplateDeployment**: implanta o Aplicativo Web do Azure usando o modelo do Bicep.
1. Seu pipeline assumirá um nome com base no nome do projeto. Vamos renomeá-lo para identificar melhor o pipeline.

1. Vá para **Pipelines > Pipelines** e selecione o pipeline criado recentemente. Selecione as reticências e, em seguida, selecione a opção **Renomear/mover**.

1. Nomeie-o **eshoponweb-cd-webapp-code** e selecione **Salvar**.

Agora você deve ter dois pipelines em execução em seu projeto eShopOnWeb.

![Captura de tela dos pipelines de CI/CD executados com êxito.](media/pipeline-successful-executed.png)

#### Tarefa 4: mover as variáveis de pipeline de CD para um modelo YAML

Nesta tarefa, você criará um modelo YAML para armazenar as variáveis usadas no pipeline de CD. Isso permitirá que você reutilize o modelo em outros pipelines.

1. Vá para **Repos** e, em seguida, **Arquivos**.

1. Expanda a pasta **.ado** e selecione **Novo arquivo**.

1. Nomeie o arquivo **eshoponweb-secure-variables.yml** e selecione **Criar**.

1. Adicione a seção de variáveis usada no pipeline do CD ao novo arquivo. O arquivo deverá ter a seguinte aparência:

    ```YAML
    variables:
      resource-group: 'rg-eshoponweb-secure'
      location: 'southcentralus' #name of the Azure region you want to deploy your resources
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: 'YOUR-SUBSCRIPTION-ID'
      azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
      webappname: 'eshoponweb-lab-secure'

    ```

    > [!IMPORTANT]
    > Substitua os valores das variáveis pelos valores do seu ambiente (grupo de recursos, local, ID da assinatura, conexão de serviço do Azure e nome do aplicativo Web).

1. Selecione **Confirmar**, adicione um comentário e, em seguida, selecione o botão **Confirmar**.

1. Abra a definição de pipeline **eshoponweb-cd-webapp-code.yml** e substitua a seção de variáveis pelo seguinte:

    ```YAML
    variables:
      - template: eshoponweb-secure-variables.yml
    ```

    > [!NOTE]
    > Se você estiver usando um caminho diferente para o arquivo de modelo, será necessário atualizar o caminho na definição de pipeline.

1. Selecione **Salvar** e **Executar** o pipeline novamente.

Agora você tem um modelo YAML com as variáveis usadas no pipeline de CD. Você pode reutilizar esse modelo em outros pipelines em cenários em que você precisa implantar os mesmos recursos. Além disso, sua equipe de operações pode controlar o grupo de recursos e o local onde os recursos são implantados e outras informações em seus valores de modelo, e você não precisa fazer alterações na definição de pipeline.

#### Tarefa 5: mover os modelos YAML para um repositório e projeto separados

Nesta tarefa, você moverá os modelos YAML para um repositório e projeto separados.

1. No seu projeto eShopSecurity, vá para **Repos > Arquivos**.

1. Crie um novo arquivo chamado **eshoponweb-secure-variables.yml**.

1. Copie o conteúdo do arquivo **.ado/eshoponweb-secure-variables.yml** do repositório eShopOnWeb para o novo arquivo.

1. Confirme as alterações.

1. Abra a definição de pipeline **eshoponweb-cd-webapp-code.yml** a partir do projeto eShopOnWeb.

1. Adicione o seguinte à seção de recursos:

    ```YAML
    resources:
      repositories:
        - repository: eShopSecurity
          type: git
          name: eShopSecurity/eShopSecurity #name of the project and repository

    ```

1. Substitua a seção variáveis pelo conteúdo a seguir:

    ```YAML
    variables:
      - template: eshoponweb-secure-variables.yml@eShopSecurity #name of the template and repository
    ```

    ![Captura de tela da definição do pipeline com as novas variáveis e seções de recursos.](media/pipeline-variables-resource-section.png)

1. Selecione **Salvar** e **Executar** o pipeline novamente. Você verá que o pipeline está usando o modelo YAML do repositório eShopSecurity.

    ![Captura de tela da execução do pipeline usando o modelo YAML do repositório eShopSecurity.](media/pipeline-execution-using-template.png)

Agora você tem os modelos YAML em um repositório e projeto separados. Você pode reutilizar esses modelos em outros pipelines em cenários em que você precisa implantar os mesmos recursos. Além disso, sua equipe de operações pode controlar o grupo de recursos, o local, a segurança e onde os recursos são implantados e outras informações em seus valores de modelo, e você não precisa fazer alterações na definição de pipeline.

### Exercício 2: executar a limpeza dos recursos do Azure e do Azure DevOps

Neste exercício, você removerá os recursos do Azure e do Azure DevOps criados neste laboratório.

#### Tarefa 1: remover recursos do Azure

1. No portal do Azure, abra o grupo de recursos criado e selecione **Excluir grupo de recursos** para todos os recursos criados neste laboratório.

    ![Captura de tela do botão excluir grupo de recursos.](media/delete-resource-group.png)

    > [!WARNING]
    > Lembre-se sempre de remover todos os recursos do Azure que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

#### Tarefa 2: remover pipelines do Azure DevOps

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb**.

1. Acesse **Pipelines > Pipelines**.

1. Vá para **Pipelines > Pipelines** e remova os pipelines existentes.

## Revisão

Neste laboratório, você aprendeu a configurar e organizar uma estrutura segura de repositório e projeto no Azure DevOps. Ao gerenciar permissões de forma eficaz, você pode garantir que os usuários certos tenham acesso aos recursos de que precisam e, ao mesmo tempo, manter a segurança e a integridade de seus pipelines e processos de DevOps.
