<properties
   pageTitle="Criar seu primeiro aplicativo do Service Fabric no Linux usando Java | Microsoft Azure"
   description="Criar e implantar um aplicativo do Service Fabric usando Java"
   services="service-fabric"
   documentationCenter="java"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="java"
   ms.topic="hero-article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/28/2016"
   ms.author="seanmck"/>


# Criar seu primeiro aplicativo do Azure Service Fabric

> [AZURE.SELECTOR]
- [C Sharp](service-fabric-create-your-first-application-in-visual-studio.md)
- [Java](service-fabric-create-your-first-linux-application-with-java.md)

O Service Fabric fornece SDKs para compilação de serviços no Linux em .NET Core e Java. Neste tutorial, vamos ver como criar um aplicativo para Linux e compilar um serviço usando Java.

## Pré-requisitos

Antes de começar, verifique se você [configurar o ambiente de desenvolvimento Linux](service-fabric-get-started-linux.md). Se você estiver usando Mac OS X, poderá [configurar um ambiente de uma caixa do Linux em uma máquina virtual usando Vagrant](service-fabric-get-started-mac.md).

## Criar o aplicativo

Um aplicativo do Service Fabric pode conter um ou mais serviços, cada um com uma função específica no fornecimento de funcionalidade do aplicativo. O SDK do Service Fabric para Linux inclui um gerador [Yeoman](http://yeoman.io/) que facilita a criação de seu primeiro serviço e a adição de mais serviços posteriormente. Vamos usar Yeoman para criar um novo aplicativo com um único serviço.

1. Em um terminal, digite **yo azuresfjava**.

2. Nome do seu aplicativo.

3. Escolha o tipo de seu primeiro serviço e dê um nome para ele. Para os fins deste tutorial, escolheremos o Serviço Reliable Actor.

  ![Gerador de Yeoman do Service Fabric para Java][sf-yeoman]

>[AZURE.NOTE] Para obter mais informações sobre as opções, confira [Visão geral do modelo de programação do Service Fabric](service-fabric-choose-framework.md).

## Compilar o aplicativo

Os modelos Yeoman do Service Fabric incluem um script de compilação para [Gradle](https://gradle.org/), que pode ser usada para compilar o aplicativo no terminal.

  ```bash
  gradle
  ```

## Implantar o aplicativo

Após a compilação do aplicativo, você pode implantá-lo no cluster local usando a CLI do Azure.

1. Conectar-se ao cluster local do Service Fabric.

    ```bash
    azuresfcli servicefabric cluster connect
    ```

2. Use o script de instalação fornecido no modelo para copiar o pacote de aplicativo no repositório de imagens do cluster, registrar o tipo de aplicativo e criar uma instância do aplicativo.

    ```bash
    cd myapp
    ./install.sh
    ```

3. Abra um navegador e navegue até o Service Fabric Explorer em http://localhost:19080/Explorer (substitua localhost pelo IP privado da VM se estiver usando Vagrant no Mac OS X).

4. Expanda o nó Aplicativos e observe que agora há uma entrada para o seu tipo de aplicativo e outra para a primeira instância desse tipo.

## Inicie o cliente de teste e execute um failover

Projetos de atores não fazem nada por conta própria. Eles exigem outro serviço ou cliente para enviar mensagens a eles. O modelo de ator inclui um script de teste simples que você pode usar para interagir com o serviço de ator.

1. Execute o script usando o utilitário de inspeção para ver a saída do serviço de ator.

    ```bash
    cd myactorsvcTestClient
    watch -n 1 ./testclient.sh
    ```

2. No Service Fabric Explorer, localize o nó que hospeda a réplica primária para o serviço de ator. Na captura de tela abaixo, é o nó 3.

    ![Localizar a réplica primária no Service Fabric Explorer][sfx-primary]

3. Clique no nó encontrado na etapa anterior e selecione **Desativar (Reiniciar)** no menu Ações. Isso reiniciará um dos cinco nós no cluster local e forçará um failover para uma das réplicas secundárias em execução em outro nó. Ao fazer isso, preste atenção à saída do cliente de teste e observe que o contador continua a aumentar apesar do failover.

## Compilar e implantar um aplicativo com o plug-in Eclipse Neon

Se você instalou o Plug-in de serviço para Eclipse Neon, você pode usá-lo para criar, compilar e implantar aplicativos do Service Fabric compilados com o Java.

### Criar o aplicativo

O Plug-in de serviço do Service Fabric está disponível por meio da extensibilidade do Eclipse.

1. No Eclipse, escolha **Arquivo > Outros > Service Fabric**. Você verá um conjunto de opções, incluindo Atores e Contêineres.

    ![Modelos de Service Fabric no Eclipse][sf-eclipse-templates]

2. Nesse caso, escolha Serviço Sem Estado.

3. Você deve confirmar o uso da perspectiva do Service Fabric, que otimiza o Eclipse para uso com projetos do Service Fabric. Selecione “Sim”.

### Implantar o aplicativo

Os modelos do Service Fabric incluem um conjunto de tarefas do Gradle para compilar e implantar aplicativos, que você pode disparar pelo Eclipse.

1. Escolha **Executar > Executar Configurações**.

2. Expanda **Projeto Gradle** e escolha **ServiceFabricDeployer**.

3. Clique em **Executar**.

Seu aplicativo será compilado e implantado em poucos instantes. Você pode monitorar o status no Service Fabric Explorer.

## Próximas etapas

- Saiba mais sobre as [Reliable Actors](service-fabric-reliable-actors-introduction.md)

<!-- Images -->
[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-yeoman.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-java/sfx-primary.png
[sf-eclipse-templates]: ./media/service-fabric-create-your-first-linux-application-with-java/sf-eclipse-templates.png

<!---HONumber=AcomDC_0928_2016-->