# Null 어쩌면 좋니 🤦‍♀️

코틀린은 널이 가능한 자료형 (Nullable Type)과 불가능한 자료형 (Non-Null Type)을 구분하여 자바보다 NPE가 발생할 가능성을 현저히 낮춰주었다. 

코틀린에서 널 처리를 똑똑하게 하기 위한 방법들에 대해 알아보자! 

# nullable type - ?

코틀린은 타입 뒤에 ?를 붙여서 **널이 가능한 타입임을 명시적으로 표현**할 수 있다. 

```kotlin
fun getLength(s: String?): Int = if (s != null) s.length else 0

fun main(args: Array) {
    val x: String? = null
    println(getLength(x))     // 0 
    println(getLength("abc")) // 3 
}
```

# safe call - ?.

**?. 앞의 변수가 null이 아닐 때만 뒤의 함수가 실행되고, null이면 null을 반환한다.** 

```kotlin
fun printAllCaps(s: String?) {
    // if (s != null) s.toUpperCase() else null 
    val allCaps: String? = s?.toUpperCase() 
    println(allCaps)
}

fun main(args: Array) {
    printAllCaps("abc") // ABC 
    printAllCaps(null)  // null 
}
```

```kotlin
class Employee(val name: String, val manager: Employee?)

fun getManagerName(employee: Employee): String? = employee.manager?.name

fun main(args: Array) {
    val ceo = Employee("Da Boss", null)
    val developer = Employee("Bob Smith", ceo)
    println(getManagerName(developer)) // null 
    println(getManagerName(ceo)) // Da Boss 
}
```

# Elvis 연산자 - ?:

?. 연산자는 왼쪽 항이 null이면 null을 반환하지만, 때로는 **null이 아닌 디폴트 값을 리턴하고 싶은 경우**가 있다. 이때 ?: 라는 엘비스 연산자를 사용할 수 있다. 

```kotlin
fun getName(str: String?) {
    val name = str ?: "Unknown"
}
```

그리고 엘비스 연산자를 이용하면, 오른쪽 항에 **return이나 throw**도 넣을 수 있다. 따라서 간결한 코드로 원하는 형태의 널 처리가 가능하다. 

```kotlin
class Address(val streetAddress: String, val zipCode: Int,
              val city: String, val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun printShippingLabel(person: Person) {
    // company가 null이면 exception 발생
    val address = person.company?.address ?: throw IllegalArgumentException("No address")
		
    // with 스코프 함수로 코드 간소화  
    with(address) { 
        println(streetAddress)
        println("$zipCode $city, $country")
    }
}

fun main(args: Array) {
    val address = Address("Elsestr. 47", 80687, "Munich", "Germany")
    val jetbrains = Company("JetBrains", address)
    val person = Person("Dmitry", jetbrains)
    printShippingLabel(person)
    printShippingLabel(Person("Alexey", null))
}
```

# safe cast - as?

스마트 캐스트인 is를 이용하면, as 없이도 형변환이 가능하다. 

반면에 as를 사용하여 강제 형변환 시, 타입이 맞지 않으면 ClassCastException이 발생한다. 

이를 방지하기 위해 코틀린에서는 as? 를 지원한다. 

**as?는 캐스팅을 시도하고, 캐스팅이 불가능하면 널을 반환한다.** 

```kotlin
class Person(val firstName: String, val lastName: String) {
   override fun equals(obj: Any?): Boolean {
      val otherPerson = obj as? Person ?: return false
      return otherPerson.firstName == firstName &&
             otherPerson.lastName == lastName
   }

   override fun hashCode(): Int = firstName.hashCode() * 37 + lastName.hashCode()
}

fun main(args: Array) {
    val p1 = Person("Dmitry", "Jemerov")
    val p2 = Person("Dmitry", "Jemerov")
    println(p1 == p2) // true 
    println(p1.equals(42)) // false (Int에서 Person으로 캐스팅 불가) 
}
```

```java
// 자바로 구현하는 경우 
Person obj = null; 
if (obj instanceOf Person) { 
    obj = (Person)obj; 
} else {
    return false;
}
```

# non null assertion - !!

코드 플로우 상 null이 절대로 될 수 없는 변수도 있다. 하지만 컴파일러는 이를 인식할 수 없기 때문에 nullable 타입의 변수에 대한 널 처리를 매번 해줘야 한다. 

이러한 번거로움을 줄이기 위해 nullable 변수를 **강제로 not null로 바꿔주는 !! 연산자**가 있다. 즉, 이 변수는 널이 아니라고 단언하는 것이다. 

단, !! 연산자로 널이 아니라고 단언했는데 **실제로 널이면 NPE가 발생한다.** 따라서, 매우 주의해서 사용해야 하는 연산자이다. 

```kotlin
fun ignoreNulls(s: String?) {
    val sNotNull: String = s!! // NPE 발생 
    println(sNotNull.length)
}

fun main(args: Array) {
    ignoreNulls(null)
}
```

아래와 같이 !! 연산자를 체이닝 해서 사용하는 경우에는 어디서 NPE가 발생했는지 알 수 없으므로 권장되지 않는다. 

```kotlin
person.company!!.address!!.country
```

# let 함수

**널이 아닌 경우에만 특정 내용을 실행시키고 싶을 때,** if문 대신 사용할 수 있는 let 함수가 있다. 

let 함수는 자신의 수신 객체를 람다식의 인자로 넘기며, **non-null 타입으로 변환**한다. 

**수신 객체가 null인 경우에는 let 함수의 람다식이 아예 실행되지 않는다.** 

let 함수는 중첩해서 사용이 가능하지만, 너무 과하면 가독성이 오히려 떨어질 수 있으므로 차라리 if문으로 널 체크를 하는 게 나을 수도 있다. 

```kotlin
fun sendEmailTo(email: String) {
    println("Sending email to $email")
}

fun main(args: Array) {
    val email: String? = "yole@example.com"
    email?.let { sendEmailTo(it) } // it는 non-null 타입 

    email = null
    email?.let { sendEmailTo(it) } // let 함수의 람다식 실행 X 
}
```

# 지연 초기화 - lateinit

객체의 인스턴스를 선언(decleration)하고 나서, 초기화(initialization)는 나중에 하고 싶은 경우가 있다. 이를 지원하기 위해 코틀린에서는 lateinit 키워드를 제공한다. 

단, **val로 선언된 인스턴스는 선언과 동시에 초기화를 반드시** 해줘야 한다. 

lateinit으로 선언된 변수가 아직 초기화 되지 않았는데 접근하려고 하면, UninitializedException이 발생할 수 있으므로 주의해서 사용해야 한다. 

```kotlin
class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    private lateinit var myService: MyService // 선언 (변수 생성) 

    @Before 
    fun setUp() {
        myService = MyService() // 초기화 (메모리와 값의 할당) 
    }

    @Test 
    fun testAction() {
        Assert.assertEquals("foo", myService.performAction())
    }
}
```

# 제네릭의 nullable

제네릭을 사용하면 이는 **무조건 nullable 타입으로 인식**된다. 따라서, 함수 내부에서는 반드시 제네릭 변수에 대한 널 체크를 해줘야 한다. 

```kotlin
fun <T> printHashCode(t: T) { // nullable 
    println(t?.hashCode()) 
}

fun main(args: Array) {
    printHashCode(null)
}
```

non-null을 기본으로 제네릭을 사용하려면 아래 코드에서 \<T: Any>처럼 **부모 클래스에 대한 제한을 명시적으로 작성**해줘야 한다. 

```kotlin
fun <T: Any> printHashCode(t: T) { // non null 
    println(t.hashCode())
}

fun main(args: Array) {
    printHashCode(null)
}
```

# 플랫폼 타입

코틀린에서는 변수를 nullable, non-null 타입으로 구분하여 선언함으로써 NPE의 발생 확률을 낮춘다. **단, 자바와의 호환성에 있어 주의해야 할 점이 있다.** 

자바는 변수에 붙은 어노테이션에 따라 코틀린에서의 널 타입이 결정된다. 

- 자바의 @Nullable String == 코틀린의 String?
- 자바의 @NonNull String == 코틀린의 String

**그런데 문제는 자바에서 위와 같은 어노테이션 없이 쓰이는 변수들이 대부분이라는 것이다.** 

**이러한 불분명한 자바의 타입은 코틀린에서 ‘플랫폼 타입’으로 변환된다.** 

플랫폼 타입은 널 처리를 해도 되고, 안 해도 된다. **다만, 널 체크를 해야 되는데 하지 않으면 NPE가 발생하며, 이러한 널 체크는 전적으로 개발자의 몫이 된다.** 

코틀린 컴파일러는 플랫폼 타입에 한해서는 널 처리가 중복되거나, 불필요하게 널 체크를 했다거나 하는 경고 자체를 띄우지 않는다. 

아래와 같은 자바 코드에서는 getName 함수 호출 시, null이 반환될 가능성이 존재한다. 

```java
// Java 
public class Person {
    private final String mName;

    public Person(String name) {
        mName = name;
    }

    public String getName() {
        return mName;
    }
}
```

이를 코틀린에서 플랫폼 타입으로 변환하여 널 체크 없이 사용하더라도, 컴파일 시에는 전혀 문제가 되지 않는다. **단, 개발자의 판단 하에 아래와 같이 널 처리를 해줘야 런타임 에러가 발생하는 걸 피할 수 있다.** 

```java
fun main(args: Array) {
    yellAtSafe(Person(null)) // 자바로 작성한 Person 클래스 사용 
}

fun yellAtSafe(person: Person) {
    // 코틀린에서 따로 널처리 안하면 name에서 IllegalArgumentException 발생 
    println((person.name ?: "Anyone").toUpperCase() + "!!!")
}
```

자바에서 코틀린으로 코드 변환 시, 플랫폼 타입이 아닌 nullable 타입으로 모두 변환한다면

ArrayList<String> 대신에 ArrayList<String?>? 로 사용해야만 한다. 

이를 막기 위해 플랫폼 타입이 적용되었으며, ! 를 써서 표현한다. 따라서 컴파일 내부에서 String! 타입으로 표현되지만, 실제 개발자가 코드 상에서 플랫폼 타입을 직접 선언할 수는 없다.

# 마무리

**자바와 코틀린의 null에 대한 접근 자체가 다르지만, 이 둘의 호환성을 지원하기 위해서는 개발자가 신경 써서 널 처리를 해줘야 한다.** 

예를 들어, **자바의 인터페이스나 클래스를 상속 받아 메서드를 오버라이딩 할 때, 메서드의 인자는 널 체크가 필요한지 아닌지를 명확하게 결정하고 사용**해야 한다. 해당 함수를 다른 코틀린 코드가 접근할 수 있으므로, 컴파일러는 non-null 타입으로 선언한 모든 파라미터에 대해 널이 아님을 검사하는 단언문을 만든다. 따라서 이 함수를 자바에서 null을 넣고 호출한다면 전부 예외가 발생하게 된다. 

# 참고자료 

https://tourspace.tistory.com/114