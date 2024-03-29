# CAPTUS ANDROID SDK
![version](https://img.shields.io/badge/version-v1.0.7-blue)

The Captus SDK is a set of screens to capture the front and back images of ID documents. It also allows the user to manually verify that the documents are clean and clear. This SDK is useful for IDs that cannot be processed on the mobile and needs server-side processing. 

**You can find the latest version and release history [Here](CHANGELOG.md)**

# Table Of Content

- [Prerequisite](#prerequisite)
- [Android SDK Requirements](#android-sdk-requirements)
- [Download](#download)
  - [Using maven repository](#using-maven-repository)
- [Setup](#setup)
  - [Permissions](#permissions)
- [Quick Start](#quick-start)
  - [Initiating the Captus scanner](#initiating-the-captus-scanner)
  - [Handling the result](#handling-the-result)
- [Captus Result](#captus-result)
- [Captus Error Codes](#captus-error-codes)
- [Captus Parameters](#captus-parameters)
- [Help](#help)

## Prerequisite

***NOTE : Encryption of CAPTUS SDK Result is under development***

You will need a valid license to use the Captus SDK, which can be obtained by contacting `support@frslabs.com` . 

Once you have the license , follow the below instructions for a successful integration of Captus SDK onto your Android Application.

## Android SDK Requirements

**Minimum SDK Version** -  **21** or higher

## Download

#### Using maven repository

Add the following code to your `project` level `build.gradle` file

```groovy
allprojects { 
    repositories { 
        maven { 
            // Maven Url and Credentials for Captus SDK. 
            url "https://captus-android.repo.frslabs.space/"                  
            credentials { 
                   username 'repo-username' 
                   password 'repo-password' 
            }
       }
    }
}
```

After that, add the following code to your `app` level `build.gradle` file

```groovy
// ...

defaultConfig { 

    // ...
    
    ndk { 
        abiFilters "armeabi-v7a", "arm64-v8a", "x86", "x86_64" 
    } 

    vectorDrawables.useSupportLibrary true 
}

// ...
```

And then, add the dependencies
```groovy

// ...

dependencies {
    /* Dependencies for Captus SDK */ 
    implementation 'com.android.support:design:<version above 23.4.0>'      
    implementation 'com.android.support.constraint:constraint-layout:<version above 1.1.3>'
   
    implementation 'com.frslabs.android.sdk:captus:1.0.7' 
}
```

## Setup

#### Permissions

Captus requires the camera permission to work properly

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="your.package.name" >

    <!-- Required by Captus -->
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <application>
      ...
    </application>

</manifest>
```

## Quick Start

#### Initiating the Captus scanner

Initialize the `Captus` instance with the appropriate configurations to invoke the Captus Sdk

```java
public class MainActivity extends AppCompatActivity implements CaptusResultCallback {

    // ...

    /* Enter the Captus license key here */
    private String CAPTUS_LICENSE_KEY = "ENTER_YOUR_LICENSE_KEY_HERE";

    /* (OPTIONAL)  Enter the Captus api credentials here */
    private String CAPTUS_API_BASE_URL = "ENTER_BASE_URL_HERE"
            , CAPTUS_API_REFERENCE_ID = "ENTER_REF_ID_HERE"
            , CAPTUS_API_CRED1 = "ENTER_API_CRED1_HERE"
            , CAPTUS_API_CRED2 = "ENTER_API_CRED2_HERE";
    
    /* (OPTIONAL)  Enter the Captus encryption key here */
    private String CAPTUS_ENCRYPTION_KEY = "ENTER_ENCRYPTION_KEY_HERE";
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button callSdk = findViewById(R.id.call_sdk);
        callSdk.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                /* Invoke the Captus Sdk */
                callCaptusSdk();
            }
        });
    }

    public void callCaptusSdk() {

        //Initialize the Captus Sdk Config object with the appropriate configurations
        CaptusConfig captusConfig = new CaptusConfig.Builder()
                .setLicenseKey(CAPTUS_LICENSE_KEY)
                .setDocumentSide(CaptusUtility.Side.TWO)
                .setCropboxOrientation(CaptusUtility.CropboxOrientation.HORIZONTAL)
                .setCaptusSDK(true)
                //.setEncryptionKey(CAPTUS_ENCRYPTION_KEY) // Optional
                .setApiCredentials(new CaptusApiCredentials(CAPTUS_API_BASE_URL
                        ,CAPTUS_API_REFERENCE_ID
                        ,CAPTUS_API_CRED1
                        ,CAPTUS_API_CRED2)) // Optional
                .build();
               
        //Call the Captus Sdk 
        Captus captus = new Captus(captusConfig);
        captus.start(this, this);
        
    }

    // ...

}
```


#### Handling the result

Your activity must implement `CaptusResultCallback` to receive the result.

```java
    // ...

    @Override
    public void onCaptusSuccess(CaptusResult captusResult) {
        if (captusResult != null) {
            String[] imagePaths = captusResult.getImagePaths();
            String[] imageReferenceIds = captusResult.getImageReferenceIds();
            
            if(captusResult.getAmlCheckStatus().equals(Utility.AML_CHECK_PASS)){
                //Handle logic on AML Check Pass
            }else{
                //Handle logic on AML Check Fail
            }
            
        }
    }

    @Override
    public void onCaptusFailure(int errorCode) {
        /* Handle the Captus Sdk failure result here */
        Log.d(TAG, "onCaptusFailure: Error: " + errorCode);
    }

    // ...
```
## Captus Result

You can use the following methods in the `CaptusResult` instance to parse the result:

| Return Type | Method                 | Usage                                      |
| ----------- | ---------------------- | ------------------------------------------ |
| String[]    | *getImagePaths()*        | Array of JPEG File Path of captured images |
| String[]    | *getImageReferenceIds()* | Array of reference ID corresponding to each image |
| String    | *getAmlCheckStatus()* | Result of the AML Check. <br><br> Values are <br> **Utility.AML_CHECK_PASS** <br> **Utility.AML_CHECK_FAIL** |

## Captus Error Codes

Following error codes will be returned on the `onCaptusFailure` method of the callback

| CODE | DESCRIPTION                  |
| ---- | ---------------------------- |
| 803  | Camera permission denied    |
| 804  | Capture interrupted            |
| 805  | Captus SDK License has expired             |
| 806  | Captus SDK License is invalid             |
| 808  | Transaction failed / Unable to ping             |
| 901  | Network error           |
| 902  | Image upload failed             |
| 1001  | Error parsing result             |

## Captus Parameters

- `setLicenseKey(String captusLicenseKey)`   ***(Required)***
  
  Accepts the Captus SDK licence key as a `String`

  
- `setDocumentSide(Utility.Side documentSide)`  ***(Optional)*** *(Defaults to **CaptusUtility.Side.TWO**)*
  
  Sets the number of document sides to be captured
  
  | Value                   | Effect                                               |
  | ----------------------- | ---------------------------------------------------- |
  | CaptusUtility.Side.ONE        | Scans only the Front (Primary) side of the document  |
  | CaptusUtility.Side.TWO        | Scans both Front and back side of the document       |

- `setEncryptionKey(String captusEncryptionKey)`   ***(Optional)***
  
  Sets the Captus SDK encrpytion key as a `String` . If set, the output images will be encrypted.
  Obtain the appropriate encryption key through a REST API call , for details about the REST API, contact  `support@frslabs.com`
  
- `setCropboxOrientation(CaptusUtility.CropboxOrientation cropboxOrientation)` ***(Optional)*** *(Defaults to **CaptusUtility.CropboxOrientation.HORIZONTAL**)*

  Sets the Captus SDK image capture orientation

  | Value                   | Effect                                               |
  | ----------------------- | ---------------------------------------------------- |
  | CaptusUtility.CropboxOrientation.HORIZONTAL        | Captures the image with a landscape crop  |
  | CaptusUtility.CropboxOrientation.VERTICAL        | Captures the image with a portrait crop       |

- `setCaptusSDK(boolean value)` ***(Required)***

  Sets the Captus SDK capture mode.
  
  | Value                   | Effect                                               |
  | ----------------------- | ---------------------------------------------------- |
  | true (Recommended)       | Captures the image(s) and returns the result |
  | false       | Special case - Captures the image , OCR's the arabic text and returns the result along with the AML check status |
  
- `setApiCredentials(CaptusApiCredentials captusApiCredentials)`   ***(Optional)***
  
  Sets the Captus SDK apiCredentials . Obtain the appropriate api credentials through a REST API call , for details about     the REST API, contact `support@frslabs.com`
  
 
## Help
For any queries/feedback , contact us at `support@frslabs.com` 
