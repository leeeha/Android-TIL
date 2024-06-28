# 얕은 복사

- **주소 값을 복사**하는 것
- 원본 객체와 복사본 객체는 **같은 메모리 주소를 참조**한다.
- 따라서, 복사본 객체의 속성이 변경되면 **원본 객체도 같이 변경**된다.

# 깊은 복사

- 새로운 메모리 공간에 **객체의 모든 값을 복사**하는 것
- 원본 객체는 그대로 두고, **새로운 메모리 공간에 원본 객체의 모든 내용을 복사**한다.
- 원본과 복사본이 **서로 다른 메모리 주소를 참조**하기 때문에, **복사본 객체가 변경되어도 원본 객체는 변경되지 않는다.**

# 얕은 복사의 문제점

자바(코틀린)에서 기본적으로 `=` 연산자를 사용하여 객체를 복사하면, 기본적으로 ‘얕은 복사’가 수행된다. 따라서, **복사본 객체가 변경되면 원본 객체도 변경**된다. 

```kotlin
import org.junit.jupiter.api.Test 

data class Car(
	private val name: String = "", 
	var position: Int = 0
) {
	fun move() {
		position++
	}
}

class DeepCopyTest {
	@Test
	fun shallowCopyTest() {
		val car = Car("car")
		val copyCar = car // shallow copy
		copyCar.move() 
	}
}
```

![image](https://github.com/leeeha/Android-TIL/assets/68090939/3a11ac9e-599b-4b93-8b40-ae446b69a6f4)

**동일한 메모리 주소를 참조**하는 car, copyCar 객체는 **하나가 변경되면 다른 하나도 같이 변경**된다. 

위의 예시에서도 copyCar 객체의 position을 변경했는데, car 객체의 position까지 변경되는 문제가 발생한다.

## 깊은 복사 수행하는 방법

### data class의 copy() 함수

코틀린의 데이터 클래스는 copy() 함수로 깊은 복사를 수행할 수 있다. 

```kotlin
import org.junit.jupiter.api.Test 

data class Car(
	private val name: String = "", 
	var position: Int = 0
) {
	fun move() {
		position++
	}
}

class DeepCopyTest {
	@Test
	fun deepCopyTest() {
		val car = Car("car")
		val copyCar = car.copy() // deep copy 
		copyCar.move() 
	}
}
```

![image](https://github.com/leeeha/Android-TIL/assets/68090939/c0023f13-121d-4cdf-bd2b-3beef482de8a)

car, copyCar 객체가 참조하는 메모리 주소가 다르므로, copyCar 객체를 변경해도 원본 객체인 car에는 영향을 미치지 않는다. 

### 깊은 복사를 수행하는 함수 직접 구현

데이터 클래스 아닌 일반 클래스에 대해서는 깊은 복사를 수행하는 함수를 직접 구현해야 한다. 

```kotlin
import org.junit.jupiter.api.Test 

class Car(
	private val name: String = "", 
	var position: Int = 0
) {
	fun move() {
		position++
	}

	fun copy(name: String = this.name, 
			 position: Int = this.position) = Car(name, position)
}

class DeepCopyTest {
	@Test
	fun deepCopyTest() {
		val car = Car("car")
		val copyCar = car.copy()
		copyCar.move() 
	}
}
```

![image](https://github.com/leeeha/Android-TIL/assets/68090939/8f9a9872-1316-4637-8a30-6ea11b11731a)

**원본과 복사본 객체의 주소가 다르므로, 둘 중에 하나가 변해도 서로에게 영향을 미치지 않는다.**

## 코틀린 컬렉션의 방어적 복사

컬렉션에서 ‘깊은 복사’를 수행하는 대표적인 방법은, **일급 컬렉션을 만들어서 방어적 복사를 구현**하는 것이다. 

### 일급 컬렉션이란?
    
일급 컬렉션(First Class Collection)이란, 객체지향 프로그래밍에서 객체의 책임을 적절히 분배하기 위한 개념 중 하나이다. 이는 **컬렉션을 담는 클래스**를 만들고, **그 클래스에 컬렉션에 대한 연산을 위임하는 것**을 의미한다. 

자바나 코틀린에서의 일급 컬렉션은 다음과 같은 특징을 갖는다. 

- 해당 클래스는 **하나의 컬렉션만 인스턴스 변수로** 가진다. 
- 해당 컬렉션에 대한 **연산은 모두 해당 클래스에서** 이루어진다.
- 해당 클래스는 **불변성을 유지**한다.

일급 컬렉션을 사용하는 이유는 다음과 같다. 

- **비즈니스에 종속적인 자료구조**를 만들 수 있다.
- 객체의 **불변성을 유지**할 수 있다.
- **그룹을 표현**하는데 의미를 부여할 수 있다.
- 컬렉션에 대한 연산을 해당 클래스에 **캡슐화**할 수 있다.

예를 들어, 자바에서의 일급 컬렉션 예시는 다음과 같다. 

```kotlin
public class Cars {
	private final List<Car> cars;

	public Cars(List<Car> cars) {
		this.cars = cars;
	}

	// cars에 대한 연산 메소드들...
}
```

```kotlin
class Cars(private val cars: List<Car>) {
	// cars에 대한 연산 메소드들...
}
```

위와 같이 일급 컬렉션을 사용하면, **컬렉션에 대한 연산을 캡슐화하고, 불변성을 보장하며, 특정 비즈니스 로직에 맞는 자료구조를 만들 수** 있다. 이러한 일급 컬렉션은 객체지향 프로그래밍의 원칙 중 하나인 '객체에게 책임을 최대한 맡기는' 원칙을 잘 지키는 방법 중 하나이다.
    
### 방어적 복사란?

생성자로 받은 가변 데이터가 **외부의 조작에 의해 변경되는 것을 막기 위해** 복사본을 이용하는 방법

컬렉션을 방어적 복사할 때는 아래 3가지를 모두 수행해야 완벽한 방어적 복사가 된다.

- 일급 컬렉션 ‘내부로 들어오는’ 객체에 대한 방어적 복사 수행
- 일급 컬렉션의 내부 객체가 가변 객체라면 ‘컬렉션 내부 객체’에 대한 방어적 복사 수행
- 일급 컬렉션 ‘외부로 나가는’ 객체에 대한 방어적 복사 수행

### 일급 컬렉션 ‘내부로 들어오는’ 객체에 대한 방어적 복사

```kotlin
import org.junit.jupiter.api.Test 

class Car(val name: String = "", var position: Int = 0)

class Cars(val cars: List<Car>)

class DeepCopyTest {
	@Test
	fun listCopyTest() {
		val carList = mutableListOf(Car("carA"))
		val cars = Cars(carList) // carA, carB
		carList.add(Car("carB")) // carA, carB
	}
}
```

위의 코드를 실행시키면 cars 객체를 조작하지 않았는데도, **carList가 변경되면 cars 객체 안에 있는 리스트도 같이 변경**된다. 

그 이유는, **Cars 객체에 carList가 들어갈 때 ‘얕은 복사’가 일어나기 때문**이다. 

즉, carList와 Cars 객체 내부의 cars 리스트가 **동일한 메모리 주소를 참조**하기 때문에, carList를 조작하면 cars 또한 변경되는 것이다. 

![image](https://github.com/leeeha/Android-TIL/assets/68090939/9e069d6e-a9ea-4c37-a024-d49602b5a059)

위의 결과를 보면 carList에 carB 객체를 추가하면, cars도 같이 변경된다는 걸 알 수 있다. 

이를 해결하기 위해 Cars 클래스의 코드를 다음과 같이 바꾸자.

```kotlin
class Cars(cars: List<Car>) {
	val cars: List<Car> = cars.toList()
}
```

생성자 부분을 잘 보면, `val cars` → `cars` 이렇게 바뀐 것을 알 수 있다. **즉, cars를 파라미터 역할만 하게 만들고, value는 생성하지 않게 하는 것**이다.

그리고 클래스 내부에서 `cars` 프로퍼티를 생성하는데, 이때 파라미터 `cars`에 대해 `toList()`를 실행시키면 **새로운 리스트가 반환되며 ‘깊은 복사’가 수행된다.** 

자바로 치면 `new List(cars)` 를 수행하는 것과 비슷하다. 

```kotlin
import org.junit.jupiter.api.Test 

data class Car(val name: String = "", var position: Int = 0){
	fun move() {
		position++
	}
}

class Cars(cars: List<Car>) {
	val cars: List<Car> = cars.toList()
}

class DeepCopyTest {
	@Test
	fun listCopyTest() {
		val carList = mutableListOf(Car("carA"))
		val cars = Cars(carList) // carA 
		carList.add(Car("carB")) // carA, carB
	}
}
```

![image](https://github.com/leeeha/Android-TIL/assets/68090939/357d1153-ac12-4940-8048-f4830f084838)

일급 컬렉션 ‘내부로 들어오는’ 객체에 대한 방어적 복사를 수행했기 때문에, carList를 변경해도 cars는 변하지 않는다는 걸 확인할 수 있다. 

### 컬렉션 ‘내부 객체’에 대한 방어적 복사

Car는 가변 객체여서 `carList[0].move()` 를 수행하면 `cars` 내부에 있는 객체의 값도 바뀐다. 

Car가 불변 객체라면 위의 방법으로도 문제가 발생하지 않지만, **가변 객체인 경우에는 내부 객체에 대해서도 ‘깊은 복사’를 수행해야 한다.** 

```kotlin
import org.junit.jupiter.api.Test 

data class Car(val name: String = "", var position: Int = 0){
	fun move() {
		position++
	}
}

class Cars(cars: List<Car>) {
	val cars: List<Car> = cars.toList()
}

class DeepCopyTest {
	@Test
	fun listCopyTest() {
		val carList = mutableListOf(Car("carA"))
		val cars = Cars(carList) // cars[0] (position: 1)
		carList[0].move() // carList[0] (position: 1) 
	}
}
```

![image](https://github.com/leeeha/Android-TIL/assets/68090939/caf35a1c-ab0d-422f-abf5-de47b4cda74e)

carList[0] 객체의 position 값을 바꿨는데, cars 안의 객체 값도 바뀐 것을 확인할 수 있다. 

이 문제를 해결하기 위해 **List 내부 객체에 대한 깊은 복사를 수행**한다. 바로 다음과 같이 말이다. 

```kotlin
class Cars(cars: List<Car>){
	val cars: List<Car> = cars.map { it.copy() }
}
```

```kotlin
import org.junit.jupiter.api.Test 

data class Car(val name: String = "", var position: Int = 0){
	fun move() {
		position++
	}
}

class Cars(cars: List<Car>){
	val cars: List<Car> = cars.map { it.copy() }
}

class DeepCopyTest {
	@Test
	fun listCopyTest() {
		val carList = mutableListOf(Car("carA"))
		val cars = Cars(carList) // cars[0] (position: 0)
		carList[0].move() // carList[0] (position: 1)
	}
}
```

![image](https://github.com/leeeha/Android-TIL/assets/68090939/6f829f5b-ffeb-4895-aa1e-73137ebc9d31)

carList의 내부 객체가 바뀌어도 cars 리스트의 내부 객체에는 영향을 미치지 않는다.

Car 데이터 클래스의 copy() 메서드를 이용하여, 내부 객체들에 대해서도 깊은 복사를 수행했기 때문이다. 

### 일급 컬렉션 ‘외부로 나가는’ 객체에 대한 방어적 복사

위에서 알아본 것은 밖에서 들어오는 List에 대한 방어적 복사였다. 

마찬가지로 외부에서 Cars 클래스 내부의 List에 직접 접근하여 조작하는 것. 즉, 일급 컬렉션 ‘외부로 나가는’ 객체에 대한 방어적 복사도 구현해줘야 한다. 

그렇지 않으면, Cars 클래스의 내부 객체인 cars[0]에 접근하여 move() 함수를 호출하면, position이 변경될 것이다. 

```kotlin
import org.junit.jupiter.api.Test 

data class Car(val name: String = "", var position: Int = 0){
	fun move() {
		position++
	}
}

class Cars(cars: List<Car>){
	val cars: List<Car> = cars.map { it.copy() }
}

class DeepCopyTest {
	@Test
	fun listCopyTest() {
		val carList = mutableListOf(Car("carA"))
		val cars = Cars(carList) // carList[0] (position: 1)
		cars.cars[0].move() // cars[0] (position: 1)
	}
}
```

![image](https://github.com/leeeha/Android-TIL/assets/68090939/617c5d21-80f1-48ac-ab3f-edab849c4030)

cars 리스트의 내부 객체를 변경했는데, carList 내부 객체도 변경된 것을 알 수 있다. 

다음과 같이 코드를 개선할 수 있다. 

```kotlin
class Cars(cars: List<Car>){
	private val _cars: List<Car> = cars.map { it.copy() }
	val cars: List<Car>
		get() = _cars.map { it.copy() }
}
```

외부에서 받은 파라미터는 **내부 객체까지 복사**하여 `_cars` 리스트에 담는다. 

그리고 `val cars` 또한 **내부 객체까지 복사하여 외부에 노출**시키면, 컬렉션에 대한 개선된 방어적 복사가 완성된다. 

```kotlin
import org.junit.jupiter.api.Test 

data class Car(val name: String = "", var position: Int = 0){
	fun move() {
		position++
	}
}

class Cars(cars: List<Car>){
	private val _cars: List<Car> = cars.map { it.copy() }
	val cars: List<Car>
		get() = _cars.map { it.copy() }
}

class DefensiveCopyTest {
	@Test
	fun listCopyTest() {
		val carList = mutableListOf(Car("carA"))
		val cars = Cars(carList) // carList[0] (position: 0)
		cars.cars[0].move() // _cars[0] (position: 0)
	}
}
```

![image](https://github.com/leeeha/Android-TIL/assets/68090939/b7ecf721-f104-4d6e-a4fc-d2094468633f)

Cars 객체 내부의 public 프로퍼티인 `cars`의 내부 객체를 변경해도, **원본 객체인 `_cars`의 내부 객체는 변하지 않는다**는 걸 알 수 있다!! 

## 방어적 복사 요약 

### 일급 컬렉션 '내부로 들어오는 객체'에 대한 방어적 복사

```kotlin
import org.junit.jupiter.api.Test 

data class Car(val name: String = "", var position: Int = 0){
	fun move() {
		position++
	}
}

class Cars(cars: List<Car>) {
	val cars: List<Car> = cars.toList()
}

class DeepCopyTest {
	@Test
	fun listCopyTest() {
		val carList = mutableListOf(Car("carA"))
		val cars = Cars(carList) // carA 
		carList.add(Car("carB")) // carA, carB
	}
}
```

### 컬렉션 '내부 객체'에 대한 방어적 복사

```kotlin
import org.junit.jupiter.api.Test 

data class Car(val name: String = "", var position: Int = 0){
	fun move() {
		position++
	}
}

class Cars(cars: List<Car>){
	val cars: List<Car> = cars.map { it.copy() }
}

class DeepCopyTest {
	@Test
	fun listCopyTest() {
		val carList = mutableListOf(Car("carA"))
		val cars = Cars(carList) // cars[0] (position: 0)
		carList[0].move() // carList[0] (position: 1)
	}
}
```

### 일급 컬렉션 '외부로 나가는 객체'에 대한 방어적 복사

```kotlin
import org.junit.jupiter.api.Test 

data class Car(val name: String = "", var position: Int = 0){
	fun move() {
		position++
	}
}

class Cars(cars: List<Car>){
	private val _cars: List<Car> = cars.map { it.copy() }
	val cars: List<Car>
		get() = _cars.map { it.copy() }
}

class DefensiveCopyTest {
	@Test
	fun listCopyTest() {
		val carList = mutableListOf(Car("carA"))
		val cars = Cars(carList) // carList[0] (position: 0)
		cars.cars[0].move() // _cars[0] (position: 0)
	}
}
```

# 참고 자료

[[Kotlin/Java] 얕은복사 vs 깊은복사, Collection의 방어적 복사](https://seosh817.tistory.com/163)
