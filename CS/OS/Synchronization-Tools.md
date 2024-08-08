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

# 임계 구역 문제 (Critical Section Problem)

여러 개의 프로세스들은 임계 구역(Critical Section)이라 불리는 코드 영역에서 **공유 데이터에 접근하거나 해당 값을 변경**한다. 

여기서 중요한 점은 **하나의 프로세스가 임계 구역에서 실행되고 있으면, 다른 어떤 프로세스도 임계 구역에서 실행될 수 없다**는 것이다. 이를 통해 여러 개의 프로세스를 **동기화** 시킬 수 있다. 

<img width="500" src="https://github.com/user-attachments/assets/b70b96cc-dec7-43b0-96f3-a55e452c2af5"/>

## 임계 구역 문제를 해결하기 위한 3가지 조건 

### 1️⃣ Mutual Exclusion

**한 프로세스가 임계 구역에서 실행되고 있다면, 다른 어떤 프로세스도 임계 구역에서 실행될 수 없다.** 이러한 **상호 배제**는 프로세스 간 경쟁 상태를 막기 위해 기본적으로 갖춰야 할 조건이다.

그런데, 상호 배제로 발생할 수 있는 두 가지 문제가 있다. 바로 **데드락과 기아 상태**이다. 각 문제를 해결하기 위한 방법이 Progress, Bounded Wating이다.

### 2️⃣ Progress (데드락 회피)

**임계 구역에서 실행 중인 프로세스가 없다면, 다음 프로세스가 임계 구역에 진입할 수 있게 해주는 방법이다.** 그렇지 않으면, 다음 프로세스들이 무한정 대기하는 **데드락**이 발생할 수 있다.

### 3️⃣ Bounded Wating (기아 상태 회피)

**각 프로세스가 임계 구역에 들어올 수 있는 횟수를 제한하는 방법이다.** 그렇지 않으면, 특정 프로세스가 임계 구역에 계속 들어오지 못하고 무한정 대기하는 **기아 상태**가 발생할 수 있다. 

### Deadlock (교착 상태)

교착 상태는 **여러 프로세스가 서로의 자원을 필요로 해서 어떤 작업도 완료하지 못한 채 무한 대기 상태에 빠지는 것**을 말한다.

다음 네 가지 조건을 모두 만족시키면 데드락이 발생한다. 

1. Mutual Exclusion (상호 배제): 한번에 하나의 프로세스만 자원을 사용할 수 있다. 
2. Hold and Wait (점유와 대기): 자신의 자원을 보유한 상태에서 다른 프로세스의 자원을 점유하기 위해 대기하는 프로세스가 있어야 한다.
3. No Preemption (비선점): 프로세스가 점유한 자원을 강제로 뺏을 수 없다.
4. Circular Wait (순환 대기): 대기하고 있는 프로세스 집합이 서로의 자원을 순환 형태로 요청하고 있다.

예를 들어, 프로세스 A가 자원 1을 점유한 상태에서 자원 2를 기다리고 있는데, 프로세스 B가 자원 2를 점유한 상태에서 자원 1을 기다리고 있다면 두 프로세스 모두 데드락에 빠진다. 

데드락은 다음과 같이 해결할 수 있다. 

- 예방 : 데드락의 발생 조건 4가지 중에 하나라도 발생하지 않도록 예방하는 방법 
- 회피 : 데드락의 발생 가능성을 인정하면서도 적절하게 회피하는 방법 
- 탐지 및 회복 : 데드락의 발생 여부를 탐지하고, 발생 시 프로세스를 1개 이상 종료시키거나 자원을 선점하여 데드락으로부터 회복하는 방법 

### Starvation (기아 상태)

기아 상태는 **특정 프로세스가 오랫동안 자원을 할당받지 못해 무한 대기 상태에 빠지는 것**을 말한다. 

주로 **CPU 스케줄링이나 자원 할당 정책의 불공정성** 때문에 발생한다. 대표적으로, **우선순위 기반 스케줄링**에서 낮은 우선순위의 프로세스가 계속해서 높은 우선순위의 프로세스에 의해 자원을 할당받지 못할 때 발생한다. 

기아 현상의 해결 방법 중에 하나는, **프로세스의 대기 시간에 비례해서 우선순위를 점진적으로 늘려주는 Aging**이 있다. 

# 뮤텍스와 세마포어 

임계 구역 문제 (Critical Section Problem, CSP)를 해결하기 위한 고수준의 소프트웨어 도구인 뮤텍스, 세마포어에 대해 알아보자.

## 뮤텍스 

- Mutex: **mut**ual **ex**clusion
- 상호 배제를 통해 임계 구역의 동기화를 보장하고 경쟁 상태를 막기 위한 도구
- **프로세스는 임계 구역에 들어가기 위해 lock을 얻어야 하고, 임계 구역에서 나올 때는 lock을 반납해야 한다.** (lock은 임계 구역에 들어가기 위한 열쇠 같은 개념)
- Mutex Lock을 구현하려면 `acquire()`, `release()` 함수와 lock의 가용 여부를 알려주는 `available` 변수만 있으면 된다. 

<img width="400" src="https://github.com/user-attachments/assets/72e9d8fb-1fb4-4614-b2e2-7ee33da35d62"/>

<img width="600" src="https://github.com/user-attachments/assets/780d7da9-014f-41de-86a2-0cb126f32de1"/>

- `acquire()`: lock을 얻을 수 없을 때는 busy waiting 하다가, release() 되어 lock을 얻을 수 있으면 다른 프로세스가 임계 영역에 들어오지 못하도록 `available = false`로 설정
- `release()`: 특정 프로세스가 lock을 얻을 수 있도록 `available = true`로 설정
- 위의 두 함수를 호출하는 것은 **원자적으로** 수행되어야 한다. 즉, 중간에 컨텍스트 스위치가 발생하면 안 된다.
- **Busy Waiting** : 임계 구역에 들어가려는 프로세스가 lock을 얻기 위해 무한히 대기하는 상태
- **Spinlock** : Busy Waiting 상태의 뮤텍스 락 (Spinlock으로 대기하고 있을 때는 컨텍스트 스위치가 발생하지 않는다.)

## 세마포어 

2개의 프로세스를 일반화 시켜서 **n개의 프로세스를 동기화 할 때 사용되는 도구**가 바로 **세마포어**다. (semaphore: 신호 장치, 신호기)

<img width="600" src="https://github.com/user-attachments/assets/d76c5d9f-ceda-4bbc-bd27-c292e8658c3a"/>

- 세마포어 S는 **사용 가능한 리소스의 개수**를 의미하며, `wait()`, `signal()`이라는 **오직 2개의 원자적 연산**으로만 접근할 수 있다. 
- 프로세스가 lock을 얻어서 리소스를 사용할 때는 `wait()`의 S를 감소시키고, 프로세스가 리소스를 해제할 때는 `signal()`의 S를 증가시킨다.

### 세마포어의 종류 

- **Binary Semaphore** : S의 값이 0과 1만 왔다갔다 하기 때문에, mutex lock과 비슷하다.
- **Counting Semaphore** : S 값의 범위가 무한대이고, 유한개의 인스턴스를 갖는 리소스에 사용될 수 있다.
  - 사용 가능한 리소스의 개수로 세마포어 S를 초기화 한다.
  - 프로세스가 lock을 얻어서 리소스를 사용할 때는 wait()의 S를 감소시키고, 프로세스가 리소스를 해제할 때는 signal()의 S를 증가시킨다.
  - S가 0이면 모든 리소스가 사용 중이기 때문에, 리소스가 필요한 프로세스는 S가 0보다 커질 때까지 블로킹 되어 대기한다.

### 세마포어로 동기화 구현하기 

프로세스 P1, P2가 동시에 돌아가고 있으며, P1의 작업은 S1, P2의 작업은 S2라고 하자. 이 상황에서 S2는 반드시 S1이 모두 끝나고 나서 실행되어야 한다. 

이를 위해 프로세스 P1, P2가 0으로 초기화 된 synch라는 세마포어를 공유하게 만든다. 그리고 S1이 모두 끝나고 signal() 함수에서 synch 값을 증가시키면, wait() 함수에서 busy waiting 하고 있던 S2가 깨어나서 실행된다. 

<img width="500" src="https://github.com/user-attachments/assets/c986fe5d-c270-4e24-ab30-e9ae84ce83ad"/>

### 세마포어의 문제점  

세마포어는 동기화를 위한 편리하고 효과적인 도구이긴 하지만, 여러가지 **타이밍 문제가 발생**할 수 있다. 즉, 실행 시퀀스가 제대로 수행되지 않을 때가 있는데 이런 문제는 발견하기 매우 어렵다. 

예를 들어, 1로 초기화 된 이진 세마포어 mutex를 모든 프로세스가 공유하고 있는 상황을 생각해보자. 각 프로세스는 임계 구역에 들어가기 전에 wait(mutex)로 대기하고, 들어간 후에는 signal(mutex)로 mutex의 값을 증가시킨다. 그런데, 이러한 **실행 시퀀스가 제대로 관찰되지 않으면, 두 프로세스는 임계 구역에 동시에 진입**할 수 있다. 

>ex1) wait(), signal()을 호출하는 순서가 바뀔 수 있다.<br>
ex2) wait()와 signal() 중에 하나만 두 번 호출할 수도 있다. <br>
ex3) 프로세스가 wait(), signal() 중에 하나를 호출하지 않거나, 둘 다 호출하지 않을 수도 있다. <br>

<img height="150" src="https://velog.velcdn.com/images/jxlhe46/post/1928bc48-7f5c-47bc-83ec-f9fb9f59b365/image.png"/> <img height="150" src="https://velog.velcdn.com/images/jxlhe46/post/06d417e8-52cf-4aba-8c78-4222ed24247b/image.png"/>

이런 문제들은 프로그래머가 세마포어나 뮤텍스락을 부정확하게 사용하면 매우 쉽게 발생할 수 있다. 이를 해결하기 위해 고수준의 언어에서 동기화 도구로 '모니터'를 제공한다. 

# 모니터

모니터는 서로 다른 프로세스 간의 **상호 배제를 보장**하는 추상 데이터 타입 (ADT)이다. 

세마포어는 개발자가 wait(), signal() 함수로 직접 동기화를 구현해야 하지만, 모니터는 **프로그래밍 언어 단에서 내부적으로 동기화 처리**를 해주기 때문에 사용법이 간단하고 실수할 가능성이 적다. 

## 모니터의 구성

모니터는 클래스, 구조체와 유사한 개념이며, 아래 그림처럼 **공유 데이터, 공유 데이터를 이용한 연산들, 초기화 코드**로 이루어져 있다. 

<img height="300" src="https://velog.velcdn.com/images/jxlhe46/post/21a93daf-e002-4e99-a759-5653cf0e7212/image.png"/> <img height="300" src="https://velog.velcdn.com/images/jxlhe46/post/fdf99a53-9b47-49b5-96ca-ec1c11fe9f3d/image.png"/>


## 모니터의 특징

- **상호 배제**: 한번에 하나의 프로세스만 모니터 객체 내의 프로시저를 호출할 수 있다. 
- **캡슐화**: 프로세스는 모니터 객체 내의 공유 자원에 직접적으로 접근 불가능하며, 프로시저를 통해서만 가능하다. 
- **추상화**: 모니터 내부에 동기화 코드가 구현되어 있으므로, 뮤텍스, 세마포어처럼 개발자가 직접 동기화 코드를 작성하지 않아도 된다. 

## 조건 변수 

기본적인 모니터 구조에서는 식사하는 철학자 문제 등 고전적인 동기화 문제를 해결하는데 한계가 있다. 그래서 조건 변수 (Conditional Variable)가 추가되었다. 이 변수는 추가적인 동기화 메커니즘을 제공한다. 

아래 코드처럼 하나 이상의 조건 변수를 정의할 수 있고, 이러한 변수들이 호출할 수 있는 유일한 함수는 wait(), signal()이다. 

- wait() : 이 함수를 호출한 프로세스는 다른 프로세스가 signal() 함수를 호출할 때까지, 해당 조건에 맞는 조건 Queue에서 대기한다.
- signal() : 대기하고 있던 프로세스를 깨운다. 대기 중인 프로세스가 없으면 아무런 작업을 수행하지 않는다.

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
- 스레드가 어떤 객체의 wait() 메서드를 호출하면, 해당 객체의 모니터락을 획득하기 위해 대기 상태로 진입함. 
- 스레드가 어떤 객체의 notify() 메서드를 호출하면, 해당 객체 모니터에 대기 중인 스레드 하나를 깨움. 
- notify() 대신 notifyAll() 메서드를 호출하면, 해당 객체 모니터에 대기 중인 스레드를 전부 깨움. 

자바의 모니터락 활용 예시들은 [블로그 내용](https://velog.io/@jxlhe46/OS-Ch6-2.-%EB%8F%99%EA%B8%B0%ED%99%94-%EB%8F%84%EA%B5%AC#java-synchronization-example-1) 참고하기! 

# 참고자료 

https://velog.io/@jxlhe46/OS-Ch6-2.-동기화-도구

https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/main/OS

https://yoongrammer.tistory.com/65