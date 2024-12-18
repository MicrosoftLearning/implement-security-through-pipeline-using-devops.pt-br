---
lab:
  title: Configurar e validar permissões
  module: 'Module 4: Configure and validate permissions'
---

# Configurar e validar permissões

Neste laboratório, você vai configurar um ambiente seguro que adere ao princípio de privilégios mínimos, garantindo que os membros possam acessar apenas os recursos necessários para executar suas tarefas e minimizar possíveis riscos de segurança. Isso envolve configurar e validar permissões de usuário e pipeline e configurar verificações de aprovação e branch no Azure DevOps.

Estes exercícios levam aproximadamente **20** minutos.

## Antes de começar

Você precisará de uma assinatura do Azure, da organização do Azure DevOps e do aplicativo eShopOnWeb para acompanhar os laboratórios.

- Siga as etapas para [validar seu ambiente de laboratório](APL2001_M00_Validate_Lab_Environment.md).
- Instale um agente auto-hospedado seguindo o laboratório [Configurar agentes e pools de agentes para pipelines seguros](APL2001_M02_L02_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md) ou as etapas em [Instalar um agente auto-hospedado](https://learn.microsoft.com/azure/devops/pipelines/agents/windows-agent).

## Instruções

### Exercício 0: (se concluído, ignorar) Importar e executar pipelines de CI/CD

Nexte exercício, você importará e executará o pipeline de CI/CD no projeto do Azure DevOps.

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

#### Tarefa 2: (se concluída, ignorar) Importar e executar o pipeline de CD

> **Observação**: nesta tarefa, você importará e executará o pipeline de CD chamado [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Acesse **Pipelines > Pipelines**.

1. Selecione o botão **Novo pipeline**.

1. Selecione **Git do Azure Repos (Yaml)**.

1. Selecione o repositório **eShopOnWeb**.

1. Selecione **Arquivo YAML existente do Azure Pipelines**.

1. Selecione o arquivo **/.ado/eshoponweb-cd-webapp-code.yml** e selecione **Continuar**.

1. Na definição do pipeline YAML, defina a seção de variáveis como:

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

1. Selecione **Salvar e executar** e escolha fazer commit diretamente na ramificação principal.

1. Selecione **Salvar e executar** novamente.

1. Abra a execução de pipeline. Se você vir a mensagem "Este pipeline precisa de permissão para acessar um recurso antes que essa execução possa continuar a Implantar no WebApp", selecione **Exibir**, **Permitir** e **Permitir** novamente. Isso é necessário para permitir que o pipeline crie o recurso do Serviço de Aplicativo do Azure.

   ![Captura de tela do acesso de permissão do pipeline YAML.](media/pipeline-deploy-permit-resource.png)

1. A implantação pode levar alguns minutos para ser concluída, aguarde a execução do pipeline. O pipeline é disparado após a conclusão do pipeline de CI e inclui as seguintes tarefas:

   - **AzureResourceManagerTemplateDeployment**: Implanta o aplicativo Web do Serviço de Aplicativo do Azure usando o modelo bicep.
   - **AzureRmWebAppDeployment**: Publica o site no aplicativo Web do Serviço de Aplicativo do Azure.

   > **Observação**: caso a implantação falhe, navegue até a página de execução do pipeline e selecione **Executar trabalhos com falha novamente** para invocar outra execução de pipeline.

   > **Observação**: seu pipeline terá um nome com base no nome do projeto. Vamos **renomear** o pipeline para melhor identificá-lo.

1. Vá para **Pipelines > Pipelines** e selecione o pipeline criado recentemente. Selecione as reticências e, em seguida, selecione a opção **Renomear/mover**.

1. Nomeio como **eshoponweb-cd-webapp-code** e clique em **Salvar**.

### Exercício 1: Configurar e validar verificações de aprovação e remificações

Neste exercício, você configurará e validará as verificações de aprovação e branch para o pipeline de CD.

#### Tarefa 1: Criar um ambiente e adicionar aprovações e verificações

1. No portal do Azure DevOps, na página do projeto **eShopOnWeb**, selecione **Pipelines > Ambientes**.

1. Selecione **Criar ambiente**.

1. Nomeie o ambiente **Teste**, selecione **Nenhum** como o recurso e selecione **Criar**.

1. No ambiente **Teste**, selecione a guia **Aprovações e verificações**.

1. Selecione **Approvals**.

1. Na caixa de texto **Aprovadores**, insira o nome de usuário.

1. Se não estiver habilitado, marque a caixa "Permitir que os aprovadores aprovem suas próprias execuções".

1. Dê as instruções **Aprovar a implantação para testar** e selecione **Criar**.

   ![Captura de tela das aprovações do ambiente com instruções.](media/add-environment-approvals.png)

1. Clique no botão **+ Adicionar novo**, selecione **Controle de ramificação** e, em seguida, selecione **Avançar**.

1. No campo **Branch permitidos**, deixe o padrão e selecione **Criar**. Você poderá adicionar mais branches se desejar.

   ![Captura de tela do controle de branch do ambiente com o branch principal.](media/add-environment-branch-control.png)

1. Crie outro ambiente chamado **Produção** e execute as mesmas etapas para adicionar aprovações e controle de ramificação. Para diferenciar os ambientes, adicione as instruções **Aprovar a implantação para Produção** e defina os branches permitidos como **refs/heads/main**.

> **Observação**: você pode adicionar mais ambientes e configurar aprovações e controle de ramificações para eles. Além disso, você pode configurar a **Segurança** para adicionar usuários ou grupos ao ambiente com funções como *Usuário*, *Criador* ou *Leitor*.

#### Tarefa 2: configurar o pipeline de CD para usar o novo ambiente

1. No portal do Azure DevOps, na página do projeto **eShopOnWeb**, selecione **Pipelines > Pipelines**.

1. Abra o pipeline **eshoponweb-cd-webapp-code**.

1. Selecione **Editar**.

1. Selecione a linha acima do comentário **#download de artefatos**, até a linha **stages:** no arquivo YAML do pipeline e substitua o conteúdo pelo seguinte código:

   ```yaml
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
     jobs:
     - deployment: Deploy
       pool: eShopOnWebSelfPool
       environment: Production
       strategy:
         runOnce:
           deploy:
             steps:
             - checkout: self
   ```

   > **Observação**: você precisará deslocar todas as linhas seguindo o código acima de seis espaços à direita para garantir que as regras de recuo yaml sejam atendidas.

   Seu pipeline deve ter esta aparência:

   ![Captura de tela do pipeline com a nova implantação.](media/pipeline-add-yaml-deployment.png)

   > [!IMPORTANT]
   > Confirme se o nome do **pool** é o mesmo que você criou no laboratório anterior.

1. Clique em **Validar e salvar**, escolha confirmar diretamente na ramificação principal e clique em **Salvar**.

1. Seu pipeline será acionado automaticamente. Abra a execução de pipeline.

   > **Observação**: se você vir a mensagem "Este pipeline precisa de permissão para acessar um recurso antes que essa execução possa continuar a Testar o WebApp", selecione **Exibir**, **Permitir** e **Permitir** novamente.

1. Abra o estágio **Testando o WebApp** do pipeline e observe a mensagem **1 aprovação precisa de sua revisão antes que essa execução possa continuar Testando o WebApp**. Selecione **Revisar** e **Aprovar**.

   ![Captura de tela do pipeline com a Etapa de teste a ser aprovada".](media/pipeline-test-environment-approve.png)

1. Aguarde a conclusão do pipeline, abra o log do pipeline e verifique se a etapa **Testar o WebApp** foi executada com êxito.

   ![Captura de tela do log de pipeline com a etapa Testar o WebApp executada com êxito".](media/pipeline-test-environment-success.png)

1. Volte para o pipeline e você verá o estágio **Implantar no WebApp** aguardando aprovação. Selecione **Revisar** e **Aprovar** como fez antes para o estágio **Testar o WebApp**.

   > **Observação**: se você vir a mensagem "Este pipeline precisa de permissão para acessar um recurso antes que essa execução possa continuar a Implantar no WebApp", selecione **Exibir**, **Permitir** e **Permitir** novamente.

1. Aguarde até que o pipeline seja concluído e verifique se o estágio **Implantar no WebApp** foi executado com êxito.

   ![Captura de tela do pipeline com o estágio Implantar no WebApp a ser aprovado".](media/pipeline-deploy-environment-success.png)

> **Observação**: você conseguirá executar o pipeline com êxito com as verificações de aprovações e de ramificação em ambos os ambientes, de Teste e de Produção.

> [!IMPORTANT]
> Lembre-se de excluir os recursos criados no portal do Azure para evitar cobranças desnecessárias.

## Revisão

Neste laboratório, você aprendeu a configurar um ambiente seguro que adere ao princípio de privilégios mínimos, garantindo que os membros possam acessar apenas os recursos necessários para executar as tarefas deles e minimizar possíveis riscos de segurança. Você configurou e validou permissões de usuário e pipeline e configurou verificações de aprovação e branch no Azure DevOps.
