# Android App Bundle(AAB):apple:

![AAB(Android App Bundle)](https://yozm.wishket.com/media/news/912/image002.png)

### 1. AAB(Android App Bundle)란?

**Android App Bundle**은 앱의 모든 컴파일된 코드 및 리소스를 포함하며 APK 생성 및 서명을 Google Play에 맡기는 게시 형식이다. **Android Package(APK)**는 이미 완성된 안드로이드 앱 파일인 반면, **Android App Bundle(AAB)**은 APK를 완성해주는 요소를 담은 패키지다.
<br>

### 2. AAB의 장점

##### Google Play는 App Bundle을 사용하여 각 기기 설정에 맞게 최적화된 APK를 생성하고 제공한다.

→ 따라서 앱을 실행하기 위해서는 **특정 기기에 필요한 코드와 리소스만 다운로드**하면 됩니다. 개발자는 더 이상 다양한 기기에 대한 지원을 최적화하기 위해 여러 개의 APK를 빌드, 서명 및 관리할 필요가 없으며, 사용자는 더 작고 최적화된 앱을 다운로드하게 된다.
<br>

- #### 압축 다운로드 크기 제한

  - Android App Bundle을 사용하여 게시하면 사용자가 가능한 한 가장 작은 다운로드로 앱을 설치가능
  - **압축 다운로드 크기 제한이 150MB로** 증가(사용자가 앱을 다운로드할 때 앱을 설치하는 데 필요한 압축 APK의 총 크기(예: 기본 APK + 구성 APK)는 150MB 이하여야 함.)
  - 하지만, App Bundle을 업로드할 때 Play Console에서 앱 또는 주문형 기능의 다운로드 크기가 **150MB**를 초과하는 것으로 확인되면 오류가 발생함.
  - AAB 형태로 배포하면 앱 크기가 **평균 15%** 줄어든다고 함.

- #### Play Feature Delivery

  ```
  App Bundle의 고급 기능을 사용하여 앱의 특정 기능을 조건부로 전송하거나 주문형으로 다운로드할 수 있도록 함.
  ```

- #### Play Asset Delivery(Game)

  ```
  대량의 게임 애셋을 전송하기 위한 Google Play의 솔루션으로, 개발자에게 고성능의 유연한 전송 방법을 제공.
  ```

  <br>

### 3. AAB의 단점

- #### 보안에 대한 우려

  - 모든 안드로이드 앱에는 개발자의 서명 파일이 들어간다. 
  - **APK에서는 서명 파일을 개발자가직접 첨부한다.**
  - 따라서 누군가가 앱을 멋대로 변형해 배포하려고 해도, 원래 개발자의 서명이 없기 때문에 공식 앱이 아님을 확인할 수 있다.
  - **하지만 AAB의 경우 서명은 개발자가 아닌 Google Play가 대신한다.**
  - Google은 대리 서명에 있어 최고의 보안을 약속했지만, 개발자 입장에서는 여전히 보안에대한 불안도 남아있을 수밖에 없음.

  <br>

### 4. 2021 8월부터 Google Play에서는..

| **TYPE OF RELEASE**            | **REPLACED**                                                 | **REQUIRED AUG 2021**                                        |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| New apps on Google Play        | APK                                                          | [Android App Bundle (AAB)](https://developer.android.com/guide/app-bundle#get_started) |
| Expansion files (OBBs)         | [Play Asset Delivery](https://developer.android.com/guide/app-bundle/asset-delivery) or [Play Feature Delivery](https://developer.android.com/guide/app-bundle/play-feature-delivery) |                                                              |
| Updates to existing apps       | No change                                                    |                                                              |
| New instant experiences        | Instant app ZIP                                              | [Instant-enabled Android App Bundle (AAB)](https://developer.android.com/topic/google-play-instant/getting-started/instant-enabled-app-bundle) |
| Updates to instant experiences |                                                              |                                                              |

**[출처] [Android Devlopers Blog](https://android-developers.googleblog.com/2021/06/the-future-of-android-app-bundles-is.html)**

2021년 8월부터 Google Play에서는 AAB로 앱을 퍼블리시하는 걸 강제한다. Google에서는 사용자들에게 필요한 코드만 제공해 앱 용량을 줄이고, 개발자들은 여러 개의 빌드를 준비할 필요가 없다는 이유에서 AAB로 앱을 퍼블리시 하는 걸 강제한다고 했지만, 일각에서는 Google Play의 점유율을 올리고 앱 배포에 있어 Google Play의 힘을 올리기위해 AAB 퍼블리시를 강제한다는 의견도 있다.

#### <br>

### 5. 참고자료

https://developer.android.com/guide/app-bundle?hl=ko

https://android-developers.googleblog.com/2021/06/the-future-of-android-app-bundles-is.html

https://yozm.wishket.com/magazine/detail/912/