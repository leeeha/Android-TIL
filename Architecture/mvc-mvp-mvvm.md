# 아키텍처 패턴

안드로이드 앱 개발 시 사용할 수 있는 여러 아키텍처 패턴들이 있다. 

- MVC (Model-View-Controller)
- MVP (Model-View-Presenter)
- MVVM (Model-View-ViewModel)
- MVI (Model-View-Intent)
- 등등

> Model : 어플리케이션에 사용되는 데이터, 비즈니스 로직
> <br>
> View : 모델을 화면에 표시하는 방법, 즉 UI 자체

프로그램의 UI 로직, 비즈니스 로직을 구현하는 데 있어서 데이터와 UI는 필수적이기 때문에, **M-V 사이의 의존성**이 생길 수밖에 없다. 그런데, 앱의 규모가 커지고 로직이 복잡해짐에 따라 둘 사이의 의존성은 더욱 강해지고, 결국 앱을 유지보수하기 어려워진다. 

이런 문제를 해결하기 위해 여러 패턴들이 나왔는데, 결국 **M-V 사이의 관계를 어떻게 처리하느냐**에 따라 패턴들을 구분지을 수 있다. 

# MVC

> **프로그램을 각각의 역할에 따라 Model, View, Controller로 나누어 설계한 아키텍처 패턴**

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/7e407394-8103-4c38-9e32-254a0a7bab87"/>

- **모든 입력은 Controller로 전달된다.**
- Controller는 입력에 해당하는 **Model을 업데이트** 한다.
- Controller는 Model의 업데이트 결과에 따라 **View를 선택**한다.
  -  Controller와 View는 1:n 관계 
- **Controller는 View를 선택만 할 뿐, 직접 업데이트 하지 않는다.**
    - View는 Controller를 알지 못한다.
- **그렇다면, View를 업데이트 하는 방법은?**
    - View가 Controller를 통해 Model을 가져와서 업데이트
    - Model이 View한테 notify 해서 업데이트
    - View가 주기적으로 Model의 변화를 감지해서 업데이트 (polling)

위의 그림을 반시계 방향으로 90도 회전하면 다음 그림과 같다. 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/4713ba4b-a3e5-4e5e-b698-8a337c0211e1"/>

MVC 구조를 안드로이드에 적용해보면 **Activity는 Controller를 맡게 된다.** 클릭 리스너 등을 통해 사용자의 입력을 받을 수 있고, 그에 따라 모델을 변경시킬 수 있기 때문이다. 

그리고 **xml이 View의 역할**을 맡게 된다. 이때, View가 Model을 알 수 있는 방법은 중간에서 Controller(Activity)가 중재하지 않는 이상은 어렵다. 

MVC 패턴은 시간이 지나면서 다양하게 해석되었는데, 초기 MVC는 View와 Model이 연결되어 있지만, 이후에는 직접 연결되지 않는다. **변화한 형태에서는 Controller가 View를 직접 다루게 된다.** 이 방식으로는 안드로이드에서도 MVC 패턴을 구현할 수 있다. 다만 이 형태가 MVC가 맞는지에 대한 논의도 많다고 한다. 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/52d2ef26-a952-4578-82b2-2afd02287ec6"/>

### 장점

- 코드의 역할이 나눠지고 관심사의 분리가 시작되었음.
- 구현하기 가장 단순한 패턴

### 단점

- Model, View 사이의 의존성 발생
- Controller가 많은 역할을 수행하면서 코드가 비대해짐.
- 앱이 커지고 복잡해질수록 유지보수가 어려움.

# MVP

> **Model과 View 간의 의존성이 없는 아키텍처 패턴**

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/87d22ee0-ffdd-4387-9deb-b53d8c7c3b46"/>

- **모든 입력은 View로 전달된다.**
- Presenter는 입력에 해당하는 **Model을 업데이트**한다.
- Presenter는 Model 업데이트 결과를 기반으로 **View를 업데이트**한다.
  - **View와 Presenter는 1:1 관계** 
- Presenter는 View와 Model의 인스턴스를 갖고, **Model과 View 사이의 중개자 역할**을 한다.

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/82d58d81-ed65-405f-afd7-df17c2296d6d"/>

MVC와 다르게, MVP에서는 유저의 입력을 View가 받는다. 따라서, **Activity를 Controller가 아닌 View 취급**을 할 수 있게 된다. 

MVP 패턴에서는 Presenter가 M-V 사이에서 관리해주기 때문에 **M-V 사이의 의존성이 없다.** 하지만, 역시나 앱이 커지고 복잡해짐에 따라 **V-P 간의 의존성이 강해지는 문제**가 발생한다. 

### 장점

- Model, View 사이의 의존성이 없다.

### 단점

- View, Presenter가 1:1 관계여서 서로 간의 의존성이 커짐. (인터페이스를 통해 그나마 느슨한 연결 유지)
- View, Presenter의 1:1 관계를 유지하기 위해 View마다 Presenter를 추가하면서, 필요한 클래스 개수가 늘어남.

# MVVM

> Model과 View 간의 의존성 뿐만 아니라 **Controller, View 사이의 의존성도 고려**하여 **각 구성요소가 독립적으로 작성되고 테스트 될 수 있도록** 설계된 아키텍처 패턴

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/4c931ffc-a51f-4539-9eba-82b75baae306"/>

- **모든 입력은 View로 전달**된다.
- ViewModel은 입력에 해당하는 UI 로직을 처리하여 View에 데이터를 전달한다.
- **ViewModel은 View를 직접 참조하지 않는다.**
  - View와 ViewModel은 1:n 관계
- **Model이 변경되면 해당 ViewModel을 이용하는 View가 자동으로 업데이트 된다.**
- ViewModel은 View를 나타내기 위한 Model이자, View의 UI 로직을 처리한다.

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/4ec520ed-3c11-4567-be4c-ab82c8dbb819"/>

안드로이드에서는 옵저버 패턴, 데이터 바인딩 등을 통해 **ViewModel이 View를 직접 참조하지 않아도** View가 ViewModel의 상태값을 가져올 수 있다. 즉, LiveData, Flow 등을 통해 ViewModel은 Model을 업데이트 하고, **View가 이 데이터를 구독하고 있다.** 

MVVM 패턴은 MVP와 마찬가지로 **M-V 사이의 의존성이 없고**, MVP처럼 **V-VM이 1:1 관계가 아닌 독립적인 관계**이기 때문에 이 둘 사이의 의존성도 없다. 

### 장점

- Model, View 사이의 의존성이 없음.
- ViewModel이 View를 모르기 때문에 의존성이 분리되고, View 교체가 쉬워짐.
- ViewModel과 View가 1:1 관계가 아니므로, ViewModel을 잘 나누면 여러 뷰에서 재활용 가능 (단, 최근 안드로이드 가이드에 따르면 뷰 단위의 뷰모델 재활용은 권장되지 않으므로 주의 필요)

### 단점

- MVC, MVP 패턴에 비해 구현 난이도가 있음.
- ViewModel 설계가 어려울 수 있음.

# 참고자료

https://velog.io/@haero_kim/Android-깔쌈하게-MVVM-패턴과-AAC-알아보기

https://brunch.co.kr/@oemilk/113

https://medium.com/delightroom/mvc-mvp-mvvm-mvi-아키텍쳐-안드로이드에서-이해하기-1-2442a4189c79

https://brunch.co.kr/@mystoryg/170

https://brunch.co.kr/@mystoryg/171

https://brunch.co.kr/@mystoryg/175

https://brunch.co.kr/@mystoryg/211