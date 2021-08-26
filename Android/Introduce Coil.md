# Coil을 소개합니다!:lemon:

### 1. Coil이란?

**Co**routine **I**mage **L**oader의 약자이며 Instacart에서 만든 **Kotlin Coroutines(코루틴)**으로 만들어진 가벼운 **이미지 로딩 라이브러리**이다. **Glide**를 많이 벤치마킹 했다고 하며, **ImageView**의 확장함수 형태로 지원된다. 함수형 언어의 특징을 가진 코틀린의 특성상, 다른 라이브러리보다 더욱 간결한코드를 구성할 수 있다.

<br>

### 2. 장점

- **Fast** - 메모리/디스크 캐싱에 최적화되어 있다.
- **Lightweight** - 약 2000개의 method로 구성되어있다. picasso와 비슷하고, Glide와 Fresco보다는 작다(메소드 개수가 적게 구성되어 있다는 의미)
- **Easy to use** - kotlin을 사용하기 때문에 간단하다
- **Modern** - 최근 library들을 사용한다, Corouitne, OkHttp, Okio, AndroidX Lifecycles
- 다이나믹 이미지 샘플링을 지원함.
- Coil의 이미지 파이프라인(PipeLine)은 크게 세 가지로 구성됩니다.
  **Mappers, Fetchers, Decoders** 이러한 인터페이스를 사용하여 기본 이미지 로딩 기능을 강화하거나 재정의하고 Coil에 새로운 파일 형식에 대한 지원을 추가할 수 있음.
- **Jetpack Compose**(컴포즈도 Coroutine 기반)와 잘 통합되도록 설계 되었다.

<br>

### 3. 단점

- 실무에서 사용하는 기업이 아직은 많지 않음.
- 최소 Android SDK 버전 14부터 지원을 하나 최신버전에서 사용을 하는 것을 권장함.(AndroidX)
- 생각보다 CPU, Memory의 사용량이 많음. - ([참고자료](https://androiddeepdive.github.io/Team-Blog/2021/06/24/2021-06-24%20Image%20Loading%20and%20Caching%20Library%20Part%203/))

<br>

### 4. 어떻게 프로젝트에 적용 할까?(기본 사용법)

1. **implementation**
   * `mavenCentral()` 추가

```kotlin
implementation("io.coil-kt:coil:1.3.2")
```

2. **ImageView로 이미지를 로딩하기 위해, Coil의 load 확장함수를 사용한다.**

```kotlin
// URL
imageView.load("https://www.example.com/image.jpg")

// Resource
imageView.load(R.drawable.image)

// File
imageView.load(File("/path/to/image.jpg"))

// And more...
```

<br>

### 5. 참고자료

https://github.com/coil-kt/coil

https://androiddeepdive.github.io/Team-Blog/2021/06/24/2021-06-24%20Image%20Loading%20and%20Caching%20Library%20Part%203/

https://tourspace.tistory.com/364

https://velog.io/@jojo_devstory/Android-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EB%A1%9C%EB%94%A9-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC-Coil-%EC%9D%84-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90