# 싱글톤 패턴  

싱글톤 패턴은 **프로그램의 시작과 종료까지 클래스의 인스턴스를 단 한번만 생성**하여, **프로그램 전역에서 해당 인스턴스를 공유**해서 사용하는 디자인 패턴이다.

### 언제 사용하는 게 좋을까?

- 프로그램 전역에서 자주 사용되고, 일관된 동작을 하는 인스턴스인 경우 
- 인스턴스 생성 비용이 많이 드는 경우 (예: 데이터베이스 또는 디스크 연결에 필요한 객체, 네트워크 통신을 위한 객체 등)

### 장점 

- 프로그램 전역에서 동일한 인스턴스를 공유하므로, 객체 생성 비용과 메모리를 절약할 수 있다.

### 단점 

- 멀티 스레드 환경에서 동기화 처리를 제대로 해주지 않으면, 인스턴스가 여러 개 생성될 수 있으므로 주의가 필요하다. (코틀린에서는 object 사용)
- 자주 사용되지 않는 객체를 싱글톤으로 만들면, 프로그램 종료 시까지 계속 메모리에 남아있기 때문에 메모리 공간 낭비가 발생할 수 있다.
- 싱글톤 객체가 하는 일이 너무 많아지면, 단일 책임 원칙(SRP)에 위배될 수 있다. 
- 프로그램 전역에서 싱글톤 객체를 공유하기 때문에, 클래스 간의 결합도가 높아져 개방 폐쇄 원칙(OCP)에 위배될 수 있다.

위와 같은 장단점을 고려하여 싱글톤 패턴이 꼭 필요할 때만 사용하는 것이 중요하다! 

## 자바에서 구현 

자바에서 싱글톤 패턴을 구현하는 방법은 [이 블로그](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%8B%B1%EA%B8%80%ED%86%A4Singleton-%ED%8C%A8%ED%84%B4-%EA%BC%BC%EA%BC%BC%ED%95%98%EA%B2%8C-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90) 설명처럼 여러가지가 있는데, 그 중에서 Thread-Safe Lazy Initialization 방법에 대해 살펴보자.

```java
public class SingletonObj {
    private static SingletonObj singletonObj = null;
    private SingletonObj() { }

    public static synchronized SingletonObj getInstance() {
        if (singletonObj == null) {
            singletonObj = new SingletonObj();
        }

        return singletonObj;
    }
}
```

- 생성자를 private으로 만들어서 외부에서 객체를 생성할 수 없게 한다. 
- 외부에서 싱글톤 객체를 사용할 때는 클래스 이름으로 참조하도록 getInstance() 메소드를 static으로 만든다.
- getInstance() 함수는 최초에 한번만 객체를 생성하고, 이후에는 이미 생성된 객체를 반환한다. 
- 여러 스레드가 동시에 getInstance() 함수를 호출하여 객체가 여러 개 생성되지 않도록 synchronized 키워드로 동기화한다. 
- getInstance() 함수를 호출할 때마다 동기화 하면 성능 저하가 발생하기 때문에, 객체를 초기화 할 때만 동기화를 적용하는 방법도 있다. 

## 코틀린에서 구현 

코틀린은 언어 차원에서 Thread-safe 하고 Lazy 한 초기화를 지원해준다. 

```kotlin
object SingletonObj {

}
```

위의 코드처럼 object 키워드를 사용하면

멀티 스레드 환경에서 **단일 인스턴스 생성이 보장**되며, 클래스가 **실제로 사용될 때까지 초기화가 지연**된다.

## 참고자료 

https://velog.io/@haero_kim/혹시-싱글톤이세요-저는-벙글톤이에요-ㅋㅋㅋ

https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%8B%B1%EA%B8%80%ED%86%A4Singleton-%ED%8C%A8%ED%84%B4-%EA%BC%BC%EA%BC%BC%ED%95%98%EA%B2%8C-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90

https://github.com/leeeha/Android-TIL/blob/main/Architecture/google-docs-hilt.md