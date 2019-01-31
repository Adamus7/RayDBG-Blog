---
layout: '[post]'
title: 理解Microsoft Bot Framework（一）：BotBuilder
date: 2019-01-29 17:53:03
tags:
- Azure
- Microsoft Bot Framework
- WeChat
---
微软为对话机器人（Chatbot）的开发提供了一整套的解决方案，其中主要的部件包括Azure Bot Service和BotBuilder。Azure Bot Service用来在云端运行开发好的Bot应用；[BotBuilder](https://github.com/Microsoft/BotBuilder)则是一个开源的Bot开发工具集，包括了SDK，CLI，Emulator和一个webchat client。BotBuilder作为Microsoft Bot Framework的一个亮点，给予开发者极大的自由度去设计和完成一个Bot应用的开发。在这里记录一下如何理解Microsoft Bot Framework的架构，解释相关的概念。
<!-- more -->
> BotBuilder SDK V4作为最新版本，和前一代V3相比，几乎是全新的一套SDK。从概念到细节都听取了开发者的大量意见，并做出了巨大的改变。因此，这里就只关注基于V4版本的开发。

# 初识BotBuilder
对于开发者来说，一套趁手的开发套件需要包括成熟的SDK，自动化工具，调试工具，测试和部署工具。BotBuilder为Chatbot的开发提供如下工具：
1.  Bot Builder V4 SDK，支持C#(.NETCore 2.2)，JavaScript, Java(preview)，Python(Preview)四种开发语言。
2.  Bot Framework Emulator，本地化的Chatbot的调试工具。
3.  Bot Builder CLI tools，为不同的Bot组件提供命令行工具，方便实现自动化的开发部署工作。
4.  [Bot Framework webchat](https://github.com/microsoft/botframework-webchat)，可以定制化的chatbot web前端，可以把开发好的Bot直接集成到现有的网页中。
5.  [BotBuilder Samples](https://github.com/microsoft/botbuilder-samples)，实现不同功能的Sample Code。

# Microsoft Bot SDK中的一些概念
在上手Bot SDK之前，最好先简单了解一些基本概念和流程。
## Activity
在BotBuilder SDK的设计中，Activity是会话系统中最基本的信息单元对象。任何用户和Bot之间的交互都需要通过一个Activity Object来承载，用户发送一条文字信息是一个activity，用户加入对话是一个activity，用户正在输入这个状态也是一个activity。从BotBuilder的源码中可以看到目前定义的Activity有如下类型：
```csharp
//botbuilder-dotnet/libraries/Microsoft.Bot.Schema/ActivityTypes.cs
public static class ActivityTypes
{
    public const string Message = "message";
    public const string ContactRelationUpdate = "contactRelationUpdate";
    public const string ConversationUpdate = "conversationUpdate";
    public const string Typing = "typing";
    public const string EndOfConversation = "endOfConversation";
    public const string Event = "event";
    public const string Invoke = "invoke";
    public const string DeleteUserData = "deleteUserData";
    public const string MessageUpdate = "messageUpdate";
    public const string MessageDelete = "messageDelete";
    public const string InstallationUpdate = "installationUpdate";
    public const string MessageReaction = "messageReaction";
    public const string Suggestion = "suggestion";
    public const string Trace = "trace";
    public const string Handoff = "handoff";
}
```
## Turn
会话，必然是一来一回，一个Turn就是`从用户给Bot发送一个Activity开始，到Bot给用户返回一条Activity结束`的一个过程。把Turn这种行为抽象出来的目的是为了Bot可以获取Turn Context。前面我们知道Activity只定义了信息的结构，那么这个消息是谁发的，发给谁的，怎么处理这个消息等等同样重要，我们需要一个Turn Context来承载。可以看一下ITurnContext这个接口的定义
```csharp
//botbuilder-dotnet/libraries/Microsoft.Bot.Builder/ITurnContext.cs
//注释已删除
public interface ITurnContext
{
    BotAdapter Adapter { get; }
    TurnContextStateCollection TurnState { get; }
    Activity Activity { get; }
    bool Responded { get; }
    Task<ResourceResponse> SendActivityAsync(string textReplyToSend, string speak = null,string inputHint = InputHints.AcceptingInput, CancellationToken cancellationToken =default(CancellationToken));
    Task<ResourceResponse> SendActivityAsync(IActivity activity, CancellationTokencancellationToken = default(CancellationToken));
    Task<ResourceResponse[]> SendActivitiesAsync(IActivity[] activities,CancellationTokencancellationToken = default(CancellationToken));
    Task<ResourceResponse> UpdateActivityAsync(IActivity activity,CancellationTokencancellationToken = default(CancellationToken));
    Task DeleteActivityAsync(string activityId, CancellationToken cancellationToken = defaul(CancellationToken));
    Task DeleteActivityAsync(ConversationReference conversationReference,CancellationTokencancellationToken = default(CancellationToken));
    ITurnContext OnSendActivities(SendActivitiesHandler handler);
    ITurnContext OnUpdateActivity(UpdateActivityHandler handler);
    ITurnContext OnDeleteActivity(DeleteActivityHandler handler);
}
```
## Adapter
Activity和Turn是Bot会话层面的抽象设计，和实际数据的传输是隔离的。我们当前的Bot应用本质上是一个基于HTTP协议的Web应用。所以在设计上，Adapter的作用是利用HTTP协议发送和接受Activity，并在接收到Activity的时候创建相应的TurnContext。另外，Adapter还一个重要的工作是调用定义好的Middleware（见下）。Adapter通过HTTP POST的Body接收一个JSON格式的Activity，实例化一个Activity对象，再创建对应的TurnContext对象，然后调用Middlerware作对应的处理，最后送给Bot做相应的逻辑处理。
{% asset_img bot-builder-activity-processing-stack.png 500 %}

## Middleware
Middleware工作在Adaptor和Bot逻辑单元之间，跟其他的消息系统的middleware的作用类似，如果有些处理工作对每个Activite都要做，那么可以把这类工作抽象成一个Middleware放在消息处理的pipeline上。举例来说，如果你的Bot只能处理英文消息，那你肯定希望Bot的输入消息都是英文。这时如果收到一个中文消息，则可以调用一个Translator的Middlerware，在message到达Bot之前先把它翻译成英文。反过来，也可以通过Middleware把发送的message翻译成用户需要的语言。Bot SDK的[文档](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0)中另外一个关于dialog state的middleware的例子如下：
{% asset_img bot-builder-dialog-state-solution.png 500 %}

## Bot State
对于一个智能对话系统来说，“记忆力”是非常重要的。上面定义的Turn是一个无状态的结构。一场对话（Conversation）往往有多个Turn组成，怎么在对话的上下文中“记住”一些事情是非常重要的。比如说开场的时候Bot问过了用户的姓名，后面的对话中，Bot应该记住这个信息并加以利用。在BotBuilde SDK V4中，Bot State被定义成一个Key-Value对，并提供了相应的Bot State方案，主要包括三个层面的组件：Storage，State Management和State Property Accessors。他们的关系可以见下面的示意图：
{% asset_img bot-builder-state.png 500 %}
### Storage
目前Bot SDK直接支持三种Storage方案：
1.  Memory storage;
2.  Azure Blob storage;
3.  Azure Cosmos DB (NoSQL) storage.
通过简单的配置，可以把State保存到以上三种不同的地方，而不用考虑数据结构或者数据库的设计。另外，当前是支持自定义其他Storage方案的，不在此详细记录。
### State Management
如何从上述的Storage层读取和写入数据，是State Management做的事情。开发者不需要关心具体的实现细节，SDK会自动处理数据的保存。除此支持，SDK粗略的划分了三种State的类型：`User state`, `Conversation state`, 
`Private conversation state`。例如，你可以在ConversationState里面记录`Bot在上一个Turn里面问问题`，`当前turn的主题`或者`上一个turn的主题`等等。
### State Property Accessors
Accessor为定义的State提供了`get`和`set`方法，方便开发者获取和设置数据。值得注意的是`set`方法只是把State数据存储到Cache中，要把State数据持久化到Storage中，需要调用对应State Object的`SaveChanges()`方法。如前面的例子所说，当Bot在一个Turn中获取了用户的姓名，就可以用Accessor把这个姓名数据放到UserState中。在处理后面的Turn时，如果需要再用Accessor把这个数据取出来。

## 小结
总的来说，Microsoft Bot SDK给开发者定义了一套Bot的开发范式，从Activity到Turn，再到Bot State。开发者可以专注于Bot逻辑的开发，而不需要关心和设计对话系统本身。

# Bot Framework Emulator
我们开发好的Bot是一个后端应用，并没有一个现成的前端。只有部署到Azure Bot Service上之后，我们才可以通过不同的Channel去和Bot进行对话。那么在开发阶段，Bot Framework Emulator就非常有用了。在本地调试Bot的时候，Emulator充当了一个前端的作用，可以直接和本地或者部署在云端的Bot应用进行交互，极大的方便了本地调试。
其中对调试开发最有用的功能就是Inspector，可以帮助trace activity，查看Activity的payload：
{% asset_img V4-Emulator-inspector.png 500 %}

# Bot Framework webchat
之前我们提到Bot Service有很多的Channels，用户可以通过不同的Channels来和Bot进行交互。如果开发者希望用户通过Web端和Bot进行交互，那么直接集成[Bot Framework Webchat](https://github.com/microsoft/botframework-webchat)到网页上：
```html
<!DOCTYPE html>
<html>
  <body>
    <div id="webchat" role="main"></div>
    <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
    <script>
      window.WebChat.renderWebChat({
        directLine: window.WebChat.createDirectLine({ secret: 'YOUR_BOT_SECRET_FROM_AZURE_PORTAL' }),
        userID: 'YOUR_USER_ID'
      }, document.getElementById('webchat'));
    </script>
  </body>
</html>
```
Reference:
* [How bots work](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-basics?view=azure-bot-service-4.0&tabs=cs)
* [Managing state](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0)
* [Middleware](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0)