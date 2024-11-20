## 동시성 프로그래밍 기초

<details>
<summary>동시성과 병렬성의 차이점을 말해주세요.</summary>

- 동시성(Concurrency): 싱글 코어에서 여러 스레드가 빠르게 전환되어 동시에 실행되는 것처럼 보이는 것 (논리적 개념)
- 병렬성(Parallelism): 멀티 코어에서 각 코어 내의 스레드가 실제로 동시에 실행되는 것 (물리적 개념)

<img width="500" src="https://github.com/user-attachments/assets/5feaf758-e134-4d1a-8524-51668bbe059b"/> 

</details>
<br>

<details>
<summary>Thread-Safe하다는 것이 무슨 의미인가요?</summary>

Thread-Safe 하다는 것은, **멀티 스레드 환경에서 동시에 여러 스레드가 같은 자원에 접근해도 프로그램이 예상대로 동작하는 것**을 의미한다.

이를 위해 다음과 같은 설계 방법을 고려할 수 있다.

- **재진입성** 
  - 어떤 함수가 한 스레드에서 실행 중일 때, 다른 스레드가 해당 함수를 호출하더라도 **결과가 각각 올바르게** 주어져야 한다. 
- **상호 배제**    
  - 공유 자원에 대한 접근은 **뮤텍스, 세마포어 같은 lock**으로 통제한다.
  - 임계 구역에는 **현재 lock을 획득한 단 하나의 스레드만 접근**할 수 있게 만든다. 
- **원자적 연산** 
  - 원자적 연산 (atomic operation)은 더 이상 쪼갤 수 없는 즉, **중간에 인터럽트를 걸 수 없는 하나의 연산 단위**를 의미한다.
  - 공유 자원에 대해서는 원자적 연산을 수행하여 중간에 **컨텍스트 스위칭이 발생하지 않도록** 한다.
- **스레드 지역 저장소**
  - 각 스레드만 접근 가능한 로컬 저장소를 만들어서 **같은 자원을 공유하지 않게** 한다. 
- **불변 객체** 
  - 공유 자원을 불변 객체로 만들어서 한번 생성한 후에 **값을 변경할 수 없게** 만든다. 

</details>
<br>

<details>
<summary>가시성 문제와 원자성 문제에 대해 설명해 주세요.</summary>

- 가시성 문제: CPU Cache Memory와 RAM의 데이터가 제대로 동기화 되지 않아서, 여러 스레드가 서로의 데이터를 볼 수 없는 것 
- 원자성 문제: 한 스레드가 기계어를 수행하는 동안 다른 스레드가 공유 변수에 접근하여 값을 수정하면서, 데이터가 일관성을 잃는 것 

</details>
<br>

<details>
<summary>자바의 동시성 이슈를 해결하는 방법을 아는만큼 설명해 주세요.</summary>

 - volatile: 여러 스레드가 CPU 캐시 메모리를 거치지 않고 RAM에서 직접 데이터를 읽고 쓰게 만들어서, 가시성 문제 해결 
 - synchronized: blocking 방식으로 상호 배제를 보장하여 원자성 문제 해결, CPU 캐시 메모리와 메인 메모리의 동기화를 통해 가시성 문제 해결 
 - atomic: CAS(Compare And Swap) 알고리즘을 기반으로 non-blocking 하게 원자성 및 가시성 문제 해결 

</details>
<br>

<details>
<summary>volatile 키워드가 무엇인가요?</summary>

동시성 프로그래밍에서 발생할 수 있는 문제 중에 하나인 가시성 문제를 해결하기 위해 사용되는 키워드다. 

가시성 문제는 멀티 스레드 환경에서 **각 스레드의 CPU Cache Memory와 RAM의 데이터가 서로 일치하지 않아서 발생하는 문제**이다. 이름 그대로, 여러 스레드가 **서로의 데이터를 볼 수 없어서** 발생하는 문제인 것이다. 

volatile 키워드를 붙인 공유 자원은 **CPU Cache Memory를 거치지 않고, RAM에 직접 읽고 쓰는 작업을 수행**하게 된다. 이를 통해 여러 스레드 간의 가시성 문제를 해결할 수 있다. 

<img height="300" src="https://github.com/user-attachments/assets/f3ae541f-b211-441c-a128-f3ce6661998f"/>
<img height="300" src="https://github.com/user-attachments/assets/b7efc131-0661-47bf-ad0a-4a0f32abbfec"/> 

</details>
<br>

<details>
<summary>synchronized 키워드가 무엇인가요?</summary>

**Java에서 제공하는 동기화 도구**로, 멀티 스레드 환경에서 **단 하나의 스레드만 임계 영역에 접근할 수 있도록 보장**하여 원자성 문제를 해결한다. 

여러 프로세스가 동시에 사용할 수 없는 공유 자원에 접근하는 프로그램 코드의 일부분을 임계 영역이라고 부른다. 

참고로, synchronized 키워드는 동기화 블록에 진입하기 전에 공유 변수에 대한 CPU 캐시 메모리와 메인 메모리의 값을 동기화하여 가시성 문제를 해결한다. 

**동작 과정**

1. 스레드가 synchronized 블록 또는 메서드에 진입을 시도한다.
2. 해당 객체의 모니터 락을 획득한다. (lock은 임계 영역에 진입할 때 필요한 열쇠 같은 것)
3. 락을 획득한 스레드는 synchronized 블록 또는 메서드를 실행한다.
4. 블록 또는 메서드의 실행이 끝나면 모니터 락을 반환한다.
5. 다른 대기 중인 스레드 중 하나가 락을 획득하고 실행을 시작한다.

**장점**

- 데이터 무결성 보장: 여러 스레드가 동시에 접근하는 상황에서 데이터의 일관성과 무결성을 유지한다.
- 안정성: 상호 배제를 통해 경쟁 조건(Race Condition)과 같은 동시성 문제를 방지할 수 있다.

**단점**

- 성능 저하: 락을 사용하면 스레드가 락을 획득할 때까지 대기해야 하므로 성능이 저하될 수 있다.
- 데드락(교착 상태): 잘못된 락 사용으로 인해, 데드락이 발생할 수 있다. 데드락은 여러 스레드가 서로의 자원을 필요로 해서, 어떤 작업도 완료되지 못한 채 무한 대기 상태에 빠지는 것을 말한다.
- 확장성 제한: 과도한 락 사용은 시스템의 확장성을 제한할 수 있다.

</details>
<br>

<details>
<summary>atomic하다는 것이 무슨 의미인가요?</summary>

원자적 연산 (atomic operation)은 더 이상 쪼갤 수 없는 즉, **중간에 인터럽트를 걸 수 없는 하나의 연산 단위**를 의미한다.

공유 자원에 대해서는 원자적 연산을 수행하여 중간에 **컨텍스트 스위칭이 발생하지 않도록** 해야 한다. 

</details>
<br>

<details>
<summary>atomic 타입이 무엇인가요?</summary>

CAS(Compare And Swap) 알고리즘 기반으로, 원자성 문제와 가시성 문제를 해결한다. non-blocking 방식이어서 synchronized 키워드보다 성능이 우수하다. 

</details>
<br>

## 동시성 프로그래밍 심화

<details>
<summary>가시성 문제에 대해 조금 더 자세히 설명해 주세요. 여러 스레드가 모두 한 CPU의 캐시 메모리를 읽으면 가시성 문제가 발생하지 않을 것 같은데, 어떻게 생각하시나요?</summary>

</details>
<br>

<details>
<summary>synchronized의 문제점은 무엇이 있나요?</summary>

특정 스레드가 synchronized 블록으로 lock을 걸면, 해당 블록에 접근하는 모든 스레드는 **블로킹 상태**가 되어서 아무 작업도 못한 채 **자원을 낭비**하게 된다. 

또한, blocking 상태의 스레드를 **runnable 또는 running 상태로 변경하기 위해 시스템 자원이 사용**되므로, **성능 저하**로 이어질 수 있다. 

이러한 문제점 때문에 논블로킹 상태로 원자성을 보장하는 방법이 atomic 키워드다. 

</details>
<br>

<details>
<summary>synchronized는 어떻게 구현되어 있나요?</summary>

```java
// 메서드 동기화 
// 이 메서드에 접근하는 모든 스레드는, 메서드가 속한 객체의 모니터 락을 획득해야 한다. 
public class SynchronizedExample {
    public synchronized void synchronizedMethod() {
        System.out.println("This method is synchronized.");
    }
}
```

```java
// 블록 동기화 
// synchronized 인자에 모니터 락을 가진 객체를 지정할 수 있다. 
public class SynchronizedExample {
    private final Object lock = new Object();

    public void synchronizedBlock() {
        synchronized(lock) {
            System.out.println("This block is synchronized.");
        }
    }
}
```

```java
// 클래스 락 
// 이 클래스의 모든 인스턴스가 하나의 공통된 락을 사용한다. 
// 따라서, 여러 인스턴스 중에 하나라도 동기화 된 메서드에 접근하면, 다른 모든 인스턴스도 락이 해제될 때까지 대기해야 한다. 
public class SynchronizedExample {
    public void classLock() {
        synchronized(SynchronizedExample.class) {
            System.out.println("This block is synchronized at class level.");
        }
    }
}
```

</details>
<br>

<details>
<summary>CAS 알고리즘에 대해 설명해 주세요.</summary>

![image](https://github.com/user-attachments/assets/87d69dcb-8ef4-4230-b2f6-6dba85a2f677)

1. 인자로 기존 값(Compared Value)과 변경할 값(Exchanged Value)을 전달한다.
2. 기존 값(Compared Value)이 현재 메모리에 저장된 값(Destination)과 같다면, 변경할 값(Exchanged Value)을 반영하며 true를 반환한다.
3. 반대로 기존 값(Compared Value)이 현재 메모리에 저장된 값(Destination)과 다르다면, 값을 반영하지 않고 false를 반환한다.

기존 값이 현재 메모리에 저장된 값과 다른 경우는 언제 발생하는가?

스레드 A가 공유 변수에 대한 연산을 마치고 메모리에 저장하려고 하는데, 이미 스레드 B가 공유 변수를 변경하여 메모리에 반영한 경우 발생할 수 있다. 이럴 때는 스레드 A의 연산 결과를 메모리에 반영하면 안 된다. 

따라서, **false를 반환할 때는 무한 루프를 돌려서 (다른 스레드에 의해 변경된) 메모리 값을 읽고 같은 시도를 반복하거나, 다른 더 중요한 작업이 있다면 먼저 수행**할 수 있다. 이 부분은 개발자가 정하면 된다.

true를 반환할 때는 캐시 메모리의 변경된 값을 메인 메모리에도 반영하여, **가시성 문제도 해결**할 수 있다. 

actomic 타입은 이러한 CAS 알고리즘을 기반으로 작동하며, **blocking 방식의 synchronized에 비해 훨씬 효율적**이다. 

그리고 무한 루프를 돌면서 값을 반영할 수 있는지 물어볼 때도, 스레드의 상태를 바꾸지 않으므로 synchronized 보다 성능이 우수하다. 

</details>
<br>

<details>
<summary>Vector, Hashtable, Collections.SynchronizedXXX의 문제점은 무엇인가요?</summary>



</details>
<br>

<details>
<summary>SynchronizedList와 CopyOnArrayList의 차이를 설명해 주세요.</summary>



</details>
<br>

<details>
<summary>ConcurrentHashMap의 동작 과정을 SynchronizedMap과 비교하여 설명해 주세요.</summary>



</details>
<br>

## 참고자료 

- https://steady-coding.tistory.com/555
- https://steady-coding.tistory.com/556
- https://steady-coding.tistory.com/554