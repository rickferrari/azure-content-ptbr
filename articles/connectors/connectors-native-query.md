<properties
	pageTitle="Adicionar a ação de consulta nos aplicativos lógicos | Microsoft Azure"
	description="Visão geral da ação de consulta para executar ações, como filtrar matriz."
	services=""
	documentationCenter=""
	authors="jeffhollan"
	manager="erikre"
	editor=""
	tags="connectors"/>

<tags
   ms.service="logic-apps"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="07/20/2016"
   ms.author="jehollan"/>

# Introdução à ação de consulta

Usando a ação de consulta, você pode trabalhar com lotes e matrizes para executar os fluxos de trabalho para:

- Criar uma tarefa para todos os registros de alta prioridade a partir de um banco de dados.
- Salvar todos os anexos em PDF dos emails em um blob do Azure.

Para começar a usar a ação de consulta em um aplicativo lógico, consulte [Criar um aplicativo lógico](../app-service-logic/app-service-logic-create-a-logic-app.md).

## Usar a ação de consulta

Uma ação é uma operação executada pelo fluxo de trabalho definido em um aplicativo lógico. [Saiba mais sobre ações](connectors-overview.md).

Atualmente, a ação de consulta tem uma operação, chamada matriz de filtro, que é exposta no designer. Isso permite que você consulte uma matriz e retorne um conjunto de resultados filtrados.

Veja como é possível adicioná-lo em um aplicativo lógico:

1. Selecione o botão **Nova Etapa**.
2. Escolha **Adicionar uma ação**.
3. Na caixa de pesquisa da ação, digite **filtro** para listar a ação **Filtrar matriz**.

	![Selecionar a ação de consulta](./media/connectors-native-query/using-action-1.png)

4. Selecione uma matriz para filtrar. (A captura de tela a seguir mostra a matriz de resultados de uma pesquisa do Twitter.)
5. Crie uma condição para avaliar em cada item. (A captura de tela a seguir filtra os tweets dos usuários que têm mais de 100 seguidores.)

	![Concluir a ação de consulta](./media/connectors-native-query/using-action-2.png)

	A ação produzirá uma nova matriz que contém somente os resultados que atendem aos requisitos do filtro.
6. Clique no canto superior esquerdo da barra de ferramentas para salvar e seu aplicativo lógico será salvo e publicado (ativado).

## Ação de consulta

Veja os detalhes da ação com suporte deste conector. O conector tem uma ação possível.

|Ação|Descrição|
|---|---|
|Filtrar matriz|Avalia uma condição para cada item em uma matriz e retorna os resultados|

## Detalhes da ação

A ação de consulta vem com uma ação possível. As tabelas a seguir descrevem os campos de entrada obrigatórios e opcionais para a ação e os detalhes de saída correspondentes associados ao uso da ação.

### Filtrar matriz
Estes são os campos de entrada para a ação, o que cria uma solicitação HTTP de saída. Um * significa que é um campo obrigatório.

|Nome de exibição|Nome da propriedade|Descrição|
|---|---|---|
|De*|de|A matriz a ser filtrada|
|Condição*|onde|A condição a ser avaliada para cada item|
<br>

### Detalhes de saída

A seguir, os detalhes de saída para a resposta HTTP.

|Nome da propriedade|Tipo de dados|Descrição|
|---|---|---|
|Matriz filtrada|array|Uma matriz que contém um objeto para cada resultado filtrado|

## Próximas etapas

Agora, experimente a plataforma e [crie um aplicativo lógico](../app-service-logic/app-service-logic-create-a-logic-app.md). Você pode explorar os outros conectores disponíveis em aplicativos lógicos examinando nossa [lista de APIs](apis-list.md).

<!---HONumber=AcomDC_0817_2016-->