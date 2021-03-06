
<properties
   pageTitle="Segurança de cluster do Service Fabric: funções de cliente | Microsoft Azure"
   description="Este artigo descreve as duas funções de clientes e as permissões fornecidas para as funções."
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="coreysa"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/14/2016"
   ms.author="subramar"/>



# Controle de acesso baseado em função para clientes do Service Fabric

O Service Fabric dá suporte a dois tipos de controle de acesso diferentes para clientes conectados a um cluster do Service Fabric: administrador e usuário. O controle de acesso permite que o administrador de cluster limite o acesso a determinadas operações de cluster para diferentes grupos de usuários, tornando o cluster mais seguro.

Os **administradores** têm acesso completo aos recursos de gerenciamento (incluindo recursos de leitura/gravação). Por padrão, os **usuários** têm apenas acesso de leitura aos recursos de gerenciamento (por exemplo, recursos de consulta) e a capacidade de resolver serviços e aplicativos.

As duas funções de clientes (administrador ou cliente) são especificadas no momento da criação do cluster, com o fornecimento de certificados separados para cada um. Confira [Service Fabric cluster security](service-fabric-cluster-security.md) (Segurança de cluster do Service Fabric) para obter detalhes sobre como configurar um cluster do Service Fabric.


## Configurações padrão de controle de acesso


O tipo de controle de acesso de administrador tem acesso total a todas as APIs FabricClient. Com ele, é possível executar qualquer operação de leitura e gravação no cluster do Service Fabric, incluindo as seguintes:


### Operações de aplicativos e serviço
* **CreateService**: criação de serviço
* **CreateServiceFromTemplate**: criação de serviço de um modelo
* **UpdateService**: atualizações de serviço
* **DeleteService**: exclusão de serviço
* **ProvisionApplicationType**: provisionamento do tipo de aplicativo
* **CreateApplication**: criação de aplicativos
* **DeleteApplication**: exclusão de aplicativos
* **UpgradeApplication**: iniciar ou interromper atualizações de aplicativo
* **UnprovisionApplicationType**: desprovisionar o tipo de aplicativo
* **MoveNextUpgradeDomain**: retomar as atualizações de aplicativo com um domínio de atualização explícito
* **ReportUpgradeHealth**: retomar as atualizações de aplicativo com o andamento da atualização atual
* **ReportHealth**: relatório de integridade
* **PredeployPackageToNode**: API de pré-implantação
* **CodePackageControl**: reiniciar pacotes de código
* **RecoverPartition**: recuperar uma partição
* **RecoverPartitions**: recuperar partições
* **RecoverServicePartitions**: recuperar partições de serviço
* **RecoverSystemPartitions**: recuperar partições de serviço do sistema


### Operações de cluster
* **ProvisionFabric**: MSI e/ou provisionamento de manifesto do cluster
* **UpgradeFabric**: inicialização de atualizações de cluster
* **UnprovisionFabric**: MSI e/ou desprovisionamento de manifesto do cluster
* **MoveNextFabricUpgradeDomain**: retomar as atualizações de cluster com um domínio de atualização explícito
* **ReportFabricUpgradeHealth**: retomar as atualizações de cluster com o progresso de atualização atual
* **StartInfrastructureTask**: iniciar tarefas de infraestrutura
* **FinishInfrastructureTask**: concluir tarefas de infraestrutura
* **InvokeInfrastructureCommand**: comandos de gerenciamento de tarefas de infraestrutura
* **ActivateNode**: ativar um nó
* **DeactivateNode**: desativar um nó
* **DeactivateNodesBatch**: desativar múltiplos nós
* **RemoveNodeDeactivations**: desativar reversão em vários nós
* **GetNodeDeactivationStatus**: verificar o status de desativação
* **NodeStateRemoved**: relatar o estado do nó removido
* **ReportFault**: falha de relatórios
* **FileContent**: transferência de arquivos de cliente de repositório de imagens (externo ao cluster)
* **FileDownload**: inicialização de download de arquivos de cliente de repositório de imagens (externo ao cluster)
* **InternalList**: operação de lista de arquivos de cliente de repositório de imagens (interno)
* **Delete**: operação de exclusão do cliente de repositório de imagens
* **Upload**: operação de upload do cliente de repositório de imagens
* **NodeControl**: iniciar, interromper e reiniciar nós
* **MoveReplicaControl**: mover réplicas de um nó para outro

### Operações diversas
* **Ping**: pings em cliente
* **Query**: todas as consultas permitidas
* **NameExists**: verificações de existência do URI de nomenclatura



O tipo de Controle de Acesso de Usuários é, por padrão, limitado às operações a seguir:

* **EnumerateSubnames**: nomenclatura de enumeração de URI
* **EnumerateProperties**: nomenclatura de enumeração de propriedade
* **PropertyReadBatch**: nomenclatura de operações de leitura de propriedade
* **GetServiceDescription**: notificações de serviço de sondagem longa e descrições do serviço de leitura
* **ResolveService**: resolução de serviço baseada em reclamações
* **ResolveNameOwner**: resolver o proprietário do URI de nomenclatura
* **ResolvePartition**: resolver serviços do sistema
* **ServiceNotifications**: notificações de serviço baseadas em evento
* **GetUpgradeStatus**: status de atualização do aplicativo de sondagem
* **GetFabricUpgradeStatus**: status de atualização do cluster de sondagem
* **InvokeInfrastructureQuery**: tarefas da infraestrutura de consulta
* **List**: operação de lista de arquivos de cliente de repositório de imagens
* **ResetPartitionLoad**: redefinir a carga de uma Unidade de Failover
* **ToggleVerboseServicePlacementHealthReporting**: alternar o posicionamento do relatório de integridade do serviço detalhado

O Controle de Acesso de administrador também tem acesso às operações anteriores.

## Alterando as configurações padrão para funções do cliente

No arquivo de manifesto do cluster,você pode fornecer recursos de administração ao cliente, se for necessário. Você pode alterar os padrões ao acessar a opção **Configurações da Malha** durante a [criação do cluster](service-fabric-cluster-creation-via-portal.md) e fornecer as configurações anteriores nos campos **nome**, **administrador**, **usuário** e **valor**.

## Próximas etapas

[Segurança do Cluster do Service Fabric](service-fabric-cluster-security.md)

[Criação de cluster do Service Fabric](service-fabric-cluster-creation-via-portal.md)

<!---HONumber=AcomDC_0921_2016-->