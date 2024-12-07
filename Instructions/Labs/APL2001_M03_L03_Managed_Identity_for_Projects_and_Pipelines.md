---
lab:
  title: Identidade gerenciada para projetos e pipelines
  module: 'Module 3: Manage identity for projects, pipelines, and agents'
---

# Identidade gerenciada para projetos e pipelines

As identidades gerenciadas oferecem um método seguro para controlar o acesso aos recursos do Azure. O Azure manipula essas identidades automaticamente, permitindo que você verifique o acesso a serviços compatíveis com a autenticação do Azure AD. Isso significa que você não precisa incorporar credenciais ao seu código, o que aumenta a segurança. No Azure DevOps, as identidades gerenciadas podem autenticar recursos do Azure em seus agentes auto-hospedados, simplificando o controle de acesso sem comprometer a segurança.

Neste laboratório, você criará uma identidade gerenciada e a usará em pipelines YAML do Azure DevOps em execução em agentes auto-hospedados para implantar recursos do Azure.

O laboratório leva cerca de **30** minutos.

## Antes de começar

Você precisará de uma assinatura do Azure, da organização do Azure DevOps e do aplicativo eShopOnWeb para acompanhar os laboratórios.

- Verifique se você tem uma conta Microsoft ou uma conta do Microsoft Entra com a função de Colaborador ou Proprietário na assinatura do Azure. Para obter detalhes, veja [Listar designações de função do Azure usando o portal do Azure](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e designar funções de administrador no Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Pré-requisitos

Concluir os laboratório:

- Siga as etapas para [validar seu ambiente de laboratório](APL2001_M00_Validate_Lab_Environment.md).
- [Configurar uma estrutura de repositório e projeto para dar suporte a pipelines seguros](APL2001_M01_L01_Configure_a_Project_and_Repository_Structure_to_Support_Secure_Pipelines.md)
- [Configurar agentes e pools de agentes para pipelines seguros](APL2001_M02_L02_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md)

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

### Exercício 1: Configurar identidade gerenciada em pipelines do Azure

Neste exercício, você usará uma identidade gerenciada para configurar uma nova conexão de serviço e incorporá-la aos pipelines de CI/CD.

#### Tarefa 1: Definir a identidade gerenciada na assinatura do Azure

1. No seu navegador, abra o portal do Azure em `https://portal.azure.com`.

1. No portal do Azure, navegue até a página exibindo a VM do Azure **eshoponweb-vm** implantada [no laboratório anterio](APL2001_M02_L02_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md).

1. Na página da VM do Azure **eshoponweb-vm**, na barra de ferramentas, selecione **Iniciar** para iniciá-la, caso tenha sido interrompida.

1. Na página da VM do Azure **eshoponweb-vm**, no menu vertical no lado esquerdo, na seção **Segurança**, selecione **Identidade**.

1. Na página **Identidade**, verifique se o **Status** é **Ativado** e selecione **Atribuições de função do Azure**.

1. Selecione o botão **Adicionar atribuição de função** e execute as seguintes ações:

   | Configuração | Ação |
   | -- | -- |
   | Lista suspensa **Escopo**. | Selecione a **Assinatura**. |
   | Lista suspensa **Assinatura** | Selecione sua assinatura do Azure. |
   | Lista suspensa **Função** | Selecione a função **Colaborador**. |

   > **Observação**: o escopo da assinatura deve acomodar implantações nos laboratórios subsequentes.

1. Selecione o botão **Salvar**.

    ![Captura de tela do painel Adicionar Atribuição](media/add-role-assignment.png)

#### Tarefa 2: Criar uma conexão de serviço com base em identidade gerenciada

1. Alterne para o navegador da Web exibindo o projeto **eShopOnWeb** no portal do Azure DevOps em `https://aex.dev.azure.com`.

1. No projeto **eShopOnWeb**, navegue até **Configurações de projeto > Conexões de serviço**.

1. Selecione o botão **Nova conexão de serviço** e selecione **Azure Resource Manager**.

1. Selecione **Identidade Gerenciada** como o **Método de autenticação**.

1. Defina o nível de escopo como **Assinatura** e forneça as informações do portal do Azure, incluindo a **ID da Assinatura**, o **Nome da Assinatura** e a **ID do Locatário**.

   > **Observação**: você pode encontrar a **ID da Assinatura** no portal do Azure navegando até a folha **Assinaturas** e selecionando a assinatura que está usando. A **ID do locatário** pode ser encontrada na folha **ID do Microsoft Entra**.

1. Em **Nome da conexão do serviço**, digite **subs do Azure gerenciados**. Esse nome será referenciado em pipelines YAML ao acessar a sua assinatura do Azure.

1. Selecione **Salvar**.

#### Tarefa 3: Atualizar o pipeline de CI com o pool de agente auto-hospedado

Nesta tarefa, você atualizará o pipeline de CI para usar o pool de agente auto-hospedado.

1. Alterne para o navegador da Web exibindo o projeto **eShopOnWeb** no portal do Azure DevOps.

1. Na página do projeto **eShopOnWeb**, navegue até **Pipelines > Pipelines**.

1. Selecione o pipeline **eshoponweb-ci** e selecione **Editar**.

1. Na subseção **trabalhos** da seção **estágios**, atualize o valor da propriedade **pool** para fazer referência ao pool de agentes auto-hospedado **eShopOnWebSelfPool** configurado nesta tarefa para que ele tem o seguinte formato:

   ```yaml
     jobs:
     - job: Build
       pool: eShopOnWebSelfPool
       steps:
       - task: DotNetCoreCLI@2
   ```

1. Selecione **Validar e salvar** e escolha fazer commit diretamente na ramificação principal.

1. Selecione **Salvar** novamente.

1. Selecione **Executar** o pipeline e clique em **Executar** novamente.

1. Verifique se o trabalho de compilação está em execução no agente **eShopOnWebSelfAgent** e ele é concluído com êxito.

    > **Observação**: se você vir a mensagem **A solicitação do agente não está em execução porque todos os agentes em potencial estão executando outras solicitações. Posição atual na fila: 1**, você pode aguardar que o agente fique disponível ou pode interromper o trabalho do agente que está em execução. Pode ser o pipeline de CD que está sendo executado automaticamente.

    > **Observação**: se você vir a mensagem "Este pipeline precisa de permissão para acessar um recurso antes que essa execução possa continuar a compilar a solução .Net Core" na página de execução de pipeline, selecione **Exibir**, **Permitir** e **Permitir** novamente. Isso é necessário para permitir que o pipeline use o pool de agente auto-hospedado.

#### Tarefa 4: Atualizar o pipeline de CD para usar a conexão do pool de agente auto-hospedado e o serviço baseado em identidade gerenciada

Nesta tarefa, você atualizará o pipeline de CD para usar a conexão de serviço baseado em identidade gerenciada e o pool de agente auto-hospedado.

1. Vá para a janela do navegador da Web exibindo o projeto **eShopSecurity** no portal do Azure DevOps.

   > **Observação**: **eShopSecurity** é o nome do projeto que você criou no [primeiro laboratório](APL2001_M01_L01_Configure_a_Project_and_Repository_Structure_to_Support_Secure_Pipelines.md).

1. Na página do projeto **eShopSecurity**, navegue até **Repos > Arquivos**.

1. Selecione o arquivo **eshoponweb-secure-variables.yml** e clique no botão **Editar**.

1. Na seção variáveis, atualize a variável **azureserviceconnection** para usar o nome da conexão de serviço que você criou na tarefa anterior, **azure subs managed**.

   ```yaml
     azureserviceconnection: 'azure subs managed'
   ```

1. Clique no botão **Confirmar** e escolha fazer commit diretamente na ramificação principal.

1. Clique no botão **Confirmar** novamente.

1. Alterne para o projeto **eShopOnWeb**.

1. Na página do projeto **eShopOnWeb**, navegue até **Pipelines > Pipelines**.

1. Selecione o pipeline **eshoponweb-cd-webapp-code** e selecione **Editar**.

1. Na subseção **trabalhos** da seção **fases**, atualize o valor da propriedade **pool** para referenciar o pool de agentes auto-hospedados que você criou no laboratório anterior, **eShopOnWebSelfPool**, para que ele tenha o seguinte formato:

   ```yaml
     jobs:
     - job: Deploy
       pool: eShopOnWebSelfPool
       steps:
       #download artifacts
       - download: eshoponweb-ci
   ```

1. Clique no botão **Validar e salvar** e escolha fazer commit diretamente na ramificação principal.

1. Clique em **Salvar** novamente.

1. Navegue até **Pipelines > Pipelines** e selecione o pipeline **eshoponweb-cd-webapp-code** já em execução por causa da tarefa anterior.

1. Clique em execução de pipeline e clique em **Cancelar**. Clique no botão **Sim** para confirmar.

   > **Observação**: você executará o diagnóstico do sistema de habilitação do pipeline para ver os logs do pipeline.

1. Clique em **Executar novo pipeline**, marque a caixa de seleção "Habilitar diagnóstico do sistema" e clique no botão **Executar**.

1. Abra a execução de pipeline.

   > **Observação**: se você vir a mensagem "Este pipeline precisa de permissão para acessar dois recursos antes que essa execução possa continuar a Implantar no WebApp", selecione **Exibir**, **Permitir** e **Permitir** novamente para todos os recursos. Isso é necessário para permitir que o pipeline use a conexão de serviço e o pool de agente auto-hospedado.

1. A implantação pode levar alguns minutos para ser concluída, aguarde a execução do pipeline.

   > [!IMPORTANT]
   > Se o pipeline falhar devido ao erro da CLI do AZ, talvez seja necessário reiniciar o agente auto-hospedado e executar novamente o pipeline.
   > Para reiniciar o agente, no portal do Azure, navegue até a página que exibe a VM do Azure **eshoponweb-vm** implantada no laboratório anterior, conecte-se à VM usando o botão **Conectar** e reinicie o nome do serviço do Agente do Azure Pipelines começando com vstsagent. Clique com o botão direito do mouse no serviço de agente e selecione **Reiniciar**.

1. Você deve ver nos logs de pipeline que o pipeline está usando a identidade gerenciada.

   ![Captura de tela dos logs de pipeline usando a identidade gerenciada.](media/pipeline-logs-managed-identity.png)

   > **Observação**: após a conclusão do pipeline, você pode usar o portal do Azure para verificar o estado dos recursos do aplicativo Web do Serviço de Aplicativo.

   > [!IMPORTANT]
   > Lembre-se de excluir os recursos criados no portal do Azure para evitar cobranças desnecessárias.

## Revisão

Neste laboratório, você aprendeu a usar uma identidade gerenciada atribuída a agentes auto-hospedados em pipelines YAML do Azure DevOps.
