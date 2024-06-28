# 학습 목표 

- Java Inner class vs. Nested Class
- Kotlin Inner class vs. Nested Class
- Java의 어떤 문제점 때문에 코틀린은 기본으로 Nested Class를 채택했는지

# Java Inner class vs. Nested Class

자바는 중첩 클래스를 정의하면 **기본 Inner Class로 정의**한다. 

다음과 같이 별도로 명시하지 않아도 Inner Class의 외부 클래스인 **Outer의 멤버에 자유롭게 접근할 수 있다.** 

```java
class Outer {
    private String outer = "Outer";

    class InnerClass {
        public InnerClass() {
            System.out.println(outer);
        }
    }
}
```

Inner Class는 내부적으로 **외부 클래스에 대한 참조를 묵시적으로 포함**하고 있기 때문에, 외부 클래스의 멤버에 자유롭게 접근할 수 있는 것이다.

여기서 **Nested Class로 변경하려면 static 키워드를 추가**하면 된다. 그러면 Inner Class는 외부 클래스에 대한 참조가 사라져서 다음과 같은 오류가 발생한다. 

> *Non-static field ‘outer’ cannot be referenced from a static context.*
> 

```java
class Outer {
    private String outer = "Outer";

    static class InnerClass { // Nested Class로 변경 
        public InnerClass() {
            System.out.println(outer); // error 
        }
    }
}
```

이 에러를 없애기 위해 다시 Inner Class로 돌리거나, Inner Class 생성자에 Outer 객체를 넘겨주는 방법을 택할 수도 있다. 

```java
class Outer {
    private final String outer = "Outer";

    static class InnerClass {
        // 생성자에 Outer 객체를 넘겨준다.
        public InnerClass(Outer out) {
            System.out.println(out.outer);
        }
    }
}
```

# Kotlin Inner class vs. Nested Class

코틀린은 자바와 완전히 반대로 동작하는데, 기본적으로 **중첩 클래스를 Nested Class로 정의한다.** 처음부터 묵시적으로 외부 클래스를 참조할 수 없도록 설계되었다. 

따라서 다음 코드는 즉시 오류가 발생한다. 

> *Unresolved reference: outer*
> 

```kotlin
class Outer {
    private val outer = "Outer"

    class InnerClass {
        init {
            print(outer) // error 
        }
    }
}
```

위의 에러를 해결하는 방법은, Java의 static과 마찬가지로 Inner Class 생성자에 Outer 객체를 넘기거나, inner 키워드를 추가하여 묵시적으로 Outer 클래스를 참조하도록 만드는 것이다. 

```kotlin
class Outer {
    private val outer = "Outer"

    inner class InnerClass { 
        init {
            print(outer)
        }
    }
}

```

```kotlin
class Outer {
    private val outer = "Outer"

    class InnerClass(
        private val out: Outer
    ) {
        init {
            print(out.outer)
        }
    }
}

```

# Inner class vs. Nested Class

|  | Java | Kotlin |
| --- | --- | --- |
| Nested Class  | static class A | class A (기본) |
| Inner Class | class A (기본) | inner class A |

자바든 코틀린이든 **모든 Nested Class는 외부 클래스에 대한 묵시적인 참조를 제공하지 않으므로, 결국 서로 다른 클래스로 취급**받게 된다. 

```java
// Java 
Outer.InnerClass innerClass = new Outer.InnerClass();
```

```kotlin
// Kotlin 
val innerClass = Outer.InnerClass()
```

반면에, **Inner Class는 항상 밖에 있는 외부 클래스가 생성되어야, 안에 있는 Inner Class도 생성**할 수 있다.

아래 코드처럼 Outer 객체가 먼저 생성되고, 그 다음에 Inner Class 객체가 생성될 수 있다. 따라서, **Inner Class가 Outer의 멤버를 사용하지 않아도 불필요하게 Outer 객체는 항상 생성되어야 한다.** 

```kotlin
val innerClass = Outer().InnerClass()
```

# 코틀린은 왜 기본으로 Nested Class인가?

이펙티브 자바, 코틀린 인 액션 책을 참고해보면, **자바의 Inner Class**는 크게 3가지 문제가 있다. 

- 객체를 바이트 코드로 바꾸는 직렬화 과정에서 Inner Class를 사용하는 경우 문제가 생길 수 있다.
- **Inner classes 내부에 Outer class 정보를 묵시적으로 보관**하는데, 이에 대한 참조를 해지하지 못하면 **메모리 누수**가 생길 수도 있고, 코드를 분석하더라도 이를 찾기가 쉽지 않아 문제를 해결하지 못하는 경우도 생긴다.
- 자바는 Outer의 멤버를 사용할 일이 없어도 기본 Inner Class이기 때문에, **불필요한 메모리 낭비와 성능 이슈**를 일으킬 수 있다.

이런 문제를 해결하기 위해 자바는 이펙티브 자바를 통해 가이드 하고 있고, **코틀린은 처음부터 이를 배제하고 기본 Nested Class를 따르도록 설계했다**고 보면 좋을 거 같다. 

결국 문제가 하나라도 있는 걸 배제한 게 코틀린이라고 보면 되겠다. 

# 안드로이드 리사이클러뷰 내부 클래스 살펴보기

리사이클러뷰에는 아래와 같이 3가지 종류의 클래스 정의를 갖고 있다.

- private class
- public static class
- public inner class

```java
public class RecyclerView extends ViewGroup {
    // private inner class
    private class ItemAnimatorRestoreListener implements ItemAnimator.ItemAnimatorListener {
   }

    // public nested class 
    public abstract static class ViewHolder {
    }

    // inner class 이지만 외부에서 생성 가능
    public final class Recycler {
    }
}
```

### ItemAnimatorRestoreListener

이 리스너는 private inner class로 정의되어 있다. RecyclerView 외부에서는 이 클래스가 있는지조차 알 필요가 없고, RecyclerView 입장에서는 이 클래스가 밖에서 수정되는 것에 대해 걱정할 필요도 없다. 

### ViewHolder

뷰홀더는 static을 통해 **inner class가 아님을 명시**했다. 즉, **이 클래스는 RecyclerView와 독립적으로 운용 가능하다**는 뜻이고, 동시에 추상 클래스이므로 상속 받은 경우 오버라이딩이 필수적이다. 

이 클래스는 RecyclerView 없이 단독 사용 가능하기 때문에, **ViewHolder 밖의 RecyclerView를 신경 쓸 필요가 없고, 단독으로 수정이 일어나도 서로에게 영향을 미치지 않는다.** 

```kotlin
val viewHolder = object : RecyclerView.ViewHolder(view) {
    // 상속 구현
}
```

### Recycler

public inner class이므로 외부에서 다음과 같이 초기화 해야 한다. 

```kotlin
val recycler = RecyclerView(this).Recycler()
```

# 그래서, 기본은 Nested Class 사용 추천!

- Outer의 멤버를 참조할 필요가 없다면, 굳이 `inner` 키워드를 사용하지 않아야 한다.
- Inner class를 보호하고 싶다면 `private`를 명시해라.
- 수정 가능한 상태로 두고 싶다면 `open` 키워드를 명시할 수 있다.

코틀린의 기본 클래스를 디컴파일 하면 다음과 같다. 

```kotlin
class Outer {
    class Nested {
    }
}
```

```java
public final class Outer {
   public static final class Nested { // 기본 nested class 
   }
}
```

이미 자바에서 생겼던 문제를 코틀린이 어느정도 방어하고, 이펙티브 자바에서도 가이드 하고 있으니, **굳이 외부 클래스에 대한 참조가 필요하지 않다면 inner class를 사용하지 말자.** 정말 필요하다면 키워드를 잘 사용해야 한다. 

inner, private, open 키워드를 적절하게 활용하는 게 가장 좋은 구조를 만들 수 있고, **수정 가능성을 최소화** 시킬 수 있다.

# 안드로이드의 ViewHolder는?

**Inner class 보다는 Nested class, 또는 top level 로 만드는 것을 추천한다.** 

- 항상 하나의 어댑터에서만 활용되는 게 아니다.
- RecyclerView는 실제로 화면 사이즈보다 많이 onCreateViewHolder가 호출되고 뷰홀더가 만들어진다.

화면 크기보다 더 많이 만들어지는데, **Inner class로 정의한다면 생성되는 뷰홀더 객체마다 묵시적으로 외부 클래스를 항상 참조하게 된다.**

하지만 안드로이드에서는 Activity/Fragment에서 리사이클러뷰를 사용하고 있고, 이들이 onDestroy 되면서 Adapter 역시 함께 사라지고, 그럼 ViewHolder도 함께 사라지기 때문에, 평소에 inner class로 사용해도 큰 문제는 없었던 것이다. 

꼭 필요하지 않다면 코틀린의 기본 값인 Nested class를 굳이 inner로 바꿀 필요는 없다고 생각한다. 조금 더 좋은 접근법인 top level로 작성하는 것도 고려하는 게 좋다고 생각한다.

cf) [ViewHolder를 top level 클래스로 작성하는 예시](https://github.com/taehwandev/AndroidMVPSample/blob/e6ce0eba9989c200e04f3d7bb75180d2d2f32383/app_kotlin/src/main/java/tech/thdev/app_kotlin/adapter/ImageViewHolder.kt)

# 요약

- Inner class는 정말 필요하지 않으면 쓰지 않는 게 좋다.
- 해당 클래스에서만 사용함이 명확하다면 private을 걸어두어야 한다.
- 참조를 어떻게 접근할 것인지에 따라 inner 키워드가 유용할 수 있다.
- 자바를 잘 알면 Kotlin의 Nested classes가 왜 기본인지 이해하기 쉽다.
- 꼭 필요한 게 아니라면 `inner` 키워드와 `open`을 필요할 때만 허용하자.

# 참고자료

[Kotlin과 Java의 Nested and Inner Classes를 알아보고, Nested classes를 왜 사용해야 하는지 알아본다.](https://thdev.tech/kotlin/2020/11/17/kotlin_effective_11/)
