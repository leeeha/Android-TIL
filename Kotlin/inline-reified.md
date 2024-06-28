# 인라인 함수란?

- 자신이 호출되는 곳에 **함수 내용을 전부 복사**한다.
- 함수 호출로 인한 분기 처리가 없어서 성능이 향상될 수 있다.
- 단, 인라인 함수가 여러 번 호출되면 한 함수에 들어가는 코드량이 많아질 수 있으므로 주의가 필요하다.

<img width="600" src="https://github.com/leeeha/Android-TIL/assets/68090939/ca52a14e-8cbd-4a15-822d-8c0541df4379"/>

일반 함수에 inline 키워드를 붙이면 다음과 같은 경고가 발생한다. 

```kotlin
inline fun foo() {}
```

>*Inlining works best for functions with parameters of functional types*

즉, 인라인 함수는 **람다식을 매개변수로 사용할 때** 성능이 가장 좋다는 것이다. 왜 그럴까? 

아래 함수는 람다식을 매개변수로 받고 있다. 

```kotlin
fun doSomething(lambda: () -> Unit) {
    println("Doing something")
    lambda()
}
```

이를 디컴파일 해보면 람다식은 Function 객체로 바뀌고, 이 객체가 invoke 메서드를 실행한다는 걸 알 수 있다. 

```kotlin
public static final void doSomething(Function0 lambda) {
    System.out.println("Doing something");
    lambda.invoke();
}
```

그렇다면, 아래 코드를 디컴파일 하면 어떻게 될까? 

```kotlin
fun main() {
    println("Before lambda")
    doSomething {
        println("Inside lambda")
    }
    println("After lambda")
}
```

doSomething 함수의 매개변수로 **새로운 Function 객체를 생성하여 넘겨준다**는 걸 알 수 있다. 

doSomething 함수를 호출할 때마다 **객체가 매번 새로 만들어진다.** 

```kotlin
public static final void main() {
    System.out.println("Before lambda");
    doSomething(new Function() {
            public final void invoke() {
            System.out.println("Inside lambda");
        }
    });
    System.out.println("After lambda");
}
```

doSomething을 인라인 함수로 만들면, 이러한 **무의미한 객체 생성을 방지할 수 있다!** 

인라인 함수는 자신이 호출된 곳에 **함수 내용을 그대로 복사하는 방식**으로 컴파일 되기 때문이다. 

다음과 같이 doSomething 함수를 inline 으로 만들어보자. 

```kotlin
inline fun doSomething(lambda: () -> Unit) {
    println("Doing something")
    lambda()
}
```

그리고 main 함수를 디컴파일 해보면, 인라인 함수의 내용이 그대로 복사된 것을 확인할 수 있다. **무의미하게 Function 객체를 생성하지 않아도 되는 것**이다! 

```kotlin
public static final void main() {
    System.out.println("Before lambda");
    System.out.println("Doing something");
    System.out.println("Inside lambda");
    System.out.println("After lambda");
}
```

만약 지역 변수를 람다식에서 사용한다면 어떻게 컴파일 될까?

```kotlin
fun main() {
    val greetings = "Hello" // Local variable
    doSomething {
        println("$greetings from lambda")  // Variable capture
    }
}
```

Function 객체의 생성자로 지역 변수가 들어가는 것을 확인할 수 있다. 즉, 객체에 멤버 변수가 추가되면서 메모리 사용량이 더 늘어난 것이다. 

```kotlin
public static final void main() {
    String greetings = "Hello";
    doSomething(new Function(greetings) {
            public final void invoke() {
            System.out.println(this.$greetings + " from lambda");
        }
    });
}
```

따라서, **람다식을 매개변수로 가지는 함수는 inline 으로 만드는 게 좀 더 나은 성능을 보장한다**는 걸 알 수 있다. 

기본적으로 JVM의 JIT 컴파일러가 일반 함수를 **인라인 함수로 사용하는 게 더 좋다고 판단하면 자동으로 만들어 준다**고 한다. 

참고로 public 인라인 함수는 private 함수에 접근 불가능하다. 

> *Public-API inline function cannot access non-public API fun*
> 

```kotlin
inline fun doSomething() {
    doItPrivately() // Error
}

private fun doItPrivately() { }
```

## noinline

인라인 함수는 자신이 호출되는 곳에 함수 내용을 모두 복사하기 때문에, **인자로 전달된 함수는 다른 함수에 전달할 수 없다.** 

아래 코드에서 두번째 인자로 받은 action2 람다식을 또 다른 고차함수인 anotherFunc에 넘기려고 하면 에러가 발생한다. 

```kotlin
inline fun doSomething(action1: () -> Unit, action2: () -> Unit) {
    action1()
    anotherFunc(action2) // Error
}

fun anotherFunc(action: () -> Unit) {
    action()
}

fun main() {
    doSomething({
        println("1")
    }, {
        println("2")
    })
}
```

이처럼 **모든 인자를 inline으로 처리하고 싶지 않을 때는 noinline 키워드를 사용**하면 된다. 그러면 해당 인자는 인라인 처리에서 제외되며, 다른 함수의 인자로 전달하는 것이 가능해진다. 

```kotlin
inline fun doSomething(action1: () -> Unit, noinline action2: () -> Unit) {
    action1()
    anotherFunc(action2) // OK 
}

fun anotherFunc(action: () -> Unit) {
    action()
}

fun main() {
    doSomething({
        println("1")
    }, {
        println("2")
    })
}
```

## crossinline

아래 코드는 뷰를 한 번만 클릭했는지 확인하는 확장 함수이다. 

```kotlin
inline fun View.setOnSingleClickListener(
    delay: Long = 500L,
    block: (View) -> Unit
) {
    var previousClickedTime = 0L
    setOnClickListener { view ->
        val clickedTime = System.currentTimeMillis()
        if (clickedTime - previousClickedTime >= delay) {
            block(view) // Error 
            previousClickedTime = clickedTime
        }
    }
}
```

block 이라는 함수를 인자로 받아서 setOnClickListener 내부에서 호출해야 하는데, 아래와 같은 에러가 발생한다.

> *Can't inline 'block' here: **it may contain non-local returns.** Add 'crossinline' modifier to parameter declaration 'block’*
> 

block 함수에서 ‘비지역 반환이 발생할 수도 있으므로’ crossinline 키워드를 붙이라고 알려주고 있다! 

비지역 반환은 **특정 블록이나 람다식의 스코프를 벗어나 바깥쪽 함수의 스코프로 반환하는 것**이다. 

아래 코드처럼 forEach 라는 람다식 내부에서 return을 사용하면, 람다식이 아닌 **그것을 포함하고 있는 함수까지 종료되어 버린다.** 

```kotlin
fun foo(){
	listOf(1, 2, 3).forEach() {
		if (it == 2) return // foo 함수를 종료하는 비지역 반환 
		println(it)
	}
	println("이 문장은 실행되지 않는다.")
}
```

**비지역 반환은 예기치 않은 흐름 제어로 인해 테스트 및 유지보수가 어렵고, 코드의 안정성도 떨어지게 만든다.**

이와 같은 문제를 해결하기 위해 crossinline 키워드를 사용할 수 있다! 

```kotlin
inline fun <T> Iterable<T>.exForEach(crossinline action: (T) -> Unit) {
	for(element in this) action(element)
}

fun foo(){
	listOf(1, 2, 3).exForEach() {
		if (it == 2) return // Errro: 'return' is now allowed here
		println(it)
	}
	println("이 문장은 실행되지 않는다.")
}
```

아까 봤던 setOnSingleClickListener 함수에서도 block 인자에 **crossinline 키워드를 붙여서 비지역 반환의 가능성을 없애면**, 다음과 같이 setOnClickListener 함수 안에서 block 인자를 사용할 수 있다. 

```kotlin
inline fun View.setOnSingleClickListener(
    delay: Long = 500L,
    crossinline block: (View) -> Unit  
) {
    var previousClickedTime = 0L
    setOnClickListener { view ->
        val clickedTime = System.currentTimeMillis()
        if (clickedTime - previousClickedTime >= delay) {
            block(view) // OK 
            previousClickedTime = clickedTime
        }
    }
}
```

## reified

아래 코드와 같이 제네릭으로 범용성 있는 함수를 만들고 싶을 때가 있다. 

```kotlin
fun <T> func1(data: T) { 
    println("$data")
}
```

하지만 **런타임에 제네릭 매개변수 T에 대한 타입 정보가 없어지기 때문에**, 다음과 같이 Class<T>를 같이 인자로 넘겨서 **타입을 확인하고 캐스팅 하는 과정**을 거치곤 한다. 

```kotlin
fun <T> func1(data: T, type: Class<T>) { 
    println("$data") // Hello
    println("$type") // class java.lang.String
    println("${T::class.simpleName}") // Error
}

fun main() {
    func1("Hello", String::class.java)
}
```

이럴 때 **inline, reified 키워드를 사용하면 런타임에도 T에 대한 타입 정보에 접근할 수 있게 된다.** 따라서 Class<T>와 같이 추가적인 인자를 넘길 필요가 없어진다. 

**reified 키워드는 인라인 함수가 아닌 일반 함수에서는 사용할 수 없다.**

```kotlin
inline fun <reified T> func2(data: T) {
    println("$data") // Hello            
    println("${T::class.java}") // class java.lang.String
    println("${T::class.simpleName}") // String 
}

fun main() {
    func2("Hello")
}
```

참고로 reify는 ‘구체화하다’ 라는 뜻을 갖고 있다. 따라서 reified 키워드는 **런타임에 제네릭 타입 T에 대한 정보를 구체화해서 알려준다**고 이해할 수 있다. 

# 참고자료

[[kotlin] 코틀린 차곡차곡 - 10. 인라인(inline) 함수와 reified 키워드](https://sabarada.tistory.com/176)

[[Kotlin] inline에 대하여 - inline, noinline, crossinline, reified](https://leveloper.tistory.com/171)

[[Kotlin] inline 사용법 (4) - crossinline 사용법](https://effortguy.tistory.com/296)

[[Kotlin] 4-2. 다양한 함수의 종류](https://velog.io/@jxlhe46/Kotlin-4-2)