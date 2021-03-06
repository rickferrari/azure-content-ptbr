<properties 
    pageTitle="Tutorial: Integração do Active Directory do Azure ao ArcGIS | Microsoft Azure" 
    description="Saiba como usar o ArcGIS com o Active Directory do Azure para habilitar o logon único, provisionamento automatizado e muito mais!" 
    services="active-directory" 
    authors="jeevansd"  
    documentationCenter="na" 
    manager="femila"/>
<tags 
    ms.service="active-directory" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.tgt_pltfrm="na" 
    ms.workload="identity" 
    ms.date="07/11/2016" 
    ms.author="jeedes" />

#Tutorial: Integração do Active Directory do Azure ao ArcGIS

O objetivo deste tutorial é mostrar a integração do Azure ao ArcGIS. O cenário descrito neste tutorial pressupõe que você já tem os seguintes itens:

-   Uma assinatura válida do Azure
-   Uma assinatura habilitada para logon único do ArcGIS

Depois de concluir este tutorial, os usuários do AD do Azure atribuídos ao ArcGIS poderão fazer logon único no aplicativo em seu site de empresa do ArcGIS (logon iniciado pelo provedor de serviços) ou usando a [Introdução ao Painel de Acesso](active-directory-saas-access-panel-introduction.md).

O cenário descrito neste tutorial consiste nos seguintes blocos de construção:

1.  Habilitando a integração de aplicativos para o ArcGIS
2.  Configurando o logon único
3.  Configurando o provisionamento de usuários
4.  Atribuindo usuários

![Cenário](./media/active-directory-saas-arcgis-tutorial/IC784735.png "Cenário")
##Habilitando a integração de aplicativos para o ArcGIS

O objetivo desta seção é descrever como habilitar a integração de aplicativos para o ArcGIS.

###Para habilitar a integração de aplicativos para o ArcGIS, execute as seguintes etapas:

1.  No Portal clássico do Azure, no painel de navegação à esquerda, clique em **Active Directory**.

    ![Active Directory](./media/active-directory-saas-arcgis-tutorial/IC700993.png "Active Directory")

2.  Na lista **Diretório**, selecione o diretório para o qual você deseja habilitar a integração de diretórios.

3.  Para abrir a visualização dos aplicativos, na exibição do diretório, clique em **Aplicativos** no menu principal.

    ![Aplicativos](./media/active-directory-saas-arcgis-tutorial/IC700994.png "Aplicativos")

4.  Clique em **Adicionar** na parte inferior da página.

    ![Adicionar aplicativo](./media/active-directory-saas-arcgis-tutorial/IC749321.png "Adicionar aplicativo")

5.  Na caixa de diálogo **O que você deseja fazer**, clique em **Adicionar um aplicativo da galeria**.

    ![Adicionar um aplicativo da galeria](./media/active-directory-saas-arcgis-tutorial/IC749322.png "Adicionar um aplicativo da galeria")

6.  Na **caixa de pesquisa**, digite **ArcGIS**.

    ![Galeria de Aplicativos](./media/active-directory-saas-arcgis-tutorial/IC784736.png "Galeria de Aplicativos")

7.  No painel de resultados, selecione **ArcGIS** e clique em **Concluir** para adicionar o aplicativo.

    ![ArcGIS](./media/active-directory-saas-arcgis-tutorial/IC784737.png "ArcGIS")
##Configurando o logon único

O objetivo desta seção é descrever como permitir que os usuários se autentiquem no ArcGIS com a respectiva conta do AD do Azure usando federação baseada no protocolo SAML.

###Para configurar o logon único, execute as seguintes etapas:

1.  No portal clássico do Azure, na página de integração do aplicativo **ArcGIS**, clique em **Configurar logon único** para abrir o diálogo **Configurar Logon Único**.

    ![Configurar o logon único](./media/active-directory-saas-arcgis-tutorial/IC784738.png "Configurar o logon único")

2.  Na página **Como você deseja que os usuários façam logon no ArcGIS**, selecione **Logon Único do AD do Microsoft Azure** e clique em **Avançar**.

    ![Configurar o logon único](./media/active-directory-saas-arcgis-tutorial/IC784739.png "Configurar o logon único")

3.  Na página **Configurar URL do Aplicativo**, na caixa de texto **URL de Entrada do ArcGIS**, digite a URL usada pelos usuários para entrar usando o padrão "*https://company.maps.arcgis.com*" e clique em **Avançar**.

    ![Configurar a URL do Aplicativo](./media/active-directory-saas-arcgis-tutorial/IC784740.png "Configurar a URL do Aplicativo")

4.  Na página **Configurar logon único no ArcGIS**, clique em **Baixar metadados** e salve o arquivo de metadados localmente no computador.

    ![Configurar o logon único](./media/active-directory-saas-arcgis-tutorial/IC784741.png "Configurar o logon único")

5.  Em outra janela do navegador da Web, faça logon em seu site de empresa ArcGIS como um administrador.

6.  Clique em **Editar Configurações**.

    ![Editar Configurações](./media/active-directory-saas-arcgis-tutorial/IC784742.png "Editar Configurações")

7.  Clique em **Segurança**.

    ![Segurança](./media/active-directory-saas-arcgis-tutorial/IC784743.png "Segurança")

8.  Em **Logons Corporativos**, clique em **Definir Provedor de Identidade**.

    ![Logons Corporativos](./media/active-directory-saas-arcgis-tutorial/IC784744.png "Logons Corporativos")

9.  Na página de configuração **Definir Provedor de Identidade**, realize as seguintes etapas:

    ![Definir Provedor de Identidade](./media/active-directory-saas-arcgis-tutorial/IC784745.png "Definir Provedor de Identidade")

    1.  Na caixa de texto, digite o nome da sua organização.
    2.  Para **Os metadados do Provedor de Identidade Corporativo serão fornecidos usando**, selecione **Um Arquivo**.
    3.  Para carregar seu arquivo de metadados baixado, clique em **Escolher arquivo**.
    4.  Clique em **Definir Provedor de Identidade**.

10. No Portal clássico do Azure, selecione a confirmação da configuração de logon único e clique em **Concluir** para fechar a caixa de diálogo **Configurar logon único**.

    ![Configurar o logon único](./media/active-directory-saas-arcgis-tutorial/IC784746.png "Configurar o logon único")
##Configurando o provisionamento de usuários

Para permitir que os usuários do AD do Azure façam logon no ArcGIS, eles devem ser provisionados no ArcGIS.  
No caso do ArcGIS, o provisionamento é uma tarefa manual.

###Para configurar o provisionamento de usuários, execute as seguintes etapas:

1.  Faça logon em seu locatário do **ArcGIS**.

2.  Clique em **Convidar Membros**.

    ![Convidar Membros](./media/active-directory-saas-arcgis-tutorial/IC784747.png "Convidar Membros")

3.  Selecione **Adicionar membros automaticamente sem enviar um email** e clique em **Avançar**.

    ![Adicionar Membros Automaticamente](./media/active-directory-saas-arcgis-tutorial/IC784748.png "Adicionar Membros Automaticamente")

4.  Na página do diálogo **Membros**, realize as seguintes etapas:

    ![Adicionar e revisar](./media/active-directory-saas-arcgis-tutorial/IC784749.png "Adicionar e revisar")

    1.  Insira **Nome**, **Sobrenome** e **Email** de uma conta válida do AAD que você deseja provisionar.
    2.  Clique em **Adicionar e Examinar**.

5.  Examine os dados inseridos e clique em **Adicionar Membros**.

    ![Adicionar membro](./media/active-directory-saas-arcgis-tutorial/IC784750.png "Adicionar membro")

>[AZURE.NOTE] É possível usar qualquer outra ferramenta de criação da conta de usuário do ArcGIS ou as APIs fornecidas pelo ArcGIS para provisionar as contas de usuário do AAD.

##Atribuindo usuários

Para testar sua configuração, é necessário conceder acesso ao aplicativo aos usuários do Azure AD que você deseja que usem seu aplicativo.

###Para atribuir usuários ao ArcGIS, execute as seguintes etapas:

1.  No Portal clássico do Azure, crie uma conta de teste.

2.  Na página de integração do aplicativo **ArcGIS**, clique em **Atribuir usuários**.

    ![Atribuir Usuários](./media/active-directory-saas-arcgis-tutorial/IC784751.png "Atribuir Usuários")

3.  Selecione seu usuário de teste, clique em **Atribuir** e, em seguida, clique em **Sim** para confirmar a atribuição.

    ![Sim](./media/active-directory-saas-arcgis-tutorial/IC767830.png "Sim")

Se você quiser testar suas configurações de logon único, abra o Painel de Acesso. Para obter mais detalhes sobre o Painel de Acesso, veja [Introdução ao Painel de Acesso](active-directory-saas-access-panel-introduction.md).

<!---HONumber=AcomDC_0713_2016-->