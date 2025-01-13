# 코루틴의 예외 전파 

코루틴 실행 도중 예외가 발생하면, 해당 코루틴은 취소되고 **부모 코루틴으로 예외가 전파**된다. 부모 코루틴에서 예외 전파가 제한되지 않으면, 계속 상위 코루틴으로 전파되어 루트 코루틴까지 전파될 수 있다. 이처럼, **코루틴의 예외는 부모 코루틴 방향으로만 전파**된다.

그리고 코루틴이 예외를 전파받아 취소되면, **해당 코루틴의 하위에 있는 모든 코루틴에게 취소가 전파**된다. 이처럼, **코루틴의 취소는 자식 방향으로만 전파**된다. 

우선, 예외가 발생하지 않는 코드부터 한번 실행시켜보자. 

```kotlin 
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> { // 1번 
    launch { // 2번 
        launch { // 4번 
            printWithThread("코루틴 실행")
        }
        delay(100L)
        printWithThread("코루틴 실행")
    }
    launch { // 3번 
        delay(100L)
        printWithThread("코루틴 실행")
    }
}

fun printWithThread(params: Any) {
    println("[${Thread.currentThread().name}] $params")
}
```

```
[main @coroutine#4] 코루틴 실행
[main @coroutine#2] 코루틴 실행
[main @coroutine#3] 코루틴 실행
```

그렇다면, 4번 코루틴에서 예외가 발생하면 어떻게 될까? 

```kotlin 
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> { // 1번
    launch { // 2번
        launch { // 4번
            throw Exception("예외 발생")
        }
        delay(100L)
        printWithThread("코루틴 실행")
    }
    launch { // 3번
        delay(100L)
        printWithThread("코루틴 실행")
    }
}

fun printWithThread(params: Any) {
    println("[${Thread.currentThread().name}] $params")
}
```

```
Exception in thread "main" java.lang.Exception: 예외 발생
	at MainKt$main$1$1$1.invokeSuspend(Main.kt:6)
	at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
	at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:108)
	at kotlinx.coroutines.EventLoopImplBase.processNextEvent(EventLoop.common.kt:280)
	at kotlinx.coroutines.BlockingCoroutine.joinBlocking(Builders.kt:85)
	at kotlinx.coroutines.BuildersKt__BuildersKt.runBlocking(Builders.kt:59)
	at kotlinx.coroutines.BuildersKt.runBlocking(Unknown Source)
	at kotlinx.coroutines.BuildersKt__BuildersKt.runBlocking$default(Builders.kt:38)
	at kotlinx.coroutines.BuildersKt.runBlocking$default(Unknown Source)
	at MainKt.main(Main.kt:3)
	at MainKt.main(Main.kt)
```

<img width="500" src="https://github.com/user-attachments/assets/4433166a-7e60-4f5c-b5d3-34a8dfa7d0ed" />

위의 그림과 같이 4번 코루틴에서 상위 코루틴으로 계속 예외가 전파되어, 결국 모든 코루틴이 취소된다. 

# 예외 전파 제한 

## 구조적 동시성을 깨는 방법 

**CoroutineContext에 새로운 Job 객체를 전달하여 독립적인 코루틴을 생성**하면, 구조적 동시성이 깨지게 된다. 즉, 부모-자식 관계가 더 이상 성립하지 않으므로, 부모 방향으로의 **예외 전파를 제한**할 수 있다. 
```kotlin 
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        launch(Job()) { // 독립적인 코루틴 생성
            launch {
                throw Exception("5번 코루틴 예외 발생")
            }
            delay(100L)
            printWithThread("3번 코루틴 실행")
        }

        launch {
            delay(100L)
            printWithThread("4번 코루틴 실행")
        }

        printWithThread("2번 코루틴 실행")
    }

    printWithThread("1번 코루틴 실행")
}

fun printWithThread(params: Any) {
    println("[${Thread.currentThread().name}] $params")
}
```

```
[main @coroutine#1] 1번 코루틴 실행
[main @coroutine#2] 2번 코루틴 실행
Exception in thread "main @coroutine#3" java.lang.Exception: 5번 코루틴 예외 발생
	at MainKt$main$1$1$1$1.invokeSuspend(Main.kt:7)
	at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
	at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:108)
	at kotlinx.coroutines.EventLoopImplBase.processNextEvent(EventLoop.common.kt:280)
	at kotlinx.coroutines.BlockingCoroutine.joinBlocking(Builders.kt:85)
	at kotlinx.coroutines.BuildersKt__BuildersKt.runBlocking(Builders.kt:59)
	at kotlinx.coroutines.BuildersKt.runBlocking(Unknown Source)
	at kotlinx.coroutines.BuildersKt__BuildersKt.runBlocking$default(Builders.kt:38)
	at kotlinx.coroutines.BuildersKt.runBlocking$default(Unknown Source)
	at MainKt.main(Main.kt:3)
	at MainKt.main(Main.kt)
	Suppressed: kotlinx.coroutines.internal.DiagnosticCoroutineContextException: [CoroutineId(3), "coroutine#3":StandaloneCoroutine{Cancelling}@3abbfa04, BlockingEventLoop@57fffcd7]
[main @coroutine#4] 4번 코루틴 실행
```

<img width="417" src="https://github.com/user-attachments/assets/325066ab-28a6-4651-a042-5cb5ad792b0c" />

3번은 새로운 Job 객체를 기반으로 독립적인 코루틴을 생성했기 때문에, 위와 같은 실행 결과가 나온다. 

하지만 이렇게 구조적 동시성을 깨는 방법은 **독립적인 코루틴 계층**을 구성하기 때문에, **취소가 의도한 대로 전파되지 않는다**는 한계가 있다. 

예를 들어, 2번 코루틴을 취소하면 4번 코루틴에만 취소가 전파될 뿐, 다른 코루틴에는 취소가 전파되지 않는다.

## SupervisorJob 

SupervisorJob 객체는 **자식 코루틴으로부터 예외를 전파받지 않는 특수한 Job 객체**이다. 예외를 전파받지 않기 때문에 하나의 자식 코루틴에서 예외가 발생해도, 다른 자식 코루틴에게 영향을 미치지 않는다.

```kotlin 
// parent 인자에 부모 Job 명시적으로 지정 가능 
public fun SupervisorJob(parent: Job? = null) : CompletableJob = SupervisorJobImpl(parent)

// 자식 코루틴이 취소되지 않도록 메서드 오버라이딩 
private class SupervisorJobImpl(parent: Job?) : JobImpl(parent) {
    override fun childCancelled(cause: Throwable): Boolean = false
}
```

### 1) CoroutineScope, SupervisorJob 함께 사용하는 경우 

```kotlin 
import kotlinx.coroutines.*

fun main() = runBlocking {
    val coroutineScope = CoroutineScope(SupervisorJob())

    coroutineScope.launch {
        launch {
            throw Exception("4번 코루틴 예외 발생")
        }
        delay(100L)
        printWithThread("2번 코루틴 실행")
    }

    coroutineScope.launch {
        delay(100L)
        printWithThread("3번 코루틴 실행")
    }

    delay(1000L)
    printWithThread("1번 코루틴 실행")
}


fun printWithThread(params: Any) {
    println("[${Thread.currentThread().name}] $params")
}
```

```
Exception in thread "DefaultDispatcher-worker-1 @coroutine#2" java.lang.Exception: 4번 코루틴 예외 발생
	at MainKt$main$1$1$1.invokeSuspend(Main.kt:8)
	at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
	at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:108)
	at kotlinx.coroutines.scheduling.CoroutineScheduler.runSafely(CoroutineScheduler.kt:584)
	at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.executeTask(CoroutineScheduler.kt:793)
	at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.runWorker(CoroutineScheduler.kt:697)
	at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.run(CoroutineScheduler.kt:684)
	Suppressed: kotlinx.coroutines.internal.DiagnosticCoroutineContextException: [CoroutineId(2), "coroutine#2":StandaloneCoroutine{Cancelling}@5a989620, Dispatchers.Default]
[DefaultDispatcher-worker-1 @coroutine#3] 3번 코루틴 실행
[main @coroutine#1] 1번 코루틴 실행
```

<img width="476" src="https://github.com/user-attachments/assets/07bc0b97-90ba-4bc4-afe7-7610c0c04ba6" />

4번 코루틴에서 예외가 발생해도 2번 코루틴에만 전파되기 때문에, 1번과 3번 코루틴은 정상적으로 실행된 것을 확인할 수 있다. 

### 2) SupervisorJob만 사용하는 경우 

```kotlin 
import kotlinx.coroutines.*

fun main() = runBlocking {
    val supervisorJob = SupervisorJob()
    
    launch(supervisorJob) {
        launch {
            throw Exception("4번 코루틴 예외 발생")
        }
        delay(100L)
        printWithThread("2번 코루틴 실행")
    }

    launch(supervisorJob) {
        delay(100L)
        printWithThread("3번 코루틴 실행")
    }

    delay(1000L)
    printWithThread("1번 코루틴 실행")
}

fun printWithThread(params: Any) {
    println("[${Thread.currentThread().name}] $params")
}
```

```
Exception in thread "main @coroutine#2" java.lang.Exception: 4번 코루틴 예외 발생
	at MainKt$main$1$1$1.invokeSuspend(Main.kt:8)
	at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
	at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:108)
	at kotlinx.coroutines.EventLoopImplBase.processNextEvent(EventLoop.common.kt:280)
	at kotlinx.coroutines.BlockingCoroutine.joinBlocking(Builders.kt:85)
	at kotlinx.coroutines.BuildersKt__BuildersKt.runBlocking(Builders.kt:59)
	at kotlinx.coroutines.BuildersKt.runBlocking(Unknown Source)
	at kotlinx.coroutines.BuildersKt__BuildersKt.runBlocking$default(Builders.kt:38)
	at kotlinx.coroutines.BuildersKt.runBlocking$default(Unknown Source)
	at MainKt.main(Main.kt:3)
	at MainKt.main(Main.kt)
	Suppressed: kotlinx.coroutines.internal.DiagnosticCoroutineContextException: [CoroutineId(2), "coroutine#2":StandaloneCoroutine{Cancelling}@1a968a59, BlockingEventLoop@4667ae56]
[main @coroutine#3] 3번 코루틴 실행
[main @coroutine#1] 1번 코루틴 실행
```

앞서 살펴본 CoroutineScope(SupervisorJob())로 코루틴 스코프를 생성하는 방법과 이번 방법은 어떤 차이점이 있을까? 

바로 runBlocking의 CoroutineScope로 launch 함수를 실행시켰기 때문에, **runBlocking의 CoroutineContext를 상속 받는다**는 점이 다르다. 

따라서, 2번과 3번 코루틴의 CoroutineDispatcher, CoroutineExceptionHandler는 runBlocking 코루틴의 CoroutineDispatcher, CoroutineExceptionHandler와 동일하다. 

<img width="476" src="https://github.com/user-attachments/assets/07bc0b97-90ba-4bc4-afe7-7610c0c04ba6" />

위의 그림을 통해 **runBlocking과의 구조적 동시성이 깨져있다**는 걸 확인할 수 있다. 구조적 동시성을 깨지 않고 SupervisorJob을 사용할 수는 없을까? 

바로 다음과 같이 **parent 인자에 부모 Job을 명시적으로 지정**하면 된다. 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    // supervisorJob의 parent로 runBlocking으로 생성된 Job 객체 설정
    val supervisorJob = SupervisorJob(parent = this.coroutineContext[Job])

    launch(supervisorJob) {
        launch {
            throw Exception("4번 코루틴 예외 발생")
        }
        delay(100L)
        printWithThread("2번 코루틴 실행")
    }

    launch(supervisorJob) {
        delay(100L)
        printWithThread("3번 코루틴 실행")
    }

    delay(1000L)
    printWithThread("1번 코루틴 실행")
    supervisorJob.complete() // 명시적으로 완료 처리를 해줘야 한다. 
}

fun printWithThread(params: Any) {
    println("[${Thread.currentThread().name}] $params")
}
```

<img width="460" src="https://github.com/user-attachments/assets/5d8e8778-d15f-42be-8920-0878c0d4b380" />

위의 그림처럼 **runBlocking과의 구조적 동시성을 깨지 않을 수 있다.** 

단, Job 함수를 통해 생성된 Job 객체와 같이, SupervisorJob 함수를 통해 생성된 Job 객체도 **자동으로 완료 처리되지 않는다.** 

그래서 **complete() 함수를 통해 명시적으로 완료 처리**를 해줘야 한다.

### 3) supervisorScope 사용하는 경우 

```kotlin 
import kotlinx.coroutines.*

fun main() = runBlocking {
    supervisorScope {
        launch {
            launch {
                throw Exception("4번 코루틴 예외 발생")
            }
            delay(100L)
            printWithThread("2번 코루틴 실행")
        }

        launch {
            delay(100L)
            printWithThread("3번 코루틴 실행")
        }
    }

    delay(1000L)
    printWithThread("1번 코루틴 실행")
}

fun printWithThread(params: Any) {
    println("[${Thread.currentThread().name}] $params")
}
```

supervisorScope은 **예외 전파를 제한**하면서 runBlocking 코루틴과의 **구조적 동시성을 깨지 않는다.** 

그리고 자식 코루틴들이 모두 실행 완료되면, **자동으로 완료 처리**도 해준다. 그래서 complete() 함수로 명시적으로 완료 처리를 할 필요가 없다. 

### SupervisorJob 사용 시 주의할 점 

```kotlin 
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch(SupervisorJob()) {
        launch {
            launch {
                throw Exception("5번 코루틴 예외 발생")
            }
            delay(100L)
            printWithThread("3번 코루틴 실행")
        }

        launch{
            delay(100L)
            printWithThread("4번 코루틴 실행")
        }

        delay(100L)
        printWithThread("2번 코루틴 실행")
    }

    delay(1000L)
    printWithThread("1번 코루틴 실행")
}

fun printWithThread(params: Any) {
    println("[${Thread.currentThread().name}] $params")
}
```

```
Exception in thread "main" java.lang.Exception: 5번 코루틴 예외 발생
	at MainKt$main$1$1$1$1.invokeSuspend(Main.kt:7)
	at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
	at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:108)
	at kotlinx.coroutines.EventLoopImplBase.processNextEvent(EventLoop.common.kt:280)
	at kotlinx.coroutines.BlockingCoroutine.joinBlocking(Builders.kt:85)
	at kotlinx.coroutines.BuildersKt__BuildersKt.runBlocking(Builders.kt:59)
	at kotlinx.coroutines.BuildersKt.runBlocking(Unknown Source)
	at kotlinx.coroutines.BuildersKt__BuildersKt.runBlocking$default(Builders.kt:38)
	at kotlinx.coroutines.BuildersKt.runBlocking$default(Unknown Source)
	at MainKt.main(Main.kt:3)
	at MainKt.main(Main.kt)
	Suppressed: kotlinx.coroutines.internal.DiagnosticCoroutineContextException: [StandaloneCoroutine{Cancelling}@1fbc7afb, BlockingEventLoop@45c8e616]
[main] 1번 코루틴 실행
```

<img width="485" src="https://github.com/user-attachments/assets/6bf0bcec-c40d-4047-bfa8-a28d81940393" />

코루틴의 부모-자식 관계를 그려보면, 실질적으로 2번 코루틴의 자식으로 하위 코루틴들이 생성되어 있기 때문에 4번 코루틴으로 취소가 전파된다. 즉, SupervisorJob이 예외 전파 제한 역할을 제대로 수행하고 있지 못한 것이다. 그래서 위의 실행 결과를 보면, 1번 코루틴만 실행되고 나머지는 모두 취소된 것을 확인할 수 있다. 

안드로이드에서도 흔히 이와 같은 문제가 발생할 수 있는데, 그 이유는 lifecycleScope, viewModelScope가 내부적으로 SupervisorJob을 사용하고 있기 때문이다. 

```kotlin 
class ExampleViewModel : ViewModel() {
    fun test() { 
        viewModelScope.launch {
            launch(CoroutineName("Coroutine1")) {
                launch(CoroutineName("Coroutine3")) {
                    throw Exception("예외 발생")
                }
                delay(100L)
                Timber.d("[${Thread.currentThread().name}] Coroutine1 코루틴 실행")
            }
            launch(CoroutineName("Coroutine2")) {
                delay(100L)
                Timber.d("[${Thread.currentThread().name}] Coroutine2 코루틴 실행")
            }
        }
	}
}
```

그래서 위와 같이 코루틴 스코프를 생성하면, 상위 코루틴으로 예외가 전파되어 2번 코루틴까지 취소된다. 

```kotlin 
class ExampleViewModel : ViewModel() {
    fun test() { 
        viewModelScope.launch(CoroutineName("Coroutine1")) {
            launch(CoroutineName("Coroutine3")) {
                throw Exception("예외 발생")
            }
            delay(100L)
            Timber.d("[${Thread.currentThread().name}] Coroutine1 코루틴 실행")
        }
        
        viewModelScope.launch(CoroutineName("Coroutine2")) {
            delay(100L)
            Timber.d("[${Thread.currentThread().name}] Coroutine2 코루틴 실행")
        }
	}
}
```

이렇게 코드를 수정하면 각 코루틴 스코프에서 독립적인 SupervisorJob 객체를 갖고 있으므로, 3번 코루틴에서 예외가 발생해도 2번 코루틴은 정상적으로 실행된다. 

# 참고자료 

- https://velog.io/@sdhong0609/코루틴의-예외-전파
- https://thdev.tech/coroutines/2024/12/08/Kotlin-Coroutines-effect-exception/

