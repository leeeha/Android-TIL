# Data Stream

**반응형 프로그래밍** (Reactive Programming)이란 **데이터가 변경될 때 이벤트를 발생시켜 데이터를 계속해서 전달하도록 하는 프로그래밍 방식**을 의미한다. 

기존 명령형 프로그래밍에서는 소비자가 데이터를 요청한 후 받은 결과값을 일회성으로 수신한다. 즉, 데이터가 필요할 때마다 매번 반복적으로 요청해야 하므로 비효율적이다. 

반응형 프로그래밍에서는 데이터를 발행하는 ‘생산자’가 있고, 해당 데이터를 구독하는 ‘소비자’가 있다. 그리고 **생산자가 데이터를 발행할 때마다 이를 구독하고 있던 소비자는 지속적으로 데이터를 수신**할 수 있다. 이러한 **생산자, 소비자 간의 데이터 흐름을 데이터 스트림**이라고 부른다. 

# Cold Stream vs. Hot Stream

데이터 스트림에는 두 가지 종류가 있는데, 바로 콜드 스트림과 핫 스트림이다. 이 둘은 다음과 같은 점에서 특징을 구분지을 수 있다.

>1. 데이터가 생성되는 위치
>2. 스트림이 데이터를 생산하는 시점
>3. 생산자가 발행한 데이터를 동시에 여러 소비자들이 수신할 수 있는지 여부

# Cold Stream

### 데이터가 내부에서 생성된다.

```kotlin
val coldStream: Flow<Int> = flow {
    // flow 내부에서 데이터 생성
    emit(1)
    emit(2)
}

/** or */

val coldStream: Flow<Int> = flowOf(1, 2)
public fun <T> flowOf(vararg elements: T): Flow<T> = flow {
    for (element in elements) {
        // flow 내부에서 데이터 생성
        emit(element)
    }
}

/** or */

val data = listOf(1, 2)
val coldStream: Flow<Int> = data.asFlow()
public fun <T> Iterable<T>.asFlow(): Flow<T> = flow {
    forEach { value ->
        // flow 내부에서 데이터 생성
        emit(value)
    }
}
```

flow 블록, flowOf(), asFlow()와 같은 flow 빌더로 생성한 **flow 내부에서 데이터가 방출된다**는 걸 확인할 수 있다. 

### 소비자가 소비를 시작할 때 데이터를 생산한다.

```kotlin
val coldStream = flow<Int> {  
    emit(1)
    emit(2)
}
    
// 이 시점에 Cold Stream의 데이터가 생산된다.
coldStream.collect {
    println(it)
}
```

flow는 **소비를 시작하는 함수인 종단 연산자** (collect, fold, reduce, first 등)**가 호출되어야 데이터를 생산한다.** 

물론 중간 연산자 (map, onEach, filter 등)도 종단 연산자가 호출되어야 실행된다. 

### 하나의 생산자에는 하나의 소비자가 존재한다. (유니캐스트)

```kotlin
val coldStream = flow<Int> {  
    emit(1)
    emit(2)
}
    
coldStream.collect {
    println(it) // 1 2 수신
}

coldStream.collect {
    println(it) // 1 2 수신
}
```

flow를 여러 곳에서 collect 할 수 있다. 하지만 **collect 할 때마다 flow 블록이 새롭게 실행**되며, 이것은 **이전 구독과는 독립적**이다. 

## Cold Stream의 특징

1. 데이터가 내부에서 생성된다.
2. 소비자가 소비를 시작할 때 데이터가 생산된다. 
3. 하나의 생산자에는 하나의 소비자가 존재한다. (유니캐스트) 

위의 3가지를 모두 충족해야 Cold Stream이라고 볼 수 있으며, Flow는 이에 해당한다. 

# Hot Stream

### 데이터가 외부에서 생성된다.

```kotlin
CoroutineScope(Dispatchers.Default).launch {
    val channel = Channel<Int>()

    launch {
        // 외부에서 데이터 생성
        channel.send(1)
    }

    launch {
        // 외부에서 데이터 생성
        channel.send(2)
        channel.close()
    }

    channel.consumeEach {
        println(it)
    }
}
```

위의 코드를 보면 channel의 send 메서드를 통해 데이터를 생성하는 걸 볼 수 있다. flow와 다르게 내부에서 미리 데이터를 생성한 게 아니라, 외부에서 데이터를 생성한다는 걸 알 수 있다. 

### 생산자가 소비자의 소비를 신경쓰지 않고 데이터를 생산한다.

```kotlin
val channel = Channel<Int>(2)
channel.trySend(1)
println(channel.isEmpty) // false
```

channel의 send 메서드로 데이터를 생성하면, 해당 데이터는 소비되기 전까지 내부적으로 큐에 보관된다. 그래서 위의 코드에서 `channel.isEmpty`가 false를 반환하는 것이다.

이렇게 큐에 데이터가 아직 있는 상태에서도 생산자는 새로운 데이터를 생성할 수 있다. 즉, 소비자의 소비와 무관하게 생산자는 데이터를 생산할 수 있다. 

### 하나의 생산자에 다수의 소비자가 존재한다. (멀티캐스트)

```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
fun CoroutineScope.produceNumbers() = produce {
    println(coroutineContext)
    var count = 0
    while (true) {
        send(count++)
        delay(100)
    }
}

fun CoroutineScope.consumeNumbers(index: Int, receiveChannel: ReceiveChannel<Int>) = launch {
    receiveChannel.consumeEach {
        println("$index 가 ${it}을 수신했습니다.")
    }
}

fun main(): Unit = runBlocking {
    val receiveChannel = produceNumbers()

    val job = launch {
        repeat(5) { index ->
            // 5개의 소비자 생성
            consumeNumbers(index, receiveChannel)
        }
    }

    delay(1000)
    job.cancel()
}
```

```kotlin
fun CoroutineScope.produceNumbers() = produce {
    println(coroutineContext)
    var count = 0
    while (true) {
        send(count++)
        delay(100)
    }
}
```

CoroutineScope의 확장함수인 produce는 채널에 0.1초 간격으로 데이터를 send 하고 있다. 

```kotlin
val job = launch {
        repeat(5) { index ->
            // 5개의 소비자 생성
            consumeNumbers(index, receiveChannel)
        }
    }
```

launch 코루틴 빌더 내에서 5개의 코루틴을 생성하고, 각 코루틴에 존재하는 소비자는 consumeEach를 통해 채널에 전달된 데이터를 소비한다.

실행 결과는 다음과 같다. 

```
0 가 0을 수신했습니다.
0 가 1을 수신했습니다.
1 가 2을 수신했습니다.
2 가 3을 수신했습니다.
3 가 4을 수신했습니다.
4 가 5을 수신했습니다.
0 가 6을 수신했습니다.
1 가 7을 수신했습니다.
2 가 8을 수신했습니다.
3 가 9을 수신했습니다.
```

이를 통해 알 수 있는 것은 하나의 데이터 stream을 각 코루틴에 존재하는 여러 개의 수신자가 함께 소비하고 있다는 것이다.

## Hot Stream의 특징

1. 데이터가 외부에서 생성된다. 
2. 생산자가 소비자의 소비를 신경쓰지 않고 데이터를 생산한다. 
3. 하나의 생산자에 다수의 소비자가 존재한다. (멀티캐스트) 

위의 3가지를 모두 충족해야 Hot Stream이라고 볼 수 있으며, Channel은 이에 해당한다. 

# 차이점 요약 

|  | Cold Stream | Hot Stream |
| --- | --- | --- |
| 데이터를 생산하는 위치 | 내부 | 외부 |
| 데이터를 생산하는 시점 | 소비자가 소비를 시작할 때 생산 <br> (Lazy Stream) | 소비자와 무관하게 생산 <br> (Eager Stream)  |
| 생산자에 대응하는 소비자 개수 | 하나의 생산자에 하나의 소비자 존재  <br> (Unicast) | 하나의 생산자에 다수의 소비자 존재 <br> (Multicast) |
| 사용 사례  | 서버에 데이터 요청, DB 쿼리 등  | - 모든 수신자를 최신 상태로 갱신 <br> - 키보드, 클릭 이벤트 등  |
| 예시 | Flow | Channel, StateFlow, SharedFlow |

# 참고자료

- [Flow와 Channel, Cold Stream과 Hot Stream](https://medium.com/@apfhdznzl/flow와-channel-cold-stream과-hot-stream-c42c64cf4996)
- https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/
- https://github.com/mdb1217/TIL/blob/main/Kotlin/Coroutine/Hot%20Stream%20VS%20Cold%20Stream.md