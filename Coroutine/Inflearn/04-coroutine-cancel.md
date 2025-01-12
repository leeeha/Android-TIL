# 코루틴의 취소 

더 이상 사용하지 않는 코루틴을 취소하는 것은 중요하다. 특히 여러 코루틴을 사용할 때, **필요 없어진 코루틴을 적절하게 취소해야 컴퓨터 자원을 절약**할 수 있다. 

코루틴을 취소하려면 저번 포스팅에서 살펴봤던 것처럼 Job 객체의 cancel() 함수를 사용할 수 있다. 단, **취소 대상인 코루틴도 취소에 협조해줘야 한다.** 

다음 예시를 살펴보자. 

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
	
	delay(1_00L)
	job1.cancel()
}

// 실행 결과 
// [main @coroutine#3] Job 2 
```

위의 코드에서 runBlocking 안에 두 개의 코루틴을 만들고 0.1초 지연시킨 후 첫번째 코루틴을 취소시켰다. 

첫번째 코루틴은 **취소에 잘 협조하고 있기 때문에** 정상적으로 취소되어 “Job 1”이 출력되지 않고, “Job 2”만 출력되었다.

이처럼 코루틴이 취소에 협조하게 만들기 위해 delay 함수를 사용할 수 있다. 더 근본적으로는 **delay(), yield() 같은 kotlinx.coroutines 패키지의 suspend 함수를 사용**하면 코루틴이 취소에 협조하게 만들 수 있다. 

# suspend 함수

코루틴의 취소에 협조하는 첫번째 방법은 **코루틴 패키지의 suspend 함수를 사용**하는 것이다. 

단, 여기서 주의할 점이 하나 있는데 **취소되기 전에 코루틴이 완료될 수도 있다**는 점이다. 

```kotlin
fun main(): Unit = runBlocking {
	val job = launch {
		delay(10L) // 0.01초  
		printWithThread("Job 1")
	}
	
	delay(100L) // 0.1초 
	job.cancel()
}

// 실행 결과 
// [main @coroutine#2] Job 1 
```

이런 경우는 취소되지 않은 것이 아니라, 취소되기 전에 코루틴이 진작 완료된 것이다! 

자 그럼, **코루틴이 취소에 협조하지 않으면 정말 취소되지 않을까?** 

코루틴 패키지의 suspend 함수인 delay 함수를 사용하지 않고 5번의 출력을 반복해보자.

```kotlin
fun main(): Unit = runBlocking {
    val job = launch {
        var i = 1
        var nextPrintTime = System.currentTimeMillis()
        while(i <= 5){
            if(nextPrintTime <= System.currentTimeMillis()) {
                printWithThread("${i++}번째 출력!")
                nextPrintTime += 1_000L
            }
        }
    }

    delay(100L)
    job.cancel()
}

// 실행 결과
// [main @coroutine#2] 1번째 출력!
// [main @coroutine#2] 2번째 출력!
// [main @coroutine#2] 3번째 출력!
// [main @coroutine#2] 4번째 출력!
// [main @coroutine#2] 5번째 출력!
```

- nextPrintTime <= 현재 시간 → print 함수 실행, nextPrintTime += 1_000L
- nextPrintTime > 현재 시간 → 1초가 지나서 if문을 만족시킬 때까지 while문 반복 (1초 delay)

while문 안의 if문이 위와 같이 동작하기 때문에 delay 함수와 동일한 효과를 볼 수 있다. 

<img width="800" src="https://github.com/leeeha/Android-TIL/assets/68090939/a13c71b0-e69e-44ef-b57f-8ac55ee29b95"/>

분명 job.cancel() 함수로 코루틴을 취소했지만, launch 블록에 의해 만들어진 코루틴은 1초 간격으로 5개의 문자열을 모두 출력할 때까지 취소되지 않는다. 

이를 통해 알 수 있는 것은, **협력하는 코루틴만 취소 가능하다**는 것이다.

# 코루틴 자신의 상태 확인

코루틴이 취소에 협력하게 만드는 두번째 방법은, **코루틴 스스로가 본인의 상태를 확인**하여 **취소 요청을 받았을 때 CancellationException을 던지는 것**이다. 

- `isActive`
    - 코루틴을 만들 때 사용한 함수 블록 안에서는 isActive라는 프로퍼티에 접근 가능하다.
    - 이 프로퍼티는 **현재 코루틴이 활성화 되어 있는지, 아니면 취소 신호를 받았는지 구분**할 수 있게 해준다.
- `Dispatchers.Default`
    - 취소 신호를 정상적으로 전달하려면, launch 블록으로 만든 코루틴이 **다른 스레드에서 동작**하게 만들어야 한다. 
    - Dispatchers를 통해 코루틴을 특정 스레드에 배정할 수 있다.

```kotlin
fun main(): Unit = runBlocking {
	val job = launch(Dispatchers.Default) { // 디스패처 지정 (다른 스레드에서 돌아가도록) 
		var i = 1 
		var nextPrintTime = System.currentTimeMillis()
		
		while(i <= 5){
			if(nextPrintTime <= System.currentTimeMillis()) {
				printWithThread("${i++}번째 출력!")
				nextPrintTime += 1_000L // 1초 지연 
			}
			
			if(!isActive) { // 코루틴 자신의 상태 확인 
				throw CancellationException() 
			}
		}
	}
	
	delay(100L)
	printWithThread("취소 시작")
	job.cancel()
}

// 실행 결과 
// [DefaultDispatcher-worker-1 @coroutine#2] 1번째 출력!
// [main @coroutine#1] 취소 시작
```

<img width="800" src="https://github.com/leeeha/Android-TIL/assets/68090939/6e80c292-0f0b-4e50-9b2d-e9985a194168"/>

Dispatchers.Default로 다른 스레드에 코루틴을 배정했기 때문에 실행 결과에서 **스레드 이름이 변경**된 것을 확인할 수 있다. 

또한, while문의 루프가 최초 한 번만 동작하고, 두번째 반복하기 전에 **cancel() 신호가 정상적으로 수신되어 코루틴이 취소**된 것을 알 수 있다. 

만약 위의 코드에서 launch 블록에 디스패처를 지정하지 않으면, launch에 의한 코루틴이 메인 스레드를 점유한 채 비켜주지 않기 때문에 1~5번째까지 모두 출력된다. 

```
[main @coroutine#2] 1번째 출력!
[main @coroutine#2] 2번째 출력!
[main @coroutine#2] 3번째 출력!
[main @coroutine#2] 4번째 출력!
[main @coroutine#2] 5번째 출력!
[main @coroutine#1] 취소 시작
```

CancellationException을 던지지 않고도 while문에서 isActive를 조건으로 루프를 돌리는 방법도 있다. 

```kotlin
while(isActive){
	if(nextPrintTime <= System.currentTimeMillis()) {
		printWithThread("${i++}번째 출력!")
		nextPrintTime += 1_000L
	}
}
```

여기까지의 내용을 요약해보면, 컴퓨팅 자원을 효율적으로 관리하기 위해서는 코루틴을 취소할 수 있어야 하고, 이를 위해서는 **취소될 코루틴이 적절한 협조**를 해줘야 한다. 

코루틴이 취소에 협조하게 만드는 방법은 다음 2가지가 있다. 

1. kotlinx.coroutines 패키지의 suspend 함수 호출 
2. 코루틴이 isActive로 스스로의 상태를 확인하여 CancellationException 던지기 

## CancellationException

CancellationException에 대해 조금 더 자세히 알아보자. 

사실 우리가 자주 사용하고 있는 **delay 함수 역시 내부적으로 CancellationException을 던지며 코루틴을 취소**시킨다.

따라서 try-catch문을 활용해 CancellationException을 잡은 뒤에 다시 던지지 않으면, 코루틴이 취소되지 않는다. 이럴 때는 **finally문으로 필요한 자원을 닫을 수 있다.** 

```kotlin
fun main(): Unit = runBlocking {
	val job = launch {
		try {
			 delay(1_000L)
		}catch(e: CancellationException){
			// 예외를 잡아서 먹어버린다!
		}
		
		// 코루틴이 취소되지 않고 실행됨.
		printWithThread("delay에 의해 취소되지 않았다!")
	}
	
	delay(100L)
	printWithThread("취소 시작")
	job.cancel()
}

// 실행 결과 
// [main @coroutine#1] 취소 시작 
// [main @coroutine#2] delay에 의해 취소되지 않았다! 
```

```kotlin
fun main(): Unit = runBlocking {
	val job = launch {
		try {
			 delay(1_000L)
		} finally {
			// 자원을 적절히 닫을 수 있다. 
		}
		printWithThread("delay에 의해 취소되지 않았다!")
	}
	
	delay(100L)
	printWithThread("취소 시작")
	job.cancel()
}

// 실행 결과 
// [main @coroutine#1] 취소 시작 
```

# 참고 자료 

- [2시간으로 끝내는 코루틴, 최태현](https://www.inflearn.com/course/2%EC%8B%9C%EA%B0%84%EC%9C%BC%EB%A1%9C-%EB%81%9D%EB%82%B4%EB%8A%94-%EC%BD%94%EB%A3%A8%ED%8B%B4)