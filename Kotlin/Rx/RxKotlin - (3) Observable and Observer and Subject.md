# 3장. 옵저버블과 옵저버와 구독자

`written by 문다빈`

> **코틀린 리액티브 프로그래밍** 저서를 읽고 작성하는 글입니다.
> 

## 3장. 옵저버블과 옵저버와 구독자

### ➕ 더 알아보기..

- **Rx 기본 용어 정리하기**
    - ****Observable****
        
        > event를 내보낼 수 있는 능력 **(emit)**, 관찰 대상
        > 
        - event가 흘러가는 흐름을 추상화한 것
        - Observer 패턴에서 사용하는 용어로는 **Subject**라고 보면 된다
            
            ![여기서 Subject는 관찰당하는 대상](https://johngrib.github.io/wiki/pattern/observer/observer.svg)
            
            여기서 Subject는 관찰당하는 대상
            
        - **Sequence, Stream**으로도 불린다(데이터 타입을 의미)
        - Observable이 내보내는 event를 받아 어떤 작업을 하기 위해 `subscribe()` 메소드가 정의되어 있다
        - **Hot Observable, Cold Observable** 존재
        - Observable이 emit하는 Event 는 세가지 종류가 있다.
            - next, error, completed
            - error, completed 이벤트 발생시 Sequence는 종료된다.
    - ****Observer****
        
        > event를 받을 수 있는 능력 **(receive)**, 관찰자
        > 
        - Observer는 event를 외부에서 주입받을 수 있다.
        - `on()` 메소드의 argument로 event를 주입한다.
    - ****Subject****
        
        > **Observable 인 동시에 Observer**
        > 
        - **Observable 인 동시에 Observer이다.**
        - event를 주입받고, 내보낼 수 있는 능력을 모두 가진다.
        - **ex. 사용자의 button tap event ⇢ subject 가 그 이벤트 받으면 api 찔러서 특정 데이터를 받아와서 내보냄**
            - **observer 능력 : button tap event를 받음**
            - **observable 능력 : 데이터를 내보냄**
    - ****Trait****
        
        > **Observable 보다는 좁은 개념(Observables with a narrower set of behavior than regular observable)**
        > 
        - **특정 작업을 타겟팅한 Observable**
        - **ex. Single, Maybe, Completable 등**
        - 더 특정된, 좁은 개념의 역할을 하는 Observable

### 💡 옵저버블

- 옵저버블은 컨슈머가 소비할 수 있는 값을 산출해 내는 기본 계산 작업을 갖고 있다.
- 컨슈머가 값을 **풀 방식(풀 시나리오 → 콜백형식)**을 사용해 접근하지 않는다.
- 옵저버블은 오히려 **컨슈머에게 값을 푸시하는 역할(푸시 시나리오)**을 한다.
- 옵저버블은 일련의 연산자를 거친 아이템을 최종 옵저버로 내보내는 푸시 기반의 조합 가능한 이터레이터이다.
    - 하지만 Iterable의 경우 다른 개념!
        - **Iterable은 pull, Observable은 push**
        - 차이가 분명 있으면서도 같은 기능을 하고 있어서 **duality를 가졌다**고 한다.
        - **duality란?** **A와 B가 있을 때 A에서 성립하는 정리를 뒤집어서 B에도 적용할 수 있는 경우**를 말한다. A와 B의 본질이 같다는 뜻.
- 옵저버는 아이템들을 소비한다.
- 옵저버는 옵저버블을 구독한다.
- 옵저버블이 그 내부의 아이템들을 내보내기 시작한다.
- 옵저버는 옵저버블에서 내보내는 모든 아이템에 반응한다.

### 💡 옵저버블이 동작하는 방법

옵저버블의 세 가지 중요한 이벤트 메서드

- **onNext : 옵저버블은 모든 아이템을 하나씩 이 메서드에 전달한다**
- **onComplete : 모든 아이템이 onNext 메서드를 통과하면 옵저버블은 onComplete 메서드를 호출한다.**
- **onError : 옵저버블에서 에러가 발생하면 onError 메서드가 호출돼 정의된 대로 에러를 처리한다. onError와 onComplete가 호출되지 않으며, 반대의 경우도 마찬가지이다.**

→ 여기서 주목해야 할 것은 언급하고 있는 옵저버블이 어떤 유형(type)도 될 수 있다는 점이다. **Observable <T>로 정의**되며, 여기서 T는 임의의 클래스가 될 수 있다. **array/list도 옵저버블**로 지정할 수 있다.

```kotlin
fun main(args: Array<String>) {

    val observer: Observer<Any> = object :Observer<Any>{//1
        override fun onComplete() {//2
            println("All Completed")
        }

        override fun onNext(item: Any) {//3
            println("Next $item")
        }

        override fun onError(e: Throwable) {//4
            println("Error Occured $e")
        }

        override fun onSubscribe(d: Disposable) {//5
            println("Subscribed to $d")
        }
    }

    val observable: io.reactivex.Observable<Any> = listOf("One", 2, "Three", "Four", 4.5, "Five", 6.0f).toObservable() //6

    observable.subscribe(observer)//7

    val observableOnList: io.reactivex.Observable<List<Any>> = io.reactivex.Observable.just(listOf("One", 2, "Three", "Four", 4.5, "Five", 6.0f),
            listOf("List with Single Item"),
            listOf(1,2,3,4,5,6))//8

    observableOnList.subscribe(observer)//9
}
```

- Any 타입 → 코틀린에서 모든 클래스는 Any 클래스의 자식 클래스 → **코틀린은 원시 타입(primitive type)과 래퍼 타입(wrapper type)을 따로 구분하지 않는다(ex. int, Integer로 구분하지 않고 Int하나로 처리함 → java코드에선 해당 타입이 자동 변환됨)**
- Observer 인터페이스에는 4개의 메서드가 선언돼 있다.
    - **onComplete() - Observable이 오류 없이 모든 아이템을 처리하면 호출**
    - **onNext(item : Any) -** **함수를 정의했으며 이 함수는 옵저버블이 내보내는 각 아이템에 대해 호출**
    - **onError(e : Throwable) - 옵저버블에 오류가 발생했을 때 호출**
    - **onSubscribe(d : Disposable) - 옵저버가 옵저버블을 구독할 때마다 호출**
- observer는 observable을 구독한다.
- 옵저버블은 리스트를 아이템으로 가질 수 있다.
- Observable은 구독에 의해 onSubscribe 메서드가 호출되고 옵저버블은 아이템을 내보내기 시작.
- onNext 메서드를 통해 아이템을 수신.
- 모든 아이템이 옵저버블에서 배출되면 onComplete 메서드를 호출해 모든 항목이 성공적으로 내보내졌음을 알림.

### 💡 Observable.create 메서드 이해

> Observable.create : 옵저버블을 직접 생성하는 메서드
> 

이 메서드는 **ObservableEmitter <T> 인터페이스의 인스턴스를 입력**받는다.

```kotlin
fun main(args: Array<String>) {

    val observer: Observer<String> = object : Observer<String> {
        override fun onComplete() {
            println("All Completed")
        }

        override fun onNext(item: String) {
            println("Next $item")
        }

        override fun onError(e: Throwable) {
            println("Error Occured ${e.message}")
        }

        override fun onSubscribe(d: Disposable) {
            println("New Subscription ")
        }
    }//Create Observer

    val observable:Observable<String> = Observable.create<String>{//1
        it.onNext("Emit 1")
        it.onNext("Emit 2")
        it.onNext("Emit 3")
        it.onNext("Emit 4")
        it.onComplete()
}

observable.subscribe(observer)

    val observable2:Observable<String> = Observable.create<String>{//2
        it.onNext("Emit 1")
        it.onNext("Emit 2")
        it.onNext("Emit 3")
        it.onNext("Emit 4")
        it.onError(Exception("My Custom Exception"))
}

observable2.subscribe(observer)
}
```

- **Observable.create**를 통해 옵저버블 생성
- **사용자 정의 Exception으로 onError를 호출**하는 것도 가능.
- **Observable.create** 메서드는 사용자가 지정한 데이터 구조를 사용하거나 내보내는 값을 제어 할 때 유용하다.
- 다른 스레드에서 옵저버로 값을 내보낼 수도 있다.
- [Observable 계약](https://reactivex.io/documentation/ko/contract.html)
    - Observable은 Observer에게 연속적으로 알림을 보내야 함(병렬X)
    - 알림 간에는 전후 관계가 있다

### 💡 Observable.from 메서드 이해

- **Observable.from** 메서드는 **Observable.create** 메서드에 비해 상대적으로 간단함
- from 메서드의 도움을 받아 거의 모든 코틀린 구조체로부터 **Observable 인스턴스를 생성 가능**
- RxKotlin 2.0부터 **연산자 오버로드로서 fromArray, fromiterabloe, fromFuture 등과 같은 접미사가 추가**됨

### 💡 Observable.from 메서드 이해

```kotlin
fun main(args: Array<String>) {

    val observer: Observer<String> = object : Observer<String> {
        override fun onComplete() {
            println("All Completed")
        }

        override fun onNext(item: String) {
            println("Next $item")
        }

        override fun onError(e: Throwable) {
            println("Error Occured ${e.message}")
        }

        override fun onSubscribe(d: Disposable) {
            println("New Subscription ")
        }
    }//Create Observer

    val list = listOf("String 1","String 2","String 3","String 4")
    val observableFromIterable: Observable<String> = Observable.fromIterable(list)//1
    observableFromIterable.subscribe(observer)

    val callable = object : Callable<String> {
        override fun call(): String {
            return "From Callable"
        }

    }
    val observableFromCallable:Observable<String> = Observable.fromCallable(callable)//2
    observableFromCallable.subscribe(observer)

    val future:Future<String> = object :Future<String> {
        override fun get(): String = "Hello From Future"

        override fun get(timeout: Long, unit: TimeUnit?): String  = "Hello From Future"

        override fun isDone(): Boolean = true

        override fun isCancelled(): Boolean = false

        override fun cancel(mayInterruptIfRunning: Boolean): Boolean = false

    }
    val observableFromFuture:Observable<String> = Observable.fromFuture(future)//3
    observableFromFuture.subscribe(observer)
}
```

- **Observable.fromIterable** 메서드를 사용해 **Iterable 인스턴스로부터 옵저버블을 생성**
- **Observable.fromCallable** 메서드를 호출해 Callable 인스턴스에서 옵저버블 생성
- **Observable.fromFuture** 메서드를 호출해 Future 인스턴스에서 옵저버블 파생

### 💡 toObservable의 확장 함수 이해

```kotlin
fun main(args: Array<String>) {

    val observer: Observer<String> = object : Observer<String> {
        override fun onComplete() {
            println("All Completed")
        }

        override fun onNext(item: String) {
            println("Next $item")
        }

        override fun onError(e: Throwable) {
            println("Error Occured ${e.message}")
        }

        override fun onSubscribe(d: Disposable) {
            println("New Subscription ")
        }
    }//Create Observer

    val list:List<String> = listOf("String 1","String 2","String 3","String 4")

    val observable:Observable<String> = list.toObservable()

    observable.subscribe(observer)
}
```

- Rx에서는 Iterable 인스턴스에대한 확장함수로 **toObservable()**을 제공한다.
- **toObservable() 내부를 보면 Observables.from** 메서드를 사용해서 구현되어 있음

### 💡 Observable.just 메서드 이해

- **Observable.just**는 넘겨진 인자만을 배출한다.
- Iterable 인스턴스를 Observable.just에 단일 인자로 넘기면 **전체 리스트를 하나의 아이템으로 배출**
- **Observable.from vs Observable.just**
    - **Observable.from : Observable.from은 Iterable 내부의 각각 아이템을 Observable로 생성**
    - **Observable.just : Observable.just는 Iterable을 단일 아이템으로 취급해서 내보냄**
- 인자와 함께 **Observable.just**를 호출
- **Observable.just는 옵저버블을 생성**
- **onNext 알림**을 통해 각각의 아이템을 내보냄
- 모든 인자의 방출이 완료되면 **onComplete 실행**

```kotlin
fun main(args: Array<String>) {
    val observer: Observer<Any> = object : Observer<Any> {
        override fun onComplete() {
            println("All Completed")
        }

        override fun onNext(item: Any) {
            println("Next $item")
        }

        override fun onError(e: Throwable) {
            println("Error Occured ${e.message}")
        }

        override fun onSubscribe(d: Disposable) {
            println("New Subscription ")
        }
    }//Create Observer

    Observable.just("A String").subscribe(observer)
    Observable.just(54).subscribe(observer)
    Observable.just(listOf("String 1","String 2","String 3","String 4")).subscribe(observer)
    Observable.just(mapOf(Pair("Key 1","Value 1"),Pair("Key 2","Value 2"),Pair("Key 3","Value 3"))).subscribe(observer)
    Observable.just(arrayListOf(1,2,3,4,5,6)).subscribe(observer)
    Observable.just("String 1","String 2","String 3").subscribe(observer)//1
}
```

- **Observable.just**의 경우 각각의 인자를 별개의 아이템으로 받아들임

### 💡 Observable의 다른 팩토리 메서드

```kotlin
fun main(args: Array<String>) {
    val observer: Observer<Any> = object : Observer<Any> {
        override fun onComplete() {
            println("All Completed")
        }

        override fun onNext(item: Any) {
            println("Next $item")
        }

        override fun onError(e: Throwable) {
            println("Error Occured ${e.message}")
        }

        override fun onSubscribe(d: Disposable) {
            println("New Subscription ")
        }
    }//Create Observer

    Observable.range(1,10).subscribe(observer)//(1)
    Observable.empty<String>().subscribe(observer)//(2)

    runBlocking{

Observable.interval(300,TimeUnit.MILLISECONDS).subscribe(observer)//(3)
        delay(900)
        val subscription = Observable.timer(400,TimeUnit.MILLISECONDS).subscribe(observer)//(4)
        delay(450)
}

}
```

- **Observable.range()** 팩토리 메서드로 옵저버블을 생성
    - 이 메서드는 옵저버블을 생성하고 제공된 start부터 시작해 count 만큼의 정수를 내보낸다.
- **Observable.empty()** 메서드로 옵저버블을 생성
    - **onNext()로 데이터를 방출하지 않고 즉시 onComplete() 발생**
- **Observable.interval()** 메서드로 옵저버블 생성
    - 지정된 간격만큼의 숫자를 0부터 순차적으로 내보냄
- **Observable.timer()** 메서드로 옵저버블 생성
    - 지정된 시간이 경과한 후에 한 번만 실행된다

### 💡 구독자: Observer 인터페이스

- RxKotlin 1.x의 Subscriber는 RxKotlin 2.x에서 Observer로 변경
- RxKotlin 1.x에는 **Observer 인터페이스**가 있다
- **subscribe()** 메서드에 전달하는 것은 Subscriber이다
    - RxJava 2.x에서 **Subscriber**는 Flowables에 관해서만 존재
- **Observer**는 onNext(item : T), onError(error : Throwable), onComplete(), onSubscribe(d : Disposable)의 네 가지 메서드를 가지는 인터페이스이다
    - 옵저버블을 옵저버에 연결하면 이 네 가지 메서드가 호출
        - **onNext :** 아이템을 하나씩 넘겨주기 위해서 옵저버블은 옵저버의 이 메서드를 호출
        - **onComplete :** 옵저버블이 onNext를 통한 아이템 전달이 종료됐음을 알리고 싶을 때 옵저버의 onComplete를 호출
        - **onError :** 옵저버블에서 에러가 발생했을 때 옵저버에 정의된 로직이 있다면 onError를 호출하고 그렇지 않다면 예외를 발생
        - **onSubscriber :** Observable이 새로운 Observer를 구독할 때마다 호출

### 💡 구독과 해지

- **Observable :** 관찰해야 하는 대상
- **Observer :** 관찰해야 하는 주체
- **Subscribe 연산자는 Observable을 Observer에 연결하는 매개체**의 용도로 사용
- Subscribe 연산자에 대해 1개에서 3개의 메서드를 전달할 수 있다
    - Observer 인터페이스의 인스턴스를 연산자에 전달해 연결할 수도 있다

```kotlin
fun main(args: Array<String>) {
    val observable:Observable<Int> = Observable.range(1,5)//1

    observable.subscribe({//2
        //onNext method
        println("Next $it")
    },{
        //onError Method
        println("Error ${it.message}")
    },{
        //onComplete Method
        println("Done")
    })

    val observer: Observer<Int> = object : Observer<Int> {//3
        override fun onComplete() {
            println("All Completed")
        }

        override fun onNext(item: Int) {
            println("Next $item")
        }

        override fun onError(e: Throwable) {
            println("Error Occurred ${e.message}")
        }

        override fun onSubscribe(d: Disposable) {
            println("New Subscription ")
        }
    }

    observable.subscribe(observer)
}
```

### 💡 구독을 중지하는 방법

- **Disposable 활용**
    - subscribe메서드에..
        - **메서드를 전달한 경우 :** subscribe 연산자는 **Dispossable 인스턴스 반환**
        - **Observer 인스턴스를 전달한 경우 :** onSubscribe 메서드의 매개 변수에서 Disposable 인스턴스를 얻을 수 있음

```kotlin
fun main(args: Array<String>) {
    runBlocking {
        val observale:Observable<Long> = Observable.interval(100,TimeUnit.MILLISECONDS)//1
        val observer:Observer<Long> = object : Observer<Long> {

            lateinit var disposable:Disposable//2

            override fun onSubscribe(d: Disposable) {
                disposable = d//3
            }

            override fun onNext(item: Long) {
                println("Received $item")
                if(item>=10 && !disposable.isDisposed) {//4
                    disposable.dispose()//5
                    println("Disposed")
                }
            }

            override fun onError(e: Throwable) {
                println("Error ${e.message}")
            }

            override fun onComplete() {
                println("Complete")
            }

        }

        observale.subscribe(observer)
        delay(1500)//6
    }
}
```

- **interval로 생성한 옵저버블**은 프로그램이 실행을 멈출 때까지 중지되지 않는다

```java
/**
 * Represents a disposable resource.
 */
public interface Disposable {
    /**
     * Dispose the resource, the operation should be idempotent.
     */
    void dispose();

    /**
     * Returns true if this resource has been disposed.
     * @return true if this resource has been disposed
     */
    boolean isDisposed();
}
```

- **dispose 메서드 → 배출 중단을 알림**
- **isDisposed → 배출 중단을 전달받았는지 알리는 속성**

### 💡 콜드 옵저버블

- 동일한 옵저버블을 여러 번 구독해도, Observable의 모든 동일한 데이터 배출을 받을 수 있다

```kotlin
fun main(args: Array<String>) {
    val observable: Observable<String> = listOf("String 1","String 2","String 3","String 4").toObservable()//1

    observable.subscribe({//2
        println("Received $it")
    },{
        println("Error ${it.message}")
    },{
        println("Done")
    })

    observable.subscribe({//3
        println("Received $it")
    },{
        println("Error ${it.message}")
    },{
        println("Done")
    })

}
```

- 두 번의 구독 모두 똑같은 결과를 얻음
- 각 구독마다 처음부터 아이템을 배출하는 것을 **콜드 옵저버블**이라고 함
- 콜드 옵저버블은 구독 시에 실행을 시작하고, subscribe가 호출되면 아이템을 푸시하기 시작
- 각각의 구독 전부 동일한 순서로 아이템을 푸시함
- SQLite나 Room 데이터베이스 같은 경우 핫 옵저버블 보다 콜드 옵저버블에 더욱 많이 의존

### 💡 핫 옵저버블

- 핫 옵저버블은 콜드 옵저버블과 반대로, 배출을 시작하기 위해 구독할 필요가 없음
- 핫 옵저버블은 시청자가 시청하는지 여부에 관계없이 콘텐츠를 계속 브로드캐스팅(배출) 함
- 핫 옵저버블은 데이터보다는 이벤트와 유사
- 최근 가입한 Observer는 이전에 방출된 데이터를 놓칠 수 있음(시간에 민감한 특징을 가짐)
- UI 이벤트/Fake Server를 다룰 때 유용

### 💡 ConnectableObservable 객체의 소개

- **ConnectableObservable : 대표적인 핫 옵저버블 중 하나**
- ConnectableObservable은 옵저버블, 콜드 옵저버블을 핫 옵저버블로 바꿀 수 있다
- **connect 메서드**를 호출한 후에 활성화 됨(subscribe 호출로 배출 시작 X)
- **connect를 호출하기 전에 반드시 subscribe를 호출**해야 한다
- connect를 호출한 후 구독하는 모든 호출은 이전에 생성된 배출을 놓치게 됨

```kotlin
fun main(args: Array<String>) {
    val connectableObservable = listOf("String 1","String 2","String 3","String 4","String 5").toObservable()
            .publish()//1
    connectableObservable.
            subscribe({ println("Subscription 1: $it") })//2
    connectableObservable.map(String::reversed)//3
            .subscribe({ println("Subscription 2 $it")})//4
    connectableObservable.connect()//5

    connectableObservable.
            subscribe({ println("Subscription 3: $it") })//6 //Will never get called
}
```

- ConnectableObservable의 주요 목적은 한 옵저버블에 여러 개의 구독을 연결해 하나의 푸시에 대응할 수 있도록 하는 것
- 푸시를 반복하고 각 구독마다 따로 푸시를 보내는 콜드 옵저버블과는 상이
- ConnectableObservable은 connect 메서드 이전에 호출된 모든 **subscriptions(Observers)를 연결하고 모든 옵저버에 단일 푸시를 전달함**
- **publish** 연산자로 콜드 옵저버블을 **ConnectableObservable**로 변환할 수 있음
- **map 연산자 : 옵저버블 소스에서 배출된 각 항목에 선택한 함수를 적용하고 이런 함수 적용 결과를 배출하는 옵저버블을 반환**
- 각 배출은 각 옵저버에게 동시에 전달됨
    - 인터리브 방식으로 데이터를 처리
- 옵저버블에서 단 한 번의 배출로 모든 **Subscriptions, Observers에게 배출을 전달하는 메커니즘**을 **멀티캐스팅**이라고 한다
- **ConnectableObservable**이 핫 옵저버블이기 때문에 주석 6에서의 subscribe 호출은 어떤 배출도 수신하지 않는다
    - connect 이후에 일어난 모든 새로운 구독은 이전에 생성된 배출을 놓침

```kotlin
fun main(args: Array<String>) {
    val connectableObservable = Observable.interval(100,TimeUnit.MILLISECONDS)
            .publish()//1
    connectableObservable.
            subscribe({ println("Subscription 1: $it") })//2
    connectableObservable
            .subscribe({ println("Subscription 2 $it")})//3
    connectableObservable.connect()//4
    runBlocking { delay(500) }//5

    connectableObservable.
            subscribe({ println("Subscription 3: $it") })//6
    runBlocking { delay(500) }//7
}
```

- 500밀리초 지연 → 세 번째 구독은 5번째 배출을 받고 이전의 구독을 모두 놓침

### 💡 Subjects

- **Hot Observables를 구현하는 또 다른 좋은 방법 : Subject**
- **Subject**
    - **Observables - Observer의 조합(두 가지 모두의 공통된 동작을 갖고 있음)**
    - 핫 옵저버블과 마찬가지로 내부 **Observer** 목록을 유지하고 배출 시에 가입한 모든 옵저버에게 단일 푸시를 전달
    - 옵저버블이 가져야 하는 모든 연산자를 가짐
    - 옵저버와 마찬가지로 배출된 모든 값에 접근할 수 있음
    - Subject가 **완료(completed)/오류(errored)/구독 해지(unsubscribed)**된 후에는 재사용할 수 없음
    - onNext를 사용해 값을 Subject(Observer) 측에 전달하면 Observable에서 접근 가능하게 됨

```kotlin
fun main(args: Array<String>) {
    val observable = Observable.interval(100, TimeUnit.MILLISECONDS)//1
    val subject = PublishSubject.create<Long>()//2
    observable.subscribe(subject)//3
    subject.subscribe({//4
        println("Received $it")
    })
    runBlocking { delay(1100) }//5
}
```

- PublishSubject.create()로 Subject 생성
- **PublishSubject**
    - 여러 가지 Subject 유형 중 하나
    - **PublishSubject**는 구독 시점 이후 Observable 소스가 배출한 항목만 Observer에게 전달함
- Observable 인스턴스의 배출을 구독할 수 있음
- Subject의 배출에 접근하기 위해 람다를 사용해 구독
- delay 메서드는 자바의 sleep 메서드와 비슷
    - 단 delay 메서드는 Coroutine 컨텍스트 내에서 사용해야한다 → 그래서 여기서는 runBlocking 혼합하는 것
    - runBlocking은 모든 코드 실행이 완료될 때까지 스레드를 차단한다
        - 호출 스레드 내부에서 코루틴 컨텍스트를 모킹(Mocking)함
- Subject 인스턴스는 옵저버블 인스턴스에 의한 배출을 받고, 그 배출을 자신들의 Observers에게 브로드캐스팅함(TV 채널과 유사)

```kotlin
fun main(args: Array<String>) {
    val observable = Observable.interval(100, TimeUnit.MILLISECONDS)//1
    val subject = PublishSubject.create<Long>()//2
    observable.subscribe(subject)//3
    subject.subscribe({//4
        println("Subscription 1 Received $it")
    })
    runBlocking { delay(1100) }//5
    subject.subscribe({//6
        println("Subscription 2 Received $it")
    })
    runBlocking { delay(1100) }//7
}
```

- Subject는 모든 옵저버에게 전달된 배출을 중계하고, 콜드 옵저버블을 핫 옵저버블로 변경시킴

### 💡 다양한 구독자

- AsyncSubject
- PublishSubject
- BehaviorSubject
- ReplaySubject

[Subject · ReactiveX/RxJava Wiki](https://github.com/ReactiveX/RxJava/wiki/Subject)

### 💡 AsyncSubject 이해

- **AsyncSubject**는 수신 대기 중인 소스 옵저버블의 마지막 값과 배출만 전달함
    - AsyncSubject는 마지막 값을 한 번만 배출함

![https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/AsyncSubject.png](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/AsyncSubject.png)

```kotlin
fun main(args: Array<String>) {
    val observable = Observable.just(1,2,3,4)//1
    val subject = AsyncSubject.create<Int>()//2
    observable.subscribe(subject)//3
    subject.subscribe({//4
        //onNext
        println("Received $it")
    },{
        //onError
        it.printStackTrace()
    },{
        //onComplete
        println("Complete")
    })
    subject.onNext(5)//5
    subject.onComplete()//6
}
```

- Subject 인스턴스에서 옵저버블을 구독하지 않고 onNext 메서드로 직접 값을 전달할 수 있다
- onNext를 사용해 값을 전달
- Subject로 다른 옵저버블을 구독하거나, onNext로 값을 전달할 수 있음
    - 기본적으로 Subject로 옵저버블에 가입하면, Subject는 옵저버블이 값을 배출할 때마다 내부적으로 onNext를 호출함

```kotlin
fun main(args: Array<String>) {
    val subject = AsyncSubject.create<Int>()
    subject.onNext(1)
    subject.onNext(2)
    subject.onNext(3)
    subject.onNext(4)
    subject.subscribe({
        //onNext
        println("S1 Received $it")
    },{
        //onError
        it.printStackTrace()
    },{
        //onComplete
        println("S1 Complete")
    })
    subject.onNext(5)
    subject.subscribe({
        //onNext
        println("S2 Received $it")
    },{
        //onError
        it.printStackTrace()
    },{
        //onComplete
        println("S2 Complete")
    })
    subject.onComplete()//6
}
```

- onNext를 통해 모든 값을 전달
- 두 가지 구독 모두 마지막 값 5만 출력
- **ConnectableObservable VS AsyncSubject**
    - **ConnectableObservable : connect 호출 시 배출되기 시작**
    - **AsyncSubject : onComplete 호출에서만 유일한 값을 배출함**
- AsyncSubject는 인터리브 방식으로 작동하지 않음
    - 하나의 값을 사용해 여러 옵저버에 내보내는 작업을 반복함

### 💡 PublishSubject 이해

- **PublishSubject**는 onNext 메서드 또는 다른 구독을 통해 값을 받았는지 여부에 관계없이 구독 시점에 이어지는 모든 값을 배출함
- 가장 많이 사용되는 Subject 변형

![https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/PublishSubject.png](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/PublishSubject.png)

### 💡 BehaviorSubject 이해

- **BehaviorSubject**는 멀티캐스팅으로 동작함
- 구독 전의 마지막 아이템과 구독 후 모든 아이템을 배출함
    - AsyncSubject와 PublishSubject를 결합한 형태라고도 볼 수 있음
- 내부 옵저버 목록을 유지하는 데 중복 전달 없이 모든 옵저버에게 동일한 배출을 전달함

![https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/S.BehaviorSubject.v3.png](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/S.BehaviorSubject.v3.png)

```kotlin
fun main(args: Array<String>) {
    val subject = BehaviorSubject.create<Int>()
    subject.onNext(1)
    subject.onNext(2)
    subject.onNext(3)
    subject.onNext(4)
    subject.subscribe({
        //onNext
        println("S1 Received $it")
    },{
        //onError
        it.printStackTrace()
    },{
        //onComplete
        println("S1 Complete")
    })
    subject.onNext(5)
    subject.subscribe({
        //onNext
        println("S2 Received $it")
    },{
        //onError
        it.printStackTrace()
    },{
        //onComplete
        println("S2 Complete")
    })
    subject.onComplete()
}
```

- 첫 번째 구독은 4와 5를 받음
- 4는 구독하기 앞서, 5는 구독 이후 배출됨
- 두 번째 구독은 구독 전에 배출된 5만을 받음

### 💡 ReplaySubject 이해

- ReplaySubject는 갖고 있는 모든 아이템을 옵저버의 구독 시점과 상관없이 다시 전달함 → 콜드 옵저버블과 유사

![https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/ReplaySubject.u.png](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/ReplaySubject.u.png)

```kotlin
fun main(args: Array<String>) {
    val subject = ReplaySubject.create<Int>()
    subject.onNext(1)
    subject.onNext(2)
    subject.onNext(3)
    subject.onNext(4)
    subject.subscribe({
        //onNext
        println("S1 Received $it")
    },{
        //onError
        it.printStackTrace()
    },{
        //onComplete
        println("S1 Complete")
    })
    subject.onNext(5)
    subject.subscribe({
        //onNext
        println("S2 Received $it")
    },{
        //onError
        it.printStackTrace()
    },{
        //onComplete
        println("S2 Complete")
    })
    subject.onComplete()
}
```

- 두 가지 구독 모두 모든 배출을 받음