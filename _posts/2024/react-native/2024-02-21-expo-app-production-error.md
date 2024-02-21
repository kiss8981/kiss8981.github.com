---
title: "[Expo + React Native] 빌드하면서 프로덕션에서만 발생하던 문제들"
tag:
  - Expo
  - React Native
  - eas
  - ios
  - android
categories:
  - React Native
---

Expo 개발에서 가장 짜증 나는 부분 중 하나인 개발에서는 잘 작동하는 앱이 꼭 프로덕션 빌드를 하면 오류를 뱉는다.
아래 사진처럼 앱을 실행하면 바로 앱이 충돌함이 뜨면서 꺼지는 경우는 보통 이 부분을 확인해 보면 해결된다.

![]({{ 'assets/images/post/2024/expo-app-production-error/app-crashing.png' | relative_url }}){: width="200" height="200"}

### 첫번째 오류 (requireNativeComponent: "RNSScreen" was not found in the UIManager)

Expo Go 앱에서는 잘 실행되어도 빌드 후 배포 환경에서는 생기는 오류이다.
React Native에서 Screen 패키지가 없어서 발생하는 오류로 아래와 같이 설치해 주면 바로 해결이 가능하다.

```
npm expo install @react-navigation/native
npx expo install react-native-gesture-handler react-native-screens react-native-safe-area-context @react-native-community/masked-view
```

### 두번째 오류 (libreanimated.so)

Expo SDK 49 , 50 버전을 사용 시에 나타났던 오류이다.
`react-native-reanimated` 라이브러리에서 충돌이 일어나서 생기는 오류인 거 같다.
간단하게 `npm uninstall react-native-reanimated`라이브러리를 삭제하면 된다.
`npx expo prebuild`를 하면 AVD를 통하여 Logcat에서 로그를 확인할 수 있다.

```bash
08-29 02:13:46.129 15060 15060 F DEBUG   :       #00 pc /lib/arm64/libreanimated.so
08-29 02:13:46.129 15060 15060 F DEBUG   :       #01 pc /lib/arm64/libreanimated.so (reanimated::Scheduler::triggerUI()+176)
08-29 02:13:46.129 15060 15060 F DEBUG   :       #02 pc =/lib/arm64/libreanimated.so (facebook::jni::detail::MethodWrapper<void (reanimated::AndroidScheduler::*)(), &(reanimated::AndroidScheduler::triggerUI()), reanimated::AndroidScheduler, void>::dispatch(facebook::jni::alias_ref<facebook::jni::detail::JTypeFor<facebook::jni::HybridClass<reanimated::AndroidScheduler, facebook::jni::detail::BaseHybridClass>::JavaPart, facebook::jni::JObject, void>::_javaobject*>)+44)
08-29 02:13:46.129 15060 15060 F DEBUG   :       #03 pc /lib/arm64/libreanimated.so (facebook::jni::detail::FunctionWrapper<void (*)(facebook::jni::alias_ref<facebook::jni::detail::JTypeFor<facebook::jni::HybridClass<reanimated::AndroidScheduler, facebook::jni::detail::BaseHybridClass>::JavaPart, facebook::jni::JObject, void>::_javaobject*>), facebook::jni::detail::JTypeFor<facebook::jni::HybridClass<reanimated::AndroidScheduler, facebook::jni::detail::BaseHybridClass>::JavaPart, facebook::jni::JObject, void>::_javaobject*, void>::call(_JNIEnv*, _jobject*, void (*)(facebook::jni::alias_ref<facebook::jni::detail::JTypeFor<facebook::jni::HybridClass<reanimated::AndroidScheduler, facebook::jni::detail::BaseHybridClass>::JavaPart, facebook::jni::JObject, void>::_javaobject*>))+60)
08-29 02:13:46.129 15060 15060 F DEBUG   :       #04 pc /lib/arm64/libreanimated.so (facebook::jni::detail::MethodWrapper<void (reanimated::AndroidScheduler::*)(), &(reanimated::AndroidScheduler::triggerUI()), reanimated::AndroidScheduler, void>::call(_JNIEnv*, _jobject*)+36)
```

### 세번째 오류 (expo-updates)

Expo expo-update를 이용하여 기본적으로 OTA 업데이트를 지원해서 스토어 심사 없이 바로 업데이트가 가능한데, 이게 가끔 오류를 만든다.
우선 내가 경험한 것으로는 기존 배포된 앱이 있었는데 새롭게 리팩토링을 진행해서 expo@49 버전으로 업데이트하고 eas build를 사용하여 빌드하고 배포하였을 때 오류가 발생했다.
Expo Update에서 구버전 코드와 충돌이 일어나서 생기던 현상이었다. 이럴 때는 app.json / app.config.js 파일에서 아래와 같이 updates 기능을 사용하지 않고 배포하면 정상 작동한다.

```js
{
 expo: {
  updates: {
   enabled: false,
   },
  }
}
```

```bash
오류  06:42:43.946165+0900    tccd    Requestor: TCCDProcess: identifier=com.apple.nsurlsessiond, pid=117, auid=501, euid=501, binary_path=/usr/libexec/nsurlsessiond is not entitled to check access for accessor TCCDProcess: identifier=**, pid=51973, auid=501, euid=501, binary_path=/private/var/containers/Bundle/Application/29401AD7-72BE-4A20-AE9E-A7A65032B1FE/app.app/app
오류  06:42:43.972450+0900    SpringBoard Advisor: No handle found for currently focused PID: 51973; sceneIdentity: com.apple.frontboard.systemappservices::FBSceneManager:sceneID%3A**-default
오류  06:42:43.998714+0900    SpringBoard Ignoring update for invalidated scene: (FBSceneManager):sceneID:**-default
오류  06:42:44.013312+0900    symptomsd   COSMCtrl _foregroundAppActivity incoming bundle ** has nil supplied UUID, finds existing 8C005B27-DE58-3BE4-B1B6-FE884E357580
오류  06:44:18.491532+0900    tccd    Requestor: TCCDProcess: identifier=com.apple.nsurlsessiond, pid=117, auid=501, euid=501, binary_path=/usr/libexec/nsurlsessiond is not entitled to check access for accessor TCCDProcess: identifier=**, pid=51989, auid=501, euid=501, binary_path=/private/var/containers/Bundle/Application/29401AD7-72BE-4A20-AE9E-A7A65032B1FE/app.app/app
오류  06:44:18.516489+0900    SpringBoard Scene FBSceneManager/sceneID:**-default update failed: <NSError: 0x28b0f1230; domain: FBSceneErrorDomain; code: 1 ("operation-failed"); "Scene update failed."> {
    NSUnderlyingError = <NSError: 0x28b023c00; domain: BSServiceConnectionErrorDomain; code: 3 ("OperationFailed"); "XPC error received on message reply handler">;
}
오류  06:44:18.516567+0900    SpringBoard Scene FBSceneManager/sceneID:**-default update failed: <NSError: 0x28b0fbf90; domain: FBSceneErrorDomain; code: 1 ("operation-failed"); "Scene update failed."> {
    NSUnderlyingError = <NSError: 0x2830be280; domain: BSServiceConnectionErrorDomain; code: 3 ("OperationFailed"); "XPC error received on message reply handler">;
}
오류  06:44:18.517538+0900    SpringBoard Advisor: No handle found for currently focused PID: 51989; sceneIdentity: com.apple.frontboard.systemappservices::FBSceneManager:sceneID%3A**-default
오류  06:44:18.550419+0900    SpringBoard Ignoring update for invalidated scene: (FBSceneManager):sceneID:**-default
오류  06:44:18.563987+0900    symptomsd   COSMCtrl _foregroundAppActivity incoming bundle ** has nil supplied UUID, finds existing 8C005B27-DE58-3BE4-B1B6-FE884E357580
오류  06:44:18.565015+0900    symptomsd   COSMCtrl _foregroundAppActivity incoming bundle ** has nil supplied UUID, finds existing 8C005B27-DE58-3BE4-B1B6-FE884E357580
오류  06:44:36.792783+0900    tccd    Requestor: TCCDProcess: identifier=com.apple.nsurlsessiond, pid=117, auid=501, euid=501, binary_path=/usr/libexec/nsurlsessiond is not entitled to check access for accessor TCCDProcess: identifier=**, pid=51990, auid=501, euid=501, binary_path=/private/var/containers/Bundle/Application/29401AD7-72BE-4A20-AE9E-A7A65032B1FE/app.app/app
오류  06:44:36.814224+0900    SpringBoard Advisor: No handle found for currently focused PID: 51990; sceneIdentity: com.apple.frontboard.systemappservices::FBSceneManager:sceneID%3A**-default
오류  06:44:36.867409+0900    runningboardd   RBSStateCapture remove item called for untracked item 34-14990-2386815 (target:[app<**(A7E14B0A-11CC-4E38-A0DE-8FA575E5248D)>:51990](UIScene:com.apple.frontboard.systemappservices::FBSceneManager:sceneID%3A**-default))
오류  06:44:36.869945+0900    symptomsd   COSMCtrl _foregroundAppActivity incoming bundle ** has nil supplied UUID, finds existing 8C005B27-DE58-3BE4-B1B6-FE884E357580
오류  06:44:36.873975+0900    symptomsd   COSMCtrl _foregroundAppActivity incoming bundle ** has nil supplied UUID, finds existing 8C005B27-DE58-3BE4-B1B6-FE884E357580
오류  06:45:18.545274+0900    tccd    Requestor: TCCDProcess: identifier=com.apple.nsurlsessiond, pid=117, auid=501, euid=501, binary_path=/usr/libexec/nsurlsessiond is not entitled to check access for accessor TCCDProcess: identifier=**, pid=51991, auid=501, euid=501, binary_path=/private/var/containers/Bundle/Application/29401AD7-72BE-4A20-AE9E-A7A65032B1FE/app.app/app
오류  06:45:18.568855+0900    SpringBoard Advisor: No handle found for currently focused PID: 51991; sceneIdentity: com.apple.frontboard.systemappservices::FBSceneManager:sceneID%3A**-default
오류  06:45:18.601229+0900    SpringBoard Ignoring update for invalidated scene: (FBSceneManager):sceneID:**-default
오류  06:45:18.626454+0900    runningboardd   RBSStateCapture remove item called for untracked item 34-14990-2386838 (target:[app<**(A7E14B0A-11CC-4E38-A0DE-8FA575E5248D)>:51991](UIScene:com.apple.frontboard.systemappservices::FBSceneManager:sceneID%3A**-default))
오류  06:45:18.636034+0900    symptomsd   COSMCtrl _foregroundAppActivity incoming bundle ** has nil supplied UUID, finds existing 8C005B27-DE58-3BE4-B1B6-FE884E357580
오류  06:45:18.641839+0900    symptomsd   COSMCtrl _foregroundAppActivity incoming bundle ** has nil supplied UUID, finds existing 8C005B27-DE58-3BE4-B1B6-FE884E357580
오류  06:45:54.483172+0900    tccd    Requestor: TCCDProcess: identifier=com.apple.nsurlsessiond, pid=117, auid=501, euid=501, binary_path=/usr/libexec/nsurlsessiond is not entitled to check access for accessor TCCDProcess: identifier=**, pid=52003, auid=501, euid=501, binary_path=/private/var/containers/Bundle/Application/29401AD7-72BE-4A20-AE9E-A7A65032B1FE/app.app/app
오류  06:45:54.483191+0900    tccd    Requestor: TCCDProcess: identifier=com.apple.nsurlsessiond, pid=117, auid=501, euid=501, binary_path=/usr/libexec/nsurlsessiond is not entitled to check access for accessor TCCDProcess: identifier=**, pid=52003, auid=501, euid=501, binary_path=/private/var/containers/Bundle/Application/29401AD7-72BE-4A20-AE9E-A7A65032B1FE/app.app/app
오류  06:45:54.516021+0900    SpringBoard Advisor: No handle found for currently focused PID: 52003; sceneIdentity: com.apple.frontboard.systemappservices::FBSceneManager:sceneID%3A**-default
오류  06:45:54.516040+0900    SpringBoard Advisor: No handle found for currently focused PID: 52003; sceneIdentity: com.apple.frontboard.systemappservices::FBSceneManager:sceneID%3A**-default
오류  06:45:54.559575+0900    runningboardd   RBSStateCapture remove item called for untracked item 34-14990-2386948 (target:[app<**(A7E14B0A-11CC-4E38-A0DE-8FA575E5248D)>:52003](UIScene:com.apple.frontboard.systemappservices::FBSceneManager:sceneID%3A**-default))
오류  06:45:54.559556+0900    runningboardd   RBSStateCapture remove item called for untracked item 34-14990-2386948 (target:[app<**(A7E14B0A-11CC-4E38-A0DE-8FA575E5248D)>:52003](UIScene:com.apple.frontboard.systemappservices::FBSceneManager:sceneID%3A**-default))
```

### Expo 배포 환경에서의 빌드 테스트

Expo 프로젝트 폴더에서 `npx expo prebuild` 후 Android Studio에서 `./android` 폴더를 열어 최대한 프로덕션 환경과 비슷하게 테스트해 볼 수 있다.
웬만한 경우라면 Android Studio 에서 Logcat을 사용하여 로그를 확인하면 대부분의 오류를 수정할 수 있다고 생각한다.

IOS 테스트의 경우는(맥북 필요) `npx expo prebuild` 후 `cd ./ios` -> `pod install` -> `xed` 명령어를 사용하여 Xcode에서 테스트 가능하다.
