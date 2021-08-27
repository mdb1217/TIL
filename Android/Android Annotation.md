# Android Annotation:snail:

### 1. Annotation이란?

개발자가 앱의 코드를 이해하는 데 도움이 되는 메타데이터를 노출함. 주석과 다르게 특정 코드에 달아 어떤 의미를 부여하거나 기능을 주입할 수 있음.

### 2. 종류

# ✨ Res 관련 어노테이션

메서드나 지역 변수, 필드 반환 값이 리소스 참조가 될 것으로 예상됨을 나타냄.

- **@AnimatorRes** : animator 리소스 참조 — `android.R.animator.fade_in`
- **@AnimRes** : anim 리소스 참조 — `android.R.anim.fade_in`
- **@AnyRes** : 모든 타입의 리소스 참조 (특정 타입을 알고 있는 경우 구체적인 주석을 사용하면 됨)
- **@ArrayRes** : 배열 타입의 리소스 참조 — `android.R.array.phoneTypes`
- **@AttrRes** : attribute 리소스 참조 — `android.R.attr.action`
- **@BoolRes** : boolean 리소스 참조
- **@ColorRes** : color 리소스 참조 — `anroid.R.color.white`
- **@DimenRes** : dimen 리소스 참조 — `android.R.dimen.app_icon_size`
- **@DrawableRes** : drawable 리소스 참조 — `android.R.attr.alertDialogIcon`
- **@FontRes** : font 리소스 참조 — `R.font.myfont`
- **@FractionRes** : 퍼센트 리소스 참조 — `R.fraction.pivot_x`
- **@IdRes** : id 리소스 참조 — `R.id.button`
- **@IntegerRes** : integer 리소스 참조 — `android.R.integer.config_shortAnimTime`
- **@InterpolatorRes** : interpolator 리소스 참조 — `android.R.interpolator.cycle`
- **@LayoutRes** : layout 리소스 참조 — `android.R.layout.list_content`
- **@MenuRes** : menu 리소스 참조
- **@NavigationRes** : navigation 리소스 참조 — `R.navigation.flow`
- **@PluralsRes** : plurals(개수에 따른 문자열 표현)리소스 참조 — `getQuantityString` 메서드 처럼 아이템의 갯수에 따른 각각 다른 문자열을 표현하고 싶을 때 사용하는 resource 타입입니다. [참고](https://medium.com/@info.anikdey003/android-quantity-strings-plurals-5ed3d7c62c16)
- **@RawRes** : raw(원본 리소스들: txt, 음악 파일 등) 리소스 참조
- **@StringRes** : string 리소스 참조
- **@StyleableRes** : styleable 리소스 참조 —`android.R.styleable.TextView_text`
- **@StyleRes** : style 리소스 참조 — `android.R.style.TextAppearance`
- **@TransitionRes** : transition(전환 애니메이션 효과) 리소스 참조
- **@XmlRes** : XML 리소스 참조

# 🚀 Thread 관련 어노테이션

메소드에 해당 어노테이션이 붙은 경우 해당 스레드에서 호출될 수 있음을 나타내며 클래스에 붙은 경우 클래스의 모든 메소드는 해당 스레드에서 호출 될 수 있음.

- **@AnyThread** : 모든 스레드에서 호출 가능합니다.
- **@BindThread** : [바인더](https://developer.android.com/guide/components/aidl) 스레드에서만 호출 가능합니다. (서비스의 별도 스레드)
- **@MainThread(@UiThread)** : 메인(UI) 스레드에서만 호출 가능합니다.
- **@WorkerThread** : worker(백그라운드 작업 같은 오래 걸리는 작업) 스레드에서만 호출 가능합니다.

# 🛠 그 밖의 유용한 어노테이션

- **@CallSuper** : 이 어노테이션이 붙은 메서드를 하위 클래스에서 오버라이드 할 땐 반드시 상위 클래스의 메서드를 호출하도록 강제함을 나타냄.

ex ) 액티비티의 라이프사이클 메서드

- **@CheckResult** : 해당 어노테이션이 붙은 메서드가 리턴한 결과값을 개발자가 따로 사용하지 않고 무시했을 때 오류를 뱉어냅니다.

```
@CheckResult
fun trim(s: String) = s.trim()...
s.trim() // 결과값을 따로 세팅하지 않아주었으므로 컴파일 에러 발생
val str = s.trim() // ok
```

- **@CheckSdkIntAtLeast** : 해당 어노테이션이 추가된 메서드가 SDK_INT API 레벨이 최소 주어진 값인지 검사하고 결과를 반환하거나 지정된 람다식을 실행합니다.

```
@ChecksSdkIntAtLeast(api = Build.VERSION_CODES.O)
fun isAtLeastO() = Build.VERSION.SDK_INT >= 26// 어노테이션의 인자로 들어갈 수 있는 속성은 다음과 같습니다.
// api: Int -> 최소 API
// codename: String -> 최소 API의 코드명("R")
// lambda: Int -> 최소 API 레벨을 만족할 경우 실행될 람다의 파라미터 번호
// parameter: Int -> 최소 API 레벨이 파라미터에 지정되며 첫 번째 파라미터 번호는 0부터 시작입니다.@ChecksSdkIntAtLeast(parameter = 0, lambda = 1)
inline fun fromApi(value: Int, action: () -> Unit) {
  if (Build.VERSION.SDK_INT >= value) {
    action()
  }
}
// 0번째 파라미터에 최소 API 레벨을 주었으며 1번째 파라미터에 실행될 람다식을 넣어줌을 // 뜻합니다.// Kotlin 프로퍼티에도 사용 가능
@get:ChecksSdkIntAtLeast(api = Build.VERSION_CODES.GINGERBREAD)
val isGingerbread: Boolean
  get() = Build.VERSION.SDK_INT >= Build.VERSION_CODES.GINGERBREAD
```

- **@ContentView** : 생성자에 연결할 수 있는 어노테이션이며 `LayoutRes`를 생성자의 인자로 받으면 `setContentView`를 대체해줍니다.

```
class MainFragment: Fragment {
  fun MainFragment() {
    // 아래 생성자는 @ContentView 어노테이션이 연결되어 있습니다.
    super(R.layout.main)
  }
}// Fragment 내부 코드
@ContentView
public Fragment(@LayoutRes int contentLayoutId) {
    this();
    mContentLayoutId = contentLayoutId;
}
```

- **@DoNotInline** : 코드가 빌드 시 최적화 할 때 어노테이션이 연결된 메소드는 인라인으로 변경되지 않아야 함을 나타냅니다. 이는 일반적으로 별도의 클래스에 포함되도록 의도적으로 라인 외부 메서드의 인라인을 방지하는 데 사용됩니다.
- **@GuardedBy** : 해당 필드나 메소드를 사용하려면 락을 보유해야만 접근할 수 있다는 것을 의미합니다.

```
private val anyLock = Any()

@GuardedBy("anyLock") // anyLock 객체를 보유해야 필드 접근 가능
@Volatile
private var lockObject : Any? = null

fun getLock(): Any? {
    synchronized(anyLock) {
        if (lockObject == null) {
            lockObject = Any()
        }
    }
    return lockObject
}
```

- **@Keep** : 빌드 시 코드가 최적화 될 때 어노테이션이 연결된 요소를 제거하지 않아야 함을 나타냅니다.
- **@OptIn** : 어노테이션이 연결된 파일, 선언, 식에서 [opt-in](https://kotlinlang.org/docs/opt-in-requirements.html) API를 사용할 수 있습니다.
- **@RequiresOptIn** : 해당 어노테이션이 연결된 요소가 OptIn API의 마커임을 나타냅니다. 불안정하거나 비표준 API의 일부로 간주되며 해당 요소를 호출하려면 OptIn을 사용하거나 **@OptIn** 어노테이션을 달아야합니다.
- **@RequiresApi** : 해당 어노테이션은 **@TargetApi** 와 유사하게 지정된 API 레벨 이상에서만 호출되어야 함을 의미하지만 해당 메소드에 대한 호출자의 요구 사항임을 더욱 명확하게 나타냅니다.
- **@RequirePermission** : 하나 이상의 권한이 필요하거나 필요할 수 있음을 나타내는 어노테이션 입니다.

```
// 단일 권한이 필요한 예
@RequiresPermission(ACCESS_COARSE_LOCATION)
abstract fun getLastKnownLocation(String provider): Location// 하나 이상의 권한을 요구하는 예
@RequiresPermission(
  anyOf = { ACCESS_COARSE_LOCATION, ACCESS_FINE_LOCATION }
)
abstract fun getLastKnownLocation(String provider): Location// 여러 권한이 필요한 예
@RequiresPermission(
  allOf = {ACCESS_COARSE_LOCATION, ACCESS_FINE_LOCATION}
)
abstract fun getLastKnownLocation(String provider): Location// Content Provider에 대해 별도의 읽기 및 쓰기 권한을 요구하는 예
@RequiresPermission.Read(
  @RequiresPermission(READ_HISTORY_BOOKMARKS)
)    @RequiresPermission.Write(
  @RequiresPermission(WRITE_HISTORY_BOOKMARKS)
)
const val BOOKMARKS_URI = Uri.parse("content://browser/bookmarks")// 매개 변수에 지정된 경우 어노테이션은 메소드에 매개 변수 값에 따른 권한이 필요함을 
// 나타냅니다.
fun startActivity(@RequiresPermission intent: Intent) { ... }
```

- **@RestrictTo** : 해당 요소는 특정 범위 내에서만 액세스해야 함을 나타냅니다([**Scope**](https://developer.android.com/reference/kotlin/androidx/annotation/RestrictTo.Scope))

```
// 라이브러리 내 사용 제한 예(Group Id)
@RestrictTo(GROUP_ID)
fun resetPaddingToInitialValues() { ... }// 테스트에서만 사용 제한 예
@RestrictScope(TESTS)
abstract fun getUserId(): Int// 서브 클래시에서만 사용을 제한하는 예
@RestrictScope(SUBCLASSES)
fun onDrawForeground(canvas: Canvas) { ... }
```

- **@VisibleForTesting** : 선택적으로 가시성을 지정할 수 있습니다. 기본적으로 테스팅 용으로만 공개될 수 있게 지정됩니다.

```
@VisibleForTesting(otherwise = VisibleForTesting.PROTECTED)
fun printDiagnostics(): String { ... }// NONE -> 인자가 없을 시 기본 지정. 해당 요소는 테스트에서만 호출되어야 합니다.
// PACKAGE_PRIVATE -> 패키지 비공개 가시성을 갖습니다.
// PRIVATE 
// PROTECTED
```

- **@ColorInt** : 패킹된 색상 int를 나타냄을 의미합니다.
- **@Dimension** : 변수, 필드 또는 메소드의 반환 값이 dimension을 나타낼 것으로 예상됨을 나타냅니다. `unit` 인수로 **DP(0)**, **PX(1)**, **SP(2)**를 지정할 수 있습니다.
- **@Px** : 변수, 필드 또는 메소드의 반환 값이 pixel dimension임을 나타냅니다.
- **@FloatRange**, **@IntRange** : 각각 주어진 범위에서 float 또는 double, int이어야 함을 나타냅니다.

```
@FloatRange(from=0.0, to=1.0)
fun getAlpha(): Float { ... }
```

- **@Size** : 어노테이션이 연결된 요소가 주어진 크기 또는 길이를 가져야 함을 나타냅니다. -1은 설정되지 않음을 의미합니다.

```
fun getLocationInWindow(@Size(2) location: IntArray) { ... }Size(value: Long, min: Long, max: Long, multiple: Long)
// value -> 정확한 크기 지정
// min -> 최소 크기
// max -> 최대 크기
// multiple -> 해당 인자의 배수 크기
```

### 3. 참고자료

https://developer.android.com/studio/build/migrate-to-kts?hl=ko

https://techblog.woowahan.com/2625/

https://aroundck.tistory.com/7150