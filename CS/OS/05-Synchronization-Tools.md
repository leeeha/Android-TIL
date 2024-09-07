# 경쟁 상태 (Race Condition)

**여러 개의 프로세스나 스레드가 공유 데이터에 동시에 접근하여 값을 변경할 때, 그 실행 결과가 특정한 접근 순서에 따라 달라지는 것**을 경쟁 상태(Race Condition)라고 한다. 

즉, 여러 프로세스나 스레드가 하나의 자원을 놓고 서로 본인이 사용하려고 경쟁하는 상황인 것이다. 

이러한 경쟁 상태를 방지하기 위해서, 개발자는 **오직 한번에 하나의 프로세스만 공유 데이터에 접근하도록** 만들어야 한다. 이를 위해 서로 다른 프로세스를 **동기화(synchronization)** 해줘야 한다.

```java 
public class RaceCondition1 {
    public static void main(String[] args) throws InterruptedException {
        RunnableOne run1 = new RunnableOne();
        RunnableOne run2 = new RunnableOne();

        Thread t1 = new Thread(run1);
        Thread t2 = new Thread(run2);

        t1.start(); t2.start();
        t1.join(); t2.join();

        System.out.println("Result: " + run1.count + ", " + run2.count);
    }
}

class RunnableOne implements Runnable {
    int count = 0;

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            count++;
        }
    }
}
```

>Result: 10000, 10000

RunnableOne 객체 run1, run2는 서로 다른 메모리 공간에 할당되어 별도의 count 변수를 갖는다. 

즉, count 변수는 두 객체의 공유 데이터가 아니므로 실행 결과가 항상 일관성 있게 나온다. 

```java 
public class RaceCondition2 {
    public static void main(String[] args) throws InterruptedException {
        RunnableTwo run1 = new RunnableTwo();
        RunnableTwo run2 = new RunnableTwo();

        Thread t1 = new Thread(run1);
        Thread t2 = new Thread(run2);

        t1.start(); t2.start();
        t1.join(); t2.join();

        System.out.println("Result: " + RunnableTwo.count); // 클래스 멤버 변수 
    }
}

class RunnableTwo implements Runnable {
    static int count = 0; // 모든 객체가 공유하는 데이터

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            count++;
        }
    }
}
```

>Result: 15094<br>
Result: 14440<br>
Result: 14815<br>
... 

반면에, **static 키워드를 붙여서 count 변수를 모든 객체가 공유하는 데이터로 만들면, 실행할 때마다 결과가 다르게 나온다!** 

run 함수의 for문 반복 횟수를 100이나 500, 1000으로 줄이면 일정한 값이 나오지만, 5000 정도부터는 실행할 때마다 결과가 다르게 나온다. (data inconsistency) 

## Race Condition 발생 예시 

- **공유 메모리를 사용하는 프로세스의 경우** 
- **커널 모드에서 작업 수행 중에 인터럽트가 발생하여 인터럽트 처리 루틴이 수행되는 경우** 
  - 인터럽트 처리 루틴 (Interrupt Service Routine, ISR)은 결국 커널 코드다. 
  - 따라서 커널 모드에서 작업 수행 중에 인터럽트가 발생하면, 커널의 공유 데이터에 여러 프로세스가 동시에 접근하는 경쟁 상태가 발생할 수 있다.
- **프로세스가 시스템 콜을 호출하여 커널 모드에서 수행되다가 컨텍스트 스위치가 발생하는 경우** 
  - 프로세스 A가 커널 모드에서 작업을 수행하다가 프로세스 B가 CPU를 선점하면, 커널의 공유 데이터에 대한 경쟁 상태가 발생할 수 있다.
  - 이를 방지하기 위해 커널 모드일 때는 CPU 선점을 불가능하게 만들고, 유저 모드로 전환되면 선점하도록 구현할 수 있다.
- **멀티 프로세서 환경에서 여러 CPU가 공유 메모리 내의 커널 데이터에 동시 접근하는 경우** 
  - 공유 데이터에 대한 lock/unlock 연산으로 공유 자원에 하나의 CPU만 접근할 수 있도록 한다. 

# 임계 구역 문제 (Critical Section Problem)

**여러 프로세스가 동시에 사용할 수 없는 공유 자원**을 임계 자원 (Critical Resource)라고 부르며, **임계 자원에 접근하는 프로그램 코드의 일부분**을 임계 구역 (Critical Section)이라고 부른다.

**하나의 프로세스가 임계 구역에서 실행되고 있으면, 다른 어떤 프로세스도 임계 구역에서 실행될 수 없다.** 이를 통해 여러 프로세스를 동기화 시킬 수 있다. 

<img width="500" src="https://github.com/user-attachments/assets/b70b96cc-dec7-43b0-96f3-a55e452c2af5"/>

## 임계 구역 문제를 해결하기 위한 3가지 조건 

### 1️⃣ Mutual Exclusion

**한 프로세스가 임계 구역에서 실행되고 있다면, 다른 어떤 프로세스도 임계 구역에서 실행될 수 없다.** 이러한 **상호 배제**는 프로세스 간 경쟁 상태를 막기 위해 기본적으로 갖춰야 할 조건이다.

그런데, 상호 배제로 발생할 수 있는 두 가지 문제가 있다. 바로 **데드락과 기아 상태**이다. 각 문제를 해결하기 위한 방법이 Progress, Bounded Wating이다.

### 2️⃣ Progress (데드락 회피)

**임계 구역에서 실행 중인 프로세스가 없다면, 다음 프로세스가 임계 구역에 진입할 수 있게 해주는 방법이다.** 그렇지 않으면, 다음 프로세스들이 무한정 대기하는 **데드락**이 발생할 수 있다.

### 3️⃣ Bounded Wating (기아 상태 회피)

**각 프로세스가 임계 구역에 들어올 수 있는 횟수를 제한하는 방법이다.** 그렇지 않으면, 특정 프로세스가 임계 구역에 계속 들어오지 못하고 무한정 대기하는 **기아 상태**가 발생할 수 있다. 

## Deadlock (교착 상태)

교착 상태는 **여러 프로세스가 서로의 자원을 필요로 해서 어떤 작업도 완료하지 못한 채 무한 대기 상태에 빠지는 것**을 말한다.

### 데드락의 발생 조건 

1. Mutual Exclusion (상호 배제): 한번에 하나의 프로세스만 자원을 사용할 수 있다.
2. Hold and Wait (점유와 대기): 자신의 자원을 보유한 상태에서 다른 프로세스의 자원을 요청하는 프로세스가 있다.
3. No Preemption (비선점): 프로세스가 할당 받은 자원을 강제로 빼앗을 수 없다. 
4. Circular Wait (순환 대기): 대기하고 있는 프로세스 집합이 서로의 자원을 순환 형태로 요청하고 있다.

예를 들어, 프로세스 A가 자원 1을 점유한 상태에서 자원 2를 기다리고 있는데, 프로세스 B가 자원 2를 점유한 상태에서 자원 1을 기다리고 있다면 두 프로세스 모두 데드락에 빠진다. 

<img width="400" src="https://github.com/user-attachments/assets/efed6e95-44bc-470c-be48-98420e7ca45f"/>

위의 그림은 프로세스의 자원 할당 상태를 나타내는 **자원 할당 그래프**이다. 동그라미는 프로세스, 네모는 자원, 네모 안의 점은 해당 자원의 인스턴스를 의미한다.

- 프로세스 → 자원: 프로세스가 해당 자원을 요청하는 것 
- 프로세스 ← 자원: 프로세스가 해당 자원을 할당받은 것 

자원 할당 그래프로 **데드락의 발생을 탐지하는 방법**은 다음과 같다. 

- 자원 할당 그래프에 사이클이 존재하지 않는 경우 
  - 데드락 상황이 아니다. 
- 자원 할당 그래프에 사이클이 존재하는 경우 
  - 자원이 갖는 인스턴스가 하나라면, 데드락 상황이다. 
  - 자원이 갖는 인스턴스가 여러 개라면, 데드락일 수도 아닐 수도 있다. 

### 데드락 해결 방법 

**Deadlock Prevention (예방)**
- 프로세스에 자원을 할당할 때, **데드락의 발생 조건 4가지 중에 하나라도 발생하지 않도록 예방**한다. 
- 상호 배제 조건 위배: 데이터의 비일관성 문제가 발생하므로 위배할 수 없다. 
- 점유 대기 조건 위배: 프로세스가 다른 자원을 요청할 때 자신이 가진 자원은 모두 놓도록 한다. 
- 비선점 조건 위배: 중간에 프로세스의 자원을 가로챌 수 있게 한다. 
- 순환 대기 조건 위배: 모든 프로세스가 정해진 순서대로만 자원을 할당 받게 한다. 
- **데드락 예방의 단점: CPU 사용률 및 처리량 감소, Starvation 발생 가능** 
  
**Deadlock Avoidance (회피)**

- 프로세스의 **자원 요청이 데드락으로부터 안전한지 동적으로 조사**하여, **안전한 경우에만 자원을 보수적으로 할당**하는 방법 
- 특정 프로세스의 최대 자원 요구량 < 현재 사용 가능한 자원량 -> 이때만 자원 할당 
- 프로세스가 필요로 하는 **자원별 최대 사용량을 미리 선언**하여 정보로 활용한다. 

**Deadlock Detection and Recovery (탐지 및 회복)**
- **데드락의 발생을 허용하되, 그 발생 여부를 탐지하여 데드락에서 벗어날 수 있게 한다.** 
- 데드락이 발생한 경우, **프로세스를 종료**시키거나 **필요한 자원을 선점**하는 회복 기법을 사용할 수 있다.
- 데드락을 탐지하는 로직에서 **시스템 오버헤드**가 발생할 수 있다. 

**Deadlock Ignorance (무시)**
- **데드락이 발생해도 아무런 처리를 하지 않는 방법이다.** 
- 데드락은 실제로 매우 드물게 발생하므로, 이에 대한 조치 자체가 오버헤드가 될 수 있다.
- 데드락이 발생하면 사용자가 알아서 이를 감지하여 프로세스를 종료시키게 한다. 
- 유닉스, 윈도우 등 **대부분의 범용 OS에서 채택하는 방법**이다. 

## Starvation (기아 상태)

기아 상태는 **특정 프로세스가 오랫동안 자원을 할당받지 못해 무한 대기 상태에 빠지는 것**을 말한다. 

주로 **CPU 스케줄링이나 자원 할당 정책의 불공정성** 때문에 발생한다. 대표적으로, **우선순위 기반 스케줄링**에서 낮은 우선순위의 프로세스가 계속해서 높은 우선순위의 프로세스에 의해 자원을 할당받지 못할 때 발생한다. 

기아 현상의 해결 방법 중에 하나는, **프로세스의 대기 시간에 비례해서 우선순위를 점진적으로 늘려주는 Aging 기법**이 있다. 

# 뮤텍스와 세마포어 

임계 구역 문제 (Critical Section Problem, CSP)를 해결하기 위한 **고수준의 소프트웨어 도구**인 뮤텍스, 세마포어에 대해 알아보자.

## 뮤텍스 

- mutex: mutual exclusion
- **상호 배제를 보장**하여, 임계 구역을 보호하고 경쟁 상태를 막기 위한 도구
- **Locking 메커니즘**으로 오직 하나의 프로세스만 **뮤텍스 락**을 얻어 임계 구역에 진입할 수 있다. (lock은 임계 구역에 들어가기 위한 하나의 열쇠와 같다.)
- **lock을 설정한 프로세스만 lock을 해제할 수 있다.** (다른 프로세스가 대신 lock을 해제할 수 없다.)
- Mutex Lock을 구현하려면 `acquire()`, `release()` 함수와 lock의 가용 여부를 알려주는 `available` 변수만 있으면 된다. 

<img width="400" src="https://github.com/user-attachments/assets/72e9d8fb-1fb4-4614-b2e2-7ee33da35d62"/>

<img width="600" src="https://github.com/user-attachments/assets/780d7da9-014f-41de-86a2-0cb126f32de1"/>

- `acquire()`: lock을 얻을 수 없을 때는 busy waiting 하다가, release() 되어 lock을 얻을 수 있으면 다른 프로세스가 임계 영역에 들어오지 못하도록 `available = false`로 설정
- `release()`: 특정 프로세스가 lock을 얻을 수 있도록 `available = true`로 설정
- 위의 두 함수를 호출하는 것은 **원자적으로** 수행되어야 한다. 즉, 중간에 컨텍스트 스위치가 발생하면 안 된다.
- **Busy Waiting** : 임계 구역에 들어가려는 프로세스가 lock을 얻기 위해 무한히 대기하는 상태 (CPU, 메모리 같은 자원 소모 증가)
- **Spinlock** : Busy Waiting 상태의 뮤텍스 락 (Spinlock으로 대기하고 있을 때는 컨텍스트 스위치가 발생하지 않는다.)

## 세마포어

- 세마포어는 **Signaling 메커니즘**이라는 점에서 뮤텍스와 다르다.
- **세마포어는 lock을 걸지 않은 다른 프로세스도 signal을 보내서 lock을 해제할 수 있다.**
- 세마포어 변수 S는 **사용 가능한 리소스의 개수**를 의미하며, `wait()`, `signal()`이라는 **오직 2개의 원자적 연산**으로만 접근할 수 있다. 
- 프로세스가 lock을 얻어서 리소스를 사용할 때는 `wait()`의 S를 감소시키고, 프로세스가 리소스를 해제할 때는 `signal()`의 S를 증가시킨다.

<img width="600" src="https://github.com/user-attachments/assets/d76c5d9f-ceda-4bbc-bd27-c292e8658c3a"/>

### 세마포어의 종류

- **Binary Semaphore** : S의 값이 0과 1만 왔다갔다 하기 때문에, mutex lock과 비슷하다.
  - 차이점은 뮤텍스에서는 lock을 설정한 프로세스만 unlock을 할 수 있는 반면에 
  - 이진 세마포어에서는 lock을 설정한 프로세스와 unlock을 하는 프로세스가 다를 수 있다. 
- **Counting Semaphore** : S 값의 범위가 무한대이고, 유한개의 인스턴스를 갖는 리소스에 사용될 수 있다.
  - 세마포어 S를 사용 가능한 리소스의 개수로 초기화 한다.
  - 프로세스가 lock을 얻어서 리소스를 사용할 때는 wait()의 S를 감소시키고, 프로세스가 리소스를 해제할 때는 signal()의 S를 증가시킨다.
  - S가 0이면 모든 리소스가 사용 중이기 때문에, 리소스가 필요한 프로세스는 S가 0보다 커질 때까지 블로킹 되어 대기한다.

### Busy Waiting (Spin-Lock)으로 구현한 세마포어 

<img width="600" src="https://github.com/user-attachments/assets/d76c5d9f-ceda-4bbc-bd27-c292e8658c3a"/>

세마포어 변수 S가 0 이하면 프로세스가 사용할 수 있는 공유 자원이 없으므로, Busy Waiting 하면서 대기한다.

이 방법은 컨텍스트 스위칭이 발생하지 않지만, CPU와 메모리를 계속 사용하므로 비효율적이다.

### Block and Wake up (Sleep-Lock)으로 구현한 세마포어 

<img height="350" src="https://github.com/user-attachments/assets/22a00ca3-79d9-46e0-bd93-810e60bd8eef">

- `sleep()` : 세마포어 변수가 음수일 때 호출되며, 해당 프로세스를 suspend 시킨다. 그리고 프로세스 PCB를 semaphore 구조체 안에 정의된 대기 큐에 넣는다. 
- `wakeup()` : 자원을 반납하면서 대기 큐에 있던 프로세스 P를 깨운다. 그리고 해당 프로세스의 PCB를 준비 큐로 옮긴다. 
- 임계 구역의 길이가 긴 경우, Busy waiting 보다 Block & Wake up 방법이 더 효율적이다. 

### 세마포어의 문제점  

세마포어는 동기화를 위한 편리하고 효과적인 도구이긴 하지만, 여러가지 **타이밍 문제가 발생**할 수 있다. 즉, 실행 시퀀스가 제대로 수행되지 않을 때가 있는데 이런 문제는 발견하기 매우 어렵다. 

예를 들어, 1로 초기화 된 이진 세마포어 mutex를 모든 프로세스가 공유하고 있는 상황을 생각해보자. 각 프로세스는 임계 구역에 들어가기 전에 wait(mutex)로 대기하고, 들어간 후에는 signal(mutex)로 mutex의 값을 증가시킨다. 그런데, 이러한 **실행 시퀀스가 제대로 관찰되지 않으면, 두 프로세스는 임계 구역에 동시에 진입**할 수 있다. 

>ex1) wait(), signal()을 호출하는 순서가 바뀔 수 있다.<br>
ex2) wait()와 signal() 중에 하나만 두 번 호출할 수도 있다. <br>
ex3) 프로세스가 wait(), signal() 중에 하나를 호출하지 않거나, 둘 다 호출하지 않을 수도 있다. <br>

<img height="150" src="https://velog.velcdn.com/images/jxlhe46/post/1928bc48-7f5c-47bc-83ec-f9fb9f59b365/image.png"/> <img height="150" src="https://velog.velcdn.com/images/jxlhe46/post/06d417e8-52cf-4aba-8c78-4222ed24247b/image.png"/>

이런 문제들은 프로그래머가 **세마포어나 뮤텍스를 부정확하게 사용하면 매우 쉽게 발생**할 수 있다. 이를 해결하기 위해 고수준의 언어에서 동기화 도구로 '모니터'를 제공한다. 

# 모니터

모니터는 **서로 다른 프로세스 간의 상호 배제를 보장하는 추상 데이터 타입** (ADT)이다. 

세마포어는 개발자가 wait(), signal() 함수로 직접 동기화를 구현해야 하지만, 모니터는 **프로그래밍 언어 단에서 내부적으로 동기화 처리**를 해주기 때문에 사용법이 간단하고 실수할 가능성이 적다. 

## 모니터의 구성

모니터는 클래스, 구조체와 유사한 개념이며, 아래 그림처럼 **공유 데이터, 공유 데이터를 사용하는 프로시저, 초기화 코드**로 이루어져 있다. 

<img height="300" src="https://velog.velcdn.com/images/jxlhe46/post/21a93daf-e002-4e99-a759-5653cf0e7212/image.png"/> <img height="300" src="https://velog.velcdn.com/images/jxlhe46/post/fdf99a53-9b47-49b5-96ca-ec1c11fe9f3d/image.png"/>

## 모니터의 특징

- **상호 배제**: 한번에 하나의 프로세스만 모니터 객체 내의 프로시저를 호출할 수 있다. 
- **캡슐화**: 프로세스는 모니터 객체 내의 공유 자원에 직접적으로 접근 불가능하며, 프로시저를 통해서만 가능하다. 
- **추상화**: 모니터 내부에 동기화 코드가 구현되어 있으므로, 뮤텍스, 세마포어처럼 개발자가 직접 동기화 코드를 작성하지 않아도 된다. 

## 조건 변수 

기본적인 모니터 구조에서는 식사하는 철학자 문제 등 고전적인 동기화 문제를 해결하는데 한계가 있다. 그래서 **조건 변수** (Conditional Variable)가 추가되었다. 이 변수는 추가적인 동기화 메커니즘을 제공한다. 

아래 코드처럼 하나 이상의 조건 변수를 정의할 수 있고, 이러한 변수들이 호출할 수 있는 유일한 함수는 wait(), signal()이다. 

- `wait()` : 이 함수를 호출한 프로세스는 다른 프로세스가 signal() 함수를 호출할 때까지, 해당 조건에 맞는 조건 Queue에서 대기한다.
- `signal()` : 대기하고 있던 프로세스를 깨운다. 대기 중인 프로세스가 없으면 아무런 작업을 수행하지 않는다.

<img width="300" src="https://velog.velcdn.com/images/jxlhe46/post/c1275972-676a-4297-b055-28ba7c65dbdd/image.png"/>

<img width="500" src="https://velog.velcdn.com/images/jxlhe46/post/cde07f9f-14b9-40db-aa70-d946b7d787d4/image.png"/>


## 자바의 모니터락 

자바에서는 모니터와 비슷한, **스레드 동기화**를 위한 동시성 메커니즘인 **monitor-lock**을 제공한다. 

>- synchronized 키워드<br>
>- wait(), notify() 메서드 

**synchronized 키워드** 

- **임계 영역에 해당하는 코드 블록**을 선언할 때 사용하는 자바 키워드 
- 해당 코드 블록 (임계 영역)에는 **모니터락을 획득해야 진입 가능**
- 모니터락을 가진 **객체 인스턴스를 지정**할 수 있음.
- 메서드에 선언하면, **메서드 전체가 임계 영역으로 지정**됨. 이때, 모니터락을 가진 객체 인스턴스는 this 객체임. 

<img width="600" src="https://velog.velcdn.com/images/jxlhe46/post/caa201a7-eebe-485e-9565-9d75d4ab56b2/image.png"/>

**wait(), notify() 메서드**

- java.lang.Object 클래스에 선언되어 있으므로, 모든 자바 객체가 가진 메서드임. 
- 스레드가 어떤 객체의 wait() 메서드를 호출하면, 해당 객체는 모니터락을 획득하기 위해 대기 상태로 진입함. 
- 스레드가 어떤 객체의 notify() 메서드를 호출하면, 해당 객체 모니터에 대기 중인 스레드 하나를 깨움. 
- notify() 대신 notifyAll() 메서드를 호출하면, 해당 객체 모니터에 대기 중인 스레드를 전부 깨움. 

자바의 모니터락 활용 예시들은 [블로그 내용](https://velog.io/@jxlhe46/OS-Ch6-2.-%EB%8F%99%EA%B8%B0%ED%99%94-%EB%8F%84%EA%B5%AC#java-synchronization-example-1) 참고하기! 

# 면접 예상 질문 

<details>
<summary>동시성에 대해 설명해주세요.</summary>

싱글 코어 내에서 여러 개의 프로세스 또는 스레드가 번갈아가며 실행되어 마치 동시에 실행되는 것처럼 보이는 것 (논리적 개념)

여러 프로세스가 공유 자원에 동시 접근하는 문제가 생길 수 있어 프로세스 간의 동기화가 필요하다. 
</details>
<br>

<details>
<summary>병렬성에 대해 설명해주세요.</summary>

멀티 코어 내에서 여러 개의 프로세스 또는 스레드가 실제로 동시에 실행되는 것 (물리적 개념)

</details>
<br>

<details>
<summary>프로세스 동기화에 대해 설명해 주세요.</summary>

협력하는 프로세스 사이에서 공유 자원에 대한 일관성을 보장하는 것 

lock을 통해 공유 자원에 대한 접근을 제한함으로써, 한번에 하나의 프로세스만 공유 자원에 접근할 수 있게 한다. 

</details>
<br>

<details>
<summary>Critical Section에 대해 설명해주세요.</summary>

여러 프로세스가 동시에 사용할 수 없는 자원을 임계 자원, 이런 자원에 접근하는 코드의 일부분을 임계 구역이라고 한다.

임계 구역에서는 오직 단 하나의 프로세스만 실행되도록, 상호 배제를 위한 기법이 필요하다. 

</details>
<br>

<details>
<summary>Race Condition이 무엇인가요?</summary>

여러 프로세스가 동시에 공유 데이터에 접근하는 상황 

어떤 프로세스가 마지막으로 접근했는지에 따라 연산의 최종 결과가 달라지므로, 프로세스 간의 동기화 작업이 필요하다. 

</details>
<br>

<details>
<summary>Race Condition을 어떻게 해결할 수 있나요?</summary>

lock을 획득한 단 하나의 프로세스만 공유 자원에 접근할 수 있게 만든다. (프로세스 동기화)

</details>
<br>

<details>
<summary>Mutual Exclusion에 대해 설명해주세요.</summary>

임계 구역에서 실행 중인 프로세스가 있으면, 다른 어떤 프로세스도 임계 구역에 들어오지 못하도록 제한하는 것 

</details>
<br>

<details>
<summary>Mutual Exclusion을 할 수 있는 방법은?</summary>

고전적인 소프트웨어 솔루션으로는 **피터슨의 알고리즘**이 있다. 각 프로세스에 할당된 flag 배열과 turn 변수의 값을 서로 교차하면서 동기화를 수행할 수 있다. 

하드웨어적인 지원으로는 **원자적 연산을 보장**하는 test_and_set(), compare_and_swap() 같은 명령어가 제공된다. 

해당 명령어를 실행할 때는 컨텍스트 스위치가 발생하지 않음을 보장하여, 데이터의 비일관성 문제를 해결할 수 있다. 

운영체제에서 지원하는 **뮤텍스, 세마포어**, 프로그래밍 언어 레벨에서 지원하는 **모니터**를 통해서도 상호 배제를 구현할 수 있다. 

</details>
<br>

<details>
<summary>뮤텍스(Mutex)에 대해 설명해주세요.</summary>

상호 배제를 보장하기 위한 Locking 메커니즘으로, 뮤텍스 락을 획득한 프로세스만 임계 구역에 진입할 수 있다. 

- acquire(): 프로세스가 lock을 획득할 때까지 busy waiting, lock을 획득하면 다른 프로세스를 배제하기 위해 `available = false` 설정 
- release(): `available = true` 를 실행하여 busy waiting 하고 있던 프로세스를 깨운다.
- 두 연산은 원자적으로 수행되어야 한다. (하드웨어 연산으로 가능)
- 뮤텍스는 lock을 설정한 프로세스만 해당 락을 해제할 수 있다. (다른 프로세스가 대신 해제하는 것은 불가능)

</details>
<br>

<details>
<summary>세마포어에 대해 설명해주세요.</summary>

상호 배제를 보장하기 위한 Signaling 메커니즘으로, 세마포어 변수와 wait(), signal() 연산을 사용한다. 

- 세마포어 변수 S를 여러 프로세스가 사용 가능한 리소스 개수로 초기화 한다.
- wait(): 프로세스가 lock을 획득하여 리소스를 사용할 때 S 감소 
- signal(): 프로세스의 lock이 해제되어 리소스를 반납할 때 S 증가 
- 본인이 아닌 다른 프로세스에 의해서도 lock이 해제될 수 있다는 점에서 뮤텍스와 차이가 있다. 
- 종류로는 S가 0과 1만 왔다갔다 하는 바이너리 세마포어, S를 사용 가능한 리소스 개수로 설정하는 카운팅 세마포어가 있다. 

</details>
<br>

<details>
<summary>뮤텍스와 이진 세마포어의 차이에 대해 설명해주세요.</summary>

뮤텍스는 lock을 설정한 프로세스만 unlock을 할 수 있는 반면에

이진 세마포어는 lock을 설정한 프로세스와 unlock을 하는 프로세스가 서로 다를 수 있다. 

</details>
<br>

<details>
<summary>모니터에 대해 설명해주세요.</summary>

뮤텍스, 세마포어로 동기화를 직접 구현하면 오류가 발생할 가능성이 높다. 

이를 해결하기 위해 프로그래밍 언어 단에서 제공하는 추상 데이터 타입이 모니터다. 

공유 자원, 공유 자원을 이용한 연산, 초기화 코드로 구성되며, 한번에 단 하나의 프로세스만 모니터 내의 프로시저를 호출하여 공유 자원에 접근할 수 있다.

- wait(): 공유 자원에 접근하기 위해서, signal() 함수가 호출될 때까지 조건에 해당하는 큐에서 대기한다. 
- signal(): 큐에서 대기하고 있던 프로세스를 깨운다. 

</details>
<br>

<details>
<summary>데드락이 무엇인가요?</summary>

여러 프로세스가 서로의 자원을 필요로 해서 어떤 작업도 완료되지 못한 채 무한정 대기하는 상태 

</details>
<br>

<details>
<summary>데드락 발생 조건 4가지를 설명해 주세요.</summary>

- 상호 배제: 임계 구역에 단 하나의 프로세스만 들어갈 수 있다.
- 점유와 대기: 자신의 자원을 보유한 상태에서 다른 프로세스의 자원을 요청하는 프로세스가 있다. 
- 비선점: 현재 실행 중인 프로세스의 자원을 빼앗을 수 없다. 
- 순환 대기: 대기 중인 여러 개의 프로세스가 서로의 자원을 순환 형태로 요청한다. 

</details>
<br>

<details>
<summary>데드락을 막는 방법에 대해 설명해주세요.</summary>

- 예방: 프로세스에 자원을 할당할 때, 데드락의 발생 조건 4가지 중에 하나라도 발생하지 않도록 예방하는 것 
- 회피: 프로세스의 자원 요청에 대한 부가적인 정보를 통해, 데드락이 발생할 가능성이 없을 때만 자원을 할당하는 것 
- 탐지 및 회복: 데드락의 발생을 허용하되, 발생 여부를 탐지하여 이를 회복하는 것 (프로세스 종료 또는 자원 선점)
- 무시: 데드락이 발생해도 아무런 처리를 하지 않는 것 (현대적인 OS에서 채택하는 방법)

</details>
<br>

# 참고자료 

https://velog.io/@jxlhe46/OS-Ch6-1.-동기화-도구

https://velog.io/@jxlhe46/OS-Ch6-2.-동기화-도구

https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/main/OS

https://yoongrammer.tistory.com/65