# 🍒 DataBinding을 사용할 때 NameSpace app과 bind의 차이점(이라 쓰고 차이가 없다고 읽는다)

## 0. **xmlns란?**

**→ xmlns = xml name space**

- XML Namespaces란 각각의 요소들의 네임 충돌을 방지할 수 있는 방법이다(진짜 말 그대로 NameSpace이다..)
- 접두어를 통해 네임 충돌을 해결하는 것이다
- **기본 문법 : `xmlns:prefix="URI"`**

**ex.**

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
```

❗여기서 `xmlns:android`, `xmlns:app`, `xmlns:tools` 는 오른쪽에 표기된 URI를 담는 네임스페이스의 역할을 한다. **따라서 해당 네임을 써야만 URI가 보유한 속성에 절대적으로 접근할 수 있는 것은 아니다!**(그래도 혼란을 방지하기 위해 default로 많이 사용하는 `xmlns:android`, `xmlns:app`, `xmlns:tools`의 네이밍을 따르자)

❗여기서 URI들은 모두 패키지이다

**ex. 아래의 두 코드는 같은 동작을 한다.**

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView
         android:id="@+id/tv_order"
         ...
    />
```

```xml
<layout xmlns:mdb="http://schemas.android.com/apk/res/android">
    <TextView
         mdb:id="@+id/tv_order"
         ...
    />
```

- 보다 자세한 내용은 [문서](https://www.w3schools.com/xml/xml_namespaces.asp) 참조

## 1. app name space

- `xmlns:app="http://schemas.android.com/apk/res-auto"`
- 즉, `http://schemas.android.com/apk/res-auto` URI를 담는 네임 스페이스
- 정확히는, `http://schemas.android.com/apk/res/[yourpackage name]` 형식으로 작성

  **ex. 레이아웃 매니저 속성 :** 
       `http://schemas.android.com/apk/res/layoutManager`

## 2. app(`http://schemas.android.com/apk/res/[yourpackage name]`) 속성의 용도

- 사실은 `http://schemas.android.com/apk/res` 하위의 모든 attribute에 접근할 때 사용
- 그러나 주로 아래와 같은 용도로 사용
    - 하위 버전 대응
    - 커스텀 뷰 생성 시 커스텀 attribute 정의

## 3. app vs bind

![Untitled](https://worried-capybara-0f7.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F92ed8f9b-4c5c-497f-b691-9940a24f2749%2FUntitled.png?id=fe2f60a0-c511-434b-93e3-a66627a71ad8&table=block&spaceId=791e5689-e98d-4b2d-8794-8b0cb8b434cc&width=2000&userId=&cache=v2)

![Untitled](https://worried-capybara-0f7.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fcb220be3-e591-49c9-af70-47c3bd2aa4fd%2FUntitled.png?id=fae9c0a1-fbd5-455a-8501-4eb93b9be24a&table=block&spaceId=791e5689-e98d-4b2d-8794-8b0cb8b434cc&width=2000&userId=&cache=v2)

xmlns가 이런 식으로 정의되어 있고 DataBinding 관련 예제 코드를 볼 때 `bind:~~` 이런 식으로 된 코드를 많이들 보았을 것이다. 결**론부터 얘기하자면 `xmlns:app`이나 `xmlns:bind` 나 `http://schemas.android.com/apk/res-auto`를 참조하는 name space이기 때문에 둘다 같은 동작을 한다.** **따라서 `xmlns:bind="http://schemas.android.com/apk/res-auto"` 는 DataBinding에서 활용하는 attribute를 해당 name space로 관리한다는 걸 명시적으로 보여주기위해서 사용한다고 볼 수 있다.**

## 3. 참고자료

[https://velog.io/@odesay97/레이아웃-xml-파일에서-xmlns-코드의-의미는-xml에서의-import-기능](https://velog.io/@odesay97/%EB%A0%88%EC%9D%B4%EC%95%84%EC%9B%83-xml-%ED%8C%8C%EC%9D%BC%EC%97%90%EC%84%9C-xmlns-%EC%BD%94%EB%93%9C%EC%9D%98-%EC%9D%98%EB%AF%B8%EB%8A%94-xml%EC%97%90%EC%84%9C%EC%9D%98-import-%EA%B8%B0%EB%8A%A5)

[https://stackoverflow.com/questions/28045648/android-layout-when-to-use-app-vs-android](https://stackoverflow.com/questions/28045648/android-layout-when-to-use-app-vs-android)

[https://stackoverflow.com/questions/56503357/difference-between-using-bind-and-app-namespaces-to-set-a-custom-attribute-w](https://stackoverflow.com/questions/56503357/difference-between-using-bind-and-app-namespaces-to-set-a-custom-attribute-w)
