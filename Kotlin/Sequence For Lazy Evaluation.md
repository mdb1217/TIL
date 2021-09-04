# Lazy Evaluation을 위한 Sequence :camera_flash:

### 1. Sequence 란?

`Collection`과는 또다른 **Container Type**. ` Sequence`는 각각 하나의 Element에 대해 모든 단계의 처리를 수행한다. `Sequence` 는 중간 단계의 결과에 대한 처리를 피할 수 있게 해서 전체 처리에 대한 수행 성능을 향상시켜준다. 

<br>

### 2. Lazy Evaluation이란?

:arrow_forward: **계산의 결과값이 필요할 때까지 계산을 늦추는 기법**

------

:arrow_forward: **Lazy Evaluation은 필요없는 계산을 하지 않으므로 실행을 더 빠르게 할 수 있다.** 

------

:arrow_forward: 그 외 Lazy Evaluation의 장점은 [링크](https://ko.wikipedia.org/wiki/%EB%8A%90%EA%B8%8B%ED%95%9C_%EA%B3%84%EC%82%B0%EB%B2%95) 참조.

<br>

### 3. Sequence VS Collection

|            |    Sequence     |                  Collection                  |
| ---------- | :-------------: | :------------------------------------------: |
| Evaluation | Lazy Evaluation | Eager Evaluation<sup>[[1]](#footnote1)</sup> |

> <a name="footnote1">[1]</a> 대부분의 전통적 프로그래밍 언어에서 사용하는 계산 전략. 각 단계가 끝나고 다음 단계로 넘어가게된다.

위 표에서 나온 Evaluation의 차이가 두 **Container Type**의 가장 큰 차이이다. 간단한 예시를 통해 위의 차이점을 확실히 알아보자.

- **Collection(Eager Evaluation)**

  ```kotlin
  val findFirst = listOf(1, 2, 3, 4, 5, 6).filter { it % 2 == 0 }.map { value -> value * 10 }.first() 
  ```

  1. 1~6까지 원소를 포함하고있는 리스트를 생성하고 **[1, 2, 3, 4, 5, 6]**
  2. 원소 중 짝수만 걸러낸 후 **[2, 4, 6]**
  3. 원소에 10을 곱하여 **[20, 40, 60]**
  4. 첫번째 원소를 반환한다. **20**

- **Sequence(Eager Evaluation)**

  ```kotlin
  val findFirst = listOf(1, 2, 3, 4, 5, 6).asSequence().filter { it % 2 == 0 }.map { value -> value * 10 }.first()
  ```

  1. 1~6까지 원소를 포함하고있는 리스트를 생성하고 **[1, 2, 3, 4, 5, 6]**
  2. filter의 조건에 충족하는 첫 번째 원소를 찾은 후 **2**
  3. 10을 곱하고 반환한다.  **20** 

  **→ Sequence를 사용하면 불필요한 중간단계의 리스트가 생성되지 않기때문에 연산을 효율적으로 실행할 수 있음.**

<br>

### 4. 주의점

Sequence를 사용했을 때 무조건 효율적인 결과를 얻을 수 있는 것은 아니다. 크기가 작은 collection 이나 단순한 연산 동작에 대해서는 오히려 불필요한 오버헤드가 생길 수 있다. 그러므로 어느 경우에 **Sequence** 나 **Iterable** 이 나을지 적절한 선택을 해야 한다.

<br>

### 5. 참고자료

https://jjeda.tistory.com/25

https://iosroid.tistory.com/79
