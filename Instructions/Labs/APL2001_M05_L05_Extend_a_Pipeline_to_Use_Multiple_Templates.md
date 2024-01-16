---
lab:
  title: Estender um pipeline para usar vários modelos
  module: 'Module 5: Extend a pipeline to use multiple templates'
---

# Estender um pipeline para usar vários modelos

Neste laboratório, explore a importância de estender um pipeline para vários modelos e como fazê-lo usando o Azure DevOps. O laboratório aborda conceitos fundamentais e práticas recomendadas para criar um pipeline de várias fases, criar um modelo de variáveis, criar um modelo de trabalho e criar um modelo de fase.

Estes exercícios levam aproximadamente **20** minutos.

## Antes de começar

Você precisará de uma assinatura do Azure, da organização do Azure DevOps e do aplicativo eShopOnWeb para acompanhar os laboratórios.

- Siga as etapas para [validar seu ambiente de laboratório](APL2001_M00_Validate_Lab_Environment.md).

## Instruções

### Exercício 1: criar um pipeline YAML de vários estágios

#### Tarefa 1: criar um pipeline YAML principal de vários estágios

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb**.

1. Acesse **Pipelines > Pipelines**.

1. Selecione o botão **Novo pipeline** .

1. Selecione **Git do Azure Repos (Yaml)**.

1. Selecione o repositório **eShopOnWeb**.

1. Selecione **Pipeline de início**.

1. Substitua o conteúdo do **azure-pipelines.yml** pelo seguinte código:

    ```YAML
    trigger:
    - main

    pool:
      vmImage: 'windows-latest'

    stages:
    - stage: Dev
      jobs:
      - job: Build
        steps:
        - script: echo Build
    - stage: Test
      jobs:
      - job: Test
        steps:
        - script: echo Test
    - stage: Production
      jobs:
      - job: Deploy
        steps:
        - script: echo Deploy

    ```

1. Selecione **Salvar e executar**. Escolha se deseja confirmar diretamente com a ramificação principal ou criar uma nova ramificação. Selecione o botão **Salvar e executar**.

   > [!NOTE]
   > Se você optar por criar uma nova ramificação, será necessário criar uma solicitação de pull para mesclar as alterações na ramificação principal.

1. Você verá o pipeline em execução com os três estágios (Desenvolvimento, Teste e Produção) e os trabalhos correspondentes. Aguarde até que o pipeline termine e retorne à página **Pipelines**.

    ![Captura de tela do pipeline em execução com as três etapas e os trabalhos correspondentes](media/eshoponweb-pipeline-multi-stage.png)

1. Selecione **...** (Mais opções) no lado direito do pipeline que você acabou de criar e selecione **Renomear/mover**.

1. Renomeie o pipeline para **eShopOnWeb-MultiStage-Main** e selecione **Salvar**.

#### Tarefa 2: criar um modelo de variáveis

1. Vá para **Repos > Arquivos**.

1. Expanda a pasta **.ado** e clique em **Novo arquivo**.

1. Nomeie o arquivo como **eshoponweb-variables.yml** e clique em **Criar**.

1. Adicione o seguinte código ao arquivo:

    ```YAML
    variables:
      resource-group: 'YOUR-RESOURCE-GROUP-NAME'
      location: 'southcentralus' #name of the Azure region you want to deploy your resources
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: 'YOUR-SUBSCRIPTION-ID'
      azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
      webappname: 'YOUR-WEB-APP-NAME'

    ```

    > [!IMPORTANT]
    > Substitua os valores das variáveis pelos valores do seu ambiente (grupo de recursos, local, ID da assinatura, conexão de serviço do Azure e nome do aplicativo Web).

1. Selecione **Confirmar**, adicione um comentário e selecione o botão **Confirmar**.

#### Tarefa 3: preparar o pipeline para usar modelos

1. Acesse **Pipelines > Pipelines**.

1. Abra o pipeline **eShopOnWeb-MultiStage-Main**.

1. Selecione **Editar**.

1. Substitua o conteúdo do **azure-pipelines.yml** pelo seguinte código:

    ```YAML
    trigger:
    - main
    variables:
    - template: .ado/eshoponweb-variables.yml
    
    stages:
    - stage: Dev
      jobs:
      - template: .ado/eshoponweb-ci.yml
    - stage: Test
      jobs:
      - template: .ado/eshoponweb-cd-webapp-code.yml
    - stage: Production
      jobs:
      - job: Deploy
        steps:
        - script: echo Deploy to Production or Swap

    ```

1. Salve o pipeline.

1. Escolha se deseja confirmar diretamente com a ramificação principal ou criar uma nova ramificação. Selecione o botão **Salvar**.

   > [!NOTE]
   > Se você optar por criar uma nova ramificação, será necessário criar uma solicitação de pull para mesclar as alterações na ramificação principal.

#### Tarefa 4: atualização de modelos de CI/CD

1. Acesse **Pipelines > Pipelines**.

1. Edite o pipeline **eshoponweb-ci**.

1. Remova tudo o que estiver acima da seção **trabalhos**.

    ```YAML
    #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
    # trigger:
    # - main
    
    resources:
      repositories:
        - repository: self
          trigger: none
    
    stages:
    - stage: Build
      displayName: Build .Net Core Solution

    ```

1. Salve o pipeline.

1. Acesse **Pipelines > Pipelines**.

1. Edite o pipeline **eshoponweb-cd-webapp-code**.

1. Remova tudo o que estiver acima da seção **trabalhos**.

    ```YAML
    #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
    
    # Trigger CD when CI executed successfully
    resources:
      pipelines:
        - pipeline: eshoponweb-ci
          source: eshoponweb-ci # given pipeline name
          trigger: true

    variables:
      resource-group: 'rg-eshoponweb'
      location: 'southcentralus'
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: ''
      azureserviceconnection: 'azure subs'
      webappname: 'eshoponweb-lab'
      # webappname: 'webapp-windows-eshop'
    
    stages:
    - stage: Deploy
      displayName: Deploy to WebApp`

    ```

1. Atualize a etapa de **download** para:

    ```YAML
    - download: current
      artifact: Website
    - download: current
      artifact: Bicep
    ```

1. Salve o pipeline.

1. (Opcional) Atualize a etapa de produção para implantar seu aplicativo em outro ambiente ou troque os slots de implantação.

#### Tarefa 5: executar o pipeline principal

1. Acesse **Pipelines > Pipelines**.

1. Abra o pipeline **eShopOnWeb-MultiStage-Main**.

1. Selecione **Executar pipeline**.

1. Aguarde até que o pipeline termine e verifique os resultados.

    ![Captura de tela do pipeline em execução com as três etapas e os trabalhos correspondentes](media/multi-stage-completed.png)

### Exercício 2: remover os recursos de laboratório do Azure

1. No portal do Azure, abra o grupo de recursos criado e selecione **Excluir grupo de recursos** para todos os recursos criados neste laboratório.

    ![Captura de tela do botão excluir grupo de recursos.](media/delete-resource-group.png)

    > [!WARNING]
    > Lembre-se sempre de remover todos os recursos do Azure que você não usa mais. Remover recursos não utilizados garante que você não veja encargos inesperados.

## Revisão

Neste módulo, você aprendeu como estender um pipeline para vários modelos e como fazer isso usando o Azure DevOps. Este laboratório abordou os conceitos fundamentais e as práticas recomendadas para criar um pipeline de vários estágios, criar um modelo de variáveis, criar um modelo de trabalho e criar um modelo de estágio.
