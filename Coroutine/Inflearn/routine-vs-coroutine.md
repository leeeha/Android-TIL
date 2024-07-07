## co-routine 의미

- co : 협력하는
- routine : 루틴, 함수
- **co-routine : 협력하는 루틴, 함수**

## 루틴이란?

```kotlin
package org.example

fun main() {
    println("START")
    newRoutine()
    println("END")
}

fun newRoutine() {
    val num1 = 1
    val num2 = 2
    println("${num1 + num2}")
}
```

실행 결과 

```
START
3
END
```

1. main 루틴이 START를 출력한 후 new 루틴을 호출한다. 
2. new 루틴은 1과 2를 더해 3을 출력한다.
3. 이후 new 루틴은 종료되고 main 루틴으로 돌아온다. 
4. main 루틴은 END를 출력하고 종료된다. 

이를 그림으로 나타내면 다음과 같다.

<img width="600" src="https://github.com/leeeha/Android-TIL/assets/68090939/ece9499e-6fdc-480b-aac2-1c6367f38bdb"/>

new 루틴이 종료된 후에는 new 루틴에서 사용했던 num1, num2에는 다시 접근할 수 없으며, 따라서 메모리에서도 정보가 사라질 것이다. 

여기까지 정리하자면 

- **루틴은 진입하는 곳이 한 곳이고**
- **루틴이 종료되면 그 루틴에서 사용했던 정보가 초기화**

된다고 생각할 수 있다. 

## 코루틴이란?

코루틴을 사용하려면 다음과 같이 의존성을 추가해줘야 한다. 

```kotlin
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.2")
    testImplementation(kotlin("test"))
}
```

```kotlin
package org.example

import kotlinx.coroutines.launch
import kotlinx.coroutines.runBlocking
import kotlinx.coroutines.yield

fun main(): Unit = **runBlocking** {
    println("START")
    **launch** {
        newRoutine()
    }
    **yield**()
    println("END")
}

**suspend** fun newRoutine() {
    val num1 = 1
    val num2 = 2
    yield()
    println("${num1 + num2}")
}
```

`runBlocking` 함수는 일반 루틴 세계와 코루틴 세계를 연결하는 함수이다. 이 함수 자체로 **새로운 코루틴을 만들게 되고**, `runBlocking` 에 넣어준 람다가 새로운 코루틴 안에 들어가게 된다. 

다음으로 `launch` 라는 함수 역시 새로운 코루틴을 만드는 함수이다. 주로 **반환값이 없는 코루틴을 만드는데 사용**한다. 즉, 우리는 runBlocking, launch 함수를 사용해 2개의 코루틴을 만든 것이다. 

다음으로 `yield()` 라는 함수는 **지금 코루틴의 실행을 잠시 멈추고 다른 코루틴이 실행되도록 양보**한다. 

마지막으로 함수 앞에 `suspend` 키워드를 붙이면 **다른 suspend 함수를 호출**할 수 있게 된다. yield 함수가 바로 suspend 함수이므로 newRoutine 함수 앞에 suspend 키워드를 붙인 것이다. 

위 코드의 실행 결과는 다음과 같다. 

```
START
END
3
```

yield 함수를 호출하지 않더라도 동일한 결과가 나온다. 

1. main 코루틴이 runBlocking에 의해 시작되고 START가 출력된다. 
2. launch에 의해 새로운 코루틴이 생긴다. 하지만, newRoutine의 실행은 바로 일어나지 않는다. 
3. main 코루틴 안에 있는 yield가 실행되면 main 코루틴은 new 코루틴에게 실행을 양보한다. 따라서 launch가 만든 새로운 코루틴 안에서 newRoutine 함수가 실행된다. 
4. newRoutine 함수는 다시 yield 함수를 호출하고 main 코루틴으로 돌아온다. 
5. main 루틴은 END를 출력하고 종료된다. 
6. 아직 newRoutine 함수가 끝나지 않았으니, newRoutine 함수로 돌아가 3이 출력되고 프로그램이 종료된다. 

일반 루틴 세계에 비해 복잡하고 직관적이지 않다.. 😅 이를 그림으로 나타내면 다음과 같다. 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/d2bb1bad-4e66-47b9-b7af-214fba47ce0f"/>

위의 그림을 보면 알 수 있듯이, 루틴과 코루틴의 가장 큰 차이점은 **‘중단’**과 **‘재개’**이다. **루틴은 한 번 시작되면 종료될 때까지 멈추지 않지만, 코루틴은 상황에 따라 잠시 중단되었다가 다시 시작되기도 한다.** 따라서 완전히 종료되기 전까지는 newRoutine 함수 안에 있는 num1, num2 변수가 메모리에서 제거되지 않는다.

IntelliJ의 VM optines에 `-Dkotlinx.coroutines.debug` 를 설정하면 어떤 코루틴에서 출력이 일어났는지 확인할 수도 있다.

<img width="900" src="https://github.com/leeeha/Android-TIL/assets/68090939/c0b5b577-40a5-43e3-8108-d6553f9b8224"/>

```kotlin
package org.example

import kotlinx.coroutines.launch
import kotlinx.coroutines.runBlocking
import kotlinx.coroutines.yield

fun main(): Unit = runBlocking {
    printWithThread("START")
    launch {
        newRoutine()
    }
    yield()
    printWithThread("END")
}

suspend fun newRoutine() {
    val num1 = 1
    val num2 = 2
    yield()
    printWithThread("${num1 + num2}")
}

fun printWithThread(params: Any) {
    println("[${Thread.currentThread().name}] $params")
}
```

실행 결과 

```
[main @coroutine#1] START
[main @coroutine#1] END
[main @coroutine#2] 3
```

START, END 문자열은 runBlocking에 의해 만들어진 코루틴에서, 숫자 3은 launch에 의해 만들어진 코루틴에서 출력된다는 걸 알 수 있다. 

## 루틴 vs 코루틴

| 루틴 | 코루틴 |
| --- | --- |
| 시작되면 끝날 때까지 멈추지 않는다. | 중단되었다가 재개될 수 있다. |
| 한번 끝나면 루틴 내의 정보가 사라진다. | 중단되더라도 루틴 내의 정보가 사라지지 않는다. |

우리가 항상 직관적으로 사용했던 루틴과 다르게, 코루틴은 **중간에 멈추었다가 다시 시작할 수 있다**는 특징이 핵심이다.