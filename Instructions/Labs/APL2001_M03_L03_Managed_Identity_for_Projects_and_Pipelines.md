---
lab:
  title: Identidade gerenciada para projetos e pipelines
  module: 'Module 3: Manage identity for projects, pipelines, and agents'
---

# Identidade gerenciada para projetos e pipelines

As identidades gerenciadas oferecem um método seguro para controlar o acesso aos recursos do Azure. O Azure manipula essas identidades automaticamente, permitindo que você verifique o acesso a serviços compatíveis com a autenticação do Azure AD. Isso significa que você não precisa incorporar credenciais ao seu código, o que aumenta a segurança. No Azure DevOps, as identidades gerenciadas podem autenticar recursos do Azure em seus agentes auto-hospedados, simplificando o controle de acesso sem comprometer a segurança.

Neste laboratório, você criará uma identidade gerenciada e a usará em pipelines YAML do Azure DevOps em execução em agentes auto-hospedados para implantar recursos do Azure.

O laboratório leva cerca de **45** minutos.

## Antes de começar

Você precisará de uma assinatura do Azure, da organização do Azure DevOps e do aplicativo eShopOnWeb para acompanhar os laboratórios.

- Siga as etapas para [validar seu ambiente de laboratório](APL2001_M00_Validate_Lab_Environment.md).
- Verifique se você tem uma conta Microsoft ou uma conta do Microsoft Entra com a função de Colaborador ou Proprietário na assinatura do Azure. Para obter detalhes, veja [Listar designações de função do Azure usando o portal do Azure](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) e [Exibir e designar funções de administrador no Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).
- Conclua o laboratório [Configurar agentes e pools de agentes para pipelines seguros](APL2001_M03_L02_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md).

## Instruções

### Exercício 1: Importar e executar pipelines de CI/CD

Neste exercício, você importará e executará o pipeline de CI, configurará a conexão de serviço com sua Assinatura do Azure e, em seguida, importará e executará o pipeline de CD.

#### Tarefa 1: importar e executar o pipeline de CI

Vamos começar importando o pipeline de CI chamado [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb**.

1. Acesse **Pipelines > Pipelines**.

1. Selecione o botão **Criar Pipeline**.

1. Selecione **Git do Azure Repos (Yaml)**.

1. Selecione o repositório **eShopOnWeb**.

1. Selecione **Arquivo YAML existente do Azure Pipelines**.

1. Selecione o arquivo **/.ado/eshoponweb-ci.yml** e clique em **Continuar**.

1. Clique no botão **Executar** para executar o pipeline.

   > [!NOTE]
   > Seu pipeline assumirá um nome com base no nome do projeto. Renomeie-o para identificar melhor o pipeline.

1. Vá para **Pipelines > Pipelines**, selecione o pipeline criado recentemente, selecione as reticências e selecione a opção **Renomear/mover**.

1. Nomeie-o **eshoponweb-ci** e selecione **Salvar**.

#### Tarefa 2: importar e executar o pipeline de CD

> [!NOTE]
> Nesta tarefa, você importará e executará o pipeline de CD chamado [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. No painel **Pipelines** do projeto **eShopOnWeb**, selecione o botão **Novo pipeline**.

1. Selecione **Git do Azure Repos (Yaml)**.

1. Selecione o repositório **eShopOnWeb**.

1. Selecione **Arquivo YAML existente do Azure Pipelines**.

1. Selecione o arquivo **/.ado/eshoponweb-cd-webapp-code.yml** e selecione **Continuar**.

1. Na definição do pipeline YAML, defina a seção de variáveis como:

   ```yaml
   variables:
     resource-group: 'AZ400-EWebShop-NAME'
     location: 'southcentralus'
     templateFile: '.azure/bicep/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'azure subs'
     webappname: 'az400-webapp-NAME'
   ```

1. Na seção de variáveis, substitua os espaços reservados pelos valores a seguir:

   - **AZ400-EWebShop-NAME** com o nome de sua preferência, por exemplo, **rg-eshoponweb**.
   - **local** com o nome da região do Azure que deseja implantar seus recursos, por exemplo, **southcentralus**.
   - **YOUR-SUBSCRIPTION-ID** com sua ID de assinatura do Azure.
   - **az400-webapp-NAME** por um nome globalmente exclusivo do aplicativo Web a ser implantado, por exemplo, a cadeia de caracteres **eshoponweb-lab-id-** seguida por um número aleatório de seis dígitos. 

1. Selecione **Salvar e executar** e escolha fazer commit diretamente na ramificação principal.

1. Selecione **Salvar e executar** novamente.

1. Abra o pipeline. Se você vir a mensagem "Este pipeline precisa de permissão para acessar um recurso antes que essa execução possa continuar a Implantar no WebApp", selecione **Exibir**, **Permitir** e **Permitir** novamente. Isso é necessário para permitir que o pipeline crie o recurso do Serviço de Aplicativo do Azure.

   ![Captura de tela do acesso de permissão do pipeline YAML.](media/pipeline-deploy-permit-resource.png)

1. A implantação pode levar alguns minutos para ser concluída, aguarde a execução do pipeline. A definição de CD consiste nas seguintes tarefas:

   - **AzureResourceManagerTemplateDeployment**: Implanta o aplicativo Web do Serviço de Aplicativo do Azure usando o modelo bicep.
   - **AzureRmWebAppDeployment**: Publica o site da Web no aplicativo Web do Serviço de Aplicativo do Azure.

   > [!NOTE]
   > Caso a implantação falhe, navegue até a página de execução do pipeline e selecione **Executar novamente trabalhos com falha** para invocar outra execução de pipeline.

   > [!NOTE]
   > Seu pipeline assumirá um nome com base no nome do projeto. Vamos **renomeá-lo** para identificar melhor o pipeline.

1. Vá para **Pipelines > Pipelines**, selecione o pipeline criado recentemente, selecione as reticências e selecione a opção **Renomear/mover**.

1. Nomeie-o **eshoponweb-cd-webapp-code** e selecione **Salvar**.

### Exercício 2: Configurar identidade gerenciada em pipelines do Azure

Neste exercício, você usará uma identidade gerenciada para configurar uma nova conexão de serviço e incorporá-la aos pipelines de CI/CD.

#### Tarefa 1: Configurar um agente auto-hospedado para usar a identidade gerenciada e atualizar o pipeline de CI

1. No seu navegador, abra o portal do Azure em `https://portal.azure.com`.

1. No portal do Azure, navegue até a página exibindo a VM do Azure **eshoponweb-vm** que você implantou neste laboratório

1. Na página da VM do Azure **eshoponweb-vm**, na barra de ferramentas, selecione **Iniciar** para iniciá-la.

1. Na página da VM do Azure **eshoponweb-vm**, no menu vertical no lado esquerdo, na seção **Segurança**, selecione **Identidade**.

1. Na página **eshoponweb-vm /| Identidade**, verifique se o **Status** é **Ativado** e selecione **Atribuições de função do Azure**.

1. Selecione o botão **Adicionar atribuição de função** e execute as seguintes ações:

   | Configuração | Ação |
   | -- | -- |
   | Lista suspensa **Escopo**. | Selecione a **Assinatura**. |
   | Lista suspensa **Assinatura** | Selecione sua assinatura do Azure. |
   | Lista suspensa **Função** | Selecione a função **Colaborador**. |

   > [!NOTE]
   > O escopo da assinatura é necessário para acomodar implantações nos laboratórios subsequentes.

1. Selecione o botão **Salvar**.

    ![Captura de tela do painel Adicionar Atribuição](media/add-role-assignment.png)

#### Tarefa 2: Criar uma conexão de serviço com base em identidade gerenciada

1. Alterne para o navegador da Web exibindo o projeto **eShopOnWeb** no portal do Azure DevOps em `https://dev.azure.com`.

1. No projeto **eShopOnWeb**, navegue até **Configurações de projeto > Conexões de serviço**.

1. Selecione o botão **Nova conexão de serviço** e selecione **Azure Resource Manager**.

1. Selecione **Identidade Gerenciada** como o **Método de autenticação**.

1. Defina o nível de escopo para **Assinatura** e forneça as informações coletadas durante a validação da fase do ambiente de laboratório, incluindo ID da assinatura, nome da assinatura e ID do locatário. 

1. Em **Nome da conexão do serviço**, digite **subs do Azure gerenciados**. Esse nome será referenciado em pipelines YAML ao acessar a sua assinatura do Azure.

1. Selecione **Salvar**.

#### Tarefa 3: Atualizar o pipeline de CD

1. Alterne para o navegador da Web exibindo o projeto **eShopOnWeb** no portal do Azure DevOps.

1. Na página do projeto **eShopOnWeb**, navegue até **Pipelines > Pipelines**.

1. Selecione o pipeline **eshoponweb-cd-webapp-code** e selecione **Editar**.

1. Na seção variáveis, atualize a variável **serviceConnection** para usar o nome da conexão de serviço que você criou na tarefa anterior, **azure subs managed**.

   ```yaml
     azureserviceconnection: 'azure subs managed'
   ```

1. Na subseção **trabalhos** da seção **fases**, atualize o valor da propriedade **pool** para referenciar o pool de agentes auto-hospedados que você criou no laboratório anterior, **eShopOnWebSelfPool**, para que ele tenha o seguinte formato:

   ```yaml
     jobs:
     - job: Deploy
       pool: eShopOnWebSelfPool
       steps:
       #download artifacts
       - download: eshoponweb-ci
   ```

1. Selecione **Salvar** e escolha fazer commit diretamente na ramificação principal.

1. Selecione **Salvar** novamente.

1. Selecione **Executar** o pipeline e clique em **Executar** novamente.

1. Abra o pipeline. Se você vir a mensagem "Este pipeline precisa de permissão para acessar um recurso antes que essa execução possa continuar a Implantar no WebApp", selecione **Exibir**, **Permitir** e **Permitir** novamente. Isso é necessário para permitir que o pipeline crie o recurso do Serviço de Aplicativo do Azure.

1. A implantação pode levar alguns minutos para ser concluída, aguarde a execução do pipeline.

1. Você deve ver nos logs de pipeline que o pipeline está usando a identidade gerenciada.

   ![Captura de tela dos logs de pipeline usando a identidade gerenciada.](media/pipeline-logs-managed-identity.png)

   > [!NOTE]
   > Após a conclusão do pipeline, você pode usar o portal do Azure para verificar o estado dos recursos do aplicativo Web do Serviço de Aplicativo.

### Exercício 3: Executar a limpeza dos recursos do Azure e do Azure DevOps

Neste exercício, você executará a limpeza pós-laboratório dos recursos do Azure e do Azure DevOps criados neste laboratório.

#### Tarefa 1: Parar e desalocar a VM do Azure

1. Navegue até a página exibindo o grupo de recursos **rg-eshoponweb** e selecione **Excluir grupo de recursos** para excluir todos os recursos dele.

   > [!IMPORTANT]
   > Não exclua o grupo de recursos **rg-eshoponweb-agentpool** que contém os recursos do agente auto-hospedado. Você precisará dele na próxima tarefa.

1. No portal do Azure, navegue até a página exibindo a VM do Azure **eshoponweb-vm** que você implantou neste laboratório

1. Na página da VM do Azure **eshoponweb-vm**, na barra de ferramentas, selecione **Parar** para pará-la e desalocá-la.

#### Tarefa 2: remover pipelines do Azure DevOps

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb**.

1. Acesse **Pipelines > Pipelines**.

1. Vá para **Pipelines > Pipelines** e exclua os pipelines existentes.

   > [!IMPORTANT]
   > Não exclua a conexão de serviço e o pool de agentes que você criou neste laboratório. Você precisará deles na próxima tarefa.

#### Tarefa 3: Recriar o repositório do Azure DevOps

1. No portal do Azure DevOps, no projeto **eShopOnWeb**, selecione **Configurações do projeto** no canto inferior esquerdo.

1. No menu vertical **Configurações do projeto** ao lado esquerdo, na seção **Repositórios**, selecione **Repositórios**.

1. No painel **Todos os Repositórios**, passe o mouse sobre a extremidade direita da entrada do repositório **eShopOnWeb** até que o ícone de reticências **Mais opções** apareça. Selecione-o e, no menu **Mais opções**, selecione **Renomear**.  

1. Na janela **Renomear o repositório eShopOnWeb**, na caixa de texto **Nome do repositório**, insira **eShopOnWeb_old** e selecione **Renomear**.

1. De volta ao painel **Todos os Repositórios**, selecione **+ Criar**.

1. No painel **Criar um repositório**, na caixa de texto **Nome do repositório**, insira **eShopOnWeb**, desmarque a caixa de seleção **Adicionar um LEIAME** e selecione **Criar**.

1. De volta ao painel **Todos os Repositórios**, passe o mouse sobre a extremidade direita da entrada do repositório **eShopOnWeb_old** até que o ícone de reticências **Mais opções** apareça. Selecione-o e, no menu **Mais opções**, selecione **Excluir**.  

1. Na janela **Excluir o repositório eShopOnWeb_old**, insira **eShopOnWeb_old** e selecione **Excluir**.

1. No menu de navegação esquerdo do portal do Azure DevOps, selecione **Repositórios**.

1. No painel o **eShopOnWeb está vazio. Adicione algum código!** no painel, selecione **Importar um repositório**.

1. Na janela **Importar um repositório do Git**, cole a seguinte URL `https://github.com/MicrosoftLearning/eShopOnWeb` e selecione **Importar**:

## Revisão

Neste laboratório, você aprendeu a usar uma identidade gerenciada atribuída a agentes auto-hospedados em pipelines YAML do Azure DevOps.