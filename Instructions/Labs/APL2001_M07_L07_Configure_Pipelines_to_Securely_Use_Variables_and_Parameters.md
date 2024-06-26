---
lab:
  title: Configurar pipelines para usar variáveis e parâmetros com segurança
  module: 'Module 7: Configure pipelines to securely use variables and parameters'
---

# Configurar pipelines para usar variáveis e parâmetros com segurança

Neste laboratório, você vai aprender a como configurar pipelines para usar variáveis e parâmetros com segurança.

Estes exercícios levam aproximadamente **20** minutos.

## Antes de começar

Você precisará de uma assinatura do Azure, da organização do Azure DevOps e do aplicativo eShopOnWeb para acompanhar os laboratórios.

- Siga as etapas para [validar seu ambiente de laboratório](APL2001_M00_Validate_Lab_Environment.md).

## Instruções

### Exercício 1: garantir os tipos de variáveis e parâmetros

#### Tarefa 1: importar e executar o pipeline de CI

Comece importando o pipeline de CI chamado [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto eShopOnWeb.

1. Acesse **Pipelines > Pipelines**.

1. Selecione **Criar pipeline**.

1. Selecione **Git do Azure Repos (Yaml)**.

1. Selecione o repositório **eShopOnWeb**.

1. Selecione **Arquivo YAML existente do Azure Pipelines**.

1. Selecione o arquivo **/.ado/eshoponweb-ci.yml** e selecione **Continuar**.

1. Clique no botão **Executar** para executar o pipeline.

   > [!NOTE]
   > Seu pipeline assumirá um nome com base no nome do projeto. Renomeie-o para identificar melhor o pipeline.

1. Vá para **Pipelines > Pipelines** e selecione o pipeline criado recentemente. Selecione as reticências e a opção **Renomear/mover**.

1. Nomeie-o **eshoponweb-ci-parameters** e selecione **Salvar**.

#### Tarefa 2: garantir tipos de parâmetros para pipelines YAML

Nesta tarefa, você definirá parâmetros e tipos de parâmetros para o pipeline.

1. Vá para **Pipelines > Pipelines** e selecione o pipeline **eshoponweb-ci-parameters**.

1. Selecione **Editar**.

1. Adicione as seguintes seções de parâmetros e recursos à parte superior do arquivo YAML:

   ```yaml
   parameters:
   - name: dotNetProjects
     type: string
     default: '**/*.sln'
   - name: testProjects
     type: string
     default: 'tests/UnitTests/*.csproj'

   resources:
     repositories:
     - repository: self
       trigger: none

   ```

1. Substitua os caminhos codificados nas tarefas `Restore`, `Build` e `Test` pelos parâmetros que você acabou de criar.

   - **Substitua projetos**: `**/*.sln` por projetos: `${{ "{{" }} parameters.dotNetProjects }}` nas tarefas `Restore` e `Build`.
   - **Substituir projetos**: `tests/UnitTests/*.csproj` por projetos: `${{ "{{" }} parametertestProjects }}` na tarefa `Test`

    As tarefas `Restore`, `Build` e `Test` na seção de etapas do arquivo YAML devem ter esta aparência:

    {% raw %}

    ```yaml
    steps:
    - task: DotNetCoreCLI@2
      displayName: Restore
      inputs:
        command: 'restore'
        projects: ${{ parameters.dotNetProjects }}
        feedsToUse: 'select'
    
    - task: DotNetCoreCLI@2
      displayName: Build
      inputs:
        command: 'build'
        projects: ${{ parameters.dotNetProjects }}
    
    - task: DotNetCoreCLI@2
      displayName: Test
      inputs:
        command: 'test'
        projects: ${{ parameters.testProjects }}
    
    ```

    {% endraw %}

1. Salve e execute o pipeline. Verifique se a execução do pipeline foi concluída com êxito.

   ![Captura de tela da execução de pipeline com parâmetros.](media/pipeline-parameters-run.png)

#### Tarefa 3: Proteger variáveis e parâmetros

Nesta tarefa, você protegerá as variáveis e os parâmetros do pipeline usando grupos de variáveis.

1. Vá para **Pipelines > Biblioteca**.

1. Selecione o botão **+ Grupo de variáveis** para criar um novo grupo de variáveis chamado **BuildConfigurations**.

1. Adicione uma variável chamada **buildConfiguration** e defina seu valor como `Release`.

1. Salve o grupo de variáveis.

   ![Captura de tela do grupo de variáveis com BuildConfigurations.](media/eshop-variable-group.png)

1. Selecione o botão **Permissões de pipeline** e selecione o botão **+** para adicionar um novo pipeline.

1. Selecione o pipeline **eshoponweb-ci-parameters** para permitir que o pipeline use o grupo de variáveis.

   ![Captura de tela das permissões de pipeline.](media/pipeline-permissions.png)

   > [!NOTE]
   > Você também pode definir que usuários ou grupos específicos possam editar o grupo de variáveis clicando no botão **Segurança**.

1. Acesse **Pipelines > Pipelines**.

1. Abra o pipeline **eshoponweb-ci-parameters** e selecione **Editar**.

1. Na parte superior do arquivo yml, logo abaixo dos parâmetros, faça referência ao grupo de variáveis adicionando o seguinte:

   ```yaml
   variables:
     - group: BuildConfigurations
   ```

1. Na tarefa "Compilar", substitua o comando: "build" pelas seguintes linhas para utilizar a configuração de compilação do grupo de variáveis.

    {% raw %}

    ```yaml
            command: 'build'
            projects: ${{ parameters.dotNetProjects }}
            configuration: $(buildConfiguration)
    ```

    {% endraw %}

1. Salve e execute o pipeline. Ele deve ser executado com êxito com a configuração de compilação definida como `Release`. Você pode verificar isso observando os logs da tarefa "Compilar".

> [!NOTE]
> Seguindo essa abordagem, você pode proteger suas variáveis e parâmetros usando grupos de variáveis, sem precisar codificá-los em arquivos YAML.

#### Tarefa 4: Validar variáveis e parâmetros obrigatórios

Nesta tarefa, você validará as variáveis obrigatórias antes da execução do pipeline.

1. Acesse **Pipelines > Pipelines**.

1. Abra o pipeline **eshoponweb-ci-parameters** e selecione **Editar**.

1. Bem no início da seção de fases (seguindo a linha `stage:`), adicione uma nova fase chamada **Validate** para validar as variáveis obrigatórias antes da execução do pipeline.

    ```yaml
    - stage: Validate
      displayName: Validate mandatory variables
      jobs:
      - job: ValidateVariables
        pool:
          vmImage: ubuntu-latest
        steps:
        - script: |
            if [ -z "$(buildConfiguration)" ]; then
              echo "Error: buildConfiguration variable is not set"
              exit 1
            fi
          displayName: 'Validate Variables'
     ```

    > [!NOTE]
    > Este estágio executará um script para validar a variável buildConfiguration. Se as variáveis não forem definidas, o script falhará e o pipeline será interrompido.

1. Faça com que a fase **Build** dependa da fase **Validate** adicionando `dependsOn: Validate` no início da fase **Build**:

    ```yaml
    - stage: Build
      displayName: Build .Net Core Solution
      dependsOn: Validate
    ```

1. Salve e execute o pipeline. Ele será executado com êxito porque a variável buildConfiguration está definida no grupo de variáveis.

1. Para testar a validação, remova a variável buildConfiguration do grupo de variáveis ou exclua o grupo de variáveis e execute o pipeline novamente. Ele deve falhar com o seguinte erro:

    ```yaml
    Error: buildConfiguration variable is not set   
    ```

    ![Captura de tela da execução de pipeline com falha na validação.](media/pipeline-validation-fail.png)

1. Adicione o grupo de variáveis e a variável buildConfiguration de volta ao grupo de variáveis e execute o pipeline novamente. Ele deve ser executado com êxito.

## Revisão

Neste laboratório, você aprendeu como configurar pipelines para usar variáveis e parâmetros com segurança e como validar variáveis e parâmentros obrigatórios.
