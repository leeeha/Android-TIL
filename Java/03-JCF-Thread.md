## 컬렉션 기초

<details>
<summary>JCF란 무엇인가요?</summary>

**Java Collections Framework**의 약어로, **다수의 데이터를 쉽고 효과적으로 처리할 수 있는 표준화 된 방법을 제공하는 클래스들의 집합**을 의미한다. 

즉, 데이터를 저장하는 **자료구조**와 데이터를 처리하는 **알고리즘**을 구조화하여 클래스로 구현해놓은 것이다.

</details>
<br>

<details>
<summary>JCF의 계층 구조를 설명해 주세요.</summary>

JCF는 크게 Collection 인터페이스와 Map 인터페이스로 나뉜다.

<img width="400" src="https://github.com/user-attachments/assets/d9b939eb-977d-400e-9f08-0c0836d05a70"/>
<img width="500" src="https://github.com/user-attachments/assets/3160f51d-b079-49dd-84f1-8d0ae4b3322a"/> 

</details>
<br>

<details>
<summary>List 인터페이스는 무엇이고, 구현체의 종류는 무엇이 있나요?</summary>

**순서가 있는 데이터의 집합**으로, 데이터의 **중복을 허용**한다. List 인터페이스의 구현체로는 ArrayList, LinkedList, Vector(Legacy)가 있다. 

</details>
<br>

<details>
<summary>ArrayList에 대해 설명해 주세요.</summary>

**크기가 가변적인 선형 리스트**로, **인덱스**로 내부 요소를 관리한다는 점에서 **배열과 유사**하다. 

하지만, 크기를 변경할 수 없는 배열과 달리, ArrayList는 용량이 초과될 때 **크기를 동적으로 늘릴 수 있다.**

생성자의 인자로 별도 지정하지 않으면, ArrayList의 초기 용량은 10으로 설정된다. 

- 원소 접근: 인덱스를 사용하므로 O(1)
- 마지막에 원소 추가: O(1)
- 임의의 위치에 원소 추가: 원소들을 한칸씩 뒤로 밀어야 하므로 O(N)
- 임의의 원소 삭제: 원소들을 한칸씩 앞으로 당겨야 하므로 O(N)

인덱스를 기반으로 O(1)만에 원소에 접근할 수 있지만, 메모리 상에 원소들이 연속적으로 배치되어 있으므로 임의의 원소를 추가/삭제하는 연산은 성능이 느리다. 

</details>
<br>

<details>
<summary>ArrayList는 어떻게 동적으로 사이즈가 늘어나나요?</summary>

배열이 가득 차면 크기를 1.5배 늘린 새 배열을 메모리에 할당한다. 그리고 기존 배열의 모든 원소를 새 배열로 복사하므로, 시간복잡도는 O(N)이 든다. 

</details>
<br>

<details>
<summary>LinkedList에 대해 설명해 주세요.</summary>

각 노드가 **데이터와 포인터**를 갖고 연결되어 있는 자료구조다. 노드의 포인터는 자신의 이전 또는 다음 노드의 주소를 가리킨다. 

- 임의의 위치에 원소 추가: 노드 주소를 알고 있다면 O(1), 그렇지 않으면 시작 노드부터 따라가야 하므로 O(N)
- 임의의 원소 삭제: 노드 주소를 알고 있다면 O(1), 그렇지 않으면 시작 노드부터 따라가야 하므로 O(N)
- 원소 접근: 시작 노드부터 따라가야 하므로 O(N)

배열과 달리, 연결 리스트는 원소들이 메모리 상에 연속적으로 배치되어 있지 않기 때문에, **원소의 추가/삭제 연산이 더 효율적**이다. 

하지만, 인덱스라는 개념이 없어서 **임의의 원소에 접근할 때는 순차 탐색**을 해야 하므로 O(N)이 걸린다. 

</details>
<br>

<details>
<summary>언제 ArrayList를 사용하고, 언제 LinkedList를 사용할까요?</summary>

ArrayList는 접근 시간이 빠르지만 삽입/삭제 연산이 느리고, LinkedList는 접근 시간이 느리지만 삽입/삭제가 빠르다는 특징이 있다. 

따라서, 리스트의 원소를 삽입/삭제하는 연산이 많은 경우 LinkedList를 사용하고, 그렇지 않으면 ArrayList를 사용하는 게 좋다. 

</details>
<br>

<details>
<summary>ArrayList와 Vector는 어떠한 차이가 있나요?</summary>

Vector는 **동기화 되어 있는 ArrayList**로, 지금은 사용되지 않지만 **호환성을 위해 남겨둔 레거시 클래스**다. 

Vector의 모든 메서드는 동기화 되어 있기 때문에, 동기화가 필요하지 않은 상황에서는 성능이 떨어진다. 

그리고 JCF가 나오기 전에 설계된 클래스이므로 일부 구조가 JCF와 상응하지 않는다. ArrayList, LinkedList 같은 JCF 클래스가 등장하면서, Vector는 점차 대체되었다. 

</details>
<br>

<details>
<summary>Stack과 Queue가 무엇인가요?</summary>

- Stack: LIFO 기반의 자료구조. 레거시 클래스인 Vector를 상속 받고 있어서 오라클 문서에서도 Stack 대신 Deque 사용을 권한다. 
- Queue: FIFO 기반의 자료구조. Queue 인터페이스의 구현체로는 LinkedList, PriorityQueue가 있다. 
- Deque: Double-Ended Queue의 약어로, Queue의 양끝에서 추가/삭제 연산 가능. Deque 인터페이스의 구현체로는 ArrayDeque가 있다. 

</details>
<br>

<details>
<summary>Set이 무엇이고, 구현 클래스가 무엇이 있는지 설명해 주세요.</summary>

Set은 **순서가 없고 중복을 허용하지 않는** 자료구조로, 구현체로는 HashSet, LinkedHashSet, TreeSet이 있다. 

- **HashSet**: **해시 기반으로 구현한 Set**으로, 내부적으로 HashMap을 사용하고 있다. PRESENT는 map에 value를 넣기 위해 어쩔 수 없이 사용하는 더미 데이터로, HashSet은 HashMap의 key만 사용하고, value는 버리는 방식으로 구현되어 있다. (LinkedHashSet, TreeSet도 마찬가지)
- **LinkedHashSet**: HashSet를 상속 받았고, **원소가 추가된 순서대로 저장**된다. 
- **TreeSet**: SortedSet 인터페이스의 구현체로, **특정 정렬 기준**에 따라 원소의 순서를 정할 수 있다. 

<img width="600" src="https://github.com/user-attachments/assets/30bb203f-7abf-47dc-a0b8-a84e799e4f9c">

<img width="600" src="https://github.com/user-attachments/assets/cb0002ab-bd5d-474e-92a5-e732e334fcde">

</details>
<br>

<details>
<summary>Set에서 중복 요소를 어떻게 걸러내는지 설명해 주세요.</summary>

Set 인터페이스 내에 정의된 equals(), hashCode()를 사용한다. 

먼저, 객체를 추가하기 전에 hashCode()를 통해 **동일한 해시 코드를 가진 객체가 있는지** 검사한다. 

해시 코드가 동일하면, 그 다음으로 **equals() 메서드로 두 객체의 동등성을 검사**한다. 이때 **true가 나오면 동일한 객체로 판단**하고, 중복으로 저장하지 않는다. 

</details>
<br>

<details>
<summary>Map이 무엇이고, 구현 클래스가 무엇이 있는지 설명해 주세요.</summary>

**Key-Value 쌍을 저장하는 자료구조**로, Key의 중복을 허용하지 않으며, Value는 중복이 가능하다. 

Key를 통해 Value에 O(1)만에 접근할 수 있지만, 데이터의 순서는 보장하지 않는다. 

Map 인터페이스의 구현체로는 HashMap, LinkedHashMap, TreeMap, HashTable(Legacy)이 있다. 

- **HashMap**: 해시 기반으로 구현한 Map
- **LinkedHashMap**: HashMap을 상속 받았고, 요소가 추가된 순서대로 저장된다. 
- **TreeMap**: SortedMap 인터페이스의 구현체로, Key를 기준으로 원소의 순서를 정할 수 있다. (레드-블랙 트리 기반)

</details>
<br>

<details>
<summary>HashMap은 어떻게 동작하나요?</summary>

자바 초기 버전에 나온 HashTable 레거시 클래스를 보완하였다. 해시 테이블은 **Key를 해싱해서 나온 해시 코드를 배열의 인덱스로 활용하여 Value를 찾는 방식**으로 동작한다.  

![image](https://github.com/user-attachments/assets/bcee6d22-1393-4aac-bebd-85e32775c1ba)

HashMap은 멀티 스레드 환경에서 사용하기에 부적합하며, 이를 보완하기 위해 ConcurrentHashMap이 등장했다. 

</details>
<br>

<details>
<summary>HashMap의 최악의 시간 복잡도를 설명해 주세요.</summary>

HashMap은 평균적으로 O(1)의 시간복잡도를 가지지만, **해시 충돌**이 발생하면 O(N)까지 늘어날 수 있다. 

**해시 충돌은 서로 다른 키가 동일한 해시 값을 가지는 것**을 의미하며, 자바의 HashMap은 **체이닝 기법**으로 이를 해결한다.

</details>
<br>

## 컬렉션 심화

<details>
<summary>Map 인터페이스는 왜 Collection 인터페이스에 상속을 받지 않았나요?</summary>

우선, JCF 개발자들은 처음에 데이터의 저장 방식을 List, Set, Map으로 생각했다. 그러나, List와 Set은 비슷한 점이 많아 Collection 인터페이스로 묶을 수 있었지만, Map은 그럴 수 없었다. 

Collection은 요소들의 집합이라 생각하면 되는데, Map은 요소를 정의하기 위해 Key, Value가 모두 필요하다. 

그리고 Map을 Collection으로 구현했다면, remove() 함수는 단순히 Key-Value 쌍을 지우게 될 것이다. 하지만, 우리가 원하는 동작은 Key를 기준으로 Value를 삭제하는 것이다. 

이처럼 **Collection과 Map은 구조적으로 상응하지 않는 부분이 많아서, Map은 JCF에 포함시키되, Collection 인터페이스는 구현하지 않도록 설계한 것**이다! 

</details>
<br>

<details>
<summary>iterable과 iterator의 차이는 무엇인가요?</summary>

Iterable 인터페이스는 **Collection의 상위 인터페이스**로, **iterator()라는 추상 메서드**를 갖고 있다. 

Iterable을 구현하는 클래스들은 iterator() 메서드가 반환하는 **반복자 객체를 통해 컬렉션의 요소를 순회**할 수 있다. 

```java
public interface Iterable<T> {
    Iterator<T> iterator();

    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

Iterator 역시 인터페이스이며, 주요 메서드로는 hashNext(), next(), remove() 등이 있다. 

</details>
<br>

<details>
<summary>Collection과 Collections의 차이는 무엇인가요?</summary>

- Collection: List, Set, Queue 같은 컬렉션들의 상위 인터페이스로, **데이터 집합을 관리하는 기본적인 메서드**를 제공한다. (add(), remove(), contains(), size() 등)
- Collections: 컬렉션을 다루기 위한 **유틸리티 정적 메서드**를 제공하는 클래스 (sort(), reverse(), shuffle(), min(), max() 등)

</details>
<br>

## 스레드 기초

<details>
<summary>Java에서 스레드를 만드는 방법을 설명해 주세요.</summary>

**1) Thread 클래스의 객체를 직접 생성** 

```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("작업 스레드입니다.");
    }
});
```

```java
// 람다식을 이용한 방법 
Thread thread = new Thread(() -> System.out.println("작업 스레드입니다."));
```

작업 스레드는 생성되자마자 바로 실행되지 않고, start() 메서드를 호출해야 run() 메서드의 내용이 실행된다. 

```java
public static void main(String[] args) {
    // 작업 스레드 생성 
    Thread thread = new Thread(() -> {
        // run() 메서드 오버라이딩 
        Toolkit toolkit = Toolkit.getDefaultToolkit();

        // 0.5초 간격으로 비프음 발생 
        for (int i = 0; i < 5; i++) {
            toolkit.beep();
            try {
                Thread.sleep(500);
            } catch (Exception e) {
                e.printStackTrace();;
            }
        }
    });

    // 작업 스레드 실행 
    thread.start();

    // 메인 스레드에서는 0.5초 간격으로 문자열 출력 
    for (int i = 0; i < 5; i++) {
        System.out.println("띵");
        try {
            Thread.sleep(500);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

메인 스레드와 작업 스레드가 빠르게 번갈아가며 실행되기 때문에, 동시에 실행되는 것처럼 보인다. 

**2) Thread 클래스 상속 or Runnable 인터페이스 구현**

```java
// 작업 스레드 정의 
public class BeepThread extends Thread { 
    @Override
    public void run() {
        Toolkit toolkit = Toolkit.getDefaultToolkit();

        for (int i = 0; i < 5; i++) {
            toolkit.beep();
            try {
                Thread.sleep(500);
            } catch (Exception e) {
                e.printStackTrace();;
            }
        }
    }

    // 추가 로직 작성 가능
}

public class Main {
    public static void main(String[] args) {
        // 메인 스레드에서 작업 스레드 생성 및 실행 
        Thread thread = new BeepThread();
        thread.start();

        for (int i = 0; i < 5; i++) {
            System.out.println("띵");
            try {
                Thread.sleep(500);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

```java
public class MyRunnable implements Runnable { 
    @Override
    public void run() {
        Toolkit toolkit = Toolkit.getDefaultToolkit();

        for (int i = 0; i < 5; i++) {
            toolkit.beep();
            try {
                Thread.sleep(500);
            } catch (Exception e) {
                e.printStackTrace();;
            }
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Thread thread = new Thread(new MyRunnable());
        thread.start();

        for (int i = 0; i < 5; i++) {
            System.out.println("띵");
            try {
                Thread.sleep(500);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

</details>
<br>

<details>
<summary>스레드 풀이란 무엇이고, 왜 사용할까요?</summary>

스레드 풀은 작업 처리에 사용되는 **스레드를 제한된 개수만큼** 정해놓고, 작업 큐에 들어오는 작업들을 하나씩 스레드가 맡아서 처리하는 기법이다. 

작업 처리 요청이 급격히 증가하더라도 작업 큐에서 작업이 대기하다가, 여유 있는 스레드가 이를 처리하므로 

스레드 전체 개수는 일정하며, 어플리케이션의 성능도 저하되지 않는다. 

이를 통해 스레드를 매번 생성 및 제거하는 데 드는 비용도 줄일 수 있다. 

</details>
<br>

## 스레드 심화

<details>
<summary>스프링과 같은 프레임워크에서는 스레드 풀의 스레드 개수를 수백 개 이상으로 운영합니다. 이는 Context Switching이 일어남에도 불구하고도 이런 선택을 내린 것인데, 왜 그럴까요?</summary>

현대 운영체제는 컨텍스트 스위칭이 매우 최적화 되어 있어서, 많은 스레드를 사용할 때도 효율적으로 처리할 수 있다. 

그리고 많은 클라이언트의 요청을 처리해야 하는 어플리케이션의 경우, **빠른 응답 속도를 위해 스레드 개수를 늘리는 게 유리**하다. 

하나의 스레드가 I/O 바운드 작업(DB 쿼리, 파일 입출력, 외부 API 호출 등)을 처리하고 있을 때, **다른 스레드는 블로킹 되지 않고 다른 요청을 처리**할 수 있기 때문이다. 

스프링에서는 주로 ExecutorService로 스레드 풀을 구성하고, 요청이 많아지면 자동으로 스레드를 생성하거나 재사용하여 성능을 최적화한다. 따라서, 요청의 성격에 따라 유동적으로 스레드 수가 조정된다. 

</details>
<br>
