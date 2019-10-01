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
- [Holus Parameters](#holus-parameters)
- [Help](#help)

## Prerequisite

You will need a valid license to use the Holus SDK, which can be obtained by contacting `support@frslabs.com` . 

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

    vectorDrawables.useSupportLibrary true 
}

// ...
```

And then, add the dependencies
```groovy

// ...

dependencies {
    /* Dependencies for Holus SDK */ 
    implementation 'com.android.support:design:<version above 23.4.0>'      
    implementation 'com.android.support.constraint:constraint-layout:<version above 1.1.3>'
   
    implementation 'com.frslabs.android.sdk:holus:1.0.0' 
}
```

## Setup

#### Permissions

Holus requires the camera permission to work properly

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="your.package.name" >

    <!-- Required by Holus -->
    <uses-permission android:name="android.permission.CAMERA" />

    <application>
      ...
    </application>

</manifest>
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

## Holus Parameters

- `setLicenseKey(String holusLicenseKey)`   ***(Required)***
  
  Accepts the Holus SDK licence key as a `String`
  
- `setEncryptionKey(String holusEncryptionKey)`   ***(Optional)***
  
  Accepts the Holus SDK encrpytion key as a `String` . If set, the output video will be encrypted.
  Obtain the appropriate encryption key through a REST API call , for details about the REST API, contact          `support@frslabs.com`

## Help
For any queries/feedback , contact us at `support@frslabs.com` 
