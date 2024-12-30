# 데이터 클래스 

코틀린에서 제공하는 데이터 클래스는 **데이터의 저장 및 관리를 위한 보일러 플레이트 코드를 줄여준다.**

일반 클래스와 달리, toString(), copy(), hashCode(), equals(), componentN() 같은 메서드를 **자동으로 생성**해주기 때문이다. 

## toString()

```java
class User {
    String name;
    int age;

    @Override
    public String toString(){
        return "[User] name : " + name + ", age : " + Integer.toString(age);
    }

    public String getName(){
        return name;
    }

    public void setName(String name){
        this.name = name;
    }
}
```

👇 모든 프로퍼티 값들을 **가독성 좋게 출력**해준다! 

```kotlin 
data class User(val name: String, val age: Int)

fun main() {
    val user = User("Alice", 12)
    println(user) // User(name=Alice, age=12)
}
```

## copy()

객체의 **특정 프로퍼티만 변경**하고 싶을 때 유용하게 사용할 수 있다!

```kotlin 
data class User(val name: String, val age: Int)

fun main() {
    val user = User("Alice", 12)
    val olderUser = user.copy(age = 30)
    println(olderUser) // User(name=Alice, age=30)
}
```

## hashCode()

자바에서 hashCode() 메서드는 객체의 메모리 주소를 기반으로 고유한 해시 코드를 생성하여 반환한다. 반면에, 코틀린의 데이터 클래스는 **프로퍼티 값을 기반으로 해시 코드를 생성**하므로, 프로퍼티 값이 모두 동일하면 같은 해시 코드를 반환한다. (단, 일반 클래스는 hashCode()를 재정의하지 않으면 자바의 hashCode() 메서드처럼 동작한다.) 

```kotlin 
data class User(val name: String, val age: Int)

fun main() {
    val user1 = User("Alice", 12)
    val user2 = User("Alice", 12)
    val user3 = User("Alice", 15)
    println(user1.hashCode()) // 1963861420
    println(user2.hashCode()) // 1963861420
    println(user3.hashCode()) // 1963861423
}
```

## equals()

코틀린의 데이터 클래스는 == 연산자만 사용해도, 내부적으로 equals() 메서드가 호출되어 동등성 검사를 수행할 수 있다. 

- 프로퍼티 값을 비교하는 동등성 검사: [Java] equals() [Kotlin] == 
- 메모리 주소를 비교하는 동일성 검사: [Java] == [Kotlin] === 

```kotlin 
data class User(val name: String, val age: Int)

fun main() {
    val user1 = User("Alice", 12)
    val user2 = User("Alice", 12)
    println(user1 == user2) // T
    println(user1 === user2) // F
}
```

## componentN()

**객체의 프로퍼티를 여러 개의 변수로 분리**하여 사용하고 싶을 때가 있다. 

코틀린의 데이터 클래스는 **각 프로퍼티에 대해 componentN() 메서드를 자동으로 생성**해주기 때문에, 아래 코드처럼 **각 프로퍼티에 해당하는 변수를 쉽게 선언**할 수 있다. 

이를 **구조 분해 선언** (destructuring declaration)이라고 부른다. 

```kotlin 
data class User(val name: String, val age: Int)

fun main() {
    val (name, age) = User("Alice", 12)
    println("$name $age") 
}
```

# 2025 안드로이드 탐구 영역 9번 문제 

📌 시험지 링크: https://android-exam25.gdg.kr/

```kotlin 
data class User(val name: String, val age: Int)
class UserTwo(val name: String, val age: Int)

fun main() {
    val user1 = User("Alice", 30)
    val user2 = UserTwo("Alice", 30)
    
    val (name, age) = user1 
    val user3 = User(name, age)
    
    // println(user1 == user2) 
    // println(user1 === user2)
    // Compile Error: Operator '===' cannot be applied to 'User' and 'UserTwo'.

    println(user1 == user3) // T (동등성 비교)
    println(user1 === user3) // F (동일성 비교)
    
    // 여기서부터는 제가 코드를 조금 수정했습니다! 
    val literal = "Alice" // 문자열 리터럴 (String Pool에 같은 문자열이 있으면, 새로 할당하지 않고 해당 메모리 주소 참조) 
    val newString = String(literal.toCharArray()) // 새로운 String 객체 생성 (새로운 메모리 주소 할당)
    println(user1.name === literal) // T 
    println(user1.name === newString) // F 
    
    val internedString = newString.intern() // String Pool에 이미 있는 문자열의 참조 반환 
    println(user1.name === internedString) // T 
}
```

# 참고 자료 

https://medium.com/kenneth-android/kotlin-kotlin-data-class-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-7d7f51885075

https://velog.io/@haero_kim/Kotlin-감동-실화-Data-Class-알아보기

https://hoestory.tistory.com/50

