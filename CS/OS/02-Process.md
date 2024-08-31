# 폰 노이만 아키텍처 

**프로그램**은 컴퓨터 하드웨어에게 어떤 작업을 수행하도록 시키는 **명령어들의 집합**이다. 이를 메모리에 올려서 CPU에 패치시키고 실행하면, **실행 중인 프로그램**인 **프로세스**가 된다.

이처럼 프로그램을 메모리에서 CPU로 패치(patch)시키고 실행(execute)한 다음, 그 결과를 다시 메모리에 저장하는 구조를 **폰 노이만 아키텍처** 또는 ISA(Instruction Set Architecture)라고 부른다. 

폰 노이만은 **메모리에 프로그램을 저장**하는 **내장형 프로그램** 방식을 처음으로 도입한 사람이다.

<img width="450" src="https://github.com/user-attachments/assets/092fad6d-0c43-490f-b1da-2b4a78e31f48"/>

# 프로세스 

프로세스는 **실행 중인 프로그램**을 의미하며, 운영체제에서 **작업의 단위**가 된다. 

프로세스는 자신의 작업을 수행하기 위해 특정한 **리소스** (CPU, 메모리, 파일, 입출력 장치 등)을 필요로 하며, 운영체제는 이를 관리하는 역할을 한다.

## 메모리 레이아웃

<img width="550" src="https://github.com/user-attachments/assets/eef00fbf-a28e-4895-bfb0-deb1de6ff07f"/>

### Code (Text) 영역  

- **실행 가능한 프로그램 코드**가 저장되는 영역으로, Text 영역이라고도 부른다. 
- **컴파일 타임**에 메모리 공간이 할당되며, 코드를 바꿀 수 없도록 **Read-Only**로 지정되어 있다.

### Data 영역 

- **프로그램의 전역 변수, 정적 변수**가 저장되는 영역이다. 즉, 프로그램이 실행되는 동안 항상 접근 가능한 변수들이 저장되는 영역이다. 
- 프로그램 실행 중에 전역 변수가 변경될 수도 있으므로, **Read-Write**로 지정되어 있다. 
- 초기화 된 데이터는 Data 영역에 저장되고, 초기화 되지 않은 데이터는 BSS 영역에 저장된다. 

<details>
<summary>BSS 영역이 존재하는 이유</summary>

**메모리 공간을 효율적으로 사용하기 위함이다.** 

>int arr[100000]; // 전역으로 선언했다고 가정 

위의 코드에서 배열 arr의 크기는 400000 바이트다. 하지만 선언만 하고 아직 값이 들어있지 않기 때문에, 굳이 400000B 라는 메모리 공간을 할당할 필요가 없다. 

따라서 배열 arr의 크기와 이름에 대한 정보는 BSS 영역에 저장하여, 메모리 공간을 더 효율적으로 사용할 수 있다. 

</details>

### Heap 영역 

- **런타임에 크기가 동적으로 할당**되는 메모리 영역이다. 
- 주로 **참조 타입의 실제 데이터**가 저장된다. 
- 메모리의 낮은 주소부터 높은 주소 방향으로 할당된다. 
- 더 이상 사용하지 않는 메모리 공간을 반환하지 않으면, **메모리 누수**가 발생할 수 있다.

### Stack 영역 

- **함수 호출에 사용되는 임시 데이터**를 저장하는 메모리 영역이다. (함수 매개변수, 지역 변수 등)
- 함수 호출과 함께 할당되며, 함수가 종료되면 같이 소멸한다.
- 원시 타입의 데이터, Heap 영역에 있는 **참조 타입을 가리키는 참조 값**이 저장된다.
- 메모리의 높은 주소에서 낮은 주소 방향으로 할당된다.
- **컴파일 타임에 크기가 결정**되므로 무한히 할당할 수 없다. 재귀함수가 너무 깊게 호출되거나 함수가 지역 변수를 너무 많이 가지고 있어, 스택 영역을 초과하면 Stack Overflow가 발생한다. 
- 사실 **Heap, Stack 영역은 같은 메모리 공간을 공유**한다. 그래서 서로의 영역을 침범하는 경우 Heap Overflow, Stack Overflow가 발생한다. 

## 생명 주기 

프로세스가 실행될 때 그 상태가 바뀌는데, 이를 프로세스의 생명 주기라고 한다. 

- `New` : 새로운 프로세스가 생성된 상태 - fork() 
- `Running` : CPU를 점유하여 프로그램을 실행하고 있는 상태 
- `Waiting` : 입출력 또는 이벤트가 완료될 때까지 Waiting Queue에서 대기하고 있는 상태 
- `Ready` : CPU를 점유할 준비가 다 돼서 Ready Queue에서 대기하고 있는 상태 
- `Terminated` : 프로세스가 종료된 상태 - exit(), return 0;

<img width="600" src="https://github.com/user-attachments/assets/0f5c0430-d0d6-43f1-b1c4-58b8f153c864"/>

## PCB (Process Control Block)

운영체제는 PCB라는 구조체를 만들어서 **프로세스와 관련된 모든 정보를 저장**한다.

- 프로세스 식별 번호 (Process ID, PID)
- 프로세스 상태 (New, Running, Waiting, Ready, Terminated)
- 프로그램 카운터 (다음에 실행될 명령어의 주소를 저장하고 있는 레지스터)
- CPU 레지스터 
- CPU 스케줄링 정보 
- 메모리 관리 정보 
- 입출력 상태 정보 
- 계정 정보 

<img width="500" src="https://github.com/user-attachments/assets/85db5492-42a3-4d4a-a231-50af8fb90944"/>

# 프로세스 스케줄링 

## 스케줄링 큐 

- 프로세스가 시스템에 진입하면 `Ready Queue`에 들어가게 되는데, 레디 큐 안의 프로세스들은 **CPU 코어에서 실행될 준비를 마치고 대기**하고 있다. 
- **입출력 작업이나 특정 이벤트가 완료될 때까지 대기**하고 있는 프로세스들은 `Waiting Queue`에 들어간다.
- 이러한 큐들은 일반적으로 **PCB의 연결 리스트**로 구현된다.

<img width="700" src="https://github.com/user-attachments/assets/ca3269bd-6f9e-485c-9a19-fb9485de83f5"/>

<img width="700" src="https://github.com/user-attachments/assets/d2b8d743-9d75-41b2-b6bd-8f9c62e5ea50"/>

## 문맥 교환 (Context Switching)

**프로세스가 사용되고 있는 상태**를 **문맥**이라고 하는데, 이 정보는 다 PCB에 저장되어 있다. 따라서, **운영체제 입장에서 문맥, 컨텍스트는 결국 PCB를 의미한다**고 볼 수 있다. 

인터럽트가 발생하면 시스템은 **현재 실행 중인 프로세스의 상태를 저장하고, 다른 프로세스의 상태를 복원**한다. 즉, 문맥 교환은 **CPU의 제어권을 한 프로세스에서 다른 프로세스로 넘겨주는 과정**을 의미한다.

<img width="700" src="https://github.com/user-attachments/assets/9022403d-4bc3-4c8a-922f-397e4a567c71"/>

위의 그림처럼 프로세스 P0를 실행하다가 인터럽트가 발생하면 PCB0에 프로세스 P0의 상태를 저장하고

PCB1으로부터 프로세스 P1의 상태를 복원하여 실행시키는 것을 프로세스의 문맥 교환이라고 한다.

## 프로세스의 생성과 종료 

### 프로세스의 생성 

- 부모 프로세스는 자식 프로세스를 생성할 수 있다. - fork()
- 부모 프로세스는 자식 프로세스와 함께 동시에 실행되거나, 자식 프로세스가 종료될 때까지 대기한다. - wait()
- 자식 프로세스는 부모 프로세스의 복사본이거나, 아예 새로운 프로세스일 수 있다. 

```c
#include <stdio.h>
#include <unistd.h>
#include <wait.h>

int main(){
    pid_t pid;

	// fork a child process
	pid = fork();

	if(pid < 0){ // error
		fprintf(stderr, "Fork Failed");
		return 1;
	}
	else if(pid == 0){ // child process
		execlp("/bin/ls", "ls", NULL);
	}
	else{ // parent process
		// parent will wait for the child to complete.
		wait(NULL);
		printf("Child Complete");
	}
	
    return 0;
}
```

<img width="750" src="https://github.com/user-attachments/assets/1c879fd5-6235-46cb-b381-c8c52cdeaf2a"/>

### 프로세스의 종료 

프로세스는 return문으로 정상 종료되거나, exit()이라는 시스템 콜로 강제 종료될 수 있다. 그러면 OS는 프로세스에 할당되었던 메모리를 해제하고, 파일, 입출력 버퍼 등 모든 리소스를 회수한다. 

### 좀비, 고아 프로세스

- 좀비 프로세스 : 부모 프로세스가 wait()을 호출하지 않았는데, 자식 프로세스가 먼저 종료되는 경우. 자식 프로세스는 이미 종료되었지만 부모 프로세스가 그 상태를 회수하지 않아서, 마치 자식이 죽었는데 다시 살아난 거 같은 '좀비 프로세스'가 된다. (ex. 데몬 프로세스, 백그라운드 프로세스)
- 고아 프로세스 : 부모 프로세스가 wait()을 호출하지 않고 먼저 종료된 경우, 자식 프로세스는 부모를 잃은 고아 프로세스가 된다. 

# 프로세스 간 통신 

IPC (Inter-Process Communication)는 결국 **데이터를 주고 받는 것**이며, 두 가지 기본적인 모델이 있다. 

- 공유 메모리 (Shared Memory)
- 메시지 전달 (Message Passing)

<img width="700" src="https://github.com/user-attachments/assets/bb1393ed-fbf2-4f40-bfdb-add5ecf1008b"/>

## 공유 메모리 방식

- 생산자가 공유 메모리의 버퍼에 데이터를 채우면, 소비자가 이를 소비하는 방식 
- 프로그래머가 명시적으로 공유 메모리에 접근하거나 조작하는 코드를 작성해야 해서 다소 번거롭다. 
- 실제 구현: POSIX Shared Memory 

<img width="700" src="https://github.com/user-attachments/assets/1ae0152c-46f8-45f8-ab32-57cb6d324cfd"/>

## 메시지 전달 방식 

- 한 프로세스가 OS 커널의 메시지 큐에 메시지 객체를 넣어두면, OS가 알아서 다른 프로세스에게 해당 메시지를 전달한다. 
- OS가 메시지 송수신에 필요한 API를 제공하므로 편리하게 사용할 수 있다. 
  - send(message)
  - receive(message)
- 프로세스 간 통신에 사용되는 링크를 구현하는 여러가지 방식 
  - direct or indirect communication
  - synchronous or asynchronous communication
  - automatic or explicit buffering
- 실제 구현: 파이프, 소켓, RPC (Remote Procedure Call)

# 면접 예상 질문

<details>
<summary>프로그램에 대해 설명해주세요.</summary>

프로그램은 컴퓨터 하드웨어에게 어떤 작업을 수행하도록 시키는 **명령어들의 집합**이다.

이를 실행시키려면 운영체제로부터 CPU, 메모리 같은 리소스를 할당 받아야 한다. 

</details>
<br>

<details>
<summary>프로세스에 대해 설명해주세요.</summary>

메모리에 적재된 프로그램이 CPU에 패치되어 실행될 때, 그 **실행 중인 프로그램**을 프로세스라고 부른다. 	

</details>
<br>

<details>
<summary>프로세스의 메모리 공간에 대해 설명해주세요.</summary>

- Code (Text) 영역: 실행 가능한 프로그램 코드가 저장되는 영역 (Read-Only)
- Data 영역: 프로그램의 전역 변수, 정적 변수가 저장되는 영역 (Read-Write)
- Heap 영역: 프로그램이 실행되는 런타임에 크기가 동적으로 할당되는 영역 
- Stack 영역: 컴파일 타임에 크기가 결정되며, 함수 매개변수, 지역변수 등 함수 호출에 사용되는 임시 데이터를 저장하는 영역 

</details>
<br>

<details>
<summary>프로세스 수행 상태 변화 과정에 대해 설명해주세요.</summary>

- New: 새로운 프로세스가 생성된 상태 
- Running: CPU를 점유하여 프로그램이 실행되고 있는 상태 
- Waiting: I/O 작업이 완료될 때까지 Waiting Queue에서 대기하고 있는 상태 
- Ready: CPU를 점유할 준비가 다 되어서 Ready Queue에서 대기하고 있는 상태 
- Terminated: 프로세스가 종료된 상태 

<img width="600" src="https://github.com/user-attachments/assets/0f5c0430-d0d6-43f1-b1c4-58b8f153c864"/>

</details>
<br>

<details>
<summary>프로세스 제어 블록(PCB)에 대해 설명해주세요.</summary>

프로세스를 실행하는 데 필요한 정보를 담고 있는 구조체 

- 프로세스 식별 번호 (Process ID, PID)
- 프로세스 상태 (New, Running, Waiting, Ready, Terminated)
- 프로그램 카운터 (다음에 실행될 명령어의 주소를 저장하고 있는 레지스터)
- CPU 레지스터 정보 (CPU 레지스터는 CPU의 명령어 처리 및 데이터 연산에 필요한 고속의 작은 메모리 공간)
- CPU 스케줄링 정보 
- 메모리 관리 정보 
- 입출력 상태 정보 
- 계정 정보 

<img width="500" src="https://github.com/user-attachments/assets/85db5492-42a3-4d4a-a231-50af8fb90944"/>

</details>
<br>

<details>
<summary>프로세스 문맥에 대해 설명해주세요.</summary>

**프로세스의 현재 상태를 나타내는 정보**를 문맥(Context)라고 하는데, 이 정보는 모두 PCB(Process Control Block)라는 구조체에 저장되어 있다. 

결국 운영체제 입장에서 프로세스 문맥은 PCB를 의미한다고 볼 수 있다.

</details>
<br>

<details>
<summary>문맥교환(context switch)에 대해 설명해주세요.</summary>

시스템 콜 또는 인터럽트가 발생할 때 시스템이 **현재 실행 중인 프로세스의 상태를 저장하고, 다른 프로세스의 상태를 복원**하는 것을 문맥 교환(Context Switch)라고 한다. 

즉, 문맥 교환은 **CPU의 제어권을 한 프로세스에서 다른 프로세스로 넘겨주는 과정**을 의미한다. 

</details>
<br>

<details>
<summary>문맥 교환은 언제 발생하나요?</summary>

### 시스템 콜 또는 인터럽트가 발생한 경우

- 예를 들어, 현재 실행 중인 프로세스가 **시스템 콜**을 통해 파일 입출력, 네트워크 통신 같은 **I/O 작업을 요청**하면, 이는 CPU가 아닌 다른 디바이스 컨트롤러가 처리해야 하므로 현재 프로세스는 **Waiting 상태로 전환**된다. 
- I/O 작업이 수행되는 동안 CPU는 유휴 상태로 남아있지 않고, 다른 프로세스로 **문맥 교환**하여 해당 프로세스의 작업을 수행한다.
- **I/O 작업이 완료**되면 해당 장치는 CPU에게 **인터럽트**를 발생시켜서 I/O 작업이 완료되었음을 알린다. 
- Waiting 상태의 프로세스는 **Ready 상태로 전환**되고, **문맥 교환**을 통해 CPU를 다시 점유하면 원래 작업을 이어서 수행하게 된다.

### 시분할 시스템에서 타이머가 완료된 경우

- 시분할 시스템에서는 CPU 시간을 여러 프로세스에게 공평하게 분배하기 위해 일정한 시간 간격으로 문맥 교환이 발생한다. 
- 현재 실행 중인 프로세스의 실행 시간이 끝나면, 운영체제는 다른 프로세스에게 CPU를 할당하기 위해 문맥 교환을 수행한다. 

### 더 높은 우선순위의 프로세스가 Ready 큐에 도착한 경우

- SRTF 같은 선점형 스케줄링의 경우, 우선순위가 더 높은 프로세스가 Ready 큐에 도착하면 기존 프로세스를 선점하면서 문맥 교환이 발생한다. 

<img width="600" src="https://github.com/user-attachments/assets/3e9c8eda-dce6-4960-9e31-5e976cb84e8e"/>

</details>
<br>

<details>
<summary>문맥 교환 발생 과정에 대해서 조금 더 상세히 설명해주세요.</summary>

<img width="600" src="https://github.com/user-attachments/assets/9022403d-4bc3-4c8a-922f-397e4a567c71"/>

위의 그림처럼 인터럽트나 시스템 콜이 발생하면, 기존에 실행 중이던 프로세스 P0의 상태는 PCB0에 저장하고 

새로운 프로세스 P1의 상태를 PCB1으로부터 복원하여 CPU에서 실행시킨다. 

</details>
<br>

<details>
<summary>멀티 프로세스에 대해서 설명해주세요.</summary>

### 정의 

하나의 프로그램에서 동시에 여러 개의 프로세스를 실행하는 기술이다. 

하나의 부모 프로세스가 여러 개의 자식 프로세스를 생성하는 구조이며, 서로 독립적인 메모리 공간을 갖는다. 

### 특징  

- 장점 
  - 각 프로세스는 독립적인 메모리 공간을 가지므로, 한 프로세스에 오류가 발생해도 다른 프로세스에 영향을 미치지 않아서 안정성이 높다.
- 단점 
  - 메모리와 CPU 자원을 많이 소모한다.
  - 컨텍스트 스위칭 비용이 많이 든다. (다음 프로세스의 메모리 주소 검색, 프로세스 상태 저장 및 복원, CPU 캐시 메모리 비우는 작업 등) 
  - 프로세스 간 통신 (IPC, Inter Process Communication)을 위한 별도의 메커니즘으로 오버헤드가 발생할 수 있다.

### 예시 

- 분산 서버: 여러 클라이언트의 요청을 동시에 처리하기 위해, 하나의 컴퓨터에서 서버 프로세스를 여러 개 돌릴 수 있음. 
- 웹 브라우저: 구글 크롬 브라우저에서 여러 개의 탭을 띄웠을 때, 하나의 탭에서 오류가 발생해도 다른 탭에 영향을 미치지 않음. 

</details>
<br>

<details>
<summary>fork() 명령어에 대해 설명해주세요.</summary>

- 부모 프로세스의 메모리 공간을 복사하여 자식 프로세스를 생성하는 시스템 콜 
- 부모, 자식 프로세스는 각자 자신만의 고유한 PID를 가지며, 독립적으로 실행된다. 
- fork() 리턴값
  - 부모 프로세스: 자신 프로세스의 PID 반환 
  - 자식 프로세스: 0 반환
  - 오류 발생: -1 반환 

</details>
<br>

<details>
<summary>프로세스끼리 협력하는 방법에 대해서 설명해주세요.</summary>

- **공유 메모리 방식**
  - 생산자, 소비자 프로세스가 공유 메모리 공간을 만들어서 읽고 쓰는 방식 
  - 프로그래머가 공유 메모리의 접근 및 사용과 관련된 코드를 직접 작성해야 해서 번거로움. 
  - 실제 구현: POSIX Shared Memory 
- **메시지 전달 방식**
  - 한 프로세스가 다른 프로세스에게 전달할 메시지를 운영체제에게 시스템 콜로 요청하는 방식
  - send(message), receive(message) 같은 함수를 OS가 제공하므로 편리하게 사용할 수 있음. 
  - 실제 구현: 파이프, 소켓 
 
</details>
<br>

# 참고자료 

- https://velog.io/@jxlhe46/OS-Ch3-1-프로세스의-이해
- https://velog.io/@kangdev/기술면접운영체제-Process와-Thread의-메모리-공유-규칙
- https://velog.io/@dahyeon405/프로세스-메모리-레이아웃
- https://velog.io/@jxlhe46/OS-Ch3-3.-프로세스-간-통신
- https://inpa.tistory.com/entry/%F0%9F%91%A9%E2%80%8D%F0%9F%92%BB-multi-process-multi-thread