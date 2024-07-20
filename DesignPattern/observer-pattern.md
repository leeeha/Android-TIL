# 옵저버 패턴이란?

Observer는 관찰자, 감시자 라는 뜻을 갖고 있다. 즉, 옵저버 패턴이란 **어떤 이벤트가 일어나는 것을 감시하는 패턴**이다. 

안드로이드를 예로 들면, 다음과 같은 것들을 모두 '이벤트'라고 할 수 있다. 

- 사용자가 키보드를 눌렀을 때 
- 사용자가 어떤 버튼을 클릭했을 때
- 호출한 API의 응답 데이터가 수신됐을 때 

즉, 개발자가 함수로 직접 요청한 적 없지만 **시스템에 의해 발생하는 동작들**을 이벤트라고 한다. 

이러한 이벤트를 감시하여 **이벤트가 발생할 때마다 미리 정의해둔 어떤 동작을 즉시 수행하게 해주는 디자인 패턴**을 옵저버 패턴이라고 한다. 

ex) A라는 버튼이 클릭될 때마다 화면에 '안녕'을 출력하는 동작 등 

옵저버 패턴을 이용하면 **다른 객체의 상태 변화를 별도의 함수 호출 없이도 즉시 알 수 있기** 때문에, 이벤트 처리를 자주 해야 하는 경우 **매우 효율적인 프로그램을 작성**할 수 있다.

# 구현 원리 

옵저버 패턴은 어떻게 구현할 수 있을까? 아래 그림으로 요약할 수 있다.

<img width="600" src="https://github.com/user-attachments/assets/9a7abf4f-cea1-402a-8f64-f46ce9ebd93f"/>

- A : B의 이벤트를 수신하는 객체 
- B : 이벤트를 발생시키는 객체 

A는 인터페이스를 상속하여 **B의 이벤트가 발생할 때마다 실행할 메소드를 구현**해둔다. 

그리고 B 객체를 생성할 때 해당 인터페이스 구현체를 생성자로 전달한다. 

그러면 B는 **이벤트가 발생할 때마다 A가 구현한 인터페이스의 메소드를 실행**하면 된다.

이렇게 이벤트를 수신하는 A와, 이벤트를 발생시키는 B 사이에서 중개자 역할을 하는 인터페이스를 **옵저버, 리스너**라고 한다.

그리고 B가 A의 메소드를 호출하여 **이벤트를 전달하는 행위**를 **콜백**이라고 한다. 

# 구현 방법 

### 이벤트를 관찰하는 옵저버 (리스너)

```kotlin
interface EventListener {
    fun onEvent(count: Int)
}
```

### B: 이벤트가 발생할 때마다 콜백 메소드 실행 

```kotlin
// 5의 배수라는 이벤트가 발생할 때마다 리스너의 메소드 호출 
class Counter(val listener: EventListener) {
    fun count(){
        for(i in 1..100){
            if(i % 5 == 0){
                listener.onEvent(i)
            }
        }
    }
}
```

### A: 리스너의 콜백 메소드 구현 

```kotlin
// 이벤트가 수신될 때마다 실행되는 리스너의 메소드 구현 
class EventPrinter : EventListener {
    override fun onEvent(count: Int){
        print("${count} ")
    }

    fun start(){
        Counter(this).count()
    }
}
```

```kotlin
class EventPrinter {
    fun start(){
        // 익명 객체에서 EventListener의 추상 메서드 구현 
        Counter(object: EventListener {
            override fun onEvent(count: Int){
                print("${count} ")
            }
        }).count()
    }
}
```

### 프로그램 실행부 

```kotlin
fun main(){
    EventPrinter().start()
}
```

# 참고 자료 

https://velog.io/@haero_kim/옵저버-패턴-개념-떠먹여드립니다