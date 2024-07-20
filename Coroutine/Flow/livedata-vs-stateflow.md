# LiveData 특징 

LiveData는 [옵저버 패턴](https://github.com/leeeha/Android-TIL/blob/main/DesignPattern/observer-pattern.md)을 활용해 구현되었으며, 관찰 가능한 일반 클래스인 ObservableXXX 클래스와 달리 **안드로이드 앱 컴포넌트의 생명주기 변화를 인식**한다. 

즉, Activity, Fragment, Service 등 안드로이드 앱 컴포넌트의 생명주기 인식으로 **Active 상태에 있는 컴포넌트에 대해서만 UI 및 데이터를 갱신**한다. 

LiveData 사용 시 얻을 수 있는 이점은 다음과 같다. 

- Activity, Fragment, Service 등 안드로이드 앱 컴포넌트의 생명주기가 끝나는 즉시 관찰을 멈추기 때문에 메모리 누수를 걱정하지 않아도 된다.
- LiveData는 옵저버 패턴에 따라 관찰 대상의 상태가 변하면 옵저버에게 알리고, 옵저버가 UI를 업데이트 하기 때문에 개발자가 직접 조정하지 않아도 된다.
- Activity가 백 스택에 있을 때 등 컴포넌트의 생명주기가 비활성 상태에 있으면, 옵저버는 어떤 LiveData 이벤트도 수신하지 않기 때문에 비정상 종료가 발생하지 않는다. 
- Inactive 상태에서 Active 상태로 변하면 다시 최신 데이터를 수신할 수 있다.
- Room, Retrofit 같은 라이브러리와의 호환성이 좋다.

이런 LiveData는 주로 AAC DataBinding, ViewModel과 함께 사용된다.

# 클린 아키텍처 관점에서 LiveData의 한계

그렇다면, 이번에는 [클린 아키텍처 관점](https://github.com/leeeha/Android-TIL/blob/main/Architecture/clean-architecture.md)에서 LiveData에 대해 다시 생각해보자.

- LiveData는 UI에 밀접하게 연관되어 있으므로 Data Layer에서 비동기 방식으로 데이터를 처리할 때는 사용할 수 없다.
- LiveData는 안드로이드 플랫폼에 종속적이기 때문에, 순수 자바, 코틀린 언어로만 작성해야 하는 Domain Layer에서 사용하기에 적합하지 않다. 

그런데, 코틀린 코루틴이 점차 발전하며 Flow가 등장했고, LiveData를 Flow로 대체할 수 있을지에 대한 기대감이 생기기 시작했다. 

그런데, LiveData를 Flow로 대체하는 것은 쉬운 일이 아니었다.

- Flow는 안드로이드 앱 컴포넌트의 생명주기를 인식하지 못한다. 따라서, 생명주기에 따른 중지 및 재개가 어렵다. 
- Flow는 상태가 없어서 현재 값이 무엇인지 알기 어렵다. 
- Flow는 Cold Stream 방식으로, 연속해서 들어오는 데이터를 처리하지 못하며 collect 되었을 때만 생성되고 값을 반환한다. 
- 하나의 생산자에 여러 소비자가 있으면, 수신자마다 개별적으로 생산자를 호출한다. 따라서, 업스트림 로직이 비싼 비용을 요구하는 DB 접근이나 서버 통신 등이라면 여러 번 리소스를 요청하게 될 수 있다. 

이를 위해 kotlin 1.41 버전에서 Stable API로 등장한 것이 바로 StateFlow, SharedFlow이다! 

# StateFlow 

StateFlow는 현재 상태와 새로운 상태 업데이트를 수신자에게 보내는 관찰 가능한 State holder Flow이다. LiveData와 마찬가지로 value 프로퍼티로 현재 상태의 값을 읽을 수 있다. 

StateFlow는 SharedFlow의 한 종류이며, LiveData에 가장 가깝다. 

- StateFlow는 항상 하나의 값을 가지고 있다.
- StateFlow는 하나의 생산자에 여러 개의 수신자가 대응될 수 있다. 즉, 생산자의 데이터를 여러 개의 수신자가 공유할 수 있다는 뜻이며, 업스트림이 수신자마다 개별적으로 처리되지 않는다.
- StateFlow는 수신자 개수와 상관없이 항상 구독하고 있는 생산자의 최신 데이터를 받는다.
- StateFlow는 Hot Stream 방식으로, collector가 데이터를 수집해도 생산자 코드가 트리거 되지 않는다. 그리고 마지막 값의 개념이 있으며, 생성하자마자 활성화 된다.
- zip, flatMapMerge 등 다양한 Flow API를 활용할 수 있다. 

## LiveData vs StateFlow 

| LiveData | StateFlow |
| --- | --- |
| 초기 값이 없음.  | 반드시 초기 값이 있어야 함.  |
| 뷰가 STOPPED 상태가 되면 <br> `LiveData.observe()`에서 이를 인식하여 구독 취소  | `Lifecycle.repeatOnLifecycle` 또는 `Flow.flowWithLifecycle`  <br> 블록 안에서 flow를 수집해야 생명주기에 따른 제어 가능|

# 사용 방법 비교 

ViewModel 생성 단계에서 Loading 상태로 설정해주고, 이어서 비동기 처리를 통해 결과 값을 받아오면 상태를 업데이트 하는 코드를 작성해보자. 

## 생성자로 초기화 

```kotlin
class MyViewModel {
    private val _myUiState = MutableLiveData<Result<UiState>>(Result.Loading)
    val myUiState: LiveData<Result<UiState>> = _myUiState
 
    init {
        viewModelScope.launch { 
            val result = ...
            _myUiState.value = result
        }
    }
}
```

```kotlin
class MyViewModel {
    private val _myUiState = MutableStateFlow<Result<UiState>>(Result.Loading)
    val myUiState: StateFlow<Result<UiState>> = _myUiState
 
    init {
        viewModelScope.launch { 
            val result = ...
            _myUiState.value = result
        }
    }
}
```

## 코루틴 빌더 사용 

```kotlin
class MyViewModel(...) : ViewModel() {
    val result: LiveData<Result<UiState>> = liveData {
        emit(Result.Loading)
        emit(repository.fetchItem())
    }
}
```

```kotlin
class MyViewModel(...) : ViewModel() {
    val result: StateFlow<Result<UiState>> = flow {
        emit(repository.fetchItem())
    }.stateIn(
        scope = viewModelScope, 
        started = WhileSubscribed(5000),
        initialValue = Result.Loading
    )
}
```

- stateIn(): Flow -> StateFlow 변환하는 확장 함수 
- scope: flow 공유가 시작되는 코루틴 스코프 
- started: flow 공유의 시작 및 중지를 제어하는 전략 설정 
  - Eagerly: 수신자와 상관없이 즉시 공유가 시작되며, scope가 취소되면 중지 
  - Lazily: 첫번째 수신자가 나타나면 공유를 시작하고, scope가 취소되면 중지 
  - WhileSubscribed: 지정한 타임아웃이 지나면 upstream flow 취소 (화면 회전 시에는 취소하지 않고, 앱이 백그라운드로 전환되면 취소하는 등의 전략 가능)
- intialValue: StateFlow의 초기 값

# 요약 

**LiveData 대신 StateFlow를 사용하면 얻을 수 있는 이점**을 정리해보자.

- 안드로이드 플랫폼에 종속적이었던 LiveData 와는 달리, StateFlow는 **순수 kotlin 라이브러리**이기 때문에 **Domain Layer에서 사용**할 수 있다.
- 코루틴을 통해 Work Thread에서도 비용이 많이 드는 데이터 스트림을 처리할 수 있기 때문에, **Data Layer에서 LiveData를 사용하는 것보다 향상된 성능**을 기대할 수 있다. 
- StateFlow는 zip, flatMapMerge 등 **다양한 Flow API**를 사용할 수 있기 때문에 LiveData 보다 풍부하게 활용할 수 있다. 

# 참고자료 

https://readystory.tistory.com/207