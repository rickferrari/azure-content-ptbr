<properties
	pageTitle="Módulo PowerShell para Aprendizado de Máquina | Microsoft Azure"
	description="O módulo PowerShell para Aprendizado de Máquina do Azure está disponível no modo de visualização pública. Use o PowerShell para criar e gerenciar os espaços de trabalho, os experimentos, os serviços Web e muito mais."
	keywords="teste,regressão linear,algoritmos de aprendizado de máquina,tutorial de aprendizado de máquina,técnicas de modelos de previsão, teste de ciência de dados"
	services="machine-learning"
	documentationCenter=""
	authors="hning86"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="08/05/2016"
	ms.author="garye;haining"/>

# Módulo do PowerShell para o Aprendizado de Máquina do Microsoft Azure

O módulo PowerShell do Azure para Aprendizado de Máquina é uma ferramenta poderosa que permite a você usar o Windows PowerShell para gerenciar espaços de trabalho, experimentos, conjuntos de dados, serviços da web e muito mais.

Você pode exibir a documentação e baixar o módulo, juntamente com o código-fonte completo, de [https://aka.ms/amlps](https://aka.ms/amlps).

## O que é o módulo PowerShell do Aprendizado de Máquina?

O módulo PowerShell do Aprendizado de Máquina é um módulo DLL baseado em .NET que permite que você gerencie por completo os espaços de trabalho, os experimentos, os conjuntos de dados, os serviços Web e os pontos de extremidade de serviço Web do Aprendizado de Máquina do Azure usando o Windows PowerShell. Junto com o módulo, você pode baixar o código-fonte completo, que inclui uma [camada de API do C#](https://github.com/hning86/azuremlps/blob/master/code/AzureMLSDK.cs) separada de forma limpa. Isso significa que você pode referenciar essa DLL do seu próprio projeto .NET e gerenciar o Aprendizado de Máquina do Azure por meio do código .NET. Além disso, a DLL depende de APIs REST subjacentes que você pode usar diretamente do seu cliente favorito.

## O que posso fazer com o módulo PowerShell?

Aqui estão algumas das tarefas que podem ser executadas com este módulo PowerShell. Confira a [documentação completa](https://aka.ms/amlps) dessas e de várias outras funções.

- Provisionar um novo espaço de trabalho usando um certificado de gerenciamento ([New-AmlWorkspace](https://github.com/hning86/azuremlps#new-amlworkspace))
- Exportar e importar um arquivo JSON que representa um gráfico de experimento ([Export-AmlExperimentGraph](https://github.com/hning86/azuremlps#export-amlexperimentgraph) e [Import-AmlExperimentGraph](https://github.com/hning86/azuremlps#import-amlexperimentgraph))
- Executar um experimento ([Start-AmlExperiment](https://github.com/hning86/azuremlps#start-amlexperiment))
- Criar um serviço Web de um experimento de previsão ([New-AmlWebService](https://github.com/hning86/azuremlps#new-amlwebservice))
- Criar um novo ponto de extremidade em um serviço Web publicado ([Add-AmlWebServiceEndpoint](https://github.com/hning86/azuremlps#add-amlwebserviceendpoint))
- Invocar um ponto de extremidade de serviço Web RRS e/ou BES ([Invoke-AmlWebServiceRRSEndpoint](https://github.com/hning86/azuremlps#invoke-amlwebservicerrsendpoint) e [Invoke-AmlWebServicBESEndpoint](https://github.com/hning86/azuremlps#invoke-amlwebservicebesendpoint))

Veja um exemplo rápido de uso do PowerShell para executar um experimento existente:

		#Find the first Experiment named “xyz”
		$exp = (Get-AmlExperiment | where Description -eq ‘xyz’)[0]
		#Run the Experiment
		Start-AmlExperiment -ExperimentId $exp.ExperimentId 

Para obter um caso de uso mais detalhado, veja este artigo sobre como usar o módulo PowerShell para automatizar uma tarefa muito solicitada: [Criar muitos modelos e pontos de extremidade de serviço Web do de Aprendizado de Máquina por meio de um experimento usando o PowerShell](machine-learning-create-models-and-endpoints-with-powershell.md).

## Como começar?

Para começar com o PowerShell do Aprendizado de Máquina, baixe o [pacote de versão](https://github.com/hning86/azuremlps/releases) do GitHub e siga as [instruções de instalação](https://github.com/hning86/azuremlps/blob/master/README.md). Você precisará desbloquear a DLL baixada/descompactada e importá-la para o ambiente do PowerShell. A maioria dos cmdlets exige que você forneça a ID do espaço de trabalho, o token de autorização do espaço de trabalho e a região do Azure em que o espaço de trabalho está. A maneira mais simples de fornecê-los é por meio de um arquivo config.json padrão, abordado em detalhes nas instruções de instalação. Obviamente, também é possível clonar a árvore git e modificar/compilar o código localmente usando o Visual Studio.

## Próximas etapas

O módulo PowerShell continuará a ser aprimorado e expandido durante esse período de visualização. Fique atento ao [Blog do Cortana Intelligence e do Aprendizado de Máquina](https://blogs.technet.microsoft.com/machinelearning/) para obter mais informações e notícias.

<!---HONumber=AcomDC_0914_2016-->