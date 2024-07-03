# 테스트 코드의 필요성

[러넥트](https://github.com/Runnect/Runnect-Android)라는 프로젝트에서 상당히 큰 규모의 리팩토링을 몇번 했었는데, 리팩토링 하기 전후에 **외부가 그대로인지 객관적으로 측정할 수 있는 지표가 없었다.** 

[위니](https://github.com/team-winey/Winey-AOS)라는 프로젝트에서 닉네임 유효성 검증을 할 때도 **매번 코드를 직접 실행시켜야만 의도한 대로 작동하는지 확인할 수 있어서 생산성이 매우 떨어진다**고 느꼈다. 

그리고 개인적으로 QA를 꼼꼼히 진행하더라도 **예상치 못한 예외가 발생**할 수도 있다는 불안감이 들었다.

다양한 시나리오에 대한 테스트를 통해 **앱의 완성도와 견고함**을 높이고, **내가 작성한 코드에 대한 즉각적인 피드백**을 받기 위해 테스트 코드 작성을 시작할 때이다!

# 안드로이드 공식문서

## 테스트의 이점

- 앱 개발 프로세스의 필수적인 부분
- 앱을 공개적으로 출시하기 전에 앱의 정확성, 기능적 동작, 사용성 확인 가능
- 수동 테스트
    - 다양한 디바이스와 에뮬레이터 사용 가능
    - 시스템 언어 변경 가능
    - 모든 사용자 에러 생성 및 사용자 플로우 추적 가능
    - 확장성이 떨어짐.
- 자동화 테스트
    - 테스트를 대신 수행하는 도구 사용
    - 더 빠르고 반복 가능
    - 앱에 대한 실행 가능한 피드백 제공

## 테스트의 종류

모바일 앱은 다양하고 복잡한 환경에서 잘 작동해야 한다. 따라서 다양한 유형의 테스트가 있다. 

### 테스트 대상에 따른 분류

- **기능 테스트** : 앱이 의도한 대로 작동하는가?
- **성능 테스트** : 앱이 빠르고 효율적으로 작동하는가?
- **접근성 테스트** : 앱이 접근성 서비스와 함께 잘 작동하는가?
- **호환성 테스트** : 모든 디바이스와 API 레벨에서 잘 작동하는가?

### 테스트 범위에 따른 분류

- **Unit test (small test)** : 메서드나 클래스와 같이 앱의 매우 작은 부분에 대한 검증
- **End-to-end test (big test)** : 전체 화면이나 사용자 플로우 같이 앱의 더 큰 부분을 동시에 확인 가능
- **Intergration test (medium test)** : 두 개 이상의 단위 간의 통합을 확인할 때 사용 

<img width="600" src="https://github.com/leeeha/Android-TIL/assets/68090939/28569ca5-300c-4bef-8c0c-3eeeda7db6b1"/>

테스트를 분류하는 방법은 여러가지가 있는데, 앱 개발자에게 가장 중요한 구분은 **테스트가 실행되는 위치**이다. 

## 계측 테스트 vs 로컬 테스트

테스트는 안드로이드 디바이스 또는 호스트 PC에서 실행시킬 수 있다. 전자를 계측 테스트, 후자를 로컬 테스트라고 부른다. 

<img width="600" src="https://github.com/leeeha/Android-TIL/assets/68090939/2772dad1-3f4f-4fa9-a4f1-cedcabdb1632"/>

### Instrumented test

- 실제 기기 또는 에뮬레이터에서 실행
- 보통 앱을 실행한 다음에 앱과 상호작용하는 UI 테스트에 해당 

```kotlin
// 버튼 클릭
onView(withText("Continue"))
		.perform(click())

// 웰컴 화면 표시
onView(withText("Welcome"))
		.check(matches(isDisplayed()))
```

### Local test (Host-side test)

- 개발 컴퓨터 또는 서버에서 실행
- 규모가 작고 빠르며 테스트 대상과 나머지 앱을 격리함.

```kotlin
val viewModel = MyViewModel(myFakeDataRepository)
viewModel.loadData()

// 데이터를 로드하고나서 제대로 노출되는지 확인 
assertTrue(viewModel.data != null)
```

모든 단위 테스트가 로컬에 있는 건 아니며, 반대로 모든 End-to-end 테스트가 기기에서 실행되는 건 아니다. 

- 대규모 로컬 테스트: 로컬에서 실행되는 안드로이드 시뮬레이터 사용 가능
- 소규모 계측 테스트: 코드가 SQLite 데이터베이스 같은 프레임워크에서 잘 작동하는지 확인 가능, 여러 기기에서 테스트 실행하여 여러 버전의 프레임워크와의 통합 확인

## 테스트 전략 수립

좋은 테스트 전략은 **테스트의 충실도, 속도, 신뢰성** 사이에서 적절한 균형을 찾는 것이다. 

**테스트 환경과 실제 디바이스의 유사성에 따라 테스트의 충실도가 결정된다.** 

- 충실도 ⬆️ : 에뮬레이터 또는 실제 기기에서 실행 (속도가 느리고 더 많은 리소스가 필요함)
- 충실도 ⬇️ : 로컬 컴퓨터의 JVM에서 실행

올바르게 설계하고 구현한 테스트에서도 오류가 발생할 수 있다. 예를 들어, 실제 기기에서 테스트를 실행할 때 테스트 도중에 자동 업데이트가 시작되어 테스트에 실패할 수 있다. 코드에서 미묘한 경쟁 조건이 일부만 발생할 수도 있다. **100% 통과하지 못하는 테스트는 불안정한 테스트**다. 

## 테스트 가능한 아키텍처

테스트 가능한 아키텍처를 사용하면 **코드의 여러 부분을 개별적으로 쉽게 테스트 할 수 있는 구조**를 따르게 된다. 

테스트 가능한 아키텍처는 **가독성, 유지보수성, 확장성, 재사용성 증가**와 같은 장점도 지닌다. 

테스트 할 수 없는 아키텍처에서는 다음과 같은 문제가 발생할 수 있다. 

- 더 크고, 느리고, 불안정한 테스트. 단위 테스트가 불가능한 클래스는 더 큰 통합 테스트나 UI 테스트로 커버해야 할 수 있다.
- 다양한 시나리오를 테스트 할 수 있는 기회가 줄어든다. 테스트 규모가 클수록 속도가 느려지므로 앱의 모든 가능한 상태를 테스트 하는 것은 비현실적이다.

## Decoupling

**함수, 클래스, 모듈의 일부를 나머지 부분에서 추출할 수 있다면 테스트가 더 쉽고 효과적이다.** 이를 결합도(디커플링)이라고 하며, 테스트 가능한 아키텍처에서 가장 중요한 개념이다. 

일반적인 디커플링 기법은 다음과 같다. 

- 앱을 **Presentation, Domain, Data 레이어로 분리**한다. 앱을 기능별로 하나씩 모듈로 분리할 수도 있다.
- 액티비티, 프래그먼트와 같이 **종속성이 큰 엔터티에는 로직을 추가하지 않는다.** 이러한 클래스는 프레임워크의 진입점으로 사용하고, UI 및 비즈니스 로직은 컴포저블, 뷰모델, 도메인 레이어 같은 곳으로 이동시킨다.
- **비즈니스 로직을 포함하는 클래스가 직접적으로 프레임워크에 종속되는 것을 피한다.** 예를 들어, 뷰모델에서 안드로이드 컨텍스트를 사용하지 않는다.
- **의존성을 쉽게 교체할 수 있도록 설계한다.** 예를 들어, 구체적인 구현 대신에 **인터페이스를 사용**한다. DI 프레임워크를 사용하지 않더라도 **의존성 주입**을 사용한다.

# 참고자료

https://developer.android.com/training/testing?hl=ko