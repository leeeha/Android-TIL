# 범위 지정 함수란?

범위 지정 함수 (scope function)이란, **특정 객체에 대한 작업을 블록 안에 넣어서 실행할 수 있도록 하는 함수**이다. 블록은 **특정 객체에 대한 작업의 범위**가 되며, 따라서 **범위 지정 함수**라고 부른다.

특정 객체에 대한 작업을 블록 안에 넣게 되면, 더욱 간결하고 유연하게 코드를 작성할 수 있다. 코틀린에서는 apply, run, with, let, also 총 5가지의 범위 지정 함수를 지원한다. 

범위 지정 함수는 다른 말로 **수신객체 지정 함수**라고도 불린다. **수신객체를 명시하지 않거나 it을 호출**하는 것만으로도 람다식 안에서 **수신객체의 메서드를 호출**할 수 있기 때문이다. 

이것이 가능한 이유는 범위 지정 함수의 람다식에서 **수신객체를 람다식의 매개변수 또는 수신객체로 사용**했기 때문이다.

도대체 이게 무슨 말이냐고? 예시와 함께 이해해보자! 

예시에 사용할 데이터 클래스는 다음과 같다.

```kotlin
data class Person(
    var name: String = "",
    var age: Int = 0,
    var temperature: Float = 36.5f
)
```

5가지 스코프 함수에 대한 요약 설명을 먼저 쓱~ 보자! 개발하면서 헷갈릴 때 치팅 시트로 활용하면 좋을 거 같다.

<img width="700" src="https://github.com/user-attachments/assets/02812fac-7434-41f8-869d-226b4e49dfb1"/>

# apply 

- **수신객체의 내부 프로퍼티를 변경**한 후에 **수신객체 자체를 반환**하는 함수 
- 객체를 생성하며 **다양한 프로퍼티를 설정해야 하는 경우**에 자주 사용된다. 

<img width="600" alt="스크린샷 2024-07-21 오후 5 45 48" src="https://github.com/user-attachments/assets/8621710b-8146-4d87-83f0-401d13ad6d8b">

apply의 수신객체를 block 람다식의 수신객체로 지정하고 있다. 

따라서, 람다식 내부에서 수신객체에 대한 명시 없이도 수신객체의 메서드를 호출할 수 있다. 

```kotlin
val person = Person().apply {
    name = "Haeun"
    age = 25
    temperature = 36.8f
}
```

프로퍼티 설정할 때마다 person을 작성하지 않아도 돼서 가독성이 개선되었다!

apply를 적용하지 않으면 다음과 같다. 

```kotlin
val person = Person()
person.name = "Haeun"
person.age = 25 
person.temperature = 36.8f
```

# run 

- apply와 다르게, 수신 객체가 아니라 **run 블록의 마지막 라인을 반환**한다.
- **수신객체에 대해 특정 동작을 수행한 다음 결과값을 리턴 받아야 할 때** 주로 사용한다.

<img width="600" alt="스크린샷 2024-07-21 오후 5 46 31" src="https://github.com/user-attachments/assets/5817d4ba-e17e-4d7d-a83a-e94fcd7918a0">

```kotlin
data class Person(
    var name: String = "",
    var age: Int = 0,
    var temperature: Float = 36.5f
){
    fun isSick(): Boolean = temperature > 37.5f
}
```

```kotlin
fun main(){
    val person = Person(name = "Haeun", age = 25, temperature = 36.5f)
    val isPersonSick = person.run {
        temperature = 37.2f
        isSick() // return 값 
    }

    println("$isPersonSick") // false
}
```

수신객체를 지정하지 않은 경우에는 블록 내부에서 수신객체를 명시해줘야 한다. 

```kotlin
val person = Person("Haeun", 25, 36.5f)
val isPersonSick = run {
    person.temperature = 37.2f 
    checkPersonIsSick(person)
}
```

# with

- run과 동일하게, **수신객체에 대해 특정 동작을 수행**한 다음 **블록의 마지막 라인을 반환**한다.
- run과 다르게, **확장 함수가 아니라 매개변수로 수신객체를 받아서** 사용한다.

<img width="700" alt="스크린샷 2024-07-21 오후 5 46 57" src="https://github.com/user-attachments/assets/00499d30-5930-43ab-9a91-793987a194ad">

```kotlin
fun main() {
    val person = Person(name = "Haeun", age = 25, temperature = 36.5f)
    val isPersonSick = with(person) { // 함수의 매개변수로 수신객체를 받는다.
        temperature = 38.0f
        isSick() // return 값
    }

    println("$isPersonSick") // true
}
```

# let 

- run, with와 동일하게, **수신객체에 대해 특정 동작을 수행**한 다음 **블록의 마지막 라인을 반환**한다.
- 차이점은 **수신객체에 접근할 때 it을 사용해야 한다**는 것이다. 

<img width="600" alt="스크린샷 2024-07-21 오후 5 47 21" src="https://github.com/user-attachments/assets/6d04701e-934a-49eb-8e7e-e4ca4971cc61">

let은 다음과 같은 상황에서 자주 사용된다.

- null 체크 이후 코드를 실행해야 하는 경우 
- nullable한 수신객체를 non-null 타입으로 변환해야 하는 경우 

```kotlin
fun main() {
    var person: Person? = null
    val isReserved = person?.let { it: Person ->
        reserveMovie(it)
    }
}
```

# also 

- apply와 동일하게, **수신객체 자신을 반환**한다. 
- apply와 다르게, 수신객체를 전혀 사용하지 않거나 **수신 객체의 속성을 변경하지 않는 경우**에 주로 사용한다.
- 예를 들어, **객체의 사이드 이펙트를 확인**하거나 수신객체의 프로퍼티에 데이터를 할당하기 전에 해당 **데이터의 유효성을 검사할 때** 매우 유용하다.

<img width="600" alt="스크린샷 2024-07-21 오후 5 47 39" src="https://github.com/user-attachments/assets/bf0d3440-a074-45b0-8e1d-47ce94a7e620">

```kotlin
class Book(author: Person) {
    val author = author.also {
      requireNotNull(it.age) // 데이터의 유효성 검증 
      print(it.name)
    } // 수신객체 자신을 반환 
}
```

# 요약 

이제 아래 표에 대해 정확하게 이해할 수 있을 것이다! 

<img width="700" src="https://github.com/user-attachments/assets/02812fac-7434-41f8-869d-226b4e49dfb1"/>

# 참고 자료

- https://kotlinworld.com/255
- https://medium.com/@limgyumin/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9D%98-apply-with-let-also-run-%EC%9D%80-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80-4a517292df29