Nesta seção, você aprenderá a atualizar o código em seu projeto de back-end de Aplicativos Móveis para enviar uma notificação por push sempre que um novo item for adicionado. Como os clientes são registrados para notificações por push usando um registro de modelo, uma mensagem de notificação pode ser enviada por push para todas as plataformas de cliente. Cada registro de modelo de cliente contém um parâmetro *messageParam*. Quando a notificação é enviada, o *messageParam* contém uma cadeia de caracteres que é o texto do item que está sendo inserido. Para saber mais sobre como usar modelos com os Hubs de Notificação, confira [Modelos](../articles/notification-hubs/notification-hubs-templates-cross-platform-push-messages.md).

Escolha abaixo o procedimento que corresponde ao seu tipo de projeto de back-end: [back-end .NET](#dotnet) ou [back-end Node.js](#nodejs).

### <a name="dotnet"></a>Projeto de back-end .NET
1. No Visual Studio, clique com o botão direito do mouse no projeto do servidor e clique em **Gerenciar pacotes NuGet**, pesquise por `Microsoft.Azure.NotificationHubs` e clique em **Instalar**. Isso instala a biblioteca de Hubs de Notificação para enviar notificações do seu back-end.

3. No projeto do servidor, abra **Controladores** > **TodoItemController.cs** e adicione os seguintes elementos usando instruções:

		using System.Collections.Generic;
		using Microsoft.Azure.NotificationHubs;
		using Microsoft.Azure.Mobile.Server.Config;
	

2. No método **PostTodoItem**, adicione o seguinte código após a chamada para **InsertAsync**:

        // Get the settings for the server project.
        HttpConfiguration config = this.Configuration;
        MobileAppSettingsDictionary settings = 
			this.Configuration.GetMobileAppSettingsProvider().GetMobileAppSettings();
        
        // Get the Notification Hubs credentials for the Mobile App.
        string notificationHubName = settings.NotificationHubName;
        string notificationHubConnection = settings
            .Connections[MobileAppSettingsKeys.NotificationHubConnectionString].ConnectionString;

        // Create a new Notification Hub client.
        NotificationHubClient hub = NotificationHubClient
        .CreateClientFromConnectionString(notificationHubConnection, notificationHubName);

        // Sending the message so that all template registrations that contain "messageParam"
        // will receive the notifications. This includes APNS, GCM, WNS, and MPNS template registrations.
        Dictionary<string,string> templateParams = new Dictionary<string,string>();
        templateParams["messageParam"] = item.Text + " was added to the list.";

        try
        {
            // Send the push notification and log the results.
            var result = await hub.SendTemplateNotificationAsync(templateParams);

            // Write the success result to the logs.
            config.Services.GetTraceWriter().Info(result.State.ToString());
        }
        catch (System.Exception ex)
        {
            // Write the failure result to the logs.
            config.Services.GetTraceWriter()
                .Error(ex.Message, null, "Push.SendAsync Error");
        }

	Isso envia uma notificação de modelo que contém o item.Text quando um novo item todo é inserido.

4. Republicar o projeto de servidor.

### <a name="nodejs"></a>Projeto de back-end Node.js

1. Se você ainda não fez isso, [baixe o projeto de back-end de início rápido](app-service-mobile-node-backend-how-to-use-server-sdk.md#download-quickstart) ou, caso contrário, use o [editor online no portal do Azure](app-service-mobile-node-backend-how-to-use-server-sdk.md#online-editor).

2. Substitua o código existente em todoitem.js pelo código a seguir:

		var azureMobileApps = require('azure-mobile-apps'),
	    promises = require('azure-mobile-apps/src/utilities/promises'),
	    logger = require('azure-mobile-apps/src/logger');
	
		var table = azureMobileApps.table();
		
		table.insert(function (context) {
	    // For more information about the Notification Hubs JavaScript SDK, 
	    // see http://aka.ms/nodejshubs
	    logger.info('Running TodoItem.insert');
	    
	    // Define the template payload.
	    var payload = '{"messageParam": "' + context.item.text + '" }';  
	    
	    // Execute the insert.  The insert returns the results as a Promise,
	    // Do the push as a post-execute action within the promise flow.
	    return context.execute()
	        .then(function (results) {
	            // Only do the push if configured
	            if (context.push) {
					// Send a template notification.
	                context.push.send(null, payload, function (error) {
	                    if (error) {
	                        logger.error('Error while sending push notification: ', error);
	                    } else {
	                        logger.info('Push notification sent successfully!');
	                    }
	                });
	            }
	            // Don't forget to return the results from the context.execute()
	            return results;
	        })
	        .catch(function (error) {
	            logger.error('Error while running context.execute: ', error);
	        });
		});

		module.exports = table;  

	Isso envia uma notificação de modelo que contém o item.text quando um novo item todo é inserido.

2. Ao editar o arquivo no seu computador local, republique o projeto do servidor.

<!---HONumber=AcomDC_0629_2016-->