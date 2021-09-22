# Hot Stream VS Cold Stream :ice_cream:

### 1. Hot Stream이란?

- Stream을 생성하자마자 아이템들을 흘려보낸다는 의미.
- 모든 데이터가 스트림 외부에서 생성됨.(이 데이터는 스트림 없이 존재할 수 있으며, 일반적으로 외부 컴포넌트에서 가져옴)

- Stream을 생성하고 일정 시간이 지난 상태에서 subscribe하게 되면 처음부터 아이템을 받아보지는 못하고, 중간부터 나온 아이템들 부터 subscribe 할 수 있음.

- 구독자의 존재 여부와 상관없이 데이터를 배출하는 Stream.(Eager Stream)
- 구독 시점으로부터 발행하는 값을 받는 걸 기본으로 한다.
- 마우스 이벤트, 키보드 이벤트, 시스템 이벤트 등이 주로 사용된다.
- Hot 스트림은 0개 이상의 구독자를 가질 수 있으며 동시에 모든 구독자에게 값을 내보낸다.(멀티캐스팅)

<br>

### 2. Cold Stream이란?

- Hot Stream과 반대의 개념.
- 모든 데이터가 스트림 내부에서 생성
- 구독자가 오로지 1명 뿐이며, 해당 구독자에게만 값을 내보낸다. 또한 해당 구독자는 스트림을 초기화 하는 구독자임.(유니캐스팅)
- 스크림이 수신하는(receive) 각각의 새로운 구독에 대해서는 해당 코드의 새로운 실행이 트리거되며 이것은 이전 구독과는 독립적임. 즉 동일한 스트림의 여러 인스턴스를 가질 수 있으며 이들은 각각 독립적임.

- Subscribe 할 때, 아이템들을 흘려보내게 됨.(Lazy Stream)

- 일반적인 옵저버 형태를 말함.
- 누가 구독하지 않으면 데이터를 배출해주지 않음.
- 일반적인 웹 요청, 데이터베이스 쿼리 등이 사용되며 내가 요청하면 결과를 받는 과정을 거침.
- 처음부터 발행하는 것을 기본으로 함.

<br>

### 3. Hot Stream VS Cold Stream

|                                              | Hot Stream | Cold Streamee |
| -------------------------------------------- | :--------: | :-----------: |
| 데이터가 생성되는 위치                       |    외부    |     내부      |
| 얼마나 많은 리시버가 데이터를 얻을 수 있는가 | Multicast  |    Unicast    |
| Lazy                                         |   Eager    |     Lazy      |

<br>

### 4. 참고자료

https://wonsohana.wordpress.com/2021/07/17/coroutine-flow-vs-channel-1-스트림이란/

https://taehyungk.github.io/posts/android-RxJava2-Cold-Hot-Observable-and-Subject/
