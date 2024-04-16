---
lab:
  title: Configurar agentes e pools de agentes para pipelines seguros
  module: 'Module 3: Configure secure access to pipeline resources'
---

# Configurar agentes e pools de agentes para pipelines seguros

Neste laboratório, você vai aprender a configurar agentes e pools de agentes do Azure DevOps e a gerenciar permissões para esses pools. Os Pools de Agentes do Azure DevOps fornecem os recursos para executar seus pipelines de build e lançamento.

Estes exercícios levam aproximadamente **25** minutos.

## Antes de começar

Você precisará de uma assinatura do Azure, da organização do Azure DevOps e do aplicativo eShopOnWeb para acompanhar os laboratórios.

- Siga as etapas para [validar seu ambiente de laboratório](APL2001_M00_Validate_Lab_Environment.md).
- Token PAT para a configuração do agente.

## Instruções

Você criará agentes e configurará agentes auto-hospedados usando o Windows. Se você quiser configurar agentes no Linux ou MacOS, siga as instruções na [documentação do Azure DevOps](https://docs.microsoft.com/azure/devops/pipelines/agents/v2-linux).

Durante a configuração, tenha em mente o seguinte:

- **Manter agentes separados por projeto**: cada agente só pode ser vinculado a um pool. Embora o compartilhamento de pools de agentes entre projetos possa economizar nos custos de infraestrutura, ele também cria o risco de movimentação lateral. Portanto, é melhor ter pools de agentes separados com agentes dedicados para cada projeto para evitar a contaminação cruzada.
- **Utilize contas com privilégios baixos para executar agentes**: executar um agente com uma identidade com acesso direto aos recursos do Azure DevOps pode representar ameaças à segurança. Operar o agente em uma conta local não privilegiada, como o Serviço de Rede, é aconselhável, o que minimiza o risco.
- **Cuidado com nomes de grupo enganosos**: o grupo "Contas de Serviço de Coleta de Projeto" no Azure DevOps é um risco potencial à segurança. A execução de agentes usando uma identidade que faz parte desse grupo e é apoiada pelo Azure AD pode comprometer a segurança de toda a sua organização do Azure DevOps.
- **Evite contas com privilégios altos para agentes auto-hospedados**: o uso de contas com privilégios altos para executar agentes auto-hospedados, especialmente para acessar segredos ou ambientes de produção, pode expor seu sistema a ameaças graves se um pipeline for comprometido.
- **Priorize a segurança**: para proteger seus sistemas, use a conta com menos privilégios para executar agentes auto-hospedados. Por exemplo, considere usar sua conta de computador ou uma identidade de serviço gerenciado. Também é aconselhável permitir que o Azure Pipelines manipule o acesso a segredos e ambientes.

### Exercício 1: criar agentes e configurar pools de agentes

Neste exercício, você criará um agente e configurará pools de agentes.

#### Tarefa 1: criar um pool de agentes

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb** e selecione **Configurações do projeto** no menu inferior esquerdo.

1. Em **Pipelines > Pools de agentes**, selecione o botão **Adicionar pool**.

1. Escolha o tipo de pool **Auto-hospedado**.

1. Forneça um nome para o pool de agentes, como **eShopOnWebSelfPool**, e adicione uma descrição opcional.

1. Deixe a opção **Conceder permissão de acesso a todos os pipelines** desmarcada.

    ![Captura de tela mostrando opções de adicionar pool de agentes com o tipo auto-hospedado.](media/create-new-agent-pool-self-hosted-agent.png)

1. Selecione o botão **Criar** para criar o pool de agentes.

#### Tarefa 2: criar um agente

1. Selecione o pool de agentes recém-criado e, em seguida, selecione a guia **Agentes**.

1. Selecione o botão **Novo agente** e, em seguida, o botão **Baixar** em **Baixar agente** na nova janela pop-up.

1. Siga as instruções de instalação para instalar o agente em sua máquina a partir da janela pop-up.
   1. Execute os seguintes comandos do Powershell para criar uma nova pasta de agente em sua máquina.

        ```powershell
        mkdir agent ; cd agent        
        ```

        > [!NOTE]
        > Verifique se você está na pasta raiz do seu perfil de usuário ou na pasta onde deseja instalar o agente.

   2. Se você escolher a pasta **Download** em sua máquina, no Powershell, execute o comando sugerido:

        ```powershell
        Add-Type -AssemblyName System.IO.Compression.FileSystem ; [SysteIO.Compression.ZipFile]::ExtractToDirecto("$HOME\Downloads\vsts-agent-win-x64-3.220.2.zip", "$PWD")
        
        ```
        > [!NOTE]
        > Se você baixou o agente em um local diferente, substitua o caminho no comando acima.

#### Tarefa 3: criar um token PAT

Antes de configurar seu agente, crie um novo token PAT ou escolha um existente. Siga as etapas abaixo para criar um novo token PAT:

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Navegue até o projeto eShopOnWeb e selecione **Configurações do usuário** no menu superior direito (à esquerda da foto do perfil do usuário).

1. Selecione o menu **Tokens de acesso pessoal**.

    ![Captura de tela mostrando o menu de tokens de acesso pessoal.](media/personal-access-token-menu.png)

1. Selecione o botão **Novo Token**.

1. Forneça um nome para o token, como **eShopOnWebToken**.

1. Selecione a organização do Azure DevOps na qual você deseja usar o token.

1. Defina a data de validade do token (usado apenas para configurar o agente).

1. Selecione o escopo personalizado definido.

1. Selecione para mostrar todos os escopos.

1. Selecione o escopo **Pools de Agentes (Ler e Gerenciar)**.

1. Selecione o botão **Criar** para criar o token.

1. Copie o valor do token e salve-o em um local seguro (você não poderá vê-lo novamente. Você só pode regenerar o token).

    ![Captura de tela mostrando a configuração do token de acesso pessoal.](media/personal-access-token-configuration.png)

    > [!IMPORTANT]
    > Use a última opção de privilégio, **Pools de Agentes (Ler e Gerenciar)**, somente para a configuração do agente. Além disso, certifique-se de definir a data de validade mínima para o token, se for a única finalidade para o token. Você pode criar um novo token com os mesmos privilégios se precisar configurar o agente novamente.

#### Tarefa 4: configurar o agente

1. Abra uma nova janela do Powershell e navegue até a pasta do agente criada na etapa anterior.

1. Para configurar o seu agente, execute o seguinte comando:

    ```powershell
    .\config.cmd
    ```

    > [!NOTE]
    > Opcionalmente, execute o agente interativamente executando .\run.cmd. Não é possível fechar a janela do prompt de comando durante a execução interativa.

1. Insira as seguintes informações quando solicitado a configurar o agente:
    - Insira a URL da organização do Azure DevOps: `https://dev.azure.com/`{nome da sua organização}.
    - Escolha o tipo de autenticação: **PAT**.
    - Insira o valor do token PAT que você criou na etapa anterior.
    - Insira o nome do pool de agentes **eShopOnWebSelfPool** que você criou na etapa anterior.
    - Digite o nome do agente **eShopOnWebSelfAgent**.
    - Escolha a pasta de trabalho do agente (o padrão é _work).
    - Escolha o modo de execução do agente (Y para executar como serviço).
    - Digite Y para habilitar SERVICE_SID_TYPE_UNRESTRICTED para o serviço de agente (somente Windows).
    - Insira a conta de usuário a ser usada para o serviço.

        > [!IMPORTANT]
        > Para executar o serviço de agente, evite usar contas com privilégios elevados. Em vez disso, empregue uma conta com poucos privilégios que mantenha as permissões mínimas necessárias para a operação do serviço. Essa abordagem ajuda a manter um ambiente seguro e estável.

    - Insira se o serviço deve ser iniciado imediatamente após a conclusão da configuração (N para iniciar o serviço).

        ![Captura de tela mostrando a configuração do agente.](media/agent-configuration.png)

    - Verifique o status do agente navegando até o pool de agentes e clicando na aba **Agentes** . Você deve ver o novo agente na lista.

        ![Captura de tela mostrando o status do agente.](media/agent-status.png)

Para obter mais detalhes sobre agentes do Windows, consulte: [Agentes do Windows auto-hospedados](https://learn.microsoft.com/azure/devops/pipelines/agents/windows-agent)

### Exercício 2: criar e configurar um novo grupo de segurança para o pool de agentes

Neste exercício, você criará um novo grupo de segurança para o pool de agentes.

#### Tarefa 1: criar um novo grupo de segurança

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb** e selecione **Configurações do projeto** no menu inferior esquerdo.

1. Abra Permissões em Geral.

1. Selecione o botão **Novo Grupo**.

1. Forneça um nome para o grupo, como **Grupo de Segurança eShopOnWeb**.

1. Adicione os usuários que você deseja que façam parte do grupo.

1. Selecione o botão **Criar** para criar o grupo.

    ![Captura de tela mostrando a tela de criação do grupo de segurança.](media/create-security-group.png)

#### Tarefa 2: configurar o grupo de segurança

1. Selecione o novo grupo para exibir sua guia **Permissões**.

1. Negar permissões desnecessárias para o grupo, como **Renomear projeto de equipe**, **Excluir permanentemente itens de trabalho** ou quaisquer outras permissões que você não deseja no grupo, pois ele é usado apenas para o pool de agentes.

    ![Captura de tela mostrando as configurações do grupo de segurança.](media/security-group-settings.png)

    > [!IMPORTANT]
    > Se você deixar permissões que não deseja no grupo, scripts ou tarefas em execução no agente poderão usar as permissões de grupo para executar ações que você não deseja que eles executem.

### Exercício 3: gerenciar permissões do pool de agentes

Neste exercício, você gerenciará permissões para o pool de agentes.

1. Navegue até o portal do Azure DevOps em `https://dev.azure.com` e abra sua organização.

1. Abra o projeto **eShopOnWeb** e selecione **Configurações do projeto** no menu inferior esquerdo.

1. Selecione **Pipelines** e, em seguida, **Pools de agentes**.

1. Selecione o pool de agentes **eShopOnWebSelfPool**.

1. Na exibição de detalhes do pool de agentes, selecione a guia **Segurança**.

1. Selecione o botão **Adicionar** e adicione o novo grupo **Grupo de Segurança eShopOnWeb** às permissões de usuário do pool de agentes.

1. Escolha a função apropriada para o usuário ou grupo, como Leitor do Pool de Agentes, Usuário ou Administrador. Nesse caso, escolha **Usuário**.

1. Selecione Adicionar para aplicar as permissões.

    ![Captura de tela mostrando a configuração de segurança do pool de agentes.](media/agent-pool-security.png)

Agora você está pronto para usar com segurança o pool de agentes em seus pipelines. Para obter mais detalhes sobre pools de agentes, consulte: [Pools de agentes](https://learn.microsoft.com/azure/devops/pipelines/agents/pools-queues).

### Exercício 4: remover os recursos usados no laboratório

1. Pare e remova o serviço do agente executando `.config.cmd remove`.

1. Exclua o pool de agentes.

1. Exclua o grupo de segurança.

1. Revogar o token PAT.

## Revisão

Neste laboratório, saiba como configurar um agente auto-hospedado e pools de agentes do Azure DevOps e gerenciar permissões para esses pools. Ao gerenciar permissões de forma eficaz, você pode garantir que os usuários certos tenham acesso aos recursos de que precisam e, ao mesmo tempo, manter a segurança e a integridade de seus processos de DevOps.
