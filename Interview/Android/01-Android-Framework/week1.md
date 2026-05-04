## Q0. 안드로이드란 무엇인가요?

<details>
<summary>안드로이드 플랫폼 아키텍처는 리눅스 커널, ART, HAT 등 여러 계층으로 구성됩니다. 이 구성 요소들이 어플리케이션 실행과 하드웨어와의 상호작용을 위해 어떻게 작동하는지 설명해 주세요.</summary>

안드로이드 플랫폼 아키텍처는 다음과 같이 구성됩니다. 

- Application: 사용자가 실행하는 앱 
- Android Framework: 앱이 사용할 수 있는 고수준 API 제공 
- ART (Android Runtime): Kotlin/Java에서 컴파일된 바이트 코드 실행 
- Native Libraries: 그래픽 렌더링, 데이터베이스 등 네이티브 기능 제공 
- HAL (Hardware abstraction layer): 프레임워크와 하드웨어 구현 사이의 추상화 
- Linux Kernel: 프로세스, 메모리, 장치 드라이버, 보안 등 OS 핵심 기능 담당 
- Hardware: 실제 카메라, 센서, 디스플레이, 블루투스 등 

결국 안드로이드 앱은 프레임워크 API를 통해 시스템 서비스에 요청하고, 시스템은 필요에 따라 HAL과 Linux Kernel을 거쳐 하드웨어를 제어합니다. 그리고 앱의 Kotlin/Java 코드는 ART 위에서 실행됩니다. 

</details>

## Q1. 인텐트란 무엇인가요?

<details>
<summary>인텐트란 무엇인가요?</summary>

안드로이드에서 어떤 작업을 수행해달라고 시스템이나 다른 컴포넌트에 전달하는 메시지 객체입니다.

</details>

<br>

<details>
<summary>명시적 인텐트와 암시적 인텐트의 차이점은 무엇이며, 각각 어떤 시나리오에서 사용해야 하나요?</summary>

||명시적 인텐트|암시적 인텐트|
|---|---|---|
|대상 지정|컴포넌트 직접 지정|수행할 작업만 선언|
|실행 대상|보통 앱 내부 컴포넌트|시스템이 적절한 앱, 컴포넌트 탐색|
|안정성|실행 대상 명확|처리 가능한 앱이 없을 수 있음|
|사용 사례|액티비티 전환, 서비스 실행 등|웹 열기, 전화 걸기, 공유하기 등|

```kotlin
// 화면 전환하기 
val intent = Intent(this, ProfileActivity::class.java)
startActivity(intent)
```

```kotlin
// 웹 페이지 열기 
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://example.com"))
startActivity(intent)

// 전화 앱 열기 
val intent = Intent(Intent.ACTION_DIAL, Uri.parse("tel:01012345678"))
startActivity(intent)

// 공유하기 
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, "공유할 내용")
}
startActivity(Intent.createChooser(intent, "공유하기"))
```

</details>

<br>

<details>
<summary>안드로이드 시스템은 암시적 인텐트를 처리할 앱을 어떻게 결정하며, 적합한 어플리케이션을 찾지 못하면 어떻게 되나요?</summary>

안드로이드 시스템은 암시적 인텐트의 action, data, category 등을 설치된 앱들의 인텐트 필터와 비교해서 실행 대상을 찾습니다. 

👉 처리 가능한 앱이 여러 개 있으면?

시스템은 사용자에게 앱 선택 창을 보여줄 수 있고, 기본 앱이 설정되어 있다면 선택 창 없이 해당 앱이 바로 실행됩니다. 

공유하기처럼 사용자가 매번 선택하는 게 자연스러운 기능은 보통 `Intent.createChooser()`를 사용합니다.

👉 적합한 앱을 찾지 못하면? 

`ActivityNotFoundException` 예외가 발생할 수 있으므로, 해당 인텐트를 처리할 수 있는 액티비티가 있는지 확인하는 게 좋습니다.

```kotlin 
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://example.com"))

if (intent.resolveActivity(packageManager) != null) {
    startActivity(intent)
} else {
    // 처리할 앱이 없는 경우
    Toast.makeText(this, "실행할 수 있는 앱이 없습니다.", Toast.LENGTH_SHORT).show()
}
```

</details>

<br>

<details>
<summary>인텐트 필터란 무엇인가요?</summary>

액티비티, 서비스, 또는 브로드캐스트가 처리할 수 있는 인텐트 유형을 선언하는 필터 역할이며, AndroidManifest.xml 파일에 명시됩니다. 

각 인텐트 필터는 들어오는 인텐트와 정확히 일치시키기 위해 액션, 카테고리, 데이터 유형 등을 포함할 수 있습니다.

</details>


## Q2. PendingIntent의 목적은 무엇인가요?

<details>
<summary>PendingIntent란 무엇이며 일반 Intent와 어떻게 다른가요? PendingIntent 사용이 필요한 시나리오를 제시해 줄 수 있나요?</summary>

일반 Intent는 앱이 즉시 컴포넌트를 실행하거나 작업을 요청할 때 사용하는 메시지 객체입니다. 반면에, PendingIntent는 Intent를 즉시 실행하지 않고 알림 클릭 같은 특정 시점에 시스템이나 다른 앱에서 해당 인텐트를 실행하도록 하는 객체입니다. 

PendingIntent 사용이 필요한 시나리오는 다음과 같습니다. 

- 알림 클릭 시 특정 화면 열기 
- 알람 매니저로 특정 시간에 작업 실행 
- 앱 위젯 (런처가 관리하는 UI) 클릭 시 앱 실행 및 특정 화면 열기 
- 알림의 액션 버튼 클릭 시 해당 동작 수행 

</details>

## Q3. Serializable과 Parcelable의 차이점은 무엇인가요?

<details>
<summary>안드로이드에서 Serializable과 Parcelable의 차이점은 무엇이며, 일반적으로 컴포넌트 간 데이터 전달에 Parcelable이 선호되는 이유는 무엇인가요?</summary>

||Serializable|Parcelable|
|---|---|---|
|유형|Java 표준 인터페이스|안드로이드 전용 인터페이스|
|구현 난이도|매우 쉬움|kotlin-parcelize 플러그인으로 구현 쉬워짐|
|동작 방식|리플렉션 기반 직렬화|명시적으로 Parcel에 값을 읽고 쓰기|
|성능|상대적으로 느림|상대적으로 빠름|
|메모리 사용|직렬화 중에 많은 임시 객체 생성|가비지 컬렉션 최소화|
|사용 사례|파일 저장, 자바 객체 직렬화 등 범용적|안드로이드 컴포넌트 간 데이터 전달|

안드로이드의 Intent, Bundle, Binder IPC는 내부적으로 데이터를 Parcel 타입으로 전달합니다. Parcelable은 해당 구조에 맞게 최적화 되어 있어, 객체를 더 빠르고 효율적으로 전달할 수 있습니다. 따라서, 안드로이드 컴포넌트 간 데이터 전달에는 Parcelable이 선호됩니다. 

</details>

<br>

<details>
<summary>리플렉션이 무엇인가요?</summary>

프로그램 실행 중에 동적으로 클래스, 필드, 메서드 같은 정보에 접근할 수 있는 기능이며, 런타임에 객체 구조를 분석해야 하므로 느릴 수 있습니다. 

</details>

<br>

<details>
<summary>Parcel와 Parcelable이란 무엇인가요?</summary>

Parcel는 안드로이드에서 다른 컴포넌트나 프로세스에 데이터를 전달하기 위한 컨테이너 클래스입니다. 객체 안의 값을 순서대로 Parcel에 쓰고, 수신자도 동일한 순서로 값을 읽어서 객체를 복원합니다. (쓰는 순서와 읽는 순서가 중요)

Parcelable은 객체를 Parcel에 어떻게 쓰고 읽을지 정의하는 인터페이스입니다. 

- Parcelable: 포장 방법 설명서
- Parcel: 실제 포장 상자
- Intent: 배송 수단

Parcel, Parcelable은 어떤 값을 어떤 순서로 담을지 명확하게 정해져 있기 때문에, 리플렉션 기반의 Serializable에 비해 성능이 좋습니다. 

</details>

## Q4. Context란 무엇이며 어떤 유형의 Context가 있나요?

<details>
<summary>Context란 무엇이며 어떤 유형의 Context가 있나요?</summary>

Context란 어플리케이션의 환경 또는 상태를 나타내며, 앱 컴포넌트와 안드로이드 시스템 간의 브릿지 역할을 합니다. 

- Application Context : 어플리케이션의 생명주기와 연결되며, 앱 전역적으로 오래 지속되는 Context가 필요할 때 사용 
- Activity Context : Activity의 생명주기와 연결되며, Activity의 특정 리소스 접근, UI 컴포넌트 생성 및 수정, 다른 액티비티 실행 등에 사용 
- Service Context : Service의 생명주기와 연결되며, 주로 네트워크나 음악 재생 같은 백그라운드 작업에서 사용 
- Broadcast Context : BroadcastReceiver가 호출될 때 제공되며, 수명이 짧고 특정 브로드캐스트에 응답할 때 사용 

</details>

<br>

<details>
<summary>안드로이드 어플리케이션에서 올바른 유형의 Context를 사용하는 것이 왜 중요하며, Activity Context에 대해 오랜 참조를 유지하는 것은 잠재적으로 어떤 문제를 발생시킬 수 있나요?</summary>

Context는 단순히 환경 정보가 아니라, 각 종류에 따라 생명 주기를 갖고 있기 때문에 용도에 따라 적절하게 선택해야 합니다. 

Activity Context를 싱글톤, 뷰모델, 비동기 콜백 등에서 오래 참조하는 경우, Activity가 소멸된 후에도 참조가 유지되어 메모리 누수가 발생할 수 있습니다. 특히 Activity는 View, Fragment, Adapter 등을 많이 참조하기 때문에 누수 비용이 큽니다. 

따라서, 오래 지속되는 객체는 Application Context를 사용하고, Activity Context는 UI 관련 작업에서 짧게 사용하는 게 권장됩니다. 

</details>

<br>

<details>
<summary>ContextWrapper란 무엇인가요?</summary>

ContextWrapper는 Context 동작을 커스텀 하기 위한 유연하고 재사용 가능한 API입니다. 

개발자가 원본 Context를 직접 변경하지 않고 ContextWrapper에 호출을 위임하여 Context의 동작을 재정의 할 수 있습니다. 

</details>

<br>

<details>
<summary>Activity에서 this와 baseContext 인스턴스의 차이점은 무엇인가요?</summary>

- this : 현재 액티비티 인스턴스를 참조하며, 액티비티 생명주기나 UI 관련 작업에 사용되는 고수준 Context 
- baseContext : Activity가 기반으로 하는 저수준 Context (Context의 핵심 구현을 제공하는 ContextImpl의 인스턴스), 커스텀 ContextWrapper 구현 같은 고급 시나리오에서 사용 

</details>
