<properties 
	pageTitle="Codificação avançada com o Codificador de Mídia Padrão" 
	description="Esse tópico mostra como executar a codificação avançada ao personalizar predefinições de tarefa do Codificador de Mídia Padrão. Esse tópico mostra como usar o SDK do .NET dos Serviços de Mídia para criar um trabalho e uma tarefa de codificação. Ele também mostra como fornecer predefinições personalizadas para o trabalho de codificação." 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="08/30/2016"   
	ms.author="juliako"/>


#Codificação avançada com o Codificador de Mídia Padrão

##Visão geral

Esse tópico mostra como executar tarefas de codificação avançada com o Codificador de Mídia Padrão. O tópico mostra [como usar o .NET para criar uma tarefa de codificação e um trabalho que executa essa tarefa](media-services-custom-mes-presets-with-dotnet.md#encoding_with_dotnet). Ele também mostra como fornecer predefinições personalizadas para a tarefa de codificação. [Esse documento](https://msdn.microsoft.com/library/mt269962.aspx) contém descrições dos elementos que são usados por essas predefinições.

As predefinições personalizadas que executam as seguintes tarefas de codificação são demonstradas:

- [Gerar miniaturas](media-services-custom-mes-presets-with-dotnet.md#thumbnails)
- [Cortar um vídeo (recorte)](media-services-custom-mes-presets-with-dotnet.md#trim_video)
- [Criar uma sobreposição](media-services-custom-mes-presets-with-dotnet.md#overlay)
- [Inserir uma faixa de áudio silenciosa quando a entrada não tiver áudio](media-services-custom-mes-presets-with-dotnet.md#silent_audio)
- [Desabilitar desentrelaçamento automático](media-services-custom-mes-presets-with-dotnet.md#deinterlacing)
- [Predefinições somente de áudio](media-services-custom-mes-presets-with-dotnet.md#audio_only)
- [Concatenar dois ou mais arquivos de vídeo](media-services-custom-mes-presets-with-dotnet.md#concatenate)
- [Cortar vídeos com o Codificador de Mídia Padrão](media-services-custom-mes-presets-with-dotnet.md#crop)
- [Inserir uma faixa de vídeo quando a entrada não tiver vídeo](media-services-custom-mes-presets-with-dotnet.md#no_video)

##<a id="encoding_with_dotnet"></a>Codificação com o SDK .NET dos Serviços de Mídia

O exemplo de código a seguir usa o SDK .NET dos Serviços de Mídia para executar as seguintes tarefas:

- Crie um trabalho de codificação.
- Obtenha uma referência para o Codificador de Mídia Padrão.
- Carregar a predefinição personalizada de XML ou JSON. É possível salvar esse XML ou JSON (por exemplo, [XML](media-services-custom-mes-presets-with-dotnet.md#xml) ou [JSON](media-services-custom-mes-presets-with-dotnet.md#json)) em um arquivo e usar o código a seguir para carregar o arquivo.

		// Load the XML (or JSON) from the local file.
	    string configuration = File.ReadAllText(fileName);  
- Adicione uma tarefa de codificação para o trabalho.
- Especifique o ativo de entrada a ser codificado.
- Crie um ativo de saída contendo o ativo codificado.
- Adicione um manipulador de eventos para verificar o progresso do trabalho.
- Enviar o trabalho.
	
		using System;
		using System.Collections.Generic;
		using System.Configuration;
		using System.IO;
		using System.Linq;
		using System.Net;
		using System.Security.Cryptography;
		using System.Text;
		using System.Threading.Tasks;
		using Microsoft.WindowsAzure.MediaServices.Client;
		using Newtonsoft.Json.Linq;
		using System.Threading;
		using Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization;
		using Microsoft.WindowsAzure.MediaServices.Client.DynamicEncryption;
		using System.Web;
		using System.Globalization;
		
		namespace CustomizeMESPresests
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
		
		        private static readonly string _mediaFiles =
		            Path.GetFullPath(@"../..\Media");
		
		        private static readonly string _singleMP4File =
		            Path.Combine(_mediaFiles, @"BigBuckBunny.mp4");
		
		        static void Main(string[] args)
		        {
		            // Create and cache the Media Services credentials in a static class variable.
		            _cachedCredentials = new MediaServicesCredentials(
		                            _mediaServicesAccountName,
		                            _mediaServicesAccountKey);
		            // Used the chached credentials to create CloudMediaContext.
		            _context = new CloudMediaContext(_cachedCredentials);
		
		            // Get an uploaded asset.
		            var asset = _context.Assets.FirstOrDefault();
		
		            // Encode and generate the output using custom presets.
		            EncodeToAdaptiveBitrateMP4Set(asset);
		
		            Console.ReadLine();
		        }
		
		        static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset asset)
				{
				    // Declare a new job.
				    IJob job = _context.Jobs.Create("Media Encoder Standard Job");
				    // Get a media processor reference, and pass to it the name of the 
				    // processor to use for the specific task.
				    IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");
				
		
				    // Load the XML (or JSON) from the local file.
				    string configuration = File.ReadAllText("CustomPreset_JSON.json");
				
				    // Create a task
		            ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
		                processor,
		                configuration,
		                TaskOptions.None);
				
				    // Specify the input asset to be encoded.
				    task.InputAssets.Add(asset);
				    // Add an output asset to contain the results of the job. 
				    // This output is specified as AssetCreationOptions.None, which 
				    // means the output asset is not encrypted. 
				    task.OutputAssets.AddNew("Output asset",
				        AssetCreationOptions.None);
				
				    job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
				    job.Submit();
				    job.GetExecutionProgressTask(CancellationToken.None).Wait();
				
				    return job.OutputMediaAssets[0];
				}
		
		        static public IAsset UploadMediaFilesFromFolder(string folderPath)
		        {
		            IAsset asset = _context.Assets.CreateFromFolder(folderPath, AssetCreationOptions.None);
		
		            foreach (var af in asset.AssetFiles)
		            {
		                // The following code assumes 
		                // you have an input folder with one MP4 and one overlay image file.
		                if (af.Name.Contains(".mp4"))
		                    af.IsPrimary = true;
		                else
		                    af.IsPrimary = false;
		
		                af.Update();
		            }
		
		            return asset;
		        }
		
		
		        static public IAsset EncodeWithOverlay(IAsset assetSource, string customPresetFileName)
		        {
		            // Declare a new job.
		            IJob job = _context.Jobs.Create("Media Encoder Standard Job");
		            // Get a media processor reference, and pass to it the name of the 
		            // processor to use for the specific task.
		            IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");
		
		            // Load the XML (or JSON) from the local file.
		            string configuration = File.ReadAllText(customPresetFileName);
		
		            // Create a task
		            ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
		                processor,
		                configuration,
		                TaskOptions.None);
		
		            // Specify the input assets to be encoded.
		            // This asset contains a source file and an overlay file.
		            task.InputAssets.Add(assetSource);
		
		            // Add an output asset to contain the results of the job. 
		            task.OutputAssets.AddNew("Output asset",
		                AssetCreationOptions.None);
		
		            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
		            job.Submit();
		            job.GetExecutionProgressTask(CancellationToken.None).Wait();
		
		            return job.OutputMediaAssets[0];
		        }
		

		        private static void JobStateChanged(object sender, JobStateChangedEventArgs e)
		        {
		            Console.WriteLine("Job state changed event:");
		            Console.WriteLine("  Previous state: " + e.PreviousState);
		            Console.WriteLine("  Current state: " + e.CurrentState);
		            switch (e.CurrentState)
		            {
		                case JobState.Finished:
		                    Console.WriteLine();
		                    Console.WriteLine("Job is finished. Please wait while local tasks or downloads complete...");
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
		                    break;
		                default:
		                    break;
		            }
		        }
		
		
		        private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
		        {
		            var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
		            ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();
		
		            if (processor == null)
		                throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));
		
		            return processor;
		        }
		
		    }
		}


##Suporte para tamanhos relativos

Ao gerar miniaturas, você não precisa sempre especificar a largura e a altura da saída em pixels. Você pode especificá-los em porcentagens, no intervalo de [% 1, …, 100%].

###Predefinição JSON 
	
	"Width": "100%",
	"Height": "100%"

###Predefinição XML
	
	<Width>100%</Width>
	<Height>100%</Height>
	
##<a id="thumbnails"></a>Gerar miniaturas

Essa seção mostra como personalizar uma predefinição que gera miniaturas. A predefinição definida abaixo contém informações sobre como você deseja codificar seu arquivo, bem como as informações necessárias para gerar miniaturas. Você pode usar qualquer uma das predefinições MES documentadas [aqui](https://msdn.microsoft.com/library/mt269960.aspx) e adicionar o código que gera miniaturas.

>[AZURE.NOTE]A configuração **SceneChangeDetection** na predefinição a seguir só poderá ser definida como true se você estiver codificando para um vídeo com taxa de bits única. Se você estiver codificando para um vídeo de múltiplas taxas de bits e definir **SceneChangeDetection** como true, o codificador retornará um erro.


Para obter informações sobre o esquema, consulte [este](https://msdn.microsoft.com/library/mt269962.aspx) tópico.

Certifique-se de examinar a seção [Considerações](media-services-custom-mes-presets-with-dotnet.md#considerations).

###<a id="json"></a>Predefinição JSON


	{
	  "Version": 1.0,
	  "Codecs": [
	    {
	      "KeyFrameInterval": "00:00:02",
	      "SceneChangeDetection": "true",
	      "H264Layers": [
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 4500,
	          "MaxBitrate": 4500,
	          "BufferWindow": "00:00:05",
	          "Width": 1280,
	          "Height": 720,
	          "ReferenceFrames": 3,
	          "EntropyMode": "Cabac",
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	   
	        }
	      ],
	      "Type": "H264Video"
	    },
	    {
	      "JpgLayers": [
	        {
	          "Quality": 90,
	          "Type": "JpgLayer",
	          "Width": 640,
	          "Height": 360
	        }
	      ],
	      "Start": "{Best}",
	      "Type": "JpgImage"
	    },
	    {
	      "PngLayers": [
	        {
	          "Type": "PngLayer",
	          "Width": 640,
	          "Height": 360,
	        }
	      ],
	      "Start": "00:00:01",
		  "Step": "00:00:10",
	      "Range": "00:00:58",
	      "Type": "PngImage"
	    },
	    {
	      "BmpLayers": [
	        {
	          "Type": "BmpLayer",
	          "Width": 640,
	          "Height": 360
	        }
	      ],
	      "Start": "10%",
		  "Step": "10%",
	      "Range": "90%",
	      "Type": "BmpImage"
	    },
	    {
	      "Channels": 2,
	      "SamplingRate": 48000,
	      "Bitrate": 128,
	      "Type": "AACAudio"
	    }
	  ],
	  "Outputs": [
	    {
	      "FileName": "{Basename}_{Index}{Extension}",
	      "Format": {
	        "Type": "JpgFormat"
	      }
	    },
	    {
	      "FileName": "{Basename}_{Index}{Extension}",
	      "Format": {
	        "Type": "PngFormat"
	      }
	    },
	    {
	      "FileName": "{Basename}_{Index}{Extension}",
	      "Format": {
	        "Type": "BmpFormat"
	      }
	    },
	    {
	      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
	      "Format": {
	        "Type": "MP4Format"
	      }
	    }
	  ]
	}


###<a id="xml"></a>Predefinição XML


	<?xml version="1.0" encoding="utf-16"?>
	<Preset xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="http://www.windowsazure.com/media/encoding/Preset/2014/03">
	  <Encoding>
	    <H264Video>
	      <KeyFrameInterval>00:00:02</KeyFrameInterval>
	      <SceneChangeDetection>true</SceneChangeDetection>
	      <H264Layers>
	        <H264Layer>
	          <Bitrate>4500</Bitrate>
	          <Width>1280</Width>
	          <Height>720</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>4500</MaxBitrate>
	        </H264Layer>
	      </H264Layers>
	    </H264Video>
	    <AACAudio>
	      <Profile>AACLC</Profile>
	      <Channels>2</Channels>
	      <SamplingRate>48000</SamplingRate>
	      <Bitrate>128</Bitrate>
	    </AACAudio>
	    <JpgImage Start="{Best}">
	      <JpgLayers>
	        <JpgLayer>
	          <Width>640</Width>
	          <Height>360</Height>
	          <Quality>90</Quality>
	        </JpgLayer>
	      </JpgLayers>
	    </JpgImage>
	    <BmpImage Start="10%" Step="10%" Range="90%">
	      <BmpLayers>
	        <BmpLayer>
	          <Width>640</Width>
	          <Height>360</Height>
	        </BmpLayer>
	      </BmpLayers>
	    </BmpImage>
	    <PngImage Start="00:00:01" Step="00:00:10" Range="00:00:58">
	      <PngLayers>
	        <PngLayer>
	          <Width>640</Width>
	          <Height>360</Height>
	        </PngLayer>
	      </PngLayers>
	    </PngImage>
	  </Encoding>
	  <Outputs>
	    <Output FileName="{Basename}_{Width}x{Height}_{VideoBitrate}.mp4">
	      <MP4Format />
	    </Output>
	    <Output FileName="{Basename}_{Index}{Extension}">
	      <JpgFormat />
	    </Output>
	    <Output FileName="{Basename}_{Index}{Extension}">
	      <BmpFormat />
	    </Output>
	    <Output FileName="{Basename}_{Index}{Extension}">
	      <PngFormat />
	    </Output>
	  </Outputs>
	</Preset>

###Considerações

As seguintes considerações se aplicam:

- O uso de carimbos explícitos para Início/Etapa/Intervalo pressupõe que a fonte de entrada tem duração de pelo menos 1 minuto.
- Elementos Jpg/Png/BmpImage têm atributos de cadeia de caracteres de Início, Etapa e Intervalo que podem ser interpretados como:

	- Número de quadro se eles forem números inteiros não negativos, por exemplo, "Início": "120",
	- Relativos à duração da origem se expressos com sufixo %, por exemplo, "Início": "15%", OU
	- Carimbo de data/hora se expresso no formato HH:MM:SS, por exemplo, "Início": "00:01:00"

	Você pode combinar as notações como desejar.
	
	Além disso, o Início também dá suporte a uma Macro especial: {Best}, que tenta determinar o primeiro quadro "interessante" da NOTA de conteúdo: (Etapa e Intervalo são ignorados quando Início é definido como {Best})
	
	- Padrões: Start:{Best}
- O formato de saída precisa ser fornecido explicitamente para cada formato de Imagem: Jpg/Png/BmpFormat. Quando presente, o MES corresponde JpgVideo a JpgFormat e assim por diante. OutputFormat introduz uma nova Macro específica do codec de imagem: {Index}, que precisa estar presente (apenas uma vez) para formatos de saída de imagem.

##<a id="trim_video"></a>Cortar um vídeo (recorte)

Essa seção fala sobre como modificar as predefinições do codificador para recortar ou cortar o vídeo de entrada no qual a entrada é um arquivo de mezanino ou arquivo sob demanda. O codificador também pode ser usado para recortar ou cortar um ativo capturado ou arquivado em uma transmissão ao vivo. Os detalhes desse processo estão disponíveis [neste blog](https://azure.microsoft.com/blog/sub-clipping-and-live-archive-extraction-with-media-encoder-standard/).

Para cortar seus vídeos, use qualquer uma das predefinições MES documentadas [aqui](https://msdn.microsoft.com/library/mt269960.aspx) e modifique o elemento **Fontes** (conforme mostrado abaixo). O valor da StartTime precisa corresponder aos carimbos de hora absolutos do vídeo de entrada. Por exemplo, se o primeiro quadro do vídeo de entrada tem um carimbo de data/hora de 12:00:10.000, então, a StartTime deve ser pelo menos 12:00:10.000 e mais. No exemplo abaixo, estamos supondo que o vídeo de entrada tenha um carimbo de data/hora inicial igual a zero. O elemento **Fontes** deve ser colocado no início da predefinição.
 
###<a id="json"></a>Predefinição JSON
	
	{
	  "Version": 1.0,
	  "Sources": [
	    {
	      "StartTime": "00:00:04",
	      "Duration": "00:00:16"
	    }
	  ],
	  "Codecs": [
	    {
	      "KeyFrameInterval": "00:00:02",
	      "StretchMode": "AutoSize",
	      "H264Layers": [
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 3400,
	          "MaxBitrate": 3400,
	          "BufferWindow": "00:00:05",
	          "Width": 1280,
	          "Height": 720,
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	        },
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 2250,
	          "MaxBitrate": 2250,
	          "BufferWindow": "00:00:05",
	          "Width": 960,
	          "Height": 540,
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	        },
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 1500,
	          "MaxBitrate": 1500,
	          "BufferWindow": "00:00:05",
	          "Width": 960,
	          "Height": 540,
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	        },
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 1000,
	          "MaxBitrate": 1000,
	          "BufferWindow": "00:00:05",
	          "Width": 640,
	          "Height": 360,
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	        },
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 650,
	          "MaxBitrate": 650,
	          "BufferWindow": "00:00:05",
	          "Width": 640,
	          "Height": 360,
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	        },
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 400,
	          "MaxBitrate": 400,
	          "BufferWindow": "00:00:05",
	          "Width": 320,
	          "Height": 180,
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "FrameRate": "0/1"
	        }
	      ],
	      "Type": "H264Video"
	    },
	    {
	      "Profile": "AACLC",
	      "Channels": 2,
	      "SamplingRate": 48000,
	      "Bitrate": 128,
	      "Type": "AACAudio"
	    }
	  ],
	  "Outputs": [
	    {
	      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
	      "Format": {
	        "Type": "MP4Format"
	      }
	    }
	  ]
	} 

###Predefinição XML
	
Para cortar seus vídeos, use qualquer uma das predefinições MES documentadas [aqui](https://msdn.microsoft.com/library/mt269960.aspx) e modifique o elemento **Fontes** (conforme mostrado abaixo).

	<?xml version="1.0" encoding="utf-16"?>
	<Preset xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="http://www.windowsazure.com/media/encoding/Preset/2014/03">
	  <Sources>
	    <Source StartTime="PT4S" Duration="PT14S"/>
	  </Sources>
	  <Encoding>
	    <H264Video>
	      <KeyFrameInterval>00:00:02</KeyFrameInterval>
	      <H264Layers>
	        <H264Layer>
	          <Bitrate>3400</Bitrate>
	          <Width>1280</Width>
	          <Height>720</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>3400</MaxBitrate>
	        </H264Layer>
	        <H264Layer>
	          <Bitrate>2250</Bitrate>
	          <Width>960</Width>
	          <Height>540</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>2250</MaxBitrate>
	        </H264Layer>
	        <H264Layer>
	          <Bitrate>1500</Bitrate>
	          <Width>960</Width>
	          <Height>540</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>1500</MaxBitrate>
	        </H264Layer>
	        <H264Layer>
	          <Bitrate>1000</Bitrate>
	          <Width>640</Width>
	          <Height>360</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>1000</MaxBitrate>
	        </H264Layer>
	        <H264Layer>
	          <Bitrate>650</Bitrate>
	          <Width>640</Width>
	          <Height>360</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>650</MaxBitrate>
	        </H264Layer>
	        <H264Layer>
	          <Bitrate>400</Bitrate>
	          <Width>320</Width>
	          <Height>180</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>3</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cabac</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>400</MaxBitrate>
	        </H264Layer>
	      </H264Layers>
	    </H264Video>
	    <AACAudio>
	      <Profile>AACLC</Profile>
	      <Channels>2</Channels>
	      <SamplingRate>48000</SamplingRate>
	      <Bitrate>128</Bitrate>
	    </AACAudio>
	  </Encoding>
	  <Outputs>
	    <Output FileName="{Basename}_{Width}x{Height}_{VideoBitrate}.mp4">
	      <MP4Format />
	    </Output>
	  </Outputs>
	</Preset>

##<a id="overlay"></a>Criar uma sobreposição

O Codificador de Mídia Padrão permite sobrepor uma imagem em um vídeo existente. Atualmente, há suporte para os seguintes formatos: png, jpg, gif e bmp. A predefinição definida abaixo é um exemplo básico de uma sobreposição de vídeo.

Além de definir um arquivo de predefinição, você também precisa permitir que os Serviços de Mídia saibam qual arquivo no ativo é uma imagem de sobreposição e qual arquivo contém o vídeo de origem no qual você deseja fazer a sobreposição de imagem. O arquivo de vídeo precisa ser o arquivo **primário**.

O exemplo do .NET acima define duas funções: **UploadMediaFilesFromFolder** e **EncodeWithOverlay**. A função UploadMediaFilesFromFolder carrega arquivos de uma pasta (por exemplo, BigBuckBunny.mp4 e Image001.png) e define o arquivo mp4 a ser o arquivo primário no ativo. A função **EncodeWithOverlay** usa o arquivo de predefinição personalizado que foi transmitido para ele (por exemplo, a predefinição a seguir) para criar a tarefa de codificação.

>[AZURE.NOTE]Limitações atuais:
>
>Não há suporte para a configuração de opacidade de sobreposição.
>
>O arquivo de vídeo de origem e o arquivo de imagem de sobreposição precisam estar no mesmo ativo e o arquivo de vídeo deve ser definido como o arquivo principal no ativo.

###Predefinição JSON
	
	{
	  "Version": 1.0,
	  "Sources": [
	    {
	      "Streams": [],
	      "Filters": {
	        "VideoOverlay": {
	          "Position": {
	            "X": 100,
	            "Y": 100,
	            "Width": 100,
	            "Height": 50
	          },
	          "AudioGainLevel": 0.0,
	          "MediaParams": [
	            {
	              "OverlayLoopCount": 1
	            },
	            {
	              "IsOverlay": true,
	              "OverlayLoopCount": 1,
	              "InputLoop": true
	            }
	          ],
	          "Source": "Image001.png",
	          "Clip": {
	            "Duration": "00:00:05"
	          },
	          "FadeInDuration": {
	            "Duration": "00:00:01"
	          },
	          "FadeOutDuration": {
	            "StartTime": "00:00:03",
	            "Duration": "00:00:04"
	          }
	        }
	      },
	      "Pad": true
	    }
	  ],
	  "Codecs": [
	    {
	      "KeyFrameInterval": "00:00:02",
	      "H264Layers": [
	        {
	          "Profile": "Auto",
	          "Level": "auto",
	          "Bitrate": 1045,
	          "MaxBitrate": 1045,
	          "BufferWindow": "00:00:05",
	          "ReferenceFrames": 3,
	          "EntropyMode": "Cavlc",
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "Width": "640",
	          "Height": "360",
	          "FrameRate": "0/1"
	        }
	      ],
	      "Type": "H264Video"
	    },
	    {
	      "Type": "CopyAudio"
	    }
	  ],
	  "Outputs": [
	    {
	      "FileName": "{Basename}{Extension}",
	      "Format": {
	        "Type": "MP4Format"
	      }
	    }
	  ]
	}


###Predefinição XML
	
	<?xml version="1.0" encoding="utf-16"?>
	<Preset xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="http://www.windowsazure.com/media/encoding/Preset/2014/03">
	  <Sources>
	    <Source>
	      <Streams />
	      <Filters>
	        <VideoOverlay>
	          <Source>Image001.png</Source>
	          <Clip Duration="PT5S" />
	          <FadeInDuration Duration="PT1S" />
	          <FadeOutDuration StartTime="PT3S" Duration="PT4S" />
	          <Position X="100" Y="100" Width="100" Height="50" />
	          <Opacity>0</Opacity>
	          <AudioGainLevel>0</AudioGainLevel>
	          <MediaParams>
	            <MediaParam>
	              <IsOverlay>false</IsOverlay>
	              <OverlayLoopCount>1</OverlayLoopCount>
	              <InputLoop>false</InputLoop>
	            </MediaParam>
	            <MediaParam>
	              <IsOverlay>true</IsOverlay>
	              <OverlayLoopCount>1</OverlayLoopCount>
	              <InputLoop>true</InputLoop>
	            </MediaParam>
	          </MediaParams>
	        </VideoOverlay>
	      </Filters>
	      <Pad>true</Pad>
	    </Source>
	  </Sources>
	  <Encoding>
	    <H264Video>
	      <KeyFrameInterval>00:00:02</KeyFrameInterval>
	      <H264Layers>
	        <H264Layer>
	          <Bitrate>1045</Bitrate>
	          <Width>640</Width>
	          <Height>360</Height>
	          <FrameRate>0/1</FrameRate>
	          <Profile>Auto</Profile>
	          <Level>auto</Level>
	          <BFrames>0</BFrames>
	          <ReferenceFrames>3</ReferenceFrames>
	          <Slices>0</Slices>
	          <AdaptiveBFrame>true</AdaptiveBFrame>
	          <EntropyMode>Cavlc</EntropyMode>
	          <BufferWindow>00:00:05</BufferWindow>
	          <MaxBitrate>1045</MaxBitrate>
	        </H264Layer>
	      </H264Layers>
	    </H264Video>
	    <CopyAudio />
	  </Encoding>
	  <Outputs>
	    <Output FileName="{Basename}{Extension}">
	      <MP4Format />
	    </Output>
	  </Outputs>
	</Preset>


##<a id="silent_audio"></a>Inserir uma faixa de áudio silenciosa quando a entrada não tiver áudio

Por padrão, se você enviar uma entrada para o codificador que contenha apenas vídeo e sem áudio, o ativo de saída conterá os arquivos que contêm apenas dados de vídeo. Alguns reprodutores podem não ser capazes de lidar com tais fluxos de saída. Você pode usar essa configuração para forçar o codificador a adicionar uma faixa de áudio silenciosa à saída nesse cenário.

Para forçar o codificador a produzir um ativo que contenha uma faixa de áudio silenciosa quando a entrada não tiver áudio, especifique o valor de "InsertSilenceIfNoAudio".

Você pode usar qualquer uma das predefinições MES documentadas [aqui](https://msdn.microsoft.com/library/mt269960.aspx) e fazer a seguinte modificação:

###Predefinição JSON

    {
      "Channels": 2,
      "SamplingRate": 44100,
      "Bitrate": 96,
      "Type": "AACAudio",
      "Condition": "InsertSilenceIfNoAudio"
    }

###Predefinição XML

    <AACAudio Condition="InsertSilenceIfNoAudio">
      <Channels>2</Channels>
      <SamplingRate>44100</SamplingRate>
      <Bitrate>96</Bitrate>
    </AACAudio>

##<a id="deinterlacing"></a>Desabilitar desentrelaçamento automático

Os clientes não precisam fazer nada se desejarem que o conteúdo de entrelaçamento seja automaticamente desentrelaçado. Quando o desentrelaçamento automático está ativado (padrão), o MES faz a detecção automática de quadros entrelaçados e apenas desentrelaça quadros marcados como entrelaçados.

Você pode desativar o desentrelaçamento automático. No entanto, isso não é recomendado.

###Predefinição JSON
	
	"Sources": [
	{
	 "Filters": {
	    "Deinterlace": {
	      "Mode": "Off"
	    }
	  },
	}
	]

###Predefinição XML
	
	<Sources>
	<Source>
	  <Filters>
	    <Deinterlace>
	      <Mode>Off</Mode>
	    </Deinterlace>
	  </Filters>
	</Source>
	</Sources>


##<a id="audio_only"></a>Predefinições somente de áudio

Esta seção demonstra duas predefinições MES somente de áudio: áudio AAC e Áudio de Boa Qualidade AAC.

###Áudio AAC 

	{
	  "Version": 1.0,
	  "Codecs": [
	    {
	      "Profile": "AACLC",
	      "Channels": 2,
	      "SamplingRate": 48000,
	      "Bitrate": 128,
	      "Type": "AACAudio"
	    }
	  ],
	  "Outputs": [
	    {
	      "FileName": "{Basename}_AAC_{AudioBitrate}.mp4",
	      "Format": {
	        "Type": "MP4Format"
	      }
	    }
	  ]
	}

###Áudio de Boa Qualidade AAC

	{
	  "Version": 1.0,
	  "Codecs": [
	    {
	      "Profile": "AACLC",
	      "Channels": 2,
	      "SamplingRate": 48000,
	      "Bitrate": 192,
	      "Type": "AACAudio"
	    }
	  ],
	  "Outputs": [
	    {
	      "FileName": "{Basename}_AAC_{AudioBitrate}.mp4",
	      "Format": {
	        "Type": "MP4Format"
	      }
	    }
	  ]
	}

##<a id="concatenate"></a>Concatenar dois ou mais arquivos de vídeo

O exemplo a seguir ilustra como você pode gerar uma predefinição para concatenar dois ou mais arquivos de vídeo. O cenário mais comum é quando você deseja adicionar um cabeçalho ou um rodapé ao vídeo principal. O uso pretendido é quando os arquivos de vídeo que estão sendo editados juntos compartilham propriedades (resolução de vídeo, taxa de quadros, contagem de faixa de áudio, etc.). Você deve ter cuidado para não misturar vídeos de taxas de quadros diferentes, ou com um número diferente de faixas de áudio.

###Requisitos e considerações

- Vídeos de entrada devem ter apenas uma faixa de áudio.
- Todos os vídeos de entrada devem ter a mesma taxa de quadros.
- Você deve carregar seus vídeos em ativos separados e definir os vídeos como o arquivo primário de cada ativo.
- É necessário saber a duração de seus vídeos.
- Os exemplos de predefinição abaixo pressupõem que todos os vídeos de entrada começam com um carimbo de data/hora de zero. Você precisará modificar os valores de StartTime se os vídeos tiverem o carimbo de data/hora de partida diferente, como normalmente é o caso com arquivos em tempo real.
- A predefinição JSON faz referências explícitas aos valores AssetID dos ativos de entrada.
- O exemplo de código pressupõe que a predefinição JSON foi salva em um arquivo local, como "C:\\supportFiles\\preset.json". Ele também pressupõe que foram criados dois ativos carregando dois arquivos de vídeo e que você conhece os valores de AssetID resultantes.
- O trecho de código e predefinição JSON mostram um exemplo da concatenação de dois arquivos de vídeo. Você pode estendê-lo para mais de dois vídeos da seguinte forma:

	1. Chamando task.InputAssets.Add() repetidamente para adicionar mais vídeos em ordem.
	2. Fazendo edições correspondentes ao elemento “Fontes” no JSON, adicionando mais entradas na mesma ordem.


###Código do .NET

	
	IAsset asset1 = _context.Assets.Where(asset => asset.Id == "nb:cid:UUID:606db602-efd7-4436-97b4-c0b867ba195b").FirstOrDefault();
	IAsset asset2 = _context.Assets.Where(asset => asset.Id == "nb:cid:UUID:a7e2b90f-0565-4a94-87fe-0a9fa07b9c7e").FirstOrDefault();
	
	// Declare a new job.
	IJob job = _context.Jobs.Create("Media Encoder Standard Job for Concatenating Videos");
	// Get a media processor reference, and pass to it the name of the 
	// processor to use for the specific task.
	IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");
	
	// Load the XML (or JSON) from the local file.
	string configuration = File.ReadAllText(@"c:\supportFiles\preset.json");
	
	// Create a task
	ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
	    processor,
	    configuration,
	    TaskOptions.None);
	
	// Specify the input videos to be concatenated (in order).
	task.InputAssets.Add(asset1);
	task.InputAssets.Add(asset2);
	// Add an output asset to contain the results of the job. 
	// This output is specified as AssetCreationOptions.None, which 
	// means the output asset is not encrypted. 
	task.OutputAssets.AddNew("Output asset",
	    AssetCreationOptions.None);
	
	job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
	job.Submit();
	job.GetExecutionProgressTask(CancellationToken.None).Wait();

###Predefinição JSON

Atualize sua predefinição personalizada com IDs que você deseja concatenar e com o segmento de tempo apropriado para cada vídeo.

	{
	  "Version": 1.0,
	  "Sources": [
	    {
	      "AssetID": "606db602-efd7-4436-97b4-c0b867ba195b",
	      "StartTime": "00:00:01",
	      "Duration": "00:00:15"
	    },
	    {
	      "AssetID": "a7e2b90f-0565-4a94-87fe-0a9fa07b9c7e",
	      "StartTime": "00:00:02",
	      "Duration": "00:00:05"
	    }
	  ],
	  "Codecs": [
	    {
	      "KeyFrameInterval": "00:00:02",
	      "SceneChangeDetection": true,
	      "H264Layers": [
	        {
	          "Level": "auto",
	          "Bitrate": 1800,
	          "MaxBitrate": 1800,
	          "BufferWindow": "00:00:05",
	          "BFrames": 3,
	          "ReferenceFrames": 3,
	          "AdaptiveBFrame": true,
	          "Type": "H264Layer",
	          "Width": "640",
	          "Height": "360",
	          "FrameRate": "0/1"
	        }
	      ],
	      "Type": "H264Video"
	    },
	    {
	      "Channels": 2,
	      "SamplingRate": 48000,
	      "Bitrate": 128,
	      "Type": "AACAudio"
	    }
	  ],
	  "Outputs": [
	    {
	      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
	      "Format": {
	        "Type": "MP4Format"
	      }
	    }
	  ]
	}

##<a id="crop"></a>Cortar vídeos com o Codificador de Mídia Padrão

Veja o tópico [Cortar vídeos com o Codificador de Mídia Padrão](media-services-crop-video.md).

##<a id="no_video"></a>Inserir uma faixa de vídeo quando a entrada não tiver vídeo

Por padrão, se você enviar uma entrada para o codificador que contenha apenas áudio e sem vídeo, o ativo de saída conterá os arquivos contendo apenas dados de vídeo. Alguns reprodutores, incluindo o Player de Mídia do Azure (consulte [aqui](https://feedback.azure.com/forums/169396-azure-media-services/suggestions/8082468-audio-only-scenarios)) talvez não sejam capazes de lidar com esses fluxos. Você pode usar essa configuração para forçar o codificador a adicionar uma faixa de vídeo monocromático à saída nesse cenário.

>[AZURE.NOTE]Forçar o codificador para inserir uma faixa de vídeo de saída aumenta o tamanho do ativo de saída e, consequentemente, o custo incorrido para a tarefa de codificação. Você deve executar testes para certificar-se de que esse aumento resultante tem apenas um impacto pequeno sobre os encargos mensais.

### Inserindo vídeo somente com a taxa de bits mais baixa

Suponha que você está usando uma predefinição de codificação de taxa de bits múltipla como ["H264 com taxa de bits múltipla de 720p"](https://msdn.microsoft.com/library/mt269960.aspx) para codificar todo o seu catálogo de entrada para streaming, que contém uma mistura de arquivos de vídeo e arquivos de áudio. Nesse cenário, quando a entrada não tiver vídeo, é recomendável que você force o codificador a inserir uma faixa de vídeo monocromático somente na menor taxa de bits, em vez de inserir vídeo em cada taxa de bits de saída. Para fazer isso, você precisa especificar o sinalizador "InsertBlackIfNoVideoBottomLayerOnly".

Você pode usar qualquer uma das predefinições MES documentadas [aqui](https://msdn.microsoft.com/library/mt269960.aspx) e fazer a seguinte modificação:

#### Predefinição JSON

	{
	      "KeyFrameInterval": "00:00:02",
	      "StretchMode": "AutoSize",
	      "Condition": "InsertBlackIfNoVideoBottomLayerOnly",
	      "H264Layers": [
	      …
	      ]
	}

#### Predefinição XML

	<KeyFrameInterval>00:00:02</KeyFrameInterval>
	<StretchMode>AutoSize</StretchMode>
	<Condition>InsertBlackIfNoVideoBottomLayerOnly</Condition>

### Inserindo vídeo em todas as taxas de bits de saída

Suponha que você está usando uma predefinição de codificação de taxa de bits múltipla como ["H264 com taxa de bits múltipla de 720p"](https://msdn.microsoft.com/library/mt269960.aspx) para codificar todo o seu catálogo de entrada para streaming, que contém uma mistura de arquivos de vídeo e arquivos de áudio. Nesse cenário, quando a entrada não tiver vídeo, é recomendável que você force o codificador a inserir uma faixa de vídeo monocromático em todas as taxas de bits de saída. Isso garante que seus ativos de saída sejam todos homogêneos quanto ao número de faixas de vídeo e de faixas de áudio. Para fazer isso, você precisa especificar o sinalizador "InsertBlackIfNoVideo".

Você pode usar qualquer uma das predefinições MES documentadas [aqui](https://msdn.microsoft.com/library/mt269960.aspx) e fazer a seguinte modificação:

#### Predefinição JSON

	{
	      "KeyFrameInterval": "00:00:02",
	      "StretchMode": "AutoSize",
	      "Condition": "InsertBlackIfNoVideo",
	      "H264Layers": [
	      …
	      ]
	}

#### Predefinição XML
	
	<KeyFrameInterval>00:00:02</KeyFrameInterval>
	<StretchMode>AutoSize</StretchMode>
	<Condition>InsertBlackIfNoVideo</Condition>

##Roteiros de aprendizagem dos Serviços de Mídia

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##Fornecer comentários

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

##Consulte também 

[Visão geral da codificação de serviços de mídia](media-services-encode-asset.md)

<!---HONumber=AcomDC_0831_2016-->