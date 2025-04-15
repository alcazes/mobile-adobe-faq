# mobile-adobe-faq

## Adobe Launch Mobile Set up

Install the following extension:
- Adobe Experience Platform Edge Network (Will allow to send data to Adobe DataStream)
- AEP Assurance (Will allow debuging for Adobe Integration using the assurance tool. You will need to use deep links if app is not live)
- Consent (use to know what can and cannot be tracked based on customer selection)
- Identity (required for some product like analytics)
- Mobile Core (installed by default)
- Profile (installed b default)

By default the lifecycle events will not be present for AEP calls so we will need to create a specific rule to tailor for this:
- Events:
  - Mobile Core - Foreground
  - Mobile Core - Background
- Actions:
  - Extension: Adobe Experience Platform Edge Network
  - Action type: Forward event to Edge Network

Create a build and add all extensions and rules created.

## Installation instructions

When to do an application rebuild and when not to:
- Rebuild when adding new extensions as the corresponding SDK will be required
- No need to rebuild if configuration is changed for extension or new rule/data elements are created. These changes will be part in the JSON file that will be downloaded when the application start.
  - It will however require an adobe launch rebuild and release to correct environment as needed

Once the build is ready in an environment then you can do the following to get the details on how to deploy correct SDKs:
- In Adobe Launch mobile property go to `Environments`
- For the environment targeted click on the far right box icon
- Both the Android and IOS installation steps will be listed

Some documentation:
https://developer.adobe.com/client-sdks/home/getting-started/get-the-sdk/
https://github.com/adobe/aepsdk-edgebridge-android/blob/main/Documentation/tutorials/EdgeBridgeTutorialAppFinal/app/src/main/java/com/adobe/marketing/mobile/tutorial/MainApp.java

### Update Android dependencies

Add the following to your build.gradle
```gradle
//Adobe SDKs
implementation platform('com.adobe.marketing.mobile:sdk-bom:3.+')
implementation 'com.adobe.marketing.mobile:edge'
implementation 'com.adobe.marketing.mobile:edgeconsent'
implementation 'com.adobe.marketing.mobile:assurance'
implementation 'com.adobe.marketing.mobile:edgeidentity'
implementation 'com.adobe.marketing.mobile:core'
implementation 'com.adobe.marketing.mobile:identity'
implementation 'com.adobe.marketing.mobile:lifecycle'
implementation 'com.adobe.marketing.mobile:signal'
implementation 'com.adobe.marketing.mobile:userprofile'
implementation 'com.adobe.marketing.mobile:edgebridge' //required for analytics migration
```

Notice that you should lock to a specific bom version for security reasons most likely, yo ucan check what is latest of the bom and what is in the bom using:
https://central.sonatype.com/artifact/com.adobe.marketing.mobile/sdk-bom

Notice that we added `implementation 'com.adobe.marketing.mobile:edgebridge'` which is not listed in the adobe launch extentions but is needed in our case as we will migrate legacy adobe analytics integration to AEP. This extension sole purpose if to transform the payload from MobileCore.tracState and MobileCore.trackAction to AEP XDM payload type. This approach also allows us to not do any changes to Adobe Analytics processing rules.

### Update Android manifest with some access and details for deep link

We need to add the following permssions:
`<uses-permission android:name="android.permission.INTERNET" />`
`<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />`

For deeplinking for assurance puporses we added
```xml
<intent-filter>
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.LAUNCHER" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data
    android:host="test.my.app"
    android:scheme="http" />
  </intent-filter>
```
We will specify `http://test.my.app` in the adobe assurance session interface when we want to debug

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:name="AdobeSDKTest"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.AdobeSDKTest"
        tools:targetApi="31">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>

            <intent-filter>
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.LAUNCHER" />
                <category android:name="android.intent.category.BROWSABLE" />

                <data
                    android:host="test.my.app"
                    android:scheme="http" />
            </intent-filter>

            <meta-data
                android:name="android.app.lib_name"
                android:value="" />
        </activity>
    </application>

</manifest>
```
### Android Initiliazation code

You should have had some initialization code provided to you. However the code provided is not the best practice one anymore. A new API can be used and is simpler.

```java
package com.example.adobesdktest;

import android.app.Application;
import com.adobe.marketing.mobile.LoggingMode;
import com.adobe.marketing.mobile.MobileCore;

public class AdobeSDKTest extends Application {
    private static final  String LOG_TAG = "Adobe SDK Test";

    @Override
    public void onCreate() {
        super.onCreate();
        MobileCore.setLogLevel(LoggingMode.VERBOSE);
        MobileCore.initialize(this, "application_ID_provided_from_adobe_launch_interface");
    }
}
```

### Lifecycle Event, trackState and trackAction

As we do not want to modify Adobe Analytics integration, we will continue to use `MobileCore` APIs to:
- track lifecycle events
- track state change aka page views
- trac actions anything that is not a page view change

```java
package com.example.adobesdktest;

import androidx.appcompat.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.TextView;
import android.widget.Toast;
import com.adobe.marketing.mobile.MobileCore;
import com.google.android.material.button.MaterialButton;
import java.util.HashMap;
import java.util.Map;

public class MainActivity extends AppCompatActivity {

    private static final  String LOG_TAG = "ALEXIS";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //Track State START
        Map<String, String> pagePayload = new HashMap<>();
        pagePayload.put("pageType", "test");
        MobileCore.trackState("Login screen", pagePayload);
        //Track State END

        TextView username = (TextView)  findViewById(R.id.username);
        TextView password = (TextView)  findViewById(R.id.password);

        MaterialButton loginbtn = (MaterialButton) findViewById(R.id.loginbtn);

        loginbtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Log.d(LOG_TAG, "Login Button Clicked");
                if (username.getText().toString().equals("admin") &&
                    password.getText().toString().equals("admin")) {
                    Toast.makeText(MainActivity.this, "LOGIN SUCCESSFUL", Toast.LENGTH_SHORT).show();

                    //Track action start
                    Map<String, String> loginSuccess = new HashMap<>();
                    loginSuccess.put("login", "success");
                    MobileCore.trackAction("Track Action Login Success", loginSuccess);
                    //Track action end
                } else {
                    Toast.makeText(MainActivity.this, "LOGIN FAILED", Toast.LENGTH_SHORT).show();
                    //Track action start
                    Map<String, String> loginFailed = new HashMap<>();
                    loginFailed.put("login", "failure");
                    MobileCore.trackAction("Track Action Login Failure", loginFailed);
                    //Track action end
                }
            }
        });
    }

    @Override
    public void onResume() {
        super.onResume();
        //Adobe: Start mobile lifecycle data
        MobileCore.setApplication(getApplication());
        MobileCore.lifecycleStart(null);
    }

    @Override
    public void onPause() {
        super.onPause();
        //Adobe: Pause mobile lifecycle data
        MobileCore.lifecyclePause();
    }
}
```

Ok what are we going to see in the logs:
```
2025-04-15 14:41:05.714 11229-11269 AdobeExperienceSDK      com.example.adobesdktest             D  Edge/EdgeHitProcessor - Sending network request with id (0158f80f-a73c-41a7-929c-431eaa7143e9) to URL 'https://rbs.data.adobedc.net/ee/v1/interact?configId=ecdd7b20-07bb-4542-a87b-ba07c2947b70&requestId=0158f80f-a73c-41a7-929c-431eaa7143e9' with body:
{
  "xdm": {
    "implementationDetails": {
      "environment": "app",
      "name": "https:\/\/ns.adobe.com\/experience\/mobilesdk\/android",
      "version": "3.3.1+3.0.2"
    },
    "identityMap": {
      "ECID": [
        {
          "id": "00710055241504043176719972112582724955",
          "authenticatedState": "ambiguous",
          "primary": false
        }
      ]
    }
  },
  "meta": {
    "state": {
      "entries": [
        {
          "maxAge": 34128000,
          "value": "CiYwMDcxMDA1NTI0MTUwNDA0MzE3NjcxOTk3MjExMjU4MjcyNDk1NVIRCPTm2cXjMhgBKgRJUkwxMAPwAfTm2cXjMg==",
          "key": "kndctr_C50417FE52CB33480A490D4C_AdobeOrg_identity"
        }
      ]
    },
    "konductorConfig": {
      "streaming": {
        "lineFeed": "\n",
        "enabled": true,
        "recordSeparator": "\u0000"
      }
    }
  },
  "events": [
    {
      "xdm": {
        "eventType": "analytics.track",
        "_id": "a465ac85-d497-4b2a-846f-cc4c7b64fd3d",
        "timestamp": "2025-04-15T13:41:04.527Z"
      },
      "data": {
        "mobile": {
          "dude": ""
        },
        "__adobe": {
          "analytics": {
            "pageName": "Login screen",
            "cp": "foreground",
            "contextData": {
              "pageType": "test",
              "a.AppID": "Adobe SDK test 1.0 (1)"
            }
          }
        }
      }
    }
  ]
}
```



