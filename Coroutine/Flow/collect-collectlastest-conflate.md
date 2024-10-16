# collect 

```kotlin 
public suspend inline fun <T> Flow<T>.collect(crossinline action: suspend (value: T) -> Unit): Unit =
    collect(object : FlowCollector<T> {
        override suspend fun emit(value: T) = action(value)
    })
```

collect 함수는 **flow에서 발행된 데이터를 순차적으로 받아서 suspend 함수를 수행**한다. 

이 방법의 문제점은, **데이터의 발행 속도에 비해 소비 속도가 느린 경우 지연이 발생**할 수 있다는 것이다. 

```kotlin
fun main(): Unit = runBlocking {
    val flow = flow {
        repeat(5) {
            emit(it)
            delay(100)
        }
    }

    launch {
        flow.collect {
            println("collect $it")
            delay(1000)
        }
    }
}
```

>collect 0<br>
collect 1<br>
collect 2<br>
collect 3<br>
collect 4<br>

0.1초씩 총 5개의 데이터를 발행하여 0.5초만에 발행이 끝났다. 하지만, 발행된 데이터를 소비할 때는 하나당 1초씩 걸려서 총 5초가 걸린다.

이러한 지연 문제를 해결하는 간단한 방법은, **최신 데이터가 발행되면 이전에 수행 중이던 suspend 함수를 취소하고 새 데이터로 suspend 함수를 수행**하는 것이다. 

<img width="650" src="https://github.com/user-attachments/assets/6a2ecddf-c5fc-44c0-9db1-0d7b02e872ff"/>

이것을 구현한 함수가 바로 collectLatest이다. 

# collectLatest 

```kotlin
fun main(): Unit = runBlocking {
    val flow = flow {
        repeat(5) {
            emit(it)
            delay(100)
        }
    }

    launch {
        flow.collectLatest {
            println("collectLatest $it")
            delay(1000)
        }
    }
}
```

>collectLatest 0<br>
collectLatest 1<br>
collectLatest 2<br>
collectLatest 3<br>
collectLatest 4<br>

새 데이터가 발행될 때마다 수신자의 delay 함수가 취소되어서, 발행과 마찬가지로 소비도 0.5초만에 끝난다. 

그런데, 실제 사용 사례에서는 delay 시간이 걸린 뒤에 데이터가 소비되는 것이 더 일반적이다. 그래서 다음과 같이 delay 함수의 실행 위치를 옮겼다. 

```kotlin
fun main(): Unit = runBlocking {
    val flow = flow {
        repeat(5) {
            emit(it)
            delay(100)
        }
    }

    launch {
        flow.collectLatest {
            delay(1000) // 데이터 소비보다 먼저 실행 
            println("collectLatest $it")
        }
    }
}
```

>collectLatest 4

0부터 3까지는 소비되기 전에 suspend 함수가 취소되고, 마지막 4만 소비된다. 

<img width="650" src="https://github.com/user-attachments/assets/dd3ef96e-ba51-48a2-8db3-2b9e322cfa62"/>

이처럼 **데이터의 발행 속도에 비해 소비 속도가 느릴 때** collectLatest 함수를 사용하면, **중간 데이터를 하나도 얻지 못하고 마지막 데이터만 얻는 문제**가 생길 수 있다. 

이 문제를 해결하기 위해 conflate 함수를 사용할 수 있다. 

# conflate 

conflate 함수는 **한 번 데이터를 소비하면 끝까지 소비하고, 데이터 소비가 끝난 시점에서의 가장 최신 데이터를 다시 소비**한다. 

<img width="650" src="https://github.com/user-attachments/assets/8dd68633-7e38-430b-b97a-01d153fd9f2c"/>

```kotlin 
fun main(): Unit = runBlocking {
    val flow = flow {
        repeat(10) {
            emit(it)
            delay(3000)
        }
    }

    launch {
        flow.onEach {
            println("emit: $it")
        }.conflate().collect {
            delay(5000)
            println("conflated flow: $it")
        }
    }
}
```

>emit: 0<br>
emit: 1<br>
conflated flow: 0<br>
emit: 2<br>
emit: 3<br>
conflated flow: 1<br>
emit: 4<br>
conflated flow: 3<br>
emit: 5<br>
emit: 6<br>
conflated flow: 4<br>
emit: 7<br>
emit: 8<br>
conflated flow: 6<br>
emit: 9<br>
conflated flow: 8<br>
conflated flow: 9<br>

구체적인 동작 과정은 다음과 같다.

1. 0이 발행되고 나서, 0에 대한 소비가 5초 후에 끝난다. 
2. 5초 시점에서 가장 최근에 발행된 데이터인 1이 즉시 소비된다. 
3. 1에 대한 소비는 시작 시점으로부터 10초 후에 끝난다.
4. 10초 시점에서 가장 최근에 발행된 데이터인 3이 즉시 소비된다. 

| 시간(초) | 발행 (3초 간격) | 소비 (5초 간격) |
| :---: | :---: | :---: |
| 0 | 0 | 0 |
| 3 | 1 | - |
| 5 | - | 1 |
| 6 | 2 | - |
| 9 | 3 | - |
| 10 | - | 3 |
| 12 | 4 | - |
| 15 | 5 | 4 |
| 18 | 6 | - |
| 20 | - | 6 |
| 21 | 7 | - |
| 24  | 8 | - |
| 25 | - | 8 |
| 27 | 9 | - |
| 30 | 10 | 9 |

이 방법을 통해 데이터의 발생 속도보다 소비 속도가 느릴 때도, 최근에 발행된 데이터를 계속해서 소비할 수 있다. 

# 참고 자료 

- https://kotlinworld.com/252?category=973477
- https://kotlinworld.com/254?category=973477