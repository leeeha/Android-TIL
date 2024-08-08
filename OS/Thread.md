# 스레드

프로세스는 여러 개의 실행 흐름을 가질 수 있는데, 이러한 흐름을 스레드(Thread)라고 부른다.

스레드는 프로세스 메모리의 **Code, Data, Heap 영역을 다른 스레드와 공유**하며, **스레드 ID, 프로그램 카운터, 레지스터 집합, Stack 영역은 각 스레드마다 독립적으로 할당**된다. 

<img width="700" src="https://github.com/user-attachments/assets/c891984b-e0fb-4018-8edc-24067f9f59cf"/>

**스택 영역을 스레드마다 독립적으로 할당하는 이유**

프로세스의 스택 영역은 함수 매개변수, 지역 변수, 리턴 주소 등 함수 호출에 사용되는 임시 데이터들을 저장하는 메모리 공간이다. 

따라서, 스레드마다 독립적인 실행 흐름 및 함수 호출이 가능하려면 스택 영역은 별도로 가지고 있어야 한다.

**PC 레지스터를 스레드마다 독립적으로 할당하는 이유**

PC 레지스터에는 다음에 실행될 명령어의 주소가 저장되어 있다. 스레드는 CPU를 점유하고 있다가 스케줄러에 의해 선점 당하면 다른 스레드로 대체된다. 

이처럼 프로그램의 명령어가 연속적으로 수행되지 못하기 때문에 어디까지 수행했는지 기억할 필요가 있다. 따라서, PC 레지스터는 스레드마다 독립적으로 할당된다.

# 멀티 프로세스 vs 멀티 스레드 

|  | 멀티 프로세스 | 멀티 스레드 |
| --- | --- | --- |
| 메모리 공유 | X | Code, Data, Heap 영역 공유 (Stack 영역은 독립적)  |
| 문맥 교환 | 비용이 크다.  | 비교적 비용이 작다. |
| 동기화 필요  | X | 동시에 여러 스레드가 Data, Heap 영역의 데이터에 접근하여 비일관적인 결과가 나올 수 있으므로, 동기화 작업이 필요함. 단, 과도한 락으로 인한 병목 현상을 주의해야 함.  |
| 전파 | 한 프로세스가 종료되어도 다른 프로세스에는 영향을 미치지 않음.  | 한 스레드가 종료되면 다른 스레드도 종료될 수 있음.  |

멀티 프로세스와 멀티 스레드 방식 모두 장단점이 있으므로, 대상 시스템의 특징에 따라 적합한 동작 방식을 선택하고 적용해야 한다.

# 스레드 제어 

## 스레드 대기 

join() 함수로 부모 스레드가 자식 스레드가 끝날 때까지 대기하게 만들 수 있다. 

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

interrupt() 함수로 실행 중인 스레드에 인터럽트를 걸어서 스레드를 종료시킬 수 있다. 

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

# 동시성 vs 병렬성 

- 동시성(Concurrency): 싱글 코어에서 여러 스레드가 빠르게 전환되어 동시에 실행되는 것처럼 보이는 것 
- 병렬성(Parallelism): 멀티 코어에서 각 코어 내의 스레드가 실제로 동시에 실행되는 것 

<img width="700" src="https://github.com/user-attachments/assets/5feaf758-e134-4d1a-8524-51668bbe059b"/> 

# 참고자료 

- https://velog.io/@kangdev/기술면접운영체제-Process와-Thread의-메모리-공유-규칙
- https://velog.io/@jxlhe46/OS-Ch4.-쓰레드