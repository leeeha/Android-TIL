# GlobalScope 

CoroutineScope.kt 파일에 정의된 GlobalScope에 적혀있는 주석을 직접 읽어보고, 사용을 지양해야 되는 이유에 대해 알아보자! 

```kotlin 
@DelicateCoroutinesApi
public object GlobalScope : CoroutineScope {
    /**
     * Returns [EmptyCoroutineContext].
     */
    override val coroutineContext: CoroutineContext
        get() = EmptyCoroutineContext
}
```

- `GlobalScope`는 특정한 job에 바인딩 되지 않은 **전역적인 CoroutineScope**이다.  
- `GlobalScope`는 **어플리케이션 전체의 생명주기 동안 동작**하며, 작업이 조기 종료되지 않도록 설계되었다.  

## Delicate API 경고

`GlobalScope`는 **주의가 필요한 API**이다. `GlobalScope`를 사용할 경우, **리소스 또는 메모리 누수가 발생하기 쉽다.** 

`GlobalScope`에서 실행된 코루틴은 **구조적 동시성의 원칙을 따르지 않으므로**, 만약 해당 코루틴이 중단되거나 지연(예: 느린 네트워크)될 경우 **계속 실행되며 리소스를 소모**하게 된다. 

다음은 이와 관련된 코드 예시이다. 

```kotlin
fun loadConfiguration() {
    GlobalScope.launch {
        val config = fetchConfigFromServer() // 네트워크 요청
        updateConfiguration(config)
    }
}
```

위 코드에서 `loadConfiguration`을 호출하면 `GlobalScope`에 새로운 코루틴이 생성된다. 이 코루틴은 백그라운드에서 실행되며, **취소하거나 완료를 기다릴 방법이 제공되지 않는다.** 

만약 네트워크가 느리다면 **코루틴은 계속 대기하면서 리소스를 소비**한다. 또한, `loadConfiguration`을 반복적으로 호출하면 리소스를 계속 소모하는 코루틴이 늘어나게 된다. 

## 가능한 대안 

### GlobalScope 제거 및 suspend 함수 사용

대부분의 경우 `GlobalScope` 사용을 제거하고, `suspend` 함수로 만드는 것이 좋다. 

```kotlin
suspend fun loadConfiguration() {
    val config = fetchConfigFromServer() // 네트워크 요청
    updateConfiguration(config)
}
```

### coroutineScope 사용해 동시 작업 그룹화

여러 개의 동시(concurrent) 작업을 실행하려면, `GlobalScope.launch` 대신 `coroutineScope`를 사용하는 것이 좋다. 

```kotlin
// 설정 및 데이터를 동시에 로드
suspend fun loadConfigurationAndData() {
    coroutineScope {
        launch { loadConfiguration() }
        launch { loadData() }
    }
}
```

### 제한된 CoroutineScope 사용
 
- top-level 코드의 non-suspending 컨텍스트에서 동시 작업을 수행하는 경우, `GlobalScope` 대신 **제한된 CoroutineScope**를 사용하는 것이 권장된다. 
- 자세한 내용은 [CoroutineScope](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/) 문서를 참조하자. 

## GlobalScope vs custom scope 

`GlobalScope.launch { ... }`를 `CoroutineScope().launch { ... }`로 단순히 대체하면 안 된다. 

`CoroutineScope()` 생성자를 사용할 때도 `GlobalScope`와 동일한 문제점이 발생할 수 있다. `CoroutineScope()` 생성자는 의도에 맞게 설계되어야 한다. 

## 정당한 사용 사례

`GlobalScope`를 합법적이고 안전하게 사용할 수 있는 상황은 제한적이다. 예를 들어, **어플리케이션의 전체 생명주기 동안 활성 상태를 유지해야 하는 top-level의 백그라운드 작업**이 그 예시이다. 

이러한 이유로, `GlobalScope`를 사용하는 모든 경우에는 명시적으로 `@OptIn(DelicateCoroutinesApi::class)` 어노테이션을 붙여야 한다. 

참고: @OptIn 어노테이션은 **실험적(Experimental) 또는 주의가 필요한(Delicate) API를 명시적으로 사용하겠다는 의도**를 나타낸다. 이를 통해 안정적이지 않은 API를 사용할 때 발생할 수 있는 위험성을 개발자가 인지하고 있다는 것을 컴파일러에게 알린다. 

```kotlin
// 어플리케이션의 생명주기 동안 항상 활성 상태여야 하는 글로벌 코루틴
@OptIn(DelicateCoroutinesApi::class)
val globalScopeReporter = GlobalScope.launch {
    while (true) {
        delay(1000)
        logStatistics()
    }
}
```
