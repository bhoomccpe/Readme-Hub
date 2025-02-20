The Chat Bot API provides real-time interaction and monitoring capabilities through Microsoft Teams. This API allows users to receive notifications about the calibration process. The implementation allows the Calibration Automation System to communicate with the users and external services.
The Chat Bot API is built by Microsoft Bot Framework and integrates with Azure Bot Services to enable communication via Microsoft Teams. The architecture consists of:
- Bot Service Layer: Handling incoming messages and user interactions.
- Messaging Layer: Managing message processing and proactive notifications.
- Integration Layer: Connecting the bot to the Calibration Automation System and databases.
- Storage and Logging: Maintaining conversation persistence and logging chatbot activity for debugging.
The API is deployed on Microsoft Azure App Service via Visual Studio. The published profile and the obtained data are provided while setting up the Chat Bot Environment.

As mentioned earlier, the Chat Bot API is implemented using C# 8 with ASP.NET Core, using Microsoft.Bot.Builder SDK. The API has the following main components:
## Bot Configuration
The bot is configured in Program.cs to integrate with essential services:
```C#
  var builder = WebApplication.CreateBuilder(args);
  builder.Services.AddControllers();
  builder.Services.AddSingleton<GraphAuthService>();
  builder.Services.AddTransient<IBot, CalibratoChatBot>();
  var app = AppSetup.Configure(builder);
  app.Run();
```
With this configuration, the bot allows access to the authentication and message-handling services.

## Handling Incoming Messages
The bot processes incoming messages inside CalibratoChatBot.cs that implements the ActivityHandler class:
```C#
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    string incomingMessage = turnContext.Activity.Text;
    IActivity responseActivity = await _messageHandlerService.HandleMessage(incomingMessage);
    await turnContext.SendActivityAsync(responseActivity, cancellationToken);
}
```
This method extracts user messages and processes them before sending back the appropriate responses.

## Sending Proactive Notifications
For proactive messaging, the class ProactiveMessageService is used to send updates to users:
```C#
public async Task<string?> SendProactiveMessageWithAttachmentAsync(string userId, string chatId, string message, Attachment? attachment = null)
{
    var conversationReference = new ConversationReference { User = new ChannelAccount { Id = userId }, Conversation = new ConversationAccount(id: chatId) };
    await ((BotAdapter)_adapter).ContinueConversationAsync(_configuration["MicrosoftAppId"], conversationReference, async (ITurnContext turnContext, CancellationToken cancellationToken) => {
        await turnContext.SendActivityAsync(MessageFactory.Text(message), cancellationToken);
    }, default(CancellationToken));
    return "Message Sent";
}
``
This service allows the chatbot to send real-time calibration updates without any user interaction.
