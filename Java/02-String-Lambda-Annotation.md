>- 자료 출처: [JSCODE Java 모의면접 스터디](https://jscode-study.oopy.io/)
>- 스터디 기간: 2024.11.01 ~ 2024.11.29

## 문자열, 예외, 제네릭

<details>
<summary>String literal과 new String(””)의 차이를 설명해 주세요.</summary>

- String literal : String Constant Pool에 저장되며, 동일한 리터럴은 재사용된다. 
- new String("") : 힙 메모리 영역에 매번 새로운 String 인스턴스를 생성한다. 

따라서, **문자열 리터럴을 사용하는 게 메모리를 더 절약**할 수 있는 방법이다.

```java
String first = "leeeha"; 
String second = "leeeha"; 
String third = new String("leeeha");
String fourth = new String("leeeha"); 

System.out.println(first == second); // true 
System.out.println(third == fourth); // false
System.out.println(first == third); // false

String constantString = "interned leeeha";
String newString = new String("interned leeeha");
System.out.println(constantString == newString); // false

String internedString = newString.intern(); // String Pool에서 내용이 같은 문자열 검색 
System.out.println(constantString == internedString); // true
```

<img width="500" src="https://github.com/user-attachments/assets/33678da2-fb8c-471e-8f39-0f161c55c1bd">

<img width="500" src="https://github.com/user-attachments/assets/3bcfa433-1cdd-4591-a800-334935b8999b">

</details>
<br>

<details>
<summary>String, StringBuilder, StringBuffer의 차이점에 대해서 설명해주세요.</summary>

- **String**
  - 메모리에 한번 할당되면 값이 변하지 않는 **불변 객체** 
  - 동기화를 신경쓰지 않아도 된다. 
  - 문자열이 변경될 때마다 메모리의 할당 및 해제가 반복되어 성능이 좋지 않다. 
- **StringBuild**
  - 원본 문자열을 변경할 수 있는 **가변 객체** 
  - 문자열이 변경될 때 바로 메모리를 할당하지 않고, 버퍼 공간을 먼저 활용하기 때문에 메모리 효율적이다. 
  - 버퍼 공간에 있는 문자를 포인터로 가리키듯이 문자열 변경이 일어나므로, String보다 처리 속도가 느린 편이다.
  - 문자열이 변경되지 않을 때는 일정한 버퍼 공간이 오히려 메모리 낭비가 될 수 있다. 
  - 동기화 처리가 되어 있지 않다. 
- **StringBuffer**
  - 각 메서드가 synchronized 키워드로 **동기화 처리**되어 있어서, 멀티 스레드 환경에서 Thread-Safe하게 사용할 수 있다.
  - 싱글 스레드 환경에서는 StringBuilder를 사용하는 게 성능이 더 좋다. 

```kotlin 
fun main() {
    val s = java.lang.StringBuilder("Hello")
    s[2] = 'x' // 요소의 변경이 가능해짐.
    println(s) // Hexlo 
}
```

<img width="400" src="https://velog.velcdn.com/images/jxlhe46/post/8ecbc62a-e51f-484b-b6b3-a2cc77d2c3cc/image.png"/>

</details>
<br>

<details>
<summary>Exception과 Error의 차이는 무엇인가요?</summary>

- 오류(Error): 시스템 종료와 같이 개발자가 수습할 수 없는 심각한 문제. 개발자가 미리 예측하여 방지할 수 없다. 
- 예외(Exception): 개발자의 프로그램 구현 로직이나 사용자 입력에 의해 발생하는 문제. 개발자가 미리 예측하여 방지할 수 있으므로, 예외 처리가 중요하다. 

</details>
<br>

<details>
<summary>Exception 클래스의 예시를 말해주세요.</summary>

크게 Checked Exception, Unchecked Exception으로 나눌 수 있다.

- Checked Exception: IOException, FileNotFoundException
- Unchecked Exception: NullPointerException, IllegalArgumentException, IllegalStateException, IndexOutOfBoundException, ClassCastException, ArithmeticException

</details>
<br>

<details>
<summary>Checked Exception과 Unchecked Exception의 차이는 무엇인가요?</summary>

- **Checked Exception**
  - Compile Exception이라고도 하며, Exception을 바로 상속 받는다.
  - 컴파일 단계에서 명시적으로 예외 처리를 해야 한다. 
- **Unchecked Exception**
  - RuntimeException을 상속 받는다. 
  - 컴파일 단계에서 명시적인 예외 처리를 강제하지 않는다. 
  - 컴파일 타임에 예외의 발생 여부를 예측할 수 없다. 

<img width="600" src="https://github.com/user-attachments/assets/8eb03534-ee04-4cad-8dd3-9ea283e02370"/>

</details>
<br>

<details>
<summary>throw와 throws의 차이는 무엇인가요?</summary>

- throw: 개발자가 의도적으로 예외를 발생시키고 싶을 때 사용
- throws: 현재 메서드에서 발생한 예외에 대한 처리를, 호출자 메서드한테 위임하고 싶을 때 사용 

```java
public class File {
    public File(String pathname) {
        if (pathname == null) {
            throw new NullPointerException();
        }
        this.path = fs.normalize(pathname);
        this.prefixLength = fs.prefixLength(this.path);
    }
}
```

```java
public class FileOutputStream {
    public FileOutputStream(String name) throws FileNotFoundException {
        this(name != null ? new File(name) : null, false);
    }
}
```

</details>
<br>

<details>
<summary>try~catch~finally 구문에서 finally은 어떠한 역할을 하나요?</summary>

예외가 발생하든 안하든 항상 실행되어야 하는 블록이다. close() 같은 메서드로 사용했던 자원을 해제하는 동작이 대표적이다. 

</details>
<br>

<details>
<summary>Throwable과 Exception의 차이는 무엇인가요?</summary>

- Throwable: 모든 Error, Exception에 대한 부모 클래스. printStackTrace(), getMessage() 같은 메서드 제공 
- Exception: Error를 제외한 모든 Exception에 대한 부모 클래스

</details>
<br>

<details>
<summary>제네릭이란 무엇이고, 왜 사용할까요?</summary>

제네릭(generic)이란, **데이터 타입을 일반화**하여 **코드의 재사용성**을 높이고 **타입 안정성**을 보장하는 기능이다. 

장점 

- 비슷한 기능을 하는 클래스, 인터페이스, 메서드 **코드의 재사용**이 가능해진다. 
- 지정한 범위에 맞지 않는 데이터 타입이 들어오면, 컴파일 단계에서 에러가 발생하므로 **타입 안전성**이 강화된다. 
- 객체를 사용하기 전에 타입 캐스팅을 하지 않아도 돼서 **코드 가독성**이 향상된다. 

제약 조건 

- **원시 타입 사용 불가** (참조 타입에서만 사용 가능)
- **런타임에 타입 정보 소실** 
- **정적 멤버(필드, 메서드)에 사용 불가** (제네릭 타입은 인스턴스가 생성되는 시점에 타입이 결정되므로)

</details>
<br>

<details>
<summary>제네릭을 사용한 경험을 소개해 주세요.</summary>

UiState.Success 데이터 클래스의 data 속성을 제네릭 타입으로 일반화하여, 코드의 재사용성을 높였습니다. 

```kotlin 
sealed interface UiState<out T> { 
    object Empty : UiState<Nothing>

    object Loading : UiState<Nothing>

    data class Success<T>(
        val data: T
    ) : UiState<T>

    data class Failure(
        val msg: String
    ) : UiState<Nothing>
}
```

액티비티 간의 전환을 위해 사용되는 메서드에 제네릭을 적용하여, 코드의 재사용성과 타입 안전성을 높였습니다. 

```kotlin 
private inline fun <reified T : Activity> navigateTo() {
    Intent(this@LoginActivity, T::class.java).apply {
        flags = Intent.FLAG_ACTIVITY_CLEAR_TASK or Intent.FLAG_ACTIVITY_NEW_TASK
        startActivity(this)
    }
}
```

ArrayList 대신 ArrayList<String>을 사용하여, 런타임에 발생할 수 있는 ClassCastException을 방지했습니다. 

</details>
<br>

## 람다, 스트림, 어노테이션, 리플렉션

<details>
<summary>람다란 무엇인가요?</summary>

람다식은 **메서드를 하나의 식으로 표현한 것**이다. 람다식은 함수의 이름이 없기 때문에 **익명 함수**라고 부르며, 마치 변수처럼 **메서드의 매개변수나 리턴값으로 사용**될 수 있다. 

</details>
<br>

<details>
<summary>스트림이란 무엇인가요?</summary>

스트림은 **컬렉션의 요소를 하나씩 참조하며 람다식으로 처리**할 수 있게 해주는 API이다. (Java 8 버전부터 제공)

</details>
<br>

<details>
<summary>람다와 스트림은 왜 생겨났을까요?</summary>

- **함수형 프로그래밍으로 코드의 가독성, 유지보수성 향상**
  - 기존에는 반복문, 조건문으로 어떻게 할지 일일이 명령했다면, 스트림은 무엇을 할지 **선언**만 하면 된다. (HOW -> WHAT) 
  - filter, map, sort, forEach 같은 연산자를 **체이닝** 해서 간결하게 코드를 작성할 수 있다. 
- **병렬 처리 지원** 
  - 데이터의 흐름을 나눠서 **멀티 스레드로 병렬 처리**하고, 그 결과를 다시 합치는 연산을 통해 **대량의 데이터를 효율적으로 처리**할 수 있다. 

</details>
<br>

<details>
<summary>자바에서 어노테이션이란 무엇일까요?</summary>

자바 소스코드에 추가할 수 있는 **일종의 메타 데이터**로, 컴파일 타임이나 런타임에 **특정 코드에 필요한 추가적인 처리**를 해준다. 

일반적으로 클래스, 인터페이스, 메서드, 변수, 매개변수 등에 사용된다. 

</details>
<br>

<details>
<summary>어노테이션을 왜 사용할까요?</summary>

- **컴파일 타임 검증**: javac 컴파일러에 포함된 어노테이션 프로세서는, 어노테이션을 기반으로 프로그램 소스코드의 오류를 검사한다. 
- **코드 생성**: 비슷한 형태로 반복되는 보일러 플레이트 코드를 자동으로 생성해준다.  
- **리플렉션**: 리플렉션을 통해 런타임에 특정 클래스나 메서드의 어노테이션 정보를 조회하고 그에 따른 동작을 수행할 수 있다. 

</details>
<br>

<details>
<summary>어노테이션의 종류는 무엇이 있나요?</summary>

**Built-in annotation**
- 자바에 기본적으로 내장된 어노테이션
- @Override, @Deprecated, @SupressWarning, @NonNull, @FuntionalInterface, @Native 

**Meta annotation**
- 어노테이션을 위한 어노테이션 
- @Target: 어노테이션 적용 가능한 대상 지정 ex) METHOD, PARAMETER, PACKAGE 등 
- @Retention: 어노테이션의 유지 기간 지정 (SOURCE, CLASS, RUNTIME)
- @Documented: 어노테이션을 javadoc으로 작성한 문서에 포함 
- @Inherited: 어노테이션을 자식 클래스에 상속 

**Custom annotation**
- 개발자가 직접 정의한 어노테이션 

```java
import javax.inject.Qualifier

@Qualifier // 여러 구현체 중에 특정 구현을 구분하기 위해 사용 
@Retention(AnnotationRetention.BINARY) // 어노테이션을 컴파일 된 바이너리 파일에 포함 (런타임에는 유지 X)
annotation class Logging

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class Auth
```

어노테이션이 저장하는 Element 개수에 따라 Marker(0개), Single-value(1개), Full annotation(2개 이상)으로도 구분할 수 있다.  

</details>
<br>

<details>
<summary>어노테이션은 리플렉션으로 동작한다고 말씀해 주셨는데, 리플렉션은 무엇인가요?</summary>

**정의** 

런타임에 클래스 인스턴스를 생성하고, 접근 제어자와 상관없이 동적으로 필드와 메서드에 접근할 수 있게 해주는 API

**사용하는 이유** 

규모가 작은 프로젝트에서는 컴파일 단계에서도 프로그램에 사용될 객체와 그들의 의존 관계를 모두 파악할 수 있다. 

그러나, 프레임워크 같이 큰 규모의 개발 단계에서는 수많은 객체와 그들의 의존 관계를 파악하기 어렵다. 

이때 리플렉션을 사용하면 동적으로 클래스 인스턴스를 만들어서 의존 관계를 맺고, 필요한 메서드나 필드에 접근할 수 있다. 

**단점** 

- 캡슐화를 저해한다. 
- 런타임에 인스턴스를 생성하므로, 컴파일 타임에 해당 타입을 체크할 수 없다. 
- 런타임에 인스턴스를 생성하므로, 구체적인 동작 흐름을 파악하기 어렵다. 
- 단순히 필드 및 메서드에 접근할 때보다 상대적으로 성능이 느리다. 

</details>
<br>

## 심화 질문

<details>
<summary>`System.out.println` 메서드는 성능이 좋지 않다고 하는데 이유가 무엇일까요?</summary>

prinln 메서드는 **블로킹 I/O**이므로 메서드를 호출한 스레드는 작업이 끝날 때까지 다른 작업을 수행할 수 없다. 

그리고 메서드 내부에 synchronized 블록으로 **동기화 처리**가 되어 있어서, 멀티 스레드 환경에서 Thread-Safe 하지만 

한 스레드가 실행 중일 때 다른 스레드는 블로킹 된다는 성능 이슈가 있다. 

그외의 단점은 다음과 같다. 

- **로그 레벨 지정 불가**
  - 로그 레벨을 지정할 수 없으므로, 프로덕션 버전에서도 불필요한 디버깅 정보가 출력되어 시스템의 보안과 성능에 악영향을 줄 수 있다. 
- **유지보수성 저하**
  - 출력 메시지가 하드코딩 되어 있으면, 나중에 메시지를 수정하기 번거롭다. 

</details>
<br>

## 참고자료 

문자열, 예외, 제네릭 

- https://madplay.github.io/post/java-string-literal-vs-string-object
- https://www.baeldung.com/java-string-pool
- https://12bme.tistory.com/42
- https://hbase.tistory.com/115
- https://toneyparky.tistory.com/40
- https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%97%90%EB%9F%ACError-%EC%99%80-%EC%98%88%EC%99%B8-%ED%81%B4%EB%9E%98%EC%8A%A4Exception-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC
- https://st-lab.tistory.com/153

람다, 스트림, 어노테이션

- https://mangkyu.tistory.com/112
- https://www.elancer.co.kr/blog/detail/255
- https://velog.io/@eia51/Annotation-완전-정복기
- https://steady-coding.tistory.com/609
- https://systemdata.tistory.com/21
- https://medium.com/@ans188/kotlin-reflection%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C-f2a23b7e6fae