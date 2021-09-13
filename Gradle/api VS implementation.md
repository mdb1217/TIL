# api VS implementation :pineapple:

`compile` 키워드가 **Deprecated** 되고 Gradle에 나온 두개의 키워드 `implementation` 과 `api` 의 차이점을 알아보자.

<br>

### 1. api

- `api` 키워드는 이전 `compile` 키워드와 정확히 동일하게 작동함.
- 따라서 모든 `compile`을 `api`로 바꿔도 제대로 동작함.

- `api` 키워드를 사용하여 모듈을 호출할 경우 모듈이 의존하고 있는 다른 모듈까지 접근이 가능함.
- 하지만 동시에 모듈과 라이브러리 사이에 **의존성**이 생겨 `api`로 호출된 라이브러리가 수정 된다면 빌드 시 라이브러리와 관련된 모든 모듈들이 컴파일된다는 단점이 있다.(빌드시간이 오래 걸림) 

### 2. implementation

- `api` 키워드와 달리 `implementation` 키워드를 사용해 라이브러리를 호출하게 되면 의존성에 추가하는 모듈 외 의존하는 다른 모듈에는 접근이 불가능 하다.
- 하지만 모듈과 라이브러리 사이 **의존성**이 없어 `implementation` 키워드를 사용해 호출한 라이브러리의 경우 수정이 되더라도 빌드 시 라이브러리와 관련된 모든 모듈에 영향을 미치지 않는다.

<br>

### 3. api VS implementation(Build Time)

#### :one: api![Change in LibraryD](https://sikeeoh.github.io/css/images/20170828/change_in_libraryd.png)

#### :two: implementation

[![Change in LibraryC](https://sikeeoh.github.io/css/images/20170828/change_in_libraryc.png)](https://sikeeoh.github.io/css/images/20170828/change_in_libraryc.png)

#### :bulb: implementation을 사용하면 빌드 시간이 좀 더 짧아진다는 것을 알 수 있다.

<br>

### 4. 요약

- `api`는 의존성에 추가하는 모듈이 의존하고 있는 다른 모듈 까지 접근이 가능함.

- `implementation`은 의존성에 추가하는 모듈 외 의존하는 다른 모듈에는 접근이 불가능 함.
- 이로 인한 차이가 발생하여 빌드 속도의 개선이 있을 수 있음.

<br>

### 5. 참고자료

https://sikeeoh.github.io/2017/08/28/implementation-vs-api-android-gradle-plugin-3/