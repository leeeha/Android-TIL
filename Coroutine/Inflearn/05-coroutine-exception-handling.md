# 코루틴의 예외 처리

## 부모 코루틴, 자식 코루틴

```kotlin
fun main(): Unit = runBlocking { // coroutine#1
	val job1 = launch { // coroutine#2
		delay(1_000L)
		printWithThread("Job 1")	
	}
	
	val job2 = launch { // coroutine#3
		delay(1_000L)
		printWithThread("Job 2")	
	}
}
```

위의 코드에서 코루틴은 총 3개이다. runBlocking으로 만들어진 코루틴 안에 launch로 다시 2가지 코루틴이 만들어졌다. 

이때 runBlocking으로 만들어진 코루틴은 최상위 코루틴, 즉 **루트 코루틴**이자 **부모 코루틴**이 되고, launch로 만들어진 2가지 코루틴은 각각 **자식 코루틴**이 된다. 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/4f0c627e-55b4-4d2e-a5c6-a49a39cc0a89"/>

<br>

그렇다면, 만약 새로운 루트 코루틴을 만들고 싶으면 어떻게 해야 할까? launch로 코루틴을 생성할 때 **새로운 영역**에서 만들면 된다. 그 방법은 간단하다! 

**CoroutineScope 함수를 이용해 새로운 영역**을 만들고, 그 영역에서 launch를 호출하면 된다. 

```kotlin
fun main(): Unit = runBlocking {
	val job1 = CoroutineScope(Dispatchers.Default).launch {
		delay(1_000L)
		printWithThread("Job 1")
	}
	
	val job2 = CoroutineScope(Dispatchers.Default).launch {
		delay(1_000L)
		printWithThread("Job 2")	
	}
}
```

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/9149722f-f5db-4c89-a5a0-073bad9de116"/>

이제 우리는 루트 코루틴을 만드는 방법을 알게 되었다 🙂

## launch, async의 예외 발생 차이

자 그럼 이제 launch, async의 예외 발생 차이에 대해 알아보자! 먼저 launch이다. 

```kotlin
fun main(): Unit = runBlocking {
	val job = CoroutineScope(Dispatchers.Default).launch {
		throw IllegalArgumentException()
	}
	
	delay(1_000L)
}

// 실행 결과 
// Exception in thread "DefaultDispatcher-worker-1 @coroutine#2" java.lang.IllegalArgumentException
```

launch 함수는 **예외가 발생하자마자 해당 예외를 출력하고 코루틴이 종료**된다.

```kotlin
fun main(): Unit = runBlocking {
	val job = CoroutineScope(Dispatchers.Default).async {
		throw IllegalArgumentException()
	}
	
	delay(1_000L)
}

// 실행 결과 
// 
```

```kotlin
fun main(): Unit = runBlocking {
	val job = CoroutineScope(Dispatchers.Default).async {
		throw IllegalArgumentException()
	}
	
	delay(1_000L)
	job.await() // 이때 예외 발생 
}

// 실행 결과 
// Exception in thread "main" java.lang.IllegalArgumentException
```

반면, async 함수는 **예외가 발생해도 예외를 출력하지 않는다.** 이를 확인하고 싶다면 **await 함수**를 사용해야 한다. 

async는 launch와 다르게 값을 반환하는 코루틴에 사용되기에, **예외 역시 값을 반환할 때 처리할 수 있도록 설계**된 것이다. 

자 그럼 이번에는 새로운 영역에 루트 코루틴을 만들지 않고, **runBlocking 코루틴의 자식 코루틴으로** 만들어보자. 

이 경우에는 launch, async 모두 예외가 발생하면 바로 에러 로그를 확인할 수 있다. 

```kotlin
fun main(): Unit = runBlocking {
	val job = launch { // async로 변경해도 결과 동일 
		throw IllegalArgumentException()
	}
	
	delay(1_000L)
}

// 실행 결과 
// Exception in thread "main" java.lang.IllegalArgumentException
```

분명 async는 예외를 즉시 반환하지 않는다고 했는데, 왜 이런 차이가 생기는 걸까?! 

그것은 바로 **코루틴 안에서 발생한 예외가 부모 코루틴으로 전파**되기 때문이다. runBlocking 안에 있는 async 코루틴에서 예외가 발생하면, 그 예외는 부모 코루틴으로 전파되고, 부모 코루틴도 취소되는 절차로 들어가게 된다.

runBlocking의 경우 예외가 발생하면 해당 예외를 출력하므로, async의 예외를 받아 즉시 출력하는 것이다. 

그렇다면, **부모 코루틴에게 예외를 전파시키지 않는 방법**은 없을까? 

바로 `SupervisorJob()`을 사용하는 것이다! 

```kotlin
fun main(): Unit = runBlocking {
	val job = async(SupervisorJob()) {
		throw IllegalArgumentException()
	}
	
	delay(1_000L)
}

// 실행 결과 
// 
```

```kotlin
fun main(): Unit = runBlocking {
	val job = async(SupervisorJob()) {
		throw IllegalArgumentException()
	}
	
	delay(1_000L)
	job.await() // 이때 예외 발생 
}

// 실행 결과 
// Exception in thread "main" java.lang.IllegalArgumentException
```

`SupervisorJob()`을 사용하면 async **자식 코루틴에서 발생한 예외가 부모 코루틴에게 전파되지 않고**, job.await()을 해야 예외가 발생하는 원래 행동 패턴으로 돌아가게 된다. 

## 코루틴의 예외 처리

코루틴에서 발생하는 예외를 핸들링 하고싶다면 어떻게 해야 할까? 가장 직관적인 방법은 **try-catch 구문을 사용**하는 것이다. 

```kotlin
fun main(): Unit = runBlocking {
    val job = launch {
        try {
            throw IllegalArgumentException()
        } catch (e: IllegalArgumentException) {
            printWithThread("정상 종료")
        }
    }
}

// 실행 결과 
// [main @coroutine#2] 정상 종료
```

위의 코드처럼 발생한 예외를 잡아 코루틴이 취소되지 않게 만들 수도 있고, 적절한 처리 후에 다시 예외를 던질 수도 있다. 

만약 try-catch 대신에 예외가 발생한 이후 **에러를 로깅하거나 일관적으로 예외 처리**를 하고 싶은 경우, `CoroutineExceptionHandler` 라는 객체를 활용할 수도 있다. 

`CoroutineExceptionHandler` 객체는 코루틴의 구성요소와 발생한 예외를 파라미터로 받을 수 있다.

```kotlin
fun main(): Unit = runBlocking {
    val exceptionHandler = CoroutineExceptionHandler { context, throwable -> 
            printWithThread("예외") 
    }

    // runBlocking과 다른 영역에 루트 코루틴 생성 
    val job = CoroutineScope(Dispatchers.Default).launch(exceptionHandler) {
        throw IllegalArgumentException()
    }
    
    delay(1_000L) 
}

// 실행 결과 
// [DefaultDispatcher-worker-1 @coroutine#2] 예외
```

다만, `CoroutineExceptionHandler`는 launch와 같이 **결과 값을 반환하지 않는 코루틴 빌더에서만 동작**하며, async에서는 동작하지 않는다. 

async는 Deferred 타입으로 결과 값을 반환하므로, try-catch 블록으로 예외를 직접 처리해야 한다. 

또한, `CoroutineExceptionHandler`는 **루트 코루틴일 때만 동작한다**는 것도 주의해야 한다. 

부모-자식 관계에서 **자식 코루틴의 예외는 기본적으로 부모로 전파**되기 때문에, CoroutineExceptionHandler는 루트 코루틴일 때만 동작한다. 

예외적으로, SupervisorJob으로 자식 코루틴의 예외가 부모로 전파되지 않게 제한하면, 자식 코루틴에서도 CoroutineExceptionHandler를 사용할 수 있다. 

## 코루틴의 취소 vs 예외

이전에 `CancellationException`을 던져서 코루틴을 취소시킬 수 있다고 했는데, 그럼 코루틴 입장에서 취소와 예외는 어떻게 다른 것일까? 

코루틴은 코루틴 내부에서 발생한 예외에 대해 다음과 같이 처리한다. 

1. 발생한 예외가 `CancellationException` 인 경우 → 취소로 간주하고 부모 코루틴에게 전파 X
2. 다른 예외가 발생한 경우 → 실패로 간주하고 부모 코루틴에게 전파 O 
    
그리고 코루틴은 예외가 발생하면, 해당 예외가 `CancellationException` 이든 다른 종류의 예외든 내부적으로 `CANCELLING` 상태로 간주한다. 

state machine으로 표현하면 다음과 같다. 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/60ccd43b-23b2-47c0-a0b9-bf40820e27de"/>

만약 예외가 발생하지 않고 코루틴이 정상적으로 처리되면 다음과 같은 흐름을 가진다. 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/66a34080-5584-4a90-bda6-b64b071969fc"/>

위의 그림에서 Completing, Completed 상태로 나뉘는 이유는 무엇일까??

다음 글에서 알아보도록 하자~!

# 참고 자료 

- [2시간으로 끝내는 코루틴, 최태현](https://www.inflearn.com/course/2%EC%8B%9C%EA%B0%84%EC%9C%BC%EB%A1%9C-%EB%81%9D%EB%82%B4%EB%8A%94-%EC%BD%94%EB%A3%A8%ED%8B%B4)