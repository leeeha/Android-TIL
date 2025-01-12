# CoroutineScope

이전 글에서 CoroutineScope를 이미 활용한 적이 있다. 바로 **루트 코루틴을 만들기 위해 CoroutineScope을 이용해 새로운 영역을 만들고** launch로 코루틴을 생성했다. 

```kotlin
fun main(): Unit = runBlocking {
	val job1 = CoroutineScope(Dispatchers.Default).launch {
		delay(1_000L)
		printWithThread("Job 1")
	}
}
```

사실 우리가 사용했던 **launch, async와 같은 코루틴 빌더는 CoroutineScope의 확장함수**이다. 즉, launch, async를 사용하려면 CoroutineScope가 필요했던 것이다.

```kotlin
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job
```

```kotlin
public fun <T> CoroutineScope.async(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> T
): Deferred<T>
```

따라서 지금까지 사실은 runBlocking이 일반 세계와 코루틴 세계를 이어주며, 코루틴 스코프를 제공해주고 있었기 때문에, runBlocking 안에서 launch, async를 사용할 수 있었던 것이다. 

만약 우리가 직접 코루틴 스코프를 만든다면 runBlocking이 굳이 필요하지 않다. 

main 함수를 일반 함수로 만들어 코루틴이 끝날 때까지 main 스레드를 대기시킬 수도 있고

```kotlin
fun main() {
	val job = CoroutineScope(Dispatchers.Default).launch {
		delay(1_000L)
		printWithThread("Job 1")
	}
	
	Thread.sleep(1_500L) // job1 종료까지 대기 
}
```

 main 함수 자체를 suspend로 만들어 join() 함수로 코루틴 종료까지 대기할 수도 있다. 

```kotlin
suspend fun main() {
	val job = CoroutineScope(Dispatchers.Default).launch {
		delay(1_000L)
		printWithThread("Job 1")
	}
	
	job.join()
}
```

## 클래스 내부에서 독립적인 CoroutineScope 관리

한 영역에 있는 코루틴들은 **영역 자체를 cancel() 시킴으로써 모든 코루틴을 종료**시킬 수 있다. 

예를 들어, 다음 코드처럼 **클래스 내부에서 독립적인 CoroutineScope를 관리**한다면, **해당 클래스에서 사용하던 코루틴을 한번에 종료**시킬 수 있다. 

```kotlin
class AsyncLogic {
	private val scope = CoroutineScope(Dispatchers.Default)
	
	fun doSomething() {
		scope.launch {
			// 여기서 어떤 작업을 하고 있다.
		}
	}
	
	fun destroy() {
		scope.cancel()
	}
}
```

AsyncLogic 클래스의 인스턴스를 만들고, doSomething() 함수를 호출해 비동기 로직을 처리하다가 더 이상 필요가 없어지면, destroy() 함수를 호출해 모든 코루틴을 정리할 수 있다. 

```kotlin
val asyncLogic = AsyncLogic()
asyncLogic.doSomething()

asyncLogic.destroy() // 필요 없어지면 모두 정리
```

## 참고: GlobalScope 사용이 지양되는 이유 

- **어플리케이션의 생명주기와 일치**: 특정 컨텍스트 (뷰모델, 액티비티, 프래그먼트 등)가 사라져도 코루틴은 계속 실행되어 메모리 누수나 에러 발생으로 이어질 수 있음.  
- **구조적 동시성에 위배**: 부모 코루틴이 취소되어도 자식 코루틴이 취소되지 않는 문제 발생
- **테스트 및 디버깅의 어려움**: 전역적으로 코루틴이 실행되어, 문제 발생의 원인을 찾기 어려움.

# CoroutineContext

이 CoroutineScope의 주요한 역할은 **CoroutineContext라는 데이터를 보관하는 것**이다. 실제 CoroutineScope 인터페이스 역시 매우 단순하다. 

```kotlin
public interface CoroutineScope {
  public val coroutineContext: CoroutineContext
}
```

CoroutineContext는 **코루틴의 실행 환경 정보를 갖고 있는 인터페이스**로, 주요 구성 요소로는 CoroutineName, CoroutineDispatcher, Job, CoroutineExceptionHandler 이렇게 네가지가 있다. 

Dispatcher는 **코루틴이 어떤 스레드에 배정될지 관리하는 역할**을 한다. Dispatcher의 종류에 대해서는 잠시 후 다시 살펴보자. 

이제까지의 내용을 요약하면 다음과 같다. 

- **CoroutineScope: 코루틴이 탄생할 수 있는 영역**
- **CoroutineContext: 코루틴의 실행 환경 정보를 갖고 있음.**

## CoroutineContext 내부 구조 

```kotlin 
/**
 * Persistent context for the coroutine. It is an indexed set of [Element] instances.
 * An indexed set is a mix between a set and a map.
 * Every element in this set has a unique [Key].
 */
@SinceKotlin("1.3")
public interface CoroutineContext {
    // ...
    
    /**
     * An element of the [CoroutineContext]. An element of the coroutine context is a singleton context by itself.
     */
    public interface Element : CoroutineContext {
        /**
         * A key of this coroutine context element.
         */
        public val key: Key<*>

        public override operator fun <E : Element> get(key: Key<E>): E? =
            @Suppress("UNCHECKED_CAST")
            if (this.key == key) this as E else null

        public override fun <R> fold(initial: R, operation: (R, Element) -> R): R =
            operation(initial, this)

        public override fun minusKey(key: Key<*>): CoroutineContext =
            if (this.key == key) EmptyCoroutineContext else this
    }
}
```

CoroutineContext의 구현체를 살펴보면, Element 인스턴스들에 대한 indexed set라고 설명되어 있다. 

추가적으로 indexed set은 **set과 map을 혼합한 형태**이며, 이 **set의 모든 요소는 고유한 Key를 가진다**고 설명되어 있다.

즉, CoroutineContext 객체는 **Key-Value 쌍으로 구성 요소를 관리**하며, 동일한 키에 대해 중복 값을 허용하지 않는다. 만약 동일한 키에 대한 새로운 값을 추가하면 **기존에 존재하던 값은 새로운 값으로 덮어씌워진다.** 

따라서 CoroutineContext 객체는 CoroutineName, CoroutineDispatcher, Job, CoroutineExceptionHandler 객체를 각각 **한 개씩만** 가질 수 있다. 

### CoroutineContext 구성 요소의 Key는 싱글톤 객체 

```kotlin 
public data class CoroutineName(
    /**
     * User-defined coroutine name.
     */
    val name: String
) : AbstractCoroutineContextElement(CoroutineName) {
    /**
     * Key for [CoroutineName] instance in the coroutine context.
     */
    public companion object Key : CoroutineContext.Key<CoroutineName>

    /**
     * Returns a string representation of the object.
     */
    override fun toString(): String = "CoroutineName($name)"
}
```

예시로 CoroutineContext의 구성요소 중 하나인 CoroutineName을 보면, **Key가 companion object로 선언되어 있으므로 싱글톤 객체**라는 것을 알 수 있다. 즉, **서로 다른 코루틴 컨텍스트 인스턴스여도 Key 값은 동일**하다. 

```kotlin 
import kotlinx.coroutines.*

fun main() = runBlocking {
    val coroutineName1 = CoroutineName("Coroutine1")
    val coroutineName2 = CoroutineName("Coroutine2")

    println(coroutineName1.key === coroutineName2.key) // true
    println(Dispatchers.IO.key === Dispatchers.Default.key) // true
}
```

key 프로퍼티는 companion object로 선언된 Key와 동일한 객체를 가리킨다. 서로 다른 CoroutineName이어도 두 객체의 Key는 동일하다는 걸 확인할 수 있다. 

마찬가지로 서로 다른 Dispatcher여도 Key는 싱글톤 객체로 하나이다. 

### 구성 요소 조합 

CoroutineName, CoroutineDispatcher, Job, CoroutineExceptionHandler와 같이 CoroutineContext의 구성 요소들은 아래 코드처럼 모두 Element 인터페이스를 구현하고 있다. 

따라서, CoroutineContext에 정의된 plus() 함수로 여러 구성 요소를 조합하여 새로운 CoroutineContext를 생성할 수 있다. 

```kotlin 
/**
 * Base class for [CoroutineContext.Element] implementations.
 */
@SinceKotlin("1.3")
public abstract class AbstractCoroutineContextElement(public override val key: Key<*>) : Element
```

```kotlin 
public data class CoroutineName(
    val name: String
) : AbstractCoroutineContextElement(CoroutineName)
```

```kotlin
public abstract class CoroutineDispatcher :
    AbstractCoroutineContextElement(ContinuationInterceptor), ContinuationInterceptor
```

```kotlin 
public interface Job : CoroutineContext.Element
```

```kotlin 
public interface CoroutineExceptionHandler : CoroutineContext.Element
```

```kotlin 
public operator fun plus(context: CoroutineContext): CoroutineContext
```

앞서 CoroutineContext는 CoroutineName, CoroutineDispatcher, Job, CoroutineExceptionHandler 객체를 **각각 한 개씩만 가질 수 있다**고 했다. 

즉, **기존의 CoroutineContext 객체에 동일한 구성 요소가 추가되면, 이전의 구성 요소는 새로운 구성 요소에 의해 덮어씌워진다.** 

```kotlin 
import kotlinx.coroutines.*
import kotlin.coroutines.EmptyCoroutineContext

fun main() = runBlocking<Unit> {
    val coroutineContext = Dispatchers.IO + CoroutineName("OldCoroutine")
    launch(coroutineContext + CoroutineName("NewCoroutine") + EmptyCoroutineContext) {
        println(Thread.currentThread().name)
    }
}
```

위 코드의 실행 결과는 `DefaultDispatcher-worker-1 @NewCoroutine#2` 이다. CoroutineName 외의 다른 구성 요소에도 동일한 원리가 적용된다. 

### CoroutineContext에서 구성 요소 제거 

minusKey() 메서드를 이용해 CoroutineContext에서 특정 구성 요소를 제거할 수도 있다. 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val coroutineContext = CoroutineName("MyCoroutine") + Dispatchers.IO + Job()
    val deletedCoroutineContext = coroutineContext.minusKey(CoroutineName.Key)

    println(coroutineContext) // [CoroutineName(MyCoroutine), JobImpl{Active}@136432db, Dispatchers.IO]
    println(deletedCoroutineContext) // [JobImpl{Active}@136432db, Dispatchers.IO]
}
```

## CoroutineContext의 상속 

우리가 부모, 자식 코루틴이라고 불렀던 것도 한 영역 안에서 코루틴이 생기는 것을 의미하는데 그림과 함께 이해해보자. 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/663e6696-dca8-4f9e-9e75-992581f6ae94"/>

위의 그림처럼 최초의 한 영역에 부모 코루틴이 있다고 가정하자. 

이때 CoroutineContext에는 이름, Dispatchers.Default, 부모 코루틴 그 자체가 들어있다. 

이 상황에서 부모 코루틴에서 자식 코루틴을 만들면 다음과 같을 것이다. 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/65897b39-e35c-4cf9-8e43-fc9b3d3dd504"/>

자식 코루틴은 부모 코루틴과 같은 영역에서 생성되고, **부모 코루틴의 Context를 복사해 적절히 내용을 덮어씌운 새로운 Context를 만든다.** 이 과정에서 **부모, 자식 관계도 설정**해준다. 

이 원리가 바로 이전 시간에 살펴봤던 구조적 동시성을 작동시킬 수 있는 기반이 되는 것이다. 

여기서 한 가지 퀴즈! **자식 코루틴은 부모 코루틴의 CoroutineContext를 전부 상속 받을까?** 정답은..... No! 

자식 코루틴은 부모 코루틴으로부터 주요 구성요소인 CoroutineName, CoroutineDispatcher, CoroutineExceptionHandler를 상속 받는다. 그러나, **Job은 상속 받지 않는다.** 

```kotlin 
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> { // 부모 코루틴
    val parentJob = coroutineContext[Job]

    launch { // 자식 코루틴
        val childJob = coroutineContext[Job]
        println(parentJob === childJob) // false 
    }
}
```

Job 객체로 코루틴을 제어할 수 있는데, 이를 부모 코루틴으로부터 상속 받으면 **개별 코루틴의 제어가 어려워진다.** 그래서 launch, async 같은 코루틴 빌더 함수는 Job 객체를 새로 생성한다. 

## 구조적 동시성의 기반이 되는 Job 

부모 코루틴과 자식 코루틴은 각각 독립적인 Job 객체를 갖지만, 서로 아무런 관계가 없는 것은 아니다. **Job 객체는 코루틴을 구조화하는 데 사용**된다.

```kotlin 
public interface Job : CoroutineContext.Element {
    // ...
    @ExperimentalCoroutinesApi
    public val parent: Job?
    // ...
    public val children: Sequence<Job>
    // ...
}
```

부모 코루틴의 Job 객체는 children 프로퍼티를 통해 자식 코루틴의 Job 객체를 참조하고, 자식 코루틴 Job 객체는 parent 프로퍼티를 통해 부모 코루틴의 Job 객체를 참조한다. 

루트 코루틴은 부모가 없기 때문에 parent 프로퍼티는 `Job?` 타입이며, 부모 코루틴은 여러 개의 자식 코루틴을 가질 수 있으므로 `Sequence<Job>` 타입이다. 

# Dispatcher

마지막으로 Context에 들어갈 수 있는 Dispatcher에 대해 더 알아보자.

코루틴은 **스레드에 배정되어 실행**될 수 있으며, 중단되었다가 다른 스레드에 배정될 수도 있다. 

이렇게 **코루틴을 스레드에 배정하는 역할을 Dispatcher가 수행**한다. 

Dispatcher의 대표적인 종류는 다음과 같다. 

- `Dispatchers.Default`
    - 가장 기본적인 디스패처
    - 무거운 연산 작업 등 **CPU 자원을 많이 사용할 때 권장**되며, 별다른 설정이 없다면 기본으로 사용됨.
    - JVM의 공유 스레드 풀을 사용하며, 병렬 처리 가능한 최대 개수는 **CPU 코어 수**와 동일 (최소 2개)
- `Dispatchers.IO`
    - 파일, 네트워크, 데이터베이스 등 **입출력 작업에 최적화** 된 디스패처
    - **최대 64개까지 스레드를 생성**하여, 대기 시간이 있는 IO 작업을 효과적으로 병렬 처리 가능 
    - Default 디스패처와 동일한 스레드를 공유하므로, IO 디스패처로 전환해도 컨텍스트 스위칭 발생하지 않음.
- `Dispatchers.Main`
    - 보통 **UI 컴포넌트를 조작**하기 위해 사용되는 디스패처
    - 특정 의존성을 갖고 있어야 정상적으로 활용 가능 (ex. 안드로이드)
- `Dispatchers.Unconfined`
  - **특정 스레드에 국한되지 않는** 디스패처 (Unconfined: 제한 없음)
  - 자신을 호출한 스레드에서 실행되다가, 중단점 이후에는 해당 suspend 함수가 호출된 스레드에서 재개됨.
- `asCoroutineDispatcher()`
    - Java의 스레드 풀인 ExecutorService를 디스패처로 변환하는 확장함수

```kotlin
fun main() {
    val threadPool = Executors.newSingleThreadExecutor()

    CoroutineScope(threadPool.asCoroutineDispatcher()).launch { 
        printWithThread("새로운 코루틴")
    }
}
```

- 하나의 스레드를 갖는 스레드 풀을 만든다. (ExecutorService 타입)
- CoroutineScope로 새로운 영역을 만들 때, 스레드 풀을 디스패처로 변환하여 적용한 다음 launch 블록 안에서 새로운 코루틴을 만든다.
- 그러면 **해당 코루틴은 우리가 만든 스레드 풀에 배정하여 실행**시킬 수 있다.
- 이 방법을 이용하면 손쉽게 스레드 풀을 만들어서 **여러 코루틴을 해당 스레드 풀에서 돌릴 수 있게 된다.**

# 참고 자료 

- [2시간으로 끝내는 코루틴, 최태현](https://www.inflearn.com/course/2%EC%8B%9C%EA%B0%84%EC%9C%BC%EB%A1%9C-%EB%81%9D%EB%82%B4%EB%8A%94-%EC%BD%94%EB%A3%A8%ED%8B%B4)
- https://velog.io/@sdhong0609/CoroutineContext란
- https://velog.io/@sdhong0609/부모-코루틴과-자식-코루틴-상속되지-않는-Job
