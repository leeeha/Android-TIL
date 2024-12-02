## 기존 MVVM 패턴의 장단점

- 장점
    - UI 로직과 비즈니스 로직의 분리 (View, Model 사이의 의존성 제거)
    - View는 ViewModel의 데이터를 구독하고, 상태 변화를 감지하여 UI 갱신 (반응형 프로그래밍)
- 단점
    - 요구사항이 늘어남에 따라 **상태 관리가 복잡**해짐.
    - **스레드 안정성의 한계** (여러 스레드가 동시에 상태를 변경하려고 시도)
    - **양방향 데이터 바인딩의 한계**
        - ViewModel에서 데이터 바인딩 & View 내부에서 스스로 바인딩하는 코드가 섞임.
        - View-ViewModel 간의 양방향 데이터 전달로 인해 디버깅이 어려워짐.

## MVI 패턴이란?

MVI 패턴은 **단일 상태 관리와 단방향 데이터 흐름**을 통해, MVVM의 문제점을 해결하고자 했다. View에서 ViewModel을 직접 참조하기 보다는, **Intent라는 개념을 도입하여 V-VM 사이의 결합도를 감소**시킨다. 다시 말해, **UI는 이벤트를 던지고 Model로부터 상태를 받기만 하면 된다.** 이처럼 상태 관리에만 집중하면 된다는 점에서 Compose와도 궁합이 잘 맞는 패턴이다. 

- `Model`: **현재 앱의 상태를 나타내는 불변 객체**
- `View`: **사용자와 상호작용하는 UI 그 자체** (View, Activity, Fragment, Compose 등)
- `Intent`: **사용자 액션이나 시스템 이벤트가 발생했을 때, 앱의 상태를 어떻게 바꿀 것인지에 대한 의도**

<img width="500" src="https://github.com/user-attachments/assets/e2b133f3-c942-4156-a0bc-8ed020410f08"/> 

<img width="500" src="https://github.com/user-attachments/assets/fd604b1a-80b7-4937-b890-b8df6e7fec74"/> 

MVI 패턴은 위의 그림처럼 **단방향 순환 구조**를 갖고 있으며, **앱의 상태는 불변 객체**이다. 따라서, **Intent에 의해 새로운 Model이 생성될 때만 기존 상태를 변경**할 수 있다. (주로 copy 함수 사용) 

이러한 단방향 데이터 흐름(Unidirectional Data Flow, UDF) 덕분에 개발자는 **상태의 변경을 더 쉽게 예측**할 수 있고, 디버깅도 더 수월해진다. 

이제까지 MVVM 패턴을 사용해왔다면, MVI는 완전히 새로운 것이 아니다. 아래 그림과 같이 View Model은 상태를 저장하고 관리하는 좋은 장소가 될 수 있다. 

<img width="700" src="https://github.com/user-attachments/assets/b0e938dc-c84a-4045-8156-0807c3bfc23f"/> 

## 장단점

- 장점
    - **상태 관리 용이 → 디버깅 및 테스트 편의성 증가**
    - **단방향 데이터 흐름**
    - 스레드 안정성 보장 (외부에서 상태 변경할 수 없도록 보장)
- 단점
    - 보일러플레이트 코드 양산
    - 작은 변경도 Intent로 처리해야 하는 번거로움.
    - State, SideEffect 등을 구현하기 위해 필요한 파일 개수가 늘어남.

## 부수 효과

<img width="500" src="https://github.com/user-attachments/assets/308b474e-2c94-494b-8e64-ed4a2339984d"/>

앱에서는 **상태를 변경하지 않는 이벤트**도 발생할 수 있다. 예를 들어, 화면 전환, 스낵바 표시, 이벤트 로깅과 같은 작업들이 대표적이다. MVI 패턴에서는 이를 **Side Effect로 처리**한다. 

## 실제 코드 예시

// todo 

## 참고자료

- https://charlezz.com/?p=46365
- https://munseong.dev/android/mvi_architecture/
- https://kennethss.medium.com/android-mvi%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-uda-b16f116c7e34

