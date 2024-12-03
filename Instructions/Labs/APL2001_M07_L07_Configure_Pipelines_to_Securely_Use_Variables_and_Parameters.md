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

#### Tarefa 1: (se concluída, ignorar) Importar e executar o pipeline de CI

Vamos começar importando o pipeline de CI chamado [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Navegue até o portal do Azure DevOps em `https://aex.dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb** no Azure DevOps.

1. Acesse **Pipelines > Pipelines**.

1. Selecione o botão **Criar pipeline**.

1. Selecione **Git do Azure Repos (Yaml)**.

1. Selecione o repositório **eShopOnWeb**.

1. Selecione **Arquivo YAML existente do Azure Pipelines**.

1. Selecione o arquivo **/.ado/eshoponweb-ci.yml** e clique em **Continuar**.

1. Clique no botão **Executar** para executar o pipeline.

   > **Observação**: seu pipeline terá um nome com base no nome do projeto. Você o renomeará para facilitar a identificação do pipeline.

1. Vá para **Pipelines > Pipelines** e selecione o pipeline criado recentemente. Selecione as reticências e, em seguida, selecione a opção **Renomear/mover**.

1. Nomeie-o **eshoponweb-ci** e selecione **Salvar**.

#### Tarefa 2: garantir tipos de parâmetros para pipelines YAML

Nesta tarefa, você definirá parâmetros e tipos de parâmetros para o pipeline.

1. Vá para **Pipelines > Pipelines** e selecione o pipeline **eshoponweb-ci**.

1. Selecione **Editar**.

1. Adicione os seguintes parâmetros acima das seções de trabalho na parte superior do arquivo YAML:

   ```yaml
   parameters:
   - name: dotNetProjects
     type: string
     default: '**/*.sln'
   - name: testProjects
     type: string
     default: 'tests/UnitTests/*.csproj'

   jobs:
   - job: Build
     pool: eShopOnWebSelfPool
     steps:

   ```

1. Substitua os caminhos codificados nas tarefas "Restaurar", "Compilar" e '"Testar" pelos parâmetros que você acabou de criar.

   - **Substitua projetos**: `**/*.sln` por projetos: `${{ parameters.dotNetProjects }}` nas tarefas `Restore` e `Build`.
   - **Substituir projetos**: `tests/UnitTests/*.csproj` por projetos: `${{ parameters.testProjects }}` na tarefa `Test`

   As tarefas "Restaurar", "Compilar" e "Testar" na seção de etapas do arquivo YAML devem ter esta aparência:

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

1. Clique em **Validar e salvar** para salvar as alterações e, em seguida, clique em **Salvar**.

1. Navegue até **Pipelines > Pipelines** e abra o pipeline **eshoponweb-ci** que está sendo executado e é disparado automaticamente.

1. Verifique se a execução do pipeline foi concluída com êxito.

   ![Captura de tela da execução de pipeline com parâmetros.](media/pipeline-parameters-run.png)

#### Tarefa 3: Proteger variáveis e parâmetros

Nesta tarefa, você protegerá as variáveis e os parâmetros do pipeline usando grupos de variáveis.

1. Vá para **Pipelines > Biblioteca**.

1. Selecione o botão **+ Grupo de variáveis** para criar um novo grupo de variáveis chamado `BuildConfigurations`.

1. Adicione uma variável chamada `buildConfiguration` e defina seu valor como `Release`.

1. Salve o grupo de variáveis.

   ![Captura de tela do grupo de variáveis com BuildConfigurations.](media/eshop-variable-group.png)

1. Selecione o botão **Permissões de pipeline** e selecione o botão **+** para adicionar um novo pipeline.

1. Selecione o pipeline **eshoponweb-ci** para permitir que o pipeline use o grupo de variáveis.

   ![Captura de tela das permissões de pipeline.](media/pipeline-permissions.png)

   > **Observação**: você também pode definir que usuários ou grupos específicos possam editar o grupo de variáveis clicando no botão **Segurança**.

1. Acesse **Pipelines > Pipelines**.

1. Abra o pipeline **eshoponweb-ci** e selecione **Editar**.

1. Na parte superior do arquivo yml, logo abaixo dos parâmetros, faça referência ao grupo de variáveis adicionando o seguinte:

   ```yaml
   variables:
     - group: BuildConfigurations
   ```

1. Na tarefa "Compilar", adicione o parâmetro de configuração à tarefa para utilizar a configuração de compilação do grupo de variáveis.

    ```yaml
            command: 'build'
            projects: ${{ parameters.dotNetProjects }}
            configuration: $(buildConfiguration)
    ```

1. Clique em **Validar e salvar** para salvar as alterações e, em seguida, clique em **Salvar**.

1. Edite o pipeline **eshoponweb-ci**. Ele deve ser executado com êxito com a configuração de compilação definida como "Lançamento". Você pode verificar isso observando os logs da tarefa "Compilar".

> **Observação**: se você não vir a configuração de compilação definida como "Lançamento" nos logs, habilite o diagnóstico do sistema e execute novamente o pipeline para ver o valor da configuração.

> **Observação**: seguindo essa abordagem, você pode proteger suas variáveis e parâmetros usando grupos de variáveis, sem precisar codificá-los em arquivos YAML.

#### Tarefa 4: Validar variáveis e parâmetros obrigatórios

Nesta tarefa, você validará as variáveis obrigatórias antes da execução do pipeline.

1. Acesse **Pipelines > Pipelines**.

1. Abra o pipeline **eshoponweb-ci** e selecione **Editar**.

1. Na seção de etapas, em no início (após a linha **steps:**), adicione uma nova tarefa de script para validar as variáveis obrigatórias antes da execução do pipeline.

    ```yaml
    - script: |
        IF NOT DEFINED buildConfiguration (
          ECHO Error: buildConfiguration variable is not set
          EXIT /B 1
        )
      displayName: 'Validate Variables'
     ```

    > **Observação**: esta é uma validação simples para verificar se a variável está definida. Se as variáveis não forem definidas, o script falhará e o pipeline será interrompido. Você pode adicionar uma validação mais complexa para verificar o valor da variável ou se ela está definida como um valor específico.

1. Clique em **Validar e salvar** para salvar as alterações e, em seguida, clique em **Salvar**.

1. Edite o pipeline **eshoponweb-ci**. Ele será executado com êxito porque a variável buildConfiguration está definida no grupo de variáveis.

1. Para testar a validação, remova a variável buildConfiguration do grupo de variáveis ou renomeie o grupo de variáveis e execute o pipeline novamente. Ele deve falhar com o seguinte erro:

    ```yaml
    Error: buildConfiguration variable is not set   
    ```

    ![Captura de tela da execução de pipeline com falha na validação.](media/pipeline-validation-fail.png)

1. Adicione a variável buildConfiguration de volta ao grupo de variáveis e execute o pipeline novamente. Ele deve ser executado com êxito.

> [!IMPORTANT]
> Lembre-se de excluir os recursos criados no portal do Azure para evitar cobranças desnecessárias.

## Revisão

Neste laboratório, você aprendeu como configurar pipelines para usar variáveis e parâmetros com segurança e como validar variáveis e parâmentros obrigatórios.
