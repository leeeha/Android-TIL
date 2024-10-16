안드로이드에서 이벤트를 처리할 때 주로 Channel 또는 SharedFlow를 사용한다. 각각의 장단점을 정리하여, 적재적소에 맞게 사용하도록 하자! 

```kotlin
private val _channel = Channel<T>()
val channel = _channel.receiveAsFlow()
```

```kotlin
private val _sharedFlow = MutableSharedFlow<T>() 
val sharedFlow: SharedFlow<T> = _sharedFlow.asSharedFlow()
```

# Channel 

## 장단점 

- 장점: 백그라운드에서 발생한 이벤트도 수집 가능 
- 단점: 여러 개의 구독자를 가지기에는 적합하지 않음. 

## 백그라운드에서 이벤트 발생하는 경우 

```kotlin 
class MainViewModel: ViewModel() {
    private val _channel = Channel<Int>()
    val channel = _channel.receiveAsFlow()

    init {
        viewModelScope.launch {
            repeat(30) {
                Timber.d("send $it")
                _channel.send(it)
                delay(1000)
            }
        }
    }
}
```

```kotlin 
class MainActivity : ComponentActivity() {
    private val viewModel by viewModels<MainViewModel>()

    override fun onStart() {
        super.onStart()
        Timber.d("onStart")
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.channel.collect {
                    Timber.d("collect $it")
                }
            }
        }
    }

    override fun onStop() {
        super.onStop()
        Timber.d("onStop")
    }
}
```

```
22:16:27.920 MainActivity             D  onStart
22:16:27.921 MainViewModel            D  send 0
22:16:27.922 MainActivity$onCreate    D  collect 0
22:16:28.923 MainViewModel            D  send 1
22:16:28.924 MainActivity$onCreate    D  collect 1
22:16:29.926 MainViewModel            D  send 2
22:16:29.926 MainActivity$onCreate    D  collect 2
22:16:30.929 MainViewModel            D  send 3
22:16:30.930 MainActivity$onCreate    D  collect 3
22:16:31.930 MainViewModel            D  send 4
22:16:31.931 MainActivity$onCreate    D  collect 4
22:16:32.292 MainActivity             D  onStop
22:16:32.933 MainViewModel            D  send 5 // suspend 
22:16:35.538 MainActivity             D  onStart
22:16:35.539 MainActivity$onCreate    D  collect 5 
22:16:36.541 MainViewModel            D  send 6
22:16:36.543 MainActivity$onCreate    D  collect 6
```

1. MainActivity - onStop (백그라운드 진입)
2. MainViewModel - channel send 5 (백그라운드에서 send)
3. MainActivity - onStart (포그라운드 진입)
4. MainActivity - collect 5 (포그라운드에서 collect)

channel의 send() 함수는 **버퍼가 가득차거나 존재하지 않으면, suspend 되는 방식으로 동작**한다. 따라서, **백그라운드에서 생산한 데이터가 유실되지 않고, 포그라운드로 재진입 했을 때 수집**되는 것을 확인할 수 있다.


## 여러 구독자를 가지는 경우

```kotlin 
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    lifecycleScope.launch {
        repeatOnLifecycle(Lifecycle.State.STARTED) {
            viewModel.channel.collect {
                Timber.d("collector #1 $it")
            }
        }
    }

    lifecycleScope.launch {
        repeatOnLifecycle(Lifecycle.State.STARTED) {
            viewModel.channel.collect {
                Timber.d("collector #2 $it")
            }
        }
    }
}
```

``` 
22:23:40.325 MainViewModel            D  send 14
22:23:40.325 MainActivity$onCreate    D  collector #2 14
22:23:41.330 MainViewModel            D  send 15
22:23:41.331 MainActivity$onCreate    D  collector #1 15
22:23:42.337 MainViewModel            D  send 16
22:23:42.337 MainActivity$onCreate    D  collector #2 16
22:23:43.341 MainViewModel            D  send 17
22:23:43.342 MainActivity$onCreate    D  collector #1 17
22:23:44.345 MainViewModel            D  send 18
22:23:44.346 MainActivity$onCreate    D  collector #2 18
22:23:45.350 MainViewModel            D  send 19
22:23:45.351 MainActivity$onCreate    D  collector #1 19
22:23:46.354 MainViewModel            D  send 20
22:23:46.356 MainActivity$onCreate    D  collector #2 20
```

**Channel은 평등**하기 때문에 구독자가 여러 개인 경우, **각 구독자는 번갈아가면서 데이터를 수집**하게 된다. 

모든 구독자가 동일한 데이터 스트림을 공유하게 만들려면, Channel 보다 SharedFlow가 더 적합하다. 

# SharedFlow 

## 장단점 

- 장점: 여러 개의 구독자를 가지기에 적합함. 
- 단점: 백그라운드에서 발생한 이벤트 수집 불가능 

## 백그라운드에서 이벤트 발생하는 경우 

```kotlin 
class MainViewModel: ViewModel() {
    private val _sharedFlow = MutableSharedFlow<Int>()
    val sharedFlow: SharedFlow<Int> = _sharedFlow.asSharedFlow()

    init {
        viewModelScope.launch {
            repeat(30) {
                Timber.d("emit $it")
                _sharedFlow.emit(it)
                delay(1000)
            }
        }
    }
}
```

```kotlin 
class MainActivity : ComponentActivity() {
    private val viewModel by viewModels<MainViewModel>()

    override fun onStart() {
        super.onStart()
        Timber.d("onStart")
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.sharedFlow.collect {
                    Timber.d("collect $it")
                }
            }
        }
    }

    override fun onStop() {
        super.onStop()
        Timber.d("onStop")
    }
}
```

```
22:28:54.694 MainViewModel            D  emit 4
22:28:54.695 MainActivity$onCreate    D  collect 4
22:28:55.108 MainActivity             D  onStop
22:28:55.703 MainViewModel            D  emit 5
22:28:56.706 MainViewModel            D  emit 6
22:28:57.709 MainViewModel            D  emit 7
22:28:58.712 MainViewModel            D  emit 8
22:28:59.183 MainActivity             D  onStart
22:28:59.713 MainViewModel            D  emit 9
22:28:59.713 MainActivity$onCreate    D  collect 9
22:29:00.716 MainViewModel            D  emit 10
22:29:00.716 MainActivity$onCreate    D  collect 10
22:29:01.719 MainViewModel            D  emit 11
22:29:01.720 MainActivity$onCreate    D  collect 11
```

1. MainActivity - onStop (백그라운드 진입)
2. MainViewModel - sharedFlow emit 5, 6, 7, 8 (백그라운드에서 emit)
3. MainActivity - onStart (포그라운드 진입)
4. MainActivity - collect 9 (포그라운드에서 collect) - 5, 6, 7, 8 유실 

Channel과 달리, **SharedFlow는 백그라운드에서 방출된 데이터는 무시한다**는 걸 확인할 수 있다. 

단, 아래 코드처럼 collect 함수를 repeatOnLifecycle(Lifecycle.State.STARTED) 블록으로 감싸지 않으면, 백그라운드에서도 데이터의 발행과 수집이 모두 일어난다. 

```kotlin 
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    lifecycleScope.launch {
        viewModel.sharedFlow.collect {
            Timber.d("collect $it")
        }
    }
}
```

```
22:32:52.020 MainViewModel            D  emit 4
22:32:52.020 MainActivity$onCreate    D  collect 4
22:32:52.092 MainActivity             D  onStop // 백그라운드 진입 
22:32:53.022 MainViewModel            D  emit 5
22:32:53.023 MainActivity$onCreate    D  collect 5
22:32:54.030 MainViewModel            D  emit 6
22:32:54.031 MainActivity$onCreate    D  collect 6
22:32:55.039 MainViewModel            D  emit 7
22:32:55.039 MainActivity$onCreate    D  collect 7
22:32:56.042 MainViewModel            D  emit 8
22:32:56.043 MainActivity$onCreate    D  collect 8
22:32:56.128 MainActivity             D  onStart // 포그라운드 진입 
22:32:57.045 MainViewModel            D  emit 9
22:32:57.045 MainActivity$onCreate    D  collect 9
22:32:58.049 MainViewModel            D  emit 10
22:32:58.049 MainActivity$onCreate    D  collect 10
```

## 여러 구독자를 가지는 경우 

```kotlin 
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    lifecycleScope.launch {
        repeatOnLifecycle(Lifecycle.State.STARTED) {
            viewModel.sharedFlow.collect {
                Timber.d("collector #1 $it")
            }
        }
    }

    lifecycleScope.launch {
        repeatOnLifecycle(Lifecycle.State.STARTED) {
            viewModel.sharedFlow.collect {
                Timber.d("collector #2 $it")
            }
        }
    }
}
```

```
22:36:50.862 MainActivity             D  onStop
22:36:50.880 MainViewModel            D  emit 2
22:36:51.881 MainViewModel            D  emit 3
22:36:52.885 MainViewModel            D  emit 4
22:36:53.890 MainViewModel            D  emit 5
22:36:54.279 MainActivity             D  onStart
22:36:54.892 MainViewModel            D  emit 6
22:36:54.892 MainActivity$onCreate    D  collector #1 6
22:36:54.893 MainActivity$onCreate    D  collector #2 6
22:36:55.895 MainViewModel            D  emit 7
22:36:55.895 MainActivity$onCreate    D  collector #1 7
22:36:55.895 MainActivity$onCreate    D  collector #2 7
22:36:56.899 MainViewModel            D  emit 8
22:36:56.900 MainActivity$onCreate    D  collector #1 8
22:36:56.900 MainActivity$onCreate    D  collector #2 8
```

Channel과 달리, **SharedFlow에서는 모든 구독자가 동일한 데이터 스트림을 공유한다**는 걸 확인할 수 있다. 

## 단점 극복 방법 

https://jinukeu.hashnode.dev/android-channel-vs-sharedflow#heading-6rkw66gg
