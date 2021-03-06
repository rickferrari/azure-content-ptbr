<properties
	pageTitle="Migrar no Resource Manager com o PowerShell | Microsoft Azure"
	description="Este artigo apresenta a migração de recursos IaaS com suporte da plataforma do clássico ao Azure Resource Manager usando os comandos do Azure PowerShell"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="danlep"/>

# Migrar recursos de IaaS do Clássico para o Azure Resource Manager usando o Azure PowerShell

Estas etapas mostram como usar os comandos do Azure PowerShell para migrar os recursos de IaaS (infraestrutura como serviço) do modelo de implantação clássico para o Modelo de implantação do Azure Resource Manager. Estas etapas seguem uma abordagem de preencher lacunas para migrar seu ambiente personalizado. Assimile os comandos e substitua seus próprios valores pelas variáveis (as linhas que começam com "$").

Se quiser, você também pode migrar recursos usando a [Interface de linha de comando do Azure (CLI do Azure)](virtual-machines-linux-cli-migration-classic-resource-manager.md).

Para obter informações sobre cenários de migração com suporte, confira [Migração de recursos de IaaS com suporte da plataforma do clássico ao Azure Resource Manager](virtual-machines-windows-migration-classic-resource-manager.md). Para obter orientação e um passo a passo sobre a migração, confira [Análise técnica aprofundada sobre a migração com suporte da plataforma do clássico ao Azure Resource Manager](virtual-machines-windows-migration-classic-resource-manager-deep-dive.md).

## Etapa 1: Planejar a migração

Veja a seguir algumas das práticas que recomendamos durante a avaliação de migração dos recursos de IaaS do modelo clássico para o Gerenciador de Recursos:

- Leia os [recursos e configurações com e sem suporte](virtual-machines-windows-migration-classic-resource-manager.md). Caso você tenha máquinas virtuais que usam recursos ou configurações sem suporte, recomendamos aguardar até que o suporte do recurso/configuração seja anunciado. Como alternativa, se isso atender às suas necessidades, remova esse recurso ou mude a configuração para habilitar a migração.
-	Se você tiver scripts automatizados que implantam sua infraestrutura e aplicativos atualmente, tente criar uma configuração de teste semelhante usando esses scripts para migração. Você também pode configurar ambientes de exemplo usando o portal do Azure.

>[AZURE.IMPORTANT]Gateways de rede virtual (site a site, ExpressRoute do Azure, gateway de aplicativo, ponto a site) não têm suporte no momento para a migração do clássico para o Resource Manager. Para migrar uma rede virtual clássica com um gateway, remova o gateway antes de executar uma operação de Confirmação para mover a rede. (Você pode executar uma etapa de Preparação sem excluir o gateway.) Em seguida, depois de concluir a migração, reconecte o gateway no Azure Resource Manager.

## Etapa 2: instalar a versão mais recente do Azure PowerShell

Há duas opções principais para instalar o Azure PowerShell, a [Galeria do PowerShell](https://www.powershellgallery.com/profiles/azure-sdk/) e o [WebPI (Web Platform Installer)](http://aka.ms/webpi-azps). WebPI recebe atualizações mensais. A Galeria do PowerShell receberá atualizações continuamente. Este artigo tem base no Azure PowerShell versão 2.1.0.

Para obter instruções de instalação, consulte [Como instalar e configurar o Azure PowerShell](../powershell-install-configure.md).

## Etapa 3: Definir sua assinatura e se inscrever para a migração

Primeiro, inicie um prompt do PowerShell. Para cenários de migração, é necessário instalar seu ambiente tanto para o modelo clássico quanto para o Gerenciador de Recursos.

Entre em sua conta para o modelo do Gerenciador de Recursos.

	Login-AzureRmAccount

Obtenha as assinaturas disponíveis usando o comando a seguir:

	Get-AzureRMSubscription | Sort SubscriptionName | Select SubscriptionName

Defina sua assinatura do Azure para a sessão atual. Substitua tudo que estiver entre aspas, incluindo os caracteres < e >, pelo nome correto.

	Select-AzureRmSubscription –SubscriptionName "<subscription name>"

>[AZURE.NOTE] O registro é uma etapa única, mas é preciso executá-lo uma vez antes de tentar a migração. Sem o registro, você verá a seguinte mensagem de erro:

>	*BadRequest : Subscription is not registered for migration.* 

Registre-se no provedor de recursos de migração usando o comando a seguir:
	
	Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate

Aguarde cinco minutos para concluir o registro. É possível verificar o status da aprovação usando o comando a seguir:

	Get-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate

Verifique se RegistrationState é `Registered` antes de continuar.

Agora, entre em sua conta para o modelo clássico.

	Add-AzureAccount

Obtenha as assinaturas disponíveis usando o comando a seguir:

	Get-AzureSubscription | Sort SubscriptionName | Select SubscriptionName

Defina sua assinatura do Azure para a sessão atual. Substitua tudo que estiver entre aspas, incluindo os caracteres < e >, pelo nome correto.

	Select-AzureSubscription –SubscriptionName "<subscription name>"

## Etapa 4: Verificar se você tem uma quantidade suficiente de núcleos de Máquina Virtual do Azure Resource Manager na região do Azure de sua implantação atual ou VNET

Você pode usar o seguinte comando do PowerShell para verificar a quantidade atual de núcleos no Azure Resource Manager. Para saber mais sobre cotas de núcleos, veja [Limites e o Azure Resource Manager](../azure-subscription-service-limits.md#limits-and-the-azure-resource-manager).

```
Get-AzureRmVMUsage -Location "<Your VNET or Deployment's Azure region"
```

## Etapa 5: Executar comandos para migrar os recursos de IaaS

>[AZURE.NOTE] Todas as operações descritas aqui são idempotentes. Caso você tenha algum problema que não seja um recurso sem suporte ou um erro de configuração, recomendamos que repita a operação de preparação, anulação ou confirmação. Em seguida, a plataforma tentará novamente a ação.

### Migrar máquinas virtuais em um serviço de nuvem (não em uma rede virtual)

Obtenha a lista de serviços de nuvem usando o comando a seguir e escolha o serviço de nuvem que deseja migrar. Se as VMs no serviço de nuvem estiverem em uma rede virtual, ou se tiverem funções Web ou de trabalho, o comando retornará uma mensagem de erro.

	Get-AzureService | ft Servicename

Obtenha o nome da implantação do serviço de nuvem usando os comandos a seguir:

	$serviceName = "<service name>"
	$deployment = Get-AzureDeployment -ServiceName $serviceName
	$deploymentName = $deployment.DeploymentName

Prepare as máquinas virtuais no serviço de nuvem para migração. Você tem duas opções entre as quais escolher.

* **Opção 1. Migrar as VMs para uma rede virtual criada com uma plataforma**

	Primeiro, valide se você pode migrar o serviço de nuvem usando os seguintes comandos:

		$validate = Move-AzureService -Validate -ServiceName $serviceName -DeploymentName $deploymentName -CreateNewVirtualNetwork
		$validate.ValidationMessages

	O comando anterior exibe avisos e erros que bloqueiam a migração. Se a validação for bem-sucedida, você poderá prosseguir com a seguinte etapa de Preparação:

		Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName -CreateNewVirtualNetwork

* **Opção 2. Migrar para uma rede virtual existente no modelo de implantação do Gerenciador de Recursos**

		$existingVnetRGName = "<Existing VNET's Resource Group Name>"
		$vnetName = "<Virtual Network Name>"
		$subnetName = "<Subnet name>"

	Primeiro, valide se você pode migrar o serviço de nuvem usando o seguinte comando:

		$validate = Move-AzureService -Validate -ServiceName $serviceName -DeploymentName $deploymentName -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName -VirtualNetworkName $vnetName -SubnetName $subnetName
		$validate.ValidationMessages

	O comando anterior exibe avisos e erros que bloqueiam a migração. Se a validação for bem-sucedida, você poderá prosseguir com a seguinte etapa de Preparação:

		Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName -VirtualNetworkName $vnetName -SubnetName $subnetName

Após a operação de Preparação ser bem-sucedida com uma das opções anteriores, consulte o estado de migração das VMs. Certifique-se de que elas estejam no estado `Prepared`.

	$vmName = "<vm-name>"
	$vm = Get-AzureVM -ServiceName $serviceName -Name $vmName
	$migrationState = $vm.VM.MigrationState

Verifique a configuração dos recursos preparados usando o PowerShell ou o Portal do Azure. Se você não estiver pronto para a migração e desejar voltar para o estado anterior, use o comando a seguir:

	Move-AzureService -Abort -ServiceName $serviceName -DeploymentName $deploymentName

Se a configuração preparada estiver correta, será possível continuar e confirmar os recursos usando o comando a seguir:

	Move-AzureService -Commit -ServiceName $serviceName -DeploymentName $deploymentName

### Migrar máquinas virtuais em uma rede virtual

Para migrar máquinas virtuais em uma rede virtual, migre a rede. As máquinas virtuais são migradas automaticamente com a rede. Selecione a rede virtual que você deseja migrar.

	$vnetName = "<Virtual Network Name>"

>[AZURE.NOTE] Se a rede virtual contiver funções Web ou de trabalho ou VMs com configurações sem suporte, você receberá uma mensagem de erro de validação.

Primeiro, valide se você pode migrar a rede virtual usando o seguinte comando:

	Move-AzureVirtualNetwork -Validate -VirtualNetworkName $vnetName

O comando anterior exibe avisos e erros que bloqueiam a migração. Se a validação for bem-sucedida, você poderá prosseguir com a seguinte etapa de Preparação:
	
	Move-AzureVirtualNetwork -Prepare -VirtualNetworkName $vnetName

Verifique a configuração para as máquinas virtuais preparadas usando o Azure PowerShell ou o Portal do Azure. Se você não estiver pronto para a migração e desejar voltar para o estado anterior, use o comando a seguir:

	Move-AzureVirtualNetwork -Abort -VirtualNetworkName $vnetName

Se a configuração preparada estiver correta, será possível continuar e confirmar os recursos usando o comando a seguir:

	Move-AzureVirtualNetwork -Commit -VirtualNetworkName $vnetName

### Migrar uma conta de armazenamento

Depois de concluir a migração das máquinas virtuais, recomendamos a migração da conta de armazenamento.

Prepare cada conta de armazenamento para migração usando o comando a seguir:

	$storageAccountName = "<storage account name>"
	Move-AzureStorageAccount -Prepare -StorageAccountName $storageAccountName

Verifique a configuração da conta de armazenamento preparada usando o Azure PowerShell ou o Portal do Azure. Se você não estiver pronto para a migração e desejar voltar para o estado anterior, use o comando a seguir:

	Move-AzureStorageAccount -Abort -StorageAccountName $storageAccountName

Se a configuração preparada estiver correta, será possível continuar e confirmar os recursos usando o comando a seguir:

	Move-AzureStorageAccount -Commit -StorageAccountName $storageAccountName

## Próximas etapas

- Para obter informações sobre migração, confira [Migração de recursos de IaaS com suporte da plataforma do clássico ao Azure Resource Manager](virtual-machines-windows-migration-classic-resource-manager.md).
- Para migrar recursos de rede adicionais para o Resource Manager usando o Azure PowerShell, use etapas semelhantes com **Move-AzureNetworkSecurityGroup**, **Move-AzureNetworkSecurityGroup** e **Move-AzureRouteTable**.
- Para scripts de código-fonte aberto que você pode usar para migrar os recursos do Azure do clássico ao Resource Manager, confira [Ferramentas de comunidade para a migração no Azure Resource Manager](virtual-machines-windows-migration-scripts.md)

<!---HONumber=AcomDC_0921_2016-->