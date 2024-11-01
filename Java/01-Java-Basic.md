# 자바 기본

## 면접 예상 질문

<details>
<summary>Java의 특징에 대해서 설명해주세요.</summary>

- **높은 이식성**
  - 자바의 바이트 코드는 **JVM의 인터프리터**에 의해 기계어로 번역되어, **플랫폼 독립적으로 실행**될 수 있다. 
  - "Write Once, Run Anywhere" 한번 작성된 자바 코드는 어떤 플랫폼에서든 동일하게 실행시킬 수 있다. 
- **안전성**
  - JVM의 **가비지 컬렉터**가 개발자를 대신하여 메모리를 관리해주기 때문에, 메모리 누수를 방지할 수 있다. 
  - 컴파일 타임에 변수의 타입이 결정되는 **정적 언어**이므로, 타입 안정성을 높일 수 있다. 
- **객체지향 언어**
  - 추상화, 상속, 다형성, 캡슐화 같은 객체지향 프로그래밍으로 **코드의 재사용성, 유지보수성**을 높일 수 있다. 
- **멀티 스레드 지원**
  - 여러 스레드를 동시에 실행시키는 멀티 스레드를 지원하여, **빠른 속도로 작업을 처리**할 수 있다. 

</details>

<br>

<details>
<summary>Java의 단점에 대해서 설명해주세요.</summary>

- **느린 성능** 
  - 프로그램을 실행할 때마다 인터프리터에 의해 한줄씩 번역되기 때문에, 실행 속도가 느린 편이다. (JIT 컴파일러로 성능 최적화 가능)
- **높은 메모리 사용률** 
  - 더 이상 참조되지 않는 메모리를 가비지 컬렉션 하는 과정에서, 추가적인 메모리와 CPU 리소스가 소모된다. 
  - 대부분의 데이터가 객체로 생성되고, 객체마다 추가적인 메타데이터를 포함하고 있어 메모리 사용률이 증가한다. 
- **장황한 코드량** 
  - 다른 함수형 프로그래밍 언어에 비해, 코드 길이가 길고 문법이 복잡한 편이다. 
- **정교한 메모리 제어의 어려움**
  - 가비지 컬렉션이 메모리를 관리하기 때문에, 개발자가 직접 제어하기는 어렵다. (C++의 포인터 개념이 없다.)

</details>

<br>

<details>
<summary>Java 프로그램의 실행 과정에 대해서 설명해주세요.</summary>

- **소스 코드 작성 (.java 파일)**
- **컴파일 (.class 파일)**
  - 소스 코드를 javac 컴파일러로 컴파일 하면, 바이트코드가 포함된 .class 파일 생성 
- **JVM 로드 및 클래스 로딩** 
  - JVM에 .class 파일 로드 
  - 클래스 로더가 프로그램 실행에 필요한 클래스를 메모리에 로드
- **바이트코드 실행** 
  - 초기에는 **인터프리터**가 바이트 코드를 한줄씩 번역하다가 
  - **JIT(Just-In-Time) 컴파일러**가 자주 사용되는 코드를 캐싱하여 성능 최적화 
- **메모리 관리**
  - JVM은 힙 영역에 객체를 동적으로 생성하며, 더 이상 참조되지 않는 객체는 가비지 컬렉터로 메모리 회수 
- **프로그램 종료**

</details>

<br>

<details>
<summary>Java Bytecode에 대해서 설명해주세요.</summary>

**소스 코드를 컴파일 하면 생성되는 중간 코드**로, 하드웨어, 운영체제 같은 **플랫폼에 독립적으로 실행**될 수 있다. 

</details>

<br>

<details>
<summary>Java의 인터프리터(interpreter) 방식과 JIT 컴파일(compile) 방식에 대해서 설명해주세요.</summary>

- 인터프리터 방식: 바이트 코드를 한줄씩 번역하며 즉시 실행시키는 방식으로, 프로그램 실행 초기에 빠른 응답성을 보장할 수 있다. 
- JIT 컴파일 방식: 자주 사용되는 코드를 미리 컴파일하여 캐싱해두는 방식으로, 인터프리터의 느린 실행 속도를 개선할 수 있다. 

</details>

<br>

<details>
<summary>사용해본 Java 버전과 특징 그리고 왜 그 버전을 사용했는지 설명해주세요.</summary>

안드로이드 프로젝트를 개발할 당시에, 가장 최신 LTS 버전이었던 17을 사용했습니다. 

LTS (Long-Term Support) 버전은 장기간 지원되는 버전을 의미합니다. (최대 6~8년)

</details>

<br>

<details>
<summary>Java 8, 11, 17 버전에 대해 아는대로 설명해주세요.</summary>

- **Java 8**
  - 2014년 출시, LTS 버전 (~ 2030.12 지원)
  - 람다 표현식, 스트림 API 제공 
  - java.time 패키지에 새로운 API 제공 
  - 인터페이스의 디폴트 메서드 지원 
- **Java 11**
  - 2018년 출시, LTS 버전 (~ 2032.01 지원)
  - Open JDK와 Oracle JDK가 통합되고, Oracle JDK가 구독형 유료 모델로 전환 
  - var 키워드로 지역 변수의 타입 추론 가능 (전역 변수는 불가)
  - HTTP 클라이언트 API 표준화 (버전 11 이전에는 타사 HTTP 라이브러리에 의존해야 했음.)
  - String, File 클래스에 새로운 메서드 추가 
- **Java 17**
  - 2021년 출시, LTS 버전 (~ 2029.09 지원)
  - Spring Boot 3.0에서 요구하는 최소 자바 버전 
  - sealed class 제공 (자식 클래스의 종류 제한)
  - record class 제공 (코틀린의 데이터 클래스)
  - Incubator: JNI(Java Native Interface)보다 성능이 좋고, 안전하게 외부 네이티브 함수 호출 가능
  - instanceof, switch 사용 편의성 증가 
- **Java 21** 
  - 2023년 출시, LTS 버전(~ 2031.09 지원)
  - Spring Boot 3.2부터 지원
  - Java 플랫폼에 경량의 가상 스레드 도입
  - UTF-8 기본값으로 사용

</details>

<br>

<details>
<summary>JDK와 JRE에 대해서 설명해주세요.</summary>

<img width="600" src="https://github.com/user-attachments/assets/74b6dd5b-68e9-4df4-87c1-f36f9dd7ba65"/> 

### JDK (Java Development Kit)

- **자바 개발 도구 모음 (자바 개발을 위한 SDK 집합)**
- SDK (Software Development Kit): 하드웨어 플랫폼, OS, 프로그래밍 언어 제작사에서 제공하는 도구 
- 자바 개발에 필요한 **라이브러리**, javac, javadoc 등의 **개발 도구**, 자바 실행을 위한 **JRE 포함** 
- JDK 종류 
  - Oracle JDK: 오라클에서 제공하는 JDK로 유료 라이선스 구독 필요 
  - Open JDK: 가장 유명한 무료 JDK 
  - Azul Zulu: 인지도가 높은 JDK 중 하나로, Mac에서 사용 가능한 바이너리 제공 
  - Amazon Corretto: AWS에서 제공하는 JDK로, AWS 환경에서 쉽게 사용 가능 
  - Temurin (Adopt Open JDK): Eclipse에서 제공하는 JDK 

### JRE (Java Runtime Environment)

- **JVM과 자바 어플리케이션 실행에 필요한 라이브러리 등을 묶어서 배포한 것** 
- JRE는 **기본적으로 JDK에 포함**되어 있으며, 기존에는 JDK와 별도로 설치 가능했으나 JDK 11부터는 따로 제공하지 않음.

### JVM (Java Virtual Machine)

- **자바 가상 머신 (자바를 실행시키는 프로그램)**
- **자바로 작성된 모든 프로그램은 JVM 위에서만 실행 가능하다.** 이를 통해 자바는 플랫폼 독립적으로 실행할 수 있다. 
- 하지만 **JVM 자체는 OS에 종속적**이므로, 각 OS에 맞는 JVM이 필요하다. 

</details>

<br>

<details>
<summary>동일성과 동등성에 대해 설명해 주세요.</summary>

- 동일성 (identity): 객체에 할당된 메모리 주소가 같은지 판별 
- 동등성 (equality): 객체의 내용이 같은지 판별 
- 동일하면 동등하지만, 동등하다고 동일하진 않다. (내용만 같고, 메모리 주소는 다를 수 있으므로)

</details>

<br>

<details>
<summary>equals()와 ==의 차이점은 무엇일까요?</summary>

- `==` : 동일성 판별에 사용 
- `equals()` : 동등성 판별에 사용 
- `equals()`는 내부적으로 `==` 연산자와 같은 로직이므로, 객체의 특성에 맞게 오버라이딩을 해줘야 동등성 기능을 수행한다. 

</details>

<br>

<details>
<summary>HashCode를 설명하고, equals()와 hashCode()의 차이점에 대해 설명해주세요.</summary>

- HashCode: 해싱은 **해시 함수를 사용하여 가변 길이의 입력 값을 고정 길이의 출력 값으로 변환하는 과정**을 의미한다. 해싱으로 얻은 값을 **해시 코드**라고 한다.
- `equals()` : 객체의 **내용이 같은지 비교**할 때 사용 (따로 오버라이딩 하지 않으면 == 연산자처럼 동작하므로, 동등성 비교를 위해서는 오버라이딩 필수)
- `hashCode()` : 힙 메모리에 할당된 **객체의 주소에 대한 해시 코드** 반환 (각 객체마다 고유한 값을 가진다.)

</details>

<br>

<details>
<summary>왜 equals() 외에 hashCode()도 재정의해야 하나요?</summary>

hashCode()를 재정의하지 않으면, 객체의 메모리 주소를 비교하기 때문이다. 

객체의 내용이 같은지 동등성을 판별하려면, hashCode()를 재정의해야 한다. 

</details>

<br>

<details>
<summary>toString()에 대해서 설명해주세요.</summary>

자바에서 모든 클래스의 최상위 클래스인 Object에는 `toString()` 메서드가 다음과 같이 정의되어 있다. 

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

따라서, 별도로 오버라이딩 하지 않으면 `MyObject@251a69d7` 같은 문자열이 출력된다. 

좀 더 유의미한 객체 정보를 출력하려면, 다음과 같이 메서드를 오버라이딩 해줘야 한다. 

```java
class Person {
    String name;
    int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return String.format("이름 : %s, 나이 : %d세", this.name, this.age);
    }
}

public class Main {
    public static void main(String[] args) {
        Person p1 = new Person("홍길동", 54);
        System.out.println(p1); // 이름 : 홍길동, 나이 : 54세
    }
}
```

```java
import java.util.Arrays;

class MyInt {
    final int num;

    MyInt(int num) {
        this.num = num * 100;
    }

    @Override
    public String toString() {
        return Integer.toString(num);
    }
}

public class Main {
    public static void main(String[] args) {
        Object[] arr = new Object[5];
        arr[0] = new MyInt(1);
        arr[1] = new MyInt(2);
        arr[2] = new MyInt(3);
        arr[3] = new MyInt(4);
        arr[4] = new MyInt(5);

        System.out.println(Arrays.toString(arr)); // [100, 200, 300, 400, 500]
    }
}
```

</details>

<br>

<details>
<summary>자바에서 메인 메서드는 왜 static으로 되어 있을까요?</summary>

static은 **정적인, 고정된**이라는 뜻을 가지고 있다. 즉, static 키워드로 변수나 메서드를 정의하면, 이는 메모리에 '고정'되어 **여러 객체가 공유**할 수 있게 된다. 

정적 변수와 정적 메서드는 인스턴스 없이도, **클래스가 메모리에 로드되면 바로 사용할 수 있다**는 점에서 **클래스 멤버**라고도 불린다. 

이들은 **프로그램이 종료되기 전까지 사용 가능**하며, 가비지 컬렉션의 대상이 되지 않는다. (단, 정적 객체는 더 이상 참조되지 않으면 GC에 의해 수집될 수 있다.)

자바의 **main 메서드는 프로그램이 시작될 때 JVM에 의해 실행되며, 어플리케이션의 진입점**이라고 볼 수 있다. 이때 main 메서드는 static으로 선언되어 있기 때문에, **별도의 인스턴스 생성 없이도 메서드 호출이 가능**하다.

만약 main 메서드가 static으로 선언되어 있지 않다면, 매번 프로그램을 실행할 때마다 인스턴스 생성이 필요하므로 리소스가 더 들었을 것이다. 

</details>

<br>

<details>
<summary>상수(Constant)와 리터럴(Literal)에 대해서 설명해주세요.</summary>

- 상수 : **변하지 않는 값**, `final` 키워드로 정의 
- 리터럴 : **특정 자료형의 값 자체**를 나타내는 표현, **변수나 상수에 실제로 할당된 값** 
- 프로그램 개발 시, 특정 자료형의 리터럴을 그대로 사용하는 것보다 상수로 정의하는 게 더 좋다. 
- 상수 이름을 통해 리터럴의 의미 파악이 더 수월해지고, 비즈니스 요구사항에 따라 상수 값을 변경해야 할 때도 수고가 덜 들기 때문이다.

</details>

<br>

<details>
<summary>Primitive Type과 Reference Type에 대해서 설명해주세요.</summary>

- Primitive Type (원시 타입): **값 자체를 저장하는 자료형**
  - 정수형: byte, short, int, long 
  - 실수형: float, double
  - 문자형: char
  - 논리형: boolean 
  - 고정된 크기의 메모리 사용 (자료형에 따라 1, 2, 4, 8 바이트)
  - 제네릭 타입에 사용 불가 
- Reference Type (참조 타입): **객체의 메모리 주소를 저장하는 자료형** 
  - 클래스, 인터페이스, 배열 (String, Runnable, int[] 등)
  - 실제 값이 아니라, 힙 영역에 할당된 객체의 메모리 주소를 저장한다. 
  - 초기화 하지 않으면, 기본값은 null 
  - 제네릭 타입에 사용 가능 

</details>

<br>

<details>
<summary>Java는 Call by Value 일까요? 아님 Call by Reference 일까요?</summary>

자바는 **모든 인수를 값으로 전달하기 때문에 Call by Value** 방식이다. 

Reference Type도 참조 자체를 전달하는 게 아니라, **객체의 주소를 복사하여 전달**하는 방식이므로 

주소 참조를 통해 객체의 속성을 변경할 수는 있지만, **참조 변수 자체를 변경해도 원본에는 영향을 미치지 않는다.** 

```java
class User {
    public int age;

    public User(int age) {
        this.age = age;
    }
}

public class ReferenceTypeTest {
    @Test
    void test() {
        User a = new User(10);
        User b = new User(20);

        // Before
        assertEquals(a.age, 10);
        assertEquals(b.age, 20);

        // 호출자의 argument를 복사하여, 수신자의 parameter가 만들어진다.
        // 원시 타입: 값이 같을 뿐, 서로 독립적인 변수이다. 
        // 참조 타입: 객체의 주소를 복사하므로, 서로 같은 객체를 가리킨다. 
        modify(a, b);

        // After
        assertEquals(a.age, 11);
        assertEquals(b.age, 20);
    }
    
    private void modify(User a, User b) {
        // test 함수에서 정의한 객체 a의 속성을 변경한다.
        a.age++;

        // 변수에 새로운 객체를 할당하면, 원본 객체에는 영향을 미치지 않는다. 
        b = new User(30);
        b.age++;
    }
}
```

</details>

<br>

<details>
<summary>Java 직렬화(Serialization)에 대해서 설명해주세요.</summary>

- **직렬화** (Serialization): **객체를 바이트 스트림으로 변환하는 과정** (객체를 파일 또는 데이터베이스에 **저장**하거나, 네트워크로 **전송**할 때 필요)
- **역직렬화** (Deserialization): **바이트 스트림을 다시 원래의 객체로 복원하는 과정** 
- 자바에서 직렬화를 수행하려면, 해당 클래스가 `java.io.Serializable ` 인터페이스를 구현해야 한다. 
- 보안을 위해 직렬화 한 데이터를 암호화 할 수 있다. (역직렬화 하기 전에는 복호화 과정이 필요하다.)
- 안드로이드에서는?
  - 도메인 레이어에서는 안드로이드에 의존성을 갖지 않는 Serializable 사용
  - UI 레이어에서 Intent로 객체를 전달할 때는 Parcelable 사용
  - Serializable도 직접 직렬화 로직을 작성하면, 리플렉션을 수행하지 않아서 Parcelable과 성능이 비슷하거나 더 좋아질 수 있음. 

</details>

<br>

# 자바 객체 지향

## 면접 예상 질문

<details>
<summary>오버로딩과 오버라이딩의 차이는 뭔가요?</summary>

||오버로딩 (Overloading)|오버라이딩 (Overriding)|
|---|---|---|
|정의| **이름이 같고, 매개변수의 개수나 타입이 다른 메서드를 정의하는 것** | **부모 클래스의 메서드를 자식 클래스에서 재정의하는 것** |
|언제 사용| 동일한 기능의 메서드를 하나의 이름으로 사용하고 싶을 때 (중복 코드 제거) | 부모 클래스의 동작을 자식 클래스에서 다르게 정의하고 싶을 때 |
|메서드명| 동일해야 함. | 동일해야 함. |
|매개변수| 달라야만 함. | 동일해야 함. |
|리턴 타입| 다를 수 있음. (리턴 타입만 다르면 오버로딩 X) | 동일해야 함. |
|접근 제어자| 제한 없음. | 부모 클래스의 메서드보다 더 넒은 범위의 접근 제어자만 가능 |
|적용 범위| **같은 클래스** 내에서 적용 | **상속 관계**에 적용 |

</details>

<br>

<details>
<summary>다형성이 무엇이고, 왜 필요할까요?</summary>

다형성(Polymorphism)이란, 이름 그대로 해석하면 '형태가 다양하다'는 뜻이다. 

객체지향 프로그래밍에는 정적 다형성과 동적 다형성이 있다. 

- **정적 다형성**
  - **컴파일 타임**에 결정되는 다형성으로, 대표적으로 **메서드 오버로딩**이 있다.
  - **이름이 같은 메서드가 매개변수의 개수나 타입에 따라 다르게 동작**하는 것을 의미한다.
- **동적 다형성**
  - **런타임**에 결정되는 다형성으로, **상속과 메서드 오버라이딩**으로 구현할 수 있다.
  - 부모 클래스의 메서드를 자식 클래스에서 오버라이딩 한 다음에, 부모 클래스 타입에 자식 클래스의 객체를 대입하면 (업 캐스팅)
  - **런타임에 실제로 가리키고 있는 객체가 무엇인지에 따라 메서드의 동작이 달라진다.**

이러한 다형성을 활용하면, **코드의 중복을 줄이고 변경과 확장에 유연한 코드를 작성**할 수 있기 때문에 객체지향의 핵심이라고 볼 수 있다.

</details>

<br>

<details>
<summary>상속은 무엇인가요?</summary>

객체지향에서 상속이란, **부모 클래스의 필드와 메서드를 자식 클래스에 물려주는 것**을 의미한다. 

부모 클래스를 상속 받은 자식 클래스는, **부모 클래스의 메서드를 재정의하거나 완전히 새로운 기능을 추가**할 수 있다. 

이처럼 상속은 **다형성**을 구현할 수 있게 해주며, 중복되는 코드를 부모 클래스로 추상화하여 **코드의 재사용성**도 높일 수 있다. 

</details>

<br>

<details>
<summary>상속의 단점은 무엇이 있을까요?</summary>

- 자식 클래스가 부모 클래스의 필드와 메서드를 그대로 사용하면서, **부모 클래스에 대한 결합도가 높아진다.** 즉, 자식 클래스는 부모 클래스에 의존하는 수동적인 객체가 되어버리고, 변경에 유연하게 대처하기 어려워진다. 
- 잘 정의된 부모 클래스의 메서드를 자식 클래스에서 임의로 재정의하면, 부모 클래스의 **캡슐화가 깨진다.**
- 부모 클래스의 메서드를 오버라이딩 할 때, 기존 구현에 문제가 없는지 확인하는 과정 자체에서 캡슐화가 깨진다. ex) 정사각형과 직사각형 예시 
- 부모 클래스에 결함이 있다면, 자식 클래스도 해당 **결함을 그대로 넘겨 받게 된다.** ex) Vector를 상속 받는 Stack 클래스 

</details>

<br>

<details>
<summary>상속과 조합의 차이에 대해 설명해 주세요.</summary>

- 상속 (Inheritance) 
  - **자식 클래스가 부모 클래스의 필드와 메서드를 물려 받는 것** 
  - **IS-A** 관계 ex) Dog는 Animal이다. 
  - 장점: 메서드 오버라이딩으로 다형성 구현, 코드의 재사용성 증가 
  - 단점: 클래스 간 **결합도** 증가, **캡슐화** 저해 
- 조합 (Composition)
  - 한 클래스가 다른 클래스의 **객체를 필드로 포함**하여, 필요할 때 **해당 객체의 기능을 호출**하는 것 
  - **HAS-A** 관계 ex) Car는 Engine을 가진다. 
  - 클래스 간 **결합도** 감소 -> 코드의 유연성, 확장성 증가 
  - 클래스의 **캡슐화** 보장 

```java
// 상속 코드 예시 
public abstract class Man {
    public void move() {
        System.out.println("걷는다");
    }

    public void eat() {
        System.out.println("먹는다");
    }
    
    public boolean canTouchKryptonite(){
        return true;
    }
    
    public abstract void attack();
}

// 슈퍼맨은 외계인이 된 상태
class SuperMan extends Man {
    public void fly() {
        System.out.println("날아간다.");
    }
    
    @Override
    public boolean canTouchKryptonite(){
        return false;
    }
    
    @Override
    public void attack() {
    // 대충 공격 어떻게 한다는 뜻.
    }
}
```

```java
// 조합 코드 예시 
public class Man {
    public void move() {
        System.out.println("걷는다");
    }

    public void eat() {
        System.out.println("먹는다");
    }
}

class SuperMan {
    private final Man man = new Man();

    public void move() {
        man.move();
    }

    public void eat() {
        man.eat();
    }

    public boolean canTouchKryptonite(){
        return false;
    }

    public void fly() {
        System.out.println("날아간다.");
    }
    
    public void attack() {
    // 대충 공격한다는 뜻.
    }
}
```

</details>

<br>

<details>
<summary>instanceof 키워드란 무엇인가요?</summary>

객체가 **특정 클래스나 인터페이스의 인스턴스인지 확인**하기 위해 사용한다. 

</details>

<br>

<details>
<summary>instanceof 키워드를 사용할 때 문제점으로 무엇이 있을까요?</summary>

![image](https://github.com/user-attachments/assets/abfd63d5-2265-4e12-9be0-f8acfe922cab)

### 다형성 활용 (상속, 메서드 오버라이딩)

```java
public abstract class Piece {
    public abstract int calculate(int point);
}

public class King extends Piece {
    public int calculate(int point) {
        return point + 10;
    }
}

public class Pawn extends Piece {
    public int calculate(int point) {
        return point + 1;
    }
}

public class Empty extends Piece {
    public int calculate(int point) {
        return point;
    }
}

public class Point {
  public int calculate(Piece p, int point) {
    return p.calculate(point);
  }
}
```

### instanceof 활용 

```java
public class Point {
    public int calculate(Piece p, int point) {
        if(p instanceof King) {
            return point + 10;
        } else if(p instanceof Pawn) {
            return point + 1;
        } else if(p instanceof Empty) {
            return point;
        }
    }
}
```

instanceof 사용 시 발생할 수 있는 문제점은? 

- **캡슐화 저해**
  - calculate 함수에서 여러 자식 객체가 **불필요하게 외부 객체의 동작까지 알게 되면서** 캡슐화가 깨진다. 
- **OCP(Open Closed Principle) 위배**
  - Piece를 상속하는 Queen 객체가 추가로 생긴다고 가정해보자. 
  - 다형성을 활용하면, Queen 클래스 내부에서 calculate 메서드를 구현하면 되는데 
  - **instanceof는 해당 연산자가 사용되는 모든 메서드를 찾아서 고쳐야 한다.**
  - **확장에는 열려있고, 수정에는 닫혀 있어야 한다는 OCP에 위배**되는 것이다. 
- **SRP(Single Reponsibility Principle) 위배**
  - instanceof는 인스턴스의 타입을 알아내고, 해당 타입의 동작을 실행시키기 위해 사용된다.
  - 결국 instanceof를 사용하는 calculate 함수는, **각 자식 객체의 calculate 구현을 모두 알고 있어야 하는 책임이 부가**된다. 
  - **한 클래스는 하나의 책임만 가져야 한다는 SRP에 위배**되는 것이다. 
- **인스턴스 타입 매칭에 대한 성능 감소**
  - 동적 다형성을 활용하면, 런타임에 실제로 가리키고 있는 객체 타입을 찾아서 그에 맞는 구현을 실행한다.
  - **instanceof의 경우, 컴파일 타임에 알맞은 객체 타입을 찾을 때까지 모든 타입을 검사해야 한다.** 
  - 성능 면에서도 instanceof 연산자보다 다형성을 활용하는 것이 더 좋다. 

</details>

<br>

<details>
<summary>interface와 abstract class는 어떤 차이가 있나요?</summary>

우선, 공통점은 다음과 같다.

- **추상 메서드의 구현을 강제한다.** (추상 메서드는 선언부만 있고, 구현부가 없는 미완성 메서드)  
- **인스턴스화 불가** (new 연산자 사용 불가)

|| abstract class | interface |
|---|---|---|
|비유| 일부만 구현된, **미완성 설계도** | 밑그림만 그려진, **기본 설계도** |
|사용 가능 변수| 제한 없음. | static final (상수) |
|사용 가능 메서드| 제한 없음. | - abstract <br> - default, static (Java 8 버전부터) <br> - private (Java 9 버전부터) |
|사용 가능 접근 제어자| 제한 없음. (public, private, protected, default) | public  |
|다중 상속| 불가능 | 가능 (인터페이스 간의 다중 상속, 클래스에서 인터페이스의 다중 구현) |

</details>

<br>

<details>
<summary>언제 interface 사용하고, 언제 abstract class 사용 하나요?</summary>

- abstract class: 여러 클래스 간의 중복 코드를 공유하고 싶을 때 
- interface: 객체의 행동 규약을 정의하고 싶을 때 

</details>

<br>

<details>
<summary>final 키워드에 대해 설명해 주세요.</summary>

변수, 메서드, 클래스를 **변경 불가능**하게 만들기 위해 사용한다. 

- final 변수: 값이 변하지 않는 상수
- final 메서드: 오버라이딩 불가 
- final 클래스: 상속 불가 

</details>

<br>

# 참고 자료 

자바 기본 

- https://hyeinisfree.tistory.com/26
- https://juran-devblog.tistory.com/262
- https://github.com/jvm-hater/java-study
- https://coding-factory.tistory.com/524
- https://inpa.tistory.com/entry/
- https://june0122.github.io/2021/08/04/term-literal/
- https://bcp0109.tistory.com/360

자바 객체 지향 

- https://hyoje420.tistory.com/14
- https://tecoble.techcourse.co.kr/post/2020-10-27-polymorphism/
- https://tecoble.techcourse.co.kr/post/2021-04-26-instanceof/
- https://tecoble.techcourse.co.kr/post/2020-07-16-static-method/
- https://steady-coding.tistory.com/451
- https://myjamong.tistory.com/150

