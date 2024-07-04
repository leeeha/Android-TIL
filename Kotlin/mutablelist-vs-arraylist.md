안드로이드에서 리사이클러뷰 어댑터 안에 가변형 리스트를 만들 때, MutableList와 ArrayList 중에 무엇으로 선언해야 하는지 고민될 때가 있었다. 그래서 그 차이점에 대해 간단하게 알아보았다. 

## MutableList vs ArrayList

```kotlin
/**
 * Returns an empty new [MutableList].
 * @sample samples.collections.Collections.Lists.emptyMutableList
 */
@SinceKotlin("1.1")
@kotlin.internal.InlineOnly
public inline fun <T> mutableListOf(): MutableList<T> = ArrayList()

/**
 * Returns an empty new [ArrayList].
 * @sample samples.collections.Collections.Lists.emptyArrayList
 */
@SinceKotlin("1.1")
@kotlin.internal.InlineOnly
public inline fun <T> arrayListOf(): ArrayList<T> = ArrayList()
```

이 둘의 유일한 차이점은 **의도를 전달하는 것**이다. 

`val a = mutableListOf()`를 작성하면 "나는 가변 목록을 원하며, 구현에는 특별히 신경 쓰지 않는다"고 말하는 것입니다. 대신 `val a = ArrayList()`를 작성하면 "나는 특별히 ArrayList를 원한다"고 말하는 것이다. 

실제로 현재 JVM으로 컴파일되는 Kotlin의 구현에서 `mutableListOf()`를 호출하면 **ArrayList가 생성되며, 동작에는 차이가 없다.** 일단 리스트가 빌드되면 모든 것이 동일하게 동작한다.

Kotlin의 향후 버전에서 mutableListOf가 다른 타입의 리스트를 반환하도록 변경되었다고 가정해보자. 

대부분의 사용 사례에서 새 구현이 더 잘 작동한다고 판단되는 경우에만 Kotlin 팀에서 변경할 가능성이 높다. mutableListOf를 통해 새로운 리스트 구현을 투명하게 사용할 수 있고, 더 나은 동작을 무료로 사용할 수 있다. 이 경우에 적합하다면 mutableListOf를 사용하면 된다. 

반면에, **문제에 대해 많은 시간을 고민한 결과 ArrayList가 정말 문제에 가장 적합하다고 생각했고, 차선책으로 넘어가는 위험을 감수하고 싶지 않을 수도 있다.** 그렇다면 ArrayList를 직접 사용하거나 arrayListOf 팩토리 함수를 사용하면 된다.

## 요약

- MutableList: 추가/수정/삭제 등이 가능한 가변 리스트
    - **Mutable한 리스트를 사용하겠다.**
- ArrayList: 내부적으로 배열로 구현되어 있는, 추가/수정/삭제 가능한 가변 리스트
    - **MutableList 중에서도 ArrayList를 사용하겠다.**
- mutableListOf(), arrayListOf() → **모두 ArrayList 반환** (동작에 차이 없음)

## 참고자료

[[Android/kotlin] List vs MutableList vs ArrayList vs LinkedList](https://zladnrms.tistory.com/140)

https://stackoverflow.com/questions/43114367/difference-between-arrayliststring-and-mutablelistofstring-in-kotlin
