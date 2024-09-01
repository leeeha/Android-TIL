# 스레드

프로세스는 여러 개의 실행 흐름을 가질 수 있는데, 이러한 흐름을 스레드(Thread)라고 부른다. 즉, **스레드는 프로세스 내의 실행 흐름**이다. 

스레드는 프로세스 메모리의 **Code, Data, Heap 영역을 다른 스레드와 공유**하며, 스레드 ID, 프로그램 카운터, 레지스터 집합, **Stack 영역은 각 스레드마다 독립적으로 할당**된다. 

<img width="700" src="https://github.com/user-attachments/assets/c891984b-e0fb-4018-8edc-24067f9f59cf"/>

**스택 영역을 스레드마다 독립적으로 할당하는 이유**

프로세스의 스택 영역은 함수 매개변수, 지역 변수, 리턴 주소 등 함수 호출에 사용되는 임시 데이터들을 저장하는 메모리 공간이다. 

따라서, 스레드마다 독립적인 실행 흐름 및 함수 호출이 가능하려면 스택 영역은 별도로 가지고 있어야 한다.

**PC 레지스터를 스레드마다 독립적으로 할당하는 이유**

PC 레지스터에는 다음에 실행될 명령어의 주소가 저장되어 있다. 스레드는 CPU를 점유하고 있다가 스케줄러에 의해 선점 당하면 다른 스레드로 대체된다. 

이처럼 프로그램의 명령어가 연속적으로 수행되지 못하기 때문에 어디까지 수행했는지 기억할 필요가 있다. 따라서, PC 레지스터는 스레드마다 독립적으로 할당된다.

# 멀티 프로세스 vs 멀티 스레드 

## 멀티 프로세스의 장단점 

- 장점 
  - 각 프로세스는 **독립적인 메모리 공간**을 가지므로, 한 프로세스에 오류가 발생해도 다른 프로세스에 영향을 미치지 않아서 **안정성**이 높다.
- 단점 
  - **메모리와 CPU 자원**을 많이 소모한다.
  - **컨텍스트 스위칭 비용**이 많이 든다. (다음 프로세스의 메모리 주소 검색, 프로세스 상태 저장 및 복원, CPU 캐시 메모리 비우는 작업 등) 
  - **프로세스 간 통신을 위한 별도의 메커니즘**으로 오버헤드가 발생할 수 있다.

## 멀티 스레드의 장단점 

- 장점
    - **응답성**: 프로세스의 일부가 블로킹 되어 있어도 스레드는 계속 실행된다. (UI 처리할 때 특히 중요)
  - **자원 공유**: 스레드는 프로세스 메모리의 Code, Data, Heap 영역을 공유하므로, 프로세스 간의 통신보다 구현이 더 쉽다. 
  - **경제성**: 프로세스 생성보다 들어가는 비용이 적다. (컨텍스트 스위칭으로 PCB를 전환하는 것보다, 스레드 스위칭이 더 경제적이다.)
  - **확장성**: 멀티 프로세서 (또는 멀티 코어) 환경에서 각 스레드는 병렬적으로 수행되므로 시스템 규모를 확장하기 좋다.
- 단점
  - 자원을 공유하기 때문에 **동기화 문제**를 고려해야 한다. (경쟁 상태, 교착 상태 등)
  - 잘못된 동기화로 인한 병목 현상이 발생할 수 있으므로 주의 깊은 설계가 필요하며, **디버깅이 어려운 편**이다.
  - **한 스레드에 문제가 발생하면 다른 스레드에 영향을 미친다.**

멀티 프로세스와 멀티 스레드 방식 모두 장단점이 있으므로, 대상 시스템의 특징에 따라 적합한 동작 방식을 선택하고 적용해야 한다.

# 스레드의 종류 

### 사용자 수준 스레드 (User-Level Thread)

- 정의 
  - 커널 영역이 아닌, **사용자 영역의 라이브러리에 의해 생성 및 관리되는 스레드** 
  - 커널은 사용자 수준 스레드의 존재를 알지 못하며, **하나의 프로세스로 인식**함. 
  - 여러 개의 사용자 수준 스레드가 하나의 커널 수준 스레드에 대응되는 **다대일 매핑** 방식 
- 장점 
  - **높은 이식성**: 커널에 독립적으로 수행되므로 모든 운영체제에 적용 가능 
  - **오버헤드 감소**: 사용자 영역에서 스레드를 관리하므로, 커널 영역으로 전환할 필요 없음. 
  - **스케줄링의 유연성**: 라이브러리에서 스레드 스케줄링 제어하므로, 사용자 정의 스케줄링 알고리즘 구현 가능 
- 단점 
  - **동시성 지원 불가**: 커널은 여러 개의 사용자 수준 스레드를 하나의 프로세스로 인식하므로, 하나의 스레드가 블로킹 되면 다른 모든 스레드가 대기해야 함.
  - **멀티 프로세싱에서 병렬성 지원 불가**: 여러 스레드가 하나의 프로세스로 인식되므로, 멀티 프로세서 (또는 멀티 코어) 환경이어도 여러 스레드를 병렬 처리할 수 없음. 

<img width="300" alt="스크린샷 2024-08-22 오후 9 17 11" src="https://github.com/user-attachments/assets/74f3183a-0921-4673-9fcb-34484a6003d2">
<img width="200" alt="스크린샷 2024-08-22 오후 9 17 21" src="https://github.com/user-attachments/assets/e6513a14-0908-484e-831c-5d0a97cbe63a">

### 커널 수준 스레드 (Kernel-Level Thread)

- 정의 
  - **운영체제 커널에서 직접 관리하는 스레드** 
  - 하나의 사용자 수준 스레드마다 하나의 커널 수준 스레드가 대응되는 **일대일 매핑** 방식 
- 장점 
  - **동시성 지원**: 하나의 스레드가 블로킹 되어도 다른 스레드로 문맥 교환하여 다른 작업을 수행할 수 있음. 
  - **멀티 프로세싱에서 병렬성 지원**: 멀티 프로세서 (또는 멀티 코어) 환경에서 여러 개의 스레드를 병렬 처리할 수 있음. 
- 단점
  - **이식성 감소**: 운영체제에 따라 스레드 스케줄링 방식이나 구현이 달라질 수 있음.
  - **오버헤드 증가**: 스레드 관리 및 컨텍스트 스위칭 과정에서 커널이 개입하므로 오버헤드 증가

<img width="300" alt="스크린샷 2024-08-22 오후 9 17 35" src="https://github.com/user-attachments/assets/62691f47-f84f-46b6-af4d-03b57ed033d4">
<img width="200" alt="스크린샷 2024-08-22 오후 9 17 45" src="https://github.com/user-attachments/assets/8bbcc8a5-1539-4da3-8f67-376df8bf3ceb">

### 혼합형 스레드

- 사용자 수준 스레드와 커널 수준 스레드의 **다대다 매핑** 방식
- 사용자 수준 스레드, 커널 수준 스레드의 장점만을 취할 수 있다.

<img width="600" alt="스크린샷 2024-08-22 오후 9 19 37" src="https://github.com/user-attachments/assets/e5aecf1a-9af3-47cc-ad1e-0ff223b9ee89">

# 동시성 vs 병렬성 

- 동시성(Concurrency): 싱글 코어에서 여러 스레드가 빠르게 전환되어 동시에 실행되는 것처럼 보이는 것 
- 병렬성(Parallelism): 멀티 코어에서 각 코어 내의 스레드가 실제로 동시에 실행되는 것 

<img width="700" src="https://github.com/user-attachments/assets/5feaf758-e134-4d1a-8524-51668bbe059b"/> 

# 스레드의 제어 

## 스레드 대기 

`join()` 함수로 부모 스레드가 자식 스레드가 끝날 때까지 대기하게 만들 수 있다. 

```java
public class ThreadExample4 {
    public static void main(String[] args) {
        Runnable task = () -> {
          for(int i = 0; i < 5; i++){
              System.out.println("Hello, Lambda Runnable!");
          }
        };

        Thread thread = new Thread(task);
        thread.start(); // 자식 프로세스 실행 
        
        try{
            thread.join(); // 자식 프로세스가 종료될 때까지 대기 
        }catch (InterruptedException ie){
            System.out.println("Parent thread is interrupted");
        }
        
        System.out.println("Hello, My Joined Child!"); // 메인 스레드로 복귀 
    }
}
```

>Hello, Lambda Runnable!<br>
Hello, Lambda Runnable!<br>
Hello, Lambda Runnable!<br>
Hello, Lambda Runnable!<br>
Hello, Lambda Runnable!<br>
Hello, My Joined Child!<br>

## 스레드 종료 

`interrupt()` 함수로 실행 중인 스레드에 인터럽트를 걸어서 스레드를 종료시킬 수 있다. 

```java
public class ThreadExample5 {
    public static void main(String[] args) throws InterruptedException{
        // 0.1초씩 5번 출력 
        Runnable task = () -> {
            try{
                while(true){
                    System.out.println("Hello, Lambda Runnable!");
                    Thread.sleep(100);
                }
            }catch (InterruptedException ie){
                System.out.println("interrupted...");
            }
        };

        Thread thread = new Thread(task);
        thread.start(); // 자식 프로세스 실행 
        Thread.sleep(500);
        thread.interrupt(); // 자식 프로세스 종료 

        System.out.println("Hello, My Interrupted Child!");
    }
}
```

>Hello, Lambda Runnable!<br>
Hello, Lambda Runnable!<br>
Hello, Lambda Runnable!<br>
Hello, Lambda Runnable!<br>
Hello, Lambda Runnable!<br>
Hello, My Interrupted Child!<br>
interrupted...<br>

# 면접 예상 질문 

<details>
<summary>쓰레드에 대해 설명해주세요.</summary>

스레드는 **프로세스 내에서 실행되는 여러 흐름의 단위**이다. 

</details>
<br>

<details>
<summary>쓰레드의 메모리 공간에 대해 설명해주세요.</summary>

프로세스 메모리의 Code, Data, Heap 영역을 공유하며, Stack 영역, 레지스터 집합, PC(Program Counter), 스레드 ID는 각 스택마다 별도로 가진다.

그래서 프로세스보다 스레드의 컨텍스트 스위칭 오버헤드가 더 적게 든다. 

</details>
<br>

<details>
<summary>쓰레드 제어 블록(TCB)에 대해 설명해주세요.</summary>

운영체제가 스레드를 관리하고 문맥 전환을 수행하는 데 필수적인 정보들을 담고 있는 자료구조 (스레드 ID, 레지스터 집합, 스케줄링 정보 등) 

</details>
<br>

<details>
<summary>사용자 수준 쓰레드와 커널 수준 쓰레드의 차이를 설명해 보세요.</summary>

- 사용자 수준 스레드: 사용자 라이브러리로 구현한 스레드. 커널은 스레드의 존재를 알지 못하며, 스레드 컨텍스트 스위칭에 관여하지 않는다.
- 커널 수준 스레드: 커널이 직접 생성하고 관리하는 스레드 

</details>
<br>

<details>
<summary>멀티 쓰레딩 프로그래밍 대해서 설명해주세요.</summary>

한 프로세스에서 여러 개의 스레드가 동시에 실행되는 것 (CPU를 번갈아가면서 점유하는 시분할 시스템)

</details>
<br>

<details>
<summary>멀티 쓰레드 프로그래밍의 장단점을 설명해 주세요.</summary>

- 장점
  - **응답성**: 프로세스의 일부가 블로킹 되어 있어도 스레드는 계속 실행된다. (UI 처리할 때 특히 중요)
  - **자원 공유**: 스레드는 프로세스 메모리의 Code, Data, Heap 영역을 공유하므로, 프로세스 간의 통신보다 구현이 더 쉽다. 
  - **경제성**: 프로세스 생성보다 들어가는 비용이 적다. (컨텍스트 스위칭으로 PCB를 전환하는 것보다, 스레드 스위칭이 더 경제적이다.)
  - **확장성**: 멀티 프로세서 (또는 멀티 코어) 환경에서 각 스레드는 병렬적으로 수행되므로 시스템 규모를 확장하기 좋다.
- 단점
  - 자원을 공유하기 때문에 **동기화 문제**를 고려해야 한다. (경쟁 상태, 교착 상태 등)
  - 잘못된 동기화로 인한 데드락이 발생할 수 있으므로 주의 깊은 설계가 필요하며, **디버깅이 어려운 편**이다.
  - **한 스레드에 문제가 발생하면 다른 스레드에 영향을 미친다.** 

</details>
<br>

<details>
<summary>멀티 프로세스 대신 멀티 쓰레드를 사용하는 이유가 뭔가요?</summary>

위에서 언급한 멀티 스레드의 장점 때문에!

</details>
<br>

<details>
<summary>멀티 쓰레드 프로그래밍에서 주의할 점이 있을까요?</summary>

여러 스레드가 자원을 공유하기 때문에 Thread-Safe 하게 프로그램을 설계하여, 동시성 이슈가 발생하지 않도록 해야 한다. 

</details>
<br>

<details>
<summary>Thread-Safe 하다는 의미와 그렇게 설계하는 방법을 설명해주세요.</summary>

Thread-Safe 하다는 것은, **멀티 스레드 환경에서 동시에 여러 스레드가 같은 자원에 접근해도 프로그램의 코드가 예상대로 동작하는 것**을 의미한다.

이를 위해 다음과 같은 설계 방법을 고려할 수 있다.

- **재진입성** 
  - 어떤 함수가 한 스레드에서 실행 중일 때, 다른 스레드가 해당 함수를 호출하더라도 결과가 각각 올바르게 주어져야 한다. 
- **상호 배제**    
  - 공유 자원에 대한 접근은 뮤텍스, 세마포어 같은 lock으로 통제한다.
  - 임계 구역에는 현재 lock을 획득한 단 하나의 스레드만 접근할 수 있게 만든다. 
- **원자적 연산** 
  - 원자적 연산 (atomic operation)은 더 이상 쪼갤 수 없는 즉, 중간에 인터럽트를 걸 수 없는 하나의 연산 단위를 의미한다.
  - 공유 자원에 대해서는 원자적 연산을 수행하여 중간에 컨텍스트 스위칭이 발생하지 않도록 한다.
- **스레드 지역 저장소**
  - 각 스레드만 접근 가능한 로컬 저장소를 만들어서 같은 자원을 공유하지 않게 한다. 
- **불변 객체** 
  - 공유 자원을 불변 객체로 만들어서 한번 생성한 후에 값을 변경할 수 없게 만든다. 

</details>
<br>

# 참고자료 

- https://velog.io/@kangdev/기술면접운영체제-Process와-Thread의-메모리-공유-규칙
- https://velog.io/@jxlhe46/OS-Ch4.-쓰레드
- https://coding-start.tistory.com/199
- https://blog.naver.com/jk130694/220689577611