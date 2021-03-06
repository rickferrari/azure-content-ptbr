<properties 
    pageTitle="Tutorial: Configurando o Workday para sincronização de entrada | Microsoft Azure" 
    description="Saiba como usar Workday como fonte de dados de identidade no Active Directory do Azure." 
    services="active-directory" 
    authors="MarkusVi"  
    documentationCenter="na" 
    manager="femila"/>
<tags 
    ms.service="active-directory" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.tgt_pltfrm="na" 
    ms.workload="identity" 
    ms.date="07/13/2016" 
    ms.author="markvi" />


#Tutorial: Configurando o Workday para sincronização de entrada


O objetivo deste tutorial é mostrar as etapas necessárias para executar no Workday e no AD do Azure para importar as pessoas do Workday para o AD do Azure.

O cenário descrito neste tutorial pressupõe que você já tem os seguintes itens:

-   Uma assinatura válida do Azure AD Premium
-   Um locatário no Workday
  
O cenário descrito neste tutorial consiste nos seguintes blocos de construção:

1. Habilitando a integração de aplicativos para o Workday


2. Criação de um usuário do sistema de integração


3. Criação de um grupo de segurança


4. Atribuindo o usuário do sistema de integração ao grupo de segurança


5. Configurando opções de grupo de segurança


6. Ativando alterações de política de segurança


7. Configurando a importação de usuários no AD do Azure



##Habilitando a integração de aplicativos para o Workday
  
O objetivo desta seção é descrever como habilitar a integração de aplicativos para o Workday.

### Etapas:

1.  No Portal clássico do Azure, no painel de navegação à esquerda, clique em **Active Directory**.

    ![Active Directory](./media/active-directory-saas-workday-inbound-tutorial/IC700993.png "Active Directory")

2.  Na lista **Diretório**, selecione o diretório para o qual você deseja habilitar a integração de diretórios.

3.  Para abrir a visualização dos aplicativos, na exibição do diretório, clique em **Aplicativos** no menu principal.

    ![Aplicativos](./media/active-directory-saas-workday-inbound-tutorial/IC700994.png "Aplicativos")

4.  Clique em **Adicionar** na parte inferior da página.

    ![Adicionar aplicativo](./media/active-directory-saas-workday-inbound-tutorial/IC749321.png "Adicionar aplicativo")

  
5. Na caixa de pesquisa, digite **Workday**.

    ![Adicionar um aplicativo da galeria](./media/active-directory-saas-workday-inbound-tutorial/IC701021.png "Adicionar um aplicativo da galeria")

6. No painel de resultados, selecione Workday e clique em Concluir para adicionar o aplicativo.

    ![Galeria de aplicativos](./media/active-directory-saas-workday-inbound-tutorial/IC701022.png "Galeria de aplicativos")





## Criação de um usuário do sistema de integração

### Etapas:


1. No **Workday Workbench**, insira Criar usuário na caixa de pesquisa e clique em **Criar Usuário do Sistema de Integração**.

	![Criar usuário](./media/active-directory-saas-workday-inbound-tutorial/IC750979.png "Criar usuário")



2. Conclua a tarefa **Criar Usuário do Sistema de Integração** fornecendo um nome de usuário e senha para um novo Usuário do Sistema de Integração. Deixe a opção Exigir nova senha no próximo logon desmarcada, porque esse usuário vai efetuar logon por meio de programação. Deixe Minutos de tempo limite de sessão com seu valor padrão de 0, o que impedirá que as sessões do usuário expirarem prematuramente.

	![Criar usuário do sistema de integração](./media/active-directory-saas-workday-inbound-tutorial/IC750980.png "Criar usuário do sistema de integração")
 




## Criação de um grupo de segurança

Para o cenário descrito neste tutorial, você precisa criar um grupo de segurança do sistema de integração irrestrita e atribuí-lo a ele.

### Etapas:

1. Insira Criar grupo de segurança na caixa de pesquisa e clique em **Criar Grupo de Segurança**.

	![Grupo CreateSecurity](./media/active-directory-saas-workday-inbound-tutorial/IC750981.png "Grupo CreateSecurity")
 

2. Conclua a tarefa de criar o grupo de segurança. Selecione Grupo de segurança de sistema de integração — sem restrições no menu suspenso Tipo de grupo de segurança com locatários, para criar um grupo de segurança ao qual os membros serão adicionados explicitamente.

	![Grupo CreateSecurity](./media/active-directory-saas-workday-inbound-tutorial/IC750982.png "Grupo CreateSecurity")
 



## Atribuindo o usuário do sistema de integração ao grupo de segurança

### Etapas:


1. Insira Editar grupo de segurança na caixa de pesquisa e clique em **Editar Grupo de Pesquisa**.

	![Editar grupo de segurança](./media/active-directory-saas-workday-inbound-tutorial/IC750983.png "Editar grupo de segurança")
 
 

2. Procure e selecione o novo grupo de segurança de integração por nome.

	![Editar grupo de segurança](./media/active-directory-saas-workday-inbound-tutorial/IC750984.png "Editar grupo de segurança")

 

3. Adicione o novo usuário do sistema de integração ao novo grupo de segurança.

	![Grupo de segurança do sistema](./media/active-directory-saas-workday-inbound-tutorial/IC750985.png "Grupo de segurança do sistema")




## Configurando opções de grupo de segurança

Nesta etapa, você deve conceder permissões ao novo grupo de segurança para operações **Get** e **Put** nos objetos protegidos pelas seguintes políticas de segurança de domínio:

- Provisionamento de conta externa

- Dados de trabalho: Relatórios de trabalho público

- Dados de trabalho: Todas as posições

- Dados de trabalho: Informações atuais sobre a equipe

- Dados de trabalho: Cargo de negócios no perfil de trabalhado

 
### Etapas:

1. Insira as políticas de segurança de domínio na caixa de pesquisa e, em seguida, clique no link Políticas de segurança de domínio para a área funcional.

	![Políticas de segurança do domínio](./media/active-directory-saas-workday-inbound-tutorial/IC750986.png "Políticas de segurança do domínio")
 

2. Pesquise o sistema e selecione a área funcional **Sistema**. Clique em **OK**.

	![Políticas de segurança do domínio](./media/active-directory-saas-workday-inbound-tutorial/IC750987.png "Políticas de segurança do domínio")


3. Na lista de políticas de segurança para a área funcional do sistema, expanda a administração de segurança e selecione a política de segurança de domínio, Provisionamento de conta externa.

	![Políticas de segurança do domínio](./media/active-directory-saas-workday-inbound-tutorial/IC750988.png "Políticas de segurança do domínio")


4. Clique no botão **Editar Permissões** e, em seguida, na página do diálogo **Editar Permissões**, adicione o novo grupo de segurança à lista de grupos de segurança com as permissões de integração **Get** e **Put**.

	![Editar permissões](./media/active-directory-saas-workday-inbound-tutorial/IC750989.png "Editar permissões")

 

5. Repita a etapa 1 acima para retornar à tela de seleção de áreas funcionais e, desta vez, pesquise pela equipe, selecione a área funcional da Equipe e clique em **OK**.

	![Políticas de segurança do domínio](./media/active-directory-saas-workday-inbound-tutorial/IC750990.png "Políticas de segurança do domínio")
 

6. Na lista de políticas de segurança para a área funcional de equipe, expanda Dados de trabalho: equipe e repita a etapa 4 acima para cada uma das políticas de segurança restantes:

     - Dados de trabalho: Relatórios de trabalho público

     - Dados de trabalho: Todas as posições

     - Dados de trabalho: Informações atuais sobre a equipe

     - Dados de trabalho: Cargo de negócios no perfil de trabalhado


	![Políticas de segurança do domínio](./media/active-directory-saas-workday-inbound-tutorial/IC750991.png "Políticas de segurança do domínio")







## Ativando alterações de política de segurança

### Etapas:

1. Digite Ativar na caixa de pesquisa e, em seguida, clique no link Ativar alterações de política de segurança pendentes.

	![Ativar](./media/active-directory-saas-workday-inbound-tutorial/IC750992.png "Ativar")
 

2. Inicie a tarefa Ativar Alterações de Política de Segurança Pendentes inserindo um comentário para fins de auditoria e clique em **OK**.

	![Ativar segurança pendente](./media/active-directory-saas-workday-inbound-tutorial/IC750993.png "Ativar segurança pendente")
 

3. Conclua a tarefa na próxima tela marcando a caixa de seleção Confirmar e clique em **OK**.

	![Ativar segurança pendente](./media/active-directory-saas-workday-inbound-tutorial/IC750994.png "Ativar segurança pendente")





## Configurando a importação de usuários no AD do Azure

O objetivo desta seção é descrever como configurar o AD do Azure para importar pessoas do Workday.


### Etapas:


1. Na página de integração de aplicativos do **Workday**, clique em **Configurar importação de usuários** para abrir o diálogo **Configurar Provisionamento**.


2. Na página **Configurações e credenciais do administrador**, realize as seguintes etapas e clique em **Avançar**:

	![Configurações e credenciais do administrador](./media/active-directory-saas-workday-inbound-tutorial/IC750995.png "Configurações e credenciais do administrador")
 
	a. Na caixa de texto Nome de usuário administrador do Workday, digite o nome de usuário que você criou na seção Criando um usuário do sistema de integração.

    b. Na caixa de texto Senha do administrador do Workday, digite a senha do usuário que você criou na seção Criando um usuário do sistema de integração.

    c. Na caixa de texto URL de locatário do Workday, digite a URL ou o seu locatário do Workday.


3. Na página **Testar conexão**, clique em **Iniciar teste** para confirmar a conectividade e clique em **Avançar**.

	![Testar conexão](./media/active-directory-saas-workday-inbound-tutorial/IC750996.png "Testar conexão")
 

4. Na página de opções **Provisionamento**, clique em **Avançar**.

	![Opções de provisionamento](./media/active-directory-saas-workday-inbound-tutorial/IC750997.png "Opções de provisionamento")


5. No diálogo **Iniciar provisionamento**, clique em **Concluir**.

	![Iniciar provisionamento](./media/active-directory-saas-workday-inbound-tutorial/IC750998.png "Iniciar provisionamento")
 

Agora, você pode ir para a seção **Usuários** e verificar se o usuário do Workday foi importado.



## Recursos adicionais

* [Lista de tutoriais sobre como integrar aplicativos SaaS com o Active Directory do Azure](active-directory-saas-tutorial-list.md)
* [O que é o acesso a aplicativos e logon único com o Azure Active Directory?](active-directory-appssoaccess-whatis.md)

<!---HONumber=AcomDC_0720_2016-->