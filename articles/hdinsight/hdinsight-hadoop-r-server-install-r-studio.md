<properties
	pageTitle="Instalar o RStudio com o Servidor R no HDInsight (preview) | Microsoft Azure"
	description="Como instalar o RStudio com o Servidor R no HDInsight (preview)."
	services="hdinsight"
	documentationCenter=""
	authors="jeffstokes72"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="09/16/2016"
   ms.author="jeffstok"/>


# Instalar o RStudio com o Servidor R no HDInsight (preview)

Há vários ambientes de desenvolvimento integrado (IDE) disponíveis para R hoje, incluindo o recém-anunciado [R Tools for Visual Studio](https://www.visualstudio.com/pt-BR/features/rtvs-vs.aspx) (RTVS) da Microsoft, uma família de ferramentas para desktop e servidor do [RStudio](https://www.rstudio.com/products/rstudio-server/), ou o [StatET](http://www.walware.de/goto/statet) baseado no Eclipse da Walware. Entre os mais populares em Linux está o uso do [RStudio Server](https://www.rstudio.com/products/rstudio-server/), que fornece um IDE baseado em navegador para usar em clientes remotos. Instalar o RStudio Server no nó de borda de um cluster HDInsight Premium fornece uma experiência completa de IDE para o desenvolvimento e a execução de scripts R com o R Server no cluster, além de poder ser consideravelmente mais produtivo do que o uso padrão do R Console.

Neste artigo, você aprenderá como instalar a versão de comunidade (gratuita) do RStudio Server no nó de borda de um cluster usando um script personalizado. Se preferir a versão Pro licenciada comercialmente do RStudio Server, você deverá seguir as instruções de instalação do [RStudio Server](https://www.rstudio.com/products/rstudio/download-server/).

> [AZURE.NOTE] As etapas neste documento exigem um Servidor R no cluster HDInsight e não funcionarão corretamente se você estiver usando um cluster HDInsight em que o R foi instalado usando [Instalar a Ação de Script R](hdinsight-hadoop-r-scripts-linux.md).

## Pré-requisitos

* Um cluster Azure HDInsight com R Server instalado. Para obter instruções, confira [Introdução ao Servidor R nos clusters HDInsight](hdinsight-hadoop-r-server-get-started.md).
* Um cliente SSH. Para distribuições Linux e Unix ou o Macintosh OS X, o comando `ssh` é fornecido com o sistema operacional. Para o Windows, recomendamos [Cygwin](http://www.redhat.com/services/custom/cygwin/) com a [opção OpenSSH](https://www.youtube.com/watch?v=CwYSvvGaiWU) ou [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).


## Instalar o RStudio no cluster usando um script personalizado

1. Identifique o nó de borda do cluster. Para um cluster HDInsight com R Server, esta é a convenção de nomenclatura para o nó principal e o nó de borda:

	* Nó principal - `CLUSTERNAME-ssh.azurehdinsight.net`
	* Nó de borda – `R-Server.CLUSTERNAME-ssh.azurehdinsight.net`

2. SSH no nó de borda do cluster usando o padrão de nomenclatura acima.
 
	* Se estiver se conectando por meio de um cliente Linux, veja [Conectar-se a um cluster HDInsight baseado em Linux](hdinsight-hadoop-linux-use-ssh-unix.md#connect-to-a-linux-based-hdinsight-cluster).
	* Se estiver se conectando por meio de um cliente Windows, veja [Connect to a Linux-based HDInsight cluster using PuTTY](hdinsight-hadoop-linux-use-ssh-windows.md#connect-to-a-linux-based-hdinsight-cluster) (Conectar-se a um cluster HDInsight baseado em Linux usando o PuTTY).

3. Quando você estiver conectado, torne-se um usuário raiz no cluster. Na sessão do SSH, use o comando a seguir.

		sudo su -

4. Baixe o script personalizado para instalar o RStudio. Use o seguinte comando:

		wget http://mrsactionscripts.blob.core.windows.net/rstudio-server-community-v01/InstallRStudio.sh

5. Altere as permissões no arquivo de script personalizado e execute o script. Use os seguintes comandos.

		chmod 755 InstallRStudio.sh
		./InstallRStudio.sh

6. Se você usou uma senha do SSH ao criar um cluster HDInsight com o R Server, pode ignorar esta etapa e ir para a próxima. Se, em vez disso, você usou uma chave do SSH para criar o cluster, você deve definir uma senha para seu usuário do SSH. Você precisará dessa senha ao se conectar ao RStudio. Execute os seguintes comandos: Quando a **Senha atual do Kerberos** for solicitada, basta pressionar **ENTER**. Observe que você deve substituir `USERNAME` por um usuário SSH de seu cluster HDInsight.

		passwd USERNAME
		Current Kerberos password:
		New password:
		Retype new password:
		Current Kerberos password:
		
	Se sua senha tiver sido definida com êxito, você deverá ver uma mensagem como esta.

		passwd: password updated successfully


	Saia da sessão do SSH.

7. Crie um túnel SSH para o cluster, mapeando `localhost:8787` no cluster HDInsight para o computador cliente. Você deve criar um túnel SSH antes de abrir uma nova sessão do navegador.

	* Em um cliente Linux ou Windows com [Cygwin](http://www.redhat.com/services/custom/cygwin/), abra uma sessão de terminal e use o comando a seguir.

			ssh -L localhost:8787:localhost:8787 USERNAME@R-Server.CLUSTERNAME-ssh.azurehdinsight.net
			
		Substitua **USERNAME** por um usuário SSH de seu cluster HDInsight e substitua **CLUSTERNAME** pelo nome do seu cluster HDInsight. Você também pode usar uma chave SSH em vez de uma senha adicionando `-i id_rsa_key`

	* Se estiver usar um cliente do Windows com PuTTY

		1.  Abra o PuTTY e insira as informações da sua conexão. Se você não estiver familiarizado com o PuTTY, confira [Usar SSH com Hadoop baseado em Linux no HDInsight do Windows](hdinsight-hadoop-linux-use-ssh-windows.md) para obter informações sobre como usá-lo com o HDInsight.
		2.  Na seção **Categoria** à esquerda da caixa de diálogo, expanda **Conexão**, expanda **SSH** e selecione **Túneis**.
		3.  Forneça as seguintes informações no formulário **Opções de controle do encaminhamento de porta SSH**:

			* **Porta de Origem**: a porta no cliente que você deseja encaminhar. Por exemplo, **8787**.
			* **Destino** – o destino que deve ser mapeado para o computador cliente local. Por exemplo, **localhost:8787**.

			![Criar um túnel SSH](./media/hdinsight-hadoop-r-server-install-r-studio/createsshtunnel.png "Criar um túnel SSH")

		4. Clique em **Adicionar** para adicionar as configurações e clique em **Abrir** para abrir uma conexão SSH.
		5. Quando solicitado, faça logon no servidor. Isso estabelecerá uma sessão SSH e habilitará o túnel.

8. Abra um navegador da Web e digite a seguinte URL com base na porta que você inseriu para o túnel:

		http://localhost:8787/ 

9. Você deverá inserir o nome de usuário e a senha do SSH para se conectar ao cluster. Se você usou uma chave do SSH ao criar o cluster, deverá inserir a senha criada na etapa 5 acima.

	![Conectar-se ao RStudio](./media/hdinsight-hadoop-r-server-install-r-studio/connecttostudio.png "Criar um túnel SSH")

10. Para testar se a instalação do RStudio foi bem-sucedida, você pode executar um script de teste que execute trabalhos do MapReduce e do Spark baseados em R no cluster. Volte para o console do SSH e insira os comandos a seguir para baixar o script de teste a ser executado no RStudio.

	* Se você criou um cluster Hadoop com R, use este comando:
		
			wget http://mrsactionscripts.blob.core.windows.net/rstudio-server-community-v01/testhdi.r

	* Se você criou um cluster Spark com R, use este comando:

			wget http://mrsactionscripts.blob.core.windows.net/rstudio-server-community-v01/testhdi_spark.r

11. No RStudio, você verá o script de teste que baixou. Clique duas vezes no arquivo para abri-lo, selecione o conteúdo e clique em **Executar**. Você deverá ver a saída no painel **Console**.
 
	![Testar a instalação](./media/hdinsight-hadoop-r-server-install-r-studio/test-r-script.png "Testar a instalação")

Outra opção seria digitar `source(testhdi.r)` ou `source(testhdi_spark.r)` para executar o script.

## Confira também

- [Opções de contexto de computação para o R Server em clusters HDInsight](hdinsight-hadoop-r-server-compute-contexts.md)

- [Opções de armazenamento do Azure para o R Server no HDInsight Premium](hdinsight-hadoop-r-server-storage.md)


 

<!---HONumber=AcomDC_0921_2016-->