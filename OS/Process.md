# 폰 노이만 아키텍처 

**프로그램**은 컴퓨터 하드웨어에게 어떤 작업을 수행하도록 시키는 **명령어들의 집합**이다. 이를 메모리에 올려서 CPU에 패치시키고 실행하면, **실행 중인 프로그램**인 **프로세스**가 된다.

이처럼 명령어 집합인 프로그램을 메모리로부터 CPU로 패치(patch)시키고 그것을 실행(execute)한 다음, 그 결과를 다시 메모리에 저장하는 구조를 **폰 노이만 아키텍처**라고 부른다.

폰 노이만은 메모리에 프로그램을 저장하는 **내장형 프로그램** 방식을 처음으로 도입한 사람이다. 

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
- `Waiting` : 입출력 연산이 완료될 때까지 Waiting Queue에서 대기하고 있는 상태 
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
- **입출력 또는 특정 이벤트가 발생할 때까지 대기**하고 있는 프로세스들은 `Waiting Queue`에 들어간다.
- 이러한 큐들은 일반적으로 **PCB의 연결 리스트**로 구현된다.

<img width="700" src="https://github.com/user-attachments/assets/ca3269bd-6f9e-485c-9a19-fb9485de83f5"/>

<img width="700" src="https://github.com/user-attachments/assets/d2b8d743-9d75-41b2-b6bd-8f9c62e5ea50"/>

## 문맥 교환 

**프로세스가 사용되고 있는 상태**를 **문맥**이라고 하는데, 이 정보는 다 PCB에 저장되어 있다. 따라서, **운영체제 입장에서 문맥, 컨텍스트는 결국 PCB를 의미한다**고 볼 수 있다. 

인터럽트 또는 시스템 콜이 발생할 때 시스템은 **현재 실행 중인 프로세스의 컨텍스트를 저장하고, 나중에 그 컨텍스트를 복원**한다. 결국 문맥 교환이라는 것은, **CPU 코어를 다른 프로세스와 교환**하는 것이며, **현재 프로세스의 상태를 저장하고 다른 프로세스의 상태를 복원하는 방식**으로 진행된다. 

<img width="700" src="https://github.com/user-attachments/assets/9022403d-4bc3-4c8a-922f-397e4a567c71"/>

위의 그림처럼 프로세스 P0를 실행하다가 인터럽트나 시스템 콜이 발생하면 PCB0에 프로세스 P0의 상태를 저장하고

PCB1으로부터 프로세스 P1의 상태를 복원하여 실행시키는 것을 프로세스의 문맥 교환이라고 한다.

## 프로세스의 생성과 종료 

### 프로세스의 생성 

- 부모 프로세스는 자식 프로세스를 생성할 수 있다. 
- 부모 프로세스는 자식 프로세스와 함께 동시에 실행되거나, 자식 프로세스가 종료될 때까지 대기한다. 
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

# 참고자료 

- https://velog.io/@jxlhe46/OS-Ch3-1-프로세스의-이해
- https://velog.io/@kangdev/기술면접운영체제-Process와-Thread의-메모리-공유-규칙
- https://velog.io/@dahyeon405/프로세스-메모리-레이아웃