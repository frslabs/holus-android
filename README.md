# HOLUS SDK
![version](https://img.shields.io/badge/version-v1.0.0-blue)

The Holus SDK comes with a video recording screen to capture the identity document at different angles to check for the presence or absence of holograms in the ID. Available for limited countries only.

# Table Of Content

- [Prerequisite](#prerequisite)
- [Android SDK Requirements](#android-sdk-requirements)
- [Download](#download)
  - [Using maven repository](#using-maven-repository)
- [Setup](#setup)
  - [Permissions](#permissions)
- [Quick Start](#quick-start)
  - [Initiating the Holus SDK](#initiating-the-holus-sdk)
  - [Handling the result](#handling-the-result)
- [Holus Result](#holus-result)
- [Holus Error Codes](#holus-error-codes)
- [Holus Parameters](#holus-parameters)
- [Help](#help)

## Prerequisite

You will need a valid license to use the Holus SDK, which can be obtained by contacting `support@frslabs.com` . 

Depending on the license - offline or online - you have opted for, the ping functionality to billing servers will be disabled or enabled. For instance, if you have opted for the offline SDK model, then there will be no server ping needed to our billing server to bill you. However, if you have chosen a transaction based pricing, then after each transaction, a ping request will be made to our billing server. This cannot be overrided by the App. A point to note is that if the ping transaction fails for any reason, the whole transaction will be void without any results from the SDK.

Once you have the license , follow the below instructions for a successful integration of Holus SDK onto your Android Application.

## Android SDK Requirements

**Minimum SDK Version** -  **21** or higher

## Download

#### Using maven repository

Add the following code to your `project` level `build.gradle` file

```groovy
allprojects { 
    repositories { 
        maven { 
            // Maven Url and Credentials for Holus SDK. 
            url "https://holus-android.repo.frslabs.space/"                  
            credentials { 
                username '<ENTER-USERNAME-HERE>'
                password '<ENTER-PASSWORD-HERE>' 
            }
        }
       
        // (OPTIONAL) Maven credentials for the Torus SDK
        // Include below code only for transaction based billing
        maven {
            url "https://torus-android.repo.frslabs.space/"
            credentials {
                username '<ENTER-USERNAME-HERE>'
                password '<ENTER-PASSWORD-HERE>' 
            }
        }
        
    }
}
```

After that, add the following code to your `app` level `build.gradle` file

```groovy
// ...

    android {
    
        // ...
        
        defaultConfig { 
        
            // ...
        
            vectorDrawables.useSupportLibrary true 
        }
        
        aaptOptions {
            noCompress "tflite"
        }

        // ...
        
    }

// ...
```

And then, add the dependencies
```groovy

// ...

dependencies {
    /* Android SDK Dependencies for Holus */ 
    implementation 'com.android.support:design:<version above 23.4.0>'      
    implementation 'com.android.support.constraint:constraint-layout:<version above 1.1.3>'
   
    // REQUIRED - Holus Core Dependency
    implementation 'com.frslabs.android.sdk:holus:1.0.0' 
    
    // REQUIRED - Holus Additional Depedencies 
    implementation 'org.tensorflow:tensorflow-lite:1.14.0'
    implementation 'com.google.android.gms:play-services-vision:17.0.2'
    
    // OPTIONAL - Holus billing dependencies
    // Required only if transaction based billing is enabled
    implementation('com.frslabs.android.sdk:torus:0.0.9')
    implementation('com.squareup.retrofit2:converter-gson:2.3.0')
    implementation('com.squareup.retrofit2:retrofit:2.3.0')
    implementation('com.google.code.gson:gson:2.8.5')
}
```

## Setup

#### Permissions

Holus requires the camera permission to work properly

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="your.package.name" >

    <!-- Required by Holus -->
    <uses-permission android:name="android.permission.CAMERA" />

    <!-- Optional - Required if transaction based billing is enabled -->
    <uses-permission android:name="android.permission.INTERNET" />  
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
  
    <application>
      ...
    </application>

</manifest>
```

#### Proguard rules

Include below proguard rules only if transaction based billing is enabled

```java
-keep class retrofit.** { *; }

-keepclasseswithmembers class * {
   @retrofit2.http.* <methods>;
}
-keepclasseswithmembers interface * {
   @retrofit2.* <methods>;
}
```


## Quick Start

#### Initiating the Holus SDK

Initialize the `Holus` instance with the appropriate configurations to invoke the Holus Sdk

```java
public class MainActivity extends AppCompatActivity implements HolusResultCallback {

    // ...

    /* Enter the Holus license key here */
    private String HOLUS_LICENSE_KEY = "<ENTER_YOUR_LICENSE_KEY_HERE>";
    
    /* Enter the Holus encryption key here */
    private String HOLUS_ENCRYPTION_KEY = "<ENTER_ENCRYPTION_KEY_HERE>";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button callSdk = findViewById(R.id.call_sdk);
        callSdk.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                /* Invoke the Holus Sdk */
                callHolusSdk();
            }
        });
    }

    public void callHolusSdk() {

        //Initialize the Holus Sdk Config object with the appropriate configurations
        HolusConfig holusConfig = new HolusConfig.Builder()
                            .setLicenseKey(HOLUS_LICENSE_KEY)
                            .setEncryptionKey(HOLUS_ENCRYPTION_KEY) //Optional
                            .build();

        //Call the Holus Sdk to start scanning
        Holus holus = new Holus(holusConfig);
        holus.start(this, this);
        
    }

    // ...

}
```


#### Handling the result

Your activity must implement `HolusResultCallback` to receive the result.

```java
    // ...

    @Override
    public void onHolusSuccess(HolusResult holusResult) {
    
        if (holusResult != null) {
            //Make sure to delete/move video file stored in this path
            String holusVideoPath = holusResult.getHolusVideoPath();

            if(holusResult.isDocumentSpoofed()){
                // Document was detected to be spoofed/fake
            }else{
                // Document was detected to be original
            }
        }
    }

    @Override
    public void onHolusFailure(int errorCode) {
        /* Handle the Holus Sdk failure result here */
        Log.d(TAG, "onHolusFailure: Error: " + errorCode);
    }

    // ...
```

## Holus Result

You can use the following methods in the `HolusResult` instance to parse the success result:

| Return Type | Method                 | Usage                                      |
| ----------- | ---------------------- | ------------------------------------------ |
| String      | *getHolusVideoPath()*   | MP4 File Path of recorded video               |
| boolean     | *isDocumentSpoofed()* | Result of document spoof check (true is doucment is spoofed)|
| String      | *getHolusSessionReferenceId()* |  Reference Id corredponsing to the video |

## Holus Error Codes

Following error codes will be returned on the `onHolusFailure` method of the callback

| CODE | DESCRIPTION                  |
| ---- | ---------------------------- |
| 803  | Camera permission denied    |
| 804  | Scan was interrupted            |
| 805  | Holus SDK License got expired             |
| 806  | Holus SDK License was invalid             |

## Holus Parameters

- `setLicenseKey(String holusLicenseKey)`   ***(Required)***
  
  Accepts the Holus SDK licence key as a `String`
  
- `setEncryptionKey(String holusEncryptionKey)`   ***(Optional)***
  
  Accepts the Holus SDK encrpytion key as a `String` . If set, the output video will be encrypted.
  Obtain the appropriate encryption key through a REST API call , for details about the REST API, contact          `support@frslabs.com`

## Help
For any queries/feedback , contact us at `support@frslabs.com` 
