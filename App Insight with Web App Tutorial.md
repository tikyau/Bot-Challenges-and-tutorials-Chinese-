開始監視 Node.js Web 應用程式
=============================

Azure Application Insights 可讓您輕鬆監視 Web
應用程式的可用性、效能和使用情形。 還可讓您快速識別並診斷應用程式的錯誤，不必等使用者回報。 從
0.20 版 SDK 開始，您可以監視常見的第三方套件，包括 MongoDB、MySQL 和 Redis。

本快速入門引導您將 0.22 版 Application Insights SDK for Node.js 新增至現有的
Node.js Web 應用程式。

先決條件
--------

若要完成本快速入門：

-   您需要 Azure 訂用帳戶和現有的 Node.js Web 應用程式。

如果您沒有 Node.js Web 應用程式，請依照[建立 Node.js Web
應用程式快速入門](https://docs.microsoft.com/azure/app-service/app-service-web-get-started-nodejs)來建立。

如果您沒有 Azure
訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/) 。

登入 Azure 入口網站
-------------------

登入 [Azure 入口網站](https://portal.azure.com/)。

啟用 Application Insights
-------------------------

Application Insights
可以從任何連上網際網路的應用程式收集遙測資料，而不論應用程式在內部部署或雲端中執行。 請使用下列步驟來開始檢視此資料。

1.  選取 [建立資源] \> **[監視 + 管理]** \> **[Application Insights]**。

>   設定方塊隨即出現，請使用下表來填寫輸入欄位。

| **設定**         | **值**           | **說明**                                     |
|------------------|------------------|----------------------------------------------|
| **名稱**         | 通用唯一值       | 此名稱可識別您要監視的應用程式               |
| **應用程式類型** | Node.js 應用程式 | 您要監視的應用程式類型                       |
| **資源群組**     | myResourceGroup  | 用於裝載 App Insights 資料之新資源群組的名稱 |
| **位置**         | 美國東部         | 選擇您附近或接近應用程式裝載位置的地點       |

1.  按一下頁面底部的 [新增] 。

設定 App Insights SDK
---------------------

1.  選取 [概觀] \> **[程式集]** \> 複製應用程式的 [檢測金鑰]。

2.  將 Application Insights SDK for Node.js
    新增至您的應用程式。 從應用程式的根資料夾執行：

>   bash複製

>   npm install applicationinsights --save

1.  編輯應用程式的第一個 .js
    檔案，將下列兩行新增至指令碼的最頂端部分。 如果您使用 [Node.js
    快速入門應用程式](https://docs.microsoft.com/azure/app-service/app-service-web-get-started-nodejs)，請修改
    index.js 檔案。 將 \<instrumentation_key\> 取代為您應用程式的檢測金鑰。

>   JavaScript複製

>   const appInsights = require('applicationinsights');

>   appInsights.setup('\<instrumentation_key\>').start();

1.  重新啟動您的應用程式。

**注意**

經過 3-5
分鐘，資料就會開始出現在入口網站。 如果此應用程式是低流量測試應用程式，請記住，只有在使用中的要求或作業發生時，才會擷取大部分的計量。

在 Azure 入口網站中開始監視
---------------------------

1.  現在，您可以在 Azure 入口網站中重新開啟 Application Insights [概觀] 頁面
    (您先前在此擷取檢測金鑰)，以檢視目前執行中應用程式的詳細資料。

2.  按一下
    [應用程式對應]，以顯示應用程式元件之間相依性關聯性的視覺化配置。 每個元件會顯示負載、效能、失敗和警示等
    KPI。

3.  按一下 [應用程式分析] 圖示 。 這樣會開啟 **Application Insights
    Analytics**，它提供一種豐富查詢語言，可用於分析 Application Insights
    收集的所有資料。 此案例中會為您產生查詢，可將要求計數以圖表呈現。 您可以撰寫自己的查詢來分析其他資料。

4.  返回 [概觀] 頁面，檢查
    [健康情況概觀時間軸]。 此儀表板會提供應用程式健康情況的統計資料，包括連入要求數量、這些要求的持續時間，以及任何發生的失敗。

>   若要在 [網頁檢視載入時間]
>   圖表中填入**用戶端遙測**資料，請將此指令碼新增至您要追蹤的每個頁面：

>   HTML複製

>   \<!--

>   To collect end-user usage analytics about your application,

>   insert the following script into each page you want to track.

>   Place this code immediately before the closing \</head\> tag,

>   and before any other scripts. Your first data will appear

>   automatically in just a few seconds.

>   \--\>

>   \<script type="text/javascript"\>

>   var appInsights=window.appInsights\|\|function(config){

>   function i(config){t[config]=function(){var
>   i=arguments;t.queue.push(function(){t[config].apply(t,i)})}}var
>   t={config:config},u=document,e=window,o="script",s="AuthenticatedUserContext",h="start",c="stop",l="Track",a=l+"Event",v=l+"Page",y=u.createElement(o),r,f;y.src=config.url\|\|"https://az416426.vo.msecnd.net/scripts/a/ai.0.js";u.getElementsByTagName(o)[0].parentNode.appendChild(y);try{t.cookie=u.cookie}catch(p){}for(t.queue=[],t.version="1.0",r=["Event","Exception","Metric","PageView","Trace","Dependency"];r.length;)i("track"+r.pop());return
>   i("set"+s),i("clear"+s),i(h+a),i(c+a),i(h+v),i(c+v),i("flush"),config.disableExceptionTracking\|\|(r="onerror",i("_"+r),f=e[r],e[r]=function(config,i,u,e,o){var
>   s=f\&&f(config,i,u,e,o);return s!==!0&&t["_"+r](config,i,u,e,o),s}),t

>   }({

>   instrumentationKey:"\<insert instrumentation key\>"

>   });

>   window.appInsights=appInsights;

>   appInsights.trackPageView();

>   \</script\>

1.  按一下 [調查] 標題下的
    [瀏覽器]。 您在這裡可以找到應用程式頁面效能的相關計量。 您可以按一下
    [新增新的圖表] 來建立額外的自訂檢視，或選取 [編輯]
    來修改現有圖表的類型、高度、調色盤、群組和計量。
