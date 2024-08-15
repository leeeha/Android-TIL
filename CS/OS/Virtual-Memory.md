# 가상 메모리 

기존에는 프로세스의 실행 코드 전체를 메모리에 로드해야 했고, 이로 인해 메모리 용량보다 큰 프로그램은 실행조차 할 수 없었다. 

하지만, 실제로는 프로그램 코드의 일부에서만 대부분의 시간을 사용하고 특정 순간에는 작은 양의 메모리 공간을 사용하기 때문에 이러한 방식은 매우 비효율적이었다. 

**가상 메모리는 이러한 물리적 메모리 크기의 한계를 극복하기 위해 나온 기술이다.** 프로세스를 실행할 때, **실행에 필요한 일부만 메모리에 로드하고 나머지는 디스크에 두는 것**이다. 

결과적으로 메모리의 작은 주소 공간만 있어도 프로세스를 충분히 실행시킬 수 있고, 이에 따라 더 많은 프로그램을 동시에 실행시킬 수 있다. 

# Demand Paging

Demand Paging은 **현재 필요한 페이지만 메모리에 올리는 것**이다. 이를 통해 CPU 사용률과 처리률이 높아지고, 더 많은 프로그램을 동시에 실행시킬 수 있다.

페이지 테이블에서는 **특정 페이지가 메모리에 있는지**를 나타내는 **valid-invalid bit**를 사용한다.

- valid bit: 페이지가 메모리에 있는 경우 
- invalid bit: 페이지가 메모리에 없는 경우 

페이지의 논리적 주소를 메모리 상의 물리적 주소로 변환할 때 invalid bit인 경우, **Page Fault** 에러가 발생한다.

<img width="600" src="https://github.com/user-attachments/assets/2a6d06b1-ce9b-493a-bd61-d05667c507b3"/>

1. 하드웨어가 TLB 캐시 확인 
2. TLB hit인 경우 곧바로 주소 변환, TLB miss인 경우 page table 확인 
3. page table에서 valid bit인 경우 주소 변환하여 TLB에 page 올린다. **invalid bit인 경우 Page Fault 발생!** 
4. Page Fault가 발생하면 **MMU가 운영체제에 Trap을 건다.** 커널 모드로 전환되어 page fault handler가 호출된다. 
5. 유효하지 않은 참조인 경우 프로세스 종료, 그렇지 않으면 빈 page frame을 얻는다. **빈 page frame이 없다면, 메모리에서 victim page 선택하여 대체한다.**
6. 운영체제는 **참조된 page를 디스크에서 메모리로 로드**하고, **disk I/O 작업이 완료될 때까지 대기**한다.
7. disk I/O 작업이 완료되면 **page table이 갱신**되며, 메모리에 페이지가 존재하므로 **valid bit**로 바뀐다. 
8. Ready Queue에 들어간 프로세스가 CPU를 점유하면 작업을 이어서 수행한다. 

# Page 교체 알고리즘 

## OPT (Optimal)

**가장 먼 미래에 참조되는 page를 대체하는 방법**이며, **항상 최적의 결과를 보장**한다. 

하지만, 미래에 어떤 페이지를 참조할지 모두 알고 있어야 하므로 실제로 사용하기는 어렵고, 다른 알고리즘 성능에 대한 상한선을 제공하는 역할을 한다.

<img width="600" src="https://github.com/user-attachments/assets/bd351b56-a575-49ab-b244-0396c7acaf25"/>

OPT 알고리즘은 항상 최적이므로, 위의 예시에서 6번보다 적게 페이지 폴트가 발생할 수는 없다. 

## FIFO (First In First Out)

**가장 먼저 참조된 페이지를 가장 먼저 교체하는 방법**이며, 미래에 어떤 페이지를 참조할지 몰라도 사용할 수 있다. 

- 장점: 모든 페이지가 평등하게 메모리 프레임에 거주하며, 구현하기 쉽다.
- 단점: 어떤 페이지는 항상 필요한 경우도 있는데, 이럴 때도 페이지를 교체한다.

일반적으로는 프레임 크기가 증가할수록 페이지 폴트가 감소해야 한다. 그런데, **프레임 크기가 커졌는데 오히려 페이지 폴트가 증가하는 경우**가 있는데, 이를 `Belady's anomaly` 현상이라고 한다.

<img width="400" src="https://github.com/user-attachments/assets/32eb0bd7-c5b2-4b6d-a2ea-50731a66b846"/>

<img width="600" src="https://github.com/user-attachments/assets/ccdc31ff-bd3d-4bbe-a864-28391d79597b"/>

## LRU (Least Recently Used)

**가장 오래 전에 참조된 페이지부터 교체하는 방법**이다. 

- 장점: Optimal에 근접한 방법이며, FIFO의 Belady's anomaly 현상이 발생하지 않는다.
- 단점: 구현하기 어려운 편이며, 페이지의 접근 빈도 수는 고려하지 않는다.

<img width="600" src="https://github.com/user-attachments/assets/a3fb8066-87a4-4e30-a13f-4e62f534753a"/>

**연결 리스트**를 사용해 가장 최근에 참조된 페이지를 head로 옮기는 방식으로 구현하면, 가장 오래 전에 참조된 페이지는 tail에 위치한다. 

따라서, LRU 페이지를 O(1)만에 찾아서 새 페이지로 교체할 수 있다.

## LFU (Least Frequently Used)

**참조 횟수가 가장 적은 페이지부터 교체하는 방법**이다. 페이지의 인기도는 반영하지만 최근성은 반영하지 못한다.

LRU처럼 연결 리스트로 LFU를 구현하면, 교체될 페이지를 찾는 데 O(N)이 걸린다. 

따라서, 균형 이진 트리 기반의 **최소 힙**을 이용하면, LFU 페이지를 O(1)만에 찾아서 O(logN)만에 새 페이지로 교체할 수 있다. 

# 참고자료 

https://rebro.kr/179

https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/main/OS