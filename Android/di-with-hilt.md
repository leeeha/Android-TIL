# 의존성 주입 (Dependency Injection)

생성자 또는 메서드 등을 통해 **외부로부터 생성된 객체를 전달받는 것** 

- 클래스 간 **결합도를 느슨하게** 한다.
- 인터페이스 기반으로 설계되어 코드를 **유연하게** 한다.
- stub 또는 mock 객체를 사용하여 **단위 테스트**를 하기가 더 쉬워진다.

```kotlin
// 의존성 주입이 없는 코드 
class MemoRepository {
	private val db = SQLiteDatabase() // 객체 생성 
	
	fun load(id: String) { ... }
}

fun main() {
	val repository = MemoRepository()
	repository.load("8092")
}
```

```kotlin
// 의존성을 외부로부터 주입 받는다.
class MemoRepository(private val db: Database) {
	fun load(id: String) { ... }
}

fun main() {
	val db = SQLiteDatabase()
	val repository = MemoRepository(db)
	repository.load("8092")
}
```

## 안드로이드에서 의존성 주입이 어려운 이유?

- 액티비티, 서비스, 브로드캐스트 리시버 등 안드로이드 플랫폼 기반의 클래스들은 **안드로이드 프레임워크에 의해 인스턴화** 되기 때문이다.
- Factory를 API 28부터 제공하지만, 현실적인 방법은 아니다.

## Dagger2

Dagger2는 자바와 안드로이드를 위한 강력하고 빠른 의존성 주입 프레임워크 

장점 

- 컴파일 타임에 그래프 구성
- 생성된 코드는 명확하고 디버깅 가능함.
- reflection 사용 X, 런타임 바이트 코드 생성 X
- 자원 공유의 단순화
- 작은 라이브러리 크기
- 상위 10000개의 안드로이드 앱 중에 74%가 Dagger 사용  (20년도 기준)

단점 

- 배우기 어렵고 프로젝트 설정이 힘들다.
- 간단한 프로그램을 만들 때는 번거롭다.
- 같은 결과에 대한 다양한 방법이 존재한다.
- 개발자 중 49%가 DI 솔루션 개선을 요청함.

## 의존성 주입 프레임워크의 궁극적 목표

- 정확한 사용 방법 제안
- 쉬운 설정 방법
- 중요한 것에 집중할 수 있게 함.

# Hilt

Hilt는 **어플리케이션에서 DI를 사용하는 표준적인 방법을 제공**한다. 

Hilt의 목표 

- Dagger 사용의 단순화
- 표준화 된 컴포넌트 세트와 스코프 → 쉬운 설정, 가독성과 이해도 향상
- 쉬운 방법으로 다양한 빌드 타입에 대한 바인딩 제공

## Hilt 프로젝트 설정

```groovy
// 모듈 단위의 build.gradle 파일 
dependencies {
	implementation 'com.google.dagger:hilt-android:<version>'
	kapt 'com.google.dagger:hilt-android-compiler:<version>'
}
```

```groovy
// 프로젝트 단위의 build.gradle 파일 
buildscript {
	repositories {
		mavenCentral()
	}
	
	dependencies {
		classpath 'com.google.dagger:hilt-android-gradle-plugin:<version>'
	}
}

// 모듈 단위의 build.gradle 파일 
apply plugin: 'com.android.application'
apply plugin: 'dagger.hilt.android.plugin'

android {...}
```

## Hilt의 특징

- Dagger2 기반의 라이브러리
- 표준화 된 Dagger2 사용법 제시
- 보일러 플레이트 코드 감소
- 프로젝트 설정 간소화
- 쉬운 모듈 탐색과 통합
- 개선된 테스트 환경
- 안드로이드 스튜디오의 지원
- androidx 라이브러리와의 호환

## Quick Setup

```kotlin
class MemoRepository @Inject constructor(
	private val db: MemoDatabase // 생성자 주입 
){
	fun load(id: String) {...}
}
```

```kotlin
@HiltAndroidApp
class MemoApp: Application() // 어플리케이션의 진입점 
```

```kotlin
@AndroidEntryPoint
class MemoActivity: AppCompatActivity() {
	@Inject lateinit var repository: MemoRepository // 필드 주입 
	
	override fun onCreate(savedInstanceState: Bundle){
		super.onCreate(bundle)
		repository.load("hello") 
	}
}
```

```kotlin
@InstallIn(SingletonComponent:class)
@Module
object DataModule {
	@Provides // Room DB 객체의 생성 방법을 알려주는 모듈 설치 
	fun provideMemoDB(@ApplicationContext context: Context) = 
		Room.databaseBuilder(context, MemoDatabase::class.java, "Memo.db").build()
}
```

## Object Graph

<img width="800" src="https://github.com/leeeha/Android-TIL/assets/68090939/4985edbd-9e4f-449d-9df3-518f24b79fff"/>

## Hilt Annotation

- @HiltAndroidApp
- @AndroidEntryPoint
- @InstallIn
- @EntryPoint

```kotlin
// @HiltAndroidApp 없이 컴포넌트 생성하기 
class MemoApplication: Application() {

	override fun onCreate() {
		super.onCreate()
		
		val component = DaggerMemoComponent.builder()
			...
			.build()
			
		component.inject(this)
	}
}
```

```kotlin
// @HiltAndroidApp
// Hilt 코드 생성의 시작, 반드시 Application 클래스에 추가 
@HiltAndroidApp
class MemoApplication: Application() {
	override fun onCreate() {
		super.onCreate() // 여기서 내부적으로 의존성 주입 (바이트 코드 변환에 의해) 
	}
}
```

## 바이트 코드 변환

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/8797dcae-56f8-4679-a010-add71e7bbf01"/>

`@HiltAndroidApp` 어노테이션이 붙은 Application 클래스는, 컴파일 타임에 `Hilt_` 라는 접두어가 붙은 어플리케이션 클래스를 생성한다. 

해당 클래스에는 컴포넌트 생성 및 주입 관련 코드가 포함되어 있다. 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/ff5e969c-7d24-4153-b644-4df9dc7194a4"/>

그러면, `MemoApplication`에서 `Hilt_MemoApplication`을 상속 받아야 할 거 같지만, 실제로는 그러지 않아도 된다. 왜 그럴까?

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/0b2ddc51-aefa-4646-a78d-e9fbf90a52c0"/>

그 이유는 `MemoApplication` 소스코드를 컴파일하여 만들어진 바이트 코드가 

Gradle 플러그인에 의해 `Hilt_MemoApplication`을 상속하는 `MemoApplication` 바이트 코드로 변환되기 때문이다. 

이는 개발자의 편의를 위해 도모된 것이며, 원하지 않는다면 그레들 플러그인을 비활성화 시키면 된다. 

```kotlin
@HiltAndroidApp(Application::class)
class MyApplication: Hilt_MyApplication()
```

만약 그레드 플러그인을 사용하지 않는다면 위와 같이 코드를 작성하면 된다. 

## @AndroidEntryPoint

- 어노테이션이 붙은 안드로이드 클래스에 DI 컨테이너를 추가할 수 있다.
- @HiltAndroidApp 설정 이후 사용 가능하다.
- Dagger2 관점에서 보면 다음과 같다.
    - @HiltAndroidApp → Component 생성
    - @AndroidEntryPoint → Subcomponent 생성
- 지원하는 타입
    - Activity
    - Fragment
    - View
    - Service
    - BroadcastReceiver
    - ContentProvider → 지원 X

## Hilt Component Hierarchy

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/26ebcc56-1cbf-477a-904a-a27034865558"/>

- Hilt는 바이트 코드 변환을 사용하므로, Dagger와 다르게 **직접적으로 인스턴스화 작업을 할 필요가 없다.**
- 컴포넌트의 각 생명주기에 맞는 **표준화 된 컴포넌트 세트와 스코프**를 제공한다.
- 컴포넌트들은 계층으로 이루어져 있으며, **하위 컴포넌트는 상위 컴포넌트의 의존성에 접근**할 수 있다.

## Hilt Scope

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/ea0d8d18-c653-436e-ae9e-f4a121f8b575"/>

위와 같은 **표준화 된 스코프 어노테이션을 사용하여 동일 인스턴스를 공유**할 수 있다. 

## Scoped Binding

### No scope annotation

```kotlin
class MemoRepository @Inject contructor(
	private val db: MemoDatabase
){
	fun load(id: String) {...}
}
```

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/61d50039-bd84-4d8b-a396-bb1b745c1e2e"/>

Scope 어노테이션이 붙지 않은 경우, 기본적으로 DetailActivity에서 MemoRepository가 필요할 때 **ActivityComponent에 설치된 모듈에서 MemoRepository를 새로 생성**한다. 

따라서, MemoActivity, DetailActivity가 가지고 있는 **MemoRepository의 인스턴스는 서로 다르다.** 

### @Singleton

```kotlin
@Singleton
class MemoRepository @Inject contructor(
	private val db: MemoDatabase
){
	fun load(id: String) {...}
}
```

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/ab305052-b82c-4811-b7a3-73f3ef16045d"/>

반면에 @Singleton 어노테이션을 붙인 경우, 두 액티비티가 ApplicationComponent에 설치된 모듈로부터 **동일한 MemoRepository 인스턴스를 주입**받는다. 이처럼 힐트는 **자원 공유**를 쉽게 할 수 있도록 도와준다. 

## 기본 컴포넌트 바인딩

컴포넌트는 기본적으로 다음과 같은 객체를 그래프에 바인딩한다. 

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/01763f98-06b8-40c7-8bc3-4600117d3588"/>

@ApplicationContext 또는 @ActivityContext를 사용하여 적절한 Context를 제공 받을 수 있다. 

## Hilt Module

### @InstallIn

- 해당 모듈이 어떤 컴포넌트에 설치될 것인지 명시하는 어노테이션
- **올바르지 않은 컴포넌트 또는 스코프를 사용하면 컴파일 에러 발생하므로 주의**

![image](https://github.com/leeeha/Android-TIL/assets/68090939/00621358-21cb-4fc2-89b5-e0496f6f8a43)

만약 ActivityComponent, FragmentComponent 모두 MyModule을 필요로 하는 경우에는? 

**상위 컴포넌트인 ActivityComponent에 모듈 설치를 고려**할 수 있다. **하위 컴포넌트는 상위 컴포넌트의 의존성에 접근**할 수 있기 때문이다.

![image](https://github.com/leeeha/Android-TIL/assets/68090939/d5914091-e917-4e47-b5b2-f1890323f7f5)

### Hilt Module의 제약사항

`@Module` 클래스에 `@InstallIn`이 없으면 컴파일 에러 발생 

```kotlin
// @InstallIn 검사 비활성화 (Dagger -> Hilt 마이그레이션 시 필요할 수도)
android {
	defaultConfig {
		javaCompileOptions {
			annotationProcessorOptions {
				arguments += ["dagger.hilt.disableModulesHaveInstallInCheck" : "true"]
			}
		}
	}
}
```

## @EntryPoint

**힐트가 지원하지 않는 클래스에서 의존성이 필요한 경우 사용** 

ex) ContentProvider, DFM(Dynamic Feature Module), Dagger를 사용하지 않는 서드파티 라이브러리 등 

- 인터페이스에서만 사용 가능
- @InstallIn 어노테이션 필수
- EntryPoints 클래스의 정적 메서드를 통해 그래프에 접근 가능

```kotlin
@EntryPoint 
@InstallIn(SingletonComponent::class)
interface FooBarInterface {
	fun getBar(): Bar
}

val bar = EntryPoints.get(
	applicationContext,
	FooBarInterface::class.java
).getBar()
```

### ContentProvider 예제

<img width="700" src="https://github.com/leeeha/Android-TIL/assets/68090939/97bc072b-d979-4d02-8c82-ff8044d19945"/>

```kotlin
@EntryPoint 
@InstallIn(SingletonComponent::class)
interface MemoEntryPoint {
	fun getRepository(): MemoRepository
}

class MemoProvider: ContentProvider() {
	override fun query() {
		val entryPoint = EntryPointAccessors.fromApplication(context, MemoEntryPoint::class.java)
		val repository = entryPoint.getRepository()
		// ... 
	}
}
```

## Custom Component

- **표준 힐트 컴포넌트 이외에 새로운 컴포넌트를 만드는 것**
- 커스텀 컴포넌트를 사용하면 그래프가 복잡하고 이해하기 어려워지기 때문에 꼭 필요한 경우에만 사용
- 제약조건
    - 반드시 SingletonComponent 하위 계층의 컴포넌트로 생성
    - 표준 컴포넌트 계층 사이에 추가할 수 없음.

# 참고자료

https://youtu.be/gkUCs6YWzEY?feature=shared

https://developer.android.com/training/dependency-injection/hilt-android?hl=ko

https://velog.io/@haero_kim/Android-DI-개념-라이브러리-없이-직접-구현해보기