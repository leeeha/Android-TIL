#  학습 동기 

솝트 앱잼에서 UiState라는 sealed interface를 사용했는데, 제네릭에서 사용되는 in, out 키워드의 정체가 무엇인지 궁금해졌다. 

```kotlin
sealed interface UiState<out T> { // 여기서 out을 안 쓰면 에러 발생! Why?? 
    object Empty : UiState<Nothing>

    object Loading : UiState<Nothing>

    data class Success<T>(
        val data: T
    ) : UiState<T>

    data class Failure(
        val msg: String
    ) : UiState<Nothing>
}
```

![image](https://github.com/leeeha/Android-TIL/assets/68090939/6da59311-a547-40a8-8e4e-df552fcdc5a2)

# 제네릭이란?

클래스, 인터페이스, 함수 등에서 **일반화 된 타입 (형식 매개변수)을 사용함으로써 중복되는 코드를 줄일 수 있는 유용한 기능**이다. 

```kotlin
fun <T> wrap(value: T) {
    println(value)
}

fun main() {
    wrap(1)
    wrap("abc")
    wrap(1.3)
}
```

위의 코드에서 볼 수 있듯이, 형식 매개변수 T를 사용해 정수, 실수, 문자열 타입을 모두 함수의 인자로 받을 수 있음을 알 수 있다. 


## 불변성 (Invariance) - 디폴트 

코틀린에서 제네릭을 사용할 때 in, out 키워드를 붙이지 않으면 기본적으로 불변성(invariance)이다. 

불변성이란 무엇일까? 

아래 코드에서 Cat은 Animal의 자식 클래스임에도 불구하고

```kotlin
val cats: Array<Cat> = arrayOf(Cat(), Cat())
val animals: Array<Animal> = cats // error 
```

다음과 같은 에러가 발생한다. 

>*Error - Type mismatch: inferred type is Array\<Cat> but Array\<Animal> was expected*

**Array\<Animal>을 기대했건만, Array\<Cat>을 줘? 난 자식은 받지 않아.** 라는 의미이다. 

이처럼 제네릭 타입을 가지는 클래스, 인터페이스에 대해서는 기본적으로 **클래스의 상속 관계가 제네릭에서는 상속 관계로 유지되지 않는** 불변성(invariance)이 존재한다. 

즉, **A가 B를 상속받아도 Class\<A>는 Class\<B>를 상속받지 않는다**는 뜻이다. 

## 공변성 (Covariance) - out 키워드

불변성(Invariance)에 대한 제약은 코드의 안전성을 보장하는 데 큰 도움을 준다.

그러나 Invariance가 언제나 만능은 아니다. Invariance가 필요하지 않은 상황을 살펴보자.

```kotlin
fun copyFromTo(from: Array<Animal>, to: Array<Animal>) {
    for (i in from.indices) {
        to[i] = from[i]
    }
}

fun main() {
    val animals: Array<Animal> = arrayOf(Animal(), Animal())
    val cats: Array<Cat> = arrayOf(Cat(), Cat())

    copyFromTo(cats, animals) // error 
}
```

>*Error - Type mismatch: inferred type is Array<Cat> but Array<Animal> was expected*

위 코드는 cats를 animals에 copy하는 코드이다.

Array\<Animal>인 animals 배열에 자식인 Cat 타입의 원소를 추가하는 것은 전혀 문제되지 않는다. (업캐스팅)

그러나, **A가 B를 상속받아도 Class\<A>는 Class\<B>를 상속받지 않는다** 라는 불변성의 원리로 인해 컴파일 에러가 발생한다. 

이를 해결하기 위해서는 **A가 B를 상속받으면 Class\<A>는 Class\<B>를 상속받는다** 로 바꿔주어야 한다. 이를 **공변성(Covariance)** 이라고 한다.

그리고 이러한 Covariance로 변환하기 위해 사용하는 키워드가 바로 **out**이다. 

아래 코드처럼 out 키워드를 from 매개변수의 제네릭 타입에 적용하면, Array\<Cat>은 Array\<Animal>을 상속받게 되므로 컴파일 에러가 발생하지 않는다. 

```kotlin
fun copyFromTo(from: Array<out Animal>, to: Array<Animal>) {
    for (i in from.indices) {
        to[i] = from[i]
    }
}

fun main() {
    val animals: Array<Animal> = arrayOf(Animal(), Animal())
    val cats: Array<Cat> = arrayOf(Cat(), Cat())

    copyFromTo(cats, animals) // Cat -> Animal (업캐스팅) 
}
```

out 키워드를 붙여줌으로써 공변성의 원리를 이용해 불필요한 불변성 문제를 피할 수 있다.

이렇게 제네릭 타입이 **공변성**을 가지게 되면 **값에 대한 read만 가능하고, write이 불가능**해진다. 

이유는 다음과 같다. 

**[READ]**

Array\<out Animal> 타입인 from은 부모가 Animal인 것을 컴파일러가 인지하고 있다.

그렇다면 **from의 값을 read 할 때에는 Animal, Dog, Cat 중 하나**인 것도 알 것이며,

이는 **모두를 포함하는 Animal로 할당**해 줄 수 있으므로 read를 할 때에는 문제가 발생하지 않는다. 

**[WRITE]**

그러나 write의 경우에는 좀 다르다.

공변성을 사용한 from에게 실제로는 Array\<Cat>을 넘겨줬다.

그러나 메소드에서 from은 Array\<out Animal>로 선언되어 있으므로,

**실제 from의 타입이 Array\<Animal>인지, Array\<Cat>인지, Array\<Dog>인지 모른다.**

**from의 실제 타입을 모르는 메소드가 함부로 값을 변경할 수 없으므로**, 공변성을 갖는 제네릭 타입의 값에 대한 **write는 불가능**하다. 

### out 키워드로 알아보는 Array와 List의 차이점

```java
public class Array<T> // 읽기, 쓰기 (O) -> Mutable
public interface List<out E> // 읽기(O), 쓰기(X) -> Immutable
```

out 키워드가 붙은 **List** 컬렉션은 공변성(Invariance)에 의해 값을 변경할 수 없다. **(Immutable)** 

반면에, 형식 매개변수에 아무런 키워드가 붙어 있지 않은 **Array**는 값을 변경할 수 있다. **(Mutable)** 

```kotlin
fun myAnimals(animals: Array<Animal>) {
    // ERROR: Cat 타입에 Dog 타입을 대입할 수 없다! 
    animals[0] = Dog() 
}

fun main() {
    val cats: Array<Cat> = arrayOf(Cat(), Cat())
    myAnimals(cats)
}
```

```kotlin
fun myAnimals(animals: List<Animal>) { 
    println(animals[0]) // 읽기(O), 쓰기(X) 
}

fun main() {
    val cats: List<Cat> = listOf(Cat(), Cat())
    myAnimals(cats) 
}
```

## 반공변성 (Contravariance) - in 키워드

```kotlin
fun copyFromTo(from: Array<out Animal>, to: Array<Animal>) {
    for (i in from.indices) {
        to[i] = from[i]
    }
}

fun main() {
    val anys: Array<Any> = arrayOf(Any(), Any())
    val cats: Array<Cat> = arrayOf(Cat(), Cat())

    copyFromTo(cats, anys) // 두번째 인자에서 에러 발생 
}
```

>*Error - Type mismatch: inferred type is Array\<Any> but Array\<Animal> was expected*

위의 에러를 해결하려면 어떻게 해야 될까? 

바로 **A가 B를 상속받으면 Class\<B>는 Class\<A>를 상속받는다** 로 바꿔주어야 한다.

이를 공변성과 반대되는 방향이므로 **반공변성(Contravariance)** 이라고 부른다. 즉, **클래스의 상속관계가 제네릭에서는 반대로 작용하는 것**이다.

그리고 이러한 Contravariance로 변환하기 위해 사용하는 키워드가 바로 **in**이다. 

in 키워드를 to 매개변수의 제네릭 타입에 적용하면, 클래스의 상속관계가 반대로 작용하여 **Array\<Any>은 Array\<Animal>을 상속 받게 되므로** 에러가 발생하지 않는다.

```kotlin
fun copyFromTo(from: Array<out Animal>, to: Array<in Animal>) {
    for (i in from.indices) {
        to[i] = from[i] // from: Cat, to: Any 
    }
}

fun main() {
    val anys: Array<Any> = arrayOf(Any(), Any())
    val cats: Array<Cat> = arrayOf(Cat(), Cat())
    
	  // Cat -> Animal (업 캐스팅) - out 키워드 
	  // Any -> Animal (다운 캐스팅) - in 키워드 
    copyFromTo(cats, anys) 
}
```

그러나, 반공변성의 경우에는 **write은 가능하지만 read는 불가능**하다. 

```kotlin
fun copyFromTo(from: Array<out Animal>, to: Array<in Animal>) {
    for (i in from.indices) {
        to[i] = from[i]
    }

	// Error - Type mismatch: inferred type is Any? but Animal was expected
    val any: Animal = to[0] // Any -> Animal (다운 캐스팅 불가)
}

fun main() {
    val anys: Array<Any> = arrayOf(Any(), Any())
    val cats: Array<Cat> = arrayOf(Cat(), Cat())

    copyFromTo(cats, anys)
}
```

이유는 간단하다. **반공변성**으로 인해 **자식 타입에 부모 타입을 대입**할 수 있게 되었으므로, 제네릭 타입으로 해당 값을 read 할 때 타입이 **부모 타입이면 문제가 발생**하기 때문이다. (다운 캐스팅 불가하므로)

## Result 예제 

아래 코드의 Result<**out** T>에서 out 키워드가 빠진다면 어떤 문제가 발생하는지 이제는 알 수 있다.

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()

    object Empty : Result<Nothing>()

    companion object {
        fun <E> successOrEmpty(list: List<E>): Result<List<E>> {
            return if (list.isEmpty()) Empty else Success(list)   
        }
    }
}
```

companion object 안에 있는 successOrEmpty 메소드를 보면, list.isEmpty일 경우 Empty object를 return한다. 

이때, Empty Object는 `Result<Nothing>`의 인스턴스이며, successOrEmpty의 return 값은 `Result<List<E>>` 이다. 

Nothing 타입이 List\<E>의 하위 타입일 때,

Result\<Nothing> 타입도 Result\<List\<E>>의 하위 타입이 되게 만드는 공변성이 필요하다.

따라서, Result\<out T> 이렇게 형식 매개변수 앞에 out 키워드가 붙은 것이다. 

out 키워드를 작성하지 않으면, 다음과 같은 에러가 발생한다.

```kotlin
sealed class Result<T> {
    data class Success<T>(val data: T) : Result<T>()

    object Empty : Result<Nothing>()

    companion object {
        // Type mismatch: inferred type is Result.Empty but Result<List<T>> was expected
        fun <E> successOrEmpty(list: List<E>): Result<List<E>> {
            return if (list.isEmpty()) Empty else Success(list)
        }
    }
}
```

## UiState 예제 

```kotlin
sealed interface UiState<out T> { // 여기서 out을 안 쓰면 에러 발생! 
    object Empty : UiState<Nothing>

    object Loading : UiState<Nothing>

    data class Success<T>(
        val data: T
    ) : UiState<T>

    data class Failure(
        val msg: String
    ) : UiState<Nothing>
}
```

![image](https://github.com/leeeha/Android-TIL/assets/68090939/6da59311-a547-40a8-8e4e-df552fcdc5a2)

- UiState.Loading은 `UiState<Nothing>`의 인스턴스이다.
- **Nothing은 T의 하위 타입 → UiState\<Nothing>은 UiState\<T>의 하위 타입 (공변성)**
- T 앞에 out 키워드 붙여서 공변성이 적용되게 한다.
    - A가 B를 상속받을 때, Class\<A>도 Class\<B>를 상속 받을 수 있게 하는 성질
    - 부모, 자식 클래스 간의 상속 관계가 제네릭에서도 유지되게 만드는 성질

# 참고자료 

- https://medium.com/mj-studio/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%A0%9C%EB%84%A4%EB%A6%AD-in-out-3b809869610e
- [[Android, Kotlin] 제네릭의 in, out 키워드는 무엇일까?](https://hungseong.tistory.com/30)

