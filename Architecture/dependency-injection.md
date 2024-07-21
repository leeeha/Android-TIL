# 의존성 주입이란?

클래스는 흔히 다른 클래스에 대한 참조가 필요하다. 예를 들어, Car 클래스는 Engine 클래스에 대한 참조가 필요할 수 있다. 

이때, Car가 Engine에 의존한다고 말하며, Engine을 Car의 의존성 (dependency) 이라고 부른다. 

클래스가 의존성을 얻는 세 가지 방법이 있다. 

1. **클래스가 필요한 의존성을 직접 구성한다.** 예를 들어, Car는 자체 Engine 인스턴스를 생성하여 초기화 할 수 있다.
2. **다른 곳에서 객체를 가져온다.** Context getter, getSystemService()와 같은 일부 안드로이드 API는 이런 방식으로 작동한다. 
3. **객체를 매개변수로 제공받는다.** 앱은 특정 클래스나 함수를 구성할 때 그에 필요한 의존성을 제공할 수 있다. 예를 들어, Car 생성자는 Engine 인스턴스를 매개변수로 받는다.

이 중에서 세 번째 방법이 바로 의존성 주입 (Dependency Injection, DI)이다! 이 방법을 이용하면, **클래스가 의존성을 직접 구성하는 대신에 외부로부터 의존성을 주입 받게 된다.**

## DI 사용하지 않는 경우 

<img width="300" src="https://github.com/user-attachments/assets/cd810f1e-2f8e-4902-bccb-3fcb202511ee"/>

```kotlin
class Car {
    private val engine = Engine()

    fun start() {
        engine.start()
    }
}
```

Car 클래스가 자체 Engine 인스턴스를 직접 구성하는 경우, 다음과 같은 문제가 발생할 수 있다.

- **Car, Engine 객체는 밀접하게 연결되어 있다.** Car 인스턴스는 Engine 객체를 직접 구성하므로, **Engine의 서브 클래스 또는 대체 구현을 쉽게 사용할 수 없다.** GasEngine, ElectricEngine 타입의 서브클래스에 대해 두 가지 타입의 Car 객체를 별도로 생성해야 하기 때문이다.
- **Engine에 대한 의존성이 높은 경우, 테스트가 더 어려워진다.** Car는 실제 Engine 인스턴스를 사용하므로, 다양한 테스트 사례에서 Engine을 수정할 수 없다.

## DI 사용하는 경우 

<img width="300" src="https://github.com/user-attachments/assets/af56b68e-d7f4-4f27-bf2c-99562d67b617"/>

```kotlin
class Car(private val engine: Engine) {
    fun start() {
        engine.start()
    }
}
```

Car 인스턴스는 자체 Engine 객체를 구성하는 대신에, **Engine 객체를 생성자의 매개변수로 받는다.**

- **Car의 재사용 가능**. Engine의 다양한 구현을 Car에 제공할 수 있다. ElectricEngine이라는 서브 클래스를 Car에 제공할 수 있으며, 이때 Car는 추가적인 변경사항 없이 계속 작동할 수 있다. 
- **Car의 테스트 편의성**. FakeEngine 같은 테스트 더블을 생성하여 다양한 시나리오에서 테스트를 수행할 수 있다. 

>참고: 의존성 주입(DI)은 **제어의 역전**(IoC) 원칙을 기반으로 한다. 
>- IoC (Inversion Of Control)는 **어플리케이션의 제어 흐름을 역전**시켜, **객체의 생성과 관리를 개발자가 직접 하지 않고 외부 컨테이너나 프레임워크가 대신 처리**하게 하는 기법이다.
>- IoC는 하나의 디자인 패턴이나 아키텍처 스타일로, **의존성 관리를 위해 사용**된다. 

# DI 적용 방법 

Android에서 의존성 주입을 적용하는 두 가지 주요 방법은 다음과 같다.

- **생성자 주입** (Constructor Injection): 클래스의 의존성을 생성자에 제공한다.
- **필드 주입** (Field Injection): Activity, Fragment 같은 안드로이드 프레임워크 클래스는 시스템에서 인스턴스화하므로 생성자 주입이 불가능하다. 이때는 아래 코드처럼 필드 주입을 사용할 수 있다. 

```kotlin
class Car {
    lateinit var engine: Engine

    fun start() {
        engine.start()
    }
}
```

# 수동으로 의존성 주입 시 문제점 

- 대규모 앱의 경우, 모든 의존성을 가져와 올바르게 연결하려면 **대량의 상용구 코드가 필요**할 수 있다. 예를 들어, 다중 레이어 아키텍처에서는 최상위 레이어의 객체를 생성하려면, 그 아래에 있는 레이어의 모든 의존성을 제공해야 한다.
- 지연 초기화를 사용하거나 객체 범위를 앱의 흐름으로 지정하는 경우, 의존성을 제공하기 전에 미리 구성할 수 없다. 이때는 **메모리에서 의존성의 전체 기간을 관리하는 맞춤 컨테이너** (또는 의존성 그래프)를 작성하고 유지해야 한다.

## Hilt

[Hilt](https://developer.android.com/training/dependency-injection/hilt-android?hl=ko)는 안드로이드에서 의존성을 주입하기 위한 Jepack의 권장 라이브러리이다. Hilt는 **프로젝트의 모든 안드로이드 클래스에 컨테이너를 제공하고 생명주기를 자동 관리**함으로써 어플리케이션에서 DI를 실행하는 표준적인 방법을 정의한다. 

Hilt는 **Dagger가 제공하는 컴파일 타임 정확성, 런타임 성능, 확장성 및 안드로이드 스튜디오 지원의 이점**을 누리기 위해 인기 있는 DI 라이브러리인 Dagger 기반으로 빌드되었다.

# 의존성 주입의 이점 

- **클래스 재사용 가능 및 의존성 분리** : 의존성 구현을 쉽게 교체할 수 있다. 제어의 역전으로 인해 코드 재사용성이 개선되었고, 클래스가 더 이상 의존성 생성 방법을 제어하지 않아도 모든 구성에서 작동한다. 
- **리팩토링 편의성** : 의존성은 API 표면에서 검증 가능한 요소가 되기 때문에, 구현의 세부 정보로 숨겨지지 않고 객체 생성 시간 또는 컴파일 타임에 확인할 수 있다.
- **테스트 편의성** : 클래스는 의존성을 직접 관리하지 않으므로 테스트 시 다양한 구현을 제공하여 다양한 사례를 테스트 해볼 수 있다.

# 참고 자료 

- https://velog.io/@haero_kim/Android-DI-개념-라이브러리-없이-직접-구현해보기
- https://developer.android.com/training/dependency-injection?hl=ko
