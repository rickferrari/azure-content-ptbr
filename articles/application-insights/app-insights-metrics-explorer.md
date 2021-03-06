<properties 
	pageTitle="Explorar métricas no Application Insights" 
	description="Como interpretar os gráficos no gerenciador de métricas e como personalizar as folhas do gerenciador de métricas." 
	services="application-insights" 
    documentationCenter=""
	authors="alancameronwills" 
	manager="douge"/>

<tags 
	ms.service="application-insights" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="ibiza" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/06/2016" 
	ms.author="awills"/>
 
# Explorar métricas no Application Insights

Métricas no [Application Insights][start] são contagens e valores medidos de eventos enviados em telemetria do seu aplicativo. Eles ajudam você a detectar problemas de desempenho e observar as tendências referentes a como seu aplicativo está sendo usado. Há uma grande variedade de métricas padrão, e você também pode criar suas próprias métricas e eventos personalizados.

Contagens de métricas e eventos são exibidas em gráficos de valores agregados como somas, médias ou contagens.

Aqui está um exemplo do gráfico:

![Abrir a lâmina de visão geral do seu aplicativo no portal do Azure](./media/app-insights-metrics-explorer/01-overview.png)

Alguns gráficos são segmentados: a altura total do gráfico em qualquer ponto é a soma das métricas exibidas. A legenda mostra, por padrão, as quantidades maiores.

Linhas pontilhadas mostram o valor da métrica de uma semana atrás.



## Intervalo de tempo

Você pode alterar o intervalo de tempo coberto pelos gráficos ou grades em qualquer folha.

![Abrir a lâmina de visão geral do seu aplicativo no portal do Azure](./media/app-insights-metrics-explorer/03-range.png)


Se você estiver esperando dados que não apareceram ainda, clique em Atualizar. Os gráficos são atualizados em intervalos, mas os intervalos são mais longos para intervalos de tempo maiores. No modo de liberação, pode levar algum tempo para que dados passem pelo pipeline de análise e sejam representados em um gráfico.

Para ampliar uma parte de um gráfico, arraste sobre ele e clique no símbolo de lente de aumento:

![Arraste por parte de um gráfico.](./media/app-insights-metrics-explorer/12-drag.png)



## Valores de granularidade e ponto

Passe o mouse sobre o gráfico para exibir os valores das métricas nesse determinado ponto.

![Passar o ponteiro do mouse sobre um gráfico](./media/app-insights-metrics-explorer/02-focus.png)

O valor de métrica em um ponto específico é agregado durante o intervalo de amostragem anterior.

O intervalo de amostragem ou "granularidade" é mostrado na parte superior da folha.

![O cabeçalho de uma folha.](./media/app-insights-metrics-explorer/11-grain.png)

Você pode ajustar a granularidade na folha Intervalo de tempo:

![O cabeçalho de uma folha.](./media/app-insights-metrics-explorer/grain.png)

As granularidades disponíveis dependem do intervalo de tempo selecionado. As granularidades explícitas são alternativas à granularidade "automática" para o intervalo de tempo.

## Metrics Explorer

Clique em qualquer gráfico na folha visão geral para ver um conjunto mais detalhado de grades e gráficos relacionados. Você pode editar esses gráficos e grades para focar nos detalhes em que você está interessado.

Ou então, basta clicar no botão Metrics Explorer no cabeçalho da folha de visão geral.

Por exemplo, clique no gráfico de Solicitações com falha do aplicativo Web:

![Na folha visão geral, clicar em um gráfico](./media/app-insights-metrics-explorer/14-trix.png)


## O que as figuras significam?

A legenda na lateral normalmente mostra, por padrão, o valor agregado durante o período do gráfico. Se você passar o mouse sobre o gráfico, ele mostra o valor nesse ponto.

Cada ponto de dados no gráfico é um acumulado dos valores de dados recebidos no intervalo de amostragem anterior, ou "granularidade". A granularidade é mostrada na parte superior da folha e varia de acordo com a escala de tempo total do gráfico.

Métricas podem ser agregadas de maneiras diferentes:

 * **Soma** adiciona os valores de todos os pontos de dados recebidos no intervalo de amostragem ou no período do gráfico.
 * **Média** divide a Soma pelo número de pontos de dados recebidos durante o intervalo.
 * Contagens **Únicas** são usadas para contagens de usuários e contas. Durante o intervalo de amostragem, ou durante o período do gráfico, a figura mostra a contagem de diferentes usuários vistos no momento.


Você pode alterar o método de agregação:

![Selecione o gráfico e a agregação](./media/app-insights-metrics-explorer/05-aggregation.png)

O método padrão para cada métrica é mostrado quando você cria um novo gráfico:

![Desmarque a seleção de todas as métricas para ver os padrões](./media/app-insights-metrics-explorer/06-total.png)



## Edição de gráficos e grades

Para adicionar um novo gráfico à folha:

![No Metrics Explorer, escolher Adicionar Gráfico](./media/app-insights-metrics-explorer/04-add.png)

Escolha **Editar** em um gráfico novo ou existente para editar o conteúdo exibido por ele:

![Selecionar uma ou mais métricas](./media/app-insights-metrics-explorer/08-select.png)

Você pode exibir mais de uma métrica em um gráfico, porém há restrições sobre as combinações que podem ser exibidas em conjunto. Assim que você escolher uma métrica, algumas das outras serão desabilitadas.

Se você codificou [métricas personalizadas][track] em seu aplicativo (chamadas para TrackMetric e TrackEvent), elas serão listados aqui.

## Segmentar os dados

Você pode dividir uma métrica por propriedade - por exemplo, para comparar exibições de página em clientes com sistemas operacionais diferentes.

Selecione um gráfico ou uma grade, ative o agrupamento e escolha uma propriedade pela qual agrupar:

![Selecionar Agrupamento Ativo, então selecionar uma propriedade em Agrupar Por](./media/app-insights-metrics-explorer/15-segment.png)

> [AZURE.NOTE] Quando você usa o agrupamento, os tipos de gráfico de Barras e de Área fornecem uma exibição empilhada. Isso é adequado quando o método de Agregação é Soma. Mas onde o tipo de agregação é Média, escolha os tipos de exibição de Linha ou Grade.

Se você codificou [métricas personalizadas][track] em seu aplicativo e elas incluem valores de propriedade, você poderá selecionar a propriedade na lista.

O gráfico é muito pequeno para dados segmentados? Ajuste sua altura:


![Ajustar a barra de controle deslizante](./media/app-insights-metrics-explorer/18-height.png)


## Filtrar seus dados

Para ver apenas as métricas para um conjunto selecionado de valores de propriedade:

![Clicar em Filtro, expandir uma propriedade e verificar alguns valores](./media/app-insights-metrics-explorer/19-filter.png)

Se você não selecionar nenhum valor para uma determinada propriedade, será o mesmo que selecionar todas elas: não há nenhum filtro para essa propriedade.

Observe as contagens de eventos junto a cada valor da propriedade. Quando você seleciona valores de uma propriedade, as contagens junto a outros valores de propriedade são ajustadas.

Os filtros se aplicam a todos os gráficos em uma folha. Se você quiser filtros diferentes aplicados a gráficos diferentes, crie e salve folhas de métricas diferentes. Se desejar, você pode fixar gráficos de diferentes folhas ao painel para vê-los lado a lado.


### Remover o tráfego de testes da Web e de bot

Use o filtro **Tráfego real ou sintético** e marque **Real**.

Você também pode filtrar por **Origem do tráfego sintético**.

### Para adicionar propriedades à lista de filtros

Você deseja filtrar a telemetria em uma categoria de sua escolha? Por exemplo, talvez você divida seus usuários em categorias diferentes e queira segmentar os dados segundo essas categorias.

[Crie sua própria propriedade](app-insights-api-custom-events-metrics.md#properties). Defina-a em um [Inicializador de Telemetria](app-insights-api-custom-events-metrics.md#telemetry-initializers) para que ela apareça em toda a telemetria, incluindo a telemetria padrão enviada por diferentes módulos do SDK.


## Editar o tipo de gráfico

Observe que você pode alternar entre gráficos e grades:

![Selecionar uma grade ou um gráfico e escolher um tipo de gráfico](./media/app-insights-metrics-explorer/16-chart-grid.png)

## Salve sua folha de métricas

Quando você tiver criado alguns gráficos, salve-os como favoritos. Se você utiliza uma conta organizacional, você pode escolher entre compartilhá-la ou não com outros membros da equipe.

![Escolher Favorito](./media/app-insights-metrics-explorer/21-favorite-save.png)

Para ver a folha novamente, **vá até a folha de visão geral** e abra Favoritos:

![Na folha Visão Geral, selecionar Favoritos](./media/app-insights-metrics-explorer/22-favorite-get.png)

Se você escolheu o intervalo de tempo Relativo quando salvou, a folha será atualizada com as métricas mais recentes. Se você escolheu o intervalo de tempo Absoluto, ele mostrará sempre os mesmos dados.

## Redefinir a folha

Se você editar uma folha mas em seguida decidir voltar ao conjunto original salvo, clique em Redefinir.

![Nos botões na parte superior do Metrics Explorer](./media/app-insights-metrics-explorer/17-reset.png)

<a name="live-metrics-stream"></a>
## Live Metrics Stream: métricas instantâneas para um monitoramento preciso

O Live Metrics Stream mostra as métricas do aplicativo no momento exato, com uma latência quase de tempo real de 1 segundo. Isso é muito útil quando você está lançando um novo build e deseja ter certeza de que tudo está funcionando como esperado ou investigar um incidente em tempo real.

![Na folha Visão Geral, clique em Live Stream](./media/app-insights-metrics-explorer/live-stream.png)

Ao contrário do Metrics Explorer, o Live Metrics Stream exibe um conjunto fixo de métricas. Os dados persistem somente durante o período em que estão no gráfico e, em seguida, são descartados.

O Live Metrics Stream está disponível com o SDK do Application Insights para ASP.NET, versão 2.1.0 ou posterior.

## Definir alertas

Para ser notificado por email sobre valores incomuns de qualquer métrica, adicione um alerta. Você pode escolher para enviar o email para os administradores de conta ou para endereços de email específicos.

![No Metrics Explorer, escolher Regras de Alerta, Adicionar Alerta](./media/app-insights-metrics-explorer/appinsights-413setMetricAlert.png)

[Saiba mais sobre alertas][alerts].

## Exportar para o Excel

Você pode exportar dados da métrica que são exibidos no Gerenciador de Métrica para um arquivo do Excel. Os dados exportados incluem dados de todos os gráficos e tabelas, como visto no portal.


![No Metrics Explorer, escolher Regras de Alerta, Adicionar Alerta](./media/app-insights-metrics-explorer/31-export.png)

Os dados para cada tabela ou gráfico são exportados para uma planilha separada no arquivo do Excel.

O que você vê é o que é exportado. Altere o intervalo de tempo ou os filtros se quiser alterar o intervalo de dados exportados. Para tabelas, se o comando **carregar mais** estiver visível, você poderá clicar nele antes de clicar em Exportar para exportar mais dados.

*Exportação funciona somente para Internet Explorer e Chrome no momento. Estamos trabalhando para adicionar suporte a outros navegadores.*

## Exportação Contínua

Se desejar que os dados sejam exportados de forma contínua para que você possa processá-los externamente, considere usar a [Exportação contínua](app-insights-export-telemetry.md).

### Power BI

Se desejar obter exibições ainda mais avançadas dos seus dados, você poderá [exportar para o Power BI](http://blogs.msdn.com/b/powerbi/archive/2015/11/04/explore-your-application-insights-data-with-power-bi.aspx).

## Análise

A [Análise](app-insights-analytics.md) é uma maneira mais versátil de analisar a telemetria usando uma linguagem de consulta eficiente. Use-a se quiser combinar ou calcular resultados de métricas ou executar uma exploração detalhada do desempenho recente de seu aplicativo. Por outro lado, use o Metrics Explorer se desejar a atualização automática, gráficos no painel e alertas.

## Solucionar problemas

*Não vejo dados no gráfico.*

* Os filtros se aplicam a todos os gráficos da folha. Verifique se, ao se concentrar em um gráfico, não definiu um filtro que excluía todos os dados em outro.

    Se quiser definir filtros diferentes em gráficos diferentes, crie-os em folhas diferentes e os salve como favoritos separados. Se desejar, você poderá fixá-los ao painel para que eles sejam exibidos lado a lado.

* Se você agrupar um gráfico por uma propriedade que não esteja definida na métrica, o gráfico ficará vazio. Tente limpar “agrupar por” ou escolha uma propriedade de agrupamento diferente.
* Haverá dados de desempenho (CPU, taxa de E/S, etc.) disponíveis para serviços Web Java, aplicativos da área de trabalho do Windows, [aplicativos Web e serviços do IIS se você instalar o Status Monitor](app-insights-monitor-performance-live-website-now.md) e os [Serviços de Nuvem do Azure](app-insights-azure.md). Esses dados não estão disponíveis para sites do Azure.



## Próximas etapas

* [Monitorando o uso com o Application Insights](app-insights-overview-usage.md)
* [Usando a Pesquisa de diagnóstico](app-insights-diagnostic-search.md)


<!--Link references-->

[alerts]: app-insights-alerts.md
[start]: app-insights-overview.md
[track]: app-insights-api-custom-events-metrics.md

 

<!---HONumber=AcomDC_0713_2016-->