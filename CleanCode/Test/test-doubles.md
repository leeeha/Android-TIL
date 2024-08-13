# 테스트 기본 원칙 

## [Seven Testing Principles](https://www.boxuk.com/insight/the-seven-principles-of-testing/)

- Testing shows the presence of defects, not their absence: 테스팅은 결함의 부재가 아니라, **결함이 존재**하는 것을 보여주는 것이다.
- Exhaustive testing is impossible: 완벽한 테스트는 불가능하다. 
- Early testing saves time and money: 빨리 시작한 테스트가 돈과 시간을 절약해준다. 
- Defects cluster together: 결함은 서로 군집되어 있다.
- Beware of the pesticide paradox: 살충제의 역설 (살충제를 너무 많이 뿌리면 벌레가 내성이 생겨서 죽지 않는다.) - 동일한 테스트를 반복적으로 수행하면 새로운 결함을 찾지 못한다. 
- Testing is context dependent: 테스팅은 문맥에 의존적이다. 
- Absence-of-errors is a fallacy: 오류 부재의 궤변 - 고객의 기대와 요구사항을 만족시키는 것이 오류가 없는 시스템을 만드는 것만큼이나 중요하다. 

## [F.I.R.S.T](https://agileinaflash.blogspot.com/2009/02/first.html) Principles

- Fast: 단위 테스트는 빨라야 한다. 
- Isolated: 테스트가 다른 테스트에 의존하지 않아야 한다.
- Repeatable: 테스트는 실행할 때마다 동일한 결과여야 한다.
- Self-validating: 테스트 결과는 성공이거나 실패여야 한다. 결과에 대한 해석이 필요하면 안 된다.
- Timely: 단위 테스트는 기능이 출시된 후에도 언제든 작성할 수 있지만, 가장 적절한 시점은 프로덕션 코드를 구현하면서 같이 테스트 코드를 작성하는 것이다.

## 테스트 코드 작성 스타일 

넓은 공감대를 얻고 있는 테스트 코드 작성 스타일 중에 하나는, 마틴 파울러 선생이 제안한 Given-When-Then 스타일이다. 

- Given: 주어진 상태에서  
- When: 이런 기능을 실행하면 
- Then: 이런 결과가 나와야 한다. 

# 테스트 더블 

테스트 대상 객체가 다른 객체에 의존하고 있는 경우를 생각해보자. 이때 객체를 테스트 하기 위해 의존성이 있는 객체에 대한 실제 구현체를 사용하면, 그 구현체에 의해 테스트가 실패할 수 있다. 

<img width="600" src="https://github.com/user-attachments/assets/9209a59e-0edb-415c-8d55-707276d65dd6"/>

이럴 때 실제 구현체를 대신하여 해당 객체의 동작을 모방하는 객체를 만들어 테스트에 영향이 없도록 만들 수 있다. 이처럼 테스트를 위해 실제 객체의 동작을 모방하는 객체를 Test Doubles라고 부른다. 

<img width="600" src="https://github.com/user-attachments/assets/fb01a8a9-079b-4da4-a5d2-bea4f4f824b5"/>

F.I.R.S.T 원칙 중에 Isolated에 따르면, 테스트는 다른 테스트에 의존하지 않고 격리되어서, 어떤 순서로든 실행할 수 있어야 한다. 이때 격리된다는 것은, 테스트 대상이 의존하는 것을 실제가 아닌 다른 것으로 대체하는 것인데, 이때 사용되는 게 테스트 더블인 것이다. 

테스트 더블의 종류에는 다음 그림과 같이 Dummy, Stub, Spy, Mock, Fake가 있다.

<img width="600" src="https://github.com/user-attachments/assets/b3fc9173-f33d-4e39-acfd-1da5470f9af7"/>

## Dummy 

가장 기본적인 테스트 더블로, **기능을 전혀 구현하지 않은 객체**를 의미한다. 

인스턴스는 필요하지만 기능까지는 필요하지 않은 경우에 사용한다. 

```kotlin 
interface AccountDao {
    fun showMember()
}

class DummyAccountDao: AccountDao {
    override fun showMember(): List<String> {
        // No implementation
    }
}
```

## Stub

Stub은 **Dummy가 실제로 동작하는 것처럼 보이게 만든 객체**이다. 인터페이스 또는 클래스를 최소한으로 구현하고, **결과는 특정 상태를 가정해서 하드코딩된 값으로 제공**한다. 따라서, **로직에 따른 값의 변경은 테스트 할 수 없다.** 

Stub은 객체에 특정 메서드를 적용하기 전과 후의 상태를 비교하는 **상태 기반 테스트**에 사용되는 테스트 더블이다. (State-based Testing)

아래 코드에서 showMember() 함수는 단순히 고정된 상수값을 반환한다는 걸 알 수 있다.

```kotlin
interface AccountDao {
    fun showMember()
}

class StubAccountDao: AccountDao {
    override fun showMember(): List<String> {
        val memberList = listOf("Alice", "Bob")
        return memberList
    }
}
```

## Spy 

Spy는 **Stub과 비슷한데, 추가적인 정보를 더 기록할 수 있는 객체**이다. 예를 들어, 예상한 메서드가 잘 호출되었는지, 몇번이나 호출되었는지 등을 파악할 수 있게 해준다. 

어떤 메서드 A가 호출되었을 때, 또 다른 메서드 B가 호출되었는지 확인하는 등의 **행위 기반 테스트**에 사용되는 테스트 더블이다. (Interaction-based Testing)

아래 코드에서 showMember() 함수가 호출될 때마다 호출 횟수를 증가시키는 것을 알 수 있다. 

```kotlin
interface AccountDao {
    fun showMember()
}

class SpyAccountDao: AccountDao {
    private var callMemberCount = 0

    override fun showMember(): List<String> {
        val memberList = listOf("Alice", "Bob")
        callMemberCount++
        return memberList
    }
}
```

## Fake 

Fake는 **동작하는 구현이 있어서 테스트에는 사용할 수 있지만, 실제 프로덕션 구현과는 동일하지 않은 객체**이다. 

**상태 기반 테스트**에 사용되는 테스트 더블인데, Mocking을 필요로 하지 않고 가볍기 때문에 구글에서 추천하는 더블이다. 

Stub의 경우 메서드의 로직은 구현하지 않고 단순히 고정된 값을 반환하지만, Fake는 메서드 로직까지 구현한다는 점에서 차이가 있다. 

```kotlin 
interface AccountDb {
    fun saveItem(userId: Int, name: String)
    fun getItem(userId: Int) : String?
}
 
class FakeAccountDb : AccountDb {
    private var accounts = HashMap<Int, String>()
 
    init {
        accounts[0] = "Alice"
        accounts[1] = "Bob"
    }
 
    override fun saveItem(userId: Int, name: String) {
        accounts[userId] = name
    }
 
    override fun getItem(userId: Int) : String? {
        return accounts[userId]
    }
}
```

## Mock

Mock은 Spy처럼 **행위 기반 테스트**에 사용되는 테스트 더블이다. 차이점은 Spy가 **메서드 A와 B의 실제 동작을 추적**하는 더블인 반면에, Mock은 **A가 실행되면 단순히 B의 정상적인 작동 값을 반환한다**는 것이다.

**행위 기반 테스트**는 작성이 복잡하고 까다로운 부분이 많기 때문에 **Mocking에 사용하는 전용 라이브러리들**이 만들어져 있다. 

```kotlin
class Car(
    private val engine: Engine,
    private val dashboard: Dashboard
) {
    fun start() {
        engine.ignite()
        dashboard.display()
    }
}
 
class CarTest {
    // Mock 객체 의존성 주입 
    var engineMock: Engine = mock(Engine::class.java)
    var dashboardMock: Dashboard = mock(Dashboard::class.java)
 
    @Test
    fun start_the_car() {
        val car = Car(engineMock, dashboradMock)
        car.start()
        verify(engineMock).ignite()
        verify(dashboardMock).display()
    }
}
```

# 테스트 대상 항목 

- 단위 테스트 대상 
  - ViewModel, Presenter 
  - 데이터 레이어 
  - 유즈 케이스 
  - 값을 계산하는 유틸리티 클래스 
- 엣지 케이스 
  - 음수, 0, 경계 조건을 사용하는 수학 연산 
  - 가능한 모든 네트워크 연결 오류 
  - 형식이 잘못된 JSON 같은 손상된 데이터 
  - 파일 저장 시 스토리지가 가득 찬 상황 
  - 프로세스 중간에 다시 생성된 객체 (예: 화면 회전 시 액티비티)
- UI 테스트 
  - 스크린 UI 
  - 유저 플로우 
  - 네비게이션 
- 테스트 제외 대상 
  - 프레임워크 자체의 동작 
  - 액티비티, 프래그먼트, 서비스에는 테스트가 필요한 로직을 가능한 배치하지 말 것 

# 테스트 라이브러리 

## Testing Framework 

- Junit4 : Java의 단위 테스트 코드를 작성하기 위해 만들어진 프레임워크. Jetpack Test 라이브러리는 Junit4를 기준으로 만들어져 있음. 
- Kotest: Kotlin의 단위 테스트 코드를 작성하기 위해 만들어진 프레임워크
- Roboletric: JVM만으로 안드로이드 프레임워크를 테스트하기 위해 만들어진 프레임워크

## Assertion 

표현력이 부족한 Junit Assertion의 단점을 메워주는 여러가지 라이브러리들이 있다.

AssertJ, Truth, Hamcrest 

```kotlin
// JUnit
assertTrue(notificationText.contains("testuser@google.com"));

// AssertJ, Truth
assertThat(notificationText).contains("testuser@google.com");

// Hamcrest
assertThat(notificationText, containsString("testuser@google.com"));
```

## Mocking 

- Mockito: Java 테스트를 위한 Mocking 라이브러리 
- MockK: Kotlin 테스트를 위한 Mocking 라이브러리 

## UI Testing 

- Espresso: 단일 안드로이드 앱의 UI를 테스트 하는 프레임워크 
- Kaspresso: KasperskyLab에서 제작한 안드로이드 앱 UI 테스트 프레임워크 
- Appium: iOS와 Android에서 모두 가능한 UI 테스트 프레임워크 
- UI Automator: 여러 앱 간의 UI 기능을 테스트 하는 프레임워크 

# 참고자료 

https://cliearl.github.io/posts/android/android-test-basic

https://kotlinworld.com/481