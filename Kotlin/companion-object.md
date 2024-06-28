# 학습 목표

- companion object와 static의 차이점 이해
- static의 한계점 파악
- companion object에 대해서 

코틀린의 companion object는 단순히 자바의 static 키워드를 대체하기 위해 등장했을까? 

이 질문에 대한 답을 찾아가다 보면, 코틀린에서 왜 static을 안 쓰게 되었는지 이해하는 데 큰 도움이 될 수 있다. 

# 자바의 static

자바의 static 키워드는 **클래스 멤버**임을 지정하기 위해 사용한다. 

- static 키워드가 붙은 변수와 메서드: **클래스 변수, 클래스 메서드**
- state 키워드가 없는 변수와 메서드: **인스턴스 변수, 인스턴스 메서드**

static 멤버는 **클래스가 메모리에 적재될 때 자동으로 함께 생성**되므로, **인스턴스 생성 없이도 클래스 이름만으로 멤버를 참조**할 수 있다. 

```java
public final class MyClass {
  static public final String TEST = "test"; // 클래스 변수
  static public method(int i):int{          // 클래스 메서드 
    return i + 10
  }
}

System.out.println(MyClass.TEST);      // test
System.out.println(MyClass.method(1)); // 11
```

코틀린에서는 static 키워드가 사라지고, companion object가 그 역할을 대신할 수 있다. 그렇지만, 이 두 가지는 동일하지 않다.

```kotlin
class MyClass{
    companion object{
        val TEST = "test"
        fun method(i:Int) = i + 10
    }
}
fun main(args: Array<String>){
    println(MyClass.TEST);    //test
    println(MyClass.method(1));   //11
}
```

# 코틀린의 class 키워드

```kotlin
class WhoAmI(private val name:String){
    fun myNameIs() = "나의 이름은 ${name}입니다."
}

fun main(args: Array<String>){
    val m1 = WhoAmI("영수")
    val m2 = WhoAmI("미라")
    println(m1.myNameIs()) // 나의 이름은 영수입니다.
    println(m2.myNameIs()) // 나의 이름은 미라입니다.
}
```

```kotlin
class WhoAmI{
    private val name:String
    constructor(name:String){
        this.name = name
    }
    fun myNameIs() = "나의 이름은 ${name}입니다."
}
```

코틀린에서는 생성자를 **constructor 키워드**로 정의하며, 첫번째 코드처럼 생략할 수도 있다. 

다시 말해, **클래스를 정의하자마자 바로 생성자와 속성까지 정의**할 수 있게 된 것이다. 

그리고 중요한 점은 **클래스로부터 객체를 생성할 때 new 키워드를 사용하지 않는다**는 것이다. 

`val m1 = WhoAmI()` 이렇게 마치 함수처럼 생성자를 사용할 수 있다. 

# 코틀린의 object 키워드

코틀린에는 자바에 없는 독특한 **싱글턴 (singleton: 인스턴스가 딱 하나만 있는 클래스)** 선언 방법이 있다. class 대신 object 키워드를 사용하면, 자바에서 직접 구현하는 것보다 훨씬 편하게 싱글턴 패턴을 구현할 수 있다.

```kotlin
object MySingleton {
    val prop = "나는 MySingleton의 속성이다."
    fun method() = "나는 MySingleton의 메소드다."
}

fun main(args: Array<String>){
    println(MySingleton.prop);     // 나는 MySingleton의 속성이다.
    println(MySingleton.method()); // 나는 MySingleton의 메소드다.
}
```

object는 특정 클래스나 인터페이스를 확장해 만들 수 있다. 

`val obj = object: MyClass() {}`

`val obj = object: MyInterface {}`

위처럼 statement가 아닌 expression으로 object를 생성할 수 있다. 

**언어 수준에서 안전한 싱글턴을 만들어준다는 점에서 object는 매우 유용하다.** 

>statement(문) vs. expression(식)
>- statement: 값을 반환하지 않는다. ex) 자바의 if문
>- expression: 값을 반환한다. ex) 코틀린의 if, when

---

# Companion object는 static이 아니다.

사실 코틀린 companion object는 static이 아니며, 사용하는 입장에서 static으로 동작하는 것처럼 보일 뿐이다. 아래 코드를 보자. 

```kotlin
class MyClass2 {
    companion object {
        val prop = "나는 Companion object의 속성이다."
        fun method() = "나는 Companion object의 메소드다."
    }
}

fun main(args: Array<String>) {
    // 사실은 MyClass2.맴버는 MyClass2.Companion.맴버의 축약표현이다.
    println(MyClass2.Companion.prop)
    println(MyClass2.Companion.method())
}
```

MyClass2 클래스에서 companion object를 만들어 2개의 멤버를 정의했다. 이를 사용하는 main 함수를 보면, 이 멤버에 접근하기 위해 `클래스명.Companion` 형태로 쓴 것을 확인할 수 있다. 

즉, companion object는 MyClass2 **클래스가 메모리에 적재되며 함께 생성되는, 동반되는(Companion) 객체**이고, 이 동반 객체를 `클래스명.Companion` 으로 접근할 수 있다. 

```kotlin
fun main(args: Array<String>) {
    // 사실은 MyClass2.맴버는 MyClass2.Companion.맴버의 축약표현이다.
    println(MyClass2.prop)
    println(MyClass2.method())
}
```

위 코드에서 MyClass2.prop, MyClass2.method() 는 중간에 Companion 키워드가 생략된 표현일 뿐이다. 즉, **언어적으로 지원하는 축약 표현 때문에 companion object가 static과 동일하다는 착각이 들었던 것이다!** 

## Companion object는 객체이다.

Companion object에서 기억할 중요한 점은 이것이 **객체**라는 것이다. 따라서 다음과 같이 코드를 작성할 수 있다. 

```kotlin
class MyClass2{
    companion object{
        val prop = "나는 Companion object의 속성이다."
        fun method() = "나는 Companion object의 메소드다."
    }
}

fun main(args: Array<String>) {
    println(MyClass2.Companion.prop)
    println(MyClass2.Companion.method())

    val comp1 = MyClass2.Companion //--(1)
    println(comp1.prop)
    println(comp1.method())

    val comp2 = MyClass2 //--(2)
    println(comp2.prop)
    println(comp2.method())
}
```

주석 (1)에서 companion object는 **객체이므로 변수에 할당할 수 있다.** 그리고 할당한 변수에서 . 으로 MyClass2에 정의된 companion object의 멤버에 접근할 수 있다. 

이렇게 변수에 할당하는 것은 자바의 클래스에서 static으로 정의된 멤버한테는 적용할 수 없는 방법이다. 

주석 (2)에서는 `.Companion`을 생략하고 직접 MyClass2로 변수에 할당한 것이다. 이 또한 companion object이다. 따라서, 꼭 기억할 것은 클래스 내에 정의된 **companion object는 클래스 이름만으로도 참조 접근이 가능하다**는 것이다!! 

자바의 **static 키워드만으로는 클래스 멤버를 companion object처럼 하나의 독립된 객체로 여길 수 없다.** 이것 또한 static과의 큰 차이점이다. 

## Companion object에 이름을 지을 수 있다.

```kotlin
class MyClass3{
    companion object MyCompanion {  // -- (1)
        val prop = "나는 Companion object의 속성이다."
        fun method() = "나는 Companion object의 메소드다."
    }
}

fun main(args: Array<String>) {
    println(MyClass3.MyCompanion.prop) // -- (2)
    println(MyClass3.MyCompanion.method())

    val comp1 = MyClass3.MyCompanion // -- (3)
    println(comp1.prop)
    println(comp1.method())

    val comp2 = MyClass3 // -- (4)
    println(comp2.prop)
    println(comp2.method())

    val comp3 = MyClass3.Companion // -- (5) 에러발생 
    println(comp3.prop)
    println(comp3.method())
}
```

주석 (1)처럼 companion object에 이름을 붙일 수 있다. 주석 (4)처럼 생략 가능하지만, 주석 (5)처럼 기본 이름인 Companion은 더 이상 사용할 수 없다!  

## 클래스 내에 Companion object는 딱 하나만 정의할 수 있다.

클래스 내에 2개 이상의 동반 객체를 쓰는 것은 허용되지 않는다. 사실 이건 당연한 건데, 코틀린은 **클래스명으로 동반 객체를 참조**할 수 있으므로 동일한 클래스명으로 2개의 동반 객체를 참조한다는 건 애초부터 불가능한 일이다.  

```kotlin
class MyClass5{
    companion object{
        val prop1 = "나는 Companion object의 속성이다."
        fun method1() = "나는 Companion object의 메소드다."
    }

    companion object{ // -- 에러발생!! Only one companion object is allowed per class
        val prop2 = "나는 Companion object의 속성이다."
        fun method2() = "나는 Companion object의 메소드다."
    }
}
```

**아래처럼 이름을 다르게 지어도 마찬가지이다!!** 

```kotlin
class MyClass5{
    companion object MyCompanion1{
        val prop1 = "나는 Companion object의 속성이다."
        fun method1() = "나는 Companion object의 메소드다."
    }

    companion object MyCompanion2{ // --  에러발생!! Only one companion object is allowed per class
        val prop2 = "나는 Companion object의 속성이다."
        fun method2() = "나는 Companion object의 메소드다."
    }
}
```

## 인터페이스 내에서도 Companion object를 정의할 수 있다.

코틀린의 인터페이스 내에도 동반 객체를 정의할 수 있다. 덕분에 인터페이스 수준에서 상수항을 정의할 수 있고, 관련된 중요 로직을 이곳에 기술할 수 있다. 이를 잘 활용하면 설계에 도움이 될 것이다. 

```kotlin
interface MyInterface{
    companion object{
        val prop = "나는 인터페이스 내의 Companion object의 속성이다."
        fun method() = "나는 인터페이스 내의 Companion object의 메소드다."
    }
}
fun main(args: Array<String>) {
    println(MyInterface.prop)
    println(MyInterface.method())

    val comp1 = MyInterface.Companion
    println(comp1.prop)
    println(comp1.method())

    val comp2 = MyInterface
    println(comp2.prop)
    println(comp2.method())
}
```

## 상속 관계에서 Companion object 멤버는 같은 이름일 경우 가려진다. (섀도잉, shadowing)

```kotlin
open class Parent{
    companion object{
        val parentProp = "나는 부모값"
    }
    fun method0() = parentProp
}

class Child: Parent(){
    companion object{
        val childProp = "나는 자식값"
    }
    fun method1() = childProp
    fun method2() = parentProp
}

fun main(args: Array<String>) {
    val child = Child()
    println(child.method0()) // 나는 부모값
    println(child.method1()) // 나는 자식값
    println(child.method2()) // 나는 부모값
}
```

위의 코드처럼 부모, 자식의 동반 객체의 멤버가 다른 이름이라면 자식이 부모의 동반 객체 멤버를 직접 참조할 수 있다. 그런데, 같은 이름의 멤버라면? 

```kotlin
open class Parent{
    companion object{
        val prop = "나는 부모"
    }
    fun method0() = prop // Companion.prop과 동일
}

class Child: Parent(){
    companion object{
        val prop = "나는 자식"
    }
    fun method1() = prop // Companion.prop과 동일
}

fun main(args: Array<String>) {
    println(Parent().method0()) // 나는 부모
    println(Child().method0()) // 나는 부모
    println(Child().method1()) // 나는 자식

    println(Parent.prop) // 나는 부모
    println(Child.prop) // 나는 자식

    println(Parent.Companion.prop) // 나는 부모
    println(Child.Companion.prop) // 나는 자식 (여기 주목!!) 
}
```

부모, 자식 클래스의 동반 객체에서 prop 라는 같은 이름의 멤버가 정의되어 있다. 

이때, 자식 클래스에서 `Child.Companion.prop` 이렇게 멤버를 참조하면, **부모 멤버는 가려지고 자식 자신의 멤버만 참조할 수 있다.**

**즉, 부모의 동반 객체에서 정의되어 있는 prop 속성은 자식에게 가려져서 무시된다. (섀도잉)** 

```kotlin
open class Parent{
    companion object{
        val prop = "나는 부모"
    }
    fun method0() = prop
}

class Child: Parent(){
    companion object ChildCompanion { // -- (1) ChildCompanion로 이름을 부여했어요.
        val prop = "나는 자식"
    }
    fun method1() = prop
    fun method2() = ChildCompanion.prop
    fun method3() = Companion.prop
}

fun main(args: Array<String>) {
    val child = Child()
    println(child.method0()) // 나는 부모
    println(child.method1()) // 나는 자식
    println(child.method2()) // 나는 자식
    println(child.method3()) // -- (2)
}
```

주석 (2)의 결과는 무엇일까?

바로 "나는 부모"이다. 

자식의 동반 객체를 뛰어넘어 부모의 동반 객체에 직접 접근이 가능하게 되었다. 

`Companion.prop` 여기에서 **Companion은 자식이 아니라 부모의 동반 객체**를 가리킨다. (자식은 새 이름을 지었기 때문에) 

```kotlin
open class Parent{
    companion object ParentCompanion { // -- (1) ParentCompanion로 이름을 부여했어요.
        val prop = "나는 부모"
    }
    fun method0() = prop
}

class Child:Parent(){
    companion object ChildCompanion {
        val prop = "나는 자식"
    }
    fun method1() = prop // 나는 자식 
    fun method2() = ChildCompanion.prop // 나는 자식 
    fun method3() = Companion.prop // -- (2) Unresolved reference: Companion 에러!!
}
```

이번에는 부모의 동반 객체에도 이름을 붙였다! 그 결과, 주석 (2)에서 이제는 컴파일러가 Companion이라는 이름을 인식하지 못한다. 

`ParentCompanion.prop` 라고 작성하면 에러가 발생하지 않을 것이다. 

```kotlin
open class Parent{
    companion object{
        val prop = "나는 부모"
    }
    open fun method() = prop // Companion.prop과 동일
}
class Child: Parent(){
    companion object{
        val prop = "나는 자식"
    }
    override fun method() = prop // Companion.prop과 동일
}
fun main(args: Array<String>) {
    println(Parent().method()) // 나는 부모
    println(Child().method()) // 나는 자식

    val p: Parent = Child() // 업 캐스팅 
    println(p.method()) // -- (1)
}
```

위의 코드에서 주석(1)의 결과는 무엇일까?

바로 "나는 자식"이다. 

런타임에 동적 바인딩에 의해 실제로 가리키고 있는 객체가 자식이므로 `Child().method()` 와 같은 결과가 나온다. (객체지향 언어의 다형성) 

# 참고 자료

[[kotlin] Companion Object (1) - 자바의 static과 같은 것인가? - Bsidesoft co.](https://www.bsidesoft.com/8187)
