---
layout: post
title: "[HoloLens2] PhotoCapture 적용"
image: /assets/img/hololens2_logo.jpeg
---

* toc
{:toc}


이 포스트는 HoloLens 2 에서 PhotoCapture 함수를 사용하여 화면을 캡쳐하고, 이를 서버와 통신하는 내용을 정리해둔 것입니다.

## WebCam Capability

먼저 Application 이 WebCam 에 대한 접근 권한을 가질 수 있도록 설정해야 한다.

*Edit > Project Settings*  를 클릭한 다음, 아래 그림치럼 *Player > Publishing Settings > Capabilities > WebCam* 을 선택하도록 한다.

![img](/assets/img/Hololens2_start_project/01.png)

> 참고로, 이미 Application 이 build 된 상태라면, Capability 변경 내용이 반영되지 않는다. 때문에, 기존의 build 된 결과를 삭제하거나, build 된 결과의 manifest 파일에서 직접 Capability 내용을 수정해야 한다. 

## Make Button

버튼은 다양한 방법으로 생성할 수 있지만, (Unity 에 대해 별도의 지식이 없는 상태이기 때문에) 여기서는 HoloLens 쪽에서 작성해둔 패키지를 사용하고자 한다. 

먼저 [이 곳](https://github.com/microsoft/MixedRealityLearning/releases/download/getting-started-v2.4.0/MRTK.HoloLens2.Unity.Tutorials.Assets.GettingStarted.2.4.0.unitypackage)에서 HoloLens2 Tutorial 에 사용되는 패키지를 설치한다.

이후 Assets > Import Pakcage > Custom Package 를 클릭하고, 위에서 설치한 패키지를 선택하여 준다. 그리고 아래와 같이 팝업창이 뜨면 Import 버튼을 클릭해주면 된다.

![img](/assets/img/Hololens2_start_project/02.png)

패키지의 import 가 무사히 완료되었다면, 아래와 같이 패키지에 포함되어 있던 버튼들을 사용하는 것이 가능하다. 원하는 버튼을 검색한 뒤 드래그 하여, 프로젝트에 추가한다.

![img](/assets/img/Hololens2_start_project/03.png)

Object 를 추가할 때 아래와 같은 팝업창이 뜨는 경우 Import TMP Essentials 버튼을 클릭하면 된다.

![img](/assets/img/Hololens2_start_project/04.png)

## Write Scripts

먼저 Script 를 저장할 Scripts 라는 이름의 폴더를 Assets 에 생성하여 준다. 그리고 Scripts 라는 폴더에 C# Script 를 하나 생성하고, 이름을 Capture 로 수정해준다. 이후 해당 아래의 내용들을 Script 에 추가해준다.

먼저 아래와 같은 Global Variable 들을 추가해준다.

```
private PhotoCapture photoCaptureObject = null;
private Resolution cameraResolution;			// 장치의 카메라 화질
private bool _isLoading = false;				// 이미지를 처리하고 있는지 확인할 변수
public GameObject textButton;					// (Optional) 결과를 표시할 버튼
public static string url = "<your server url>";
```

다음으로 Start 함수에서 Resolution 을 설정해준다.

```
void Start()
{
cameraResolution = PhotoCapture.SupportedResolutions.OrderByDescending((res) => res.width * res.height).First();
}
```

그리고 로딩상태를 확인 후 화면 캡쳐를 진행하는 함수를 작성해 준다. 또한 아래 함수는 외부에서 접근할 수 있도록 `public` 으로 설정해야한다.

```
public void CaptureFunc()
{
    if (!_isLoading)
    {
    	_isLoading = true;
    	PhotoCapture.CreateAsync(OnPhotoCaptureCreated);
    }
}
```

`OnPhotoCaptureCreated` 함수에서는 캡쳐한 이미지에 대한 정보를 설정한다.

```
void OnPhotoCaptureCreated(PhotoCapture captureObject)
{
    photoCaptureObject = captureObject;

    CameraParameters c = new CameraParameters();
    c.hologramOpacity = 0.0f;
    c.cameraResolutionWidth = cameraResolution.width;
    c.cameraResolutionHeight = cameraResolution.height;
    c.pixelFormat = CapturePixelFormat.BGRA32;

    captureObject.StartPhotoModeAsync(c, OnPhotoModeStarted);
}
```

`OnPhotoModeStarted` 함수는 다음과 같이 작성한다.

```
private void OnPhotoModeStarted(PhotoCapture.PhotoCaptureResult result)
{
    if (result.success)
    {
	    photoCaptureObject.TakePhotoAsync(OnCapturedPhotoToMemory);
    }
    else
    {
    	Debug.LogError("Unable to start photo mode!");
    }
}
```

`OnCapturePhotoToMemory` 함수에는 캡쳐한 이미지 프레임의 처리 과정을 작성한다.

```
void OnCapturedPhotoToMemory(PhotoCapture.PhotoCaptureResult result, PhotoCaptureFrame photoCaptureFrame)
{
	// 이미지가 너무 넓어 중간 부분만 Crop 하여 사용
	// targetTexture 에 원본 이미지를 저장후 croppedTexture 에 crop 한 이미지를 저장
    int cropsize_width = cameraResolution.width / 2;
    int cropsize_height = cameraResolution.height / 2;
    Texture2D croppedTexture = new Texture2D(cropsize_width, cropsize_height);
    Texture2D targetTexture = new Texture2D(cameraResolution.width, cameraResolution.height);
    photoCaptureFrame.UploadImageDataToTexture(targetTexture);
    int offset_x = (cameraResolution.width - cropsize_width) / 2;
    int offset_y = (cameraResolution.height - cropsize_height) / 2;
    croppedTexture.SetPixels(targetTexture.GetPixels(offset_x, offset_y, cropsize_width, cropsize_height));
    croppedTexture.Apply();

	// 이미지를 base64 로 변환
	var base64Decoded = Texture2DToBase64(croppedTexture);
	
	// 서버에 base64 이미지를 전달
	StartCoroutine(SendPost(url, base64Decoded));
}
```

 ## Send Image to Server

입력받은 url 로 이미지를 전달하는 함수를 작성한다. 아래 함수는 `OnCapturedPhotoToMemory` 함수에서 `StartCoroutine` 을 통해 비동기적으로 실행될 것이다.

```
IEnumerator SendPost(string url, string base64, int w, int h)
{
    UnityWebRequest request;

    WWWForm form = new WWWForm();
    form.AddField("img", base64);
    form.AddField("name", "SOS");

    using (request = UnityWebRequest.Post(url, form))
    {
	    yield return request.SendWebRequest();

        if (request.isNetworkError)
        {
	        Debug.Log(request.error);
        }
        else
        {
        	// (Optional) 서버에서 전달받은 결과 text 를 textButton에 출력
            textButton.GetComponentInChildren<TextButtonHandler>().Show(request.downloadHandler.text);
            if (request.responseCode != 200)
	            Debug.Log(request.responseCode);
            }
    }
    // photoCapture 를 사용후 비활성화
    photoCaptureObject.StopPhotoModeAsync(OnStoppedPhotoMode);
}
```

## Add function to object

먼저 아무 기능도 없는 GameObject (그림에서는 `General`) 을 하나 생성한 뒤 Add Component 를 통해 위에서 작성한 Script 를 추가한다.

![img](/assets/img/Hololens2_start_project/05.png)

다음으로 버튼의 Interactable 컴포넌트의 Events > OnClick() 에 가서, Object 에는 `General` 을 드래그 하여 옮겨주고, 함수는 `Capture.CaptureFunc()` 으로 선택해준다. 

![img](/assets/img/Hololens2_start_project/03.png)

마지막으로 `Ctrl + P` 를 눌러 게임모드, 또는 build 하여 버튼을 눌렀을 때, 함수가 동작하는지 확인해 보면 된다.

## (Optional) Toast Message

장치에서는 Release 모드로 실행을 해야하다 보니 Debug 를 할 마땅한 방법을 찾지 못해 Stackoverflow 에 작성된 Toast 메세지를 사용하여 중간중간 결과를 확인하였다. 필요한 경우 [이 곳](https://stackoverflow.com/a/41042879)에서 해당 코드 및 사용법을 확인할 수 있다.

