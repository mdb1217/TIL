# 7장. RxKotlin의 스케줄러를 사용한 동시성과 병렬 처리

`written by 문다빈`

> **코틀린 리액티브 프로그래밍** 저서를 읽고 작성하는 글입니다.
> 

## 7장. RxKotlin의 스케줄러를 사용한 동시성과 병렬 처리

- RxKotlin은 기본적으로 단일 스레드에서 동작한다

### 💡 동시성 소개

- 동시성이란?

> **프로그래밍 패러다임으로서, 동시 컴퓨팅은 모듈화 프로그래밍의 한 형태. 즉 전체 계산을 작은 단위의 계산으로 분해해 동시적으로 실행할 수 있음**
> 
- 동시성은 전체 작업을 작은 부분으로 나눠 동시에 실행하는 것
- 스레드가 많이 있고 전체 태스크가 적절하게 나뉘면 전체 프로그램이 더 빨리 실행됨

### 💡 병렬 실행과 동시성

- 병렬화는 풀에서 작업을 현명하게 나누는 것
- 각 작업마다 스레드를 만드는 대신 작업 풀을 만들어 기존 스레드를 할당하고 다시 사용할 수 있다
- 동시성은 병렬화를 통해서 이뤄지지만 동일한 것은 아니다
    - 병렬화는 동시성을 달성할 수 있는 방법에 관한 것
- 데이터의 양이 대규모이거나 사용자에게 결과를 표시하기 전에 수행할 작업이 오래 걸리는 경우
    - 동시성은 이와 같은 시나리오에서 유용함
- RxKotlin은 동시에 작업을 수행하지는 않지만 선택한 작업을 동시에 수행할 수 있는 많은 옵션을 제공한다
- 옵저버블 및 플로어블을 구독하면 모든 아이템들을 옵저버 체인이 수신하기 전까지 스레드는 블로킹 상태가 됨(인터벌, 타이머 팩토리 메서드를 사용한 경우는 제외)
- Observable 체인에서 개별 스레드가 각 연산자에게 할당되는 경우(일반적으로 모든 연산자가 Observable 소스에 가입하고 배출 작업을 수행하므로 다음 연산자는 현재 배출을 구독한다)완전히 엉망이 된다
- ReactiveX는 스케줄러, 스케줄링 연산자를 제공한다
- 동기화를 거의 자동으로 수행하고 스레드 간 공유 데이터가 없으므로(함수 프로그래밍의 기본 속성, 즉 함수형 리액티브 프로그래밍) 스레드 관리가 손쉬워진다

### 💡 스케줄러는 무엇인가

- ReactiveX에서 동시성의 핵심은 스케줄러
- 기본적으로 옵저버블과 이에 적용된 연산자 체인은 subscribe가 호출된 동일한 스레드에서 작업을 수행한다
- 옵저버가 onComplete, onError 알림을 수신할 때까지 스레드가 차단된다
- 스케줄러를 사용하면 이것을 변경할 수 있음
- 스케줄러는 스레드 풀
- 스케줄러를 사용해 ReactiveX는 스레드풀을 생성하고 스레드를 실행할 수 있음
- ReactiveX에서 이것은 기본적으로 멀티 스레딩과 동시성을 추상화한 것 → 동시성 구현을 훨씬 쉽게 만들어 줌

### 💡 스케줄러의 종류

- 스레드 풀 관리를 위한 추상화 계층
- 스케줄러 API는 미리 구성된 스케줄러를 제공
- 또한 사용자가 정의하는 스케줄러를 생성할 수 있다
    - Schedulers.io()
    - Schedulers.computation()
    - Schedulers.newThread()
    - Schedulers.single()
    - Schedulers.trampoline()
    - Schedulers.from()

```kotlin
fun main(args: Array<String>) {
    Observable.range(1,10)
            .subscribe {
                runBlocking { delay(200) }
                println("Observable1 Item Received $it")
            }

    Observable.range(21,10)
            .subscribe {
                runBlocking { delay(100) }
                println("Observable2 Item Received $it")
            }
}
```

- 이 프로그램의 총 실행 시간은 3100밀리초인데 실행 중간에 스레드 풀이 유휴 상태였다(인쇄 전에 지연이 수행됨)
- 스케줄러를 사용하면 이 시간을 크게 줄일 수 있음

```kotlin
fun main(args: Array<String>) {
    Observable.range(1, 10)
            .subscribeOn(Schedulers.computation())//(1)
            .subscribe {
                runBlocking { delay(200) }
                println("Observable1 Item Received $it")
            }

    Observable.range(21, 10)
            .subscribeOn(Schedulers.computation())//(2)
            .subscribe {
                runBlocking { delay(100) }
                println("Observable2 Item Received $it")
            }
    runBlocking { delay(2100) }//(3)
}
```

- 예제에서 옵저버블은 동시에 배출을 수행함
- **subscribeOn(Schedulers.computation()) 코드는 양쪽 다운스트림이 다른 (백그라운드) 스레드에서 Observable을 구독할 수 있도록 해 동시성에 영향을 미침**
- 이렇게 코드를 작성하면 2100밀리초 만에 두 개의 구독의 모든 배출을 처리한다 → 이것으로 1000밀리초를 절약할 수 있음

### 💡 Schedulers.io(): I/O 연관 스케줄러

- Schedulers.io()는 I/O 관련 스레드를 제공
- Schedulers.io()는 I/O 관련 작업을 수행할 수 있는 무제한의 워커 스레드를 생성하는 스레드풀을 제공
- I/O 관련 스레드 → 이 풀의 모든 스레드는 블로킹이고 I/O작업을 더 많이 수행하도록 작성
    - 따라서 계산 집약적인 작업보다 CPU 부하는 적지만 대기 중인 I/O 작업으로 인해 조금 더 오래 걸릴 수 있음
    - I/O 작업은 파일 시스템, 데이터베이스, 서비스 또는 I/O 장치와의 상호작용을 의미
    - **메모리가 허용하는 무제한의 스레드를 생성해 OutOfMermory 오류를 일으킬 수 있으므로 이 스케줄러를 사용할때는 주의해야 한다**

### 💡 Schedulers.computation(): CPU 연관 스케줄러

- Schedulers.computation()은 사용 가능한 CPU 코어와 동일한 수의 스레드를 가지는 제한된 스레드풀을 제공함
- 이 스케줄러는 CPU를 주로 사용하는 작업을 위한 것(닉값함)
- Schedulers.computation()은 CPU 집중적인 작업에만 사용해야 하며 다른 종류의 작업에 사용해서는 안된다 → Why? 이 스케줄러의 스레드가 CPU 코어를 사용 중인 상태로 유지해서
- I/O 관련이나 계산과 관련되지 않은 작업에 사용되는 경우 전체 애플리케이션의 속도를 저하시킬 수 있음
- I/O에 관련된 작업에는 Schedulers.io()를 사용하고 계산 목적에는 Schedulers.computation()을 고려해야 하는 주된 이유
    - computational() 스레드가 프로세서를 더 잘 활용함
    - 사용 가능한 CPU 코어보다 더 많은 스레드를 생성하지 않고 스레드를 재사용함
    - Schedulers.io()는 무제한 스레드를 제공하고 있음
    - io() 블록에서 10,000개의 계산 작업을 병렬로 예약하는 경우 10,000개의 작업은 각각 자체 스레드를 가지게 됨
        - 그렇게 되면 CPU를 놓고 서로 경쟁하게 된다
        - 이는 컨텍스트 전환 비용을 발생시킴

### 💡 Schedulers.newThread()

- Schedulers.newThread()는 제공된 각 작업에 대해 새 스레드를 만드는 스케줄러를 제공함
- Schedulers.io()와의 차이점
    - Schedulers.io()는 스레드 풀을 사용하고 새로운 작업을 할당받을 때마다 먼저 스레드 풀을 조사해 유휴 스레드가 작업을 실행할 수 있는지 확인한다
    - 작업을 시작하기 위해 기존의 스레드를 사용할 수 없으면 새로운 스레드를 생성함
    - Schedulers.newThread()는 스레드 풀을 사용하지 않는 대신 모든 요청에 대해 새로운 스레드를 생성하고 그 사실을 잊어버린다
- 대부분의 경우 Schedulers.computation()을 사용하고, 그렇지 않은 경우 Schedulers.io()를 고려해야함
- Schedulers.newThread()는 웬만해선 사용하지 않는 것이 좋음 → 스레드는 매우 비싼 자원이라 새 스레드를 가능한 많이 만들지 않도록 우리는 노력해야해서..

### 💡 Schedulers.single()

- Schedulers.single()는 하나의 스레드만 포함하는 스케줄러를 제공하고 모든 호출에 대해 단일 인스턴스를 반환함
- 반드시 순차적으로 작업을 실행해야하는 상황에서 유용한 옵션
- 하나의 스레드만 제공하므로 여기에 대기 중인 모든 작업은 순차적으로 실행될 수 있다

### 💡 Schedulers.trampoline()

- Schedulers.trampoline() 스케줄러는 순차적으로 실행됨(Schedulers.single()과 비슷)
- Schedulers.single()은 모든 작업이 순차적으로 실행되도록 보장하지만 호출된 스레드와 병렬로 실행될 수 있다
- 스레드를 원하는 대로 호출하는 Schedulers.single()과 달리 Schedulers.trampoline()은 호출된 스레드를 작업 큐에 넣는다
- 따라서 호출된 스레드에 이어서 순차적으로 실행됨

```kotlin
fun main(args: Array<String>) {

    async(CommonPool) {
        Observable.range(1, 10)
                .subscribeOn(Schedulers.single())//(1)
                .subscribe {
                    runBlocking { delay(200) }
                    println("Observable1 Item Received $it")
                }

        Observable.range(21, 10)
                .subscribeOn(Schedulers.single())//(2)
                .subscribe {
                    runBlocking { delay(100) }
                    println("Observable2 Item Received $it")
                }

        for (i in 1..10) {
            delay(100)
            println("Blocking Thread $i")
        }
    }

    runBlocking { delay(6000) }
}
```

- 두 개의 구독이 순차적으로 실행되더라도 호출 스레드와 병렬로 실행된다

```kotlin
fun main(args: Array<String>) {

    async(CommonPool) {
        Observable.range(1, 10)
                .subscribeOn(Schedulers.trampoline())//(1)
                .subscribe {
                    runBlocking { delay(200) }
                    println("Observable1 Item Received $it")
                }

        Observable.range(21, 10)
                .subscribeOn(Schedulers.trampoline())//(2)
                .subscribe {
                    runBlocking { delay(100) }
                    println("Observable2 Item Received $it")
                }

        for (i in 1..10) {
            delay(100)
            println("Blocking Thread $i")
        }
    }

    runBlocking { delay(6000) }
}
```

- 스케줄러가 호출 스레드로 순차적으로 실행함

### 💡 Schedulers.from

- 사용자가 정의한 스케줄러를 원할 경우 ReactiveX는 모든 실행 프로그램을 스케줄러로 변환할 수 있는 **Schedulers.from(executor : Executor)**을 제공함

```kotlin
fun main(args: Array<String>) {

    val executor:Executor = Executors.newFixedThreadPool(2)//(1)
    val scheduler:Scheduler = Schedulers.from(executor)//(2)

    Observable.range(1, 10)
            .subscribeOn(scheduler)//(3)
            .subscribe {
                runBlocking { delay(200) }
                println("Observable1 Item Received $it - ${Thread.currentThread().name}")
            }

    Observable.range(21, 10)
            .subscribeOn(scheduler)//(4)
            .subscribe {
                runBlocking { delay(100) }
                println("Observable2 Item Received $it - ${Thread.currentThread().name}")
            }

    Observable.range(51, 10)
            .subscribeOn(scheduler)//(4)
            .subscribe {
                runBlocking { delay(100) }
                println("Observable3 Item Received $it - ${Thread.currentThread().name}")
            }
    runBlocking { delay(10000) }//(5)
}
```

- Executors.newFixedThreadPool() 메서드(고정 개수의 스레드 풀을 생성하는 메서드)를 사용해 executor를 생성하고 Schedulers.from(executor : Executor)의 도움으로 스케줄러 인스턴스를 생성

### 💡 스케줄러 사용법: subscribeOn, observeOn 연산자

- 스케줄러를 구현하는 데 도움이 되는 두 가지 연산자가 있다 → subscribeOn, observeOn

### 💡 구독 시 스레드 변경: subscribeOn 연산자

- 구독 기간 전체에 걸쳐 단일 스레드일 수도 있고 다른 레벨의 다른 스레드일 수도 있음
- 기본적으로 구독을 수행하는 스레드는 달리 지시하지 않는 한 모든 배출을 구독자에게 가져오는 책임을 가진다

```kotlin
fun main(args: Array<String>) {
    listOf("1","2","3","4","5","6","7","8","9","10")
            .toObservable()
            .map {
                item->
                println("Mapping $item - ${Thread.currentThread().name}")
                return@map item.toInt()
            }
            .subscribe {
                item -> println("Received $item - ${Thread.currentThread().name}")
            }

    runBlocking { delay(1000) }
}
```

- 메인 스레드가 전체 구독을 실행함

```kotlin
fun main(args: Array<String>) {
    listOf("1","2","3","4","5","6","7","8","9","10")
            .toObservable()
            .map {
                item->
                println("Mapping $item - ${Thread.currentThread().name}")
                return@map item.toInt()
            }
            .subscribeOn(Schedulers.computation())//(1)
            .subscribe {
                item -> println("Received $item - ${Thread.currentThread().name}")
            }

    runBlocking { delay(1000) }
}
```

- subscribeOn 연산자는 전체 구독에 대한 스레드를 변경함
- 구독의 흐름 중 원하는 곳 어디에서나 사용할 수 있다 → 전체 스레드를 한 번에 변경한다

### 💡 다른 스레드에서 관찰: observeOn 연산자

- computation 스레드에서 계산을 수행하고 io 스레드에서는 결과를 표시하도록 할 수 있다
- subscribeOn 전체 구독에 대해서 스레드를 지정하지만 특정 연산자에 대한 스레드를 지정하려면 support가 필요함 → 이 때 필요한 것이 **observeOn**
- observeOn 연산자는 그 후에 호출되는 모든 연산자에 스케줄러를 지정한다

```kotlin
fun main(args: Array<String>) {
    listOf("1","2","3","4","5","6","7","8","9","10")
            .toObservable()
            .observeOn(Schedulers.computation())//(1)
            .map {
                item->
                println("Mapping $item - ${Thread.currentThread().name}")
                return@map item.toInt()
            }
            .observeOn(Schedulers.io())//(2)
            .subscribe {
                item -> println("Received $item - ${Thread.currentThread().name}")
            }

    runBlocking { delay(1000) }
}
```

- Schedulers.computation()에서 map 연산을 수행하고 Schedulers.io()에서 구독 결과(onNext)를 받음
- map 연산자 바로 앞에 observeOn(Schedulers.computation())을 호출해 computation 스레드를 지정했고, 결과를 받기 위해 subscribe 전에 observeOn(Schedulers.io())를 호출해 io 스레드로 전환함
- 이 프로그램은 컨텍스트 스위치를 수행함
- 스레드 간에 데이터를 교환하고 통신하는 작업을 쉽게 구현함 → 이는 추상화된 스케줄러 덕분에 가능