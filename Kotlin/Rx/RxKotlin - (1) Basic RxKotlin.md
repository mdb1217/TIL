# Rx 스터디

`written by 문다빈`

> **코틀린 리액티브 프로그래밍** 저서를 읽고 작성하는 글입니다.
> 

## 1장. 리액티브 프로그래밍의 소개

<aside>
💡 리액티브 프로그래밍이란 무엇인가?

</aside>

1. 리액티브 프로그래밍은 데이터 스트림과 변경 사항 전파를 중심으로 하는 비동기 프로그래밍 패러다임이다. 
2. 데이터와 데이터 스트림에 영향을 미치는 모든 변경사항을 다른 구성요소에 전파하는 프로그램을 리액티브 프로그램이라고 한다.

```kotlin
fun main(args: Array<String>) {
    var number = 4
    var isEven = isEven(number)
    println("The number is " + (if (isEven) "Even" else "Odd"))
    number = 9
    isEven = isEven(number)
    println("The number is " + (if (isEven) "Even" else "Odd"))
}

fun isEven(n:Int):Boolean = ((n % 2) == 0)
```

❓**여기서 isEven의 결과값은? → 둘 다 “Even” → number 값이 9로 바뀌어도 “Even”이라고 나온다.**

**→ why?** 반응형으로 작성되지 않았기 때문.

<aside>
💡 함수형 리액티브 프로그래밍을 적용해야 하는 이유

</aside>

🔔 **함수형 리액티브 프로그래밍을 사용했을 때의 이점**

1. **콜백 지옥 제거**
    1. 함수형 리액티브는 push 시나리오로 작동한다. → 따라서 콜백이 길어지는 걸 방지할 수 있다.
        1. pull 시나리오 : **비동기 request 던지고 response를 기다렸다가 callback으로 받아서 처리**
        2. push 시나리오 : **비동기 request를 던지고, 외부데이터 스트림에 대한 subscribe형식으로 유입에 반응하도록 처리**
2. **익셉션 처리가 표준화 되어있음**
    1. 복잡한 작업, HTTP를 사용하는 작업에서는 다양한 오류가 발생할 수 있다. 따라서 표준화된 오류 처리를 제공하는 Rx는 강점을 가짐.
    2. Rx는 다양한 Error Handling 메소드를 제공함.
3. **간결해진 스레드**
    1. `Thread()` 활용해서 직접 만드는 것보다 간결함.
4. **비동기 연산이 간단하다**
    1. 스레드를 활용해서 비동기 연산을 하므로, 스레드 연산을 간단하게 만든 RxJava, RxKotlin은 비교적 간단하게 비동기 연산이 가능하다. 
5. **모든 작업에 일관되게 작용하는 API**
    1. 직관적인 API를 제공함
6. **함수형 접근**
    1. 선언형(함수형)으로 작성해서 가독성이 좋다.
        1. **선언형**이란? → 프로그래머가 무엇을 하고싶은지 기술하는 것. 즉, 최종 목적만 기술하고, 내부 구성까지는 크게 신경쓸 필요 없는 개발 방식.
7. 유지 보수 가능하고 테스트 가능한 코드
    1. 유지 보수가 쉬운 건 알겠지만, 테스팅이 쉬운지까지는 잘 와닿지 않는다.

<aside>
💡 리액티브 선언

</aside>

[리액티브 선언문](https://www.reactivemanifesto.org/ko)

리액티브 선언은 4가지 리액티브 원리를 정의해 놓은 문서로, 2022. 11. 09 기준 32398명의 개발자가 서명한 리액티브 원리 관련 문서.

![https://www.reactivemanifesto.org/images/reactive-traits-ko.svg](https://www.reactivemanifesto.org/images/reactive-traits-ko.svg)

1. **응답성 :** 시스템은 즉각 응답해야한다. 응답성 있는 시스템은 신속하고 일관성 있는 응답 시간을 유지해 일관된 서비스 품질을 제공.
2. **탄력성 :** 시스템에 장애가 발생하더라도 응답성을 유지해야 한다. 탄력성은 복제, 봉쇄, 격리, 위임에 의해서 이루어진다. 장애는 각 컴포넌트 내부로 억제돼 각 컴포넌트들을 서로 격리시키는데, 그래서 하나의 컴포넌트에 장애가 발생하더라도 전체 시스템에 영향을 끼치지 못하게 된다.
3. **유연성 :** 리액티브 시스템은 작업량이 변하더라도 그 변화에 대응하고 응답성을 유지해야한다. 리액티브 시스템은 상용 하드웨어 및 소프트웨어 플랫폼에서 효율적인 비용으로 유연성을 확보한다.
4. **메시지 구동(기반) :** 탄력성의 원칙을 지키려면 리액티브 시스템은 비동기적인 메시지 전달에 의존해 컴포넌트들 간의 경계를 형성해야 한다.

<aside>
💡 리액티브 스트림 표준 사양

</aside>

2013년 리액티브 프로그래밍의 확산이 시작되고 관련 프레임워크가 등장하면서 넷플릭스, 피보탈, 라이트벤드의 주도로 탄생한 표준 사양.

[Reactive Streams](https://www.reactive-streams.org/)

<aside>
💡 코틀린을 위한 리액티브 프레임워크

</aside>

🔔  코틀린을 활용해 리액티브 프로그래밍을 할 때 도움이 되는 라이브러리

- [RxKotlin](https://github.com/ReactiveX/RxKotlin)
- [Reactor-Kotlin](https://github.com/reactor/reactor-kotlin-extensions)
- [Redux-Kotlin](https://reduxkotlin.org)
- [FunKTionale](https://github.com/MarioAriasC/funKTionale)
- [RxJava](https://github.com/ReactiveX/RxJava)(그 외 다른 자바 리액티브 프레임워크도 가능) → 코틀린은 자바와 100% 상호 운용 가능해서..
- [Coroutine](https://kotlinlang.org/docs/coroutines-guide.html)

<aside>
💡 RxKotlin 시작하기

</aside>

코틀린을 위한 리액티브 프로그래밍을 구현하는 프레임워크. 함수형(선언형) 프로그래밍의 영향을 많이 받음.

- **함수 컴포지션**(함수를 합성해서 사용하는 것을 의미함. 체이닝 말하는 듯?)을 선호
- **전역 상태와 함수의 사이드 이펙트 방지** → 공유 자원에 대한 관리가 용이!(아무래도 반응형 프로그래밍이다 보니..)
- **Producer - Consumer 구조의 옵저버 패턴**
- **결합, 스케줄링, 스로틀링, 변형, 에러 처리, 라이프 사이클 관리** 등을 가능하게 해주는 다양한 연산자 존재

<aside>
💡 RxKotlin 다운로드와 설정

</aside>

[https://github.com/ReactiveX/RxKotlin](https://github.com/ReactiveX/RxKotlin)

깃허브 레포 위키 페이지 참조

- 라이브러리 추가

```kotlin
// 모듈 gradle(kts 기준)
implementation("io.reactivex.rxjava3:rxkotlin:3.x.y")

// project gradle -> jitpack 추가
repositories {
    maven { url 'https://jitpack.io' }
}
```

`→ 현재 참조하는 책은 RxKotlin 2.1.0 버전 기준`

<aside>
💡 RxJava의 푸시 메커니즘, 풀 메커니즘 비교

</aside>

RxKotlin은 iterator(반복자) 패턴의 Pull 메커니즘 대신, 데이터/이벤트 시스템으로 대표되는 옵저버(블) 패턴 중심의 **Push 메커니즘으로 구현**

- **iterator(반복자) 패턴** : **반복자 패턴**(iterator pattern)은 객체 지향 프로그래밍에서 반복자를 사용하여 컨테이너를 가로지르며 컨테이너의 요소들에 접근하는 디자인 패턴.(~~콜백지옥이 형성되는 이유..~~)
- **옵저버 패턴** : **옵서버 패턴**(observer pattern)은 객체의 상태 변화를 관찰하는 관찰자들, 즉 옵저버들의 목록을 객체에 등록하여 상태 변화가 있을 때마다 메서드 등을 통해 객체가 직접 목록의 각 옵저버에게 통지하도록 하는 디자인 패턴이다. 주로 분산 이벤트 핸들링 시스템을 구현하는 데 사용된다. 발행/구독 모델로 알려져 있기도 하다.
- **[옵저버블](https://reactivex.io/documentation/ko/observable.html)이란?** : ReactiveX에서 옵저버가 구독하는 대상
    - 옵저버블 안에 데이터를 조회하고 변환하는 메커니즘을 정의한 후, 옵저버블이 이벤트를 발생시키면 옵저버의 관찰자가 그 순간을 감지하고 준비된 연산을 실행시켜 결과를 리턴하는 메커니즘.

![https://reactivex.io/assets/operators/legend.png](https://reactivex.io/assets/operators/legend.png)

RxJava(Kotlin)에서는 Lazy Evaluation이 일어나며, 동기식-비동기식 처리가 모두 가능하다.

```kotlin
fun main(args: Array<String>) {
    val list:List<Any> = listOf("One", 2, "Three", "Four", 4.5, "Five", 6.0f) // 1
    val iterator = list.iterator() // 2
    while (iterator.hasNext()) { // 3
        println(iterator.next()) // Prints each element 4
    }
}
```

💡**실행 결과 :** 순차적으로 실행 됨.

→ 여기서 중요하게 볼 점 : 데이터가 수신돼 준비될 때까지 현재 스레드는 블로킹된 상태에서 리스트로부터 데이터를 당겨옴. 따라서, 네트워크 호출이나 데이터베이스 쿼리를 사용해 데이터를 가져오면 스레드가 오랫동안 블로킹될 것임.(안드로이드에선 ANR 발생시킬 위험이 큼) 별도의 스레드를 파는 방식도 있겠지만, 구현이 어렵고 복잡하다는 단점이 있음.

💡**따라서, ReactiveX에서는..**

→ 옵저버블을 제공한다!

- 옵저버블 클래스에는 컨슈머가 소비할 수 있는 값을 생성하는 기본 컬렉션, 계산식이 있음.
- iterator(반복자) 패턴에서는 이런 값들을 직접 당겨와서 사용하지만, 옵저버 패턴에서는 컨슈머가 프로듀서로부터 값을 당겨오지 않는다.
- 대신 프로듀서는 컨슈머에게 값을 알림으로 푸시한다!(구독했기 때문에 값을 보내줌!)

```kotlin
fun main(args: Array<String>) {
    var list:List<Any> = listOf("One", 2, "Three", "Four", 4.5, "Five", 6.0f)
    var observable: io.reactivex.Observable<Any> = list.toObservable(); //1

    observable.subscribeBy(  // 2 named arguments for lambda Subscribers
            onNext = { println(it) },
            onError =  { it.printStackTrace() },
            onComplete = { println("Done!") }
    )
}
```

💡**실행 결과 :** 모든 값을 출력

![Untitled](Rx%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%A5%E1%84%83%E1%85%B5%20abcf1199fe9c4586aef33804dd295055/Untitled.png)

💡**동작 과정**

1. 리스트 생성
2. 1번에서 생성한 리스트로 Observable 인스턴스를 생성
3. Observable 인스턴스를 구독한다(Named Parameter 활용해서 좀 더 가독성 좋게 표기)
4. Observable을 구독했기 때문에 모든 변경 사항은 onNext로 푸시
5. 모든 데이터가 푸시됐을 때는 onComplete, 에러가 발생했을 때는 onError가 호출

💡**알 수 있는 것**

- Observable 인스턴스를 사용해 비동기 스트림을 생성
- 모든 구독자들에게 데이터 변경을 푸시

<aside>
💡 ReactiveEvenOdd 프로그램

</aside>

```kotlin
fun main(args: Array<String>) {
    val subject: Subject<Int> = PublishSubject.create()

    subject.map({ isEven(it) }).subscribe({println("The number is ${(if (it) "Even" else "Odd")}" )})

    subject.onNext(4)
    subject.onNext(9)
}
```

💡**실행 결과**

![Untitled](Rx%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%A5%E1%84%83%E1%85%B5%20abcf1199fe9c4586aef33804dd295055/Untitled%201.png)

💡**동작 과정**

1. subject에 숫자 통지(현재 여기선 subject.onNext(4), subject.onNext(9))
2. map내의 메소드를 호출
3. 차례대로 메소드의 반환값(subscribe내에 it으로 들어감)과 함께 subscribe내의 함수가 호출됨

💡**알 수 있는 것**

- subject.onNext 메서드는 새로운 값을 subject로 전달해 처리할 수 있다.

💡 **Observable VS Subject**

- **Observable :** 옵저버를 통해 구독 되는 객체
- **Subject :** 옵저버블, 옵저버의 역할을 하나의 객체에서 다 처리하는 객체

<aside>
💡 ReactiveCalculator 프로젝트

</aside>

- 자세한 내용은 코드 참조
- map은 onNext를 통해 전달한 값을 사용해 결과값을 산출해 냄(즉 내부에서 값을 처리해서 반환)
- 데이터 변경, 사용자 입력과 같은 이벤트를 관찰할 수 있다