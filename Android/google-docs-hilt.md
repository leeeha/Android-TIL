## Hilt 어플리케이션 클래스

Hilt를 사용하는 모든 앱은 `@HiltAndroidApp`으로 어노테이션이 지정된 `Application` 클래스를 포함해야 한다. 

`@HiltAndroidApp`은 **어플리케이션 수준의 의존성 컨테이너 역할**을 하는 어플리케이션의 기본 클래스를 비롯하여 **Hilt의 코드 생성을 트리거**한다. 

```kotlin
@HiltAndroidApp
class ExampleApplication : Application() { ... }
```

생성된 이 Hilt 컴포넌트는 `Application` 객체의 생명주기에 연결되며 이와 관련한 의존성을 제공한다. 

또한 이는 앱의 상위 컴포넌트이므로 **다른 컴포넌트는 이 상위 컴포넌트에서 제공하는 의존성에 액세스할 수 있다.**

## **Android 클래스에 의존성 주입**

`Application` 클래스에 Hilt를 설정하고 어플리케이션 수준의 컴포넌트를 사용할 수 있게 되면, Hilt는 `@AndroidEntryPoint` 어노테이션이 있는 다른 Android 클래스에 의존성을 제공할 수 있다.

```kotlin
@AndroidEntryPoint
class ExampleActivity : AppCompatActivity() { ... }
```

Hilt는 현재 다음 Android 클래스를 지원한다. 

- `Application` (`@HiltAndroidApp`을 사용하여)
- `ViewModel` (`@HiltViewModel`을 사용하여)
- `Activity`
- `Fragment`
- `View`
- `Service`
- `BroadcastReceiver`

Android 클래스에 `@AndroidEntryPoint`로 어노테이션을 지정하면, **이 클래스에 의존하는 Android 클래스에도 어노테이션을 지정해야 한다.** 예를 들어, 프래그먼트에 어노테이션을 지정하면 이 **프래그먼트를 사용하는 액티비티에도 어노테이션을 지정**해야 한다. 

>🚨 **참고:** Android 클래스에 관한 Hilt 지원에는 다음 예외가 적용된다.
>
>- Hilt는 `AppCompatActivity`와 같은 `ComponentActivity`를 확장하는 액티비티만 지원한다.
>- Hilt는 `androidx.Fragment`를 확장하는 프래그먼트만 지원한다.
>- Hilt는 보존된 프래그먼트(Retained Fragment)를 지원하지 않는다.

`@AndroidEntryPoint`는 **프로젝트의 각 Android 클래스에 대한 개별 Hilt 컴포넌트를 생성**한다. 이러한 컴포넌트는 [컴포넌트 계층 구조](https://developer.android.com/training/dependency-injection/hilt-android#component-hierarchy)에 설명된 대로 **해당하는 각 상위 클래스에서 의존성을 받을 수 있다.** 

컴포넌트에서 의존성을 가져오려면 다음과 같이 `@Inject` 어노테이션을 사용하여 필드 주입을 수행할 수 있다. 

```kotlin
@AndroidEntryPoint
class ExampleActivity : AppCompatActivity() {

  @Inject lateinit var analytics: AnalyticsAdapter 
  ...
}
```

>🚨 **참고:** Hilt가 삽입한 필드는 **private이 불가능**하다. Hilt를 사용하여 private 필드를 삽입하려고 하면 **컴파일 에러가 발생**한다.

Hilt가 주입하는 클래스에는 DI를 사용하는 또 다른 기본 클래스가 있을 수 있다. 이러한 클래스는 추상적인 경우 `@AndroidEntryPoint` 어노테이션이 필요하지 않다.

Android 클래스가 삽입되는 생명주기 콜백에 관한 자세한 내용은 [Component lifetimes](https://developer.android.com/training/dependency-injection/hilt-android?hl=ko#component-lifetimes)를 참고하자. 

## Hilt binding 정의

**필드 주입**을 실행하려면 **Hilt가 해당 컴포넌트에서 필요한 의존성의 인스턴스를 제공하는 방법**을 알아야 한다. 

**바인딩**에는 **특정 타입의 인스턴스를 의존성으로 제공하는 데 필요한 정보**가 포함되어 있다.

Hilt에 바인딩 정보를 제공하는 한 가지 방법은 **생성자 주입**이다. 다음과 같이 클래스의 생성자에서 `@Inject` 어노테이션을 사용하여 **클래스의 인스턴스를 제공하는 방법**을 Hilt에 알려줄 수 있다.

```kotlin
class AnalyticsAdapter @Inject constructor(
  private val service: AnalyticsService
) { ... }
```

`@Inject` 어노테이션이 지정된 **클래스 생성자의 매개변수**는 그 **클래스의 의존성**이다. 위의 예시에서 `AnalyticsAdapter`에는 `AnalyticsService`가 의존성으로 제공된다. 따라서 Hilt는 `AnalyticsService`의 인스턴스를 제공하는 방법도 알아야 한다. 

>🚨 **참고:** 빌드 시간에 Hilt는 Android 클래스용 [Dagger](https://developer.android.com/training/dependency-injection/dagger-basics?hl=ko) 컴포넌트를 생성한다. 그런 다음, Dagger는 코드를 검토하고 다음 단계를 실행한다.
>
>- **의존성 그래프를 빌드하고 그 유효성을 검사**하여, 유효하지 않은 의존성과 의존성 주기가 없도록 한다.
>- 런타임 시 **실제 객체 및 의존성을 만드는 데 필요한 클래스를 생성**한다.

## Hilt Module

때로는 아래와 같이 **생성자 주입이 불가능한 경우**도 있다. 

>- 인터페이스
>- 외부 라이브러리의 클래스

이럴 때는 **Hilt 모듈을 사용하여 Hilt에 바인딩 정보를 제공**할 수 있다. 

Hilt 모듈은 `@Module`로 어노테이션이 지정된 클래스다. [Dagger 모듈](https://developer.android.com/training/dependency-injection/dagger-android?hl=ko#dagger-modules)과 마찬가지로 **Hilt 모듈은 특정 유형의 인스턴스를 제공하는 방법을 Hilt에 알려준다.** 

그러나 Dagger 모듈과는 달리, Hilt 모듈에 `@InstallIn` 어노테이션을 지정하여 **각 모듈을 설치할 Android 클래스를 Hilt에 알려야 한다.** 

Hilt 모듈에 제공하는 의존성은 **Hilt 모듈을 설치하는 Android 클래스와 연결되어 있는 모든 생성된 컴포넌트에서 사용**할 수 있다. 

>🚨 **참고:** Hilt 코드를 생성할 때는 Hilt를 사용하는 모든 Gradle 모듈에 접근할 수 있어야 한다. 다시 말해, `Application` 클래스를 컴파일하는 Gradle 모듈이 모든 Hilt 모듈과 생성자 주입 클래스를 가지고 있어야 Hilt가 정상적으로 관련 코드를 생성할 수 있다.

### @Binds 사용한 인터페이스 인스턴스 주입

앞에서 살펴본 `AnalyticsService` 예시를 다시 보자. 

```kotlin
class AnalyticsAdapter @Inject constructor(
  private val service: AnalyticsService
) { ... }
```

`AnalyticsService`가 인터페이스라면 이 인터페이스를 생성자 주입할 수 없다. 대신 **Hilt 모듈 내에 `@Binds`로 어노테이션이 지정된 추상 함수를 생성하여 Hilt에 바인딩 정보를 제공**할 수 있다. 

`@Binds` 어노테이션은 **인터페이스의 인스턴스를 제공할 때 필요한 인터페이스 구현체**를 Hilt에 알려준다. 

- 함수의 반환 타입: 함수가 어떤 인터페이스의 인스턴스를 제공하는지 Hilt에 알려준다.
- 함수 매개변수: 인터페이스의 구현체를 Hilt에 알려준다.

```kotlin
interface AnalyticsService {
  fun analyticsMethods()
}

class AnalyticsServiceImpl @Inject constructor(
  ...
) : AnalyticsService { ... }

@Module
@InstallIn(ActivityComponent::class)
abstract class AnalyticsModule {

  @Binds
  abstract fun bindAnalyticsService(
    analyticsServiceImpl: AnalyticsServiceImpl
  ): AnalyticsService
}
```

Hilt가 `AnalyticsModule`의 의존성을 `ExampleActivity`에 주입하길 원하기 때문에, Hilt 모듈 `AnalyticsModule`에 `@InstallIn(ActivityComponent.class)` 어노테이션을 지정한다. 이 어노테이션은 `AnalyticsModule`의 모든 의존성을 **앱의 모든 액티비티에서 사용할 수 있음**을 의미한다.

### **@Provides 사용한 인스턴스 주입**

생성자 주입이 불가능한 것은 **인터페이스**만이 아니다. **클래스가 외부 라이브러리에서 제공되어 클래스를 소유하지 않은 경우** (Retrofit, OkHttpClient, Room DB 등) 또는 **빌더 패턴으로 인스턴스를 생성해야 하는 경우**에도 생성자 주입이 불가능하다. 

앞선 예시에서 `AnalyticsService` 클래스를 직접 소유하지 않으면, **Hilt 모듈 내에 함수를 생성하고 이 함수에 `@Provides` 어노테이션을 지정하여 해당 타입의 인스턴스를 제공하는 방법을 Hilt에 알려줘야 한다.** 

- 함수 반환 타입: 함수가 어떤 타입의 인스턴스를 제공하는지 Hilt에 알려준다.
- 함수 매개변수: 해당 타입의 의존성을 Hilt에 알려준다.
- 함수 본문: 해당 타입의 인스턴스를 제공하는 방법을 Hilt에 알려준다. Hilt는 해당 인스턴스를 제공할 때마다 함수 본문을 실행한다.

```kotlin
@Module
@InstallIn(ActivityComponent::class)
object AnalyticsModule {

  @Provides
  fun provideAnalyticsService(
    // Potential dependencies of this type
  ): AnalyticsService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .build()
               .create(AnalyticsService::class.java)
  }
}
```

### 동일한 타입에 대해 여러 바인딩 제공

의존성과 동일한 타입의 다양한 구현을 제공하는 Hilt가 필요한 경우에는 Hilt에 여러 바인딩을 제공해야 한다. 이때, **한정자를 사용하여 동일한 타입에 대해 여러 바인딩을 정의**할 수 있다.

`@Qualifier`는 **특정 타입에 대해 여러 바인딩이 정의되어 있을 때, 해당하는 특정 바인딩을 식별하는 데 사용하는 어노테이션**이다. 

예를 들어 `AnalyticsService` 호출에 대한 응답값을 가로채고 싶을 때, [인터셉터](https://square.github.io/okhttp/interceptors/)와 함께 `OkHttpClient` 객체를 사용할 수 있다. 다른 서비스에서는 호출에 대한 응답값을 다른 방식으로 가로채야 할 수도 있다. 이 경우에는 서로 다른 두 가지 `OkHttpClient` 구현을 제공하는 방법을 Hilt에 알려줘야 한다. 

먼저 다음과 같이 `@Binds` 또는 `@Provides` 메서드에 어노테이션을 지정하는 데 사용할 한정자를 정의한다. 

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthInterceptorOkHttpClient

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class OtherInterceptorOkHttpClient
```

그런 다음, **Hilt는 각 한정자와 일치하는 타입의 인스턴스를 제공하는 방법을 알아야 한다.** 이 경우에는 `@Provides`와 함께 Hilt 모듈을 사용할 수 있다. **두 메서드 모두 동일한 반환 타입을 갖지만, 한정자는 다음과 같이 두 가지의 서로 다른 바인딩으로 메서드에 라벨을 지정한다.** 

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

  @AuthInterceptorOkHttpClient
  @Provides
  fun provideAuthInterceptorOkHttpClient(
    authInterceptor: AuthInterceptor
  ): OkHttpClient {
      return OkHttpClient.Builder()
               .addInterceptor(authInterceptor)
               .build()
  }

  @OtherInterceptorOkHttpClient
  @Provides
  fun provideOtherInterceptorOkHttpClient(
    otherInterceptor: OtherInterceptor
  ): OkHttpClient {
      return OkHttpClient.Builder()
               .addInterceptor(otherInterceptor)
               .build()
  }
}
```

다음과 같이 필드 또는 매개변수에 해당하는 한정자로 어노테이션을 지정하여 필요한 특정 유형을 주입할 수 있다. 

```kotlin
// As a dependency of another class.
@Module
@InstallIn(ActivityComponent::class)
object AnalyticsModule {

  @Provides
  fun provideAnalyticsService(
    @AuthInterceptorOkHttpClient okHttpClient: OkHttpClient
  ): AnalyticsService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .client(okHttpClient)
               .build()
               .create(AnalyticsService::class.java)
  }
}

// As a dependency of a constructor-injected class.
class ExampleServiceImpl @Inject constructor(
  @AuthInterceptorOkHttpClient private val okHttpClient: OkHttpClient
) : ...

// At field injection.
@AndroidEntryPoint
class ExampleActivity: AppCompatActivity() {

  @AuthInterceptorOkHttpClient
  @Inject lateinit var okHttpClient: OkHttpClient
}
```

특정 인스턴스 타입에 한정자를 추가하는 경우, **해당 의존성을 제공할 수 있는 모든 가능한 방법에 한정자를 추가하는 것이 가장 좋다.** 한정자 없이 기본 또는 공통 구현을 그대로 두면 오류가 발생하기 쉬우며 Hilt가 잘못된 의존성을 주입할 수 있다.

### Hilt에 미리 정의된 한정자

Hilt는 몇 가지 미리 정의된 한정자를 제공한다. 예를 들어, 어플리케이션 또는 액티비티의 `Context` 클래스가 필요할 수 있으므로, Hilt는 `@ApplicationContext` 및 `@ActivityContext` 한정자를 제공한다. 

앞선 예시에서 `AnalyticsAdapter` 클래스에 액티비티 컨텍스트가 필요하다고 가정해보자. 다음 코드는 `AnalyticsAdapter`에 액티비티 컨텍스트를 제공하는 방법을 보여준다. 

```kotlin
class AnalyticsAdapter @Inject constructor(
    @ActivityContext private val context: Context,
    private val service: AnalyticsService
) { ... }
```

## Android 클래스용으로 생성된 컴포넌트

필드 주입을 수행할 수 있는 각 안드로이드 클래스에는 `@InstallIn` 어노테이션에서 참조할 수 있는 연관된 Hilt 컴포넌트가 있다. **각 Hilt 컴포넌트는 해당 안드로이드 클래스에 바인딩을 주입하는 역할을 담당한다.** 

Hilt는 다음과 같은 컴포넌트를 제공한다. 

![image](https://github.com/user-attachments/assets/71c43e87-66eb-4bc4-b56d-05d5a487f092)

>🚨 **참고:** Hilt는 `SingletonComponent`에서 직접 broadcast receiver를 삽입하므로, broadcast receiver의 컴포넌트는 따로 생성하지 않는다.

### **Component lifetimes**

Hilt는 **해당 Android 클래스의 생명주기에 따라 생성된 컴포넌트 클래스의 인스턴스를 자동으로 만들고 제거**한다.

![image](https://github.com/user-attachments/assets/e320e670-b3e5-462e-9050-4d5f0a7f0d67)

>🚨 **참고:** `ActivityRetainedComponent`는 Configuration Change 전체에 걸쳐 유지되므로, 첫번째 `Activity#onCreate()`에서 생성되고 마지막 `Activity#onDestroy()`에서 소멸된다.

### **Component scopes**

기본적으로 Hilt의 모든 바인딩은 범위가 지정되어 있지 않다. 즉, **앱이 바인딩을 요청할 때마다 Hilt는 필요한 타입의 새 인스턴스를 생성한다.** 

아래 예시에서 Hilt가 필드 주입을 통해 `AnalyticsAdapter`를 `ExampleActivity`에 제공할 때마다 `AnalyticsAdapter`의 새 인스턴스가 생성된다. 

```kotlin
@AndroidEntryPoint
class ExampleActivity : AppCompatActivity() {

  @Inject lateinit var analytics: AnalyticsAdapter 
  ...
}
```

```kotlin
class AnalyticsAdapter @Inject constructor(
    @ActivityContext private val context: Context,
    private val service: AnalyticsService
) { ... }
```

그러나, Hilt는 바인딩을 특정 컴포넌트로 범위를 지정할 수도 있다. **Hilt는 범위가 지정된 컴포넌트의 인스턴스당 한 번만 바인딩을 생성하며, 해당 바인딩에 대한 모든 요청은 동일한 인스턴스를 공유한다.** 

아래 표에는 생성된 각 컴포넌트에 대한 범위 어노테이션이다. 

![image](https://github.com/user-attachments/assets/246efcc5-7ba4-448d-8139-52d15b447a24)

아래 예시에서 `@ActivityScoped`를 사용하여 `AnalyticsAdapter`의 범위를 `ActivityComponent`로 지정하면, Hilt는 해당 액티비티의 생명주기 동안 동일한 `AnalyticsAdapter` 인스턴스를 제공한다. 

```kotlin
@ActivityScoped
class AnalyticsAdapter @Inject constructor(
  private val service: AnalyticsService
) { ... }
```

>🚨 **참고:** 제공된 객체는 컴포넌트가 제거될 때까지 메모리에 남아 있기 때문에, 바인딩의 범위를 컴포넌트로 지정하면 많은 비용이 들 수 있다. 따라서 어플리케이션에서 컴포넌트로 범위가 지정된 바인딩은 최소한으로 사용하는 것이 좋다.<br>
다만, 특정 범위 내에서 동일한 인스턴스를 사용해야 하는 바인딩, 동기화가 필요한 바인딩, 만드는 데 비용이 많이 드는 바인딩에는 특정 컴포넌트로 범위를 지정하는 것이 적절하다.

`AnalyticsService`에 `ExampleActivity`뿐만 아니라 **앱의 모든 위치에서 매번 동일한 인스턴스를 사용해야 하는** 내부 상태가 있다고 가정해보자. 이럴 때는 `AnalyticsService`의 범위를 `SingletonComponent`로 지정하는 것이 적절하다. 결과적으로, 컴포넌트는 `AnalyticsService`의 인스턴스를 제공해야 할 때마다 매번 동일한 인스턴스를 제공한다.

다음 예에서는 Hilt 모듈에서 바인딩의 범위를 컴포넌트로 지정하는 방법을 보여준다. **바인딩의 범위는 바인딩이 설치된 컴포넌트의 범위와 일치해야 하므로**, 이 예에서는 `ActivityComponent` 대신 `SingletonComponent`에 `AnalyticsService`를 설치해야 한다. 

```kotlin
// If AnalyticsService is an interface.
@Module
@InstallIn(SingletonComponent::class)
abstract class AnalyticsModule {

  // SingletonComponent 범위와 일치하도록 
  // @Singleton으로 지정 
  @Singleton
  @Binds
  abstract fun bindAnalyticsService(
    analyticsServiceImpl: AnalyticsServiceImpl
  ): AnalyticsService
}

// If you don't own AnalyticsService.
@Module
@InstallIn(SingletonComponent::class)
object AnalyticsModule {

  @Singleton
  @Provides
  fun provideAnalyticsService(): AnalyticsService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .build()
               .create(AnalyticsService::class.java)
  }
}
```

Hilt 컴포넌트 범위에 관한 자세한 내용은 [이 블로그](https://medium.com/androiddevelopers/scoping-in-android-and-hilt-c2e5222317c0)를 참고하자.

`@ActivityRetainedScoped` 또는 `@ViewModelScoped`를 사용한 범위 지정 간의 차이점에 관한 자세한 내용은 [Hilt 및 Jetpack 통합 문서](https://developer.android.com/training/dependency-injection/hilt-jetpack?hl=ko#viewmodelscoped)의 `@ViewModelScoped` 섹션을 참고하자.

### **Component hierarchy**

컴포넌트에 모듈을 설치하면 **해당 컴포넌트의 다른 바인딩 또는 컴포넌트 계층 구조에서 그 아래 하위 컴포넌트의 의존성으로 해당 바인딩에 접근**할 수 있다. 

<img width="650" alt="스크린샷 2024-08-06 오후 6 46 58" src="https://github.com/user-attachments/assets/c6e27083-a87f-4e0d-a609-48edabbfd586">

>🚨 **참고:** 기본적으로 `View`에서 필드 주입을 실행하면, `ViewComponent`는 `ActivityComponent`에 정의된 바인딩을 사용할 수 있다. 그리고 `FragmentComponent`에 정의된 바인딩도 사용해야 하며, 뷰가 프래그먼트의 일부라면 `@AndroidEntryPoint`와 함께 `@WithFragmentBindings` 어노테이션을 사용하면 된다.

### **Component default bindings**

**각 Hilt 컴포넌트에는 기본 바인딩 세트가 제공되며, 이 바인딩을 사용자 지정 바인딩에 의존성으로 주입할 수 있다.** 이러한 디폴트 바인딩은 **액티비티, 프래그먼트 같은 일반 타입에 해당**하며, 특정 하위 클래스에는 해당되지 않는다. 이는 Hilt가 모든 액티비티를 주입할 때 하나의 `ActivityComponent` 정의를 사용하기 때문이다. 그리고 각 액티비티는 해당 컴포넌트에 대한 서로 다른 인스턴스를 가진다. 

![image](https://github.com/user-attachments/assets/2e5e6669-de81-415c-aba7-2727e3a783f1)

어플리케이션 컨텍스트 바인딩은 `@ApplicationContext`를 통해 사용할 수도 있다. 

```kotlin
class AnalyticsServiceImpl @Inject constructor(
  @ApplicationContext context: Context
) : AnalyticsService { ... }

// The Application binding is available without qualifiers.
class AnalyticsServiceImpl @Inject constructor(
  application: Application
) : AnalyticsService { ... }
```

액티비티 컨텍스트 바인딩은 `@ActivityContext`를 통해 사용할 수도 있다.

```kotlin
class AnalyticsAdapter @Inject constructor(
  @ActivityContext context: Context
) { ... }

// The Activity binding is available without qualifiers.
class AnalyticsAdapter @Inject constructor(
  activity: FragmentActivity
) { ... }
```

## **Hilt가 지원하지 않는 클래스에 의존성 주입**

Hilt는 가장 일반적인 안드로이드 클래스를 지원한다. 하지만 Hilt가 지원하지 않는 클래스에서 필드 주입을 수행해야 할 수도 있다. 

이러한 경우 `@EntryPoint` 어노테이션을 사용하여 진입점을 만들 수 있다. 진입점은 **Hilt에서 관리하는 코드와 그렇지 않은 코드 사이의 경계**이다. **Hilt가 관리하는 객체 그래프에 코드가 처음 들어오는 지점**이다. 진입점을 통해 Hilt는 의존성 그래프 내에서 의존성을 제공하기 위해 Hilt가 관리하지 않는 코드를 사용할 수 있다. 

예를 들어, **Hilt는 콘텐츠 프로바이더를 직접 지원하지 않는다.** 콘텐츠 프로바이더가 Hilt를 사용하여 일부 의존성을 가져오도록 하려면, 원하는 각 바인딩 타입에 대해 `@EntryPoint`로 어노테이션이 달린 인터페이스를 정의하고 한정자를 포함해야 한다. 이후 `@InstallIn`을 추가하여 다음과 같이 진입점을 설치할 컴포넌트를 지정하면 된다. 

```kotlin
class ExampleContentProvider : ContentProvider() {

  @EntryPoint
  @InstallIn(SingletonComponent::class)
  interface ExampleContentProviderEntryPoint {
    fun analyticsService(): AnalyticsService
  }

  ...
}
```

엔트리 포인트에 액세스하려면 `EntryPointAccessors`의 적절한 정적 메서드를 사용하면 된다. 매개변수는 컴포넌트 인스턴스 또는 컴포넌트 홀더 역할을 하는 `@AndroidEntryPoint` 객체여야 한다.

매개변수로 전달하는 컴포넌트와 `EntryPointAccessors` 정적 메서드가 모두 `@EntryPoint` 인터페이스의 `@InstallIn` 어노테이션에 있는 Android 클래스와 일치하는지 확인한다. 

```kotlin
class ExampleContentProvider: ContentProvider() {
    ...

  override fun query(...): Cursor {
    val appContext = context?.applicationContext ?: throw IllegalStateException()
    val hiltEntryPoint =
      EntryPointAccessors.fromApplication(appContext, ExampleContentProviderEntryPoint::class.java)

    val analyticsService = hiltEntryPoint.analyticsService()
    ...
  }
}
```

이 예시에서는 엔트리 포인트가 `SingletonComponent`에 설치되어 있으므로, 엔트리 포인트를 검색하려면 `ApplicationContext`를 사용해야 한다. 검색하려는 바인딩이 `ActivityComponent`에 있었다면, 대신 `ActivityContext`를 사용했을 것이다. 

## Hilt, Dagger

**Hilt는 Dagger 의존성 주입 라이브러리 위에 구축되어 Dagger를 Android 어플리케이션에 통합하는 표준적인 방법을 제공한다.** 

Dagger와 관련하여 Hilt의 목표는 다음과 같다.

- 안드로이드 앱의 Dagger 관련 인프라를 단순화한다.
- 표준 컴포넌트 및 범위 세트를 만들어 앱 간의 설정, 가독성 및 코드 공유를 용이하게 한다.
- 테스트, 디버그 또는 릴리즈와 같은 다양한 빌드 타입에 여러 바인딩을 쉽게 제공할 수 있게 한다.

**안드로이드 OS는 많은 자체 프레임워크 클래스를 인스턴스화하기 때문에, 안드로이드 앱에서 Dagger를 사용하려면 상당한 양의 상용구 코드를 작성해야 한다.** Hilt는 안드로이드 앱에서 Dagger를 사용하는 데 필요한 상용구 코드를 줄여준다. 

Hilt는 다음을 자동으로 생성하여 제공한다. 

- Dagger에 안드로이드 프레임워크 클래스를 통합하기 위한 **컴포넌트**
- Hilt가 자동으로 생성하는 컴포넌트와 함께 사용할 **범위 어노테이션**
- 애플리케이션, 액티비티와 같은 안드로이드 클래스를 나타내는 **사전에 정의된 바인딩**
- `@ApplicationContext` 및 `@ActivityContext`를 표현하기 위한 **사전에 정의된 한정자**

Dagger, Hilt 코드는 동일한 코드베이스에 공존할 수 있다. **그러나 대부분의 경우 Hilt를 사용하여 Android에서 Dagger의 모든 사용을 관리하는 것이 가장 좋다.** Dagger를 사용하는 프로젝트를 Hilt로 마이그레이션하려면, [마이그레이션 가이드](https://dagger.dev/hilt/migration-guide)와 [Dagger 앱을 Hilt로 마이그레이션](https://developer.android.com/codelabs/android-dagger-to-hilt?hl=ko#0) 하는 코드랩을 참고하자. 

## 참고자료

https://developer.android.com/training/dependency-injection/hilt-android?hl=ko