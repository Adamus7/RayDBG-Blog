---
title: Host your Bot in Azure Functions (Node.JS)
date: 2018-05-30 11:11:39
tags:
- Azure Functions
- Node.JS
- Microsoft Bot Framework
---
Microsoft published its serverless computing solution in 2016, [Azure Functions](https://azure.microsoft.com/en-us/services/functions/). By leveraging Azure Functions, developers are able to build HTTP-based API endpoints accessible by a wide range of applications while no need to care about the platform or server configurations. Azure Bot Service also has a new family member, Azure Function Bot, which is based on Azure Functions.
*Prashant* has published a [blog](https://azure.microsoft.com/en-us/blog/announcing-general-availability-of-azure-functions/) about how to create Function Bot with C#. But it seems that there is some difference if you choose Node.JS to build you Function Bot.
<!-- more -->
# Create a Function Bot
Let us start from the official templet. Create a new Function Bot with Node.JS:
{% asset_img create-a-new-function-bot.png 500 %}
After creation, you can test your Bot in Azure Bot Service blade by clicking `Test in Web Chat`:
{% asset_img test-the-function-bot.png 500 %}
Looks great.
# Build your Bot
*I assume you have worked with Web App Bot before or have basic knowledge about Microsoft Bot Framework and Azure Functions.*
Similar to Web App Bot, you can go through and edit your code by clicking `Open this bot in Azure Functions`.
{% asset_img open-in-azure-functions.png 800 %}
{% asset_img open-in-azure-functions-2.png 800 %}
But looking at above code in Functions, it is quite complex and does not look like a simple Bot application. 
Actually, the body code of your Bot is saved in index.js:
{% asset_img index.js.png 800 %}
But you may find that any changes of index.js here wouldn't take any effect in your Bot. 
## What happens?
Azure Functions bot templates use [Azure Functions Pack](https://github.com/Azure/azure-functions-pack) for optimal performance. The key point is `funcpack` will help you to package not only your code but also the code of Bot Framework (Node.JS) into one single js script. That script is what you have seen when clicks on `messages`. A configuration file, `function.json`, also will be generated by `funcpack` which will tell Azure Functions engine where to find the executable script `"scriptFile": "../.funcpack/index.js"`:
```JavaScript
{
  "disabled": false,
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ],
  "_originalEntryPoint": false,
  "_originalScriptFile": "index.js",
  "scriptFile": "../.funcpack/index.js",
  "entryPoint": "messages"
}

```
## Workaround
Since we know the executable script is packaged by `funcpack`, we can download the source code to a local folder and edit the Bot code locally, then pack the code by `funcpack` and upload them to Azure Functions.
{% asset_img download-source-code-1.png 500 %}
{% asset_img download-source-code-2.png 500 %}
Open the source code locally with you preferred IDE.
{% asset_img bot-src.png 500 %}
Install `funcpack`: `npm install -g azure-functions-pack`.
Install the dependence (botbuiler) by running `npm install` inside `messages` folder.
Edit the code of your Bot in `messages\index.js`.
Package the code by running `funcpack pack ./` inside `bot-src`:
```
PS ..\NodeJSFuncBot-src\bot-src> funcpack pack ./
info: Generating project files/metadata
info: Webpacking project
info: Complete!
```
Publish the code to Azure Functions by [Azure Func CLI](https://github.com/Azure/azure-functions-core-tools): `func azure functionapp publish <your app name>`. Or simply, copy the contents of `.funcpack\index.js` and paste them to the `messages`.
Enjoy it.

## More words
I do believe there will be a script or a feature of Azure Functions to help users auto package the JS code on the air, just like what they did for C#.


