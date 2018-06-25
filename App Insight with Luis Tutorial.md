将LUIS结果添加到Application Insight中从一个 web app bot

本教程将LUIS请求和响应信息添加到*Application
Insight*遥测数据存储。一旦你有了这些数据, 你可以用 Kusto 的语言来查询它,
或者PowerBi实时地分析、汇总和报告意图以及话语的实体。此分析可帮助您确定是否应添加或编辑LUIS应用程序的意图和实体。

在本教程中, 您将学习如何:

-   将应Application Insight添加到 web app bot 中

-   捕获并发送LUIS查询结果以Application Insight

-   查询应用程序对最高意图、分数和话语的洞察力

先决条件

-   你的LUIS Web App bot
    从[上一个教程](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-nodejs-tutorial-build-bot-framework-sample)随着Application
    Insight的打开。

本教程中的所有代码都可在*LUIS-samples github存储
库*与本教程相关的每一行都被注释为APPINSIGHT:.

与LUIS的Web App bot

本教程假定您具有如下所示的代码,
或者您已经完成了[其他教程](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-nodejs-tutorial-build-bot-framework-sample):

```javascript
JavaScriptCopy

/\*-----------------------------------------------------------------------------

This template demonstrates how to use dialogs with a LuisRecognizer to add

natural language support to a bot.

For a complete walkthrough of creating this type of bot see the article at

https://aka.ms/abs-node-luis

\-----------------------------------------------------------------------------\*/

var restify = require('restify');

var builder = require('botbuilder');

var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server

var server = restify.createServer();

server.listen(process.env.port \|\| process.env.PORT \|\| 3978, function () {

console.log('%s listening to %s', server.name, server.url);

});

// Create chat connector for communicating with the Bot Framework Service

var connector = new builder.ChatConnector({

appId: process.env.MicrosoftAppId,

appPassword: process.env.MicrosoftAppPassword,

openIdMetadata: process.env.BotOpenIdMetadata

});

// Listen for messages from users

server.post('/api/messages', connector.listen());

/\*----------------------------------------------------------------------------------------

\* Bot Storage: This is a great spot to register the private state storage for
your bot.

\* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can
implement your own!

\* For samples and documentation, see:
https://github.com/Microsoft/BotBuilder-Azure

\*
----------------------------------------------------------------------------------------
\*/

var tableName = 'botdata';

var azureTableClient = new botbuilder_azure.AzureTableClient(tableName,
process.env['AzureWebJobsStorage']);

var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false },
azureTableClient);

// Create your bot with a function to receive messages from the user

// This default message handler is invoked if the user's utterance doesn't

// match any intents handled by other dialogs.

var bot = new builder.UniversalBot(connector, function (session, args) {

session.send('You reached the default message handler. You said \\'%s\\'.',
session.message.text);

});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields

var luisAppId = process.env.LuisAppId;

var luisAPIKey = process.env.LuisAPIKey;

var luisAPIHostName = process.env.LuisAPIHostName \|\|
'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' +
luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot

var recognizer = new builder.LuisRecognizer(LuisModelUrl);

bot.recognizer(recognizer);

// Add a dialog for each intent that the LUIS app recognizes.

// See
https://docs.microsoft.com/en-us/bot-framework/nodejs/bot-builder-nodejs-recognize-intent-luis

bot.dialog('TurnOnDialog',

(session, args) =\> {

// Resolve and store any HomeAutomation.Device entity passed from LUIS.

var intent = args.intent;

var device = builder.EntityRecognizer.findEntity(intent.entities,
'HomeAutomation.Device');

// Turn on a specific device if a device entity is detected by LUIS

if (device) {

session.send('Ok, turning on the %s.', device.entity);

// Put your code here for calling the IoT web service that turns on a device

} else {

// Assuming turning on lights is the default

session.send('Ok, turning on the lights');

// Put your code here for calling the IoT web service that turns on a device

}

session.endDialog();

}

).triggerAction({

matches: 'HomeAutomation.TurnOn'

})

bot.dialog('TurnOffDialog',

(session, args) =\> {

// Resolve and store any HomeAutomation.Device entity passed from LUIS.

var intent = args.intent;

var device = builder.EntityRecognizer.findEntity(intent.entities,
'HomeAutomation.Device');

// Turn off a specific device if a device entity is detected by LUIS

if (device) {

session.send('Ok, turning off the %s.', device.entity);

// Put your code here for calling the IoT web service that turns off a device

} else {

// Assuming turning off lights is the default

session.send('Ok, turning off the lights.');

// Put your code here for calling the IoT web service that turns off a device

}

session.endDialog();

}

).triggerAction({

matches: 'HomeAutomation.TurnOff'

})

```

将Application Insight添加到 web app bot 中

目前, Application Insight, 使用在这个 web app bot, 收集通用的状态遥测为
bot。它不收集*LUIS*请求和响应信息, 您需要检查和修复您的意图和实体。

为了捕获LUIS的请求和响应, web app bot 需要**Application Insight**安装和配置的
NPM 包**app.js**文件。然后,
意向对话处理程序需要将LUIS请求和响应信息发送到Application Insight。

1.  在 Azure 门户中, 在 web app bot 服务中, 选择**建立**下的**Bot 管理**部分。

![](media/https://docs.microsoft.com/en-us/azure/cognitive-services/luis/media/luis-tutorial-appinsights/build.png)

>   搜索应用程序洞察力

1.  使用应用程序服务编辑器打开一个新的浏览器选项卡。在顶部栏中选择应用程序名称,
    然后选择**打开羚控制台**.

![](media/e17fba907239b8082ee07249a322667e.png)

>   搜索应用程序洞察力

1.  在控制台中, 输入以下命令以安装Application Insight和下划线包:

>   复制

>   cd 站点 \\wwwroot&&Npm安装applicationinsights&&Npm安装下划线

![](media/96fc3a0e07c38c51a65a84f07d098bde.png)

>   搜索应用程序洞察力

>   等待要安装的软件包:

>   复制

``` javascript
Copy

luisbot\@1.0.0 D:\\home\\site\\wwwroot

\`-- applicationinsights\@1.0.1

\+-- diagnostic-channel\@0.2.0

\+-- diagnostic-channel-publishers\@0.2.1

\`-- zone.js\@0.7.6

npm WARN luisbot\@1.0.0 No repository field.

luisbot\@1.0.0 D:\\home\\site\\wwwroot

\+-- botbuilder-azure\@3.0.4

\| \`-- azure-storage\@1.4.0

\| \`-- underscore\@1.4.4

\`-- underscore\@1.8.3

```

>   您已经完成了羚控制台浏览器选项卡。

捕获并发送LUIS查询结果Application Insight

1.  在 "应用程序服务编辑器浏览器" 选项卡中, 打开**app.js**文件。

2.  将下列 NPM 库添加到现有的需要线：



```javascript

>   // APPINSIGHT: Add underscore for flattening to name/value pairs

>   var \_ = require("underscore");

>   // APPINSIGHT: Add NPM package applicaitoninsights

>   let appInsights = require("applicationinsights");

```


1.  创建Application Insight对象并使用 web app bot
    应用程序设置**BotDevInsightsKey**:

```javascript

>   JavaScriptCopy

>   // APPINSIGHT: Set up ApplicationInsights with Web App Bot settings
>   "BotDevAppInsightsKey"

>   appInsights.setup(process.env.BotDevAppInsightsKey)

>   .setAutoDependencyCorrelation(true)

>   .setAutoCollectRequests(true)

>   .setAutoCollectPerformance(true)

>   .setAutoCollectExceptions(true)

>   .setAutoCollectDependencies(true)

>   .setAutoCollectConsole(true,true)

>   .setUseDiskRetryCaching(true)

>   .start();

>   // APPINSIGHT: Get client

>   let appInsightsClient = appInsights.defaultClient;

```

1.  添加 "**appInsightsLog**功能：

```javascript

>   JavaScriptCopy

>   // APPINSIGHT: Log LUIS results to Application Insights

>   // APPINSIGHT: must flatten as name/value pairs

>   var appInsightsLog = function(session,args) {

>   // APPINSIGHT: put bot session and LUIS results into single object

>   var data = Object.assign({}, session.message,args);

>   // APPINSIGHT: ApplicationInsights Trace

>   console.log(data);

>   // APPINSIGHT: Flatten data into name/value pairs

>   flatten = function(x, result, prefix) {

>   if(_.isObject(x)) {

>   \_.each(x, function(v, k) {

>   flatten(v, result, prefix ? prefix + '_' + k : k)

>   })

>   } else {

>   result["LUIS_" + prefix] = x

>   }

>   return result;

>   }

>   // APPINSIGHT: call fn to flatten data

>   var flattenedData = flatten(data, {})

>   // APPINSIGHT: send data to Application Insights

>   appInsightsClient.trackEvent({name: "LUIS-results", properties:
>   flattenedData});

>   }

```
>   函数的最后一行是将数据添加到Application
>   Insight中的位置。该事件的名称是**LUIS-Results**, 一个唯一的名称,
>   除了任何其他遥测数据收集此 web app bot。

1.  使用该**appInsightsLog**功能。将其添加到每个意向对话框中:

```javascript

>   JavaScriptCopy

>   // APPINSIGHT: Log results to Application Insights

>   appInsightsLog(session,args);

```

1.  要测试您的 web app bot, 请使用**在网络聊天中测试**特征。你应该看到没有区别,
    因为所有的工作是在应用程序的洞察力, 而不是在 bot 的反应。

在Application Insight查看LUIS条目

打开Application Insight查看LUIS条目。

1.  在门户中, 选择**所有资源**然后按 web app bot 名称进行筛选。Application
    Insight的图标是一个灯泡。

![](media/8792ca19482d865fb12ff316ea288c45.png)

>   搜索应用程序洞察力

1.  当资源打开时,
    单击**搜索**放大玻璃的图标在极右面板。右侧显示一个新面板。根据发现的遥测数据的多少,
    面板可能需要第二次显示。搜索LUIS结果然后点击键盘上的 enter
    键。该列表被缩小到仅在本教程中添加的LUIS查询结果。

![](media/f49c4f795b825d266019bef8bedf3bba.png)

>   筛选到依赖项

1.  选择顶部条目。新窗口将显示更详细的数据,
    包括极右查询的自定义数据。数据包括最高的意图, 和它的分数。

![](media/0fdf8f986effd3fd091c1af94309ad10.png)

>   相关性详细信息

>   完成后, 选择最右侧的顶部**X**返回到依赖项列表。

**提示**

如果要保存依赖项列表并在以后返回它, 请单击**...更**并单击 "**保存收藏夹**.

查询应用程序对意图、分数和话语的洞察力

Application
Insight为您提供了查询数据的能力。[Kusto](https://docs.microsoft.com/azure/application-insights/app-insights-analytics#query-data-in-analytics)语言,
以及将其导出到*PowerBI*.

1.  点击**分析**在依赖项列表的顶部, 在筛选器框的上方。

![](media/92dd65babc93f4b5987f93419af102c2.png)

>   分析按钮

1.  一个新窗口将打开, 上面有一个查询窗口,
    下面是一个数据表窗口。如果以前使用过数据库,
    这种安排是很熟悉的。查询包括从最近24小时开始的所有项目的名称LUIS-Results.的**CustomDimensions**列将LUIS查询结果用作名称/值对。

![](media/9f0ddabef1a6d62c83584aa5cbd0c106.png)

>   分析查询窗口

1.  要拉出最高意图、分数和话语, 请在查询窗口的最后一行中添加以下内容:

```sql

>   SQLCopy

>   \| extend topIntent = tostring(customDimensions.LUIS_intent_intent)

>   \| extend score = todouble(customDimensions.LUIS_intent_score)

>   \| extend utterance = tostring(customDimensions.LUIS_text)

```

1.  运行查询。滚动到数据表中的最右边。的新列topIntent, 得分和话语都有。单击
    "topIntent要排序的列。

![](media/3868f5b77090a7b1576bf20cd93cc35e.png)

>   分析顶部意图
