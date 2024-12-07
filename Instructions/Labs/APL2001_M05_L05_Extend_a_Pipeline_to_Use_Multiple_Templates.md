---
lab:
  title: Ampliar um pipeline para usar vários modelos
  module: 'Module 5: Extend a pipeline to use multiple templates'
---

# Ampliar um pipeline para usar vários modelos

Neste laboratório, explore a importância de estender um pipeline para vários modelos e como fazê-lo usando o Azure DevOps. O laboratório aborda conceitos fundamentais e práticas recomendadas para criar um pipeline de várias fases, criar um modelo de variáveis, criar um modelo de trabalho e criar um modelo de fase.

Estes exercícios levam aproximadamente **20** minutos.

## Antes de começar

Você precisará de uma assinatura do Azure, da organização do Azure DevOps e do aplicativo eShopOnWeb para acompanhar os laboratórios.

- Siga as etapas para [validar seu ambiente de laboratório](APL2001_M00_Validate_Lab_Environment.md).

## Instruções

### Exercício 1: criar um pipeline YAML de vários estágios

Neste exercício, você criará um pipeline YAML de vários estágios no Azure DevOps.

#### Tarefa 1: criar um pipeline YAML principal de vários estágios

1. Navegue até o portal do Azure DevOps em `https://aex.dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb**.

1. Acesse **Pipelines > Pipelines**.

1. Clique no botão **Novo pipeline**.

1. Selecione **Git do Azure Repos (Yaml)**.

1. Selecione o repositório **eShopOnWeb**.

1. Selecione **Pipeline de início**.

1. Substitua o conteúdo do **azure-pipelines.yml** pelo seguinte código:

   ```yaml
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

1. Selecione **Salvar e executar**. Escolha fazer commit diretamente na ramificação principal e selecione **Salvar e executar** novamente.

1. Você verá o pipeline em execução com os três estágios (Desenvolvimento, Teste e Produção) e os trabalhos correspondentes. Aguarde até que o pipeline seja concluído e volte para a página **Pipelines**.

   ![Captura de tela do pipeline em execução com as três etapas e os trabalhos correspondentes](media/eshoponweb-pipeline-multi-stage.png)

1. Selecione **...** (Mais opções) no lado direito do pipeline que você acabou de criar e selecione **Renomear/mover**.

1. Renomeie o pipeline para **eShopOnWeb-MultiStage-Main** e selecione **Salvar**.

#### Tarefa 2: criar um modelo de variáveis

1. Vá para **Repos > Arquivos**.

1. Expanda a pasta **.ado** e clique em **Novo arquivo**.

1. Nomeie o arquivo como **eshoponweb-variables.yml** e clique em **Criar**.

1. Adicione o seguinte código ao arquivo:

   ```yaml
   variables:
     resource-group: 'YOUR-RESOURCE-GROUP-NAME'
     location: 'centralus'
     templateFile: 'infra/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
     webappname: 'YOUR-WEB-APP-NAME'
   ```

1. Substitua os valores das variáveis pelos valores do seu ambiente:

   - Substitua **YOUR-RESOURCE-GROUP-NAME** pelo nome do grupo de recursos que você deseja usar neste laboratório, por exemplo, **rg-eshoponweb-secure**.
   - Defina o valor da variável **location** como o nome da região do Azure onde você deseja implantar seus recursos, por exemplo, **centralus**.
   - Substitua **YOUR-SUBSCRIPTION-ID** por sua ID de assinatura do Azure.
   - Substitua **YOUR-AZURE-SERVICE-CONNECTION-NAME** por **azure subs**
   - Substitua **YOUR-WEB-APP-NAME** por um nome globalmente exclusivo do aplicativo Web a ser implantado, por exemplo, a cadeia de caracteres **eshoponweb-lab-multi-123456** seguida por um número aleatório de seis dígitos.  

1. Selecione **Confirmar**, na caixa de texto de comentário de confirmação, insira `[skip ci]` e selecione **Confirmar**.

   > **Observação**: ao adicionar o comentário `[skip ci]` ao fazer commit, você impedirá a execução automática do pipeline, que, neste ponto, é executado por padrão após cada alteração no repositório. 

#### Tarefa 3: preparar o pipeline para usar modelos

1. No portal do Azure DevOps, na página do projeto **eShopOnWeb**, acesse **Repositórios**.

1. No diretório raiz do repositório, selecione **azure-pipelines.yml**, que contém a definição do pipeline **eShopOnWeb-MultiStage-Main**.

1. Clique no botão **Editar**.

1. Substitua o conteúdo do **azure-pipelines.yml** pelo seguinte código:

   ```yaml
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

1. Selecione **Commit**, na caixa de texto de comentário de commit, insira `[skip ci]` e selecione **Fazer commit**.

#### Tarefa 4: atualização de modelos de CI/CD

1. Nos **Repositórios** do projeto **eShopOnWeb**, selecione o diretório **.ado** e selecione o arquivo **eshoponweb-ci.yml**.

1. Clique no botão **Editar**.

1. Remova tudo o que estiver acima da seção **trabalhos**.

   ```yaml
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

1. Selecione **Commit**, na caixa de texto de comentário de commit, insira `[skip ci]` e selecione **Fazer commit**.

1. Nos **Repositórios** do projeto **eShopOnWeb**, selecione o diretório **.ado** e selecione o arquivo **eshoponweb-cd-webapp-code.yml**.

1. Clique no botão **Editar**.

1. Remova tudo o que estiver acima da seção **trabalhos**.

   ```yaml
    # NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml") #
    # Trigger CD when CI executed successfully
    
    resources:
      pipelines:
        - pipeline: eshoponweb-ci
          source: eshoponweb-ci # given pipeline name
          trigger: true
    
    repositories:
      - repository: eShopSecurity
        type: git
        name: eShopSecurity/eShopSecurity # name of the project and repository
    
    variables:
      - template: eshoponweb-secure-variables.yml@eShopSecurity # name of the template and repository
    
    stages:
      - stage: Test
        displayName: Testing WebApp
        jobs:
          - deployment: Test
            pool: eShopOnWebSelfPool
            environment: Test
            strategy:
              runOnce:
                deploy:
                  steps:
                    - script: echo Hello world! Testing environments!
    
      - stage: Deploy
        displayName: Deploy to WebApp
   ```

1. Substitua o conteúdo existente da etapa **#download artifacts** por:

   ```yaml
       - download: current
         artifact: Website
       - download: current
         artifact: Bicep
   ```

1. Selecione **Commit**, na caixa de texto de comentário de commit, insira `[skip ci]` e selecione **Fazer commit**.

#### Tarefa 5: executar o pipeline principal

1. Acesse **Pipelines > Pipelines**.

1. Abra o pipeline **eShopOnWeb-MultiStage-Main**.

1. Selecione **Executar pipeline**.

   > **Observação**: se você receber uma mensagem informando que o pipeline precisa de permissão para acessar um recurso antes que essa execução possa continuar, selecione **Exibir** e, em seguida, selecione **Permitir** e **Permitir** novamente para permitir que o pipeline seja executado.

   > **Observação**: se algum trabalho na fase de implantação falhar, vá até a página de execução de pipeline e selecione **Executar novamente trabalhos com falha***.

1. Aguarde até que o pipeline termine e verifique os resultados.

   ![Captura de tela do pipeline em execução com as três etapas e os trabalhos correspondentes](media/multi-stage-completed.png)

> [!IMPORTANT]
> Lembre-se de excluir os recursos criados no portal do Azure para evitar cobranças desnecessárias.

## Revisão

Neste laboratório, você aprendeu a estender um pipeline para vários modelos usando o Azure DevOps. Esse laboratório abordou conceitos fundamentais e melhores práticas para criar um pipeline de várias fases, criando um modelo de variáveis, um modelo de trabalho e um modelo de fase.
