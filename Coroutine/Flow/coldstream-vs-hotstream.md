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

Cold Stream은 **CD 플레이어**에 비유할 수 있다. 

<img width="250" src="https://github.com/user-attachments/assets/87aba469-ba1f-47a2-9818-10f10a18239f"/>

1. 각 CD 내부에 음악 목록이 저장되어 있다. 
2. 사용자가 재생 버튼을 눌러야 음악이 재생된다. 
3. CD 플레이어마다 주인이 있다. (하나의 CD 플레이어를 여러 사람이 공유하지 않는다고 가정함.)

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

Channel은 아래 코드처럼 capacity 매개변수가 **기본적으로 RENDEZVOUS로 설정**되어 있어서, **소비자가 없으면 send() 메서드가 suspend** 되어 다음 작업으로 넘어가지 않는다.

```kotlin 
public fun <E> Channel(
    capacity: Int = RENDEZVOUS,
    onBufferOverflow: BufferOverflow = BufferOverflow.SUSPEND,
    onUndeliveredElement: ((E) -> Unit)? = null
): Channel<E>
```

따라서, **구독자가 없어도 이벤트를 버퍼에 저장하고 바로 다음 작업으로 넘어가고 싶으면, capacity 옵션을 BUFFERED로 변경**해야 한다. 소비자의 소비와 무관하게 데이터를 생산할 수 있다는 점에서 Hot Stream이라고 볼 수 있다. 

참고로 capacity 옵션에 따른 send() 메서드의 suspend 여부를 정리하면 다음과 같다. 

- RENDEZVOUS: 수신자가 없으면 suspend 된다. (버퍼 사용 X)
- CONFLATED: 가장 최신 값만 유지되며, suspend 되지 않는다. 
- UNLIMITED: 버퍼 크기가 무제한이며, suspend 되지 않는다. (단, 메모리 관리에 주의 필요)
- BUFFERED: 버퍼 크기가 64이며, 버퍼 크기가 다 찼을 때만 suspend 된다. (단, BufferOverflow 옵션 변경 가능)

Channel 외의 다른 Hot Stream인 **StateFlow, SharedFlow는 수신자가 없어도 emit()이 suspend 되지 않는다.** StateFlow는 가장 최신 값 하나만 유지되며, SharedFlow는 수신자가 없으면 suspend 되지 않고 값이 유실된다. 

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

Hot Stream은 방송국 **라디오**에 비유할 수 있다. 

<img width="250" src="https://github.com/user-attachments/assets/4e162217-d5e1-4e17-aee7-fd1a494c8b83"/>

1. 라디오는 방송국에서 프로그램을 제작하여 
2. 다수의 청취자들에게 동시에 송신한다.
3. 뒤늦게 라디오를 듣기 시작한 청취자는 해당 시점 이후부터만 들을 수 있다. (청취자가 라디오의 시작 시점을 제어할 수 없음.)

# 차이점 요약 

|  | Cold Stream | Hot Stream |
| --- | --- | --- |
| 데이터를 생산하는 위치 | 내부 | 외부 |
| 데이터를 생산하는 시점 | 소비자가 소비를 시작할 때 생산 (Lazy Stream) | 소비자와 무관하게 생산 (Eager Stream)  |
| 생산자에 대응하는 소비자 개수 | 하나의 생산자에 하나의 소비자 존재 (Unicast, 각 소비자의 구독은 독립적) | 하나의 생산자에 다수의 소비자 존재 (Multicast, 하나의 데이터 스트림을 여러 소비자가 공유) |
| 사용 사례  | 서버에 데이터 요청, DB 쿼리 등  | - 모든 수신자를 최신 상태로 갱신 (StateFlow)<br> - 일회성 이벤트에 대한 처리 (Channel, SharedFlow)  |
| 예시 | Flow | Channel, StateFlow, SharedFlow |

📌 **일회성 이벤트를 처리할 때 Channel vs SharedFlow 중에 무엇을 사용해야 되는가?**

과거에 필자는 Channel은 구독자를 1명만 가질 수 있기 때문에, 구독자가 여러 명일 때는 SharedFlow를 사용해야 된다고 생각했다. 그런데, [이 글](https://medium.com/prnd/viewmodel%EC%97%90%EC%84%9C-%EB%8D%94%EC%9D%B4%EC%83%81-eventflow%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EB%A7%88%EC%84%B8%EC%9A%94-3974e8ddffed)을 통해 UI 이벤트를 처리할 때는 Channel을 사용하는 게 더 적합하다는 걸 알게 되었다. 

그 이유는 다음과 같다. 

1. Channel을 사용하면 **백그라운드 상태에서도 값을 유실하지 않고 버퍼에 저장**해둘 수 있다. 
2. 이벤트를 처리할 때는 하나의 ViewModel 인스턴스를 2개 이상의 View가 참조하지 않는다. 즉, **구독자가 1명만 존재**한다. (간혹 Fragment에서 activityViewModels()로 하나의 뷰모델을 공유할 때도 있지만, 이때는 해당 ViewModel의 함수나 StateFlow를 사용할 뿐 이벤트를 관찰 및 처리하는 로직을 넣지는 않는다.)

추가적으로 **일회성 이벤트를 처리할 때** 기본적으로 준수해야 되는 사항은 다음과 같다. 

1. 구독자가 없을 때 Event가 발생했어도, **나중에 다시 구독자가 생기면 해당 Event가 전달되어야 한다.** (ex. 홈버튼을 눌러 백그라운드에 있을 때 Event가 발생했어도 다시 화면에 진입했을 때 Event를 받아야 한다.) 
2. Event가 한번 처리(consume)되고 나면, **중복으로 전달되지 않아야 한다.** (ex. 화면이 다시 보여지거나, 기기가 회전했을 때 이벤트가 다시 전달되면 안 된다.)

1번 관점에서 SharedFlow는 구독자가 없을 때 값이 그대로 유실되기 때문에, UI 이벤트 처리에 부적합하다. (값을 캐싱하기 위한 별도 로직이 필요하다.) 

2번 관점에서 LiveData, StateFlow는 마지막 값이 계속 유지되기 때문에, UI 이벤트 처리에 부적합하다. 

# Cold flow → Hot flow 변환 

Cold flow는 앞서 설명한 대로, 각 소비자가 구독을 시작할 때마다 독립적으로 데이터가 생산된다. 

네트워크 연결을 맺고 서버로부터 데이터를 받아오는 것처럼, **flow 생성 자체에 비용이 많이 들 때는 하나의 flow만 생성하고 여러 소비자가 이를 공유하는 것이 훨씬 효율적**이다.

이처럼 Cold flow를 Hot flow로 변환하기 위해, **shareIn, stateIn 확장함수**를 이용할 수 있다.

## shareIn 

```kotlin 
fun <T> Flow<T>.shareIn(scope: CoroutineScope,
                        started: SharingStarted,
                        replay: Int = 0): SharedFlow<T>
```

## stateIn 

```kotlin 
fun <T> Flow<T>.stateIn(scope: CoroutineScope,
                        started: SharingStarted,
                        initialValue: T): StateFlow<T>
```

## SharingStarted

- `SharingStarted.Eagerly`
  - 구독자가 존재하지 않아도 upstream flow의 공유가 바로 시작되며, 중간에 중지되지 않는다. 
  - replay 개수만큼 이전에 방출된 데이터 캐싱 
  - BufferOverflow.DROP_OLDEST 기반으로 동작
- `SharingStarted.Lazily`
  - 첫번째 구독자가 등록되면 upstream flow의 공유가 시작되며, 중간에 중지되지 않는다.
  - 첫번째 구독자는 그동안 방출된 모든 값을, 이후의 구독자들은 replay에 설정된 개수만큼 얻어가며 collect를 시작한다. 
- `SharingStarted.WhileSubscribed`
  - 첫번째 구독자가 등록되면 upstream flow의 공유가 시작되며, 구독자가 전부 없어지면 바로 중지된다. 
  - `stopTimeoutMillis`
    - 구독자가 모두 없어진 후, flow 취소까지 걸리는 delay 시간 지정 
    - 화면 회전 시에는 취소하지 않고, 앱이 백그라운드로 전환되면 취소하도록 설정 가능 
  - `replayExpriationMilis`
    - replay를 위해 캐시해둔 데이터의 만료 시간 지정 
    - 기본값은 Long.MAX_VALUE여서 무한히 캐시되지만, 만료 시간 지정하여 replay 캐시 데이터 초기화 가능 
  
```kotlin 
fun WhileSubscribed(
    stopTimeoutMillis: Long = 0,
    replayExpirationMillis: Long = Long.MAX_VALUE
): SharingStarted
```

# 참고자료

- [Flow와 Channel, Cold Stream과 Hot Stream](https://medium.com/@apfhdznzl/flow와-channel-cold-stream과-hot-stream-c42c64cf4996)
- https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/
- https://tourspace.tistory.com/434
- https://github.com/mdb1217/TIL/blob/main/Kotlin/Coroutine/Hot%20Stream%20VS%20Cold%20Stream.md