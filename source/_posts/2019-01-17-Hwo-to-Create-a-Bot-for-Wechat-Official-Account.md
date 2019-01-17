---
layout: '[post]'
title: 给微信公众号加个Bot(一)
date: 2019-01-17 10:42:23
tags:
---
Bot，会话助手类的工具，一直有着相当广泛的应用。最早的Bot可能类似电信的电话查询系统：“查询账单请按1，修改密码请按2。。。”。这种机械式的应用渐渐被更为智能的智能对话助理所代替，典型代表有Siri和Cortana这样个人助理应用。对于个人和企业，如何开发一个具有一定业务能力的Bot，一直是一个比较热门的话题。在这篇博客中，我将记录如何基于Micrsoft Bot Framework为微信公众号添加一个智能对话机器人。

# 背景
在商业环境中，一个Bot不仅要有趣，还要有业务能力。一个优秀的Bot要有自然语言识别能力，上下文处理能力和在对话中实现业务的能力。例如，如果要开发一个Bot来处理一家咖啡店的订座服务。这个Bot需要像服务员一样能够理解客户的语言，了解客户的意图，获取语句中的关键信息（时间，数量，地点等），用对话方式在前台完成业务。
Microsoft Bot Framework提供了一套Bot开发的框架和工具，包括Azure Bot Service，Bot Builder SDK和相对丰富的Bot Channels。这套工具可以帮助开发者在短时间内开发出一个个人或企业级Bot。这套Framework可以帮助开发者轻松的集成自然语言处理服务，认知服务和打通不同的沟通渠道（channels）。Bot Channels目前支持了不少流行的社交和通信应用，因此开发者只要开发一套Bot，就可以部署到不同的社交通信应用中，包括Facebook Pages, Microsoft Teams，Skype和Slack等。
微信朋友圈和公众号作为一个非常特殊的社交网络，内容非常丰富，但生态也相对封闭。给公众号加个Bot，说不定还挺好玩的。目前Bot Channels并没有官方的支持微信和公众号Channel。虽然公众号本身提供了基于关键字的自动回复功能，但是这个功能业务能力几乎没有。公众号同时也提供了开发接口，可以把用户输入的消息转发到设定的服务器上。这就给集成Bot服务提供了可能。

# 公众号开发设置
首先需要申请注册公众号。然后接入微信公众平台开发，开发者需要按照如下步骤完成：`填写服务器配置`, `验证服务器地址的有效性`, `依据接口文档实现业务逻辑`。
## 填写服务器配置
{% asset_img dev-config-1.png 500 %}
这里需要填写服务器接口URL，注意这里微信只会给这个URL转发消息和获取回复。Token可以随便填一个。EncodingAESKey可以随机生成一个，用以后面的消息加密。加密方式可以按需设置。
## 服务器严重
点击提交时，微信会发送一个HTTP GET请求到设定的服务URL上，服务器必须的原样返回请求中所带的echostr才能通过验证。因此，在提交前，要保证你的的服务URL可以正常工作。这里我在Azure App Services上部署了一个基于ASP.NET Core的Web API应用。
```csharp
[HttpGet("")]
public string ReturnEchostr([FromQuery]string signature, [FromQuery]string nonce, [FromQuery]string timestamp, [FromQuery]string echostr)
{
        if (WXHelper.IsMessageFromWX(signature, nonce, timestamp, WxToken))
    {
            return echostr;
    }
    else
    {
            return "Failed to authenticate the request";
    }
}
```
根据微信开发介入指南的规则来验证请求来自微信，C#样例代码：
```csharp
public static bool IsMessageFromWX(string signature, string nonce, string timestamp, string wxToken)
{
    string[] tempArr = { nonce, timestamp, wxToken };
    Array.Sort(tempArr);
    string tempStr = string.Join("", tempArr);
    var sha1 = SHA1.Create();
    byte[] hashBytes = sha1.ComputeHash(Encoding.ASCII.GetBytes(tempStr));
    string calculatedSignature = BitConverter.ToString(hashBytes).Replace("-", "").ToLower();
    return calculatedSignature == signature;
}

```
验证URL有效性成功后即接入生效，成为开发者。

# 服务开发
## 消息的转发
上面我们提到，Azure Bot Service还没有官方的微信Channel，因此我们需要通过程序把用户在微信公众号中输入的消息转发给Bot。这里需要用到Bot的一个特殊Channel, **Directline Channel**。 Directline Channel支持开发者通过HTTP GET或者WebSocket stream来发送消息给Bot。所以，我们需要把开发服务器上收到的用户消息，通过Directline发送给Bot；再通过Directline获取Bot的回复，最后把返回给用户。
这里需要注意的是：微信公众号不支持主动消息（特定动作交互除外，具体见[这里](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140547)），必须是客户先说话，服务器紧接着只能回复一条消息，而通过Microsfot Bot Framework开发的Bot则没有这个要求。因此，在设计Bot的对话逻辑的时候，要注意兼容微信公众号的这种形式。服务器回复的消息内容为一个XML格式的字符串，文本消息的XML格式如下：
```
<xml> <ToUserName>< ![CDATA[toUser] ]></ToUserName> <FromUserName>< ![CDATA[fromUser] ]></FromUserName> <CreateTime>12345678</CreateTime> <MsgType>< ![CDATA[text] ]></MsgType> <Content>< ![CDATA[你好] ]></Content> </xml>"
```
Bot端的功能开发我希望在另外一篇博客中详细的解释，这里用一个样例Bot作为接入演示。
## DirectLine的配置
在Azure Bot Service上， 需要开启DirectLine Channel并获取对应的接入Secret Key。
{% asset_img bot-driectline.png 500 %}
在之前的.NET Core Web API应用中，我们可以用`Microsoft.Bot.Connector.DirectLine`来帮我们简化开发工作，避免手动的构造HTTP请求。在Directline中，一个完整的简单对话流程如下：
1. Start Conversation
2. Send an activity to the bot
3. Receive activities from the bot
4. End a conversation

通过另一个需要注意的地方是，针对不同用户发过来的消息，需要建立不同的对话。

