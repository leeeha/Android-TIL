<details>
<summary>운영체제는 무엇이고 어떤 역할을 수행하는지 설명해주세요.</summary>

운영체제는 **컴퓨터 시스템을 운영하는 소프트웨어**다.

- 하드웨어와 다른 모든 소프트웨어를 연결해주는 일종의 프로그램
- 하드웨어를 직접 다루는 부분은 운영체제가 대신 처리하고, **사용자에게는 편리한 인터페이스 환경 제공**
- 프로세스, 메모리, 파일 시스템 등 컴퓨터 **자원을 효율적으로 관리**하는 역할 
- 하드웨어 자원에 대한 접근 권한을 설정하여 **자원을 보호**하는 역할 

<img width="400" src="https://github.com/user-attachments/assets/3b6a0aa3-9c30-4354-afa1-97b7f3f12bcc"/> 

<img width="600" src="https://github.com/user-attachments/assets/0849fa9e-c572-4e5b-82cf-5c4ae86e7894"/>

</details>

<br>

<details>
<summary>시분할 시스템에 대해서 설명해주세요.</summary>

- 컴퓨터 처리 능력을 **일정한 시간 단위로 분할**하여, **여러 작업을 번갈아가며 실행**할 수 있게 만든 시스템
- 사용자 입장에서는 여러 프로그램이 **동시에 돌아가는 것처럼** 느껴진다. 

</details>

<br>

<details>
<summary>다중 프로그래밍 시스템(multi-programming system)에 대해서 설명해주세요.</summary>

- **여러 프로그램을 메모리에 동시에 올려서 CPU 사용 효율을 최대화 하는 시스템** 
- 한 프로세스가 CPU에서 실행되다가 I/O 이벤트가 발생하여 대기해야 하는 경우, 그동안 메모리에 적재된 다른 프로세스를 실행하는 방식 (컨텍스트 스위칭)

</details>

<br>

<details>
<summary>대화형 시스템(interactive system)에 대해서 설명해주세요.</summary>

사용자 요청에 대한 결과나 피드백을 즉시 얻을 수 있는 시스템

</details>

<br>

<details>
<summary>다중 처리기 시스템(multi-processor system)에 대해서 설명해주세요.</summary>

- CPU: 명령어를 실행하는 하드웨어
- Processor: 하나 이상의 CPU를 포함하는 물리적인 칩
- Multi-processor: 여러 개의 프로세서
- Core: CPU 안의 연산 단위
- Multi-core: 각 CPU가 여러 개의 코어를 가지는 것

멀티 프로세싱은 **다수의 프로세서가 협력적으로 여러 작업을 동시에 처리하는 것**이다.

이를 통해 **여러 작업을 병렬적으로 처리**할 수 있고, 한 프로세서가 고장나도 다른 프로세서에 영향을 미치지 않는다는 장점이 있다.

<img width="450" src="https://github.com/user-attachments/assets/8d88e83f-d531-4c37-8612-9fb09fe8e021"/>

CPU 자체를 여러 개 다는 것은 비용이 많이 들기 때문에, 아래 그림처럼 하나의 CPU 안에 여러 개의 코어를 다는 것을 **멀티 코어 설계**라고 한다. 

<img width="450" src="https://github.com/user-attachments/assets/ecebfc6e-eb25-45b8-8eaf-701d74d09363"/>

</details>

<br>

<details>
<summary>시스템 콜에 대해 설명해주세요.</summary>

**인터럽트의 일종**으로, **사용자 프로그램이 운영체제 커널에 있는 코드를 실행하려고 할 때 발생하는 신호**이다. 시스템 콜을 호출하면 CPU 제어권이 운영체제로 넘어가서, 운영체제 커널에 있는 코드를 실행할 수 있게 된다.

- 파일 관리: open, read, write, close 등 
- 프로세스 관리: fork, exit, wait 등 
- 메모리 관리: malloc, free 등 

</details>

<br>

<details>
<summary>커널에 대해 설명해주세요.</summary>

- 메모리에 항상 적재되어 있는 **운영체제의 핵심 부분**
- 프로세스 관리, 메모리 관리, 파일 시스템 관리, 보안 및 접근 제어 등

</details>

<br>

<details>
<summary>커널모드에 대해 설명해주세요.</summary>

- 운영체제의 핵심 기능이 실행되는 모드로, **모든 하드웨어 자원에 대한 접근이 허용**된다.
- 프로세스가 유저 모드에서 실행되다가 시스템 콜이나 인터럽트가 발생하면, 커널 모드로 전환하여 관련 작업을 수행한다. 

</details>

<br>

<details>
<summary>유저모드에 대해 설명해주세요.</summary>

- 사용자 프로그램이 실행되는 모드로, **제한된 권한으로 시스템 자원에 접근**할 수 있다. 
- 유저 모드에서는 커널에 정의된 작업을 직접 수행할 수 없으며, 시스템 콜을 통해 커널 모드로 전환해야 한다. 

</details>

<br>

<details>
<summary>폴링에 대해 설명해주세요.</summary>

- **CPU가 일정한 주기마다 특정 이벤트가 발생했는지 확인하는 방법** 
- 예시: CPU가 I/O 작업을 요청하고 나서 해당 작업이 완료되었는지 주기적으로 확인 
- 장점: 구현이 간단하다.
- 단점
  - 지속적으로 이벤트 발생 여부를 확인해야 하므로 리소스가 많이 소모된다.
  - 이벤트가 발생한 타이밍에 그 즉시 처리하기 어렵다. 

</details>

<br>

<details>
<summary>인터럽트에 대해 설명해주세요.</summary>

- 컴퓨터 작업 도중에 커널 관련 처리가 필요할 때, **CPU의 현재 작업을 중단시키고 즉시 특정한 처리를 요구하는 신호**
- 예시: I/O 장치가 입출력 작업을 완료한 경우, 프로그램 실행 중에 예외 발생, 프로세스가 직접 시스템 콜을 호출한 경우 
- 인터럽트가 발생하면 먼저 인터럽트 벡터에서 해당 인터럽트에 대한 **인터럽트 처리 루틴**을 찾아서 관련 작업을 수행한다.
- 인터럽트가 발생하기 전의 프로세스 상태를 저장해두고, 인터럽트가 처리되면 이전 상태를 다시 복구하여 원래 작업을 이어서 수행하는 것을 **컨텍스트 스위칭**이라고 한다.

</details>

<br>

<details>
<summary>DMA에 대해 설명해주세요.</summary>

DMA(Direct Memory Access)는 입출력 장치가 CPU에게 인터럽트를 걸지 않고, **직접적으로 메모리에 접근**하여 데이터를 전송하는 것을 의미한다. 이를 통해 데이터 전송의 효율성을 높이고, CPU의 부담을 줄일 수 있다.

예를 들어, 하드 디스크에서 메모리로 대량 데이터를 전송하거나, 오디오 및 비디오 스트리밍에서 실시간 데이터를 처리할 때 DMA를 사용하면 시스템 성능을 향상시킬 수 있다.

<img width="600" src="https://github.com/user-attachments/assets/52b20016-3b94-41ce-aa9a-1d1881d21ff3"/>

</details>

<br>

<details>
<summary>동기식 I/O에 대해 설명해주세요.</summary>

어떤 프로그램이 I/O 작업을 요청했을 때, 해당 작업이 완료되어야만 프로그램이 다음 작업을 수행할 수 있는 방식 (블로킹)

</details>

<br>

<details>
<summary>비동기식 I/O에 대해 설명해주세요.</summary>

어떤 프로그램이 I/O 작업을 요청했을 때, 해당 작업이 완료되지 않아도 바로 제어권을 프로그램에 넘겨줘서 다른 작업을 수행할 수 있는 방식 (논블로킹)

</details>