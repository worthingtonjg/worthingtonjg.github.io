---
layout: post
title: "Building a chatbot using Artificial Intellegence and Azure Cognitive Services"
category: 
- ai
- cognitive
- services
- unity3d
meta_description: 'Artificial Intellegence and Azure Cognitive Services'
browser_title: 'Artificial Intellegence and Azure Cognitive Services'
banner_image: chatbot-banner.png
---

This code was used in a presentation I gave on Artificial Intelligence and Azure Cognitive Services at the Sort 2017 conference.  In my presentation I demonstrated how using a combination of cognitive services such as: Speech to Text, Text to Speech, QnAMaker and Image recognition you can make a fairly intelligent chatbot that can interact with you in interesting ways.

## Youtube Video

I created a youtube play list that shows the chatbot application I built and how it works.  It starts out by showing each cognitive service seperately.  But then combines
the cognitive services to create a chatbot that you can interact with by just talking.

[![Youtube Play list](/assets/images/youtube-chatbot.png)](https://www.youtube.com/watch?v=pwptUYBXa_Q&list=PL78mPPlBvBvMXcbMYHci3VEvgWWZhbOgu){:target="_blank"}

## Github

I've posted all the files on my [Sort2017](https://github.com/worthingtonjg/Sort2017/){:target="_blank"} git hub repo.

Below is a summary of the important files:


- [Presentation.zip](https://github.com/worthingtonjg/Sort2017/raw/master/Presentation.zip){:target="_blank"} includes my powerpoint and videos of my demo that I took just in case I didn't have internet access. 
- [Output](https://github.com/worthingtonjg/Sort2017/tree/master/SortDemo/Output){:target="_blank"} is the code for the UWP project, this is where you want to look first.
- [SortDemo](https://github.com/worthingtonjg/Sort2017/tree/master/SortDemo){:target="_blank"} is the code for the unity project. 

Please note this is a Unity Universal Windows application.

__You do NOT need to open the project in Unity unless you want to change the 3D model or something Unity related.__  

Instead you need to open the [Output](https://github.com/worthingtonjg/Sort2017/tree/master/SortDemo/Output){:target="_blank"} folder and open the solution there.  

This is the UWP project that was generated from Unity.  This is the UWP application that you will need to make changes to in order to point to your own azure cognitive services.

This project is interesting because I did most my code in the UWP portion of the application.  So you will need to make changes to the code in the Output folder to make things work for you.  

See below for some instructions and a walk-through of the code

## License

You are free to use this code anyway you want to.  However the [Salsa library](https://www.assetstore.unity3d.com/en/#!/content/16944){:target="_blank"} in Unity is __NOT__ open source.  If you are going to use this, you need to purchase a license for yourself of this software.  It is awesome and only costs $35.  It is what makes the 3D model speak in sync with the audio that is playing and is well worth the money.

## Getting Started Azure

Login to the azure portal and create three services:

- Bing Speech
- Face API
- Computer Vision API

Each service will have a place for:

- Managing Keys
- Endpoint

Take note of these for each service, so you can change the code to point to them as described below.

## QnaMaker

Next you will need to go to [qnamaker.ai](https://qnamaker.ai){:target="_blank"} create an account.

The application currently uses two services. 

- NavigationAI: This controls the application navigation
- Articles of Faith: This let's the application answer questions about the articles of faith
- Mormon News Room FAQ: You don't need this one, it was another one I created to play with, but you can leave it off

Create each QnaMaker Knowledgebase  as follows:

**NavigationAI**

Question | Answer 
--- | --- 
Whats in the clipboard? | {"action":1}
Did you get that? | {"action":1}
Take a photo | {"action":2}
Lets take a photo | {"action":2}
What do you see? | {"action":3}
Global Thermonuclear war | Wouldn't you prefer a good game of chess?
How about  Thermonuclear war | Wouldn't you prefer a good game of chess?
Love to.  How about Global Thermonuclear war | Wouldn't you prefer a good game of chess?
Who do you see? | {"action":4}
Can you describe the image? | {"action":5}
What celebrities do you see? | {"action":6}
What church leaders do you see? | {"action":7}
What emotions do you see? | {"action":8}

Make sure you ...

- Save and Publish the Knowledgebase
- Take note of the url and subscription key that is generated for the knowledge base after you publish it.

**Articles of Faith**

Do the same thing for this, put in as many questions related to the articles of faith (or any topic that suits you) that you want.

Then make sure you ...

- Save and Publish the Knowledgebase
- Take note of the url and subscription key that is generated for the knowledge base after you publish it.

## Change code to point at your azure services and api keys

In the **Common** project there are several helper classes that call the Azure cognitive services

- BotHelper
- FaceHelper
- SpeechAuth
- SpeechToTextHelper
- TextToSpeechHelper
- VisionHelper

Each cognitive service uses a unique *subscription key*.  You will need to modify each helper class to use your own *subscription key* since the subscription keys in the code are not valid.

You may also need to change the url that the cognitive service is pointing at.  Below are the lines of code you may need to change in each file

**VisionHelper.cs**
```c#
        21    private string _subscriptionKey = "<insert your key here>";
        22    private string _uriBase = "https://westus.api.cognitive.microsoft.com/vision/v1.0/analyze";
```

**SpeechAuth.cs**
```c#
        13    private string subscriptionKey = "5af27030917047e88ec881c84b253134"; 
        82    UriBuilder uriBuilder = new UriBuilder(@"https://api.cognitive.microsoft.com/sts/v1.0");
```

***SpeechToTextHelper.cs***
```c#
        30    string requestUri = @"https://speech.platform.bing.com/speech/recognition/interactive/cognitiveservices/v1?language=en-US";
```

***TextToSpeechHelper.cs***
```c#
        53    var request = new HttpRequestMessage(HttpMethod.Post, @"https://speech.platform.bing.com/synthesize")
```

***FaceHelper.cs***
```c#
        22    private readonly IFaceServiceClient faceServiceClient =
        23        new FaceServiceClient("a291f46e4c99492fbd1847d66f937c9f", "https://westus.api.cognitive.microsoft.com/face/v1.0");
```

***BotHelper.cs***
```c#
        39    _bots[EnumBot.MormonNewsRoomFaq] = new Bot(
                  "2bb659be5cd54ef9b7455581a117a907",  
                  @"https://westus.api.cognitive.microsoft.com/qnamaker/v2.0/knowledgebases/5649976f-c288-4ab0-ba1c-2853440c459a/generateAnswer");
        40    _bots[EnumBot.ArticlesOfFaith] = new Bot(
                  "2bb659be5cd54ef9b7455581a117a907",
                  @"https://westus.api.cognitive.microsoft.com/qnamaker/v2.0/knowledgebases/9a6ca607-6c84-415d-bbda-fbe633555131/generateAnswer");
        41    _bots[EnumBot.NavigationAI] = new Bot(
                  "2bb659be5cd54ef9b7455581a117a907",
                  @"https://westus.api.cognitive.microsoft.com/qnamaker/v2.0/knowledgebases/0e6855a2-6c2b-465b-ba36-c6fcd61df79e/generateAnswer");
```
