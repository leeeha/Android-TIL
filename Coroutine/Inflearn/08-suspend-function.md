# suspend 함수란?

함수 앞에 suspend 키워드가 붙으면 **다른 suspend 함수를 호출할 수 있는 능력**이 생긴다. 

저번 글에서 메인 함수에서도 suspend 함수인 delay 함수를 호출한 적이 있다. 이 코드는 어떻게 가능한 것일까? 

사실 전부터 사용했던 runBlocking, launch 코루틴 빌더에서 다른 함수를 호출할 때, 이 함수는 suspend 함수로 간주된다. 

실제 launch 함수의 시그니처를 확인해보면, 마지막 인자 타입에 suspend 키워드가 붙은 것을 확인할 수 있다. 이렇게 함수 타입에 suspend 키워드를 붙이면 **suspending lambda**가 된다. 

```kotlin
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job
```

이 외에 suspend가 가지는 다른 의미는 없을까? 

suspend 단어를 사전에서 찾아보면 **정지, 유예**라는 뜻을 가지고 있다. 즉, suspend 함수는 **코루틴이 중지 되었다가 재개 ‘될 수 있는’ 지점**을 의미한다. 해당 지점을 **suspension point** 라고 부르며, suspend 함수를 호출한다고 무조건 중지되는 것이 아니라, **중지가 될 수도 있고, 중지가 되지 않을 수도 있다**는 것이 핵심이다. 

예를 들어, 다음과 같은 코드에서 첫번째 launch 블록의 코루틴에서 suspend 함수인 a(), b()를 각각 호출하고 있어서, a()가 호출된 다음 중지되어 두번째 launch 블록의 코루틴이 실행될 거 같지만! suspend 함수를 호출해도 반드시 중지되는 것은 아니므로 a(), b(), c() 순차적으로 실행된다. 

```kotlin
fun main(): Unit = runBlocking {
  	launch {
		a()
		b() 
	}
	
	launch { 
		c()
	} 
}

suspend fun a() {
  printWithThread("A")
}

suspend fun b() {
  printWithThread("B")
}

suspend fun c() {
  printWithThread("C")
}

// 실행 결과 
// [main @coroutine#2] A
// [main @coroutine#2] B
// [main @coroutine#3] C
```

# suspend 함수의 활용

suspend 함수는 **연쇄적인 API를 호출할 때 활용**할 수 있다. 첫번째 API 호출에서 나온 결과를 두번째 API 호출 시 사용해야 한다면, 다음과 같이 코드를 작성할 수 있다. 

```kotlin
fun main(): Unit = runBlocking {
	val result1 = async {
		apiCall1()
	}
	
	val result2 = async {
		apiCall2(result1.await()) 
	}
	
	printWithThread(result2.await())
}

fun apiCall1(): Int {
	Thread.sleep(1_000L)
	return 100
}

fun apiCall2(num: Int): Int {
	Thread.sleep(1_000L)
	return num * 2 
}
```

async, Deferred 활용하여 **콜백 없이 동기식으로 코드를 작성**했다. 

하지만, runBlocking 입장에서 result1, result2 타입이 Deferred 이므로 Deferred에 의존적인 코드가 되는 것은 아쉽다. Deferred 대신에 CompletableFuture, Reactor와 같은 다른 비동기 라이브러리 코드로 갈아 끼워야 할 수도 있다. 

이럴 때 suspend 함수를 사용할 수 있다. 

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.future.await
import java.util.concurrent.CompletableFuture

fun main(): Unit = runBlocking {
    val result1 = apiCall1()
    val result2 = apiCall2(result1)
    printWithThread(result2)
}

suspend fun apiCall1(): Int {
    return CoroutineScope(Dispatchers.Default).async {
        Thread.sleep(1_000L)
        100
    }.await()
}

suspend fun apiCall2(num: Int): Int {
    return CompletableFuture.supplyAsync {
        Thread.sleep(1_000L)
        num * 2
    }.await()
}
```

apiCall1(), apiCall2() 함수를 **suspend로 변경**해 이 안에서 **어떤 비동기 라이브러리 구현체를 사용하든 해당 함수의 선택으로 남겨두는 것**이다. 

CompletableFuture에 사용한 await() 함수 역시 코루틴에서 만들어둔 suspend 함수이다. 이처럼 **코루틴에서는 다양한 비동기 라이브러리와 변환 코드를 제공**한다. 

suspend 함수는 인터페이스에서도 사용할 수 있다. 따라서 **인터페이스의 구현체별로 다른 비동기 라이브러리를 사용**할 수도 있다. 

# 코루틴에서 제공하는 suspend 함수

## coroutineScope

```kotlin 
suspend fun <R> coroutineScope(block: suspend CoroutineScope.() -> R): R
```

- **구조화를 깨지 않는** CoroutineScope 객체를 생성하여, 해당 범위 안에서 **여러 코루틴을 병렬 실행**할 수 있다.
- coroutineScope 블록 내부 코드가 **실행 완료될 때까지 호출부의 코루틴은 일시 중단**된다.
- 사용 예시: 여러 API의 실행 결과를 합쳐야 할 때

```kotlin
fun main(): Unit = runBlocking {
	printWithThread("START")
	printWithThread(calculateResult())
	printWithThread("END")
}

suspend fun calculateResult(): Int = coroutineScope {
	val num1 = async {
		delay(1_000L)
		10
	}
	
	val num2 = async {
		delay(1_000L)
		20 
	}
	
	num1.await() + num2.await()
}

// 실행 결과 
// [main @coroutine#1] START
// [main @coroutine#1] 30
// [main @coroutine#1] END
```

## withContext

```kotlin
suspend fun <T> withContext(context: CoroutineContext, block: suspend CoroutineScope.() -> T): T
```

- 기본적으로 coroutineScope와 동일한데, CoroutineContext에 변화를 줄 수 있다. 
- 주로 코루틴 **실행 스레드 교체 목적**으로 자주 사용된다. 
- 새로운 코루틴을 생성하지 않고, **기존 코루틴을 유지한 채 CoroutineContext만 변경**한다. 
- 사용 예시: 네트워크 호출이나 DB 접근 시 Dispatchers.IO로 전환할 때

```kotlin
fun main(): Unit = runBlocking {
	printWithThread("START")
	printWithThread(calculateResult())
	printWithThread("END")
}

suspend fun calculateResult(): Int = withContext(Dispatchers.IO) {
	val num1 = async {
		delay(1_000L)
		10
	}
	
	val num2 = async {
		delay(1_000L)
		20 
	}
	
	num1.await() + num2.await()
}

// 실행 결과 
// [main @coroutine#1] START
// [main @coroutine#1] 30
// [main @coroutine#1] END
```

## withTimeout, withTimeoutOrNull

- coroutineScope와 유사하지만, **주어진 함수 블록이 시간 내에 완료되어야 한다.**
- 주어진 시간 안에 코루틴이 완료되지 않으면
    - withTimeout: TimeoutCancellationException 발생
    - withTimeoutOrNull: null 반환

```kotlin
fun main(): Unit = runBlocking {
	withTimeout(1_000L) {
		delay(1_500L)
		10 + 20 
	}
}

// 실행 결과 
// Exception in thread "main" kotlinx.coroutines.TimeoutCancellationException: 
// Timed out waiting for 1000 ms
```

# 참고 자료 

- [2시간으로 끝내는 코루틴, 최태현 ](https://www.inflearn.com/course/2%EC%8B%9C%EA%B0%84%EC%9C%BC%EB%A1%9C-%EB%81%9D%EB%82%B4%EB%8A%94-%EC%BD%94%EB%A3%A8%ED%8B%B4)
