# CPU 스케줄링 

CPU 스케줄링은 멀티 프로그래밍 운영체제에서 필수적이다. 멀티 프로그래밍의 목적은 **동시에 여러 개의 프로세스를 돌려서 CPU 사용 효율을 최대화** 하는 것이다.

<img width="700" src="https://github.com/user-attachments/assets/9948b4d8-cf90-4fed-a5ca-545532092a34"/> 

프로세스는 CPU burst time과 I/O burst time을 왔다갔다 하면서 상태가 바뀌는데, 주로 CPU burst time일 때는 running 상태, I/O burst time일 때는 waiting 상태이다.

오른쪽의 그래프를 보면, CPU bound 보다 I/O bound의 빈도 수가 더 높다는 걸 알 수 있다.

### 용어 정리 

- CPU burst time : 프로세스가 CPU에서 연속적으로 실행되는 시간
- I/O burst time : 입출력 작업이 완료될 때까지 프로세스가 대기하는 시간
- CPU bound process : CPU burst time이 길고, I/O burst time이 상대적으로 짧은 프로세스
- I/O bound process : I/O burst time이 길고, CPU burst time이 상대적으로 짧은 프로세스

### 프로세스 생명주기

- New : 새로운 프로세스가 생성된 상태 ex) fork()
- Running : CPU를 점유하여 프로그램을 실행하고 있는 상태
- Waiting : 입출력 연산이 완료될 때까지 Waiting Queue에서 대기하고 있는 상태
- Ready : CPU를 점유할 준비가 다 되어서 Ready Queue에서 대기하고 있는 상태
- Terminated : 프로세스가 종료된 상태 ex) exit(), return 0;

# CPU 스케줄러

CPU 스케줄러는 **Ready 상태의 프로세스 중에 어떤 프로세스를 가장 먼저 CPU에 패치시켜 실행시킬 것인지 결정**한다. 

구체적인 스케줄링 알고리즘에 대해 알아보기 전에 몇가지 용어부터 정리해보자. 

## 선점형 vs 비선점형 

- 선점형 스케줄링 : 스케줄러가 프로세스를 선점할 수 있다. (스케줄러가 프로세스를 강제로 쫓아낼 수 있다.)
- 비선점형 스케줄링 : 프로세스가 종료되거나 Waiting 상태로 전환되어 자발적으로 나오기 전까지는 계속 CPU를 선점하고 있다. (스케줄러가 프로세스를 강제로 쫓아내지 못한다.)

1. running → waiting
2. running → ready
3. waiting → ready
4. terminated

1, 4번의 경우, 자발적으로 프로세스가 CPU에서 나오기 때문에 비선점형이다. 2, 3번의 경우, 스케줄러가 프로세스를 강제로 쫓아낼 수도 있으므로 선점형 또는 비선점형이다.

## 디스패처

CPU 디스패처는 **한 프로세스에서 다른 프로세스로 문맥을 교환**하고, 유저 모드로 전환하여 유저 프로그램을 재개하기 위해 적절한 위치로 점프하는 등의 기능을 수행한다. 

디스패처는 **문맥 교환이 있을 때마다 호출**되기 때문에 가능한 속도가 빨라야 한다. 한 프로세스를 정지시키고 다른 프로세스를 실행시킬 때까지 걸리는 시간인 **디스패치 지연시간** (dispatch latency)를 최소화 하는 것이 중요하다. 

<img width="600" src="https://github.com/user-attachments/assets/0828a33d-01c3-4857-8d7f-bf3d94c23602"/> 

# CPU 스케줄링 알고리즘

## 스케줄링의 목표

- CPU utilization (사용 효율) 최대화
- Throughput (처리량, 단위시간당 처리한 프로세스의 개수) 최대화
- Turnaround time (반환 시간, 요청을 보내고 그것이 완료될 때까지 걸린 시간) 최소화
- Waiting time (프로세스가 ready queue에서 대기한 시간) 최소화
- Response time (응답 시간, 요청에 응답하는 데 걸리는 시간) 최소화

## FCFS: First-Come, First-Served

### 정의 

- **Ready Queue에 도착한 프로세스 순서대로 CPU 선점** (선착순)
- 구현하기 가장 간단한 CPU 스케줄링 알고리즘 (FIFO 큐로 구현 가능)

### 특징 

- 프로세스가 자신의 작업이 완료될 때까지 CPU를 반납하지 않으므로 **비선점형** 알고리즘 
- CPU burst time이 가장 큰 CPU bound 프로세스가 첫번째로 CPU를 선점하면, 그 뒤의 모든 I/O bound 프로세스들은 계속 대기해야 하므로 비효율적이다. (Convoy Effect)

## SJF: Shortest Job First

### 정의 

**CPU burst time이 가장 작은 프로세스부터 먼저 처리하는 알고리즘** (CPU burst time이 동일하면, FCFS에 따라 선착순으로 처리)

### 특징 

- **비선점형** 알고리즘 
- CPU burst time이 상대적으로 긴 프로세스가 Ready Queue에서 무한정 대기하는, **기아 현상** (starvation)이 발생할 수 있다. 

## SRTF: Shortest Remaining Time First

### 정의 

**선점형 SJF 알고리즘**으로, **현재 실행되고 있는 프로세스의 남은 시간보다 새로 들어온 프로세스의 CPU burst time이 더 작을 때, 새로운 프로세스가 CPU를 선점**한다. 

### 특징 

- **선점형** 알고리즘 
- SJF와 마찬가지로 **기아 현상** 발생 가능 
- 새로운 프로세스가 도달할 때마다 스케줄링을 다시 하기 때문에, CPU burst time 측정이 어렵다.

## Priority-base

### 정의 

- **프로세스에 주어진 우선순위에 따라 스케줄링 한다.** (여러 프로세스의 우선순위가 동일하다면 FCFS 알고리즘 수행) 
- **SJF, SRTF 모두 우선순위 기반 스케줄링**의 일종이다. CPU burst time이 작을수록 우선순위가 높은 것이다. 

### 특징 

- 선점형 방식 : 더 높은 우선순위의 프로세스가 Ready Queue에 도착하면, 실행 중인 프로세스를 멈추고 새로운 프로세스가 CPU를 선점한다. 
- 비선점형 방식 : 더 높은 우선순위의 프로세스가 도착하면, Ready Queue의 Head 위치에 프로세스를 배치한다. 
- 우선순위가 낮은 프로세스가 Ready Queue에서 무한정 대기하는 **기아 현상** 발생 가능 
- 기아 현상의 한 가지 해결 방법으로는, 프로세스의 대기 시간에 비례해서 우선순위를 점진적으로 늘려주는 **Aging**이 있다. 

## RR: Round-Robin (시분할)

### 정의 

- **할당 시간(time quantum)이 있는 선점형 FCFS** 
- 스케줄러가 원형 큐인 Ready Queue를 돌면서 **time quantum 간격으로 각 프로세스에 CPU를 할당**한다. 
- 현대적인 CPU 스케줄링 알고리즘 

### 특징 

- **선점형** 알고리즘
- 프로세스의 CPU burst time이 time quantum보다 길면, 해당 프로세스는 스케줄러에 의해 선점되고 Ready Queue의 tail로 추가된다.
- **time quantum 크기에 따라 성능이 크게 달라진다.** 할당 시간이 너무 길면 FCFS랑 동일해지고, 할당 시간이 너무 작으면 프로세스 간의 컨텍스트 스위칭이 빈번하게 발생하여 dispatch latency가 커진다. 


# 참고자료 

https://velog.io/@jxlhe46/OS-Ch05.-CPU-스케줄링

https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/main/OS