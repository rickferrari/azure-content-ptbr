<properties
    pageTitle="Solução Iniciar/Parar VMs fora do horário comercial [Visualização] na Automação | Microsoft Azure"
    description="As soluções de Gerenciamento de VM iniciam e param suas Máquinas Virtuais do Azure Resource Manager com agendamento e as monitoram proativamente no Log Analytics."
    services="automation"
    documentationCenter=""
    authors="MGoedtel"
    manager="jwhit"
    editor=""
	/>
<tags
    ms.service="automation"
    ms.workload="tbd"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="10/04/2016"
    ms.author="magoedte"/>

# Solução Iniciar/Parar VMs fora do horário comercial [Visualização] na Automação

A solução Iniciar/Parar VMs fora do horário comercial (Visualização) inicia e para máquinas virtuais do Azure Resource Manager em um agendamento definido pelo usuário e fornece informações sobre o sucesso dos trabalhos de Automação que iniciam e param as máquinas virtuais com o Log Analytics do OMS.

## Pré-requisitos

- O runbook funciona com uma [conta Executar como do Azure](automation-sec-configure-azure-runas-account.md). A conta Executar como é o método de autenticação preferido, pois ela usa a autenticação de certificado em vez de uma senha que pode expirar ou ser alterada com frequência.

- Essa solução só pode gerenciar VMs que estão na mesma assinatura e no mesmo grupo de recursos em que reside a conta de Automação.

- A solução só pode ser implantada nas seguintes regiões do Azure: Sudeste da Austrália, Leste dos EUA, Sudeste Asiático e Europa Ocidental. Os runbooks que gerenciam o agendamento de VM podem direcionar para VMs em qualquer região.

- Para enviar notificações por email ao concluir os runbooks Iniciar e Parar VMs, é necessário ter uma assinatura de classe executiva do Office 365.

## Componentes da solução

Essa solução consiste no recursos a seguir que serão importados e adicionados à sua conta de Automação.

### Runbooks

Runbook | Descrição|
--------|------------|
CleanSolution-MS-Mgmt-VM | Esse runbook remove todos os recursos contidos e agenda quando você vai excluir a solução da sua assinatura.|  
SendMailO365-MS-Mgmt | Esse runbook envia um email pelo Office 365 Exchange.|
StartByResourceGroup-MS-Mgmt-VM | Esse runbook destina-se a iniciar VMs (VMs clássicas e baseadas no ARM) que residem em determinada lista de grupos de recursos do Azure.
StopByResourceGroup-MS-Mgmt-VM | Esse runbook destina-se a parar VMs (VMs clássicas e baseadas no ARM) que residem em determinada lista de grupos de recursos do Azure.|
<br>

### Variáveis

Variável | Descrição|
---------|------------|
**SendMailO365-MS-Mgmt** Runbook ||
SendMailO365-IsSendEmail-MS-Mgmt | Especifica se os runbooks StartByResourceGroup-MS-Mgmt-VM e StopByResourceGroup-MS-Mgmt-VM podem enviar notificação por email após a conclusão. Selecione **True** para habilitar e **False** para desativar o alerta de email. O valor padrão é **False**.| 
Runbook **StartByResourceGroup-MS-Mgmt-VM** ||
StartByResourceGroup-ExcludeList-MS-Mgmt-VM | Insira nomes de VM a serem excluídos da operação de gerenciamento. Separe os nomes usando ponto-e-vírgula (;). Os valores diferenciam maiúsculas de minúsculas e há suporte para curingas (asterisco).|
StartByResourceGroup-SendMailO365-EmailBodyPreFix-MS-Mgmt | Texto que pode ser anexado ao início do corpo da mensagem de email.|
StartByResourceGroup-SendMailO365-EmailRunBookAccount-MS-Mgmt | Especifica o nome da Conta de Automação que contém o runbook Email. **Não modifique essa variável.**|
StartByResourceGroup-SendMailO365-EmailRunbookName-MS-Mgmt | Especifica o nome do runbook Email. Isso é usado pelos runbooks StartByResourceGroup-MS-Mgmt-VM e StopByResourceGroup-MS-Mgmt-VM para enviar email. **Não modifique essa variável.**|
StartByResourceGroup-SendMailO365-EmailRunbookResourceGroup-MS-Mgmt | Especifica o nome do Grupo de recursos que contém o runbook Email. **Não modifique essa variável.**|
StartByResourceGroup-SendMailO365-EmailSubject-MS-Mgmt | Especifica o texto da linha de assunto do email.|  
StartByResourceGroup-SendMailO365-EmailToAddress-MS-Mgmt | Especifica os destinatários do email. Digite nomes separados usando ponto-e-vírgula (;).|
StartByResourceGroup-TargetResourceGroups-MS-Mgmt-VM | Insira nomes de VM a serem excluídos da operação de gerenciamento. Separe os nomes usando ponto-e-vírgula (;). Os valores diferenciam maiúsculas de minúsculas e há suporte para curingas (asterisco). O valor padrão (asterisco) incluirá todos os grupos de recursos na assinatura.|
StartByResourceGroup-TargetSubscriptionID-MS-Mgmt-VM | Especifica a assinatura que contém as VMs a serem gerenciadas pela solução. Ela deve ser a mesma assinatura em que reside a conta de Automação da solução.|
**StopByResourceGroup-MS-Mgmt-VM** Runbook ||
StopByResourceGroup-ExcludeList-MS-Mgmt-VM | Insira nomes de VM a serem excluídos da operação de gerenciamento. Separe os nomes usando ponto-e-vírgula (;). Os valores diferenciam maiúsculas de minúsculas e há suporte para curingas (asterisco).|
StopByResourceGroup-SendMailO365-EmailBodyPreFix-MS-Mgmt | Texto que pode ser anexado ao início do corpo da mensagem de email.|
StopByResourceGroup-SendMailO365-EmailRunBookAccount-MS-Mgmt | Especifica o nome da Conta de Automação que contém o runbook Email. **Não modifique essa variável.**|
StopByResourceGroup-SendMailO365-EmailRunbookResourceGroup-MS-Mgmt | Especifica o nome do Grupo de recursos que contém o runbook Email. **Não modifique essa variável.**|
StopByResourceGroup-SendMailO365-EmailSubject-MS-Mgmt | Especifica o texto da linha de assunto do email.|  
StopByResourceGroup-SendMailO365-EmailToAddress-MS-Mgmt | Especifica os destinatários do email. Digite nomes separados usando ponto-e-vírgula (;).|
StopByResourceGroup-TargetResourceGroups-MS-Mgmt-VM | Insira nomes de VM a serem excluídos da operação de gerenciamento. Separe os nomes usando ponto-e-vírgula (;). Os valores diferenciam maiúsculas de minúsculas e há suporte para curingas (asterisco). O valor padrão (asterisco) incluirá todos os grupos de recursos na assinatura.|
StopByResourceGroup-TargetSubscriptionID-MS-Mgmt-VM | Especifica a assinatura que contém as VMs a serem gerenciadas pela solução. Ela deve ser a mesma assinatura em que reside a conta de Automação da solução.|  
<br>

### Agendas

Agenda | Descrição|
---------|------------|
StartByResourceGroup-agenda-MS-Mgmt | Agenda do runbook StartByResourceGroup.|
StopByResourceGroup-agenda-MS-Mgmt | Agenda do runbook StopByResourceGroup.|

### Credenciais

Credencial | Descrição|
-----------|------------|
O365Credential | Especifica uma conta de usuário válida do Office 365 para enviar email. Necessário apenas se a variável SendMailO365-IsSendEmail-MS-Mgmt é definida como **True**.

## Configuração

Execute as seguintes etapas para adicionar a solução Iniciar/Parar VMs fora do horário comercial [Visualização] à sua conta de Automação e configure as variáveis para personalizar a solução.

1. Na tela inicial no portal do Azure, selecione o bloco **Marketplace**. Se o bloco não está fixado em sua tela inicial, selecione **Novo** no painel de navegação esquerdo.
2. Na folha Marketplace, digite **Iniciar VM** na caixa de pesquisa e selecione a solução **Iniciar/Parar VMs fora do horário comercial [Visualização]** nos resultados da pesquisa.
3. Na folha **Iniciar/Parar VMs fora do horário comercial [Visualização]** da solução selecionada, examine as informações de resumo e clique em **Criar**.
4. A folha **Adicionar Solução** aparece e você é solicitado a configurar a solução antes de importá-la para sua assinatura da Automação.<br><br> ![Folha Adicionar Solução do Gerenciamento de VM](media/automation-solution-vm-management/vm-management-solution-add-solution-blade.png)<br><br>
5.  Na folha **Adicionar Solução** folha, selecione **Espaço de Trabalho** e selecione um espaço de trabalho do OMS que seja vinculado à mesma assinatura do Azure em que a conta de Automação está em ou crie um novo espaço de trabalho do OMS. Se você não tiver um espaço de trabalho do OMS, poderá selecionar **Criar Novo Espaço de Trabalho** e, na folha **Espaço de Trabalho do OMS**, faça o seguinte:
   - Especifique um nome para o novo **Espaço de Trabalho do OMS**.
   - Selecione uma **Assinatura** a vincular escolhendo uma na lista suspensa, se a selecionada por padrão não é adequada.
   - Em **Grupo de Recursos**, você pode selecionar um grupo de recursos existente ou criar um novo.
   - Selecione um **Local**. No momento, os únicos locais fornecidos para seleção são **Sudeste da Austrália**, **Leste dos EUA**, **Sudeste Asiático** e **Europa Ocidental**.
   - Selecione um **Tipo de preço**. A solução é oferecida em duas camadas: camada paga OMS e gratuita. A camada gratuita tem um limite de quantidade de dados coletados diariamente, de período de retenção e de minutos de tempo de execução do trabalho de runbook. A camada paga do OMS não tem um limite de quantidade de dados coletados diariamente.

        > [AZURE.NOTE]
        > While the Stadalone paid tier is displayed as an option, it is not applicable.  If you select it and proceed with the creation of this solution in your subscription, it will fail.  This will be addressed when this solution is officially released.<br>If you use this solution, it will only use automation job minutes and log ingestion.  The solution does not add additional OMS nodes to your environment.  

6. Depois de fornecer as informações necessárias na folha **Espaço de trabalho do OMS**, clique em **Criar**. Enquanto as informações são verificadas e o espaço de trabalho é criado, você pode acompanhar seu progresso no menu **Notificações**. Você será levado para a folha **Adicionar Solução**.
7. Na folha **Adicionar Solução**, selecione **Conta de Automação**. Se você estiver criando um novo espaço de trabalho do OMS, será necessário criar uma nova conta de Automação que será associada ao novo espaço de trabalho do OMS especificado anteriormente, incluindo a assinatura do Azure, o grupo de recursos e a região. Selecione **Criar uma conta de Automação** e, na folha **Adicionar Conta de Automação**, forneça o seguinte:
  - No campo **Nome**, digite o nome da conta de Automação.

    Todas as outras opções são preenchidas automaticamente com base no espaço de trabalho do OMS selecionado, e essas opções não podem ser modificadas. Uma conta Executar como do Azure é o método de autenticação padrão para os runbooks incluídos nesta solução. Após clicar em **OK**, as opções de configuração são validadas e a conta de Automação é criada. Você pode acompanhar o progresso em **Notificações** no menu.

    Caso contrário, você pode selecionar uma conta Executar como de Automação existente. Observe que a conta selecionada não pode já estar vinculada a outro espaço de trabalho do OMS; caso contrário, uma mensagem aparecerá na folha para informá-lo. Se já estiver vinculada, você precisará selecionar uma conta Executar como de Automação diferente ou criar uma nova.<br><br> ![Conta de Automação já vinculada ao Espaço de Trabalho do OMS](media/automation-solution-vm-management/vm-management-solution-add-solution-blade-autoacct-warning.png)<br>

8. Por fim, na folha **Adicionar Solução**, selecione **Configuração** e a folha **Parâmetros** será exibida. Na folha **Parâmetros**, você será solicitado a:
   - Especifique os **Nomes de Grupo de Recursos de Destino**, que é um nome de grupo de recursos que contém VMs a serem gerenciadas pela solução. Você pode inserir mais de um nome e separá-los usando ponto-e-vírgula (os valores diferenciam maiúsculas de minúsculas). O uso de um caractere curinga tem suporte para selecionar VMs em todos os grupos de recursos na assinatura.
   - Selecione uma **Agenda**, que é um conjunto de data e hora para iniciar e parar as VMs no grupo de recursos de destino.

10. Depois de ter concluído a configuração inicial necessária para a solução, selecione **Criar**. Todas as configurações serão validadas e ele tentará implantar a solução em sua assinatura. Esse processo pode levar vários segundos e você pode acompanhar seu progresso no menu **Notificações**.

## Frequência de coleta

Dados do fluxo do trabalho e do log do trabalho de automação são incluídos no repositório do OMS a cada cinco minutos.

## Usando a solução

Ao adicionar a solução de Gerenciamento de VMs ao seu espaço de trabalho do OMS, o bloco **Exibição StartStopVM** será adicionado ao painel do OMS. Esse bloco exibe uma contagem e uma representação gráfica dos trabalhos de runbooks para soluções iniciadas e concluídas com êxito.<br><br> ![Bloco de exibição StartStopVM de Gerenciamento de VM](media/automation-solution-vm-management/vm-management-solution-startstopvm-view-tile.png)

Na sua conta de Automação, você pode acessar e gerenciar a solução selecionando o bloco **Soluções** e a folha **Soluções**, e selecionando a solução **Start-Stop-VM [Workspace]** na lista.<br><br> ![Lista de soluções de automação](media/automation-solution-vm-management/vm-management-solution-autoaccount-solution-list.png)

A escolha da solução exibirá a folha da solução **Start-Stop-VM[Workspace]**, onde você pode examinar detalhes importantes, por exemplo, o bloco **StartStopVM**, como no espaço de trabalho do OMS, que exibe a contagem e a representação gráfica dos trabalhos de runbooks para a solução iniciada e concluída com êxito.<br><br> ![Folha de solução VM de Automação](media/automation-solution-vm-management/vm-management-solution-solution-blade.png)

### Configurando notificações por email

Para habilitar notificações por email quando os runbooks Iniciar e Parar VM forem concluídos, você precisará modificar a credencial **O365Credential** e, no mínimo, as seguintes variáveis:

 - SendMailO365-IsSendEmail-MS-Mgmt
 - StartByResourceGroup-SendMailO365-EmailToAddress-MS-Mgmt
 - StopByResourceGroup-SendMailO365-EmailToAddress-MS-Mgmt

Para configurar a credencial **O365Credential**, execute as seguintes etapas:

1. Da sua conta de automação, clique em **Todas as Configurações** na parte superior da janela.
2. Na folha **Configurações** na seção **Recursos de Automação**, selecione **Ativos**.
3. Na folha **Ativos**, selecione o bloco **Credencial** e, na folha **Credencial**, selecione **O365Credential**.
4. Insira um nome de usuário e uma senha válidos do Office 365 e clique em **Salvar** para salvar suas alterações.

Para configurar as variáveis destacadas anteriormente, execute as seguintes etapas:

1. Da sua conta de automação, clique em **Todas as Configurações** na parte superior da janela.
2. Na folha **Configurações** na seção **Recursos de Automação**, selecione **Ativos**.
3. Na folha **Ativos**, selecione o bloco **Variáveis** e, na folha **variáveis**, selecione a variável listada acima. Modifique o seu valor seguindo a descrição especificada na seção [Variável](##variables) mencionada anteriormente.
4. Clique em **Salvar** para salvar as alterações da variável.

## Registros do Log Analytics

A Automação cria dois tipos de registros no repositório do OMS.

### Logs de trabalho

Propriedade | Descrição|
----------|----------|
Chamador | Quem iniciou a operação. Os valores possíveis são um endereço de email ou o sistema para trabalhos agendados.|
Categoria | Classificação do tipo de dados. Para a Automação, o valor é JobLogs.|
CorrelationId | O GUID que é a Id de correlação do trabalho de runbook.|
JobId | GUID que é a Id do trabalho de runbook.|
operationName | Especifica o tipo de operação realizada no Azure. Para a Automação, o valor será Trabalho.|
resourceId | Especifica o tipo de recurso no Azure. Para a Automação, o valor é a conta da Automação associada ao runbook.|
ResourceGroup | Especifica o nome do grupo de recursos do trabalho do runbook.|
ResourceProvider | Especifica o serviço do Azure que fornece os recursos que você pode implantar e gerenciar. Para Automação, o valor é Automação do Azure.|
ResourceType | Especifica o tipo de recurso no Azure. Para a Automação, o valor é a conta da Automação associada ao runbook.|
resultType | O status do trabalho de runbook. Os valores possíveis são:<br>- Iniciado<br>- Parado<br>- Suspenso<br>- Falha<br>- Êxito|
resultDescription | Descreve o estado de resultado do trabalho de runbook. Os valores possíveis são:<br>- Trabalho Iniciado<br>- Falha no Trabalho<br>- Trabalho Concluído|
RunbookName | Especifica o nome do runbook.|
SourceSystem | Especifica o sistema de origem dos dados enviados. Em Automação, o valor será :OpsManager|
StreamType | Especifica o tipo de evento. Os valores possíveis são:<br>- Detalhado<br>- Saída<br>-Erro<br>- Aviso|
SubscriptionId | Especifica a ID da assinatura do trabalho.
Hora | Data e hora da execução do trabalho de runbook.|


### Transmissões de trabalho

Propriedade | Descrição|
----------|----------|
Chamador | Quem iniciou a operação. Os valores possíveis são um endereço de email ou o sistema para trabalhos agendados.|
Categoria | Classificação do tipo de dados. Para a Automação, o valor é JobStreams.|
JobId | GUID que é a Id do trabalho de runbook.|
operationName | Especifica o tipo de operação realizada no Azure. Para a Automação, o valor será Trabalho.|
ResourceGroup | Especifica o nome do grupo de recursos do trabalho do runbook.|
resourceId | Especifica o tipo de ID de recurso no Azure. Para a Automação, o valor é a conta da Automação associada ao runbook.|
ResourceProvider | Especifica o serviço do Azure que fornece os recursos que você pode implantar e gerenciar. Para Automação, o valor é Automação do Azure.|
ResourceType | Especifica o tipo de recurso no Azure. Para a Automação, o valor é a conta da Automação associada ao runbook.|
resultType | O resultado do trabalho de runbook no momento em que o evento foi gerado. Os valores possíveis são:<br>- InProgress|
resultDescription | Inclui o fluxo de saída do runbook.|
RunbookName | O nome do runbook.|
SourceSystem | Especifica o sistema de origem dos dados enviados. Em Automação, o valor será :OpsManager|
StreamType | O tipo de fluxo de trabalho. Os valores possíveis são:<br>-Progresso<br>- Saída<br>- Aviso<br>- Erro<br>- Depuração<br>- Detalhado|
Hora | Data e hora da execução do trabalho de runbook.|

Quando você executa uma pesquisa de log que retorna registros de categoria de **JobLogs** ou **JobStreams**, pode selecionar a exibição **JobLogs** ou **JobStreams** que exibe um conjunto de blocos resumindo as atualizações retornadas pela pesquisa.

## Pesquisas de log de exemplo

A tabela a seguir fornece pesquisas de log de exemplo para os registros de alerta coletados por essa solução.

Consultar | Descrição|
----------|----------|
Localizar trabalhos de runbook StartVM que foram concluídos com êxito | Category=JobLogs RunbookName\_s="StartByResourceGroup-MS-Mgmt-VM" ResultType=Succeeded | measure count() by JobId\_g|
Localizar trabalhos de runbook StopVM que foram concluídos com êxito | Category=JobLogs RunbookName\_s="StartByResourceGroup-MS-Mgmt-VM" ResultType=Failed | measure count() by JobId\_g
Mostrar o status do trabalho ao longo do tempo para runbooks StartVM e StopVM | Category=JobLogs RunbookName\_s="StartByResourceGroup-MS-Mgmt-VM" OR "StopByResourceGroup-MS-Mgmt-VM" NOT(ResultType="started") | measure Count() by ResultType interval 1day|

## Próximas etapas

- Para saber mais sobre como construir consultas de pesquisa diferentes e examinar os logs de trabalho de Automação com o Log Analytics, confira [Efetuar pesquisas no Log Analytics](../log-analytics/log-analytics-log-searches.md)
- Para saber mais sobre a execução de runbooks, como monitorar trabalhos de runbook e outros detalhes técnicos, confira [Acompanhar um trabalho de runbook](automation-runbook-execution.md)
- Para saber mais sobre o Log Analytics do OMS e fontes de coleta de dados, confira [Coletar dados do Armazenamento do Azure na visão geral do Log Analytics](../log-analytics/log-analytics-azure-storage.md)






   

<!---HONumber=AcomDC_1005_2016-->