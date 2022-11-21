# 4장. 백프레셔와 플로어블 소개

`written by 문다빈`

> **코틀린 리액티브 프로그래밍** 저서를 읽고 작성하는 글입니다.
> 

## 4장. 백프레셔와 플로어블 소개

### 💡 백프레셔 이해

- **옵저버블의 문제사항 : 옵저버가 옵저버블의 속도에 대처할 수 없는 경우**
    - 옵저버블은 기본적으로 **아이템을 동기적으로 옵저버에서 하나씩 푸시해서 동작**
    - **Q : 옵저버블이 각 항목을 배출하는 간격보다 옵저버가 작업을 처리하는 속도가 느리면 어떻게 되는가?**
        
        ```kotlin
        fun main(args: Array<String>) {
            val observable = Observable.just(1,2,3,4,5,6,7,8,9)//(1)
            val subject = BehaviorSubject.create<Int>()
            subject.observeOn(Schedulers.computation())//(2)
                    .subscribe({//(3)
                        println("Subs 1 Received $it")
                        runBlocking{delay(200)}//(4)
        })
        
            subject.observeOn(Schedulers.computation())//(5)
                    .subscribe({//(6)
                        println("Subs 2 Received $it")
        })
            observable.subscribe(subject)//(7)
            runBlocking{delay(2000)}//(8)
        }
        ```
        
    - **A : 첫 번째 구독보다 두 번째 구독이 먼저 다 출력됨**
    - **observeOn**
        - **구독을 실행하는 스레드를 지정**
        - **Scheduler.computation()**은 계산을 수행할 스레드 제공
        - 실행은 백그라운드에서 수행(그래서 delay 메소드 사용해서 대기해서 느리게 받을거임)
    - 첫 번째 옵저버에서 각 계산이 오래 걸려 각 배출들이 대기열에 들어감(핫 옵저버블이 멈춘 것은 아님)
        - 이것은 **OutOfMemoryError 예외를 일으킬 수 있음(그 외 다른 문제도 생길 수 있음)**
        - 따라서 좋은 케이스는 아님
    
    ```kotlin
    fun main(args: Array<String>) {
        val observable = Observable.just(1,2,3,4,5,6,7,8,9)//(1)
        observable
                .map{MyItem(it)}//(2)
                .observeOn(Schedulers.computation())//(3)
                .subscribe({//(4)
                    println("Received $it")
                    runBlocking{delay(200)}//(5)
    })
        runBlocking{delay(2000)}//(6)
    }
    
    data class MyItem (val id:Int) {
        init {
            println("MyItem Created $id")
        }
    }
    ```
    
    - **map 연산자**
        - **원본 옵저버블을 입력받아 런타임에 해당 객체가 배출한 항목을 처리하고 관찰 가능한 또 다른 옵저버블을 생성하는 연산자**
        - map 연산자는 새로운 생성된 항목을 옵저버에게 전달하기 전에 **옵저버블에 의해 배출된 각 항목을 처리**한다
            - 이는 subscribe 이전에 발생(map 연산자의 처리)
    
    **➕ 더 알아보기..**
    
    - **Data Class :** 보유하는 변수에 대한 접근자(getter)를 제공하고, toString 메소드를 제공한다(그래서 print function은 호출 시 객체에 암묵적으로 toString을 추가해서 호출한다)
    - 객체의 생성은 매우 빨라 컨슈머로 알려진 옵저버가 인쇄를 시작하기도 전에 객체 생성이 완료된다
        - 여기서 문제는.. 배출이 대기열에 쌓이고 있는데 컨슈머는 이전의 배출을 처리하고 있다는 것(속도의 차이 발생)
    
    ❗ **문제의 해결책 : 컨슈머와 생산자(Producer)가 피드백을 주고 받을 수 있는 채널**
    
    - 채널을 통해 컨슈머는 생산자에게 이전 배출의 처리가 완료될 때까지 기다려야한다고 전달할 수 있음
    - 이렇게 할 경우 컨슈머 또는 메시지를 처리하는 미들웨어가 부하가 높은 상태에서 포화 상태가 되거나 응답하지 않는 것을 막을 수 있음
    - 메시지양을 줄이도록 요구 → 따라서 생산자는 생성 속도를 줄이도록 결정할 수 있음
    - 이 때의 피드백 채널 → **백프레셔**
    - 옵저버블과 옵저버는 백프레셔를 지원하지 않음 → **대체재 : 플로어블 & 구독자**
    

### 💡 플로어블

- **플로어블은 옵저버블의 백프레셔 버전**
- **플로어블과 옵저버블의 차이점 : 플로어블은 백프레셔를 고려함**
    - **but! 옵저버블은 가능하지 않음**
- 플로어블은 연산자를 위해 **최대 128개의 항목을 가질 수 있는 버퍼**를 제공
    
    → **따라서,** 컨슈머가 시간이 걸리는 작업을 실행할 때 배출된 항목이 버퍼에서 대기할 수 있다
    
- 플로어블은 ReactiveX 2.x에 추가되었다(RxKotlin도 마찬가지)
    - 이전 버전에는 해당 내용이 포함되어 있지 않음
    - 대신 이전 버전에서는 옵저버블이 예기치 못한 **MissingBackpressureException의 원인이 되는 백프레셔 지원**

```kotlin
fun main(args: Array<String>) {
   Observable.range(1,1000)//(1)
            .map{MyItem3(it)}//(2)
            .observeOn(Schedulers.computation())
            .subscribe({//(3)
                print("Received $it;\t")
                runBlocking{delay(50)}//(4)
},{it.printStackTrace()})
    runBlocking{delay(80000)}//(5)
}

data class MyItem3 (val id:Int) {
    init {
        print("MyItem Created $id;\t")
    }
}
```

- 옵저버블이 항목을 배출하는 속도를 옵저버가 따라가지 못함(옵저버블이 모든 항목을 배출할 동안 옵저버는 첫 번째 항목만 처리)
    - 이런 현상은 OOM 오류를 비롯해 많은 문제를 발생시킬 수 있음
    
    → **Flowable로 전환해서 해결해보자!**
    

```kotlin
fun main(args: Array<String>) {
   Flowable.range(1,1000)//(1)
            .map{MyItem4(it)}//(2)
            .observeOn(Schedulers.computation())
            .subscribe({//(3)
                print("Received $it;\t")
                runBlocking{delay(50)}//(4)
},{it.printStackTrace()})
    runBlocking{delay(70000)}//(5)
}

data class MyItem4 (val id:Int) {
    init {
        print("MyItem Created $id;\t")
    }
}
```

- 플로어블은 모든 아이템을 한 번에 배출하지 않고 컨슈머가 처리를 시작할 수 있을 때까지 기다렸다가 다시 배출을 전달
- 데이터를 다 방출할 때까지 이 동작을 반복함 → 이런 방식으로 많은 문제를 제거할 수 있음

### 💡 플로어블과 옵저버블 사용 구분

- 플로어블이 옵저버블보다 항상 나은 것은 아니다
- 플로어블은 백프레셔 전략을 제공(but 옵저버블도 쓸모가있음)

### 💡 플로어블을 언제 사용할까

- 플로어블은 옵저버블보다 느리다
- 플로어블과 백프레셔는 많은 양의 데이터를 처리할 때 도움이 된다
    - 데이터 소스에서 **10000개 이상의 아이템을 배출**한다면 플로어블을 사용하는 것을 추천!(그냥 대용량이라고 생각하면 편할듯)
- 데이터 소스에 비동기 적인 처리가 필요할 경우
    - 컨슈머 체인이 생산자에게 배출량 제한/규제를 요청할 수 있는 경우에 적합하다(백프레셔를 지원해서 적합하다는 의미!)
- 파일이나 데이터베이스를 읽거나 파싱하는 경우
- 결과를 반환하는 동안 IO 소스의 양을 조절할 수 있는 블로킹을 지원하는 네트워크 IO 작업
- 스트리밍 API

### 💡 옵저버블을 언제 사용할까

- 옵저버블이 선호되는 조건
    - 소량의 데이터(10000개 미만의 배출)를 다룰 때
    - 동기 방식으로 작업하길 원하거나 제한된 동시성을 가진 작업을 수행할때
    - UI 이벤트를 발생시킬 때

→ **플로어블은 옵저버블보다 느리다**

### 💡 플로어블과 구독자

- 플로어블은 옵저버 대신 백프레셔 호환이 가능한 구독자를 사용(람다식으로 표현하면 옵저버블과 차이점이 거의 없어 보이긴함)
- **옵저버 대신 구독자(Subscriber)를 사용해야하는 이유 : 구독자(Subscriber)가 일부 추가 기능과 백프레셔를 동시에 지원하기 때문**
- 얼마나 많은 아이템을 받기를 원하는지 메시지로 전달 가능
- 구독자(Subscriber)를 사용하는 동안은 업스트림에서 수신(요청)하고자 하는 항목의 수를 지정하도록 할 수 있음
- 아무 값도 지정하지 않으면 어떤 배출도 수신하지 못함
- 람다를 사용한 구독자는 옵저버와 유사 → 이 구현은 자동으로 업스트림으로부터 제한 없는 배출을 요구
- 원하는 배출량을 지정하지 않았지만 내부적으로 배출량을 무제한으로 요청해서 배출된 모든 아이템을 수신할 수 있음

```kotlin
fun main(args: Array<String>) {
    Flowable.range(1, 1000)//(1)
            .map{MyItem5(it)}//(2)
            .observeOn(Schedulers.io())
            .subscribe(object : Subscriber<MyItem5> {//(3)
                override fun onSubscribe(subscription: Subscription) {
                    subscription.request(Long.MAX_VALUE)//(4)
                }

                override fun onNext(s: MyItem5?) {
                    runBlocking{delay(50)}
println("Subscriber received " + s!!)
                }

                override fun onError(e: Throwable) {
                    e.printStackTrace()
                }

                override fun onComplete() {
                    println("Done!")
                }
            })
    runBlocking{delay(60000)}
}

data class MyItem5 (val id:Int) {
    init {
        println("MyItem Created $id")
    }
}
```

- Subscriber 인스턴스를 활용해 작성한 코드
- Subscriber의 메서드는 옵저버와 동일
- subscribe 메서드를 사용할 때 초기에 원하는 배출량과 함께 호출해야 한다
- 전체 배출을 받기 원해서 여기서는 `Long.MAX_VALUE`와 함께 요청
- request 메서드는 어떻게 작동? → Subscriber가 호출되고 나서 업스트림에서 대기해야 하는 배출량을 요청
- Subscriber가 더 요청할 때까지 요청 이후 그 이상의 모든 배출은 무시

```kotlin
fun main(args: Array<String>) {
    Flowable.range(1, 15)
            .map{MyItem6(it)}
.observeOn(Schedulers.io())
            .subscribe(object : Subscriber<MyItem6> {
                lateinit var subscription: Subscription//(1)
                override fun onSubscribe(subscription: Subscription) {
                    this.subscription = subscription
                    subscription.request(5)//(2)
                }

                override fun onNext(s: MyItem6?) {
                    runBlocking{delay(50)}
println("Subscriber received " + s!!)
                    if(s.id == 5) {//(3)
                        println("Requesting two more")
                        subscription.request(2)//(4)
                    }
                }

                override fun onError(e: Throwable) {
                    e.printStackTrace()
                }

                override fun onComplete() {
                    println("Done!")
                }
            })
    runBlocking{delay(10000)}
}

data class MyItem6 (val id:Int) {
    init {
        println("MyItem Created $id")
    }
}
```

- 위 코드의 경우 `s.id == 5` 일때, 2개를 더 요청한다
- 따라서 1~15까지의 숫자 대신 7개의 아이템을 인쇄한다

❗ request 메서드는 업스트림의 끝까지 전달되지 않음

- request 메서드는 가장 근방에 있는 연산자로 전달
- 해당 연산자가 정보를 추가 업스트림으로 전달할 것인지 여부를 결정

### 💡 처음부터 플로어블 생성하기

- Flowable.create

```kotlin
fun main(args: Array<String>) {
    val observer: Observer<Int> = object : Observer<Int> {
        override fun onComplete() {
            println("All Completed")
        }

        override fun onNext(item: Int) {
            println("Next $item")
        }

        override fun onError(e: Throwable) {
            println("Error Occured ${e.message}")
        }

        override fun onSubscribe(d: Disposable) {
            println("New Subscription ")
        }
    }//Create Observer

    val observable: Observable<Int> = Observable.create<Int> {//1
        for(i in 1..10) {
            it.onNext(i)
        }
        it.onComplete()
    }

    observable.subscribe(observer)

}
```

- Observable.create 연산자로 옵저버블을 생성
- 이 연산자는 사용자 정의 옵저버블을 선언함
- 옵저버블에서 아이템을 배출하는 자체 규칙을 작성할 수 있음
- 옵저버블은 백프레셔를 지원하지 않는다

```kotlin
fun main(args: Array<String>) {
    val subscriber: Subscriber<Int> = object : Subscriber<Int> {
        override fun onComplete() {
            println("All Completed")
        }

        override fun onNext(item: Int) {
            println("Next $item")
        }

        override fun onError(e: Throwable) {
            println("Error Occured ${e.message}")
        }

        override fun onSubscribe(subscription: Subscription) {
            println("New Subscription ")
            subscription.request(10)
        }
    }//(1)

    val flowable: Flowable<Int> = Flowable.create<Int> ({//1
        for(i in 1..10) {
            it.onNext(i)
        }
        it.onComplete()
    },BackpressureStrategy.BUFFER)//(2)

    flowable
            .observeOn(Schedulers.io())
            .subscribe(subscriber)//(3)

    runBlocking { delay(10000) }

}
```

- **Flowable.create** 메서드로 플로어블의 인스턴스를 생성
- **Flowable.create** 메서드에 인자로 `BackpressureStrategy.BUFFER` 전달
- **Flowable.create**는 플로어블의 인스턴스를 생성하기 위해 두 개의 매개 변수를 사용함

```java
public static <T> Flowable<T> create(FlowableOnSubscribe<T> source, BackpressureStrategy mode) {
    ObjectHelper.requireNonNull(source, "source is null");
    ObjectHelper.requireNonNull(mode, "mode is null");
    return RxJavaPlugins.onAssembly(new FlowableCreate<T>(source, mode));
}
```

- **Flowable.create** 내부 코드
    - 첫 번째 매개 변수는 **배출의 원천이 되는 곳(데이터 소스)**
    - 두 번째 매개 변수는 **BackpressureStrategy**
- **열거형으로 작성 가능**하며, 다운스트림이 따라잡을 수 없는 배출이 생길 경우 캐싱/버퍼링 또는 삭제 등 다양한 백프레셔 전략을 설정하여 해당 문제를 해결할 수 있게 도와준다
- Flowable.generate를 사용해 백프레셔를 구현하는 더 강력한 방법이 있음
- 열거형 **BackpressureStrategy는 백프레셔를 위한 다섯 가지 기본 옵션을 제공한다(BackpressureStrategyenum)**
    - `BackpressureStrategy.MISSING` **: 백프레셔 구현을 사용하지 않으며 다운스트림이 스스로 오버 플로우를 처리해야 하는 전략**
        - 이 옵션은 onBackPressureXXX() 연산자를 사용할 때 유용
    - `BackpressureStrategy.ERROR` **: 이 전략은 어떤 백프레셔로도 구현하지 않는데 다운스트림이 소스를 따라잡을 수 없는 경우, MissingBackpressureException 예외를 발생시킴**
    - `BackpressureStrategy.BUFFER` **: 이 전략은 다운스트림이 배출을 소비할 수 있게 될 때까지 제한이 없는 버퍼에 저장한다. 버퍼 크기를 넘어서는 경우 OutOfMemoryError가 발생할 수 있다.**
        - 다운스트림에서 소비될 때까지 모든 배출을 버퍼에 저장한다
        - 백프레셔의 최적화된 구현이 아님
        - 너무 많은 배출을 처리하는 동안 OutOfMemoryError를 유발할 수 있음
        - 하지만 MissingBackPressureException을 방지하고 사용자 정의 Flowable을 작은 규모로 구동시킬 수 있음
    - `BackpressureStrategy.DROP` **: 다운스트림이 바쁘고 소비 속도를 계속 유지할 수 없을 때 모든 배출량을 무시하는 전략. 다운 스트림이 이전 작업을 끝내고 나서 처음으로 배출된 것을 처리하고 그 사이의 값들은 모두 생략된다.**
    - `BackpressureStrategy.LATEST` **: 다운스트림이 바쁘고 배출을 유지할 수 없는 경우 최신 배출량만 유지하고 나머지는 모두 무시한다.**
    
    → 연산자로 해당 전략들을 구현해보자!
    
    ### 💡 옵저버블로 플로어블 만들기
    
    - **Observable.toFlowable 연산자는 백프레셔를 지원하지 않는 원천(데이터 소스)에서 BackpressureStrategy를 구현하는 방법을 제공한다. 이 연산자는 Observable을 Flowable로 바꿔준다.**
        
        ```kotlin
        fun main(args: Array<String>) {
            val source = Observable.range(1, 1000)
            source.toFlowable(BackpressureStrategy.BUFFER)
                    .map { MyItem7(it) }
                    .observeOn(Schedulers.computation())
                    .subscribe{
                        print("Rec. $it;\t")
                        runBlocking { delay(600) }
                    }
            runBlocking { delay(700000) }
        }
        
        data class MyItem7 (val id:Int) {
            init {
                print("MyItem7 init $id;\t")
            }
        }
        ```
        
        - 다운스트림이 소비될 때까지 **BackpressureStrategy.BUFFER가 모든 배출량을 버퍼링**함
        
        ```kotlin
        fun main(args: Array<String>) {
            val source = Observable.range(1, 1000)
            source.toFlowable(BackpressureStrategy.ERROR)
                    .map { MyItem7(it) }
                    .observeOn(Schedulers.computation())
                    .subscribe{
                        print("Rec. $it;\t")
                        runBlocking { delay(600) }
                    }
            runBlocking { delay(700000) }
        }
        
        data class MyItem7 (val id:Int) {
            init {
                print("MyItem7 init $id;\t")
            }
        }
        ```
        
        - **BackpressureStrategy.ERROR**로 설정할 경우 다운스트림이 업스트림을 따라갈 수 없어 오류가 발생함
        
        ```kotlin
        fun main(args: Array<String>) {
            val source = Observable.range(1, 1000)
            source.toFlowable(BackpressureStrategy.DROP)
                    .map { MyItem8(it) }
                    .observeOn(Schedulers.computation())
                    .subscribe{
                        print("Rec $it;\t")
                        runBlocking { delay(100) }
                    }
            runBlocking { delay(70000) }
        }
        
        data class MyItem8 (val id:Int) {
            init {
                print("init $id;\t")
            }
        }
        ```
        
        - **BackpressureStrategy.DROP**으로 설정할 경우 플로어블이 128 이후에 출력되지 않음
        
        ### 💡 BackpressureStrategy.MISSING와 onBackpressureXXX()
        
        - **BackpressureStrategy.MISSING**은 backpressure 전략을 구현하지 않으므로 플로어블에게 어떤 전략을 따를지 명시적으로 알려줄 필요가 있음
        - **onBackpressureXXX()**
            - **onBackpressureBuffer()**
            - **onBackpressureDrop()**
            - **onBackpressureLatest()**
        
        ### 💡 onBackpressureBuffer() 연산자
        
        - 이 연산자는 **BackpressureStrategy.BUFFER의 용도**로 사용
        - 버퍼 크기, 크기 제한 여부와 같은 몇 가지 추가 구성 옵션을 얻을 수 있다.
        - 기본 동작은 구성을 생략해도 동작
        
        ```kotlin
        fun main(args: Array<String>) {
            val source = Observable.range(1, 1000)
            source.toFlowable(BackpressureStrategy.MISSING)//(1)
                    .onBackpressureBuffer()//(2)
                    .map { MyItem11(it) }
                    .observeOn(Schedulers.io())
                    .subscribe{
                        println(it)
                        runBlocking { delay(100) }
                    }
            runBlocking { delay(600000) }
        }
        
        data class MyItem11 (val id:Int) {
            init {
                println("MyItem Created $id")
            }
        }
        ```
        
        - **BackpressureStrategy.MISSING** 옵션을 사용해 플로어블 인스턴스 생성
        - 백프레셔를 지원하기 위해 onBackpressureBuffer를 사용
        - 출력은 **BackpressureStrategy.BUFFER** 예제의 결과와 유사
        - **onBackpressureBuffer()**를 사용해 버퍼 크기를 지정할 수 있다
        - **onError 메서드를 구현**하면 플로어블이 훨씬 더 큰 크기를 필요로 해도 예외처리 가능
        
        ### 💡 onBackpressureDrop() 연산자
        
        - **onBackpressureBuffer**가 **BackpressureStrategy.BUFFER**와 일치
        - **onBackpressureDrop**은 Backpressure 전략 측면에서 **BackpressureStrategy.DROP**과 일치
        
        ```kotlin
        fun main(args: Array<String>) {
            val source = Observable.range(1, 1000)
            source.toFlowable(BackpressureStrategy.MISSING)//(1)
                    .onBackpressureDrop{ print("Dropped $it;\t") }//(2)
                    .map { MyItem12(it) }
                    .observeOn(Schedulers.io())
                    .subscribe{
                        print("Rec. $it;\t")
                        runBlocking { delay(100) }
                    }
            runBlocking { delay(600000) }
        }
        
        data class MyItem12 (val id:Int) {
            init {
                print("MyItem init $id\t")
            }
        }
        ```
        
        - **onBackpressureDrop()** 연산자를 ****사용 → 컨슈머 인스턴스에 전달되는 구성 옵션을 제공
        - 처리가 거부된 배출량을 소비하므로, 추가 처리가 가능함
        - 해당 구성을 사용하고 람다를 넘겨줌 → 거부된 배출이 출력
        - 버퍼의 크기가 기본 128이라서 플로어블은 128 이후에 배출을 처리하지 못하고 있음
        - **onBackpressureDrop의 컨슈머 인스턴스**는 **Subscriber 인스턴스가 시작되기 전에도 처리를 완료**
        
        ### 💡 onBackpressureLatest() 연산자
        
        - 이 연산자는 **BackpressureStrategy.LATEST**와 똑같은 방식으로 동작함
        - 다운스트림이 바쁘고 배출을 따라 잡을 수 없을 때 최신 배출을 제외한 모든 배출을 무시함
        - 다운스트림이 처리하던 작업이 완료되면, 직전에 생성된 마지막 배출이 다음으로 전달됨
        - 해당 연산자의 경우 어떤 구성도 제공하지 않음
        
        ```kotlin
        fun main(args: Array<String>) {
            val source = Observable.range(1, 1000)
            source.toFlowable(BackpressureStrategy.MISSING)//(1)
                    .onBackpressureLatest()//(2)
                    .map { MyItem13(it) }
                    .observeOn(Schedulers.io())
                    .subscribe{
                        print("-> $it;\t")
                        runBlocking { delay(100) }
                    }
            runBlocking { delay(600000) }
        }
        
        data class MyItem13 (val id:Int) {
            init {
                print("init $id;\t")
            }
        }
        ```
        
        - 플로어블은 128 이후에 모든 배출을 무시 → 단 **onBackpressureLatest**로 인해 마지막 배출인 1000은 유지
    
    ### 💡 원천에서 백프레셔를 지원하는 플로어블 생성
    
    - 데이터 소스(원천)에서부터 백프레셔를 지원하는 법 → **Flowable.generate()**
    - 백프레셔를 고려할 때 **Flowable.fromIterable()**을 사용함
        - 데이터 소스(원천)을 Iterator로 변환할 수 있을 때마다 **Flowable.fromIterable()**을 사용하고 **Flowable.generate()**는 좀 더 복잡한 설정이 필요한 상황에서 사용
    
    ```kotlin
    fun main(args: Array<String>) {
        val flowable = Flowable.generate<Int> {
            it.onNext(GenerateFlowableItem.item)
        }//(1)
    
        flowable
                .map { MyItemFlowable(it) }
                .observeOn(Schedulers.io())
                .subscribe {
                    runBlocking { delay(100) }
                    println("Next $it")
                }//(2)
    
        runBlocking { delay(700000) }
    }
    
    data class MyItemFlowable(val id:Int) {
        init {
            println("MyItemFlowable Created $id")
        }
    }
    
    object GenerateFlowableItem {
        var item:Int = 0//(3)
            get() {
                field+=1
                return field//(4)
            }
    }
    ```
    
    - **Flowable.generate() 메서드로 플로어블 생성**
    - 플로어블이 항목을 내보내고 구독자는 수신/대기/버퍼링/삭제하는 **Flowable.create()**
    - **Flowable.generate()**는 요청 시 아이템을 생성하고 이를 배출함
    - **Flowable.generate()**는 원천으로서 사용할 람다를 허용함
    - 이 람다는 Flowable.create와 유사하게 보일 수도 있음
    - **Flowable.create()와 달리 아이템을 요청할 때마다 이를 호출**
    - 람다 내부에서 onComplete 메서드를 호출하면 플로어블은 한 번만 배출함
    - 람다 안에서 onNext를 여러 번 호출할 수 없음
    - onError를 호출하면 첫 번째 호출에서 오류가 발생한다
    - 위 코드는 Flowable.range(1, Int.MAX_VALUE)와 유사하게 작동
    - 항목이 **Int.MAX_VALUE에 도달하면 onComplete를 호출하는 대신 Int.MIN_VALUE부터 다시 반복됨**
    - 출력에서 플로어블은 첫 번째로 128개의 항목을 배출한 다음, 다운스트림이 96개 아이템을 처리하기 위해 기다린 후 다시 Flowable이 128개의 아이템을 배출하는 주기가 계속됨
    - 플로어블에서 가입 해지 하거나 프로그램 실행이 중지될 때까지 항목을 계속 배출함
    
    ### 💡 ConnectableFlowable
    
    - **ConnectableFlowable** → ConnectableObservable과 대응
    - **ConnectableFlowable**은 플로어블과 유사하지만 구독 시점에 아이템 배출을 시작하지 않고 **connect() 메서드**가 호출될 때 시작
    - **Flowable**이 아이템을 배출하기 전에 의도한 모든 구독자가 **Flowable.subscribe()**를 기다리도록 할 수 있다
    
    ```kotlin
    fun main(args: Array<String>) {
        val connectableFlowable = listOf("String 1","String 2","String 3","String 4","String 5").toFlowable()
                .publish()
        connectableFlowable.
                subscribe({
                    println("Subscription 1: $it")
                    runBlocking { delay(1000) }
                    println("Subscription 1 delay")
                })
        connectableFlowable
                .subscribe({ println("Subscription 2 $it")})
        connectableFlowable.connect()
    }
    ```
    
    - Observable과 마찬가지로 **Flowable.fromIterable() 대신 Iterable<T>.toFlowable()** 확장 함수를 사용할 수 있음
    - **Flowable.publish()는 일반 Flowable을 ConnectableFlowable로 변환**한다
    - **Iterable<T>.toFlowable() 확장 함수**를 사용해서 List에서 플로어블을 만듦
    - **Flowable.publish()** 연산자를 사용해 플로어블에서 **ConnectableFlowable**을 생성함
    - **Flowable.fromIterable()**을 사용하면, 플로어블이 모든 다운스트림이 처리되기를 기다렸다가 완료 후 다시 다음 아이템의 배출을 시작함
    - 이는 다운스트림으로 번갈아가면서 전달됨
    - **Iterable<T>.toFlowable()은 내부적으로 Flowable.fromIterable을 호출**함

### 💡 프로세서

- 프로세서는 **플로어블의 Subjects**에 해당
- 모든 **Subjects 유형은 백프레셔를 지원하는 프로세서 타입**이 있음
- **PublishProcessor**
    
    ```kotlin
    fun main(args: Array<String>) {
        val flowable = listOf("String 1","String 2","String 3","String 4","String 5").toFlowable()//(1)
    
        val processor = PublishProcessor.create<String>()//(2)
    
        processor.//(3)
                subscribe({
                    println("Subscription 1: $it")
                    runBlocking { delay(1000) }
                    println("Subscription 1 delay")
                })
        processor//(4)
                .subscribe({ println("Subscription 2 $it")})
    
        flowable.subscribe(processor)//(5)
    
    }
    ```
    
- **Iterable<T>.toFlowable() 메서드로 플로어블을 생성**
- **PublishProcessor.create() 메서드를 사용해 프로세서 인스턴스를 생성**
- 주석3과 4에서 프로세서 인스턴스를 구독
- 프로세서 인스턴스로 플로어블에 가입
- 프로세서는 구독자가 모두 완료될 때까지 다음 푸시를 대기

### 💡 버퍼, 스로틀링, 윈도우 연산자

- 백프레셔 → 컨슈머가 소비할 때까지 아이템의 배출을 미룸
- **Observable.interval/Flowable.interval**을 사용하면 원천(데이터 소스의 배출 속도를 느리게 할 수 없다
- 정지 간격은 아이템을 동시에 처리할 수 있게 해주는 일부 연산자가 될 수 있음
- 도움이 되는 연산자가 세 가지 있음
    - **buffer**
    - **throttle**
    - **window**

### 💡 buffer() 연산자

- **컨슈머가 소비할 때까지 배출을 버퍼링하는 onBackPressureBuffer() 연산자와는 달리, buffer() 연산자는 배출을 모아서 리스트나 다른 컬렉션 유형으로 전달**함
    
    ```kotlin
    fun main(args: Array<String>) {
        val flowable = Flowable.range(1,111)//(1)
        flowable.buffer(10)
                .subscribe { println(it) }
    }
    ```
    
    - 버퍼 연산자로 10을 버퍼 크기로 사용해서 버퍼 연산자는 **Flowable에서 10개의 아이템을 모아서 리스트로 전달**
    - 버퍼 연산자에는 `skip` 변수와 같은 설정 옵션이 있음
    - buffer의 두 번째 매개 변수에 정수로 건너뛰는 값을 지정해서 사용
    - skip 매개 변수의 값이 count 매개 변수와 일치하면 아무것도 수행하지 않음
    - 그렇지 않으면 count 및 skip 매개 변수 사이의 양수 차이를 `actual_numbers_to_skip`으로 계산한 다음 skip 매개 변수의 값이 count 매개 변수의 값보다 크면 각 배출의 마지막 아이템 이후에 `actual_numbers_to_skip` 크기만큼의 아이템을 건너뛴다
    - 그렇지 않고 count 매개 변수의 값이 skip 매개 변수의 값보다 큰 경우 롤링 버퍼가 발생하는데 아이템을 건너뛰는 대신 이전 배출량에서 건너뛴다
    
    ```kotlin
    fun main(args: Array<String>) {
        val flowable = Flowable.range(1,111)
        flowable.buffer(10,15)//(1)
                .subscribe { println("Subscription 1 $it") }
    
        flowable.buffer(15,7)//(2)
                .subscribe { println("Subscription 2 $it") }
    }
    ```
    
    - 첫 번째 구독에 count 10, skip 15 버퍼를 사용
    - 주석 (2)의 두 번째 구독에 대해 count 15, skip 8을 사용
    - 첫 번째 구독의 경우 구독당 5개 항목을 건너뜀(15-10)
    - 두 번째 경우에는 각 배출에서 8번째 항목 (15-7)부터 반복을 시작함
    - buffer 연산자는 시간 기반으로 버퍼링을 할 수 있도록 도와줌
    - 원천으로부터 아이템을 모아서 일정 시간 간격으로 배출할 수 있음
    
    ```kotlin
    fun main(args: Array<String>) {
        val flowable = Flowable.interval(100, TimeUnit.MILLISECONDS)//(1)
        flowable.buffer(1,TimeUnit.SECONDS)//(2)
                .subscribe { println(it) }
    
        runBlocking { delay(5, TimeUnit.SECONDS) }//(3)
    }
    ```
    
    - **buffer(timespan: Long, unit: TimeUnit)**를 사용해 연산자가 모든 배출을 잠시 동안 버퍼링하고 목록으로 배출하도록 지시
    - 출력에서 알 수 있듯이 각 배출에는 10개의 아이템이 포함돼 있음
    - **Flowable.interval()**이 100밀리초마다 하나씩 배출하고 버퍼가 두 번째 시간 프레임(1초는 1000밀리초, 즉 100밀리초 간격의 배출은 1초에 10개의 배출을 의미)내에서 배출을 수집하기 때문
    - 버퍼 연산자의 또 다른 흥미로운 특징은 다른 생산자를 경계로 취할 수 있음
    - 버퍼 연산자는 인접해 있는 생산자의 사이에서 모든 배출물을 모으고 각 생산자의 리스트로 배출함
    
    ```kotlin
    fun main(args: Array<String>) {
        val boundaryFlowable = Flowable.interval(350, TimeUnit.MILLISECONDS)
    
        val flowable = Flowable.interval(100, TimeUnit.MILLISECONDS)//(1)
        flowable.buffer(boundaryFlowable)//(2)
                .subscribe { println(it) }
    
        runBlocking { delay(5, TimeUnit.SECONDS) }//(3)
    
    }
    ```
    
    - **Buffer 연산자는 boundaryFlowable이 배출할 때마다 수집해서 목록으로 배출**함

### 💡 window() 연산자

- **window()** 연산자는 아이템을 컬렉션 형태로 버퍼링하는 대신 다른 프로듀서 형태로 버퍼링한다

```kotlin
fun main(args: Array<String>) {
    val flowable = Flowable.range(1,111)//(1)
    flowable.window(10)
            .subscribe {
                flo->flo.subscribe {
                    print("$it, ")
                }
                println()
            }
}
```

- **window 연산자는 새로운 플로어블 인스턴스로 10개의 배출을 버퍼링**함
- 이 인스턴스는 **flowable.subscribe 람다에서 다시 구독하고 쉼표를 접미사로 덧붙여 인쇄**한다
- **window 연산자는 버퍼 연산자의 다른 오버로드와 동일한 기능**을 가짐

### 💡 throttle() 연산자

- **buffer()와 window()** 연산자는 배출을 수집함
- **스로틀 연산자는 배출을 생략**함

```kotlin
fun main(args: Array<String>) {
    val flowable = Flowable.interval(100, TimeUnit.MILLISECONDS)//(1)
    flowable.throttleFirst(200,TimeUnit.MILLISECONDS)//(2)
            .subscribe { println(it) }

    runBlocking { delay(1,TimeUnit.SECONDS) }
}
```

- **throttleFirst는 200밀리초마다 발생하는 첫 번째** **배출을 건너뜀**
- **throttleLast 및 throttleWithTimeout 연산자도 있음**