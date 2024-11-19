## static이란? 

static은 **정적인, 고정된**이라는 뜻을 가지고 있다. 

즉, static 키워드로 변수나 메서드를 정의하면, 이는 메모리에 고정되어 **프로그램 실행 주기 동안에 여러 객체가 공유**할 수 있게 된다. 

정적 변수와 정적 메서드는 인스턴스 없이도, **클래스가 메모리에 로드되면 바로 사용할 수 있다**는 점에서 **클래스 멤버**라고도 불린다. 

이들은 **프로그램이 종료되기 전까지 사용 가능**하며, 가비지 컬렉션의 대상이 되지 않는다. (단, 정적 객체는 더 이상 참조되지 않으면 GC에 의해 수집될 수 있다.)

## static이 저장되는 위치 

<img width="800" src="https://github.com/user-attachments/assets/bc03e0b1-0bad-4dce-91f6-87cb7c3004e2">

Java 8 이전에는 **클래스의 메타 데이터와 정적 변수 등이 모두 Permanent 영역**에서 관리되었다. 이 영역은 **JVM에 의해 크기가 강제**되어서 OOM(Out Of Memory) 에러가 자주 발생하곤 했다. 

이런 문제를 해결하기 위해 Java 8부터는 **클래스 메타 데이터를 Metaspace 영역**에서, **정적 변수는 Heap 영역**에서 관리하도록 분리되었다. 

Metaspace 영역은 **Native Memory 영역**으로, **OS가 자동으로 크기를 조절**한다. 따라서, 메모리 크기 제한으로 인한 오류를 줄일 수 있게 되었다. 

- static 저장 위치: Java 8 기준으로, Permanent 영역 -> Heap 영역 
- 클래스 메타 데이터 저장 위치: Java 8 기준으로, Permanent 영역 -> Metaspace 영역 

## static 지양해야 되는 이유 

- **메모리 문제**
  - 프로그램이 실행되는 동안에 GC에 의해 수집되지 않고 메모리를 점유하고 있다.
- **동시성 이슈** 
  - 전역에서 static 변수 및 메서드에 접근 가능하므로 별도의 동기화 전략을 세워야 한다. 
- **런타임 다형성 불가**
  - static 메서드는 인스턴스가 아니라, 클래스 자체에 종속되므로 메서드 오버라이딩이 불가능하다. 
  - 따라서, 런타임에 실제로 가리키고 있는 객체 타입에 따라 동적으로 메서드를 결정하는 런타임 다형성을 구현할 수 없다. 
- **객체의 상태 이용 불가** 
  - static 멤버는 인스턴스 멤버를 사용할 수 없다. (아직 초기화 되지 않은 인스턴스에 staic 메서드가 접근하면 문제가 발생하므로)
  - 따라서 static 멤버는 본인의 상태를 가질 수 없으며, 필요한 모든 값을 외부에서 주입 받아야 한다. 
  - 그에 따라 외부에 의존하는 수동적인 객체가 되어버릴 수 있다. 
- **테스트의 어려움** 
  - static 필드는 전역으로 관리되기 때문에 프로그램 전체에서 해당 필드에 접근하여 값을 수정할 수 있다. 
  - 따라서, 해당 필드를 추론하기 어려워 테스트하기 까다롭다. 

```java
class Animal {
    static void staticMethod() {
        System.out.println("Static method in Animal");
    }

    void instanceMethod() {
        System.out.println("Instance method in Animal");
    }
}

class Dog extends Animal {
    static void staticMethod() {
        System.out.println("Static method in Dog");
    }

    @Override
    void instanceMethod() {
        System.out.println("Instance method in Dog");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal animal = new Dog();
        animal.staticMethod();   // Static method in Animal
        animal.instanceMethod(); // Instance method in Dog (동적 다형성)
    }
}
```

## static 사용 사례 

그럼에도, static이 유용한 경우도 몇가지 있다. 

프로그램 전체에 걸쳐 **변하지 않는 상수** 값을 정의하고 싶을 때 static을 사용하면, 메모리를 절약할 수 있다. 

```java
private static final String NAME = "leeeha";
```

**유틸리티 클래스**는 인스턴스 변수 및 메서드를 제공하지 않고, **데이터 처리를 위한 정적 메서드만 존재하는 클래스**를 말한다. 

대표적으로 java의 Math 클래스는 상수 외에 인스턴스 변수가 하나도 없고, 계산을 위한 정적 메서드만 제공한다. 

이처럼, **애초부터 객체의 상태를 이용할 생각이 없고, 여러 객체의 필요에 의해 데이터를 처리하는 공통 로직이 필요할 때**는 static을 사용하는 게 좋다. 

## 참고자료 

https://steady-coding.tistory.com/603

https://johngrib.github.io/wiki/java8-why-permgen-removed/