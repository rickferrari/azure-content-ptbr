<properties
	pageTitle="Configuração avançada com o SDK do Engagement para aplicativos universais do Windows"
	description="Opções de configuração avançadas para o Azure Mobile Engagement com os aplicativos universais do Windows" 					
	services="mobile-engagement"
	documentationCenter="mobile"
	authors="piyushjo"
	manager="erikre"
	editor="" />

<tags
	ms.service="mobile-engagement"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-windows-store"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="08/12/2016"
	ms.author="piyushjo;ricksal" />

# Configuração avançada com o SDK do Engagement para aplicativos universais do Windows

> [AZURE.SELECTOR]
- [Universal do Windows](mobile-engagement-windows-store-advanced-configuration.md)
- [Windows Phone Silverlight](mobile-engagement-windows-phone-integrate-engagement.md)
- [iOS](mobile-engagement-ios-integrate-engagement.md)
- [Android](mobile-engagement-android-advan.mdced-configuration.md)

Este procedimento descreve como definir várias opções de configuração para aplicativos Azure Mobile Engagement do Android.

## Pré-requisitos

[AZURE.INCLUDE [Pré-requisitos](../../includes/mobile-engagement-windows-store-prereqs.md)]

##Configuração avançada

### Desabilitar o relatório de falhas automático

Você pode desabilitar a recurso relatório de falhas automático do Engagement. Assim, quando uma exceção sem tratamento ocorrer, o Engagement não fará nada.

> [AZURE.WARNING] Se você desabilitar esse recurso, quando uma falha sem tratamento ocorrer em seu aplicativo, o Engagement não enviará a falha **E** não fechará a sessão e os trabalhos.

Para desabilitar o relatório de falhas automático, personalize a configuração dependendo de como você o tiver declarado:

#### No arquivo `EngagementConfiguration.xml`

Defina a falha do relatório para `false` entre as marcas `<reportCrash>` e `</reportCrash>`.

#### No objeto `EngagementConfiguration` em tempo de execução

Defina a falha do relatório como false usando o objeto EngagementConfiguration.

		/* Engagement configuration. */
		EngagementConfiguration engagementConfiguration = new EngagementConfiguration();
		engagementConfiguration.Agent.ConnectionString = "Endpoint={appCollection}.{domain};AppId={appId};SdkKey={sdkKey}";

		/* Disable Engagement crash reporting. */
		engagementConfiguration.Agent.ReportCrash = false;

### Desabilitar relatórios em tempo real

Por padrão, o serviço Engagement reporta logs em tempo real. Se seu aplicativo reporta logs com muita frequência, é melhor armazenar os logs em buffer e relatá-los todos de uma vez em uma base de tempo normal. Isso é chamado de "modo de disparo contínuo".

Para fazer isso, chame o método:

		EngagementAgent.Instance.SetBurstThreshold(int everyMs);

O argumento é um valor em **milissegundos**. Sempre que você quiser reativar o registro em log em tempo real, chame o método sem nenhum parâmetro, ou com o valor 0.

O modo de disparo contínuo aumenta ligeiramente a vida útil da bateria, mas tem um impacto no Monitor do Engagement: a duração de todas as sessões e trabalhos será arredondada para o limite de intermitência (portanto, as sessões e trabalhos mais curtos do que o limite de intermitência podem não estar visíveis). Recomendamos usar um limite de disparo contínuo até 30000 (30s). Logs salvos são limitados a 300 itens. Se o envio for muito longo, você poderá perder alguns logs.

> [AZURE.WARNING] O limite de disparo contínuo não pode ser configurado com um período inferior a um segundo. Se você fizer isso, o SDK mostrará um rastreamento com o erro e será redefinido automaticamente como o valor padrão, zero segundos. Isso disparará o SDK para relatar os logs em tempo real.

[here]: http://www.nuget.org/packages/Capptain.WindowsCS
[NuGet website]: http://docs.nuget.org/docs/start-here/overview

<!---HONumber=AcomDC_0817_2016-->