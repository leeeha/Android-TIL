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

그런데, enum 클래스는 내부적으로 각 value당 하나의 싱글 인스턴스를 사용하기 때문에, **서로 다른 타입은 가질 수 없다.** 

```kotlin
enum class Result {
    SUCCESS,
    FAILED(val exception: Exception) // 서로 다른 타입은 불가능!! 
}
```

이를 해결하기 위해 아래와 같이 추상 클래스를 정의할 수도 있는데, 그러면 Result의 자식 클래스 종류가 제한되지 않기 때문에 enum 클래스가 갖는 장점이 사라진다. 

```kotlin
abstract class Result<out T : Any>

data class Success(out T: Any>(val data: T) : Result<T>()
data class Failed(val exception: Exception) : Result<Nothing>()
```

# sealed class

sealed class란 추상 클래스로 **상속 받는  자식 클래스의 종류를 제한하는 특성**을 가지고 있다. 즉, 컴파일 타임에 sealed class의 자식 클래스가 어떤 것이 있는지 알 수 있다. 

enum 클래스와 마찬가지로 종류가 제한된 집합을 정의하면서도, **각 value는 서로 다른 타입을 가질 수 있기 때문에 확장성이 증가**한다.

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

    // data class Failed(val exception: Exception) : Result<Nothing>()
    sealed class Failed(val exception: Exception, val message: String?) : Result<Nothing>() {
        class NetworkError(exception: Exception, message: String?) : Failed(exception,message)
        class StorageIsFull(exception: Exception, message: String?) : Failed(exception, message)
    }

    object Loading : Result<Nothing>()
}
```

단, Kotlin v1.5 미만에서 sealed class의 하위 클래스들은 “같은 kt 파일 또는 패키지”에 위치해야 한다. 이는 내부적으로 sealed class가 private한 생성자를 갖기 때문이다. (외부에서 sealed class를 상속 받으려고 하면 컴파일 에러가 발생한다.) 

# sealed interface

Kotlin v1.5 미만에서 `sealed class`는 최상위 클래스가 인터페이스일 수 없었고, 모든 하위 클래스는 동일한 파일이나 패키지에 존재해야만 했다. 

하지만, v1.5에서 `sealed interface`가 추가되며, 두 가지 제약 조건이 모두 사라졌다. sealed class/interface를 구현하는 하위 클래스 및 인터페이스는 다른 파일이나 패키지에 있어도 상관 없다. (단, 모듈은 같아야 한다.) 

아래는 예시 코드이다. **같은 모듈 내에서라면 어느 곳에서든 자식 클래스를 선언할 수 있게 되었다.** 여기서 모듈이란, 함께 컴파일 되는 코틀린 파일들의 집합을 의미한다. 

```kotlin
// In File1.kt
sealed class ExampleParent

// In File2.kt
class Child1 : ExampleParent()

// In File3.kt
class Child2 : ExampleParent()
```

# 마무리

Sealed Class/Interface는 **각 value의 타입은 동일해야 한다는 enum의 제약**에서 벗어나, 

**클래스 타입과 그 안의 속성들까지 다양한 타입을 가질 수 있다는 점에서 확장성을 크게 열어준다.** 

단, enum에서 제공하는 ordinal이나, 모든 원소를 values()로 반환하는 등 편의를 제공하는 부분에서는 아직 부족함이 있긴 하다. (reflection 이용하면 가능하긴 한데, 구현이 번거롭다.) 

# 참고 자료

- [Sealed Class와 Sealed Interface](https://medium.com/hongbeomi-dev/sealed-class와-sealed-interface-db1fff634860)
- [[Kotlin] Enum의 대체 - Sealed class / Sealed interface 정리](https://tourspace.tistory.com/467)
- [[Kotlin] Kotlin sealed class란 무엇인가?](https://kotlinworld.com/165#class%EB%25A-%25-C%25--%EC%25--%25--%EC%25--%25-D%25--%EB%25B-%25-B%EA%25B-%25B-)

