# KSP(Kotlin Symbol Processing)란 무엇인가?​ :grapes:

### 1. KSP란?

**KSP(Kotlin Symbol Processing)**는 코틀린에서 경량화 된 컴파일러 플러그인을 개발할 수 있는 API다. 학습곡선을 최소한으로 줄이고, 코틀린의 기능을 활용할 수 있는 단순화된 API를 제공한다. KSP는 [KAPT](https://kotlinlang.org/docs/reference/kapt.html)와 비슷한 기능을 제공하지만, 속도가 최대 **2배** 더 빠르고 Kotlin 컴파일러 기능에 직접 액세스할 수 있으며 **여러 플랫폼과의 호환성**을 염두에 두고 개발 중인 도구이다. KSP는 Kotlin 1.4.30 버전 이상과 호환된다.(2021년 8월 7일 기준 KSP는 베타버전이다.)

<br>

### 2. KSP를 사용해야하는 이유

:arrow_forward: Kotlin은 자체적으로 **어노테이션 프로세싱(Annotation Processing)** 시스템을 갖추고 있지않다. 따라서 **KAPT(Kotlin Annotation Processing Tool)**의 경우 자바단의 어노테이션 프로세싱 시스템을 사용하는데, 이를 위해 Java 스텁(Stubs)을 생성해야해서 KAPT의 경우 실행 속도가 느릴 수 있다.

------

:arrow_forward: **KSP**는 설계 시점부터 Kotlin 단에서 어노테이션 프로세싱이 어떻게 동작할지 고려하여 만들어진 API이다. KSP의 경우 Kotlin 코드를 직접 파싱(parsing)하기 위한 강력하면서도 간단한 API를 제공하여 KAPT의 스텁 생성으로 인해 빌드 속도가 저하되는 문제를 크게 줄여준다. **어노테이션 프로세싱**이 자주 일어나는 **Room** 라이브러리를 사용하여 벤치마크를 수행했을 때, KSP가 KAPT보다 대략 **2배 정도 빨리 동작하는 결과**를 내기도 했다.

------

:arrow_forward: 같은 모듈에서 KAPT와 KSP를 사용하면 처음에는 **빌드 속도가 느려질 가능성**이 있으므로 KSP와 KAPT는 각각 별개의 모듈에서 사용하는 것이 좋다.

<br>

### 3. KSP로의 migration

KAPT와 KSP 사용 시 유일한 차이점은 다음의 사진과 같이 두 줄에 불과하다. 따라서 기존의 kapt에서 ksp로의 **migration**은 어렵지 않을 것이다.

![img](https://lh6.googleusercontent.com/T0oTigG3n1eF0dUYiq_r9mkPTXRM9mV_brURi16EjNITxjofaHqTwDTV_g4sxThDf_BfoC--ZgXFiJl2ku7-028U9iZwTwqov9UX13DPiv-QkVfvoxAkcFdTcIFnmKgJzZvm_qlMOQ)

<br>

### 4. KSP를 지원하는 라이브러리

| Library          | Status                                                       | Tracking issue for KSP                                       |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Room             | [Experimentally supported](https://developer.android.com/jetpack/androidx/releases/room#2.3.0-beta02) |                                                              |
| Moshi            | [Experimentally supported](https://github.com/ZacSweers/MoshiX/tree/main/moshi-ksp) |                                                              |
| Kotshi           | [Experimentally supported](https://github.com/ansman/kotshi) |                                                              |
| Lyricist         | [Experimentally supported](https://github.com/adrielcafe/lyricist) |                                                              |
| Auto Factory     | Not yet supported                                            | [Link](https://github.com/google/auto/issues/982)            |
| Dagger           | Not yet supported                                            | [Link](https://github.com/google/dagger/issues/2349)         |
| Hilt             | Not yet supported                                            | [Link](https://issuetracker.google.com/179057202)            |
| Glide            | Not yet supported                                            | [Link](https://github.com/bumptech/glide/issues/4492)        |
| DeeplinkDispatch | Not yet supported                                            | [Link](https://github.com/airbnb/DeepLinkDispatch/issues/307) |

[^2021.08.07 기준]: 출처 : https://github.com/google/ksp



#### <br>

### 5. 참고자료

https://developers-kr.googleblog.com/2021/02/announcing-kotlin-symbol-processing-ksp.html

https://www.charlezz.com/?p=45255

https://github.com/google/ksp