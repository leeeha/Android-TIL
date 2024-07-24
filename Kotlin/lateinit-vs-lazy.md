# 지연 초기화 

지연 초기화는 말 그대로 **변수나 객체를 늦게 초기화 하는 것**이다. 

예를 들어, 분명 변수 a를 사용할 예정인데 **결과 값은 나중에 알 수 있어서 a의 첫 상태를 결정하기 어려울 때** 사용할 수 있다. 

아래처럼 nullable 타입으로 변수를 선언할 수도 있지만, null의 사용을 최대한 지양하는 코틀린에서 이게 최선의 방법인지 고민이 든다.

```kotlin
var a: String? = null
```

이때 `lateinit` 또는 `lazy` 키워드를 사용할 수 있다.

## lateinit

```kotlin
fun main(){
    lateinit var text: String 

    val result1 = 30 
    text = "Result: $result1"
    println(text)

    val result2 = 50 
    text = "Result: ${result1 + result2}"
    println(text)
}
```

위의 코드에서 알 수 있듯이 `lateinit`으로 선언된 변수는, **어떤 동작의 결과 값을 기반으로 늦게 초기화** 할 수 있으며, var이므로 **다른 값으로 변경**할 수 있다. (val 사용 불가)

만약 `lateinit` 이라고 선언해두고 나중에 초기화 하지 않으면, `UninitializedPropertyAccessException` 에러가 발생한다. 

참고로 `lateinit` 키워드는 **Primitive Type** (Int, Float, Double, Long 등) **에서는 사용할 수 없다.**

### Primitive Type vs Reference Type

- 원시 타입 (Primitive Type): 변수의 실제 값을 저장하는 타입 (정수, 실수, 문자 등)
- 참조 타입 (Reference Type): 객체의 주소를 참조하는 타입 (문자열, 배열 등)
- 참조 타입의 변수는 스택 영역에 객체의 주소를 저장하며, 실제 객체의 값은 힙 영역에 동적으로 할당되어 있다.
- 코틀린에서는 Primitive type과 Wrapper type을 따로 구분하지 않으며, 컴파일 시 Primitive 또는 Wrapper 타입으로 자동 변환된다. 
- 예를 들어, 코틀린의 Int 타입은 대부분 자바의 int 타입으로 컴파일 된다. 제네릭이나 컬렉션에서 사용될 때는 java.lang.Integer라는 Wapper 타입으로 변환된다.

## lazy 

```kotlin
fun main() {
    lateinit var text: String 
    val textLength: Int by lazy {
        text.length
    }

    text = "leeeha"
    println(textLength)
}
```

`lazy`로 선언된 변수는, **해당 변수가 호출되는 시점에 초기화** 된다. 즉, `lazy`로 선언할 당시에는 초기화 할 수 없지만, **의존하는 값들이 초기화 된 이후에 값을 채워넣고 싶을 때 사용**할 수 있다. 

위의 코드에서도 text가 초기화 된 이후에 textLength를 초기화 할 때, text.length로 초기화 한 것을 확인할 수 있다.

`lazy`로 선언된 변수는 val만 가능하다. 즉, 한번 지연 초기화가 이루어진 다음에는 **값을 변경할 수 없다.** 

Tip: 안드로이드에서는 이전 액티비티에서 넘어온 Intent 부가 데이터를 `lazy`를 이용해 현재 액티비티의 멤버변수로 선언해두고, 이것이 호출되는 시점에서 `intent.extra` 등으로 번들에 담긴 데이터를 꺼내 변수를 초기화 할 수 있다. 이렇게 하면 생명주기에 위반하지 않고 안전하게 현재 액티비티 클래스 전역에서 이전 액티비티의 번들 데이터를 활용할 수 있다.

## 차이점 요약 

- lateinit: var 사용, 초기화 후에 계속 값이 바뀔 수 있음.
- lazy: val 사용, 초기화 후에 읽기 전용으로 값을 사용함. 

# 참고 자료

- https://velog.io/@haero_kim/Kotlin-lateinit-vs-lazy-정확히-아세요
- https://velog.io/@gillog/원시타입-참조타입Primitive-Type-Reference-Type