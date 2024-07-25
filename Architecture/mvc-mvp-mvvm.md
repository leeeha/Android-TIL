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

프로그램의 UI 로직, 비즈니스 로직을 구현하는 데 있어서 데이터와 UI는 필수적이기 때문에, **M-V 사이의 의존성**이 생길 수밖에 없다. 

그런데, 앱의 규모가 커지고 로직이 복잡해짐에 따라 둘 사이의 의존성은 더욱 강해지고, 결국 앱을 유지보수하기 어려워진다. 

이런 문제를 해결하기 위해 여러 패턴들이 나왔는데, 결국 **M-V 사이의 관계를 어떻게 처리하느냐**에 따라 패턴들을 구분지을 수 있다. 

# MVC

>프로그램을 **각각의 역할에 따라 Model, View, Controller로 나누어 설계**한 아키텍처 패턴

<img width="600" src="https://github.com/user-attachments/assets/a8792bd3-2d6b-417b-ac2f-e30b1f076409">

- **모든 입력은 Controller로 전달된다.**
- Controller는 입력에 해당하는 **Model을 업데이트** 한다.
- Controller는 Model의 업데이트 결과에 따라 **View를 선택**한다. (Controller와 View는 1:n 관계)
- Controller는 View를 선택만 할 뿐, 직접 업데이트 하지 않는다. View는 Controller를 알지 못한다.
- **그렇다면, View를 업데이트 하는 방법은?**
    - Model 스스로 자신의 변화를 View에게 알린다. (notify)
    - View가 주기적으로 Model을 가져와 업데이트한다. (polling)

## 안드로이드에서는? 

<img width="600" src="https://github.com/leeeha/Android-TIL/assets/68090939/4713ba4b-a3e5-4e5e-b698-8a337c0211e1"/>

MVC 구조를 안드로이드에 적용해보면 **Activity는 Controller를 맡게 된다.** 클릭 리스너 등을 통해 사용자의 입력을 받을 수 있고, 그에 따라 모델을 변경시킬 수 있기 때문이다. 

그리고 **xml이 View의 역할**을 맡게 된다. 이때, View가 Model을 알 수 있는 방법은 중간에서 Controller(Activity)가 중재하지 않는 이상은 어렵다. 

MVC 패턴은 시간이 지나면서 다양하게 해석되었는데, 초기 MVC는 View와 Model이 연결되어 있지만, 이후에는 직접 연결되지 않는다. 

**변화한 형태에서는 Controller가 View를 직접 다루게 된다.** 이 방식으로는 안드로이드에서도 MVC 패턴을 구현할 수 있다. (다만 이 형태가 MVC가 맞는지에 대한 논의도 많다고 한다.)

<img width="900" src="https://github.com/leeeha/Android-TIL/assets/68090939/52d2ef26-a952-4578-82b2-2afd02287ec6"/>

## 예시 코드 

```kotlin
class OrderActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_order)

        // Model 초기화 
        val americanoModel = Americano()
        val totalPriceModel = TotalPrice()

        // View 초기화 
        val americanoDeleteButton = findViewById<Button>(R.id.americanoDeleteButton)
        val americanoAddButton = findViewById<Button>(R.id.americanoAddButton)
        val americanoCountText = findViewById<TextView>(R.id.americanoCountText)
        val totalPriceText = findViewById<TextView>(R.id.totalPriceText)

        // Controller로 들어온 사용자 입력에 따라 Model, View 갱신 
        americanoDeleteButton.setOnClickListener {
            americanoModel.delete()
            americanoCountText.text = "${americanoModel.quantity}"

            totalPriceModel.decreaseTotalPrice(americanoModel.price)
            totalPriceText.text = "${totalPriceModel.totalPrice}"
        }

        americanoAddButton.setOnClickListener {
            americanoModel.add()
            americanoCountText.text = "${americanoModel.quantity}"

            totalPriceModel.increaseTotalPrice(americanoModel.price)
            totalPriceText.text = "${totalPriceModel.totalPrice}"
        }
    }
}
```

### 장점 

- 코드의 역할이 나눠지고 관심사의 분리가 시작되었음.
- 구현하기 가장 단순한 패턴

### 단점

- Model, View 사이의 의존성 발생
- Controller가 많은 역할을 수행하면서 코드가 비대해짐.
- 앱이 커지고 복잡해질수록 유지보수가 어려움.

# MVP

> **Model과 View 간의 의존성이 없는 아키텍처 패턴**

<img width="700" alt="스크린샷 2024-07-19 오후 3 38 20" src="https://github.com/user-attachments/assets/a464dba9-7e88-4a35-8b59-cd8c72498f2f">

- **모든 입력은 View로 전달된다.**
- Presenter는 입력에 해당하는 **Model을 업데이트**한다.
- Presenter는 Model 업데이트 결과를 기반으로 **View를 업데이트**한다.
- **View와 Presenter는 1:1 관계** 
- Presenter는 View와 Model의 인스턴스를 갖고, **Model과 View 사이의 중개자 역할**을 한다.

## 안드로이드에서는? 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/82d58d81-ed65-405f-afd7-df17c2296d6d"/>

MVC와 다르게, MVP에서는 유저의 입력을 View가 받는다. 따라서, **Activity를 Controller가 아닌 View 취급**을 할 수 있게 된다. 

MVP 패턴에서는 Presenter가 M-V 사이에서 관리해주기 때문에 **M-V 사이의 의존성이 없다.**

하지만, 역시나 앱이 커지고 복잡해짐에 따라 **V-P 간의 의존성이 강해지는 문제**가 발생한다. 

## 예시 코드 

```kotlin
interface Presenter {
    fun addAmericano()
    fun deleteAmericano()
}

class OrderPresenter(
    private var view: OrderView,
    private var menuModel: Beverage = Americano(),
    private var totalModel: TotalPrice = TotalPrice()
) : Presenter {
    override fun deleteAmericano() {
        menuModel.delete()
        view.setAmericanoCounterText(menuModel.quantity)
        updateTotalPriceSubtraction(menuModel.price)
    }

    override fun addAmericano() {
        menuModel.add()
        view.setAmericanoCounterText(menuModel.quantity)
        updateTotalPriceAddition(menuModel.price)
    }

    private fun updateTotalPriceSubtraction(price: Int) {
        totalModel.decreaseTotalPrice(price)
        view.setTotalPriceText(totalModel.totalPrice)
    }

    private fun updateTotalPriceAddition(price: Int) {
        totalModel.increaseTotalPrice(price)
        view.setTotalPriceText(totalModel.totalPrice)
    }
}
```

```kotlin
interface OrderView {
    fun setAmericanoCounterText(count: Int)
    fun setTotalPriceText(price: Int)
}

class OrderActivity : AppCompatActivity(), OrderView {
    private lateinit var americanoCountText: TextView
    private lateinit var totalPriceText: TextView
    private var present: OrderPresenter = OrderPresenter(this)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_order)

        val americanoDeleteButton = findViewById<Button>(R.id.americanoDeleteButton)
        val americanoAddButton = findViewById<Button>(R.id.americanoAddButton)

        americanoCountText = findViewById<TextView>(R.id.americanoCountText)
        totalPriceText = findViewById<TextView>(R.id.totalPriceText)

        // Presenter를 통해 Model, View 갱신 
        americanoDeleteButton.setOnClickListener {
            present.deleteAmericano()
        }

        americanoAddButton.setOnClickListener {
            present.addAmericano()
        }
    }

    override fun setAmericanoCounterText(count: Int) {
        americanoCountText.text = "$count"
    }

    override fun setTotalPriceText(price: Int) {
        totalPriceText.text = "$price"
    }
}
```

### 장점

- Model, View 사이의 의존성이 없다.

### 단점

- View, Presenter가 1:1 관계여서 서로 간의 의존성이 커짐. (인터페이스를 통해 그나마 느슨한 연결 유지)
- View, Presenter의 1:1 관계를 유지하기 위해 View마다 Presenter를 추가하면서, 필요한 클래스 개수가 늘어남.

# MVVM

>Model과 View 간의 의존성 뿐만 아니라 **Controller, View 사이의 의존성도 고려**하여 <br>
>각 구성요소가 **독립적으로 작성되고 테스트** 될 수 있도록 설계된 아키텍처 패턴

<img width="700" alt="스크린샷 2024-07-25 오후 10 17 09" src="https://github.com/user-attachments/assets/784675fa-11b1-43fd-ae96-886084cd35c5">

- **모든 입력은 View로 전달**된다.
- ViewModel은 입력에 해당하는 **Model을 업데이트**하고, View에게 상태 변화를 알린다. (단, View를 직접 참조 X)
- **View는 자신이 사용할 ViewModel을 선택**하여, 업데이트 된 Model을 받는다. (ViewModel과 View는 1:n 관계)

## 안드로이드에서는? 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/4ec520ed-3c11-4567-be4c-ab82c8dbb819"/>

안드로이드에서는 옵저버 패턴, 데이터 바인딩 등을 통해 **ViewModel이 View를 직접 참조하지 않아도** View가 ViewModel의 상태 값을 가져올 수 있다. 

즉, LiveData, Flow 등을 통해 **ViewModel은 Model을 업데이트** 하고, **View가 이 데이터를 구독하고 있다.**

MVVM 패턴은 MVP와 마찬가지로 **M-V 사이의 의존성이 없고**, MVP처럼 **V-VM이 1:1 관계가 아닌 독립적인 관계**이기 때문에 이 둘 사이의 의존성도 없다. 

## MVVM ViewModel vs AAC ViewModel

### MVVM ViewModel

MVVM 패턴에서의 ViewModel은 **사용자 입력에 따라 Model의 상태가 변하면, View에게 이를 알리는 역할**을 한다.

View는 ViewModel의 상태 변화를 관찰하고 있다. 옵저버 패턴을 사용하기 때문에 **View에서는 ViewModel을 알고 있지만, ViewModel은 View를 알지 못한다.**

일반적으로 **ViewModel과 View는 1:n의 관계**이며, **View는 자신이 이용할 ViewModel을 선택**하여 상태 변화 알림을 받게 된다. 

ViewModel은 View가 쉽게 사용할 수 있도록 Model의 데이터를 가공하여 View에게 제공한다. (직접 View를 참조하지는 않는다.)

한마디로 요약하자면 **ViewModel이란 View와 Model 사이에서 데이터를 관리하고 바인딩해주는 요소**이다.

### AAC ViewModel

AAC에서의 ViewModel은 Android 컴포넌트의 생명 주기를 고려하여 UI 관련 데이터를 저장하고 관리하도록 설계되었다. AAC ViewModel을 사용하면 기존의 Activity가 생명 주기 때문에 데이터 관리 측면에서 겪던 어려움들을 간단하게 처리할 수 있다.

![image](https://github.com/user-attachments/assets/aec5fe28-5a8f-4142-a2af-b8e64c44ddaf)

위의 그림처럼 ViewModel은 Activity보다 생명 주기가 길기 때문에, **화면 회전과 같은 Configuration Change 상황에서 Activity가 파괴되었다가 재생성되어도 기존 데이터를 보존**할 수 있다. 

### 요약 

둘은 서로 다른 개념이지만, AAC의 ViewModel을 이용하여 MVVM 패턴의 ViewModel을 구현할 수 있다. 

MVVM 패턴에서 ViewModel의 역할은 View에 필요한 데이터를 관리하고 바인딩 해주는 것이다. AAC의 ViewModel을 이 개념대로 구현해주면 된다.

즉, ViewModel 내에서 LiveData, Flow 등을 사용하여 View에 데이터를 바인딩한다면 MVVM 패턴의 ViewModel로써 사용 가능한 것이다. 

## 예시 코드 

```kotlin
@HiltViewModel
class RecommendViewModel @Inject constructor(
    private val recommendRepository: RecommendRepository
) : ViewModel() {
    private val _getRecommendListState =
        MutableStateFlow<UiState<List<Recommend>?>>(UiState.Loading)

    val getRecommendListState: StateFlow<UiState<List<Recommend>?>> =
        _getRecommendListState.asStateFlow()

    init {
        getRecommendList()
    }

    private fun getRecommendList() {
        viewModelScope.launch {
            recommendRepository.getRecommendList(page = 1)
                .onSuccess { response ->
                    _getRecommendListState.value = UiState.Success(response)
                }
                .onFailure { t ->
                    _getRecommendListState.value = UiState.Failure("${t.message}")
                    Timber.e("${t.message}")
                }
        }
    }
}
```

```kotlin
@AndroidEntryPoint
class RecommendFragment : BindingFragment<FragmentRecommendBinding>(R.layout.fragment_recommend) {
    private val viewModel by viewModels<RecommendViewModel>()
    private lateinit var recommendAdapter: RecommendAdapter
    private lateinit var recommendHeaderAdapter: RecommendHeaderAdapter

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding.vm = mainViewModel

        initAdapter()
        getRecommendListStateObserver()

        // ...
    }

    private fun initAdapter() {
        recommendAdapter = RecommendAdapter(
            onItemLinkClicked = { id, link ->
                sendEventToAmplitude(id)
                displayWebsite(link)
            }
        )
        recommendHeaderAdapter = RecommendHeaderAdapter()
        binding.rvRecommendPost.adapter =
            ConcatAdapter(recommendHeaderAdapter, recommendAdapter)
    }

    private fun getRecommendListStateObserver() {
        viewModel.getRecommendListState.flowWithLifecycle(viewLifeCycle)
          .onEach { state ->
              when (state) {
                  is UiState.Success -> {
                      recommendAdapter.submitList(state.data)
                  }

                  is UiState.Failure -> {
                      snackBar(binding.root) { state.msg }
                  }

                  else -> {}
              }
          }.launchIn(lifecycleScope)
    }
    
    // ...

    companion object {
        @JvmStatic
        fun newInstance() = RecommendFragment()
    }
}
```


### 장점

- Model, View 사이의 의존성이 없음.
- ViewModel이 View를 모르기 때문에 의존성이 분리되고, View 교체가 쉬워짐.
- ViewModel과 View가 1:1 관계가 아니므로, ViewModel을 잘 나누면 여러 뷰에서 재활용 가능 (단, 뷰 단위의 뷰모델 재활용은 권장되지 않으므로 주의 필요)

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