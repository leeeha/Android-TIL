# Context

Context는 **어플리케이션 환경에 대한 전체 정보를 받을 수 있도록 안드로이드 시스템이 제공하는 추상 클래스**이다. 따라서, Context가 없으면 **액티비티, 브로드캐스트 리시버, 서비스** 등을 시작할 수조차 없다. 또한 **리소스, DB, SharedPrefrences 등에 접근**하기 위해서도 Context가 필요하다.

![image](https://github.com/leeeha/Android-TIL/assets/68090939/0d7bbdf7-91f2-4863-91aa-595cf3df53bf)

## ContextWrapper

Context의 대리인(프록시)으로, **원본 Context를 변경하지 않고 동작을 수정할 수 있게 해주는 Context의 자식 클래스**이다. **보호 프록시 패턴**으로 구현되어 있어서 ContextWrapper는 ContextImpl의 변수는 노출하지 않고, **외부에서 ContextImpl의 공개 메서드만 호출**할 수 있도록 한다. 

## ContextImpl

ContextWrapper는 **Context의 여러 메서드를 구현한 ContextImpl 인스턴스를 생성자의 인자로** 받는다. ContextImpl는 앱에서 직접 사용할 수 있는 클래스가 아니므로 소스 코드로만 확인할 수 있다.

Context와 컴포넌트(Activity, Service, Application) 사이에 위치한 게 ContextWrapper이다. ContextWrapper는 **ContextImpl에서 구현한 함수를 중간에서 일부만 공개하거나 수정**한다. (보호 프록시 패턴)

이를 통해 컴포넌트들이 ContextImpl를 직접 상속하여 Context 메서드를 사용하는 게 아니라, **ContextWrapper를 중간에 두어 Context 메서드를 사용**하게 한다.

# Context 종류

- **Application Context** : 싱글톤 객체로 **어플리케이션의 생명주기**를 따른다. 즉, 앱의 시작부터 종료까지 살아있다.
- **Activity Context** : Activity 자체가 Context를 상속 받고 있기 때문에, **Activity 인스턴스 자체가 Context 역할**을 한다. 따라서, Activity Context는 **Activity의 생명주기**를 따른다.

# 컴포넌트에서 Context 참조

위의 Context 클래스 다이어그램에서 볼 수 있듯이, ContextWrapper를 상속하는 컴포넌트는 **Activity, Service, Application**이다. 

Broadcast Receiver, Content Provider는 ContextWrapper를 상속 받지 않는다. 

<img width="700" alt="스크린샷 2024-06-30 오후 7 48 58" src="https://github.com/leeeha/Android-TIL/assets/68090939/ff435dd4-cee8-4240-972e-5491abd440af">

그렇다면 각 컴포넌트에서 Context를 어떻게 참조할까? 

- **Application**: 싱글톤으로 어플리케이션 객체가 생성될 때 함께 Application Context가 생성되기 때문에, Application Context는 **동일한 앱에서 항상 동일한 인스턴스를 반환**한다.
- **Activity, Service**: 액티비티나 서비스가 생성될 때마다 **각자의 Context 인스턴스가 생성**된다.
- **Broadcast Receiver**: 자기 자신이 Context는 아니다. 리시버가 브로드캐스트를 처리할 때마다 **Context를 onReceive() 메서드의 인자로 받아서 사용**한다. 전달 받은 Context의 생명주기를 따르기 때문에, Activity Context로 브로드캐스트 실행 시, 액티비티가 종료되면 브로드캐스트 리시버도 함께 종료된다.
- **Content Provider**: 자기 자신이 Context는 아니다. 각 앱마다 고유한 Application Context를 사용한다. 하지만 같은 앱 내에서는 동일한 Context 객체를 공유하기 때문에 싱글톤 Context로 동작한다. 

# Context 참조 방법

Activity에서 Context를 가져오는 3가지 방법이 있다. 

- **Activity 인스턴스 자신 (this)**
- **getBaseContext()에서 가져오는 ContextImpl 인스턴스**
- **getApplicationContext()에서 가져오는 ApplicationContext**

이 세 개의 인스턴스는 **모두 다른 인스턴스**이다. 

따라서 getBaseContext()에서 가져온 것을 Activity로 캐스팅하면 **ClassCaseException**이 발생한다. ApplicatoinContext 또한 Activity로 캐스팅하면 에러가 발생한다. 

추가적으로 View 클래스에도 getContext() 메서드가 있어서 Context를 가져올 수 있는데, View 객체를 생성할 때 생성자의 인자로 넣은 Context가 getContext()로 반환된다. **일반적으로 View가 속해있는 Activity의 Context가 해당 View의 Context가 된다.** 

## 2025 안드로이드 탐구 영역 3번 문제 

📌 시험지 링크: https://android-exam25.gdg.kr/

안드로이드에서 제공하는 Context로 적절하지 않은 것은?

1. Application Context 
2. Activity Context
3. Base Context 
4. Resource Context 
5. Service Context 

<details>
<summary>정답은 여기서 확인!</summary>

정답: 4번

Resource Context는 공식적으로 정의된 Context 타입은 아니며, 리소스 작업에 사용되는 Context를 비공식적으로 지칭하는 용어이다. 

보통은 Activity Context, Application Context, 또는 ContextThemeWrapper를 통해 리소스나 테마에 접근한다. 

</details>

<details>
<summary>Base Context란 무엇인가?</summary>

Base Context는 ContextWrapper 클래스에서 내부적으로 사용하는 Context를 의미한다. ContextWrapper는 다른 Context 객체를 감싸는 역할을 하며, Base Context는 이 **ContextWrapper가 실제 작업을 처리할 때 위임하는 Context 객체**이다. 

예를 들어, **ContextWrapper 클래스를 상속 받아 커스텀 Context를 정의**하거나, **ContextThemeWrapper로 기존 Context에 테마를 적용할 때** 사용할 수 있다. 

```kotlin 
class MyCustomContext(base: Context) : ContextWrapper(base) {
    override fun getSystemService(name: String): Any? {
        if (name == Context.LAYOUT_INFLATER_SERVICE) {
            // 커스텀 LayoutInflater 제공
            return LayoutInflater.from(baseContext).cloneInContext(this)
        }
        return super.getSystemService(name)
    }
}
```

```kotlin 
// 테마가 적용된 Context 생성
val themedContext = ContextThemeWrapper(baseContext, R.style.CustomTheme)

// 테마가 적용된 LayoutInflater로 View 생성
val inflater = LayoutInflater.from(themedContext)
val customView = inflater.inflate(R.layout.custom_layout, null)
```

</details>

# Context 참조 시 유의사항

Context 종류에 따라 생명주기가 다르기 때문에 **Activity와 분리된 작업에 Activity Context를 사용하면 예기치 못한 에러가 발생**할 수 있다. 

반대로, **Activity에 종속된 작업에서 Application Context를 사용**하면, Activity가 종료되어도 싱글톤으로 생성된 Application Context는 가비지 컬렉션 되지 않고 계속 살아남아 결과적으로 **메모리 누수가 발생**할 수 있다. 

따라서, **컴포넌트의 생명주기에 종속될 때는 Context를 사용하고, 어플리케이션 전역에서 싱글톤으로 사용된다면 Application Context를 사용**하도록 하자! 

**항상 가까운, 밀접한 스코프의 Context**를 골라서 사용한다고 생각하면 헷갈리지 않을 것이다! 

# 참고자료

[[Android] Context, 너 대체 뭐야?](https://velog.io/@haero_kim/Android-Context-너-대체-뭐야)

[[Android, Context] Context 넌 무엇이더냐?](https://black-jin0427.tistory.com/220)

[[Android] Context 제대로 알고 사용하자!](https://s2choco.tistory.com/10)
