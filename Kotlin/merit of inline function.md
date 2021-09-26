# 함수를 파라미터로 넘길때 inline functions의 장점 :ocean:

코틀린에서 **고차함수(High order functions)** 를 파라미터로 넘기면 추가적인 메모리 할당과 함수호출로인해 Runtime overhead가 발생한다. **inline functions** 는 내부적으로 함수 내용을 호출되는 위치에 복사하며, Runtime overhead를 줄여준다.

<br>

## 1. 고차함수의 Runtime penalties

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

<br>

## 2. inline functions 구현 및 동작 원리

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

이런 이유로 **inline functions**는 일반 함수보다 성능이 좋다. 하지만 inline functions는 내부적으로 코드를 복사하기 때문에, 인자로 전달받은 함수는 다른 함수로 전달되거나 참조될 수 없다. 아래 코드에서 `newMethod()`는 inline으로 선언되었으며, `someMethod()`를 호출하며 함수를 인자로 전달합니다. 하지만 inline 함수에서는 인자로 전달받은 함수를 참조할 수 없기 때문에 전달하는 것은 불가능하다.

```kotlin
inline fun newMethod(a: Int, func: () -> Unit, func2: () -> Unit) {
    func()
    someMethod(10, func2)
}

fun someMethod(a: Int, func: () -> Unit):Int {
    func()
    return 2*a
}

fun main(args: Array<String>) {
    newMethod(2, {println("Just some dummy function")},
            {println("can't pass function in inline functions")})
}
```

다음과 같은 컴파일 에러가 발생한다.

```bash
Error:(9, 24) Kotlin: Illegal usage of inline-parameter 'func2' in 'public final inline fun newMethod(a: Int, func: () -> Unit, func2: () -> Unit): Unit defined in example.InlineFunctions'. Add 'noinline' modifier to the parameter declaration
```

<br>

## 3. noinline

모든 인자를 inline으로 처리하고 싶지 않을 때 noinline 키워드를 사용한다. noinline 키워드를 인자에 사용하면 그 인자는 inline에서 제외된다. 즉, 인자를 다른 함수의 인자로 전달할 수 있다. 다음은 위의 코드에서 `newMethod()`의 인자 `func2` 앞에 `noinline` 키워드를 붙인 것이다. 이렇게 코드를 작성할 경우 컴파일은 성공하며, 이 키워드가 붙은 함수만 제외하고 모두 inline으로 처리된다.

```kotlin
inline fun newMethod(a: Int, func: () -> Unit, noinline func2: () -> Unit) {
    func()
    someMethod(10, func2)
}

fun someMethod(a: Int, func: () -> Unit):Int {
    func()
    return 2*a
}

@JvmStatic
fun main(args: Array<String>) {
    newMethod(2, {println("Just some dummy function")},
            {println("can't pass function in inline functions")})
}
```

다음은 위의 코드를 자바로 변환한 것이고, `func2()`를 제외한 나머지 코드들은 모두 inline으로 처리된다.

```java
public final void newMethod(int a, @NotNull Function0 func, @NotNull Function0 func2) {
   func.invoke();
   this.someMethod(10, func2);
}

public final int someMethod(int a, @NotNull Function0 func) {
   func.invoke();
   return 2 * a;
}

@JvmStatic
public static final void main(@NotNull String[] args) {
   String var6 = "Just some dummy function";
   System.out.println(var6);
   this_$iv.someMethod(10, func2$iv);
}
```

<br>

## 4. noinline과 crossinline

|                           noinline                           |                         crossinline                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| 함수에서 인자로 호출하는 함수가 inline이 아닌 경우에 붙여준다. | 다른 함수의 Higher-Order function 파라메터를 block {} 형태로 사용하는 경우 crossinline을 사용한다. |

<br>

## 5. 정리

> 이미 JVM은 강력하게 inline을 지원하고 있고, JVM은 코드 실행 분석을 통해 가장 이익이 되는 방법으로 inline을 하고 있다.

이미 JVM에서 inline(대표적인 예시는 스코프함수)을 하고 있으므로 사용하지 않아도 괜찮지만, 파라미터에 고차함수를 넘겨주는 형태이면 **inline키워드**가 효과적일 수 있다.

<br>

## 6. 참고자료

https://codechacha.com/ko/kotlin-inline-functions/

https://thdev.tech/kotlin/2020/09/29/kotlin_effective_04/