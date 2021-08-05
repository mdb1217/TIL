# Gradle Kotlin DSL:sun_behind_small_cloud:
### 1. Gradle이란?

Gradle은 유연함과 성능에 초점을 둔 **오픈소스 빌드도구**다. 기존에는 **Groovy DSL**로 작	성하는 방식만을 지원했지만, Gradle 5.0부터 **Kotlin DSL**을 지원하기 시작했다.

### 2. Kotlin DSL이란?

**Kotlin DSL** 은 gradle build script 를 지원하는 kotlin 언어이다. **Android Gradle 플러그인 4.0**부터 Gradle 빌드 구성에서 Kotlin 스크립트(KTS)를 사용할 수 있다. 이는 Gradle 구성 파일에서 일반적으로 사용되는 프로그래밍 언어인 Groovy를 대체할 수 있다.

------

:arrow_forward: **Kotlin DSL**을 통해서 얻을 수 있는 이득은 다음과 같다.

​	:one: 코드 자동완성

​	:two: 오류코드 강조

​	:three: 빠른 문서보기 가능

​	:four: 리팩터링

**※ 이런 이득은 정적 언어가 제공하는 특징이기도 하다.**

<br>

:bulb: ​**기존 build.gradle에 kts 확장자(build.gradle.kts)를 붙여 rename 하면 코드도 자동으로 변경된다.**

​	`include ‘:app’ -> include(“:app”)`

------

:bulb: **Kotlin DSL**은 **모든 문자열을 큰따옴표(`"`)로 작성**하도록 한다. 문자열에서 위치변환자 사용이 자유롭다.

------

:bulb: **Kotlin DSL**을 적용하면 얻을 수 있는 장점 중 하나가 **문법오류 강조표시(Syntax Highlight)** 인데, DSL
문법오류가 발생하면 다음과 같은 강조표시를 볼 수 있다.

![(<code>build.gradle.kts</code>)Kotlin DSL](https://techblog.woowahan.com/wp-content/uploads/img/2019-04-30/gradle-kotlin-dsl-01.png)

<br>

### 3. build.gradle VS build.gradle.kts

- #### 스크립트 파일 이름 지정

  - Groovy로 작성된 Gradle 빌드 파일은 `.gradle` 파일 이름 확장자를 사용한다.

  - Kotlin으로 작성된 Gradle 빌드 파일은 `.gradle.kts` 파일 이름 확장자를 사용한다.

- #### 성능

  - 현재 KTS는 Groovy와 비교할 때 Android 스튜디오 코드 편집기에서 더 나은 통합(integration)을 제공하지만 KTS를 사용하는 빌드는 Groovy를 사용하는 빌드보다 더 느린 경향이 있다.<sup>[[1]](#footnote1)</sup> 따라서 빌드 성능에 유의하며 KTS로 이전을 고려해야함.

    > <a name="footnote1">[1]</a> [Kotlin DSL 이슈: KT-24668 – Kotlin DSL 3-4x
    > slower than Groovy DSL on FIRST USE on many simple projects
    > build](https://github.com/gradle/kotlin-dsl/issues/902)

<br>

### 4. 참고자료

https://developer.android.com/studio/build/migrate-to-kts?hl=ko

https://techblog.woowahan.com/2625/

https://aroundck.tistory.com/7150