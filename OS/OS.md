# 4
프로세스는 실행중인 프로그램이다.

사용자들은 실제로는 CPU가 한개 뿐일지라도 동시에 여러개의 프로세스를 올리고 마치 동시에 실행중인 것처럼 사용할 수 있다.

프로세스의 구성요소
1. 메모리 -> 모든 instruction이나 접근하는 data는 메모리 상에 있다. (프로세스가 접근할 수 있는 메모리 영역을 address space라고 한다.)
2. 레지스터 -> ex) program counter(PC), stack pointer, frame pointer
3. I/O information

프로그램 -> 프로세스 ?

1. 우선 code나 static data가 disk에서 메모리에 올라와야 한다. 예전엔 프로그램 실행시 관련된 모든 내용이 다 올라온 뒤에 실행했지만 최근에는 실행중에 필요한 때에 필요한 부분을 올리는 방식으로 동작한다.
2. 지역변수, 함수 호출등에 사용할 (run-time) stack 영역을 할당한다.
3. 메모리 할당을 위해 heap 영역도 잡는다.
4. 여러 초기화 작업을 진행한다. 예를 들면 기본 I/O작업을 위해 file descriptior를 할당한다.
5. main() entry point부터 프로그램을 수행한다.

프로세스는 Running, Ready, Blocked 상태를 갖는다.
OS는 프로세스를 관리하기 위해 내부적으로 여러 프로세스에 관련된 정보들을 저장하는 자료구조를 유지하고 있다. 
이러한 자료구조는 초기에는 process list라는 구조를 사용하였다. 모든 OS들은 이와 비슷한 자료구조를 가지고 프로세스를 관리한다. 구조 내에는 각 프로세스에 대한 정보가 들어있는 데 이를 Process Control Block(PCB)라고 부른다.

# 5
UNIX에서의 PID(process identifier)는 프로세스를 이용하여 무엇인가를 다루고 싶을때 각 프로세스를 구분하기 위해 붙혀진 이름이다.

새로운 프로세스를 만들기 위해서는 fork()함수를 호출하는데 이 때 만들어지는 프로세스는 calling process와 거의 동일한 프로세스가 생성된다.
생성된 프로세스는 main부터 실행되는 것이 아닌 fork함수의 반환부분부터 실행된다.

calling process와 거의 동일하다는 의미는 새로 생성된 프로세스는 레지스터, address space, PC등을 자신에게 고유한 하나를 갖게 된다. 이때 모든 내용은 부모 프로세스의 값과 동일하게 복사된다.
그렇지만 하나 다른점은 fork의 반환값이 부모 프로세스는 생성한 자식프로세스의 pid가 되며 자식 프로세스는 0이 된다는 점이다.

single processor(CPU) 환경에서 여러 프로세스가 동시에 존재할 때 어떤 것을 실행할지에 대한 결정은 scheduler가 정하게 된다.

wait()함수를 이용하면 자식프로세스가 종료될때까지 부모프로세스의 실행을 중지시킬 수 있다.

exec() 함수를 이용하면 현재 수행중인 프로그램과는 다른 프로그램을 실행할 수 있다.
이 때 새로운 프로세스가 생성되는 것이 아닌 code segment를 원하는 프로그램 내용으로 덮어 쓰는 방식으로 기존의 것을 변경하는 형태가 된다.

fork, exec 의 조합으로 linux shell(ex bash, tsh)을 구현해 낼 수 있다.
또한 파일디스크립터의 응용으로 redirection을 표현해 낼 수도 있다.

# 6
어떻게 하면 동시에 여러 프로세스가 하나의 physical CPU를 공유할 수 있을까?

우선 프로세스가 CPU를 사용할때 가장 떠올리기 쉬운 단순한 방식은 한 프로그램이 실행되면 프로세스를 생성하고 그에 필요한 여러 리소스를 할당해준뒤 main함수를 수행하는 것이다. 이렇게 프로그램이 곧장 CPU위에서 바로 실행하도록 하는 방식을 Direct Execution Protocol 이라고 한다.

여기에 약간의 제약을 추가한것이 Limited Direct Execution 이다.

이 방식의 문제는 두가지 정도가 있는데 우선 첫번째로는 현재 수행되는 프로그램이 OS가 원하지 않는 행동을 할 수도 있다는 점, 두번째로는 OS자체도 CPU를 선점해야 작업을 진행할 수 있는데 이미 프로세스가 CPU를 잡고 있을때 OS가 이를 멈추고 다른 프로세스로 전환하는 등의 작업을 할 수 없다는 점이다.
장점은 물론 단순하고 다른 추가 작업이 없기 때문에 빠르다는 점이 있다.

첫번째 문제를 해결하기 위해 새로운 processor mode를 도입하자.
user mode에선 I/O 요청, 예외 발생등의 작업을 수행할 수 없게 하고
그에 반해 kernel mode에선 이를 진행할 수 있게 하자. 

user mode에서 위와 같은 작업을 수행하고 싶다면 system call을 호출하여 적당한 제약을 주어 기능을 요청할수 있도록 한다.

system call -> trap -> (user mode to kernel mode) -> trap handler -> return from trap -> (kernel mode to user mode)

trap 발생시 어떤 trap handler code를 수행해야 하는지를 알기 위해 trap table을 유지한다.
trap table에는 trap handler의 주소가 들어가 있는데 이 주소는 컴퓨터의 부팅 시간에 값이 채워지게 된다.

이제 두번째 문제였던 OS가 현재 수행중인 프로세스가 점유하는 CPU의 소유권을 어떻게 다시 가져올지에 대해 고민해보자.

우선 첫번째 방법으로는 프로세스를 작성한 사람이 주기적으로 CPU를 놓아주는(yield) 방식으로 구현하기를 바라는(?) 것이 있다. 실제로 이런 일은 없고 대부분의 프로세스는 I/O 작업과 같이 system call을 이용한 작업을 하는데 이 때 권한을 잠시 놓아줌으로써 OS가 CPU를 가져갈 수 있게 된다.

그렇지만 이 방식은 그러한 system call 없이 무한 루프를 도는 등의 프로그램에서는 제대로 동작하지 않는다.

이것을 해결하기 위한 간단한 방법은 강제로 주기적인 system call을 호출하게 만드는 것이다
timer interrupt가 사용될 수 있는데 이는 부팅 시간에 설정되어 매번 시간마다 interrupt를 발생하여 OS가 CPU를 소유할 수 있게 도와준다.

현재 실행중인 프로세스의 레지스터를 저장하고 다음번에 실행하게 될 프로세스의 레지스터 값을 불러들이는 과정을 Context switch라고 한다.

# 7

Scheduling 
우선 약간의 가정을 하고 스케줄링 기법을 만들어본다.

1. 모든 작업은 동일한 시간을 사용한다.
2. 모든 작업은 동일한 시간에 도착
3. 한번 시작하면 끝날때 까지 수행
4. I/O작업같은 것 없이 오직 CPU만 사용하는 작업을 함
5. 각 작업의 run-time이 미리 알려져 있다.

또 스케줄링이 제대로 되는지 확인하는 기준으로 turnaround time을 사용한다.
turnaround time은 작업이 종료되는 시간 - 처음 시스템에 도달한 시간으로 계산한다.
근데 위의 가정에 의해 모든 작업은 동일한 시간에 도달하므로 이를 0으로 잡고
turnaround time = 작업의 종료시간으로 확인해보자.


## First In, First Out(FIFO)
First Come, First Served(FCFS)라고 불리기도 한다.
말 그대로 먼저 온 작업을 먼저 수행한다. -> 구현이 쉽고 간단하다

![](7.1.jpg)

만약 A, B, C가 순서대로 왔다면 위 그림과 같다. turnaround time = (10+20+30)/3 = 20으로 
괜찮아 보인다.

이제 1번 제약조건을 풀어보자, 즉 각 작업이 제각기 다른 수행시간을 쓴다고 보자.
그럼 다음과 같은 경우가 생긴다

![](7.2.jpg)

위와 같이 짧은 수행시간을 갖는 작업들이 앞에 긴 작업에 의해 수행이 늦추어 지는 현상을 Convoy effect라고 한다.

## Shortest Job First (SJF)
이런 문제를 해결하고자 새로운 스케줄링 기법을 사용해보자. 말그대로 가장 짧은 시간 수행되는 작업을 먼저 하도록 하는 것이다. 위와 같은 작업들이 들어왔을때 SJF는 다음과 같이 스케줄링한다.

![](7.3.jpg)

만약 모든 작업이 동시에 들어왔다면 가장 짧게 수행되는 작업을 먼저하는 SJF가 optimal이라는 것이 증명될수 있다고 한다.

이제 두번째 가정을 제거해보자. 모든 작업이 동시에 도달하는것이 아니라고 한다면
또다시 가장 긴 시간 수행되는 작업이 제일 먼저 옴으로써 FCFS에서의 문제점이 다시 생긴다.

이 문제를 해결하기 위해선 3번째 가정을 제거해야 한다. 작업이 한번 수행되면 끝날때까지 수행되는 것이 아닌 중간에 멈출수있도록 해야한다. 즉 스케줄러가 preemtive해야 한다.

## Shortest Time to Completion First (STCF)
SJF는 non-preemtive를 가정하였는데 여기에 preemtion을 넣어 STCF 혹은 PSJF(Preemtive Shortest job first) 라고 하자.

이 스케줄링 방식에선 새로운 job이 들어오면 현재 남아있는 job들의 완료까지 남은 시간을 비교하여 이 값이 가장 작은 작업을 우선적으로 수행하게 한다.


이제 성능 측정에 대해 새로운 기준을 추가해보자. 지금까지는 turnaround time만을 성능의 기준으로 삼았는데 최근 time sharing을 통해 interactive한 프로그램이 많이 생기게 되어 response time이 중요해졌다. 이 시간은 처음으로 스케줄링 된 시간 - 작업이 처음 시스템에 도착한 시간으로 계산된다.
STCF에선 긴 작업은 수행되기 위해서 항상 짧은 작업들이 모두 수행이 완료될때까지 시작조차 할 수 없게되어 response time에서는 뛰어난 모습을 보여주지 못한다.

## Round Robin(RR)
RR은 작업을 완료시까지 수행하는 것이 아닌 time slice만큼씩만 수행하고 순서를 넘겨준 뒤 큐에 들어가는 방식으로 동작한다. 그래서 time slicing이라고도 함
여기선 이 time slice의 길이가 매우 중요한데 이게 짧아질수록 response time은 줄어들지만
너무 줄어들게 되면 작업간의 전환에서 필요한 context switch에서 오는 cost가 전체적인 성능을 떨어뜨릴수 있다. 이러한 trade-off를 잘 조절해야 한다.

![](7.4.jpg)

RR의 문제는 처음에 언급했던 turnaround time에 그렇게 좋지 못하다는 점이다.
그런데 이렇게 여러 작업들에 대해 fair한 방식으로 동작하는 스케줄링 기법은 당연히 그럴 수  밖에 없다. 왜냐면 각 작업들을 최대한 질질 끌며 time slice만큼씩 나눠주니깐 -> 이것은 trade-off의 결과야


I/O를 사용하지 않는다는 4번 제약을 제거해보자.
실행중인 job은 I/O를 수행하는 도중에는 CPU를 사용하지 않을꺼야 그럼 OS는 다른 친구에게 CPU를 넘겨줄지 말지를 정하기도 해야하고 해당 I/O가 끝났을때 권한을 원래 친구에게 주어야 할지 말지도 정해야 해

만약 cpu작업과 i/o작업을 번갈아 하는 job A와 오직 cpu만 사용하는 job B가 왔을때를 살펴보자.
둘의 전체 수행시간은 같고 A가 먼저 도달했다면 STCF를 사용할때 다음과 같이 동작해

![](7.5.jpg)

A전체를 하나의 job으로 본다면 당연히 A가 먼저왔으니 먼저 끝나므로 A를 끝까지 실행할꺼야
그러면 I/O작업을 하는 동안은 cpu가 idle한 상태로 남게된다

![](7.6.jpg)

그렇지만 이렇게 각각의 cpu burst씩만을 하나의 job으로 보게 된다면 A가 I/O를 하게 되면
남아있는 CPU burst는 B밖에 없기 때문에 idle한 시간을 없앨수있게 된다. (i/o와 overlap되어)

# 10
## Multiprocessor Scheduling

평범한 프로그램에게 CPU의 개수가 늘어나는것은 의미가 없음 -> 오직 하나의 CPU만을 사용하기 때문

그렇기에 여러개의 CPU가 있는 환경을 잘 활용하려면 프로그램을 parallel하게 동작하도록 다시 작성해야함 (thread 같은 것을 사용함으로써)
multithreaded program은 work를 여러개의 CPU에 잘 분배하여 더 빠르게 프로그램을 수행할 수 있음

지금까지의 스케줄링 기법은 single CPU기반에서 였는데 그렇다면 multiprocesor에서의 스케줄링은 어떨까?

single CPU기반에서는 프로세서가 프로그램을 더 빠르게 수행하도록 도와주는 hardware cache가 존재한다. cache는 자주 사용되는 데이터를 저장해놓은 뒤 메모리에서보다 더 빠르게 접근하도록 도와줌으로써 속도를 증가시킨다.

![](10.1.jpg)
(single processor에서의 구조)

프로그램이 처음 데이터를 사용시 캐시에 존재하지 않기때문에 메모리에서 그 값을 가져온다. 그 후엔 그 값이 다시 참조될것을 기대하며 캐시에 저장해두게 된다.
이는 locality를 이용한 것인데 이는 2가지 종류가 있다.
1. temporal locality -> 한번 접근한 것은 재접근할 가능성이 크다. (ex loop)
2. spatial locality -> 주소 x에 접근했다면 그 주변의 x-1, x+1등에도 접근할 확률이 크다.(ex 배열 접근)


![](10.2.jpg)

위와 같은 multiprocessor 환경을 보자. (여러개의 cpu가 하나의 memory를 공유한다)
만약 CPU1에서 돌던 프로그램이 캐시에 값을 넣고 사용하던 중 그 값을 update한다고 해보자.
이때 캐시는 곧바로 메모리에 변경된 값을 업데이트하면 느려지니 캐시에서만 업데이트를 반영하고 나중에 반영하기로 했다고 해보자.

그 후 그 프로그램이 CPU1이 아닌 CPU2에서 돌기로 스케줄링이 된다면 해당 데이터에 접근할때 캐시에 존재하지 않게 되고 이를 메모리에서 다시 가져오게 되는데 이 값 또한 업데이트 되지 않은 값이 된다.

이러한 문제를 cache coherence라고 한다.

여러 cpu에 존재하는 cache의 값이 서로 다른 문제를 해결하는 간단한 방법은 bus snooping이다. 즉 연결된 bus를 잘 살피다 자신의 cache에 있는 값과 동일한 값이 update되면 내가 가진 값을 invalidate 시키거나 그 값으로 update시키는 방식으로 동작한다.

하드웨어가 캐시에서의 coherence를 해결해준다고 가정한다면 프로그램상에서는 아무 신경을 쓰지 않아도 될까? -> no

여러 CPU가 shared data에 접근한다면 여러 문제가 생길수 있다. (critical section)

Cache Affinity
프로그램이 한 CPU에서 수행되면 그 과정에서 캐시에 많은 정보를 넣어두게 되는데(ex TLB) 이 정보는 매우 유용함. 다음번에 만약 이 CPU에서 다시 돌게 된다면 이 정보들을 처음부터 다시 채우게 되는 수고를 덜 수 있음

그래서 만약 한 프로그램을 어느 CPU에서 돌릴지에 대한 선택을 해야한다면 가능한 기존에 돌렸던 CPU에 돌리는것이 더 이득이라는 사실을 알 수 있다.

SQMS , MQMS 등이 있음. (궁금하면 나중에 찾아보셈)



# 14
초기 메모리는 사용자에게 많은 abstraction을 제공하지 않았음

메모리에는 OS가 한 자리를 차지하고 나머지 공간을 현재 실행중인 프로그램이 사용하였다.

![](13.1.jpg)

multiprogramming 이 시작되면서 같은 시간에 여러개의 프로세스가 실행하는 것이 필요해지면서
OS는 여러 프로세스를 switch하며 CPU의 효율적인 사용을 추구하였다. 또한 더 나아가 interactivity가 중요해지면서 time sharing 방식이 등장하여 동시에 CPU를 사용할 수 있도록 하였다.

이 때 여러 프로세스들이 메모리를 사용하는 방식은 우선 위 그림처럼 한 순간에 실행중인 프로세스가
전체 영역을 차지하고(OS영역 제외) 다른 프로세스의 순서가 되면 상태를 저장한 뒤에 다음 프로세스가
전체 영역을 다시 차지하는 방법이 있다.

-> 이 경우 너무 느리다 -> 레지스터, cpu의 전환은 빠르지만 memory to disk 복사는 느리다.

그래서 고안한 방식은 각 프로세스를 그냥 메모리에 두고 각자의 사용영역을 제한하는 방법이다.

![](13.2.jpg)

이렇게 동시에 여러개의 프로세스의 영역이 메모리에 존재하면 보안이 또 다른 이슈가 된다.

따라서 새로운 메모리에 대한 abstraction을 만드는데 그것이 바로 Address space 이다.
이는 각 프로세스가 바라보는 메모리이다. address space에는 code, stack, heap 영역 등으로 구성되어 있다.

![](13.3.jpg)
OS가 프로세스에게 주는 메모리에 대한 추상화는 위 그림에서 처럼 프로세스는 메모리가 0~16KB까지 연속적이고 간단한 주소에 존재한다고 생각하게 해준다. 그렇지만 실제로는 OS가 해당 논리적인 주소를
실제 physical memory address로 적절히 mapping 시켜준다.
따라서 프로세스가 사용하는 주소는 virtual address라고 한다.

우리가 printf로 포인터들의 주소를 출력하여 보더라도 이것은 모두 virtual address이다.
프로그래머는 프로그래밍 단계에서 실제 physical address를 볼 수 없다.

## 15
하드웨어가 프로세스가 사용하는 모든 memory access에 대해 virtual address를 physical address로 변환시켜주는 작업을 한다.
이를 통해 모든 프로세스는 자신이 virtual address를 사용한다는 것을 인지하지 못한 채 원하는데로
프로그램을 진행시킬 수 있게 된다.


주어진 virtual address를 어떻게 physical address에 mapping시켜 줄까?

#### Dynamic(Hardware-base) Relocation
base, bound 라는 cpu register를 따로 두고 이를 이용하여 주소 변환과정을 진행한다.
매번 instruction을 진행하다 메모리 접근이 필요하게 되면 (심지어 instruction에 접근하는 것 조차 code 영역에 접근해야 한다.) 해당 주소를 접근하기 전 hardware가 관여하여 주소를 변환해준다.
``` physical address = virtual address + base ```

address의 relocation이 runtime에 일어나기 때문에 dynamic relocation이라고 한다.

bound 레지스터는 security를 위해 존재한다. -> 메모리 주소 참조가 bound를 넘어가게 되면 에러를 발생시켜준다.

이러한 base, bound 레지스터는 chip에 붙어있는 하드웨어 인데 이렇게 processor내에 address traslation을 도와주는 부분을 ==memory-management unit(MMU)==라고 한다.

이러한 방식의 단점 -> 만약 stack, heap의 크기가 크지 않다면
base ~ bound 사이의 대부분의 영역이 사용되지 않은 채 남게 된다.
이러한 낭비를 internal fragmentation이라고 한다. -> allocated된 영역의 내부가 낭비되는 문제


