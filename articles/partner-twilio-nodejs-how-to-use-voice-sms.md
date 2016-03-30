<properties 
	pageTitle="Usando o Twilio para voz, VoIP e mensagens SMS no Azure" 
	description="Saiba como fazer uma chamada telefônica e enviar uma mensagem SMS com o serviço de API do Twilio no Azure. Exemplos de códigos escritos em Node.js." 
	services="" 
	documentationCenter="nodejs" 
	authors="devinrader" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="multiple" 
	ms.workload="na" 
	ms.tgt_pltfrm="na" 
	ms.devlang="nodejs" 
	ms.topic="article" 
	ms.date="11/25/2014" 
	ms.author="wpickett"/>


# Usando o Twilio para voz, VoIP e mensagens SMS no Azure

Este guia demonstra como compilar aplicativos que se comunicam com o Twilio e o node.js no Azure.

<a id="whatis"/>
## O que é o Twilio?

Twilio é uma plataforma de API que torna mais fácil para os desenvolvedores fazer e receber chamadas telefônicas, enviar e receber mensagens de texto e inserir chamadas VoIP em aplicativos móveis nativos e baseados em navegador. Vamos ver brevemente como isso funciona antes de começar.

### Recebendo chamadas e mensagens de texto

O Twilio permite que os desenvolvedores [comprem números de telefone programáveis][purchase_phone] que podem ser usados para enviar e receber chamadas e mensagens de texto. Quando um número do Twilio recebe uma chamada de entrada ou texto, o Twilio envia uma solicitação HTTP POST ou GET ao seu aplicativo Web, solicitando que você forneça instruções sobre como lidar com o texto ou a chamada. O servidor responderá à solicitação HTTP do Twilio com [TwiML][twiml], um conjunto simples de marcas XML que contém instruções sobre como lidar com uma chamada ou um texto. Daqui a pouco, veremos exemplos de TwiML.

### Fazendo chamadas e enviando mensagens de texto

Ao fazer solicitações HTTP para a API do serviço web do Twilio, os desenvolvedores podem enviar mensagens de texto ou iniciar chamadas telefônicas de saída. Para chamadas de saída, o desenvolvedor também deve especificar uma URL que retorne instruções TwiML de como lidar com a chamada de saída depois que ela estiver conectada.

### Inserindo recursos de VoIP no código da interface do usuário (JavaScript, iOS ou Android)

O Twilio fornece um SDK do lado do cliente que pode transformar qualquer navegador da web da área de trabalho, aplicativo iOS ou aplicativo Android em um telefone VoIP. Neste artigo, iremos nos concentrar em como usar as chamadas VoIP no navegador. Além do SDK do JavaScript para Twilio executado no navegador, um aplicativo do lado do servidor (nosso aplicativo node.js) deve ser usado para emitir um "token de funcionalidade" para o cliente JavaScript. Você poderá ler mais sobre como usar VoIP com o node.js [no blog de desenvolvimento do Twilio][voipnode].

<a id="signup"/>
## Inscrever-se no Twilio (desconto da Microsoft)

Antes de usar os serviços do Twilio, você deve primeiro [inscrever-se em uma conta][signup]. Os clientes do Microsoft Azure receberão um desconto especial - [inscreva-se aqui][signup]!

<a id="azuresite"/>
## Criar e implantar um site do Azure Node. js

Em seguida, você precisará criar um site do node.js em execução no Azure. [A documentação oficial para esse procedimento está localizada aqui][azure_new_site]. Em um nível superior, você fará o seguinte:

* Inscrever-se para uma conta do Azure, se ainda não tiver uma
* Usar o console de administração do Azure para criar um novo site
* Adicionar o suporte de controle do código-fonte (vamos pressupor que você tenha usado o git)
* Criar um arquivo `server.js` com um aplicativo Web Node. js simples
* Implantar esse aplicativo simples no Azure

<a id="twiliomodule"/>
## Configurar o módulo Twilio

Em seguida, começaremos a escrever um aplicativo node.js simples, que usa a API do Twilio. Antes de começar, precisamos configurar nossas credenciais de conta do Twilio.

### Configurando as credenciais do Twilio nas variáveis de ambiente do sistema

Para fazer solicitações autenticadas no back-end do Twilio, precisamos do nosso SID de conta e do token de autenticação, que funcionam como o nome de usuário e a senha definida para a nossa conta do Twilio. A maneira mais segura de configurá-los para uso com o módulo de nó no Azure é por meio de variáveis de ambiente do sistema, que podem ser definidas diretamente no console de administração do Azure.

Selecione o site do node.js e clique no link "CONFIGURAR". Se você rolar para baixo um pouco, verá uma área na qual poderá definir propriedades de configuração para o seu aplicativo. Insira suas credenciais de conta do Twilio ([encontradas no seu painel do Twilio][twilio_dashboard]) conforme mostrado – não se esqueça de nomeá-las como "TWILIO\_ACCOUNT\_SID" e "TWILIO\_AUTH\_TOKEN", respectivamente:

![Console de administração do Azure][azure-admin-console]

Depois que você tiver configurado essas variáveis, reinicie o aplicativo no console do Azure.

### Declarando o módulo Twilio em package.json

Em seguida, precisaremos criar um package.json para gerenciar nossas dependências de módulo do nó via [npm]. No mesmo nível que o arquivo "server.js" criado no tutorial do Azure/node.js, crie um arquivo denominado "package.json". Dentro desse arquivo, coloque o seguinte:

  { "name": "application-name", "version": "0.0.1", "private": true, "scripts": { "start": "node server" }, "dependencies": { "express": "3.1.0", "ejs": "*", "twilio":"*" } }

Isso declara o módulo twilio como uma dependência, bem como a popular [estrutura do web express][express] e o mecanismo de modelo EJS. Agora estamos prontos e vamos escrever um código!

<a id="makecall"/>
## Fazer uma chamada de saída

Vamos criar um formulário simples que fará uma chamada para um número escolhido. Abra o server.js e insira o código a seguir. Observe onde está escrito “CHANGE\_ME” - coloque o nome do seu site do Azure nesse local:

    // Module dependencies
    var express = require('express'), 
      path = require('path'), 
      http = require('http'), 
      twilio = require('twilio');

    // Create Express web application
    var app = express();

    // Express configuration
    app.configure(function(){
      app.set('port', process.env.PORT || 3000);
      app.set('views', __dirname + '/views');
      app.set('view engine', 'ejs');
      app.use(express.favicon());
      app.use(express.logger('dev'));
      app.use(express.bodyParser());
      app.use(express.methodOverride());
      app.use(app.router);
      app.use(express.static(path.join(__dirname, 'public')));
    });
    app.configure('development', function(){
      app.use(express.errorHandler());
    });

    // Render an HTML user interface for the application's home page
    app.get('/', function(request, response) {
      response.render('index');
    });

    // Handle the form POST to place a call
    app.post('/call', function(request, response) {
      var client = twilio();
      client.makeCall({
          // make a call to this number
          to:request.body.number,

          // Change to a Twilio number you bought - see:
          // https://www.twilio.com/user/account/phone-numbers/incoming
          from:'+15558675309',

          // A URL in our app which generates TwiML
          // Change "CHANGE_ME" to your app's name
          url:'https://CHANGE_ME.azurewebsites.net/outbound_call'
      }, function(error, data) {
          // Go back to the home page
          response.redirect('/');
      });
    });

    // Generate TwiML to handle an outbound call
    app.post('/outbound_call', function(request, response) {
      var twiml = new twilio.TwimlResponse();

      // Say a message to the call's receiver 
      twiml.say('hello - thanks for checking out Twilio and Azure', {
          voice:'woman'
      });

      response.set('Content-Type', 'text/xml');
      response.send(twiml.toString());
    });

    // Start server
    http.createServer(app).listen(app.get('port'), function(){
      console.log("Express server listening on port " + app.get('port'));
    });

Em seguida, crie um diretório denominado “views” e crie nele um arquivo denominado "index.ejs" com o seguinte conteúdo:

    <!DOCTYPE html>
    <html>
    <head>
      <title>Twilio Test</title>
      <style>
        input { height:20px; width:300px; font-size:18px; margin:5px; padding:5px; }
      </style>
    </head>
    <body>
      <h1>Twilio Test</h1>
      <form action="/call" method="POST">
          <input placeholder="Enter a phone number" name="number"/>
          <br/>
          <input type="submit" value="Call the number above"/>
      </form>
    </body>
    </html>

Agora, implante o seu site no Azure e abra sua página inicial. Você deve ser capaz de inserir seu número de telefone no campo de texto e receber uma chamada do seu número do Twilio!

<a id="sendmessage"/>
## Enviar uma mensagem SMS

Agora, vamos configurar uma interface de usuário e a lógica de manipulação de formulário para enviar uma mensagem de texto. Abra o “server.js” e adicione o código a seguir após a última chamada para “app.post”:

    app.post('/sms', function(request, response) {
      var client = twilio();
      client.sendSms({
          // send a text to this number
          to:request.body.number,

          // A Twilio number you bought - see:
          // https://www.twilio.com/user/account/phone-numbers/incoming
          from:'+15558675309',

          // The body of the text message
          body: request.body.message
          
      }, function(error, data) {
          // Go back to the home page
          response.redirect('/');
      });
    });

Em “views/index.ejs”, adicione outro formulário sob o primeiro para enviar um número e uma mensagem de texto:

    <form action="/sms" method="POST">
      <input placeholder="Enter a phone number" name="number"/>
      <br/>
      <input placeholder="Enter a message to send" name="message"/>
      <br/>
      <input type="submit" value="Send text to the number above"/>
    </form>

Reimplante seu aplicativo no Azure. Agora você deve conseguir enviar o formulário e uma mensagem de texto para você mesmo (ou para qualquer um dos seus amigos mais íntimos)!

<a id="nextsteps"/>
## Próximas etapas

Agora, você aprendeu as noções básicas do uso do node.js e do Twilio para compilar aplicativos que se comunicam. Mas esses exemplos pouco abordam o que é possível fazer com o Twilio e o node.js. Para obter mais informações sobre como usar o Twilio com o node.js, verifique os seguintes recursos:

* [Documentos oficiais do módulo][docs]
* [Tutorial sobre VoIP com aplicativos Node.js][voipnode]
* [Votr - um aplicativo de votação SMS em tempo real com node.js e CouchDB (três partes)][votr]
* [Programação de par no navegador com node.js][pair]

Esperamos que você aprecie escrever com o node.js e o Twilio no Azure!

[purchase_phone]: https://www.twilio.com/user/account/phone-numbers/available/local
[twiml]: https://www.twilio.com/docs/api/twiml
[signup]: http://ahoy.twilio.com/azure
[azure_new_site]: /app-service-web/web-sites-nodejs-develop-deploy-mac.md
[twilio_dashboard]: https://www.twilio.com/user/account
[npm]: http://npmjs.org
[express]: http://expressjs.com
[voipnode]: http://www.twilio.com/blog/2013/04/introduction-to-twilio-client-with-node-js.html
[docs]: http://twilio.github.io/twilio-node/
[votr]: http://www.twilio.com/blog/2012/09/building-a-real-time-sms-voting-app-part-1-node-js-couchdb.html
[pair]: http://www.twilio.com/blog/2013/06/pair-programming-in-the-browser-with-twilio.html
[azure-admin-console]: ./media/partner-twilio-nodejs-how-to-use-voice-sms/twilio_1.png

<!---HONumber=AcomDC_0302_2016-->