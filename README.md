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
```
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
