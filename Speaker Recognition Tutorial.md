# AI 服务之语音识别 Azure 
=========================

本文我们介绍如何使用必应的语音识别 API(Bing Speech API) 把语音转换成文本：  
![](http://img1.af18.net/mmbiz_png/7duef2MZYIXiakN1uySlKsH9OSNhElBY5PSzdB4uYvDfknqtsztaS8uhCmcLCg4H0lsiboMibNgeaApp70DdHCWXw/640?wx_fmt=png)

使用 Bing Speech API 可以轻松地开发出下面的应用：  
![](http://img1.af18.net/mmbiz_png/7duef2MZYIXiakN1uySlKsH9OSNhElBY5z8xoXibStGUf9SnINK75XricxZGYYial7k3uCogLwuVADEdJ9bnyIqWeg/640?wx_fmt=png)
你点击 "开始录音" 按钮，然后对着麦克风说话，就能够识别输出你说的内容并输出成文本。上面的截图是 Azure 官方提供的 demo，为了演示语音识别 API 的用法，我们写一个丑点的，但是可以输出详细信息的程序：  
![](http://img1.af18.net/mmbiz_png/7duef2MZYIXiakN1uySlKsH9OSNhElBY50tcavDCm1CTiaEGYpxYIxQnD4NicTRFsTBRLicyrP3E8tqk1ZC9TGokeQ/640?wx_fmt=png)

该程序会以不同的模式识别我们 hardcode 的两段音频数据，然后输出识别的结果。其中上面的文本框会输出大量的中间识别结果，而下面的文本框则输出最终的识别结果。创建 Azure 服务  
要使用 Azure 的翻译服务需要先在 Azure 上创建对应的实例，比如我们需要先创建一个 "Bing Speech API" 服务实例：  
![](http://img1.af18.net/mmbiz_png/7duef2MZYIXiakN1uySlKsH9OSNhElBY5vpy2E7lK1x80EGbIG5zTT69uvU3JiawG7UnzpJzbey3cbqsibicXE7D8g/640?wx_fmt=png)

说明：对于学习和练习来说，你可以创建免费的 Azure 账号并创建免费版的上述实例，详细信息请参考 Azure 官网。创建 WPF 程序  
Bing Speech API 服务同时提供了 REST API 和客户端类库，因为 REST API 提供的服务会有一些限制，所以我们在演示程序中使用客户端类库。客户端类库分为 x86 和 x64 两个版本，笔者引用的是 x64 的版本 Microsoft.ProjectOxford.SpeechRecognition-x64：  
![](http://img1.af18.net/mmbiz_png/7duef2MZYIXiakN1uySlKsH9OSNhElBY5171rMBc0kgVP0VyXiaLy3QrPf1uOpCzuX196Rwc9RHQdv4EKic3ZMiapQ/640?wx_fmt=png)  

因而需要把工程的 platform target 也设置为 x64。

需要注意的是，Azure 提供的认知服务 API 都是需要认证信息的。具体的方式就是把我们创建的服务的 key 随 API 发送的服务器端进行认证。你可以在创建的服务实例的详情界面获得对应的 key，我们在程序中通过定义的常量来保存它们：  
const string SUBSCRIPTIONKEY = "your bing speech API key";  

由于 demo 的代码比较长，为了能集中精力介绍 Azure AI 相关的内容，本文中只贴出相关的代码。完整的 demo 代码在这里。识别模式  
语音识别区分不同的识别模式来应对不同的使用场景，如对话模式、听写模式和交互式模式。  

-对话模式(conversation) 在对话模式中，使用者参与的是人与人之间的对话。  
-听写模式(dictation) 在听写模式中，使用者说出一段较长的语音然后等待语音识别的结果。  
-交互式模式(interactive) 在交互模式中, 使用者发出简短的请求, 并期望应用程序执行响应操作。  

类库中提供一个叫 SpeechRecognitionMode 的枚举：  

public enum SpeechRecognitionMode{ ShortPhrase = 0, LongDictation = 1}  
它定义了ShortPhrase和LongDictation两种识别模式。ShortPhrase 模式最长支持 15 秒的语音。语音数据被分块发送到服务端，服务端会及时的返回部分的识别结果，所以客户端会收到多个部分结果和一个包含多个 n-best 选项的最终结果。LongDictation 模式支持最长两分钟的语音。语音数据被分块发送到服务器，根据服务端分辨出的语句间的停顿，客户端会受到多个部分结果和多个最终结果。  
代码中我们要通过它们来告诉语音识别 API 执行识别的类型。比如要识别比 15s 短的语音，可以使用 ShortPhrase 模式构建 CreateDataClient 类型的实例：  
// 使用工厂类型的 CreateDataClient 方法创建 DataRecognitionClient 类型的实例。this.dataClient = SpeechRecognitionServiceFactory.CreateDataClient( SpeechRecognitionMode.ShortPhrase , // 指定语音识别的模式。 "en-US", // 我们把语音中语言的类型 hardcode 为英语，因为我们的两个 demo 文件都是英语语音。 SUBSCRIPTIONKEY); // Bing Speech API 服务实例的 key。 

如果要识别长于 15s 的语音，就需要使用 SpeechRecognitionMode.LongDictation 模式。分块传输音频  
为了能得到近乎实时的识别效果，我们必须把音频数据以适当大小的块连续发送给服务端，下面代码中使用的块大小为 1024：  

/// /// 向服务端发送语音数据。/// /// wav 格式文件的名称。private void SendAudioHelper(string wavFileName){ using (FileStream fileStream = new FileStream(wavFileName, FileMode.Open, FileAccess.Read)) { // Note for wave files, we can just send data from the file right to the server. // In the case you are not an audio file in wave format, and instead you have just // raw data (for example audio coming over bluetooth), then before sending up any // audio data, you must first send up an SpeechAudioFormat descriptor to describe // the layout and format of your raw audio data via DataRecognitionClient's sendAudioFormat() method. int bytesRead = 0; // 创建大小为 1024 的 buffer。 byte[] buffer = new byte[1024]; try {do{ // 把文件数据读取到 buffer 中。 bytesRead = fileStream.Read(buffer, 0, buffer.Length); 

// 通过 DataRecognitionClient 类型的实例把语音数据发送到服务端。 this.dataClient.SendAudio(buffer, bytesRead);}while (bytesRead > 0); } finally {// 告诉服务端语音数据已经传送完了。this.dataClient.EndAudio(); } }}  


注意，在数据传送结束后需要通过 EndAudio() 方法显式的告诉服务端数据传送结束。部分结果与最终结果  
部分结果把数据分块发送给语音识别服务端，我们就能得到近乎实时的识别效果。服务器端通过 OnPartialResponseReceived 事件不断把识别的结果发送到客户端。比如 demo 中演示的 ShortPhrase 模式实例，我们会得到下面的中间结果(在上面的输出框中)：  


\--- Partial result received by OnPartialResponseReceivedHandler() ---why--- Partial result received by OnPartialResponseReceivedHandler() ---what's--- Partial result received by OnPartialResponseReceivedHandler() ---what's the weather--- Partial result received by OnPartialResponseReceivedHandler() ---what's the weather like  


在识别的过程中 OnPartialResponseReceived 事件被触发了 4 次，识别的结果也越来越完整。如果应用程序能够根据这些中间结果不断地向使用者做出反馈，则应用程序就具备了实时性。  
最终结果当使用者结束语音的输入后，demo 中就是调用了 EndAudio() 函数。语音识别服务在完成识别后会触发 OnResponseReceived 事件，我们通过下面的函数把结果输出到 UI 中：  


/// /// 把服务端返回的语音识别结果输出到 UI。/// /// 该类型的实例包含语音识别的结果。private void WriteResponseResult(SpeechResponseEventArgs e){ if (e.PhraseResponse.Results.Length == 0) { this.WriteLine("No phrase response is available."); } else { this.WriteLine("********* Final n-BEST Results *********"); for (int i = 0; i   


数据的结果大体如下：  
\--- OnDataShortPhraseResponseReceivedHandler ---********* Final n-BEST Results [*********0] Confidence=High, Text="What's the weather like?"  
上面是 ShortPhrase 模式的一个识别结果，它的特点是只有一个最终的返回结果，其中会包含多个识别结果并被称为 n-best。n-best 中的每个结果都包含 Confidence，DisplayText，InverseTextNormalizationResult，LexicalForm，MaskedInverseTextNormalizationResult 等属性，比如我们可以根据 Confidence 属性判断识别的结果是否可靠：  
  
对于 LongDictation 模式的识别，客户端事件 OnResponseReceived 会被触发多次，并返回分阶段的识别结果，结果中的内容和 ShortPhrase 模式类似。更详细的内容请大家直接看代码吧，很简单的。支持语言  
笔者图省事直接使用了 Azure 文档中提供的英语语音作为 demo 数据，其实 Bing Speech API 对中文支持还是比较全面的，现在支持的所有模式都支持中文。