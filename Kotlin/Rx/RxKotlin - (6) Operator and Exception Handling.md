# 6장. 연산자 및 오류 처리

`written by 문다빈`

> **코틀린 리액티브 프로그래밍** 저서를 읽고 작성하는 글입니다.
> 

## 6장. 연산자 및 오류 처리

### 💡 프로듀서(옵저버블/플로어블) 결합

- 애플리케이션을 개발하면서 데이터를 사용하기 앞서 여러 원천의 데이터를 결합하는 일은 일반적이다
    - **ex. 오프라인 우선 방식에 따라 오프라인 애플리케이션을 구축하고 HTTP 호출에서 가져온 결과를 로컬 데이터베이스의 데이터와 결합하려는 상황**
    - startWith()
    - merge(), mergeDelayError(), concat()
    - zip()
    - combineLatest
- 기본적으로 프로듀서(옵저버블/플로어블)를 결합하는 몇 가지 메커니즘이 있다
    - 프로듀서 병합(Merging)
    - 프로듀서 이어 붙이기(Concatenating)
    - 프로듀서 사이의 임의 결합
    - 집핑
    - 가장 최근 항목 결합

### 💡 startWith 연산자

- startWith 연산자를 사용해 여러 프로듀서를 결합할 수도 있다

```kotlin
fun main(args: Array<String>) {
    println("startWith Iterator")
    Observable.range(5,10)
            .startWith(listOf(1,2,3,4))//(1)
            .subscribe {
                println("Received $it")
            }
    println("startWith another source Producer")
    Observable.range(5,10)
            .startWith(Observable.just(1,2,3,4))//(2)
            .subscribe {
                println("Received $it")
            }
}
```

- 다른 원천이 되는 옵저버블이나 Iterator 인스턴스를 전달해 연산자가 구독하기 시작한 옵저버블이 배출하기 전에 추가할 수 있다
- startWith 연산자는 전달된 Iterator 인스턴스를 Observable 인스턴스로 내부 변환한다(Flowable을 사용하는 경우 이를 Flowable 인스턴스로 변환)

```kotlin
public final Observable<T> startWith(ObservableSource<? extends T> other) {
        ObjectHelper.requireNonNull(other, "other is null");
        return concatArray(other, this);
}
```

- concatArray를 내부적으로 사용하고 있음

### 💡 배출을 집핑하기: zip 연산자

- 여러 개의 옵저버블이나 플로어블을 사용하고 있는 상황에서 각 프로듀서에서 발생하는 배출에 특정 연산을 적용해야 하는 경우
- zip 연산자를 사용하면 목적을 달성할 수 있다
- 여러 생산자의 배출을 통합해서 지정된 함수를 거치게 해 새로운 배출물을 생성할 수 있다

![https://reactivex.io/documentation/operators/images/zip.o.png](https://reactivex.io/documentation/operators/images/zip.o.png)

- zip 연산자는 여러 프로듀서의 배출을 하나의 배출로 누적시킨다
- 스캔 또는 리듀스 연산자의 배출에 적용되는 함수를 취하지만 다른 프로듀서의 배출에도 함수를 적용한다
- zip 연산자는 최대 9개의 Observables/Flowables를 지원함

```kotlin
fun main(args: Array<String>) {
    val observable1 = Observable.range(1,10)
    val observable2 = Observable.range(11,10)
    Observable.zip(observable1,observable2, io.reactivex.functions.BiFunction<Int, Int, Int> { emissionO1, emissionO2 ->
        emissionO1+emissionO2
    }).subscribe {
        println("Received $it")
    }
}
```

- zip 연산자는 옵저버블 클래스의 companion object에 정의돼 있으므로 다른 인스턴스를 통하지 않고 Observable.zip을 입력해 직접 액세스할 수 있다
- zip 연산자를 더 잘 이해하고 사용하려면 다음 사항에 유의하자
    - zip 연산자는 제공된 프로듀서가 배출될 때마다 작동한다
        - ex. 프로듀서 x, y, z를 전달했다면 x의 n번째 배출을 y와 z의 n번째 배출로 누적시킴
    - zip 운영자는 함수를 적용하기 전에 각 프로듀서가 배출할 때까지 대기한다
        - ex. **zip 연산자의 프로듀서로 Observable.interval을 사용하면 각 배출을 기다렸다가 지정된 간격으로 누적 값을 배출**한다
    - 어떤 프로듀서가 기다리는 아이템을 배출하지 않고 onComplete 또는 onError를 알리면 다른 프로듀서의 배출을 포함해 이후 모든 배출을 폐기한다
        
        > **ex.** 
        프로듀서 x → 10개의 아이템
        프로듀서 y → 11개의 아이템
        프로듀서 z → 8개의 아이템
        각각 배출 시 zip 연산자는 모든 프로듀서의 첫 8개의 배출을 누적하고 그 이후의 생산자 x와 y의 나머지 모든 배출을 폐기한다
        > 
        

### 💡 zipWith 연산자

- zip 연산자의 인스턴스 버전(인스턴스로 호출해야 하는 정적 함수의 복사본)은 zipWith이며 Observable 인스턴스 자체에서 호출할 수 있다
- zipWith의 문제점은 다른 원천 옵저버블만을 전달할 수 있다는 것
- 세 가지 이상의 옵저버블 인스턴스와 작업하고 싶다면 zipWith 대신 zip 연산자를 사용하는 것이 좋다

```kotlin
fun main(args: Array<String>) {
    val observable1 = Observable.range(1,10)
    val observable2 = listOf("String 1","String 2","String 3","String 4",
            "String 5","String 6","String 7","String 8","String 9","String 10").toObservable()

    observable1.zipWith(observable2,{e1:Int,e2:String -> "$e2 $e1"})//(1)
            .subscribe {
                println("Received $it")
            }
}
```

### 💡 combineLatest 연산자

- combineLatest 연산자는 제공된 프로듀서의 배출을 누적함
- zip 연산자와 비슷한 방식으로 동작
- 두 연산자의 차이점은 새로운 배출을 처리하기 전에 zip 연산자는 원천 생성자 각각이 배출하기를 기다리고, combineLatest 연산자는 원천 프로듀서에서 배출을 받자마자 처리를 시작하는 것

```kotlin
fun main(args: Array<String>) {
    val observable1 = Observable.interval(100,TimeUnit.MILLISECONDS)//(1)
    val observable2 = Observable.interval(250,TimeUnit.MILLISECONDS)//(2)

    Observable.zip(observable1,observable2,
            BiFunction { t1:Long, t2:Long -> "t1: $t1, t2: $t2" })//(3)
            .subscribe{
                println("Received $it")
            }

    runBlocking { delay(1100) }
}
```

- zip 연산 후 총 간격은 350밀리초가 됨 → 지연시간은 1,100밀리초이기 때문에 결과에서 3개의 배출을 확인할 수 있다

```kotlin
fun main(args: Array<String>) {
    val observable1 = Observable.interval(100, TimeUnit.MILLISECONDS)
    val observable2 = Observable.interval(250, TimeUnit.MILLISECONDS)

    Observable.combineLatest(observable1,observable2,
            BiFunction { t1:Long, t2:Long -> "t1: $t1, t2: $t2" })
            .subscribe{
                println("Received $it")
            }

    runBlocking { delay(1100) }
}
```

- combineLatest 연산자는 다른 모든 원천 프로듀서에 대해 마지막으로 생성된 값을 사용해 배출된 값을 즉시 처리하고 출력함

### 💡 옵저버블/플로어블 병합: merge 연산자

- 모든 원천 프로듀서들의 모든 배출에 가입하고 싶다면 → 서로 다른 두 개의 프로듀서가 있으며 이들을 구독할 때 적용해야 할 동일한 일련의 작업이 있을 때 → 모든 원천 프로듀서들의 모든 배출을 합쳐서 전부 구독해야함

```kotlin
fun main(args: Array<String>) {
    val observable1 = listOf("Kotlin", "Scala", "Groovy").toObservable()
    val observable2 = listOf("Python", "Java", "C++", "C").toObservable()

    Observable
            .merge(observable1,observable2)//(1)
            .subscribe {
                println("Received $it")
            }
}
```

- 옵저버블 두 개를 병합한 뒤 하나로서 구독한다
- 병합 연산자는 두 개의 옵저버블을 병합하고 두 Observable의 배출을 순서대로 위치시킴
- 그러나 병합 작업은 지정된 순서를 유지하지 않는다
    - 오히려 공급된 모든 프로듀서의 배출을 즉시 듣기 시작할 것이고 원천에서 배출되는 즉시 전달된다

```kotlin
fun main(args: Array<String>) {
    val observable1 = Observable.interval(500, TimeUnit.MILLISECONDS).map { "Observable 1 $it" }//(1)
    val observable2 = Observable.interval(100, TimeUnit.MILLISECONDS).map { "Observable 2 $it" }//(2)

    Observable
            .merge(observable1,observable2)
            .subscribe {
                println("Received $it")
            }

    runBlocking { delay(1500) }
}
```

- Observable.interval 연산자로 Observable <Long> 인스턴스를 두 개 만든 다음 Observable 문자열에 번호를 더해서 매핑하고 Observable<String> 인스턴스를 가져왔다
- observable1이 먼저 병합 연산자에 입력됐음에도 불구하고, observable2가 먼저 배출된다
- 병합 연산자는 최대 네 개의 매개변수를 지원한다
- 대비책으로 mergeArray 연산자도 있다 → 옵저버블 타입의 가변인자(vararg)를 인자로 받아들인다
- zip 연산자와 마찬가지로 병합 연산자도 정적 호출 이외에도 옵저버블의 인스턴스 함수 버전인 mergeWith를 호출할 수 있다
- 옵저버블 인스턴스를 두 개 만들고 observable1 인스턴스에서 호출된 mergeWith 연산자로 observable2를 병합함
- 모든 합병 연산자가 같은 일을 함
- 순서를 유지하려면 하나씩 이어 붙어야함

### 💡 프로듀서 이어 붙이기(옵저버블/플로어블)

- 연결(concatenating) 연산자는 병합 연산자와 거의 동일하지만 연결 연산자는 지정된 순서를 유지한다
- 제공된 모든 프로듀서를 한 번에 구독하는 대신 프로듀서를 차레로 구독한다
- 이전 구독에서 한 번만 onComplete를 받는다

```kotlin
fun main(args: Array<String>) {
    val observable1 = Observable.interval(500, TimeUnit.MILLISECONDS)
            .take(2)//(1)
            .map { "Observable 1 $it" }//(2)
    val observable2 = Observable.interval(100, TimeUnit.MILLISECONDS).map { "Observable 2 $it" }//(3)

    Observable
            .concat(observable1,observable2)
            .subscribe {
                println("Received $it")
            }

    runBlocking { delay(1500) }
}
```

- concat 연산자는 현재 원천 옵저버블에서 onComplete를 수신한 후에만 큐에 존재하는 다음 원천 옵저버블을 구독함
- Observable.interval로 생성된 옵저버블 인스턴스가 onComplete를 배출하지 않음
- Long.MAX_VALUE에 도달할 때까지 숫자를 계속 출력한다 → 따라서 take 연산자를 사용하도록 빠르게 수정
- onComplete 알림을 추가해 concat 연산자가 다음 원천 옵저버블의 값에 접근할 수 있도록 한다.
- concat 연산자가 첫 번째 옵저버블로부터 onComplete 알림을 받은 후에만 옵저버블로 제공되는 다음 원천에 구독 됨
- 병합 연산자와 마찬가지로 concat 연산자는 concatArray, concatWith와 같은 변형을 갖고 있음
- 병합이 아니라 이어 붙인다는 점 이외에는 거의 동일한 방식으로 동작

### 💡 프로듀서 임의 결합

- 두 개의 데이터 소스에서 데이터를 가져와 먼저 도착하는 것을 사용하고 나머지는 무시하려는 경우 → **RxKotlin에서는 amb 연산자**를 사용할 수 있음
- amb 연산자는 Observable(Iterable <Observable> 인스턴스)의 목록을 매개 변수로 사용한다
- Iterable 인스턴스에 있는 모든 옵저버블을 구독하고 첫 번째로 배출한 옵저버블로부터 수신한 아이템을 배출한 뒤 나머지 옵저버블의 결과는 전부 폐기한다

```kotlin
fun main(args: Array<String>) {
    val observable1 = Observable.interval(500, TimeUnit.MILLISECONDS).map { "Observable 1 $it" }//(1)
    val observable2 = Observable.interval(100, TimeUnit.MILLISECONDS).map { "Observable 2 $it" }//(2)

    Observable
            .amb(listOf(observable1,observable2))//(3)
            .subscribe {
                println("Received $it")
            }

    runBlocking { delay(1500) }
}
```

- observable2의 배출을 먼저 입력받았기 때문에 observable1의 배출을 무시하게 됨
- amb에는 ambArray, ambWith와 같은 변형도 있다

### 💡 그룹핑

- 그룹핑을 사용하면 특정 속성을 기준으로 배출을 분류할 수 있음
- **정수형의 Observable/Flowable이 있는데 짝수와 홀수에 따라 각각의 비즈니스 로직이 존재해 별도로 처리하려 한다고 했을 때 → 그룹화는 이런 시나리오에서 가장 좋은 솔루션**

```kotlin
fun main(args: Array<String>) {
    val observable = Observable.range(1,30)

    observable.groupBy {
        it%5
    }.blockingSubscribe {
        println("Key ${it.key} ")
        it.subscribe {
            println("Received $it")
        }
    }
}
```

- 5로 나눈 나머지를 기준으로 배출량을 그룹화함
- 기본적으로 5개의 그룹(0~4)이 존재
- groupBy 연산자를 사용해 그룹화를 위해 실행되는 프레디케이트(Prdeicate)를 전달
- groupBy 연산자는 프레디케이트의 결과를 통해 배출을 그룹화함
- **blockingSubscribe 연산자를 사용해 새로 생성된 Observable<GroupedObservable<K, T>> 인스턴스에 구독**
- subscribe를 사용하면 엉망으로 보일 것 → 다음 배출 전에 현재 배출에서 주어진 연산을 기다리지 않아서
- blockingSubscribe는 새로운 처리를 진행하기 전에 배출에서 처리가 완료될 때까지 프로그램을 대기 상태로 만듦
- groupBy 연산자는 그룹을 포함하는 GroupedObservable을 배출하는 옵저버블을 반환함
- blockingSubscribe 안에서는 배출된 GroupedObservable 인스턴스를 구독해야 함
- 배출된 GroupedObservable 인스턴스의 키를 인쇄한 후에도 동일한 작업을 수행

### 💡 flatMap concatMap 세부 사항

- flatMap은 내부적으로 merge 연산자를, concatMap은 concat 연산자를 사용함

```kotlin
fun main(args: Array<String>) {
    Observable.range(1,10)
            .flatMap {
                val randDelay = Random().nextInt(10)
                return@flatMap Observable.just(it)
                        .delay(randDelay.toLong(),TimeUnit.MILLISECONDS)//(1)
            }
            .blockingSubscribe {
                println("Received $it")
            }
}
```

- delay 연산자와 함께 flatMap 연산자를 사용해 임의의 지연을 배출에 추가
- 다운스트림이 규정된 순서대로 배출을 얻지 못함
- merge 연산자가 한 번에 모두 비동기적으로 배출을 구독하고 다시 배출하기 때문에 순서가 유지되지않음

```kotlin
fun main(args: Array<String>) {
    Observable.range(1,10)
            .concatMap {
                val randDelay = Random().nextInt(10)
                return@concatMap Observable.just(it)
                        .delay(randDelay.toLong(), TimeUnit.MILLISECONDS)//(1)
            }
            .blockingSubscribe {
                println("Received $it")
            }
}
```

- concatMap 연산자는 내부적으로 concat을 사용하기 때문에 규정된 배출 순서를 유지한다

### 💡 flatMap 연산자가 적합한 경우

1. 페이지나 액티비티 또는 프래그먼트에서 대이터의 리스트를 다룰 때 해당 리스트의 아이템별로 데이터베이스나 서버에 전송하고 싶을 경우 적합. 
    1. concatMap도 사용 가능하지만 flatMap 연산자는 비동기적으로 동작해서 순서가 중요하지 않은 경우 빠르게 동작함
2. 리스트의 아이템에 대한 작업을 비동기적으로 비교적 짧은 기간에 수행하려는 경우 적합

### 💡 concatMap 연산자가 적합한 경우

1. 사용자에게 표시할 데이터 리스트를 다운로드 할때(순서가 중요해서)
2. 정렬된 목록의 순서를 그대로 유지하고 작업하고 싶은 경우 적합

### 💡 switchMap 연산자 이해

- switchMap은 원천 프로듀서(Observable, Flowable)의 모든 배출을 비동기로 대기하지만 정해진 시간 이내의 최신 아이템만 배출한다
- 원천 옵저버블이 switchMap에서 이들 중 하나를 내보내기 이전에 하나 이상의 아이템을 연속적으로 배출하면 switchMap은 마지막 항목을 가져와서 그 사이에 들어온 모든 배출을 무시한다

```kotlin
fun main(args: Array<String>) {
    println("Without delay")
    Observable.range(1,10)
            .switchMap {
                val randDelay = Random().nextInt(10)
                return@switchMap Observable.just(it)//(1)
            }
            .blockingSubscribe {
                println("Received $it")
            }
    println("With delay")
    Observable.range(1,10)
            .switchMap {
                val randDelay = Random().nextInt(10)
                return@switchMap Observable.just(it)
                        .delay(randDelay.toLong(), TimeUnit.MILLISECONDS)//(2)
            }
            .blockingSubscribe {
                println("Received $it")
            }
}
```

- 두 번째 예시의 경우 switchMap이 배출 이전에 연속적으로 값을 전달받지만 마지막 아이템만을 배출한다
- 첫 번쨰의 경우 추가 배출을 받기 전에 모든 아이템을 다시 내보냄

```kotlin
fun main(args: Array<String>) {
    Observable.range(1,10)
            .switchMap {
                val randDelay = Random().nextInt(10)
                if(it%3 == 0)
                    Observable.just(it)
                else
                    Observable.just(it)
                            .delay(randDelay.toLong(), TimeUnit.MILLISECONDS)
            }
            .blockingSubscribe {
                println("Received $it")
            }
}
```

- 위 예제는 3으로 나눌 수 있는 모든 수는 그대로 배출하고 나머지는 지연을 추가함
- switchMap 연산자는 지연 없이 배출된 숫자들만 다시 배출시킴
- switchMap 연산자가 다음 아이템을 받기 이전에 그것들을 배출하는 것이 가능하기 때문

### 💡 배출 건너뛰거나 취하기

- 원하는 배출 몇 가지만 취하고 나머지는 건너뛰고 싶을 경우가 있음
- skip과 take 연산자는 이런 경우 사용할 수 있음

### 💡 배출 건너뛰기(skip, skipLast, skipUntil, skipWhile)

- 특정 조건이 충족된 경우 또는 무조건 선두의 배출 일부를 건너뛰고자 하는 경우가 있음
- 배출을 취하기 전에 다른 프로듀서들을 기다리면서 나머지는 모두 건너뛰어야 할 수도 있다
- RxKotlin은 스킵 연산자의 다양한 변형과 오버로드를 제공함
    - skip
    - skipLast
    - skipWhile
    - skipUntil
- skip 연산자에는 **skip(count : Long)과 skip(time : Long, unit : TimeUnit)의 두 가지 중요한 오버로드**가 있음
- 첫 번째 오버로드는 카운트를 입력받아 n개의 배출량을 무시하는데 두 번째 오버로드는 지정된 시간 동안 발생한 모든 배출을 무시함
- skipLast 연산자도 skip과 마찬가지로 많은 오버로드가 있음
- skip과의 차이점은 뒤에서부터 배출을 제거한다는 것
- 카운트 또는 시간을 기준으로 배출을 건너뛰는 skip, skipLast와는 다르게 skipWhile은 프레디케이트(논리 표현식)를 기반으로 건너뛴다
- 필터 연산자와 마찬가지로 프레디케이트를 skipWhile 연산자엥 전달함
- 프레디케이트가 참이라고 평가되면 배출을 건너뛰고, 거짓이라고 반환되는 순간부터 모든 배출을 다운스트림으로 전달하기 시작한다

❗filter 연산자와 달리 skipWhile 연산자는 프레디케이트가 false를 반환하는 이후의 모든 배출을 생략함(따라서 프레디케이트가 모든 배출을 확인하길 원하면 filter 연산자를 사용)

- **생산자1과 생산자2와 동시에 작업하는 상황 → 생산자 2가 배출을 시작하자마자 생산자 1의 배출을 처리하기를 원함 → skipUnitil**

### 💡 take 연산자(take, takeLast, takeWhile, takeUntil)

- take 연산자는 skip 연산자와 정확히 반대로 동작
- skip 연산자와 정반대로 take 연산자는 지정된 배출을 다운스트림으로 통과시키고 나머지 배출은 폐기한다
- 지정된 배출을 모두 보내고 나면 onComplete 통지도 전달
- **takeLast : 마지막 지정된 배출만 통과**
- **takeWhile : skipWhile과 정확히 반대. 프레디케이트 사용**

### 💡 onErrorReturn: 에러 발생 시 기본값 반환하기

- onErrorReturn은 업스트림에서 에러가 발생했을 때 다운스트림으로 전달할 수 있는 기본값을 지정할 수 있도록 함

```kotlin
fun main(args: Array<String>) {
    Observable.just(1,2,3,4,5)
            .map { it/(3-it) }
            .onErrorReturn { -1 }//(1)
            .subscribe {
                println("Received $it")
            }
}
```

- onErrorReturn 연산자를 사용해 에러가 발생했을 때 -1을 반환하도록 하고 있음
- 업스트림에서 에러가 발생해 아이템 배출을 정지하자 다운스트림은 더이상 어떤 배출도 받지 못하고 있음
- onError와 onComplete는 최종(터미널) 연산자이다
    - 따라서 둘 중 하나라도 수신하면 다운스트림은 즉시 대기를 중단함

### 💡 onErrorResumeNext 연산자

- 해당 연산자는 에러가 발생했을 시에 다른 프로듀서를 구독할 수 있도록 한다

```kotlin
fun main(args: Array<String>) {
    Observable.just(1,2,3,4,5)
            .map { it/(3-it) }
            .onErrorResumeNext(Observable.range(10,5))//(1)
            .subscribe {
                println("Received $it")
            }
}
```

- 에러 발생 시 다른 프로듀서를 구독하기 원할 때 유용

### 💡 에러 발생 시 재시도하기

- 에러가 발생했을 때 동일한 프로듀서에 연산을 재시도하거나 다시 구독할 수 있도록 해주는 에러 처리 연산자

```kotlin
fun main(args: Array<String>) {
    Observable.just(1,2,3,4,5)
            .map { it/(3-it) }
            .retry(3)//(1)
            .subscribeBy (
                    onNext  = {println("Received $it")},
                    onError = {println("Error")}
            )
    println("\n With Predicate \n")
    var retryCount = 0
    Observable.just(1,2,3,4,5)
            .map { it/(3-it) }
            .retry {//(2)
                _, _->
                (++retryCount)<3
            }
            .subscribeBy (
                    onNext  = {println("Received $it")},
                    onError = {println("Error")}
            )
}
```

- retry 연산자를 사용해 재시도를 세 번으로 제한하고 프레디케이트와 함께 retry 연산자를 사용
- retry 연산자는 프레디케이트가 참을 반환하는 한 계속해서 재시도하고 거짓을 반환하면 바로 다운스트림에 에러를 전달한다