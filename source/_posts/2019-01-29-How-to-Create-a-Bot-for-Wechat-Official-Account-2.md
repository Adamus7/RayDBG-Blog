---
layout: '[post]'
title: 给微信公众号加个Bot（二）：BotBuilder
date: 2019-01-29 17:53:03
tags:
- Azure
- Microsoft Bot Framework
- WeChat
---
微软为对话机器人（Chatbot）的开发提供了一整套的解决方案，最主要的部件包括Azure Bot Service和BotBuilder。其中Azure Bot Service用来在云端运行开发好的Bot应用，BotBuilder是一个开源的Bot开发工具，包括了SDK，CLI，Emulator和一个Webchat client。BotBuilder作为Microsoft Bot Framework的一个亮点，给予开发者极大的自由度去设计和完成一个Bot应用的开发。在这里，我希望记录一下如何理解BotBuilder，以及如何利用BotBuilder SDK来开发一个Bot应用。
<!-- more -->
> BotBuilder SDK V4作为最新版本，和前一代V3相比，几乎是全新的一套SDK。从概念到细节都听取了开发者的大量意见，并做出了巨大的改变。因此，这里就只关注基于V4版本的开发。

# 初识BotBuilder
对于开发者来说，一套趁手的开发套件需要包括成熟的SDK，自动化工具，调试工具，测试和部署工具。BotBuilder为Chatbot的开发提供如下工具：
1.  Bot Builder V4 SDK，支持C#，JavaScript, Java(preview)，Python(Preview)四种开发语言。
2.  Bot Framework Emulator，本地化的Chatbot的调试工具。
3.  Bot Builder CLI tools，为不同的Bot组件提供命令行工具，方便实现自动化的开发部署工作。
4.  Bot Framework webchat，可以定制化的chatbot web前端，可以把开发好的Bot直接集成到现有的网页中。
5.  [BotBuilder Samples](https://github.com/microsoft/botbuilder-samples)，实现不同功能的Sample Code。

# Bot是如何工作的
在上手BotBuilder SDk之前，最好先简单了解一些基本概念和流程。
## Activity
在BotBuilder SKD的设计中，Activity是会话系统中最基本的信息单元对象。任何用户和Bot之间的交互都需要通过一个Activity Object来承载，用户发送一条文字信息是一个activity，用户加入对话是一个activity，用户正在输入这个状态也是一个activity。从BotBuilder的源码中可以看到目前定义的Activity有如下类型：
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
会话，必然是一来一回，一个Turn就是从用户给Bot发送一个Activity开始，到Bot给用户返回一条Activity结束的一个过程。把Turn这种行为抽象出来的目的是为了Bot可以获取Turn Context。前面我们知道Activity只定义了消息的结构，那么这个消息是谁发的，发给谁的，怎么处理这个消息等等也同样非常重要，我们需要一个Turn Context来承载。可以看一下ITurnContext这个接口的定义
```csharp
//botbuilder-dotnet/libraries/Microsoft.Bot.Builder/ITurnContext.cs
//注释已经删除
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
Activity和Turn是Bot会话层面的抽象设计，和实际数据的传输是隔离的。我们当前的Bot应用本质上是一个基于HTTP协议的Web应用。所以在设计上，Adapter的作用是利用HTTP协议发送和接受Activity，并在接收到Activity的时候创建相应的TurnContext，偶尔还需要处理Authentication相关的事务。另外，Adapter还一个重要的功能是调用定义的Middleware（见下）。Adapter通过HTTP POST的Body接收一个JSON格式的Activity，实例化一个Activity对象，再创建对应的TurnContext对象，然后调用Middlerware作对应的处理，最后送给Bot做相应的逻辑处理。
{% asset_img bot-builder-activity-processing-stack.png 500 %}

## Middleware
Middleware工作在Adaptor和Bot逻辑单元之间，跟其他的消息系统的middleware的作用类似，如果有些处理工作对每个Activite都要做，那么可以抽象成一个Middleware放在处理的pipeline上。举例来说，如果你的Bot只能处理英文消息，那你肯定希望Bot的输入消息都是English。如果收到一个中文消息，则可以调用一个Translator的Middlerware，在message到达Bot之前先把它翻译成英文。反过来，也可以通过Middleware把发送的message翻译成用户需要的语言。Bot SDK的[文档](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0)的另外一个例子如下：
{% asset_img bot-builder-dialog-state-solution.png 500 %} 