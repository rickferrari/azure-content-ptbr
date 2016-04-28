<properties
	pageTitle="Detectar a face e a emoção com o Azure Media Analytics"
	description="Este tópico demonstra como detectar faces e emoções com o Azure Media Analytics."
	services="media-services"
	documentationCenter=""
	authors="juliako"
	manager="erikre"
	editor=""/>

<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="04/08/2016"   
	ms.author="milanga;juliako;"/>

#Detectar a face e a emoção com o Azure Media Analytics

##Visão geral

O MP (processador de mídia) **Azure Media Face Detector** permite que você conte e acompanhe movimentos, e até mesmo meça a participação e a reação do público por meio de expressões faciais. Este serviço contém dois recursos:

- **Detecção facial**

	A detecção facial localiza e acompanha as faces humanas em um vídeo. Várias faces podem ser detectadas e, consequentemente, acompanhadas à medida que se movimentam, com os metadados de hora e local retornados em um arquivo JSON. Durante o acompanhamento, esse recurso tentará atribuir uma ID consistente à mesma face enquanto a pessoa se movimenta pela tela, mesmo se for obstruída ou deixar brevemente o quadro.

	>[AZURE.NOTE]Esses serviços não realizam o reconhecimento facial. Uma pessoa que sair do enquadramento ou ficar obstruída por muito tempo receberá uma nova ID quando retornar.

- **Detecção de emoções**
	
	A Detecção de Emoções é um componente opcional do Face Detection Media Processor que retorna uma análise sobre vários atributos emocionais das faces detectadas, incluindo felicidade, tristeza, medo, irritação e muito mais. Esses dados podem retornar de acordo com a face ou como um valor de grupo, e podem ser agregados em uma janela e intervalo personalizáveis.

No momento, o MP do **Azure Media Face Detector** está em versão de Visualização.

Este tópico fornece detalhes sobre o **Azure Media Face Detector** e mostra como usá-lo com o SDK dos Serviços de Mídia para .NET

##Arquivos de entrada do Face Detector

Arquivos de vídeo. Atualmente, há suporte para os seguintes formatos: MP4, MOV e WMV.

##Arquivos de saída do Face Detector

A API de detecção e acompanhamento facial fornece detecção e acompanhamento de alta precisão local de até 64 faces humanas em um vídeo. Faces frontais fornecem os melhores resultados, enquanto as faces laterais e faces pequenas (menores ou iguais a 24x24 pixels) talvez não sejam tão precisas.

As faces detectadas e acompanhadas retornam com coordenadas (esquerda, superior, largura e altura) que indicam o local das faces na imagem em pixels, bem como um número de identificação da face indicando o acompanhamento dessa pessoa. Os números de identificação de face estão sujeitos à redefinição em circunstâncias nas quais a face frontal é perdida ou sobreposta no quadro, resultando na atribuição de várias IDs às mesmas pessoas.

###<a id="output_elements"></a>Elementos do arquivo JSON de saída

Para a operação de detecção e acompanhamento facial, o resultado de saída contém os metadados das faces dentro do arquivo especificado no formato JSON.

O JSON de detecção e acompanhamento facial inclui os seguintes atributos:

Elemento|Descrição
---|---
Versão|Refere-se à versão da API de Vídeo.
Escala de tempo|"Tiques" por segundo do vídeo.
Deslocamento|Essa é a diferença de tempo para carimbos de hora. Na versão 1.0 das APIs de Vídeo, sempre será 0. Em cenários futuro para os quais oferecemos suporte, esse valor poderá ser alterado.
Taxa de quadros|Quadros por segundo do vídeo.
Fragmentos|Os metadados são agrupados em segmentos diferentes, chamados fragmentos. Cada fragmento contém um início, uma duração, um número de intervalo e evento(s).
Iniciar|A hora de início do primeiro evento em “tiques”.
Duração|A duração do fragmento, em "tiques".
Intervalo|O intervalo de cada entrada de evento dentro do fragmento, em "tiques".
Eventos|Cada evento contém as faces detectadas e acompanhadas dentro desse período. É uma matriz de matriz de eventos. A matriz externa representa um intervalo de tempo. A matriz interna é composta por 0 ou mais eventos que ocorreram nesse ponto no tempo. Um colchete vazio significa que nenhuma face foi detectada.
ID| A ID da face que está sendo acompanhada. Esse número pode mudar inadvertidamente se uma face se tornar indetectável. Uma certa pessoa deve ter a mesma ID durante todo o vídeo geral, mas isso não pode ser garantido devido a limitações no algoritmo de detecção (oclusão etc.)
X, Y|As coordenadas X e Y no lado superior esquerdo da caixa delimitadora da face em uma escala normalizada de 0,0 a 1,0. <br/>As coordenadas -X e Y sempre têm relação à orientação paisagem, portanto, se você tiver um vídeo em orientação retrato (ou de cabeça para baixo, no caso do iOS), será necessário transpor as coordenadas adequadamente.
Largura, Altura|A largura e a altura da caixa delimitadora de face em uma escala normalizada do 0,0 a 1,0.
facesDetected|Isso é encontrado ao final dos resultados JSON e resume o número de faces que o algoritmo detectou durante o vídeo. Como as IDs podem ser redefinidas inadvertidamente caso uma face fique indetectável (por exemplo, se a face sair da tela, olhar para outro lado), esse número nem sempre será igual ao número verdadeiro de faces no vídeo.

O Face Detector usa técnicas de fragmentação (em que os metadados podem ser divididos em partes com base no tempo, e você pode baixar apenas o necessário) e segmentação (em que os eventos são divididos, caso eles se tornem muito grandes). Alguns cálculos simples podem ajudar você a transformar os dados. Por exemplo, se um evento tiver começado em 6300 (tiques), com uma escala de tempo de 2997 (tiques/s) e uma taxa de quadros de 29,97 (quadros/s), então:

- Início/Escala de tempo = 2,1 segundos
- Segundos x (Taxa de quadros/Escala de tempo) = 63 quadros

A seguir, um exemplo simples de como extrair o JSON em um formato por quadro para detecção e acompanhamento facial:
	
	var faceDetectionResultJsonString = operationResult.ProcessingResult;
	var faceDetecionTracking = 
	     JsonConvert.DeserializeObject<FaceDetectionResult>(faceDetectionResultJsonString, settings);


##Exemplo de entrada e saída da detecção facial

###Vídeo de entrada

[Vídeo de Entrada](http://ampdemo.azureedge.net/azuremediaplayer.html?url=https%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc8834d9f-0b49-4b38-bcaf-ece2746f1972%2FMicrosoft%20Convergence%202015%20%20Keynote%20Highlights.ism%2Fmanifest&amp;autoplay=false)

###Configuração de tarefa (predefinição)

Ao criar uma tarefa com o **Azure Media Face Detector**, é necessário especificar uma predefinição de configuração. A predefinição de configuração a seguir serve apenas para detecção de face.

	{"version":"1.0"}

###Saída em JSON

O exemplo de saída JSON a seguir foi truncado.

	{
	"version": 1,
	"timescale": 30000,
	"offset": 0,
	"framerate": 29.97,
	"width": 1280,
	"height": 720,
	"fragments": [
	    {
	    "start": 0,
	    "duration": 60060
	    },
	    {
	    "start": 60060,
	    "duration": 60060,
	    "interval": 1001,
	    "events": [
	        [
	        {
	            "id": 0,
	            "x": 0.519531,
	            "y": 0.180556,
	            "width": 0.0867188,
	            "height": 0.154167
	        }
	        ],
	        [
	        {
	            "id": 0,
	            "x": 0.517969,
	            "y": 0.181944,
	            "width": 0.0867188,
	            "height": 0.154167
	        }
	        ],
	        [
	        {
	            "id": 0,
	            "x": 0.517187,
	            "y": 0.183333,
	            "width": 0.0851562,
	            "height": 0.151389
	        }
	        ],

		. . . 

##Exemplo de entrada e saída da detecção de emoção


###Vídeo de entrada

[Vídeo de Entrada](http://ampdemo.azureedge.net/azuremediaplayer.html?url=https%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc8834d9f-0b49-4b38-bcaf-ece2746f1972%2FMicrosoft%20Convergence%202015%20%20Keynote%20Highlights.ism%2Fmanifest&amp;autoplay=false)

###Configuração de tarefa (predefinição)

Ao criar uma tarefa com o **Azure Media Face Detector**, é necessário especificar uma predefinição de configuração. A configuração de predefinição a seguir especifica a criação do JSON com base na detecção de emoção.
 	
	{
	  "version": "1.0",
	  "options": {
	    "aggregateEmotionWindowMs": "987",
	    "mode": "aggregateEmotion",
	    "aggregateEmotionIntervalMs": "342"
	  }
	}


Nome do atributo|Descrição
---|---
Modo|Faces: somente detecção facial <br/>AggregateEmotion: retorna uma média de valores de emoção para todas as faces no quadro.
AggregateEmotionWindowMs|Use se o modo AggregateEmotion for selecionado. Especifica a duração do vídeo usado para produzir cada resultado da agregação, em milissegundos.
AggregateEmotionIntervalMs|Use se o modo AggregateEmotion for selecionado. Especifica com que frequência deve-se produzir resultados agregados.

Os padrões de agregação abaixo são os valores recomendados para as configurações de janela e intervalo de agregação. A janela deve ser maior do que o Intervalo.

 |Padrões|Máx.|Mín.
---|---|---|---
Duração da Janela|2|3|1
Intervalo|0,5|1|0,25

###Saída em JSON

Saída em JSON para agregação de emoção (truncada):
 
	
	{
	 "version": 1,
	 "timescale": 30000,
	 "offset": 0,
	 "framerate": 29.97,
	 "width": 1280,
	 "height": 720,
	 "fragments": [
	   {
	     "start": 0,
	     "duration": 60060,
	     "interval": 15015,
	     "events": [
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           }
	         }
	       ],
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           }
	         }
	       ],
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           }
	         }
	       ],
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           }
	         }
	       ]
	     ]
	   },
	   {
	     "start": 60060,
	     "duration": 60060,
	     "interval": 15015,
	     "events": [
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 1,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0.688541,
	             "happiness": 0.0586323,
	             "surprise": 0.227184,
	             "sadness": 0.00945675,
	             "anger": 0.00592107,
	             "disgust": 0.00154993,
	             "fear": 0.00450447,
	             "contempt": 0.0042109
	           }
	         }
	       ],
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 1,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,


##Limitações

- Os formatos de vídeo de entrada com suporte incluem MP4, MOV e WMV.
- O intervalo de tamanho de face detectável é de 24x24 a 2048x2048 pixels. As faces fora desse intervalo não serão detectadas.
- Para cada vídeo, o número máximo de faces retornadas é de 64.
- Talvez algumas faces não sejam detectadas por questões técnicas; por exemplo, ângulos de face muito grandes (foto de rosto) e oclusão grande. As faces frontais e quase frontais têm os melhores resultados.


## Exemplo de código

O programa a seguir mostra como:

1. Criar um ativo e carregar um arquivo de mídia nesse ativo.
1. Criar um trabalho com uma tarefa de detecção de face baseada em um arquivo de configuração que contém a predefinição de JSON a seguir. 
					
		{
		    "version": "1.0"
		}

1. Baixar os arquivos JSON de saída.
		 
        using System;
		using System.Configuration;
		using System.IO;
		using System.Linq;
		using Microsoft.WindowsAzure.MediaServices.Client;
		using System.Threading;
		using System.Threading.Tasks;
		
		namespace FaceDetection
		{
		    class Program
		    {
		        // Read values from the App.config file.
		        private static readonly string _mediaServicesAccountName =
		            ConfigurationManager.AppSettings["MediaServicesAccountName"];
		        private static readonly string _mediaServicesAccountKey =
		            ConfigurationManager.AppSettings["MediaServicesAccountKey"];
		
		        // Field for service context.
		        private static CloudMediaContext _context = null;
		        private static MediaServicesCredentials _cachedCredentials = null;
		
		        static void Main(string[] args)
		        {
		
		            // Create and cache the Media Services credentials in a static class variable.
		            _cachedCredentials = new MediaServicesCredentials(
		                            _mediaServicesAccountName,
		                            _mediaServicesAccountKey);
		            // Used the cached credentials to create CloudMediaContext.
		            _context = new CloudMediaContext(_cachedCredentials);
		
		            // Run the FaceDetection job.
		            var asset = RunFaceDetectionJob(@"C:\supportFiles\FaceDetection\BigBuckBunny.mp4",
		                                        @"C:\supportFiles\FaceDetection\config.json");
		
		            // Download the job output asset.
		            DownloadAsset(asset, @"C:\supportFiles\FaceDetection\Output");
		        }
		
		        static IAsset RunFaceDetectionJob(string inputMediaFilePath, string configurationFile)
		        {
		            // Create an asset and upload the input media file to storage.
		            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
		                "My Face Detection Input Asset",
		                AssetCreationOptions.None);
		
		            // Declare a new job.
		            IJob job = _context.Jobs.Create("My Face Detection Job");
		
		            // Get a reference to Azure Media Face Detector.
		            string MediaProcessorName = "Azure Media Face Detector";
		
		            var processor = GetLatestMediaProcessorByName(MediaProcessorName);
		
		            // Read configuration from the specified file.
		            string configuration = File.ReadAllText(configurationFile);
		
		            // Create a task with the encoding details, using a string preset.
		            ITask task = job.Tasks.AddNew("My Face Detection Task",
		                processor,
		                configuration,
		                TaskOptions.None);
		
		            // Specify the input asset.
		            task.InputAssets.Add(asset);
		
		            // Add an output asset to contain the results of the job.
		            task.OutputAssets.AddNew("My Face Detectoion Output Asset", AssetCreationOptions.None);
		
		            // Use the following event handler to check job progress.  
		            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);
		
		            // Launch the job.
		            job.Submit();
		
		            // Check job execution and wait for job to finish.
		            Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
		
		            progressJobTask.Wait();
		
		            // If job state is Error, the event handling
		            // method for job progress should log errors.  Here we check
		            // for error state and exit if needed.
		            if (job.State == JobState.Error)
		            {
		                ErrorDetail error = job.Tasks.First().ErrorDetails.First();
		                Console.WriteLine(string.Format("Error: {0}. {1}",
		                                                error.Code,
		                                                error.Message));
		                return null;
		            }
		
		            return job.OutputMediaAssets[0];
		        }
		
		        static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
		        {
		            IAsset asset = _context.Assets.Create(assetName, options);
		
		            var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
		            assetFile.Upload(filePath);
		
		            return asset;
		        }
		
		        static void DownloadAsset(IAsset asset, string outputDirectory)
		        {
		            foreach (IAssetFile file in asset.AssetFiles)
		            {
		                file.Download(Path.Combine(outputDirectory, file.Name));
		            }
		        }
		
		        static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
		        {
		            var processor = _context.MediaProcessors
		                .Where(p => p.Name == mediaProcessorName)
		                .ToList()
		                .OrderBy(p => new Version(p.Version))
		                .LastOrDefault();
		
		            if (processor == null)
		                throw new ArgumentException(string.Format("Unknown media processor",
		                                                           mediaProcessorName));
		
		            return processor;
		        }
		
		        static private void StateChanged(object sender, JobStateChangedEventArgs e)
		        {
		            Console.WriteLine("Job state changed event:");
		            Console.WriteLine("  Previous state: " + e.PreviousState);
		            Console.WriteLine("  Current state: " + e.CurrentState);
		
		            switch (e.CurrentState)
		            {
		                case JobState.Finished:
		                    Console.WriteLine();
		                    Console.WriteLine("Job is finished.");
		                    Console.WriteLine();
		                    break;
		                case JobState.Canceling:
		                case JobState.Queued:
		                case JobState.Scheduled:
		                case JobState.Processing:
		                    Console.WriteLine("Please wait...\n");
		                    break;
		                case JobState.Canceled:
		                case JobState.Error:
		                    // Cast sender as a job.
		                    IJob job = (IJob)sender;
		                    // Display or log error details as needed.
		                    // LogJobStop(job.Id);
		                    break;
		                default:
		                    break;
		            }
		        }
		
		    }
        }


##Roteiros de aprendizagem dos Serviços de Mídia

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##Fornecer comentários

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

##Links relacionados

[Visão geral do Azure Media Services Analytics](media-services-analytics-overview.md)

<!---HONumber=AcomDC_0413_2016-->