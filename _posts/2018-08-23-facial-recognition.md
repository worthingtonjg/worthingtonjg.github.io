---
layout: post
banner_image: facial-rec.png
---
This code demonstrates how to use the Microsoft Cognitive Service Face API in a Universal Windows application.  Topics covered include: previewing a web cam feed, capturing images and video, detecting faces locally, saving captured frames to your pictures library, analyzing images for faces facial attributes, such as emotion, match faces to people, adding new identies based on those images.

> Please note, this was written to demonstrate how to use Azure Cognitive services.  I am not following very many (if any) good coding practices, catching very few errors, just barfing everything in the code behind, etc. If this was a production application your code would likely be more complex and involved.*

## Github Repository

You can download the working code from this github repository ...

<https://github.com/worthingtonjg/FacialRecognition>

## YouTube Video

I presented this demo / code walk through at a .net engineering forum for my work that I was invited to speak at.

[![Youtube Play list](/assets/images/youtube-facial-rec.png)](https://www.youtube.com/watch?v=pwptUYBXa_Q&list=PL78mPPlBvBvMXcbMYHci3VEvgWWZhbOgu){:target="_blank"}



## How to Run the Code

To run the code you will need:

  - Visual Studio 2017
  - Windows 10 November Update (Build 10586)
  - An Azure subscription 
  - A Face API Cognitive Service setup and running in Azure 
  
  > [How to create a Cognitive Services APIs account in the Azure portal](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-apis-create-account)

Steps to Run the code:

1. Clone the repo
2. Open CogApp2.sln
3. Edit MainPage.xaml.cs
4.  Change the following code, to have values for your cognitive service:

```C#
        private string _subscriptionKey = "";
        private string _apiRoot = "https://westus.api.cognitive.microsoft.com/face/v1.0";
        private string _personGroupId = "myfriends";
```

**_subscriptionKey**

In your azure portal, open your Face Cognitive Service.  Under Resource Management => Keys, copy one of your keys.

**_apiRoot**

You can actually leave the end point as is: https://westus.api.cognitive.microsoft.com/face/v1.0

**_personGroupId**

Faces are categorized and stored in person groups.  If you already have a person group, then put the id for your group here.  Otherwise set this to whatever makes sense to you.  You can leave it as "myfriends" if you want to.

**Run the app**

Now run the app on the "Local Machine".  It should run, and display a video preview.  Once things initialize, each second the video preview is analyzed and sent to the cognitive services.  The results are displayed in the text box.

If this is your first time to run it, it won't have any faces defined in the face group.  Put a name in the textbox at the bottom of the interface, and press  the "Add Person" button.  This will analyze the current frame, detect the face,  create a new person in your person group for that face, and re-train the person group to match that face to the name you entered in the textbox.

## Tutorial

Below is a step by step tutorial / explanation of the code, and how it works.  

### Step 1: Create new UWP Application
- Open Visual Studio 2017, and choose File => New Project
- In the New Project Dialog choose, under Visual C#, Windows Universal, Choose "Blank App (Universal Windows)"
- You will be prompted with a dialog to choose a target and minimum platform.  Below are the settings I chose, but these may be different depending on what is on your machine.  The minimum version cannot be any older than Build 10586.
  - Target Version: Windows 10 Fall Creators Update (10.0; Build 16299)
  - Minimum Version: Windows 10 November Update (10.0; Build 10586)

### Step 2: Add Capabilities

Edit the Package.appxmanifest and add the following capabilities:

1. Microphone
2. Pictures Library
3. WebCam (To use the web camera, we also have to enable the microphone)

### Step 3: Setup the Video Preview

Open *MainPage.xaml* and paste in the following code snippet, in place of the empty grid.  

This splits the screen into two columns and adds two controls.

The *CaptureElement* is the control that allows us to preview the web cam feed.  And the TextBox will be used to view the data we collect about the image when it is analyzed for faces.


```xaml
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition></ColumnDefinition>
            <ColumnDefinition></ColumnDefinition>
        </Grid.ColumnDefinitions>
        
        <CaptureElement Grid.Column="0" x:Name="PreviewElement"></CaptureElement>
        <TextBox Grid.Column="1" x:Name="ResultText" TextWrapping="Wrap"></TextBox>
    </Grid>
```
Open *MainPage.xaml.cs*, and add an override for *OnNavigatedTo*, and make it async.  We need setup some async code when the app launches, and we can't do async in the constructor.

```c#
	protected async override void OnNavigatedTo(NavigationEventArgs e)
	{
	}
```

At the top, add the following using statement:

```c#
using Windows.Media.Capture;
```

Then add the following private variable at the top of the MainPage class:

```c#
private MediaCapture _mediaCapture;
```

In the OnNavigatedTo method add the following code:

```c#
		_mediaCapture = new MediaCapture();
		await _mediaCapture.InitializeAsync();

		PreviewElement.Source = _mediaCapture;
		await _mediaCapture.StartPreviewAsync();
```

The *MediaCapture* class allows us to connect to the attached Web Camera.  

The *PreviewElement.Source* is set to point at _mediaCapture, allowing us to preview the video on our screen.

**Run the application**  

You should get prompted for permission to use the microphone and web camera.  If you say yes, to both you should see a video preview from the camera on your screen.

> **Note:** On my Surface Book 2 the *MediaCapture* class defaults to the front facing camera.  It is possible you have a different camera setup, so you may need to look into the documentation for more information about setting which camera to use.
>

**More Information**

>
> [View UWP Camera Documentation](https://docs.microsoft.com/en-us/windows/uwp/audio-video-camera/camera)
>

### Step 4: Add local face detection

We want to be able to detect faces locally.  This will allow us to call the cognitive services less frequently (only when faces are in the frame).  To do this we can use the built in face detection capability of Windows 10 by adding a Face Detection Effect.  

- Add the following using statements:

```c#
using Windows.Media.Core;
using System.Threading.Tasks;
using System.Diagnostics;
using Windows.UI.Core;
```

- Add a new variable at the top of the code:

```c#
private FaceDetectionEffect _faceDetectionEffect;
```

- Add the following method to your code:

```c#
	private async Task CreateFaceDetectionEffectAsync()
        {
            // Create the definition, which will contain some initialization settings
            var definition = new FaceDetectionEffectDefinition();

            // To ensure preview smoothness, do not delay incoming samples
            definition.SynchronousDetectionEnabled = false;

            // In this scenario, balance speed and accuracy
            definition.DetectionMode = FaceDetectionMode.Balanced;

            // Add the effect to the preview stream
            _faceDetectionEffect = (FaceDetectionEffect)await _mediaCapture.AddVideoEffectAsync(definition, MediaStreamType.VideoPreview);

            // Register for face detection events
            _faceDetectionEffect.FaceDetected += FaceDetectionEffect_FaceDetected;

            // Choose the shortest interval between detection events
            _faceDetectionEffect.DesiredDetectionInterval = TimeSpan.FromMilliseconds(1000);

            // Start detecting faces
            _faceDetectionEffect.Enabled = true;
        }
```

This method leverages the built in capabilities of Windows 10 to detect faces in images.

- **_faceDetectionEffect.DesiredDetectionInterval** - this is how frequently it will check for faces

- **_faceDetectionEffect.FaceDetected += FaceDetectionEffect_FaceDetected;** - This method gets called when a face is detected.

Now make sure you call *CreateFaceDetectionEffectAsync* at the bottom of your OnNavigatedTo method:

```c#
	protected async override void OnNavigatedTo(NavigationEventArgs e)
        {
		...other code ...
		await CreateFaceDetectionEffectAsync();
	}	
	
```

- Next we need to define *FaceDetectionEffect_FaceDetected*, as mentioned above this method gets called when faces are detected.

```c#
	private async void FaceDetectionEffect_FaceDetected(FaceDetectionEffect sender, FaceDetectedEventArgs args)
        {
            Debug.WriteLine($"{args.ResultFrame.DetectedFaces.Count} faces detected");

		if (args.ResultFrame.DetectedFaces.Count == 0) return;

            await Dispatcher.RunAsync(CoreDispatcherPriority.Normal, async () =>
            {
                try
                {
                    _faceDetectionEffect.FaceDetected -= FaceDetectionEffect_FaceDetected;

					// Do stuff here
					
					//string json = JsonConvert.SerializeObject(candidates, Formatting.Indented);
                    //ResultText.Text = json;
                }
                finally
                {
                    _faceDetectionEffect.FaceDetected += FaceDetectionEffect_FaceDetected;
                }
            });
        }
```
This code executes on a seperate thread.  So we need to use the Dispatcher to marshall our code to run on the UI thread, otherwise we will get errors.

Also notice we use "-=" and "+=" to remove the event listener temporarily (until we are done), so that the event doesn't fire again until the frame has been analyzed,

**Run the Application**

Run the application.  You should still get the same video preview, but if you watch the debug window, it should list how many faces are detected in the image.

**More Information**

>
> [Analyzing Camera Frames Documentation](https://docs.microsoft.com/en-us/windows/uwp/audio-video-camera/scene-analysis-for-media-capture)
>

### Step 5: Capture Frame 

The next step is to capture the frame as an image when a face is detected.  

Add the following using statements:

```c#
using Windows.UI.Xaml.Media.Imaging;
using Windows.Media;
using Windows.Graphics.Imaging;
using Windows.Media.MediaProperties;
```

Then add the following method...

```c#
	private async Task<WriteableBitmap> GetWriteableBitmapFromPreviewFrame()
        {
            var previewProperties = _mediaCapture.VideoDeviceController.GetMediaStreamProperties(MediaStreamType.VideoPreview) as VideoEncodingProperties;

            var videoFrame = new VideoFrame(BitmapPixelFormat.Bgra8, (int)previewProperties.Width, (int)previewProperties.Height);

            var frame = await _mediaCapture.GetPreviewFrameAsync(videoFrame);

            SoftwareBitmap frameBitmap = frame.SoftwareBitmap;

            WriteableBitmap bitmap = new WriteableBitmap(frameBitmap.PixelWidth, frameBitmap.PixelHeight);

            frameBitmap.CopyToBuffer(bitmap.PixelBuffer);

            // Close the frame
            frame.Dispose();
            frame = null;

            return bitmap;
        }
```

This uses the _mediaCapture object to grab the current frame and stuff it into a writeable bitmap.

Edit your FaceDetectionEffect_FaceDetected method, and call your new GetWriteableBitmapFromPreviewFrame method.

```c#
	private async void FaceDetectionEffect_FaceDetected(FaceDetectionEffect sender, FaceDetectedEventArgs args)
        {
			... other code ...
			// Do stuff here
			var bmp = await GetWriteableBitmapFromPreviewFrame();
	}
```

**More Information**

>
> [Basics of MediaCapture](https://docs.microsoft.com/en-us/windows/uwp/audio-video-camera/basic-photo-video-and-audio-capture-with-mediacapture)
> [Preview Frame](https://docs.microsoft.com/en-us/windows/uwp/audio-video-camera/get-a-preview-frame)
>

### Step 6: Save image to Pictures Library

Now we want to save the image to the pictures library.  

First we need to add a nuget package:

1. Right-click on your project and select "Manage NuGet Packages"
2. Select the "Browse" tab, and in the search box type: "WriteableBitmapEx"
3. Select the package and install it.

Add the following using statements:

```c#
using Windows.Storage;
using Windows.Storage.Streams;
using Windows.Storage.FileProperties;
```

Now add the following method:

```c#
	private async Task<StorageFile> SaveBitmapToStorage(WriteableBitmap bitmap)
        {
            var myPictures = await StorageLibrary.GetLibraryAsync(Windows.Storage.KnownLibraryId.Pictures);
            StorageFile file = await myPictures.SaveFolder.CreateFileAsync("_photo.jpg", CreationCollisionOption.ReplaceExisting);

            using (var captureStream = new InMemoryRandomAccessStream())
            {
                using (var fileStream = await file.OpenAsync(FileAccessMode.ReadWrite))
                {
                    await bitmap.ToStreamAsJpeg(captureStream);

                    var decoder = await BitmapDecoder.CreateAsync(captureStream);
                    var encoder = await BitmapEncoder.CreateForTranscodingAsync(fileStream, decoder);

                    var properties = new BitmapPropertySet {
                            { "System.Photo.Orientation", new BitmapTypedValue(PhotoOrientation.Normal, PropertyType.UInt16) }
                        };

                    await encoder.BitmapProperties.SetPropertiesAsync(properties);

                    await encoder.FlushAsync();
                }
            }

            return file;
        }
```

This method returns a StorageFile. 

Edit your FaceDetectionEffect_FaceDetected method, and call your new method:

```c#
	private async void FaceDetectionEffect_FaceDetected(FaceDetectionEffect sender, FaceDetectedEventArgs args)
        {
			... other code ...
			// Do stuff here
			var bmp = await GetWriteableBitmapFromPreviewFrame();
			var file = await SaveBitmapToStorage(bmp);		
	}
```

**Run the application**

Open you Pictures folder and run the application.  If you did everything correctly, you should see a new image named "_photo.jpg" show up every time a frame is captured that has a face.  

**More Information**
>
> [WriteableBitmapEx](https://github.com/teichgraf/WriteableBitmapEx)
> [Windows Storage Folders](https://docs.microsoft.com/en-us/uwp/api/windows.storage.knownfolders)
> [Save WriteableBitmap to Storage](https://code.msdn.microsoft.com/windowsapps/How-to-save-WriteableBitmap-bd23d455)
>

### Step 7: Add Cognitive Services

To continue, you will need to make sure you already have an Azure account setup.  You will also need to have already created a Face Cognitive Service in your account.  

I am not going to go into details here about how to do that.  It is actually really easy to do in Azure, below is a link that can help you do that.

> [How to create a Cognitive Services APIs account in the Azure portal](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-apis-create-account)

Once your cognitive service is setup, we need to pull in the correct nuget package:

1. Right-click on your project and select "Manage NuGet Packages"
2. Select the "Browse" tab
3. This time, make sure "Include Prerelease" is checked
4. In the search box type: "Microsoft.Azure.CognitiveServices.Vision.Face"
5. Install the "2.0.0-preview" version (if you install a newer version then there could be breaking changes in the API)
3. Select the package and install it.

Add the following usings:

```c#
using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
using System.Net.Http;
```

Next add the following variables at the top of the main page

```c#
	private IFaceClient _faceClient;
        private string _subscriptionKey = "<insert>";
        private string _apiRoot = "https://westus.api.cognitive.microsoft.com/face/v1.0";
        private string _personGroupId = "myfriends";
```

**_subscriptionKey**

In your azure portal, open your Face Cognitive Service.  Under Resource Management => Keys, copy one of your keys.

**_apiRoot**

You can actually leave the end point as is: https://westus.api.cognitive.microsoft.com/face/v1.0

**_personGroupId**

Faces are categorized and stored in person groups.  If you already have a person group, then put the id for your group here.  Otherwise set this to whatever makes sense to you.  You can leave it as "myfriends" if you want to.

Next at the top of your OnNavigatedTo method add:

```c#
	protected async override void OnNavigatedTo(NavigationEventArgs e)
	{
		            _faceClient = new FaceClient(new ApiKeyServiceClientCredentials(_subscriptionKey), new DelegatingHandler[] { })
            {
                BaseUri = new Uri(_apiRoot)
            };
	    
	    ... other code ...
```  
All the cognitive services are just rest calls.  The FaceClient wraps those rest calls for us, and gives an API to code against.  You don't have to use the FaceClient, you could call the rest api end points directly, but it makes it easier and faster to get started if you do.

### Step 8: Initialize your face group

Before you can't start recognizing faces, you need a face group. 

Add the following code that will initialize your face group if it does not exist...

```c#
	private async Task InitFaceGroup()
        {
            try
            {
                PersonGroup group = await _faceClient.PersonGroup.GetAsync(_personGroupId);
            }
            catch (APIErrorException ex)
            {
                if (ex.Body.Error.Code == "PersonGroupNotFound")
                {
                    await _faceClient.PersonGroup.CreateAsync(_personGroupId, _personGroupId);
		    await _faceClient.PersonGroup.TrainAsync(_personGroupId);
                }
                else
                {
                    throw;
                }
            }
        }
```

Then in your OnNavigatedTo, make sure you call this new method:

```c#
	protected async override void OnNavigatedTo(NavigationEventArgs e)
	{
		            _faceClient = new FaceClient(new ApiKeyServiceClientCredentials(_subscriptionKey), new DelegatingHandler[] { })
            {
                BaseUri = new Uri(_apiRoot)
            };
	    
	    await InitFaceGroup();
	    
	    ... other code ...
```

### Step 9: Call Detect Faces Cognitive Service

The next step is to pass the image to the Azure Cognitive service, so it can detect the faces in the image.  This method will call the service and return a list of DetectedFace's

Add this code:

```c#
	public async Task<IList<DetectedFace>> FindFaces(StorageFile file)
        {
            IList<DetectedFace> result = new List<DetectedFace>();

            using (var stream = await file.OpenStreamForReadAsync())
            {
                result = await _faceClient.Face.DetectWithStreamAsync(stream, true, false, new FaceAttributeType[]
                {
                    FaceAttributeType.Gender,
                    FaceAttributeType.Age,
                    FaceAttributeType.Smile,
                    FaceAttributeType.Emotion,
                    FaceAttributeType.Glasses,
                    FaceAttributeType.Hair,
                               });
            }

            return result;
        }
```

Make sure to call this method in FaceDetectionEffect_FaceDetected as follows ...

```c#
	private async void FaceDetectionEffect_FaceDetected(FaceDetectionEffect sender, FaceDetectedEventArgs args)
        {
            ...other code...
	    
                    // Do stuff here
                    WriteableBitmap bmp = await GetWriteableBitmapFromPreviewFrame();
                    var file = await SaveBitmapToStorage(bmp);
		    var faces = await FindFaces(file);

    
```

Add a using for ...

```c#
using Newtonsoft.Json;
```

Then at the bottom of the FaceDetectionEffect_FaceDetected, un-comment the two lines of code and change them as follows ...

```c#
		string json = JsonConvert.SerializeObject(faces, Formatting.Indented);
		ResultText.Text = json;
```

**Run the application**

Run the application, you should now see data showing up in the text box about the faces found in the analyzed frame.

**More Information**

>
> [Faces Quickstart](https://docs.microsoft.com/en-us/azure/cognitive-services/face/quickstarts/csharp)

> [Detect and Frame Faces](https://docs.microsoft.com/en-us/azure/cognitive-services/face/tutorials/faceapiincsharptutorial)

> [Detect Faces in Images](https://docs.microsoft.com/en-us/azure/cognitive-services/face/face-api-how-to-topics/howtodetectfacesinimage)

>

### Step 10: Identify the faces

Next we want to take the faces we found and identify them.  Add the following method:

```c#
	public async Task<IList<IdentifyResult>> Identify(IList<DetectedFace> faces)
	{
		if (faces.Count == 0) return new List<IdentifyResult>();

		IList<IdentifyResult> result = new List<IdentifyResult>();
	
		try
		{
			TrainingStatus status = await _faceClient.PersonGroup.GetTrainingStatusAsync(_personGroupId);

			if (status.Status != TrainingStatusType.Failed)
			{
				IList<Guid> faceIds = faces.Select(face => face.FaceId.GetValueOrDefault()).ToList();

				result = await _faceClient.Face.IdentifyAsync(_personGroupId, faceIds, null);
			}
		}
		catch(Exception ex)
		{
			Debug.WriteLine(ex.Message);
		}

		return result;
	}
```

Remember to call the method as follows ...

```c#
	private async void FaceDetectionEffect_FaceDetected(FaceDetectionEffect sender, FaceDetectedEventArgs args)
        {
            ...other code...
	    
                    // Do stuff here
                    WriteableBitmap bmp = await GetWriteableBitmapFromPreviewFrame();
                    var file = await SaveBitmapToStorage(bmp);
		    var faces = await FindFaces(file);
		    var identities = await Identify(faces);
```

Then at the bottom of the FaceDetectionEffect_FaceDetected, change the line of code that gets the json to work on "identities" instead of "faces" ...

```c#
		string json = JsonConvert.SerializeObject(identities, Formatting.Indented);
```

**Run the Application**

This is where your person group that is part of your cognitive service comes in.  If you already have people in your person group, then it will be able to identify them, and you could get data if whoever the camera is pointing at is in your group.  But if your person group is new then you should get an empty array coming back at this point.  Don't worry, we will add some faces in a minute.

### Step 11: Extract top candidates and combine all the collected data

In this step we are going to create a final object that holds all the data we've collected so far.

Add a new class to your project ...

```c#
 	using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
	
	public class Identification
	{
		public Person Person { get; set; }
		public IdentifyResult IdentifyResult { get; set; }
		public DetectedFace Face { get; set; }

		public double Confidence;
	}	
```    

Now add this method ...

```c#
	public async Task<List<Identification>> ExtractTopCandidate(IList<IdentifyResult> identities, IList<DetectedFace> faces)
        {
            var result = new List<Identification>();

            foreach (var face in faces)
            {
                var identifyResult = identities.Where(i => i.FaceId == face.FaceId).FirstOrDefault();

                var identification = new Identification
                {
                    Person = new Person { Name = "Unknown" },
                    Confidence = 1,
                    Face = face,
                    IdentifyResult = identifyResult
                };

                result.Add(identification);

                if(identifyResult != null && identifyResult.Candidates.Count > 0)
                {
                
                    // Get top 1 among all candidates returned
                    IdentifyCandidate candidate = identifyResult.Candidates[0];
                   
                    var person = await _faceClient.PersonGroupPerson.GetAsync(_personGroupId, candidate.PersonId);

                    identification.Person = person;
                    identification.Confidence = candidate.Confidence;
                }
            }

            return result;
        }
```

This method does a couple things...

1.  So far we have data on faces and identities, this combines that data into the Identification class that we created.
2.  If the person could not be identified, then his identity comes back as unknown, but we still get data on his face.
3.  In this example, I chose to only care about the first candidate that came back for the identity.  

Make sure to call your method ...

```c#
	private async void FaceDetectionEffect_FaceDetected(FaceDetectionEffect sender, FaceDetectedEventArgs args)
        {
            ...other code...
	    
                    // Do stuff here
                    WriteableBitmap bmp = await GetWriteableBitmapFromPreviewFrame();
                    var file = await SaveBitmapToStorage(bmp);
		    var faces = await FindFaces(file);
		    var identities = await Identify(faces);
		    var candidates = await ExtractTopCandidate(identities, faces);
```

Also change the line of code that gets the json to use the candidates now ...

```c#
	string json = JsonConvert.SerializeObject(candidates, Formatting.Indented);
```

**Run the application**

You should get information about the faces coming back now, if you do not have a face group, or if it cannot identify the face in 
the image then the identity should come back as unknown.

### Step 12: Add people to your person group

In this step we are going to add people into your person group.  The way I am doing it is a bit strange, but mainly it is because
I don't want to re-write a bunch of code to accomodate this feature.  If this app was real you would do this differently.

Change your MainPage.xaml where it defines the grid to look like the code below.  Basically we are adding a textbox and button...

```xaml
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition></RowDefinition>
            <RowDefinition Height="Auto"></RowDefinition>
        </Grid.RowDefinitions>
        
        <Grid.ColumnDefinitions>
            <ColumnDefinition></ColumnDefinition>
            <ColumnDefinition></ColumnDefinition>
        </Grid.ColumnDefinitions>

        <CaptureElement Grid.Column="0" x:Name="PreviewElement"></CaptureElement>
        <TextBox Grid.Column="1" x:Name="ResultText" TextWrapping="Wrap"></TextBox>

        <Grid Grid.Row="1" Grid.ColumnSpan="2">
            <Grid.ColumnDefinitions>
                <ColumnDefinition></ColumnDefinition>
                <ColumnDefinition Width="Auto"></ColumnDefinition>
            </Grid.ColumnDefinitions>
            <TextBox x:Name="PersonName" Margin="3"></TextBox>
            <Button x:Name="AddPersonButton" Click="AddPersonButton_Click" Grid.Column="1" Margin="3">Add Person</Button>
        </Grid>
    </Grid>
```
Define this variable at the top of you MainPage.xaml.cs ...

```c#
bool _addPerson;
```

Add code to handle the button click ...

```c#
        private void AddPersonButton_Click(object sender, RoutedEventArgs e)
        {
            if(PersonName.Text != null)
            {
                _addPerson = true;
            }
        }
```

When the _addPerson variable is true the method below will take the image we've taken add it to your person group, with 
the specified person name.  It will also retrain the group.

```c#
	private async Task AddPerson(StorageFile file)
        {
            if (!_addPerson) return;

            try
            {
                using (var s = await file.OpenStreamForReadAsync())
                {
                    Person newPerson = await _faceClient.PersonGroupPerson.CreateAsync(_personGroupId, PersonName.Text);
                    await _faceClient.PersonGroupPerson.AddPersonFaceFromStreamAsync(_personGroupId, newPerson.PersonId, s);
                    await _faceClient.PersonGroup.TrainAsync(_personGroupId);
                }

            }
            finally
            {
                PersonName.Text = "";
                _addPerson = false;
            }
        }
```

Call the new method as follows

```c#
	private async void FaceDetectionEffect_FaceDetected(FaceDetectionEffect sender, FaceDetectedEventArgs args)
        {
		... other code ...

		// Do stuff here
		WriteableBitmap bmp = await GetWriteableBitmapFromPreviewFrame();
                var file = await SaveBitmapToStorage(bmp);
		await AddPerson(file);
		
		... other coede ...
```

**Run the application**

When you run the application, this time you should get a textbox and button at the bottom of the UI.  If you enter a name, and 
press the button, the _addPerson variable will be set to true.  And the next time a face is detected in the frame, it will try
to add a new person to your person group using the name you specified.  The method call to train the person group could return
before the training is finished, but it won't matter, after a few frames it should be able to identify a new person.

**More Information**

>
> [Face API Documentation](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)
>
