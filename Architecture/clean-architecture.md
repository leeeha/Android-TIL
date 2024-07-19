# 클린 아키텍처

## 클린 아키텍처의 이점

Clean Architecture의 창시자인 Robert C. Martin이 운영하는 [The Clean Code Blog](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)에 의하면, Clean Architecture가 시스템에 주는 이점은 아래와 같다.

1. **클린 아키텍처는 프레임워크에 독립적이다.** <br>
클린 아키텍처는 프레임워크의 제약에 맞추는 것이 아니라, 시스템의 도구로서 프레임워크를 사용할 수 있게 해준다.

2. **클린 아키텍처는 테스트를 용이하게 해준다.** <br>
비즈니스 규칙을 테스트 할 때는 외부 요소를 필요로 하지 않는다. 

3. **클린 아키텍처는 UI에 독립적이다.**  <br>
다른 시스템의 변경 없이, UI를 쉽게 변경할 수 있다. 예를 들면, 웹 UI에서 콘솔 UI로 변경한다고 할 때, 비즈니스 규칙을 변경하지 않고 그대로 사용할 수 있다.

4. **클린 아키텍처는 데이터베이스와 독립적이다.**  <br>
비즈니스 규칙이 데이터베이스에 의존하지 않으므로, DB 종류를 쉽게 변경할 수 있다.

5. **클린 아키텍처는 외부와 독립적이다.**  <br>

비즈니스 규칙은 외부에 대해 알 필요가 없고, 전혀 알지 못한다. 따라서, 1~4번까지의 이점을 가져올 수 있는 것이다. 

## 의존성 규칙

<img width="600" src="https://github.com/user-attachments/assets/a5618f7d-90fb-4454-8647-ca2ce4326790"/>

위의 그림에서 **의존성 방향은 Entities로 향하는 화살표**이다. 즉, 항상 **원의 바깥쪽에서 안쪽으로만 의존성이 존재**해야 하고, 그 반대로 **바깥쪽 방향으로의 의존성은 존재하면 안 된다**는 의미이다.

바깥쪽 요소 A가 안쪽 요소 B에 의존하는 것을 UML의 표현을 빌려 다음과 같이 나타낼 수 있다. 

A - - - > B

그렇다면, 과연 안쪽 요소 B는 바깥쪽 요소 A를 절대 필요로 하지 않을까?

아래에서 다시 언급하겠지만, 그렇지 않다. 다만 안쪽 요소 B가 의존 관계를 회피하여 A를 사용할 뿐이다. 이것은 **의존성 역전 법칙**으로 가능하다. 

의존성 방향은 **변경의 유연성**과도 관련 있다. 클린 아키텍처에서는 의존성 방향이 안쪽으로만 향하기 때문에, **바깥쪽에 있을수록 변하기 쉬운, 안쪽에 있을수록 변하기 어려운 요소**라고 이해할 수 있다. 

# 클린 아키텍처의 Layer

위에서 보았던 양파 껍질 같은 그림을 90도 회전하면 다음과 같다. 

<img width="600" src="https://github.com/user-attachments/assets/41b68017-67f8-4fc4-acac-2d199b5fe122"/>

<img width="800" src="https://github.com/user-attachments/assets/acd21b1e-4f4c-45f0-bf1c-ea96141a4c05"/>

## Presentation Layer

- UI (Activity, Fragment), Controller (Presenter, ViewModel) 포함
- Controller는 1개 이상의 UseCase를 실행한다.
- UI는 UI 각각의 Controller에 의해 조정된다.
- **Presentation Layer는 Domain Layer에 의존한다.**

## Domain Layer

- Entity, UseCase, Repository Interface 포함
- UseCase는 Entity와 1개 이상의 Repository Interface를 결합한다.
- **핵심 비즈니스 규칙을 포함**하고 있기 때문에, 다른 레이어와 달리 **쉽게 변경되어서는 안 된다.**
- Domain Layer는 가장 안쪽에 있는 레이어로, 앞서 언급한 의존성 규칙에 따라 **다른 어떤 레이어에도 의존성을 갖지 않는다.**
- **안드로이드 프레임워크에 의존성을 갖지 않으며, 오직 자바, 코틀린 같은 언어 단의 모듈에만 의존성을 갖는다.** 따라서, 프레임워크 간의 **도메인 레이어 재사용**도 가능하다.
- **Domain Layer가 Data Layer에 의존하지 않을 수 있는 이유는?**
    - **의존성 역전 법칙** (DIP, Dependency Inversion Principle) 덕분에 가능!
    - **DIP는 상위 레벨의 모듈은 하위 레벨의 모듈에 의존해서는 안 되며, 둘 다 추상화에 의존해야 한다는 법칙**
    - Domain Layer는 Data Layer에 있는 Repository의 구현체가 아니라, 이를 추**상화 시킨 인터페이스에 의존**하기 때문에 Data Layer에 대한 직접적인 의존성을 회피할 수 있다.

## Data Layer

- Repository Implementation, 1개 이상의 DataSource 포함
- **Domain Layer의 Repository 인터페이스를 구현할 책임이 있다.**
- **Repository는 여러 DataSource를 결합 및 조정한다.** 예를 들어, 로컬 데이터 소스와 원격 데이터 소스를 결합할 수 있다.
- 단, Data Layer의 Repository도 DataSource의 구현체에 직접적으로 의존하지 않는다. DIP의 적용으로, **DataSource의 구현체가 아닌 인터페이스에 의존한다.** 이를 통해 DataSource가 변경되어도 Repository에 미치는 영향은 최소화 할 수 있다.
- **Repository 패턴**: 데이터 소스와 무관하게 동일한 인터페이스로 데이터를 사용할 수 있는 패턴
- **Data Layer는 Domain Layer에 의존한다.**

# 제어의 흐름

<img width="500" src="https://github.com/user-attachments/assets/69f25aec-8a25-4989-83e2-00f638e9c82f"/>

위의 그림은 **제어의 흐름** (Flow of control)을 나타내고 있는데, 계층 간의 경계를 횡단하는 방법을 보여준다. 

안드로이드에 대응시켜 이해해보면, 시스템 이벤트 또는 사용자 입력을 Controller로 받아서 UseCase에서 로직을 처리한 뒤에 그 결과를 UI에 렌더링 하는 것이라고 이해할 수 있다. 

<img width="800" src="https://github.com/user-attachments/assets/14f7d7a4-ac44-422e-b122-3f8372c7b10f"/>

예를 들어, UseCase에서 Presenter와 같이 하위 계층의 코드를 직접 참조하여 호출하면, 클린 아키텍처의 의존성 규칙을 위반하게 된다. 

이렇게 제어의 흐름 (Domain → Presentation) 과 의존성 방향 (Presentation → Domain) 이 반대가 되는 경우에는 인터페이스와 함께 의존성 역전 법칙을 적용하면 된다. 

계층을 횡단하면서 데이터를 전달할 때, 데이터는 항상 내부 원에서 사용하기 가장 편리한 형태를 가져야만 한다. 

예를 들어, 서버에 저장된 사용자 정보를 갱신하기 위해 다음과 같은 로직을 생각해보자.

- 사용자 정보를 갱신하기 위해 API를 호출한다.
- Domain 레이어에서 사용자 정보는 User라는 타입으로 표현한다.
- Presentation 레이어에서 View에 사용자 정보를 나타낼 때는 UserUiModel로 표현한다.
- Data 레이어에서 서버로부터 응답을 받은 사용자 정보는 UserDto로 표현한다.

사용자 정보 갱신 API 호출을 위해 액티비티에서 유즈케이스를 호출할 때, UserUiModel 타입은 User 타입으로 변환해야 한다. 

레트로핏을 통해 API 요청에 따른 응답 UserDto를 받고, 이를 다시 유즈케이스로 넘기기 위해서는 User 타입으로 변환해야 한다. 

흐름이 반대 방향일 때도 마찬가지로 타입을 변환해줘야 한다. 

> UserUiModel ↔ User ↔ UserDto

이렇게 계층을 횡단할 때 데이터의 형태가 바뀔 수 있으며, 이 과정에서 의존성 규칙을 위배해서는 안 된다. 일반적으로 데이터 형태를 변경할 때는 Mapper라는 객체를 임의로 만들어서 사용한다. 

# 참고자료

https://charlezz.com/?p=45391

https://velog.io/@sery270/Android의-Clean-Architecture에-대해-알아보자-n9ihbaj4

https://leveloper.tistory.com/205