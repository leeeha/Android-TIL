# enum class

- 개발을 하다보면, **제한된 일들의 집합을 정의**하고 싶을 때가 있다.
  - ex) 웹 요청에 대한 로딩, 성공, 실패 상태 또는 유저의 타입 등
- 이러한 집합을 Java 시절처럼 단순히 **if-else 구문**으로 처리하면, 누락되는 분기가 있거나 의도한 바와 전혀 다른 타입이 들어올 때도 **컴파일 에러가 발생하지 않는다.**
- 이때 **enum 클래스로 가능한 타입의 종류를 제한**하면, 위와 같은 문제를 해결할 수 있다.

```java
// 전통적인 방식
const val SUCCESS = 0
const val FAILED = 1

fun checkResult(result: Int) {
    if (result == SUCCESS) {
        // do something
    else if (result == FAILED) {
        // do something
    }
    // 만약 result로 3이 들어온다면?
    ...
}

// Enum으로 교체 시
enum class Result {
    SUCCESS,
    FAILED,
}

fun checkResult(result: Result) {
    if (result == SUCCESS) {
        // do something
    else if (result == FAILED) {
        // do something
    }
    // result에는 enum 이외의 값이 들어올 수 없다.
    ...
}
```

그런데, enum 클래스는 **각 열거 상수가 단일 인스턴스로 존재**하며, **동일한 데이터 구조를 공유**한다. 따라서, 아래 코드처럼 **각 상수가 서로 다른 데이터 타입이나 구조를 가질 수 없다.** 

```kotlin
enum class Result {
    SUCCESS,
    FAILED(val exception: Exception)
}
```

이를 해결하기 위해 아래와 같이 추상 클래스를 정의할 수도 있는데, 그러면 Result의 자식 클래스 종류가 제한되지 않기 때문에 enum 클래스가 갖는 장점이 사라진다. 

```kotlin
abstract class Result<out T : Any>

data class Success(out T: Any>(val data: T) : Result<T>()
data class Failed(val exception: Exception) : Result<Nothing>()
```

이럴 때 sealed class를 사용하면, **자식 클래스의 종류를 제한**하면서도 각 상태에 따라 **다양한 데이터 타입을 포함**하는 클래스를 정의할 수 있다. 

# sealed class

sealed class란 추상 클래스로 상속 받는 **자식 클래스의 종류를 제한**하는 특성을 가지고 있다. 즉, **컴파일 타임에 sealed class의 자식 클래스가 어떤 것이 있는지** 알 수 있다. 

enum 클래스와 마찬가지로 종류가 제한된 집합을 정의하면서도, **자식 클래스들은 서로 다른 타입을 가질 수 있기 때문에 확장성이 증가**한다.

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Failed(val exception: Exception) : Result<Nothing>()
    object Loading : Result<Nothing>()
}
```

위의 코드에서 성공, 실패, 로딩이라는 세 가지 상태가 Result 라는 범위로 묶이게 된다. 

그리고 각 타입은 **일반 클래스, 데이터 클래스, object, 심지어 sealed class 등 모두 가능**하며, 그 안의 요소들은 **서로 다른 타입**이 될 수 있다. 

```kotlin
sealed class Result<out T : Any> {
    data class Success<out T : Any>(val data: T) : Result<T>()

    sealed class Failed(val exception: Exception, val message: String?) : Result<Nothing>() {
        class NetworkError(exception: Exception, message: String?) : Failed(exception,message)
        class StorageIsFull(exception: Exception, message: String?) : Failed(exception, message)
    }

    object Loading : Result<Nothing>()
}
```

단, **Kotlin v1.5 미만**에서 sealed class의 하위 클래스들은 **같은 파일 또는 패키지에 위치**해야 한다. 이는 내부적으로 sealed class가 private한 생성자를 갖기 때문이다. (외부에서 sealed class를 상속 받으려고 하면 컴파일 에러가 발생한다.) 

# sealed interface

Kotlin v1.5 미만에서 `sealed class`는 최상위 클래스가 인터페이스일 수 없었고, 모든 하위 클래스는 같은 파일이나 패키지에 존재해야 했다. 

그러나, v1.5에서 `sealed interface`가 추가되며, 두 가지 제약 조건이 모두 사라졌다. 

sealed class와 sealed interface를 구현하는 하위 클래스 및 인터페이스는 **다른 파일이나 패키지에 있어도 상관 없다. (단, 모듈은 같아야 한다.)** 

추가적으로, sealed interface는 sealed class에서 불가능한 **다중 상속도 가능**하다. 

# 마무리

Sealed Class/Interface는 **각 열거 상수의 타입이 모두 동일해야 한다는 enum의 제약**에서 벗어나, 클래스 타입과 그 안의 속성들까지 **다양한 타입을 가질 수 있다**는 점에서 **확장성**을 크게 열어준다.

단, enum에서 제공하는 ordinal이나, 모든 원소를 values()로 반환하는 등 편의를 제공하는 부분에서는 아직 부족함이 있긴 하다. (reflection 이용하면 가능하긴 한데, 구현이 번거롭다.) 

# 2025 안드로이드 탐구 영역 10번 문제 

**Kotlin 2.0 이상**에서 sealed class의 특성으로 옳은 것만을 고르시오. 

ㄱ. sealed class를 상속하는 모든 하위 클래스 또한 컴파일러가 컴파일 타임에 인지할 수 있다. (F)<br>
ㄴ. sealed class를 상속하려는 하위 클래스는 반드시 같은 파일 안에 정의되어야 한다. (F)<br>
ㄷ. when 구문에 사용 시 else를 사용하지 않고, exhaustive하게 처리할 수 있다. (T)<br>

다음과 같은 [코틀린 공식문서](https://kotlinlang.org/docs/sealed-classes.html) 설명에 따르면 

>봉인된 클래스와 인터페이스는 클래스 계층 구조의 제한된 상속을 제공합니다. 봉인된 클래스의 모든 direct 서브 클래스는 컴파일 시점에 알 수 있습니다. 다른 서브 클래스는 봉인된 클래스가 정의된 모듈이나 패키지 외부에 정의할 수 없습니다. 봉인된 인터페이스와 그 구현에도 동일한 논리가 적용됩니다. 봉인된 인터페이스가 있는 모듈이 컴파일되면 새 구현을 만들 수 없습니다.<br>
direct 서브 클래스는 super 클래스로부터 바로 상속하는 클래스입니다. indirect 서브 클래스는 super 클래스로부터 한 단계 이상 아래에서 상속하는 클래스입니다. 

보기 ㄱ에서 indirect subclass는 컴파일러가 컴파일 타임에 인지할 수 없다는 결론이 나온다. 따라서 정답은 ㄷ이다. 

# 참고 자료

- [Sealed Class와 Sealed Interface](https://medium.com/hongbeomi-dev/sealed-class와-sealed-interface-db1fff634860)
- [[Kotlin] Enum의 대체 - Sealed class / Sealed interface 정리](https://tourspace.tistory.com/467)
- [[Kotlin] Kotlin sealed class란 무엇인가?](https://kotlinworld.com/165#class%EB%25A-%25-C%25--%EC%25--%25--%EC%25--%25-D%25--%EB%25B-%25-B%EA%25B-%25B-)

