SharedFlow와 StateFlow는 모두 **비동기 데이터 스트림**을 처리하도록 특별히 설계된 Kotlin의 **kotlinx.coroutines 라이브러리의 일부**이다. 

둘 다 **Flow를 기반으로 구축**되었으며 서로 다른 용도로 사용된다. 

# SharedFlow

- SharedFlow는 **여러 수신자를 가질 수 있는 Hot stream**이다.
- 수신자와 독립적으로 값을 내보낼 수 있으며, 여러 수신자가 Flow로부터 동일한 값을 수집할 수 있다.
- **여러 수신자에 값을 브로드캐스트** 해야 하거나 **동일한 데이터 스트림에 대해 여러 명의 구독자를 보유**하려는 경우에 유용하다.
- **초기값이 없으며**, 새로운 수신자에 대해 이전에 방출된 값을 저장하도록 **replay 캐시**를 구성할 수 있다.

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/1b257a68-66b9-47a8-97b0-d09d68d60288"/>

```kotlin
package org.example

import kotlinx.coroutines.*
import kotlinx.coroutines.flow.MutableSharedFlow

fun main(): Unit = runBlocking {
    val sharedFlow = MutableSharedFlow<Int>()

    // Collect values from sharedFlow
    launch {
        sharedFlow.collect { value ->
            println("Collector 1 received: $value")
        }
    }

    // Collect values from sharedFlow
    launch {
        sharedFlow.collect { value ->
            println("Collector 2 received: $value")
        }
    }

    // Emit values to sharedFlow
    launch {
        repeat(3) { i ->
            sharedFlow.emit(i)
        }
    }
}
```

```
Collector 1 received: 0
Collector 2 received: 0
Collector 1 received: 1
Collector 2 received: 1
Collector 1 received: 2
Collector 2 received: 2
```

# StateFlow

- StateFlow는 **상태를 나타내는 Hot Stream**으로, **한 번에 하나의 값을 보유**한다.
- 새로운 값이 방출될 때 **가장 최근 값이 유지**되고, 즉시 새로운 수신자로 방출되는 conflated(융합된) flow이기도 하다.
- 상태에 대한 하나의 원천을 유지하고 **모든 수신자를 최신 상태로 자동 업데이트** 해야 할 때 유용하다.
- 항상 **초기값**을 가지며, **가장 최근에 방출된 값만 저장**한다.

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/08b2a1d9-8370-47f9-b1b4-ff190b9da9e6"/>

```kotlin
package org.example

import kotlinx.coroutines.*
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow

fun main(): Unit = runBlocking {
    val mutableStateFlow = MutableStateFlow(0)
    val stateFlow: StateFlow<Int> = mutableStateFlow.asStateFlow()

    // Collect values from stateFlow
    launch {
        stateFlow.collect { value ->
            println("Collector 1 received: $value")
        }
    }

    // Collect values from stateFlow
    launch {
        stateFlow.collect { value ->
            println("Collector 2 received: $value")
        }
    }

    // Update the state
    launch {
        repeat(3) { i ->
            mutableStateFlow.value = i
        }
    }
}
```

```
Collector 1 received: 0
Collector 2 received: 0
Collector 1 received: 2
Collector 2 received: 2
```

# 인터페이스 계층 구조

**Flow ← SharedFlow ← StateFlow**

**SharedFlow**

- replayCache 크기를 정의하여 **새로운 구독자에게 반복적으로 전달할 값의 개수를 설정**할 수 있다.
- 구독자들을 Slot이라는 형태로 관리하여 값이 전달될 시 **활성화 된 모든 구독자에게 새로운 값이 전달**되도록 한다.

**StateFlow**

- SharedFlow를 상속하는 StateFlow는 추가로 **기본값을 가지고 있으며**
- replaceCache는 **가장 최근의 값 하나**를 갖는 리스트로 재정의 하고 있다.

# Best Practices

### 올바른 flow 선택

- 여러 수집기에 값을 브로드캐스트해야 하거나 동일한 데이터 스트림에 여러 명의 구독자를 보유하려는 경우 SharedFlow 사용
- 상태에 대한 하나의 원천을 유지 및 공유하고 모든 수신자를 최신 상태로 자동 업데이트 해야 하는 경우 StateFlow 사용

### mutable flow 캡슐화

외부 변경을 방지하기 위해 변경 가능한 flow에 대한 읽기 전용 버전을 노출하자. 

이를 위해 MutableSharedFlow의 경우 SharedFlow 인터페이스를, MutableStateFlow의 경우 StateFlow 인터페이스를 사용하면 된다.

```kotlin
class ExampleViewModel {
    private val _mutableSharedFlow = MutableSharedFlow<Int>()
    // Represents this mutable shared flow as a read-only shared flow.
    val sharedFlow = _mutableSharedFlow.asSharedFlow()

    private val _mutableStateFlow = MutableStateFlow(0)
    // Represents this mutable state flow as a read-only state flow.
    val stateFlow = _mutableStateFlow.asStateFlow()
}
```

### 적절한 리소스 관리

SharedFlow, StateFlow를 사용할 때는 더 이상 필요하지 않은 코루틴이나 수집기를 취소하여 리소스를 적절히 관리해야 한다. 

```kotlin
val scope = CoroutineScope(Dispatchers.Main)

val sharedFlow = MutableSharedFlow<Int>()

val job = scope.launch {
    sharedFlow.collect { value ->
        println("Received: $value")
    }
}

// Later, when the collector is no longer needed
job.cancel()
```

### buffer, replay 용량 설정

SharedFlow의 경우 buffer 용량과 replay 용량을 설정할 수 있다. [backpressure](https://doublem.org/stream-backpressure-basic/) 문제를 방지하기 위해 적절한 버퍼 용량을 선택하고, 사용 사례의 요구사항에 따라 replay 용량을 설정하자. 

```kotlin
val sharedFlow = MutableSharedFlow<Int>(
    replay = 2, 
    extraBufferCapacity = 8 
    onBufferOverflow = BufferOverflow.DROP_OLDEST 
)
```

- replay : 새로운 구독자에게 반복적으로 전달할 이벤트의 개수
- extraBufferCapacity : 방출된 데이터를 저장해둘 추가 버퍼의 용량 (배압 문제 방지)
- onBufferOverflow : 버퍼가 가득 찼을 때의 동작 정의

### combine, map, filter 등의 연산자 사용

Kotlin flow 연산자를 활용하여 필요에 따라 데이터를 변형(transform), 조합(combine), 필터링(filter) 해보자. 이를 통해 보다 표현력이 풍부하고 효율적인 코드를 작성할 수 있다.

```kotlin
val flow1 = MutableStateFlow(1)
val flow2 = MutableStateFlow(2)

val combinedFlow = flow1.combine(flow2) { value1, value2 ->
    value1 + value2
}

// Collect and print the sum of flow1 and flow2
launch {
    combinedFlow.collect { sum ->
        println("Sum: $sum")
    }
}
```

### 적절한 예외 처리

Flow를 사용할 때는 예외를 올바르게 처리해야 한다. 

Flow 파이프라인 내에서 예외를 처리하려면 `catch` 연산자를 사용하고, cleanup 작업을 수행하거나 flow completion에 반응하려면 `onCompletion` 연산자를 사용하자. 

```kotlin
val flow = flow {
    emit(1)
    throw RuntimeException("Error occurred")
    emit(2)
}.catch { e ->
    // Handle the exception and emit a default value
    emit(-1)
}

launch {
    flow.collect { value ->
        println("Received: $value")
    }
}
```

### lifecycle-aware 연산자 사용

`lifecycleScope`, `repeatOnLifecycle` 같은 생명주기 인식 연산자들을 사용하여 생명주기를 자동으로 관리하자. 

```kotlin
class MyFragment : Fragment() {
    private val viewModel: MyViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        viewLifecycleOwner.lifecycleScope.launch {
            // repeatOnLifecycle 블록은 프래그먼트 뷰의 생명주기에 따라 
            // STARTED 이후에 시작되고 STOPPED 이후에는 취소된다. 
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.stateFlow.collect { value ->
                    // Update UI with the value
                }
            }
            // Note: at this point, the lifecycle is DESTROYED!
        }
    }
}
```

# Examples

## StateFlow Use Case: 실시간 데이터

주가, 날씨 정보 또는 채팅 메시지와 같은 **실시간 데이터**를 표시하는 애플리케이션이 있다고 가정해보자. 

StateFlow를 사용하면 **최신 데이터를 유지하고 UI를 자동으로 업데이트** 할 수 있다. 

```kotlin
class StockViewModel {
    // Create a private MutableStateFlow property to hold the stock price
    private val _stockPrice = MutableStateFlow(0.0)
    // Create a public StateFlow property that exposes the stock price as an immutable value
    val stockPrice = _stockPrice.asStateFlow()

    init {
        // Launch a coroutine in the viewModelScope to update the stock price
        viewModelScope.launch {
            updateStockPrice()
        }
    }

    private suspend fun updateStockPrice() {
        while (true) {
            delay(1000) // Update every second
            val newPrice = fetchNewPrice()
            // Update the stock price using the MutableStateFlow's value property
            _stockPrice.value = newPrice
        }
    }

    // Private function to fetch the new stock price from an API or data source
    private suspend fun fetchNewPrice(): Double {
        // TODO: Fetch the new stock price from an API or data source
        return 0.0
    }
}
```

```kotlin
class MainActivity : AppCompatActivity() {
    private val stockViewModel = StockViewModel()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // 주가의 변경을 관찰하기 위해 lifecycleScope에서 코루틴 실행 
        lifecycleScope.launch {
            // 액티비티가 STARTED 상태일 때만 코루틴 활성화 되도록 repeatOnLifecycle 함수 사용 
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                // collect 연산자로 주가의 변경사항 관찰 
                stockViewModel.stockPrice.collect { price ->
                    // 새로운 주가로 뷰 업데이트 
                    stockPriceTextView.text = "Stock Price: $price"
                }
            }
        }
    }
}
```

## SharedFlow Use Case: Event Bus

SharedFlow를 사용하여 **여러 수신자에게 이벤트를 브로드캐스트** 하는 이벤트 버스를 만들고 싶다고 가정해보겠다. 

### EventBus 

```kotlin
/**
 * An event bus implementation that uses a shared flow to 
 * broadcast events to multiple listeners.
 */
class EventBus<T> {
    // Create a private MutableSharedFlow property to hold events
    private val _events = MutableSharedFlow<T>(replay = 0, extraBufferCapacity = 64)
    // Create a public SharedFlow property that exposes events as an immutable value    
    val events: Flow<T> = _events.asSharedFlow()

    /**
     * Sends an event to the shared flow.
     */
    suspend fun sendEvent(event: T) {
        _events.emit(event)
    }
}
```

### Event 

```kotlin
sealed class Event {
    object EventA : Event()
    object EventB : Event()
    data class EventC(val value: Int) : Event()
}
```

### EventListener

```kotlin
class EventListener(
    private val eventBus: EventBus<Event>,
    private val scope: CoroutineScope
) {
    init {
        // onEach 연산자로 이벤트 flow 구독
        eventBus.events
            .onEach { event ->
                // 다른 타입의 이벤트 구분하기 위해 when 사용 
                when (event) {
                    is Event.EventA -> handleEventA()
                    is Event.EventB -> handleEventB()
                    is Event.EventC -> handleEventC(event.value)
                }
            }
            // 주어진 코루틴 스코프 내에서 이벤트 리스너 실행 
            // 스코프가 더 이상 존재하지 않으면 구독 취소 가능 
            .launchIn(scope)
    }

    // Private functions to handle specific types of events
    private fun handleEventA() {
        println("EventA received")
    }

    private fun handleEventB() {
        println("EventB received")
    }

    private fun handleEventC(value: Int) {
        println("EventC received with value: $value")
    }
}
```

### Main 

```kotlin
fun main() = runBlocking {
    val eventBus = EventBus<Event>()
    // eventBus로부터 이벤트를 수신하기 위한 이벤트 리스너 초기화 
    val eventListener = EventListener(eventBus, this)

    // 별개의 코루틴에서 eventBus를 통해 이벤트 전달 
    launch(Dispatchers.Default) {
        delay(1000)
        eventBus.sendEvent(Event.EventA)

        delay(1000)
        eventBus.sendEvent(Event.EventB)

        delay(1000)
        eventBus.sendEvent(Event.EventC(42))
    }

    // 리스너가 이벤트 처리할 수 있도록 메인 함수의 코루틴 스코프 유지 
    delay(5000)
}
```

# 참고자료

- [https://myungpyo.medium.com/stateflow-와-sharedflow-32fdb49f9a32](https://myungpyo.medium.com/stateflow-%EC%99%80-sharedflow-32fdb49f9a32)
- https://medium.com/@mortitech/sharedflow-vs-stateflow-a-comprehensive-guide-to-kotlin-flows-503576b4de31
- [https://medium.com/prnd/mvvm의-viewmodel에서-이벤트를-처리하는-방법-6가지-31bb183a88ce](https://medium.com/prnd/mvvm%EC%9D%98-viewmodel%EC%97%90%EC%84%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8%EB%A5%BC-%EC%B2%98%EB%A6%AC%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95-6%EA%B0%80%EC%A7%80-31bb183a88ce)
- https://www.youtube.com/watch?v=6Jc6-INantQ&t=369s