# Signal Android
This guide is written by using Signal-Android branch Master version 4.53.6

## Requirement
* Android Studio 3.5
* JRE 1.8.0 
* JVM OpenJDK 64
* SDK 28
* NDK (Optional, only if you want to make custom package)

## How to
1. Convert your server ssl cert to pkcs#12 
```
openssl pkcs12 -export -out keystore.pkcs12 -in fullchain.pem -inkey privkey.pem
```

2. Use 'Keystore Explorer’ (MacOS software, you can try using another software), edit `whisper.store` files (the password is "whisper" without quote), insert your pk12 certificate there. If you use AWS CDN Cloudfront, you also need to put Cloudfront's certificate there

3. Open the project in Android Studio (Open, not Import).

4. Update URL with own server in `app/build.gradle` (be sure to use https and don't include trailing slash)

3. update `UNIDENTIFIED_SENDER_TRUST_ROOT` (value is Public Key from when creating UnidentifiedDelivery of server’s config.yml)

4. Comment out `distributionSha256Sum` on `gradle/wrapper/gradle-wrapper.properties`

5. Download `google-service.json` from Firebase, put it inside `app/`.

6. Update `app/src/main/res/values/firebase_messaging.xml` according to value from `google-service.json`

7. Update `ATTACHMENT_DOWNLOAD_PATH` and `ATTACHMENT_UPLOAD_PATH` in `libsignal/service/src/main/java/org/whispersystems/signalservice/internal/push/PushServiceSocket.java` by deleting ‘attachments/‘ so attachment will be uploaded in root ( / ). 

8. Sync your project then build.

## Custom Server
Change `app/build.gradle` to use your server. Always use https and without trailing slash on the url.
```
// app/build.gradle

        buildConfigField "long", "BUILD_TIMESTAMP", getLastCommitTimestamp() + "L"
        buildConfigField "String", "SIGNAL_URL", "\"https://domain.com\""
        buildConfigField "String", "STORAGE_URL", "\"https://domain.com\""
        buildConfigField "String", "SIGNAL_CDN_URL", "\"https://your-own.cloudfrontnet\""
        buildConfigField "String", "SIGNAL_CONTACT_DISCOVERY_URL", "\"https://domain.com\""
        buildConfigField "String", "SIGNAL_SERVICE_STATUS_URL", "\"https://domain.com\""
        buildConfigField "String", "SIGNAL_KEY_BACKUP_URL", "\"https://domain.com\""
        buildConfigField "String", "CONTENT_PROXY_HOST", "\"https://domain.com\""
        buildConfigField "int", "CONTENT_PROXY_PORT", "443"
        buildConfigField "String", "USER_AGENT", "\"OWA\""
        buildConfigField "boolean", "DEV_BUILD", "false"
        buildConfigField "String", "MRENCLAVE", "\"cd6cfc342937b23b1bdd3bbf9721aa5615ac9ff50a75c5527d441cd3276826c9\""
        buildConfigField "String", "KEY_BACKUP_ENCLAVE_NAME", "\"f2e2a5004794a6c1bac5c4949eadbc243dd02e02d1a93f10fe24584fb70815d8\""
        buildConfigField "String", "KEY_BACKUP_MRENCLAVE", "\"f51f435802ada769e67aaf5744372bb7e7d519eecf996d335eb5b46b872b5789\""
        buildConfigField "String", "UNIDENTIFIED_SENDER_TRUST_ROOT", "\"CHANGE-TO-YOUR-UNIDENTIFIED-DELIVERY-PUBLIC-KEY\""

```

## Custom Package Name
To create your own Signal-based Chat application, you need to rename the package. This is to prevent your app conflicting with signal on Play Store and on user's phone.

1. Open Signal-Android in Android Studio.

Signal-Android package name is `org.thoughtcrime.securesms` and in the project it's shown as a structure like this

`app/src/main/java/org/thoughtcrime/securesms/`

It is recommended that your desired package name also consist of three part, for example `com.company.chatname`.

2. Right click on `securesms` directory, choose `Refactor`, then choose `Rename`. When prompted, choose `Rename Package`. Change it to your desired `chatname`. It is optional to choose `Search in comments and strings` and `Search for text occurance`, you can leave it blank.

3. After the search process is done, a window will appear on the bottom. Choose `Do Refactor`.

4. It will take some time, after it is done, repeat step 2-3 to `thoughtcrime` and `org`.

5. There's a chance some import could be left unchanged, to make sure, find `org.thoughtcrime.securesms`. If you found some, change it manually.

If you have done those step, you can already build your own app and install it to android. But the camera will not work because you haven't change the package name in native code.

6. Change the package name in native code, find `org_thoughtcrime_securesms` and replace with your package name, **remember to use underscore (_) like how it is before**. It is located on:

* `app/jni/Android.mk`
* `app/jni/utils/org_thoughtcrime_securesms_util_FileUtils.cpp`
* `app/jni/utils/org_thoughtcrime_securesms_util_FileUtils.h`


The last two files (.cpp and .h) also need to be renamed to follow your new package name, for example:

* `app/jni/utils/com_company_chatname_util_FileUtils.cpp`
* `app/jni/utils/com_company_chatname_util_FileUtils.h`


7. Install NDK, and run this command in your directory

```
ndk-build
```

If the command is not recognized, it means your NDK installation has not been added to Environment Variable, then run it with absolute path.

8. It will generate 4 `libnative-utils.so` inside `app/jni`. Use it to replace the originals that located in here:

* `app/src/main/jniLibs/arm64-v8a/libnative-utils.so`
* `app/src/main/jniLibs/armeabi-v7a/libnative-utils.so`
* `app/src/main/jniLibs/x86/libnative-utils.so`
* `app/src/main/jniLibs/x86_64/libnative-utils.so`

9. Update the package name that formated as `org/thoughtcrime/securesms` and replace with your package name, **remember to use slash (/) like how it is before**. It is located on: 

* `app/lint-baseline.xml`
* `app/lint.xml`

10. Remember to update the application name in :

`app/src/main/res/values/strings.xml`

## FAQ
Q: Why did I need to change the Attachment Path?

A: For now, I have no idea why you can't upload to a path except root, I've tried modifying every permission in AWS but to no avail. 
