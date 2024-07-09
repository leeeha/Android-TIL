# 코루틴 빌더

새로운 코루틴을 만들어 **루틴 세계와 코루틴 세계를 이어주는 함수**를 코루틴 빌더라고 한다. 

# runBlocking

`runBlocking` 함수는 그 안에 있는 **코루틴이 모두 완료될 때까지 현재 스레드를 블로킹** 시킨다. 스레드가 블락되면 그 스레드는 블락이 풀릴 때까지 다른 코드를 실행시킬 수 없다. 

```kotlin
expect fun <T> runBlocking(
    context: CoroutineContext = EmptyCoroutineContext, 
    block: suspend CoroutineScope.() -> T
): T
```

```kotlin
fun main() {
	runBlocking { // coroutine#1
		printWithThread("START")
		launch { // coroutine#2
			delay(2_000L)
			printWithThread("LAUNCH END")
		}
	}
	
	printWithThread("END")
}
```

main 함수에는 `runBlocking`으로 만들어진 코루틴이 있고, 그 안에는 다시 `launch`로 만들어진 코루틴이 있다. `launch` 블록 안에서 사용된 delay 함수는 **코루틴을 지정된 시간동안 지연시키는 함수**이다. 

위 코드에서 END가 출력되려면 `runBlocking` 때문에 **두 개의 코루틴이 완전히 종료되어야** 하고 실행 결과는 다음과 같다. 

```
[main @coroutine#1] START
[main @coroutine#2] LAUNCH END
[main] END
```

여기서 핵심은 END가 출력되려면 아무런 의미없이 2초를 기다려야 한다는 것이다. 즉, `runBlocking` 함수를 함부로 사용하면 **스레드가 블락되어 다른 코드를 실행하지 못하고 프로그램이 멈출 수도 있다.** 따라서 `runBlocking` 함수는 계속 사용하면 안 되고, 프로그램이 진입하는 최초의 메인 함수 또는 테스트 코드를 시작할 때만 사용하는 편이 좋다. 

# launch

launch 역시 코루틴 빌더이며, **반환값이 없는 코드를 실행할 때 사용**된다. 

launch는 runBlocking과 다르게 **만들어진 코루틴을 Job 객체로 반환**하고, 이 객체를 이용해 **코루틴을 ‘제어’할 수 있다.**

```kotlin
fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext, 
    start: CoroutineStart = CoroutineStart.DEFAULT, 
    block: suspend CoroutineScope.() -> Unit
): Job
```

```kotlin
fun main(): Unit = runBlocking {
	val job = launch {
		printWithThread("Hello launch") 
	}
}
```

코루틴을 제어한다? Job 객체로 **코루틴의 시작, 취소, 종료 시까지 대기**하게 만드는 것이 가능하다는 뜻이다.

## 코루틴의 시작

```kotlin
fun main(): Unit = runBlocking {
	val job = launch(start = CoroutineStart.LAZY) {
		printWithThread("Hello launch") 
	}
	
	delay(1_000L)
	job.start() // 이때 코루틴 시작 
}
```

`launch` 빌더에 `CoroutineStart.LAZY` 옵션을 주어 **코루틴이 즉시 실행되지 않도록 변경**했다. 따라서 이 코루틴은 `job.start()` 를 호출해서 시작 신호를 주어야만 실행된다. 

우리가 **코루틴이 시작되는 시점을 제어**한 것이다.

## 코루틴의 취소

```kotlin
fun main(): Unit = runBlocking {
	val job = launch {
		(1..5).forEach {
			printWithThread(it)
			delay(500) 
		}
	}
	
	delay(1_000L)
	job.cancel() // 코루틴 취소 
}
```

```
[main @coroutine#2] 1
[main @coroutine#2] 2
```

`cancel()` 함수를 사용하지 않으면 1~5까지 0.5초 간격으로 출력되어야 하지만, 1초가 지나고 `cancel()` 함수로 코루틴을 취소했기 때문에 1, 2까지만 출력된다. 

## 코루틴 종료까지 대기

```kotlin
fun main(): Unit = runBlocking {
	val job1 = launch {
		delay(1_000L)
		printWithThread("Job 1") 
	}
	
	val job2 = launch {
		delay(1_000L)
		printWithThread("Job 2") 
	}
}
```

위의 코드를 실행하는 데 시간이 얼마나 걸릴까? delay 함수를 두 번 사용했으니 2초일까? 

NO! 답은 1.1초 정도 걸린다! job1에서 1초 대기하는 동안 job2도 시작되어 **두 개의 코루틴이 같이 1초를 대기**하기 때문이다. 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/784beb7a-5d48-48da-8d68-714ee7635d05"/>

<br>

```kotlin
fun main(): Unit = runBlocking {
	val job1 = launch {
		delay(1_000L)
		printWithThread("Job 1") 
	}
	job1.join() // job1의 코루틴이 종료될 때까지 대기 
	
	val job2 = launch {
		delay(1_000L)
		printWithThread("Job 2") 
	}
}
```

job1 객체에서 `join()` 함수를 실행시키면, 해당 **코루틴이 끝날 때까지 대기**하게 된다. 따라서 위의 코드는 실행 시간이 약 2초 이상으로 늘어난다. 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/693f82de-0ce8-4dd3-adcc-8bbcb913bbb6"/>

# async

`async`는 `launch`와 거의 유사한데 딱 한가지 다른 점이 있다. 주어진 함수의 실행 결과를 반환할 수 없는 launch와 달리, async는 **주어진 함수의 실행 결과를 반환**할 수 있다. 

```kotlin
fun <T> CoroutineScope.async(
    context: CoroutineContext = EmptyCoroutineContext, 
    start: CoroutineStart = CoroutineStart.DEFAULT, 
    block: suspend CoroutineScope.() -> T
): Deferred<T>
```

```kotlin
fun main(): Unit = runBlocking {
	val job = async {
		3 + 5 
	}
}
```

async 역시 launch처럼 **코루틴을 제어할 수 있는 객체**를 반환하며, 그 타입은 `Deferred`이다. 

`Deferred`는 **Job의 하위 타입**으로 Job과 동일한 기능들이 있고, **async에서 실행된 결과를 가져오는 await 함수**가 추가적으로 존재한다. 

<img width="500" src="https://github.com/leeeha/Android-TIL/assets/68090939/167e3ba4-e8f7-48db-8c57-36e3fdd99c48"/>

```kotlin
fun main(): Unit = runBlocking {
	val job = async {
		3 + 5 
	}
	
	val eight = job.await() // async 블록의 실행 결과를 가져옴.
}
```

async 함수는 **여러 api를 동시에 호출하는 상황**에서 유용하게 사용될 수 있다. 

```kotlin
fun main(): Unit = runBlocking {
	val time = measureTimeMillis {
		val job1 = async { apiCall1() }
		val job2 = async { apiCall2() }
		printWithThread(job1.await() + job2.await())
	}
	
	printWithThread("소요 시간: $time ms")
}

suspend fun apiCall1(): Int {
	delay(1_000L)
	return 1 
}

suspend fun apiCall2(): Int {
	delay(1_000L)
	return 2
}
```

```
[main @coroutine#1] 3
[main @coroutine#1] 소요 시간: 1013 ms
```

async 함수를 활용해 **두 개의 api를 각각 호출함으로써 소요 시간을 최소화** 할 수 있다. 실행 시간을 확인해보면 약 1초 정도 걸린다. 

또한 첫번째 api의 결과가 두번째 api에 필요한 경우에는 **콜백 없이 동기식으로 코드를 작성**할 수 있다. (callback hell을 방지할 수 있다. 👍) 

```kotlin
fun main(): Unit = runBlocking {
	val time = measureTimeMillis {
		val job1 = async { apiCall1() }
		val job2 = async { apiCall2(job1.await()) }
		printWithThread(job2.await())
	}
	
	printWithThread("소요 시간: $time ms")
}

suspend fun apiCall1(): Int {
	delay(1_000L)
	return 1 
}

suspend fun apiCall2(num: Int): Int {
	delay(1_000L)
	return num + 2 
}
```

```
[main @coroutine#1] 3
[main @coroutine#1] 소요 시간: 2024 ms
```

- apiCall1 함수가 결과를 반환할 때까지 1초 대기
- job1 객체의 await 함수가 반환하는 값을 apiCall2 함수에 넣어서 실행
- 총 실행 시간 2초 이상

`async` 함수와 관련해 한 가지 주의할 점은 `CoroutineStart.LAZY` 옵션을 사용해 코루틴을 지연 실행시키면 

`await` 함수 호출 시 계산 결과를 계속해서 기다린다는 것이다. 

```kotlin
fun main(): Unit = runBlocking {
	val time = measureTimeMillis {
		val job1 = async(start = CoroutineStart.LAZY) { apiCall1() }
		val job2 = async(start = CoroutineStart.LAZY) { apiCall2() }
		printWithThread(job1.await() + job2.await())
	}
	
	printWithThread("소요 시간: $time ms")
}

suspend fun apiCall1(): Int {
	delay(1_000L)
	return 1 
}

suspend fun apiCall2(): Int {
	delay(1_000L)
	return 2
}
```

```
[main @coroutine#1] 3
[main @coroutine#1] 소요 시간: 2024 ms
```

만약 지연 코루틴을 async와 함께 사용하는 경우에도 동시에 api 호출을 하고 싶다면, start 함수를 먼저 사용해야 한다. 

```kotlin
fun main(): Unit = runBlocking {
	val time = measureTimeMillis {
		val job1 = async(start = CoroutineStart.LAZY) { apiCall1() }
		val job2 = async(start = CoroutineStart.LAZY) { apiCall2() }
		
		job1.start()
		job2.start()
		printWithThread(job1.await() + job2.await())
	}
	
	printWithThread("소요 시간: $time ms")
}

suspend fun apiCall1(): Int {
	delay(1_000L)
	return 1 
}

suspend fun apiCall2(): Int {
	delay(1_000L)
	return 2
}
```

```
[main @coroutine#1] 3
[main @coroutine#1] 소요 시간: 1013 ms
```