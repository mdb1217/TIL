# 2장. 코틀린과 RxKotlin을 사용한 함수형 프로그래밍

`written by 문다빈`

> **코틀린 리액티브 프로그래밍** 저서를 읽고 작성하는 글입니다.
> 

## 2장. 코틀린과 RxKotlin을 사용한 함수형 프로그래밍

### ➕ 더 알아보기..

- **함수형 프로그래밍**
    - 선언적이고 표현적인 프로그래밍
    - 불변의 데이터를 사용하는 경향이 있다. → **불변성**
        - **불변성이란?** → 상태를 변경하지 않는 것 → 쉽게 얘기하면 함수 외부의 변수에 접근, 재할당 하는 것을 막는 것
        - **불변성을 통해 얻는 이점** → 무분별한 상태의 변경을 막는다(외부 참조를 안하는 걸 권장하니까..)
        - **상태의 변경을 추적하기 쉽다** → 불변성이 필연적으로 깨지게 되더라도, 함수를 타고타고 들어가서 상태의 변경을 추적하기 쉽다는 장점이 있음.
    - OOP(객체지향 프로그래밍)와는 조금 다른 개념
        
        ![https://evan-moon.github.io/static/def489f1400293515499d529e83b420a/e9140/oop_fp.png](https://evan-moon.github.io/static/def489f1400293515499d529e83b420a/e9140/oop_fp.png)
        
        OOP는 변경 가능한 상태를 감추며 단순함을 만들어내지만
        FP는 변경 가능한 상태를 없앰으로써 단순함을 만들어낸다
        
    - 코틀린은 대표적인 함수형 프로그래밍 언어 중 하나이다.

### 💡 함수형 프로그래밍 소개

- 함수형 프로그래밍은 프로그래밍 논리를 작고 재사용 가능하며 선언적인 순수한 기능의 조각으로 나눈다.
    - 논리를 작은 코드로 분산시키면 쉽게 모듈화가 가능하고 단순해진다. 따라서, 다른 모듈에 영향을 끼치지 않고 특정 코드의 일부 또는 모든 부분을 리팩토링하거나 변경할 수 있다. → **의존성을 낮출 수 있음**
- 자바의 경우 Java8부터 함수형 프로그래밍을 지원했지만, 코틀린은 첫번째 stable 릴리즈 버전부터 함수형 프로그래밍을 지원한다.
- 코틀린은 객체지향 및 함수형 프로그래밍 스타일 모두를 사용할 수 있다.
    - class 기반 → **객체지향**
    - 순수 함수, 람다, 고차함수 → **함수형**
- **함수형 리액티브 프로그래밍이란? 리액티브 프로그래밍과 함수형 프로그래밍을 혼합한 개념이다.**
- 함수형 프로그래밍의 주요 목적은 쉽게 모듈화가 가능한 프로그램을 구현하는 것이다.
- 모듈화된 프로그램은 **반응형 프로그래밍, 리액티브 선언문의 네 가지 원칙을 구현하는 데 필요**하다.

### 💡 함수형 프로그래밍의 기초

- 함수형 프로그래밍은 람다, 순수 함수, 고차 함수, 함수 유형, 인라인 함수 등의 새로운 개념들로 구성된다.

**➕ 더 알아보기..**

- **순수 함수와 람다**
    - **함수형 프로그래밍 패러다임에서의 람다는 순수 함수라 볼 수 있지만, 모든 람다함수가 순수 함수인 것은 아니다. → 함수형 프로그래밍 패러다임에서는 모든 함수가 순수 함수여야한다**❗❗ **람다만 특별한 게 아님..**

### 💡 람다 표현식

- 이름이 없는 익명함수
- 람다식은 함수라고 말할 수 있지만 모든 함수가 람다식인 것은 아니다.
- 모든 프로그래밍 언어가 람다식을 지원하는 것은 아니다.
- 자바는 Java8 이후부터 람다를 지원한다.
- 람다 표현식의 구현 방식은 언어에 따라 다르다.
- 코틀린에서는 람다를 구현하는 것이 매우 쉽고 자유로운 편이다.

```kotlin
fun main(args: Array<String>) {
    val sum = { x: Int, y: Int -> x + y } // (1) 순수 함수
    println("Sum ${sum(12,14)}")// (2)
    val anonymousMult = {x: Int -> (Random().nextInt(15)+1) * x}// (3) 순수 함수(x) 동일한 파라미터로 호출해도 값이 달라질 수 있어서!
    println("random output ${anonymousMult(2)}")// (4)
}
```

**💡 실행 결과**

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/0d0016f3-76ab-4784-a6da-cb060eec5da8/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221110T120636Z&X-Amz-Expires=86400&X-Amz-Signature=927bc8660be1ffe3ce311ba90e4e931d09a911668696e3909e4d4c27388f2eb1&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

💡 **알 수 있는 것**

- 자바의 경우 익명 클래스는 존재했지만, 람다/익명함수의 경우 Java8 이후에 생김

### 💡 순수 함수

- 함수의 반환값이 인수/매개 변수에 전적으로 의존하면 이 함수를 순수 함수라고 한다.
    - 순수함수와 순수함수가 아닌 것
    
    ```kotlin
    fun add(a,b): Int {
        return a + b
    }
    println(add(10,5))
    ```
    
    여기서 **add는 순수함수**이다.
    
    **→ why?** 언제, 어디서 실행해도 add(10,5)는 항상 15를 리턴하고 외부 상태를 변경하지 않았기 때문이다.
    
    ```kotlin
    var c = 10
    fun add2(a,b){
        return a + b + c
    }
    println(add2(10,3)) // 23 출력
    c = 20
    println(add2(10,3)) // 33 출력
    ```
    
    여기서 **add2는 순수함수가 아니다**.
    
    **→ why?** c라는 변수의 값이 변할 때마다 함수가 리턴하는 값이 달라지기 때문이다.**(SideEffect)**
    
    - **따라서..** c가 상수(변하지 않는 값)이면 해당 함수는 순수함수가 될 수 있다.
- 그래서 **fun func1(x : Int) : Int**를 선언하면, 그 반환값은 인수 x에 전적으로 의존한다.
- func1 함수에 x 파라미터에 동일한 값을 호출했을 때, func1이 언제나 같은 반환 값을 가지면 func1은 순수 함수이다.
- 순수 함수는 람다 또는 명명된 함수일 수도 있다. → 하지만 그 반대는 항상 참이 아니다.
    - **람다는 (항상) 순수 함수이다(x)**
    - **모든 함수는 순수 함수이다(x)**

```kotlin
fun square(n:Int):Int {//(1)
    return n*n
}

fun main(args: Array<String>) {
    println("named pure func square = ${com.rivuchk.packtpub.reactivekotlin.chapter02.square(3)}")
    val qube = {n:Int -> n*n*n}//(2)
    println("lambda pure func qube = ${qube(3)}")
}
```

**💡 실행 결과**

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/f35e232a-9acb-47e7-b299-becd0fd17cb7/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221110T120802Z&X-Amz-Expires=86400&X-Amz-Signature=baff47d21e9ed641461cb953a0c303c82f2ced63607f5550e4ee54d178faa2a2&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

💡 **알 수 있는 것**

- 순수 함수에는 부작용(Side Effect)이 없다.
    - **Side Effect : 함수 또는 표현식은 자신의 범위 외부의 일부 상태를 수정하거나 호출 함수 또는 외부 세계에 영향을 끼치는 상호 작용이 존재하는 경우 부작용이 있다고 한다.**
    - 외부에 영향을 주지 않으면서도, 외부에 영향을 받지 않는 함수가 사이드 이펙트가 없다❗
    

### 💡 고차 함수

- 함수를 인자로 받아들이거나 반환하는 함수를 고차 함수(high-order functions)라고 부른다.

```kotlin
fun highOrderFunc(a:Int, validityCheckFunc:(a:Int)->Boolean) {//(1)
    if(validityCheckFunc(a)) {//(2)
        println("a $a is Valid")
    } else {
        println("a $a is Invalid")
    }
}

fun main(args: Array<String>) {
    highOrderFunc(12,{ a:Int -> a.isEven()})//(3)
    highOrderFunc(19,{ a:Int -> a.isEven()})
}
```

**💡 실행 결과**

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a48dae8d-fbdd-42f6-b0b9-cc571df62a8d/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221110T120849Z&X-Amz-Expires=86400&X-Amz-Signature=dd0557bdb391c8445de428d1f3b04e1ea10e87960aa96bf93f6420dcc2ffc2eb&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

💡 **알 수 있는 것**

- highOrderFunc에서는 함수를 인자로 받아들인다.
- main함수에서는 highOrderFunc를 호출하면서 validityCheckFunc함수를 런타임으로 정의하고 있다.
    - 여기서 validaityCheckFunc의 경우 람다(익명함수)형태로 정의되어 있음.
    - `::` 를 활용해 함수도 직접 참조해서 넘기기 가능
        - **ex. highOrderFunc(num, ::exampleFunc)**

### 💡 인라인 함수

- 함수는 이식 가능한 코드를 작성하는 좋은 방법이지만 함수의 스택 유지 관리 및 오버 헤드로 인해 프로그램 실행 시간이 늘어나고 메모리 최적화를 저하시킬 수 있다. → **함수의 인자로 함수를 받으면 컴파일타임에 추가적인 객체 생성이 발생한다!(따라서 성능 저하..)**
- 함수를 인라인으로 선언하면 함수 호출이 함수 내부의 코드로 교체된다.(객체 생성 x)
- 함수 선언으로 얻는 자유(함수 내부 코드를 변경해도 외부까지 수정할 필요 없는 것 **→ ex. 덧셈 함수의 내부 내용을 곱셈 함수로 변경해서 넘겨도 형식만 바뀌지않으면 상관 x → 근데 이건 어찌보면 단점으로 해석될 수도 있을듯..**)와 성능 향상 모두를 꾀할 수 있음.
    - ****같이 보면 좋을 내용 : [함수를 파라미터로 넘길때 inline functions의 장점(내가 쓴 TIL ㅎㅎ)](https://github.com/mdb1217/TIL/blob/main/Kotlin/merit%20of%20inline%20function.md)**

```kotlin
inline fun doSomeStuff(a:Int = 0) = a+(a*a)

fun main(args: Array<String>) {
    for (i in 1..10) {
        println("$i Output ${doSomeStuff(i)}")
    }
}
```

**💡 실행 결과**

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/cbe4b60c-d96c-426a-9329-188ca40471cb/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221110T120943Z&X-Amz-Expires=86400&X-Amz-Signature=9bdd5232d0c60586f0c288952e11cd9d9c8d13862c3a29b702057d5409f0c95e&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

💡 **알 수 있는 것**

- **inline 키워드로 고차 함수를 선언하면 inline 키워드는 함수 자체와 전달된 람다에 모두 영향을 미친다.**

> 이미 JVM은 강력하게 inline을 지원하고 있고, JVM은 코드 실행 분석을 통해 가장 이익이 되는 방법으로 inline을 하고 있다.
> 

이미 JVM에서 inline(대표적인 예시는 스코프함수)을 하고 있으므로 사용하지 않아도 괜찮지만, 파라미터에 고차함수를 넘겨주는 형태이면 **inline키워드**가 효과적이다.

- **inline 예제**
    
    **1. 고차함수의 Runtime penalties**
    
    kotlin은 다음과 같이 함수를 인자로 전달하는 람다식을 정의할 수 있다.
    
    ```kotlin
    fun someMethod(a: Int, func: () -> Unit):Int {
        func()
        return 2*a
    }
    
    fun main(args: Array<String>) {
        var result = someMethod(2, {println("Just some dummy function")})
        println(result)
    }
    ```
    
    위의 코드를 다음과 같이 자바로 변환해보면, `someMethod` 메소드를 호출하기 위해 객체를 생성한다.
    
    ```java
    public final class InlineFunctions {
       public static final InlineFunctions INSTANCE;
    
       public final int someMethod(int a, @NotNull Function0 func) {
          func.invoke();
          return 2 * a;
       }
    
       @JvmStatic
       public static final void main(@NotNull String[] args) {
          int result = INSTANCE.someMethod(2, (Function0)null.INSTANCE);
          System.out.println(result);
       }
    
       static {
          InlineFunctions var0 = new InlineFunctions();
          INSTANCE = var0;
       }
    }
    ```
    
    내부적으로 객체 생성과 함수 호출을 하도록 구현이 되어있으며 이런 부분이 성능을 떨어뜨릴 수 있다.
    
    **2. inline functions 구현 및 동작 원리**
    
    함수 앞에 inline 키워드를 붙이면 **inline function**이 된다.
    
    ```kotlin
    inline fun someMethod(a: Int, func: () -> Unit):Int {
        func()
        return 2*a
    }
    ```
    
    위의 코드가 컴파일될 때, 컴파일러는 함수 내부의 코드를 호출하는 위치에 복사한다. 컴파일되는 바이트코드의 양은 많아지겠지만, 함수 호출을 하거나 추가적인 객체를 생성하는 부분은 없다. 위의 코드를 자바로 컴파일해보면, 다음과 같이 코드가 복사된 것을 알 수 있다.
    
    ```java
    public final class InlineFunctions {
    
       @JvmStatic
       public static final void main(@NotNull String[] args) {
          int a = 2;
          int var5 = false;
          String var6 = "Just some dummy function";
          System.out.println(var6);
          int result = 2 * a;
          System.out.println(result);
       }
    }
    ```
    

### 💡 코루틴

- 킹갓황루틴이지만 Rx를 다루는 문서이기 때문에 간단히 다루고 넘어가겠다..ㅎㅎ
- RxKotlin에서의 동시성 구현은 스케줄러를 통해 이뤄진다(코루틴은 다른 방식!)

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/226be3ab-a123-41c5-8285-ca14b3227dd1/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221110T121115Z&X-Amz-Expires=86400&X-Amz-Signature=10fe85a61127a9f27d64a92e69a0561725472b7b4ec57ba7c6cf5a52a463df8f&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

**➕ 더 알아보기..**

- **시퀀스**
    - `Collection`과는 또다른 **Container Type** `Sequence`는 각각 하나의 Element에 대해 모든 단계의 처리를 수행한다. `Sequence`는 중간 단계의 결과에 대한 처리를 피할 수 있게 해서 전체 처리에 대한 수행 성능을 향상시켜준다.
        - 주의❗ 코루틴에서 제공하는 객체가 아닌 코틀린에서 제공하는 객체
    - ****같이 보면 좋을 내용 : [Lazy Evaluation을 위한 Sequence(내 TIL 222 ㅎㅎ)](https://github.com/mdb1217/TIL/blob/main/Kotlin/Sequence%20For%20Lazy%20Evaluation.md)**

### 💡 함수형 프로그래밍: 모나드

- **모나드란?** 값을 캡슐화하고 추가 기능을 더해 새로운 타입을 생성하는 구조체
- 모나드는 함수를 결합하고 추가 계산을 통해 반환 값을 유형으로 래핑하는 구조를 가진 소프트웨어 디자인 패턴이다.
- 모나드는 하나의 타입이며, 인자를 `Type` 타입으로 받아(Java와 C++의 제너릭이라고 생각하면됨!) 값을 캡슐화하여 값을 가공할 수 있는 추가기능(보통 map으로 처리) 오퍼레이터를 사용할 수 있는 `Functor`를 사용하여 최종적으로 이러한 프로세스를 구현하는 구조를 새롭게 생성하는 특징을 가진다. 하단 3가지 조건을 만족하면 모나드 패턴이다.
    1. **타입을 인자로 받는 타입이다.(ex. Maybe)**
    2. **unit(return) operator가 있어야 한다.**
    3. **bind operator가 있어야 한다.(가공자)**
    
    ![스크린샷 2022-11-10 오후 4.49.36.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/02b3dc1f-5080-4373-be5d-1a4ed9325324/스크린샷_2022-11-10_오후_4.49.36.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221110T121031Z&X-Amz-Expires=86400&X-Amz-Signature=5ce96d322f5d83e88e44e34e3b22413fc77e40ca2a88881c55b16f1bb678f7ac&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22%25EC%258A%25A4%25ED%2581%25AC%25EB%25A6%25B0%25EC%2583%25B7%25202022-11-10%2520%25EC%2598%25A4%25ED%259B%2584%25204.49.36.png%22&x-id=GetObject)
    
    ![스크린샷 2022-11-10 오후 4.38.38.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/cd75ef24-af4a-4cf5-b2b5-31498d5fd124/스크린샷_2022-11-10_오후_4.38.38.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221110T121154Z&X-Amz-Expires=86400&X-Amz-Signature=d1b80ff4d8f9deb3ae93ba374ba0ffef09258bf94fbde280918ecbbe84a09e65&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-11-10%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%25204.38.38.png%22&x-id=GetObject)
    

### 💡 Maybe

![https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/maybe.png](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/maybe.png)

모나드인 Maybe는 값을 포함할 수도 있고 포함하지 않을 수도 있으며, 값 또는 오류 여부에 관계없이 완료된다. 그래서 오류가 발생했을 때는 onError가 호출된다. 오류가 발생하지 않고 값이 존재하면 onSuccess가 값과 함께 호출된다. 여기서 주의해야 할 것은 onError, onComplete, onSuccess 세 가지 메소드가 모두 터미널 메소드인데 세가지 메소드 중 하나는 모나드에 의해 호출되지만, 다른 것은 호출되지 않는다.

- **값이 포함되지 않으면 onComplete(데이터 발행이 완료됐음을 알림)**
- **값이 존재하면 onSuccess(데이터 하나를 발행함과 동시에 종료)**
- **에러가 발생하면 onError**
- **여러 개의 데이터를 발행할 수 있는 Observable과 달리 Maybe는 한 개의 데이터만을 발행한다**
- **onSuccess가 데이터를 발행함과 동시에 종료되므로 이후에 나온 onComplete는 실행되지 않는다.(따라서 onSuccess 호출시 onComplete 실행 X 다른 것도 마찬가지!)**

```kotlin
fun main(args: Array<String>) {
    val maybeValue: Maybe<Int> = Maybe.just(14)//1
    maybeValue.subscribeBy(//2
            onComplete = {println("Completed Empty")},
            onError = {println("Error $it")},
            onSuccess = { println("Completed with value $it")}
    )
    val maybeEmpty:Maybe<Int> = Maybe.empty()//3
    maybeEmpty.subscribeBy(
            onComplete = {println("Completed Empty")},//4
            onError = {println("Error $it")},//5
            onSuccess = { println("Completed with value $it")}//6
    )
}
```

**💡 실행 결과**

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/8e86db7e-5c0f-454f-896e-0fcea8e5f764/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221110%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221110T121230Z&X-Amz-Expires=86400&X-Amz-Signature=28eb1f0860b8b947b19f2d587ae1c0888da407c4c5e2b592304f013df4844dfd&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

### 💡 단일 모나드

- Maybe는 모나드의 한 유형(Maybe 외에도 존재)
- 최대 데이터 하나를 발행
