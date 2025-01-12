# Job의 생명주기

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/31f5afad-c07e-4764-9c1b-ac6416dfc4db"/>

위의 그림에서 주어진 작업이 완료된 코루틴은 바로 `COMPLETED`가 되는 게 아니라 `COMPLETING`으로 처리된다. 왜 한 단계를 거쳐서 가는 걸까? 

그 이유는 바로 자식 코루틴이 있을 때, `COMPLETING` 상태에서 **자식 코루틴이 모두 완료될 때까지 기다릴 수 있고**

자식 코루틴 중에 하나에서 예외가 발생하면 **다른 자식 코루틴들에게도 취소 요청**을 보낼 수 있기 때문이다.

```kotlin
fun main(): Unit = runBlocking {
	launch {
		delay(600L)
		printWithThread("A")
	}
	
	launch {
		delay(500L)
		throw IllegalArgumentException("코루틴 실패!")
	}
}

// 실행 결과 
// Exception in thread "main" java.lang.IllegalArgumentException: 코루틴 실패!
```

위의 코드에서 두번째 코루틴에 예외가 발생하면, runBlocking으로 만들어진 부모 코루틴에 취소 신호를 보내게 되고, 부모 코루틴은 이 신호를 받아서 다른 자식 코루틴인 첫번째 코루틴까지 취소시킨다. 

# 구조적 동시성

<img width="500" src="https://github.com/leeeha/Android-TIL/assets/68090939/8f622bae-0ef8-4e30-b8a1-fd0e2b35c35f"/>

이처럼 **부모-자식 관계의 코루틴이 한 몸처럼 움직이는 것**을 **구조적 동시성** (Structured Concurrency)이라고 부른다. 

코틀린 공식 문서에 의하면 Structured Concurrency는 

- **수많은 코루틴이 유실되거나 누수되지 않도록 보장**한다.
- **코드 내의 에러가 유실되지 않고 적절히 보고될 수 있도록 보장**한다.

코루틴의 취소 및 예외와 합쳐서 다시 정리해보면 다음과 같다. 

- **자식 코루틴에서 예외가 발생**할 경우, 구조적 동시성에 의해 **부모 코루틴이 취소되고 부모 코루틴의 다른 자식 코루틴들도 취소**된다. 
- 자식 코루틴에서 예외가 발생하지 않더라도, **부모 코루틴이 취소되면 자식 코루틴들도 취소**된다. 
- 다만 CancellationException의 경우 **정상적인 취소로 간주**하므로, 부모 코루틴에게 전파되지 않고 다른 자식 코루틴을 취소시키지 않는다.
- **예외는 부모 코루틴 방향으로만 전파되고, 취소는 자식 코루틴 방향으로만 전파된다.** 

# 참고 자료 

- [2시간으로 끝내는 코루틴, 최태현](https://www.inflearn.com/course/2%EC%8B%9C%EA%B0%84%EC%9C%BC%EB%A1%9C-%EB%81%9D%EB%82%B4%EB%8A%94-%EC%BD%94%EB%A3%A8%ED%8B%B4)
- https://velog.io/@sdhong0609/부모-코루틴과-자식-코루틴-상속되지-않는-Job
- https://velog.io/@sdhong0609/코루틴의-예외-전파
