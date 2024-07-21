# 추상 클래스 vs 인터페이스

우선 **추상 클래스**는 일반적인 클래스와 동일하게 **멤버 변수와 메서드**를 갖는다. 

그리고 추가적으로 '추상 메서드'를 갖는다. 추상 메서드는 아래 코드와 같이 **메서드의 선언부만 존재하고 구현 코드가 없는 메서드**를 말한다. 

```java
public abstract String getUserName(int idx);
```

**구현부가 없는 메서드를 단 하나라도 가진 클래스는 추상 클래스가 된다.** 추상 클래스가 되면 new 키워드를 사용하여 인스턴스화 할 수 없다. 즉, 클래스를 쓸 수 없다. 이 클래스를 사용하려면 어떻게 해야 할까? 

다른 클래스가 이 추상 클래스를 상속 받아야 한다. **상속 받은 자식 클래스가 부모 클래스에 존재하는 추상 메서드를 전부 오버라이딩 하여 구현부를 작성하면 비로소 사용할 수 있는 객체가 된다.**

인터페이스는 사실 추상 클래스의 특수한 형태이다. **추상 클래스 중에서 멤버 변수와 메서드를 제거한 채 추상 메서드만을 남긴 형태가 인터페이스**이다. (인터페이스도, public static 상수를 가질 순 있다) 

인터페이스도 마찬가지로 이를 구현한 **자식 클래스에서 인터페이스의 추상 메서드를 모두 오버라이딩 해야 비로소 객체로 사용**할 수 있다. (추상클래스나 인터페이스를 구현한 자식 객체가 추상 메서드를 전부 구현하지 않았다면 아예 컴파일이 되지 않는다. 혹은 자식 객체도 추상클래스가 되어야 한다)

*추상 클래스와 인터페이스의 차이점은 그 목적이라고 할 수 있다. **추상 클래스**는 기본적으로 클래스이며 이를 **상속, 확장하여 사용하기 위한 것**이다.*

*반면 **인터페이스**는 해당 인터페이스를 구현한 객체들에 대한 **동일한 사용 방법과 동작을 보장하기 위해 사용**한다. 어떤 객체가 특정 인터페이스를 구현했다면, 그 객체는 **인터페이스의 기능을 모두 갖추고 있음을 보장**한다. 그리고 인터페이스를 통해 **해당 객체를 제어할 수 있음을 보장**하는 것이다.*

참고로 추상 클래스는 다중 상속이 불가능 하지만, 인터페이스는 다중 구현이 가능하다. (인터페이스끼리는 다중 상속 가능) 

다중 상속은 아래와 같이 여러 개의 부모 클래스를 상속 받는 것이다. 

```java
class MyVehicle extends Car, Plane {
	@Overrride 
	public void drive() {
		super.drive(); 	
	}
}
```

만약 Car, Plane 코드에서 모두 drive() 라는 메서드를 갖고 있다면, 어떤 메서드가 실행될까? 애매해다. 이것이 다중 상속의 모호성이며, 자바는 이를 과감하게 배제하여 클래스 간의 다중 상속이 불가능하다. 

반면에, 인터페이스는 아래와 같이 여러 개의 인터페이스를 구현할 수 있다. 

```java
class Car implements Vehicle, Engine {
	@Overrride 
	public void drive() {
		super.drive();
	}
}
```

마치 여러 개를 상속 받는 것처럼 보이기도 한다. 이렇게 외관성으로 헷갈리기 때문에 인터페이스가 다중 상속의 문제점을 해결하기 위해 존재한다는 오해를 사기도 한다. 

*하지만, 상속은 **부모 클래스의 기능을 이용하거나 확장하기 위해 사용**하는 것이고, 인터페이스는 해당 인터페이스를 **구현한 객체들에 대해 동일한 동작을 약속하기 위해 사용**하는 것이다.* 

# sealed class vs. sealed interface

sealed class, sealed interface 모두 ‘제한된 클래스 계층 구조’를 만들 수 있는 코틀린의 기능이다. 

**둘 다 가능한 서브 타입의 유한 집합을 정의하는 데 사용되며, 선언된 계층 구조 외부에서 추가적인 자식 클래스를 정의하는 걸 방지한다.**

## sealed class

sealed class는 직접적으로 인스턴스화 될 수 없으며, 주로 **제한된 클래스의 계층 구조**를 나타내는 데 사용된다. 

코틀린 버전 1.5 이전에는 같은 파일 및 패키지 내에서만 자식 클래스를 정의할 수 있었지만, 1.5 이상 버전에서는 이 제약 조건이 사라져 **같은 모듈 내에만 있다면 어디서든 자식 클래스를 정의**할 수 있다. 

하나의 예시로 Result의 자식 클래스를 Success, Error로 제한할 수 있다. 

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val error: Exception) : Result<Nothing>()
}
```

sealed class를 이용하면 when 구문에서 타입 검사를 더 안전하고 효율적으로 수행할 수 있다. 

```kotlin
sealed class Animal
class Dog(val name: String): Animal()
class Cat(val name: String): Animal()

fun makeSound(animal: Animal) = when (animal) {
    is Dog -> println("${animal.name} says woof!")
    is Cat -> println("${animal.name} says meow!")
} // 이외에는 자식 클래스가 없음을 보장하므로 else 구문 필요 X 

val myDog = Dog("Rufus")
val myCat = Cat("Whiskers")
makeSound(myDog) // outputs: "Rufus says woof!"
makeSound(myCat) // outputs: "Whiskers says meow!"
```

## sealed interface

sealed interface는 sealed class와 비슷하지만, 클래스가 아니라 **인터페이스의 제한된 집합**을 나타내는 데 사용된다.

아래 예시에서 각 서브 타입은 어플리케이션에서 발생할 수 있는 State를 나타낸다. 

```kotlin
sealed interface State {
    object Idle : State
    data class Loading(val message: String) : State
    data class Error(val error: Throwable) : State
    data class Success(val data: Any) : State
}
```

- sealed interface는 생성자를 정의할 수 없지만, 프로퍼티는 정의할 수 있다.
- sealed interface는 오직 추상 메서드만 가질 수 있지만, 필요에 따라 디폴트 구현도 가능하다.
- sealed interface는 클래스와 object에 의해 구현될 수 있지만, 다른 인터페이스나 sealed 인터페이스에 의해서는 구현 불가능하다.
- sealed interface는 (다른 클래스가 구현할 수 있는) **관련된 기능의 집합을 정의할 때** 자주 사용된다.

```kotlin
sealed interface Animal {
    val name: String
    fun makeSound()
}

class Dog(override val name: String): Animal {
    override fun makeSound() {
        println("$name says woof!")
    }
}

class Cat(override val name: String): Animal {
    override fun makeSound() {
        println("$name says meow!")
    }
}

val myDog = Dog("Rufus")
val myCat = Cat("Whiskers")
myDog.makeSound() // outputs: "Rufus says woof!"
myCat.makeSound() // outputs: "Whiskers says meow!"
```

# 요약

sealed class 

- 가능한 자식 클래스의 종류를 제한하고 싶을 때 (제한된 클래스 계층 구조)

sealed interface 

- 관련된 기능의 집합을 정의할 때
- 가능한 인터페이스의 구현을 제한하고 싶을 때 (제한된 인터페이스 집합)

sealed class, sealed interface 모두 제한된 클래스 계층 구조를 정의하고, 선언된 계층 구조 외부에서 추가적인 자식 클래스를 정의하는 걸 방지할 수 있다. 가능한 서브 타입의 종류를 제한하고 싶을 때 유용하며, 코드의 신뢰성을 더욱 높이고 오류의 발생 가능성은 낮출 수 있다. 

# 참고자료

- [Interface와 Abstract class 의 차이점 - 인프런](https://www.inflearn.com/questions/236439/interface와-abstract-class-의-차이점)
- [자바의 추상 클래스와 인터페이스](https://brunch.co.kr/@kd4/6)
- [☕ 인터페이스 vs 추상클래스 용도 차이점 - 완벽 이해](https://inpa.tistory.com/entry/JAVA-☕-인터페이스-vs-추상클래스-차이점-완벽-이해하기)
- [Sealed Class vs Sealed Interface in Kotlin](https://medium.com/@manuchekhrdev/sealed-class-vs-sealed-interface-in-kotlin-47222335040a)

