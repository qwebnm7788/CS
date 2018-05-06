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

# 16
실제 메모리는 위에서 살펴본것처럼 16kb짜리가 아니라 훨씬 크다. 그래서 internal fragmentation은 문제가 된다
![](16.1.jpg)

좀 더 나은방법? -> 한쌍의 base, bound를 두지 말고 address space의 logical segment당 한 쌍의 base, bound를 두어 segment별로 관리해보자.

위 그림과 같은 virtual address space를 가진 프로그램이 있을때 code, stack, heap 영역을 각각 하나의 segment로 보고 그에 대한 base, bound register를 유지하자.
이러면 physical memory에 mapping 시킬때 각 segment를 독립적으로 위치시킬 수 있게 된다.

![](16.2.jpg)
이렇게 되면 사용하고 있는 공간만을 차지하게 된다. 이렇게 구현되기 위해서는 기존에 한쌍만 존재했던 base, bound register를 segment의 갯수만큼의 쌍으로 관리해야 한다.

잘못된 주소 접근 ? -> segmentation fault

virtural address는 접근전에 physical address로 translation되어야 하는데 어떻게 될까?
해당 주소가 특정 segment에 있는지 알아낸 다음 그곳으로 가서 그것의 base에 offset을 더하면 됨

그렇다면 reference가 발생할때 그게 어느 segment에 속하는지 offset은 몇인지 어떻게 알까?
두가지 접근법이 있음
1. implicit approach -> 해당 reference가 어디서 발생했는지를 보고 어떤 segment일지 결정하는거야 PC에서 이루어졌다면 code segment겠지? 등등
2. explicit approach -> 지금껏 해온것과 비슷하게 진행, 주소의 일부분을 어떤 segment인지를 표시하는 부분과 offset 부분으로 나누어서 봄
예를 들면 위의 그림은 16kb짜리 address space를 쓰므로 2^14 -> 14bit address를 쓴다.
segment는 3개가 있으니깐 3을 표현하려면 최소 2bit을 써야함 따라서 상위 2bit으로 segment를 구분하는데 쓰고 나머지를 offset으로 사용함
그래서 매번 reference 발생시 상위 바이트로 해당되는 segment의 base, bound register를 찾고
그 값으로 physical address 계산을 수행하게 된다.


근데 자세히 보면 우리는 더하기만 했는데 스택은 거꾸로 커지니깐 주소를 빼줘야함
해결하려면 하드웨어의 힘을 좀 더 빌려서 어느 방향으로 증가하는지에 대한 정보도 필요해 (1bit이면 될듯 positive인지 negative인지)

예를 들어보면 위 두 그림에서 만약 virtual address 15KB에 접근한다고 해보자.
이러면 virtual 상에선 16KB기준으로 -1KB만큼 간거임 이걸 어떻게 계산할까? 제대로 한다면 physical에서 stack이 28KB에서 시작한다면 27KB로 mapping 되어야 함

15KB = 11 1100 0000 0000 (0x3C00)임 첫 2bit 11로 segment를 정함
1100 0000 0000 이 offset임 -> 3KB임

이땐 해당 segment의 최대 크기가 필요한데 4KB이니깐 -> ??
3KB - 4KB = -1KB가 됨 이것을 base 인 28KB에 더하면 27KB로 잘 mapping된다.

즉 offset - maxsize + base로 계산함


하드웨어의 힘을 또 빌려서 이번엔 메모리를 절약해보자.

생각해보면 여러 친구들이 동일한 코드를 공유할수도 있음, 즉 여러 address space가 동일한 memory segment를 참조할 수 있음 이 때 이러한 부분을 공유하는게 어떨까? 대표적으로 code segment

이걸위해선 각 segment에 대해 protection bit이 필요해 그래서 해당 segment가 readable 한지
writable, executable한지를 표시해주자. 만약 해당 segment가 read permission이 있다면 여러 address space에서 그 메모리 영역을 공유하여 사용할 수 있어 -> 물론 각 프로그램들은 자신의 virtual address만 보기때문에 다 자기껀줄 알겠지

그렇지만 이제 address translation할 때 bound만 보는게 아니고 protection bit도 봐서
해당 프로그램이 참조하는 segment에 대한 해당 permission이 있는지도 확인해 줘야해


3개의 segment뿐 아닌 더 여러개의 segment로 쪼갤수도 있음 -> fine-grained vs coarse-grained segmentation

물론 segmentation이 기존의 문제였던 unused되는 공간을 없앤것은 맞음
근데 문제가 있음
1. context switch때 어떻게 할꺼임? -> segment마다 존재하는 레지스터들이 모두 다 저장되고 복원되야 된다.
2. 이제 segment별로 크기가 다 달라서 기존과는 다르게 physical memory에 free space를 찾는 방법이 달라져야 함. 또 이렇게 크기가 다르면 physical memory에 작은 구멍들이 많이 생김 -> 그 만큼 작은 segment가 오지 않는이상 이부분은 버려지게 됨 -> 이런걸 external fragmentation이라 함

이것에 대한 하나의 솔루션은 compaction임
즉 잠시 실행되는거 다 멈추고 메모리를 allocated된 부분에 대해서 한쪽으로 다 밀어버리는거야 (그에 맞게 register도 물론 수정하고) -> 근데 이건 너무 느려 (메모리 작업이잖어) 비효율!

좀 더 간단한 해결책은 free-list management 알고리즘을 잘 떠올려보는거야
best-fit, worst-fit, first-fit, buddy-algorithm 등등 많음

물론 이런 알고리즘이 external fragmentation을 줄여줄 순 있겠지만 어찌됬던 항상 존재할수밖에 없음 -> 최소화하려고 노력하자.

또 문제는 여전히 sparse-address space를 잘 해결하지 못할 수 있음
예를 들면 되게 드문드문 사용되는 heap의 경우 segment의 전체 크기는 매우 크지만
사용안되는 부분은 많을꺼야 .. ?

# 17
free-space management에 대해 알아보자. 만약 다루어야 할 space가 fixed size라면 매우 쉬울꺼야 그냥 특정 크기의 덩어리의 리스트를 잘 가지고 있다가 필요하면 하나씩 주면 되니깐

근데 segmentation에서 봤듯이 variable size 덩어리를 줘야하면 좀 어려워져 (malloc으로 사용자가 요청하는 경우 등등도 마찬가지) -> 이 땐 free space가 들쑥날쑥 차지하게 되어 external fragmentation이 생긴다.

![](17.1.jpg)


위와 같은 free space를 list의 형태로 관리한다고 생각해보자.

![](17.2.jpg)

10byte보다 큰 메모리 할당은 실패할것이고 10byte를 요구하면 저 덩어리 하나를 주면됨
그보다 작은것은? -> splitting으로 잘라서 주고 남은것은 그대로 가지면 된다.

예를 들어 1byte를 요구하면 다음과 같아진다
![](17.3.jpg)

만약에 1byte를 요구하는 것이 아닌 10byte를 free해서 돌려줬다고 생각해보자.
단순히 반환만 한다면 다음과 같아진다
![](17.4.jpg)

근데 이렇게 되면 20byte짜리는 절대 할당해줄 수 없음
그래서 이 문제를 해결하기 위해 매번 free마다 각 free-space를 살펴보다 인접한 메모리 영역에 있는 것들은 하나로 합쳐주기로 함 -> coalescing 이라고 함

![](17.5.jpg)

free함수로 할당된 메모리를 반환할때는 크기정보를 알려주지 않음 -> 자체적으로 알아내야함
그래서 메모리를 할당해줄때 헤더를 추가로 할당해서 여러 추가 정보를 넣어두게 된다.

![](17.6.jpg)
이를 통해 사용자가 N bytes를 요구한다고 해서 라이브러리가 딱 N바이트를 찾는 것이 아니라
N + header size만큼을 찾는 다고 생각할 수 있다.

실제로 어떻게 동작하는지는 책의 예제를 확인!

allocator가 빠르고 fragmenation을 최소화 하려면 어떤 전략을 써야할까?
여기에 정답은 없음

## Best fit
free list를 한번 쭉 보고 요청된 크기보다 큰 것 중 가장 작은 것을 할당해주는 방식
-> wasted space를 최소화 할 수 있음, 그치만 전체를 exhaustive search해야 한다는 단점, 또 자잘한 구멍들이 많이 생긴다는 단점

## Worst fit
best fit의 반대로 요청된 크기보다 큰 것중 가장 큰것을 찾고 그것을 잘라서 주는 방식
여전히 리스트를 전부 봐야한다는 단점이 있음, 최대한 덩어리의 크기를 크게 남기려고 노력함 (가장 큰 것을 찾아서 잘라내면 남게되는 것도 best fit에 비해 크겠지)

## Segregated List
다른 방법중에는 segregated list란 것이 있는데 free-space를 유지하는 리스트를 단 하나만 가지는 게 아니라 자주 사용되는 크기로 구성된 (fixed size) 리스트와 general 크기의 리스트로 나눠서 유지함. -> 이러면 자주 사용되는 것을 사용시 좀 더 효율적이게 됨.

## Buddy Allocation
2의 지수승으로 나누어 관리 -> coalescing이 간단해짐


# 18
## Paging
variable-size로 나누면  fragmentation이 심함
이제 fixed-size로 나눠보자. 이렇게 나눈 fixed-size unit을 page라고 부른다.
physical memory는 이제 page frame이라 불리는 fixed-size slot의 배열로 볼 수 있게 된다.

paging을 사용하면 address space를 굉장히 flexible하게 사용할 수 있음 -> 예를 들면 segmentation에선 heap, stack처럼 늘어나는 방향을 고려해줬어야 하는데 이제 항상 크기가 고정이기 때문에 방향에 대해선 고민할 필요가 없다. -> 그냥 원하는 곳에 새로운 page를 주면 되니깐
그리고 free-list를 다루는 것이 매우 simple해진다. -> 그냥 page하나 준다.

예를 들어보자
64bit address space를 가진 프로세스에 대해 128 bit의 physical memory가 존재한다고 하자. 또한 page의 크기는 16bit이라고 하자.

![](18.1.jpg)  
(virtual address space)

![](18.2.jpg)
(physical address space)

이렇게 되면 연속된 page라도 physical memory에선 연속되지 않은 여러군데의 page frame에 들어갈 수 있게 된다. -> 이러한 mapping정보를 담을 page table이 필요함

page table은 per-process로 저장됨 -> 매 프로세스별로 다른 mapping정보를 갖는다.

virtual address 해석법 -> 상위/하위 비트를 나누어 virtual page number(VPN)과 offset으로 구분함
VPN을 page table의 index로 사용해서 physical frame number(PFN, PPN)을 얻은 뒤
그것을 offset과 하나로 합쳐서 실제 physical address를 얻게 된다.

![](18.4.jpg)

page table은 어디 저장될까?
page table은 엄청 커질수 있음, 예를들어 32bit 주소공간에 4KB 크기의 page를 사용한다면
2^32 / 2^12 = 2^20 개의 page가 존재하게 된다. (4KB=2^12Byte) 그러면 VPN은 20bit을 차지하게 되고 page table에느 2^20가지의 translation이 존재하게 된다. table의 각 entry에 들어가는 값들이 4byte라고 한다면 (page table entry(PTE)당 4byte) -> 4 * 2^20 byte = 4MB가 된다.

크기가 커질수록 이 값은 커지게 된다. 그래서 page talbe은 MMU와 같은 하드웨어에 올라가 있지 않고 physical memory에 존재하게 된다.

page table entry에는 어떤 내용들이 있을까?
valid bit -> 특정 translation이 valid한 지 여부를 알려줌 (사용되지 않는 부분에 대한 참조는 invalid로 표시해두어 trap으로 처리하게 만듬)

protection bit -> 특정 page가 readable, writable, execuable한지 여부
present bit -> 해당 page가 physical memory에 있는지 아니면 disk에 있는지 (swap out 되었는지)
dirty bit -> 메모리에 올라온뒤로 수정되었는지
reference bit (access bit) -> page가 접근되었는지 -> 나중에 page replacement에 사용됨

기타 등등

하드웨어가 주소를 translate하려면 page table이 어디있는지 알아야함 -> page-table base register를 각 프로세스별로 유지하며 해당 page-table의 physical address를 저장해둔다.

paging은 매번 메모리 접근시 해당 메모리의 physical address를 얻기 위해 page table에 접근하는 추가적인 메모리 접근이 발생한다 (page-table 자체도 메모리에 올라와있으므로)
=> paging은 느리고 메모리를 많이 사용할 수 있는 단점이 있다.

주소 translation은 다음 그림과 같은 방식으로 진행되게 된다.
![](18.5.jpg)

# 19
## Faster Translation(TLBs)

paging은 page-table을 위한 추가적인 메모리와 매번 translation을 위한 memory lookup이 수반되기 때문에 성능상의 단점이 있을 수 있다. 이를 개선? -> 하드웨어의 도움을 받자

translation-lookaside buffer (TLB)를 사용하자. TLB는 MMU(memory-management unit)의 일부로써 자주 사용되는 virtual to physical address translation을 저장해두는 hardware cache라고 할 수 있다. 그래서 매번 virtual address translation이 필요하면 먼저 TLB에 가서 이미 존재하는지를 확인하는 과정을 거친다. (있다면 page table에 갈 필요없이, 즉 추가적인 메모리 접근 없이 translation을 진행할 수 있다.)

![](19.1.jpg)
TLB를 이용한 translation은 위 코드와 같이 진행됨
1. VPN을 이용해서 TLB에 이미 있는지 확인
2. 있다면 (TLB hit) TLB의 해당 entry에 가서 PFN을 얻고 offset과 붙여서 physical address를 얻을 수 있음 이걸로 곧바로 메모리에 접근하면 된다.
3. 없다면 (TLB miss) 원래 하던대로 page-table에 접근해서 그에 맞는 주소를 얻음 그 후에 얻은 값을 TLB에 집어넣음 그러고서 다시 현재 수행한 명령을 재수행한다. -> 그렇게 되면 다음번엔 TLB hit이 되어 1번의 과정을 거치게 된다.

일반적인 cache처럼 대부분의 경우 TLB hit이 난다는 전제하에 사용함 -> 최대한 TLB miss를 줄이자.


다음과 같은 array의 원소에 접근한다고 하자.
![](19.2.jpg)

각 VPN에 처음 접근시에는 TLB miss가 나지만 이후 연속된 2~3개의 접근에 대해서는 TLB hit이 된다. 이것은 TLB가 spatial locality를 이용해서 그런것임(즉 space가 서로 인접한 것을 이용)
또한 만약 0~ 9까지 접근 후 또다시 0~9를 접근한다면 (만약 TLB가 커서 기존꺼를 다 담고있다면) 모든 접근에 대해 TLB hit이 되게 된다. 이는 temporal locality를 이용한 것 (짧은 시간에 여러번 동일한 곳에 접근함)

* temporal locality -> 한번 접근한 곳은 조만간 또 접근할꺼야
* spatial locality -> x에 접근했다면 x 주변에도 접근할꺼야


TLB miss처리?

1. 하드웨어가 ! -> 예전에는 많이 했음
2. 소프트웨어가 ! -> TLB miss가 생기면 하드웨어는 exception을 발생시킴 그 후 kernel mode로 변환한 뒤 trap handler로 점프하게 된다. trap handler는 TLB miss를 처리하기 위한 OS의 코드 일부임 그래서 처리하고 다시 원래의 하드웨어로 돌아오게 된다. 
여기서의 차이점은 return from trap에서 기존에는 trap을 발생시킨 명령의 다음 명령부터 수행하였는데 TLB miss에서는 해당 명령을 재수행해야함 -> PC를 적절히 잘 저장해줘야해
그리고 trap handler도 OS의 일부니깐 메모리에 있을텐데 이를 접근하기 위해선 TLB를 이용해야 됨 이때 또 miss가 나면 무한 재귀에 빠지게 될꺼야 -> 그래서 TLB miss handler와 같은 친구는 TLB의 일부 entry에 지정석으로 박아놓고 항상 hit이 되도록 만들어 준다.

hardware말고 software를 이용하여 TLB를 구성하게 되면 사용되는 여러 구조를 마음대로 바꿔줄 수 있다는 장점이 있다. (하드웨어를 건드리지 않고도!)

* CISC (Complex Instruction Set Computing) -> 많은 명령어를 가지는 방식, 되게 복잡하고 강력한 명령어를 가짐으로써 어셈블리어를 high-level처럼 사용함으로써 쉽게 사용할 수 있고 코드를 compact하게 만들 수 있는 장점이 있음
* RISC (Reduced Instruction Set Computing) -> CISC와 반대임, compiler target으로 만들어서 되도록이면 간단한 primitive로 구성되어 나머지는 compiler가 만들어 주도록 함 그래서 simple하고 uniform, fast한 장점이 있음

최근에는 이 둘을 섞어서 사용한다. (ex Intel)

TLB엔 뭐가 들었을까?
TLB가 fully associative라는 말은 한 translation이 TLB의 어느 곳에나 위치될 수 있다는 말임 생긴거에 따라 위치가 어느정도 특정되는 것이 아니라. 그래서 VPN으로 찾을때 병렬적으로 TLB 전체를 탐색해야함.VPN | PFN | valid bit | protection bit 등이 저장되는데 이 때의 valid bit은 page table에서의 valid bit과는 조금 다름

page table에서 valid bit이 invalid라는 말은 아직 해당 프로세스에 대해 그 page가 allocate되지 않았다는 말임(translation이 틀렸다는 말이 아니라) 그런데 TLB의 valid bit은 그냥 valid한 translation인지의 여부를 보는 것임, 처음 시작시에 TLB가 비어있으면 모든 entry를 invalid로 설정함 그리고 점차 populate되면서 valid한게 추가되는 것임

이런식으로 표시해두면 나중에 context switch할때 전부다 invalid로 함으로써 다른 프로세스가 기존에 남아있던 이전 프로세스의 page를 사용하지 않게 방지해줄 수 있음

Context switch한다면 TLB는 어떻게 되어야 할까? -> 각 프로세스별로 mapping이 다를텐데 -> 왜냐면 프로세스별로 page-table이 존재하니깐

만약 P1의 VPN 10이 PFN 100에 P2의 VPN 10이 PFN 170에 mapping된다면 VPN 10이 두가지로 mapping되는 것이므로 하드웨어가 이를 구분하지 못하게 된다.

![](19.3.jpg)

그러면 이를 해결하는 방법?

간단하게 context switch시 TLB를 flush하는 방법이 있음
그런데 이렇게 되면 각 프로세스들이 애써 채워놓은 TLB가 context switch한번 하고 돌아오면 다 사라지는 문제가 있음 -> 그러면 다시 populate하는 과정이 필요해서 느려져

flush하지 말고 동시에 존재하게 하려면? -> 각 TLB의 entry가 어느 프로세스에 속하는지에 대한 추가적인 정보를 넣자. -> address space identifier (ASID)를 넣어주자.

![](19.4.jpg)

또 추가적으로 성능을 개선할 수 있는 방법은 만약 서로 다른 프로세스가 각각의 VPN으로 동일한 PFN을 참조하는 경우 해당 page를 공유할 수 있다면 TLB의 공간을 절약하면서 사용할 수 있겠지? (physical page의 수가 줄어드니깐)

cache 사용시 항상 고려할 점은 cache replacement야 
대표적인 것으로는 random 하게 고르는 방법, least-recently used한 page를 제거하는 LRU가 있음
LRU는 locality 속성을 이용하는 방식이다.


짧은 시간에 TLB에 들어가는 page의 수(TLB coverage)보다 더 많은 page에 접근하는 프로그램의 경우 TLB miss가 엄청 발생하게 됨 -> 이는 page의 size를 키움으로써 어느정도 해결할 수 있음


physically-indexed cache vs virtually-indexed cache ??

# 20
## Smaller Tables

page table을 유지함으로써 얻는 문제 중 하나인 메모리를 많이 차지하는 것을 해결해보자.
page-table은 각 프로세스 별로 유지되기 때문에 수많은 프로세스를 동시에 사용하는 최근의 cpu에서는 문제가 될 수 있음

가장 간단한 해결법? -> page의 size를 키우자. -> 문제점은 internal fragmentaion!!

다른방법으로는 segmentation와 섞어보는것! -> code, stack, heap등의 segment로 나누고
각 segment별로 page-table을 관리해보자. 기존에 하던것 처럼 base와 limit 레지스터를 관리함으로써 각 segment별 page-table은 작게 유지될 것임 -> 문제점은 역시나 external fragmentation 대부분의 page는 fixed-size로 유지되겠지만 page-table의 관한 정보가 variable-size가 되기 때문에 이것이 그러한 문제를 일으킬 수 있다.

다른방법은?
multi level page table -> 가장 널리 쓰이는 방법..?

page-table 자체도 page로 나누어서 관리하자!
만약 사용되지 않는 page라면 (page-table에 대한) 아예 allocate 조차 하지 않음으로써 공간을 절약할 수 있음

![](20.1.jpg)

위 그림처럼 기존의 linear page table은 사용되지 않는 page가 있더라도 공간을 할당했음 
multi level에서는 사용되지 않는 것에 대해서는 table을 할당조차 하지 않음

이제 그러한 page-table을 위한 page를 관리하는 테이블이 있어야 하는데 이걸 page directory라고 함. 

multi level page table의 장점
1. 공간이 절약됨 -> 최대한 실제 사용하는 것에 비례하게 할당되거든
2. 잘 설계하면 segmentation과 다르게 page table자체도 page 크기로 잘 쪼개지니깐 관리하기도 편해

단점이라면 이제 2번의(혹은 그 이상의) indrection을 거쳐야 translation이 이루어진다는 것임
이러한 것이 대표적인 space time trade off임 (space를 절약한 대신 time을 잃었어)
그리고 복잡해져!


어떻게 multi level page table을 구성하는지는 책을 통해 확인하자

![](20.2.jpg)



# 26
## Concurrency

실행중인 하나의 프로세스를 위한 새로운 abstraction -> thread
기존에 프로그램은 하나의 execution point를 갖는다는 개념에서 벗어나 multi threaded program은 여러개의 execution point를 갖는다.

간단히 생각하면 여러개의 프로세스를 생성한것과 비슷하지만 다른 점이라면 스레드는 address space를 공유하기 때문에 동일한 데이터에 접근할 수 있다.

그것 말고는 레지스터와 같은 나머지 정보들은 각 스레드별로 고유한것을 갖기 때문에 다른 프로세스들 처럼 cpu를 사용하기 위해서는 context switch가 이루어져야 한다. 그 때마다 각 스레드의 정보들이 저장되어야 하는데 그 정보들은 TCB(thread control block)에 저장된다.

프로세스의 context switch와의 차이는 address space를 공유하기 때문에 page table을 저장하고 메모리에 올리는 작업을 하지 않아도 된다는 곳에 있다.

address space를 고유할때 주의할점이 있다. 만약 스레드들이 다른 스레드에 independent 하게 동작하고 싶다면 필요한것은 스택메모리인데 이를 공유하지 않고 따로 따로 부여해주기 위해선 다음과 같이 배치해야 한다.

![](26.1.jpg)

위처럼 하나의 address space에 각 스레드에 따로따로 스택영역을 할당해줌으로써 그 영역은 각 스레드 고유의 영역이 된다. (thread-local)

스레드를 쓰는 이유 !
1. parallelism -> 만약 여러개의 processor가 존재하는 컴퓨터라면 여러 스레드가 동시에 같은 작업을 함으로써 속도를 향상시킬 수 있음
2. I/O로 인한 blocking을 방지하기 위해 -> 한 프로그램이 실행중 I/O 작업이 생긴다면 그 동안 blocking되는 것이 아닌 스케줄러는 다른 스레드로 전환하여 작업을 계속 수행할 수 있게 해준다. 이렇게 함으로써 cpu utilization이 상승한다. (예를 들면 서버에서 클라이언트를 맞이할때)
물론 이것은 여러개의 프로세스를 돌림으로서도 가능하지만 address space를 공유하여 서로간의 데이터 공유가 간단해지는 장점이 있다.

비록 스레드가 address space를 공유하긴 하지만 고유의 register를 가지고 있기 때문에 명령 수행 도중 일어나는 context switch에 의해 race condition이 생길 수 있다.
==race condition== -> 결과가 코드 수행 타이밍에 의해 변화하는 것


여러개의 스레드가 특정 코드에 접근할 수 있게 되어 race condition을 만들어 낼 수 있을때
이러한 코드 영역을 critical section이라고 한다.

==critical section== -> 공유 변수에 접근하는 코드 영역(둘 이상의 스레드가 동시에 수행할 수 없는 (하면 안되는) 코드)

이러한 문제를 막기 위해 필요한 것이 mutual exclusion -> 즉 한 스레드가 그 부분을 돌고 있다면 다른 스레드는 그곳에 접근하지 못하게 하는것


이것을 가능하게 하는 가장 간단한 방법은 모든 명령어를 atomic하게 만드는거야
그래서 in-between state가 없게 해서 중간에 잘려나가는 일이 없도록 !

이건 불가능하니깐 하드웨어에선 synchronization primitive를 제공하고 os가 이것을 잘 사용해서 처리해보자.

한가지 문제가 더 있는데 그건 한 스레드가 다른 스레드의 작업을 무조건 기다려야 된다는거야
(I/O같이 느린 작업을 하더라도!!) -> sleep/waking mechanism으로 해결해보자.

용어 정리 =>
critical section : piece of code that access a shared resource, usually a variable or data structure

A race condition arises if multiple threads of execution enter the
critical section at roughly the same time; both attempt to update
the shared data structure, leading to a surprising (and perhaps undesirable)
outcome.

To avoid these problems, threads should use some kind of mutual
exclusion primitives; doing so guarantees that only a single thread
ever enters a critical section, thus avoiding races, and resulting in
deterministic program outputs.

# 21 
지금까진 address space가 매우 작다고 생각했는데 이제 엄청 크고 여러 프로세스가 많아서 address space들이 하나의 메모리에 올라가지 못한다고 생각해보자. -> 좀 느리더라도 큰 공간이 필요함 -> hard drive!

address space를 크게 잡는이유? -> 프로그램이 좀 더 쉽게 사용할 수 있도록 (여유 공간 확인할 필요를 제거함)

일부 page를 hard drive로 swap out하고 현재 사용할 page를 swap in함으로써 large virtual address space를 표현할 수 있게 할 수 있다.

disk에 이러한 공간을 swap space라고 함
이제 이곳과 메모리 사이를 여러 page가 왔다갔다 할 것임 -> 하드웨어가 swap space에 대한 physical address를 따로 저장하고 있어야 한다.

![](21.1.jpg)
-> 위 그림처럼 swap space를 사용함으로써 각 프로세스를 실제 가진 메모리보다 더 큰 메모리를 사용할 수 있다는 환상을 얻을 수 있다.

page fault 발생 -> 필요한 page가 swap space에 있는 것임 -> 그곳의 주소? -> page-table의 PTE에 저장해둔다.

address translation에서 이제 page table entry(PTE)를 보게 되면 그곳에 맞는 page가 실제 메모리에 올라와있는지 여부또한 저장해야됨 -> present bit으로 표현

만약 존재하지 않다면 (present bit 이 0이라면) -> page fault

present bit이 0이라면 PTE에 저장된 주소로 가서 page를 얻어와 메모리로 옮겨놓는다. 그 후 PTE가 가진 page의 주소값을 해당 메모리로 변경함

TLB miss면 -> page table참조 -> present bit = 0이면 -> I/O를 이용 disk to memory로 옮김
PTE업데이트 -> 명령 재시도 -> TLB miss -> page table참조 -> TLB 업데이트 -> 재시도 -> TLB hit

swap space에서 다시 메모리로 page in할때 공간이 없다면 그곳에 있던 다른 친구를 page out해야함 어떤 것을 고를지 결정하는 것 -> page-replacement policy

page fault control flow
![](21.2.jpg)
![](21.3.jpg)


더 이상 사용할 공간이 없을만큼 메모리가 꽉차야 page replacement가 일어날까? -> 아님 어느 정도 여유공간이 필요할 수 있음

OS는 현재 사용가능한 page 영역의 수에 대해 어느정도의 low watermark(LW), high watermark(HW)를 정해두고 이러한 기준을 바탕으로 진행하게 됨

보통은 swap daemon(page daemon)이라는 background thread가 이러한 사용가능한 page의 수가 LW이하가 된다면 replacement를 진행해 공간을 확보하게 된다.

여러 page를 묶어서 swap in/out하는 것도 disk I/O를 효율적으로 하여 성능을 최적화시킬 수 있다.


# 22
main memory는 시스템에서 사용되는 전체 page의 일부를 가지고 있는 것으로 볼 수 있는데 마치 virtual memory page의 cache처럼 작용한다고 볼 수 있다.

그래서 main memory에 없으면 cache miss가 있으면 cache hit이라고도 생각해볼수있음

결국 hit rate을 올리는것이 주요 관심사! -> 매우 적은 miss라도 속도에 큰 영향을 줌 -> disk access time은 memory access time보다 훨씬 느리니깐

우선 어떤 replacement policy를 정하기 전에 optimal은 어떤지 생각해보자.

optimal policy는 replacement가 필요할때 현재 cache에 있는 page중 가장 나중에 사용될 page를 evict 시키는 것이다. -> 그렇지만 그러한 정보를 얻는 것은 거의 불가능함 그냥 비교 대상으로 생각하자.


* cache miss의 type
* compulsory miss(cold-start miss) -> cache가 처음 비어있을때 생기는 miss (즉 item에 대한 제일 첫번째 참조일때 발생하는 miss)
* capacity miss -> 새로운 page를 넣기엔 공간이 부족해서 생기는 miss
* conflict miss -> item을 어디에 놓을지에 대한 제약사향때문에 생기는 miss -> set associative cache의 경우 발생한다. fully-associative에서는 생기지 않음


1) 가장 간단한 방식 FIFO !!
* belady's anomaly -> FIFO방식의 replacement에서 만약 cache의 수용가능한 page의 수가 늘어날때 특정 reference stream에서 hit rate이 증가하는 것이 아닌 오히려 감소하는 현상
LRU에선 생기지 않음 (stack property)?

2) 그다음은 random !

3) 1, 2는 아무 생각없이 함 -> 중요한 page를 그냥 버릴수도 있음
과거의 기록으로 미래를 예측해보자! -> page의 접근 frequency나 언제 가장 최근에 접근되었는지를 이용하여 선택하자. -> 이러한 것은 모두 locality의 특성을 활용하기 위한것임

가장 적게 사용된것 -> least-frequently-used(LFU)
가장 예전에 사용된것 -> least-recently-used(LRU)

1,2보다 더 나은 성능을 제공함 -> locality 있게 접근이 이루어진다면!

locality없으면 그냥 policy는 의미없고 cache size에 의해 성능이 결정됨
모든 page를 수용할만큼 크다면 hit rate은 100%에 수렴함

looping sequence와 같은 특수한 접근에는 LRU, FIFO가 worst case가 되고
이 땐 random의 성능이 더 좋을 수 있음

LRU에선 각 page별로 마지막 접근한 것이 언제인지를 기록해야함 어떻게 함? 효율적으로?
하드웨어적으로 시간을 기록할수도 있지만 page의 수가 커지면 evict 대상을 정할때
모두 비교해야돼

approximating LRU -> clock algorithm
use bit을 두고 page를 circular list로 만든다음 현재의 clock hand를 한칸씩 움직이며
use bit=1이면 최근에 사용된거니 0을 만들고 다음으로 이동 -> 0을 만날때까지 그것을 사용!

만약 메모리에 있던 page가 수정되었다면 disk에 있던 page를 업데이트 시켜줘야 하기 때문에 시간이 더걸린다. 수정이 안되었다면 그냥 버려도 될텐데
그래서 modified bit(dirty bit)을 둬서 우선순위를 주자
1순위는 use, dirty인 것 다음은 use인 것 그 다음은 둘다 아닌것 이런식으로 좀더 세세히 나눠줄 수 도 있다.


언제 page를 disk에서 메모리로 옮길것인지에 대한 고민도 해야돼!
근데 대부분은 demand paging -> 즉 해당 page에 접근할때 가져옴

만약 현재 필요한 page가 physical memory의 한계를 넘는다면? -> 계속 paging하기 바쁘겠지?
이런 조건을 thrashing이라고 해
어떻게 해결? -> 모든 일을 한번에 하려고 해서 생기는 일이니깐, 다 하려고 하지말고 몇개만 잘 돌리는 방법이 있음 (working set만 돌리자), 혹은 이렇게 만든 프로세스를 죽이는 방법도 있음

# 28 Locks

명령어를 atomic하게 실행하는 것을 필요로 할때 interrupt는 하나의 문제점으로써 작용한다. 이를 해결하기 위해서 lock이란 개념이 등장한다

lock은 단순히 그냥 변수에 불과함. 이 변수는 available 이나 acquired 둘 중 하나의 상태를 갖게되는데 available은 해당 lock을 어떤 스레드도 쥐고 있지 않다는 의미이며 acquired는 한 스레드가 lock을 쥐고 있다는 의미로 해석되게 된다.

lock() 함수는 lock을 얻으려고 시도함. 만약 누구도 lock을 갖고 있지 않다면 해당 스레드가 lock의 주인이 되고 다른 스레드들은 이 lock을 가질 수 없게 된다. 만약 이미 누군가가 쥐고 있다면 그것이 반환되어 얻을 수 있을때까지 blocking 상태가 된다.

unlock()은 lock을 쥐고 있던 스레드가 lock을 놓아줄때 사용함

lock은 프로그래머에게 스레드의 스케줄링에 조금은 관여할 수 있게 해주는 권한을 준다. (특정 부분에 대해 동시에 들어가지 못하게 함으로써)

POSIX에서는 이러한 lock을 mutex 라는 이름의 라이브러리로 제공한다

lock을 평가하는 기준
1. mutual exclusion을 제공하는지 (즉 동시에 여러 스레드가 critical section에 들어가지 못하게 막아주는지)
2. fairness -> lock을 원하는 스레드 중 starve되는 스레드가 존재하지 않는지
3. performance -> lock을 사용함으로써 얻어지는 오버헤드 최소화


lock을 만드는 가장 쉬운 방법은 interrupt를 꺼버리는 방법이야
만약 single processor에서 이런식으로 만들어준다면 잘 동작할거고 매우 간단해

단점은 우선 interrupt를 사용할 수 없기 때문에 또 다시 privileded operation을 하는 권한을 OS가 아닌 스레드 자체가 가져가야 해 이건 좀 위험
또 multiprocessor에선 원하는대로 동작하지 않음

여러개의 스레드가 동시에 여러 프로세서에 올라간뒤 실행되면 interrupt를 꺼놔도 각자 할일을 하니깐 동시에 critical section에 들어갈 수 있음

그리고 오랫동안 interrupt를 꺼놓으면 해당 인터럽트가 lost되어서 중요 이벤트를 처리하지 못할 수 있음

simple flag를 이용한 방법

![](28.1.jpg)

우선 제대로 동작하지 않을 수 있음 (while이후, = 1 이전에 인터럽트로 다른 스레드로 권한이 넘어간다면)
또한 spin waiting을 사용하므로 성능에 좋지않음

좀 더 나은 방식을 위해선 하드웨어의 지원이 필요함

다음과 같은 testandset 명령을 지원한다고 하자.
![](28.2.jpg)

즉 한번에 (atomically) 예전 값을 반환하여 test할 수 있게 해주면서 new값으로 set할수 있는 명령이 존재한다고 하자.

이렇게 되면 앞서 발생했던 flag 값을 갱신하는 부분이 atomic하게 변하므로 문제없이 가능해진다.

![](28_3.jpg)

이렇게 while로 기다리는 방식의 lock을 spin lock이라고 함. 가장 간단함
근데 더 나아지고 싶다면 preemptive 방식의 스케줄러가 와서 한놈이 돌고 있을때 그걸 빼앗을 수 있어야 함 (spin waiting 중이라면)

spin lock은 위에서의 lock의 판단기준에 의해 판단해보면
1. mutual exclusion은 잘 제공함
2. starvation 발생
3. multiple processor에선 꽤나 잘 동작함 (single에선 느리지만)


spin을 해결하는 방법 for에서 spin wait할 바에 그냥 CPU를 yield하는 방법
-> 이러면 어느정도 해결이 가능하지!만 이렇게 되더라도 context switch는 이루어져야 하므로
그곳에서 오는 오버헤드가 있음 또 여전히 starvation이 있음

# 30
## Condition variable

종종 thread가 진행을 계속하기 전에 특정 condition이 true인지를 확인하기를 원할때가 있음
(예를 들면 join()을 쓰면 child thread가 끝났는지를 체크하게 됨)

parent가 child가 끝날때까지 기다리게 하려면 어떻게?

![](30_1.jpg)
물론 위의 그림처럼 shared variable로 끝났는지 여부를 체크 해주면 될수도 있어 (spin lock 써서) -> 근데 waste CPU 이고 동시에 shared variable에 접근시 원하는 결과가 안나올수도 있음

Condition variale -> queue인데 스레드들이 특정 조건을 만족하지 못하면 스스로 그곳에 들어가고
다른 스레드가 그런 조건이 성립하도록 만들면 큐에서 자고있는 친구들을 깨우는 형태의 큐를 말함

wait()은 스스로 잠자러 가는 것 (큐에 들어가는것)이고 signal()은 내가 지금 상태를 바꿔서 그 상태를 원하던 자고 있는 스레드를 깨우는 작업을 한다고 볼 수 있다.

wait시엔 lock을 놓아주고 자러가고 일어나게 되면 해당 lock을 acquire해야 꺠어날 수 있음(다시 동작 할 수 있음) -> race condition 방지


![](30_2.jpg)
shared variable과 lock을 이용한 방식

우선 shared variable이 꼭 필요함?
![](30_3.jpg)
이렇게만 짜게되면 만약 child가 먼저 parent가 join을 부르기 전에 exit을 다 부르고 종료했다면
parent는 그걸 모르고 wait을 불러서 무한정 자게 된다.

그러면 lock이 signal, wait에 필요할까?
![](30_4.jpg)
이렇게 되면 race condition의 위험이 있음
만약 parent가 done==0이어서 들어가고 wait을 부르기 직전에 switch되어 child가 exit을 수행했다고 보자. 그러면 또 다시 parent는 wait에 들어가서 무한정 기다리는 상태가 됨


synchronization problem중 유명한
producer / consumer problem (bounded buffer problem)

여러개의 producer thread가 있고 또 여러개의 consumer thread가 있을 때
producer는 data를 만들어 내고 consumer는 만들어진 data를 사용한다. 그 data는 queue에 들어가고 그곳에서 사용되기 위해 꺼내진다

간단하게 그러한 큐가 하나의 정수로 표현된다고 해보자. 그러면 다음과 같은 코드로 작성할 수 있다.

![](30_5.jpg)

이걸 여러 스레드에서 다음과 같이 동시에 돌려보자.

![](30_6.jpg)

여러 문제점이 있는데 이 부분은 책을 읽는 것이 좋을것같음

# 31
## Semaphores

lock과 condition variable 둘다로 사용할 수 있는 synchronization primitive를 말함
semaphore는 integer value를 갖는 하나의 객체인데 두개의 연산이 있음
sem_wait(), sem_post(), 처음 integer value가 초기화 되는 값에 따라 행동이 달라짐

sem_wait() 은 1을 감소시키고 감소시킨 후의 값이 0이상이면 곧바로 반환
0미만이 되면 sleep에 들어간다.
sem_post()는 그냥 단순히 값을 1 증가시킨다. 그리고 만약 해당 세마포어에 대해 sleeping하는 스레드가 있으면 깨워준다.
또 integer value가 음수값이 된다면 그 것의 절대값은 기다리고 있는(sleep 중인) 스레드의 수를 의미한다.
![](31_1.jpg)


세마포어를 lock으로 사용할때 -> binary semaphore
초기값을 1로 설정함

그러면 첫 친구가 잡을땐 감소시키면 0이상이 되므로 진행하고 post전에
다른 친구가 또다시 wait을 하면 -1이 되므로 sleep에 들어가게 된다.
(held, not held 두 상태를 가지므로 binary semaphore라고 한다)


또한 순서를 강제하기 위해 사용할 수 도 있음

다음 코드에서 child가 먼저 완료한 후에 parent 가 완료하도록 만들기 위해선
세마포어의 초기값을 0으로 잡으면 된다.

![](31_2.jpg)


Producer/Consumer problem(Bounded Buffer) 문제를 해결해보자.

full, empty 세마포어를 두고 full은 처음에 0으로 empty는 동시에 실행할 수 있는 최대 스레드의 수로 초기화 해주자.(이를 MAX라고 하자)
![](31_3.jpg)

이는 MAX = 1일때 잘 동작함. 근데 만약 그 이상이라면?

두개의 producer가 동시에 put을 한다고 하자. 그러면 put을 실행중 switch가 나며 생기는 race condition이 발생할 수 있음


이 문제는 buffer를 공유하기 때문에 그래 -> mutual exclusion을 추가해보자.
![](31_4.jpg)

근데 문제가 있음 -> dead lock!
만약 consumer가 먼저 돌아서 mutex 잡고 full을 기다리려고 sleep이 된다면
producer는 이제 full을 놓아주려고 가야되는데 mutex를 잡지 못해서 그 mutex를 기다리려고 sleep하게 된다. -> dead lock이 생긴다 (cycle이 생기니깐)

간단히 mutex잡는 부분의 순서를 다음과 같이 바꾸어서 critical section의 범위를 줄인다.

![](31_5.jpg)

Reader/Writer Lock

(writer가 안쓰고 있으면 여러명이 동시에 읽을 수 있는 등의 특징이 있는 경우에 사용가능)

readlock, writelock, lock을 이용하여 진행
처음으로 들어가는 reader는 writelock을 잡게됨 그러면 이후에 writer가 들어오지 못하므로 동시에 여러명의 reader가 읽을 수 있게 된다. (마지막으로 나가는 친구는 writelock을 놓아준다)

![](31_6.jpg)

문제는 starvation -> fair하지 못할 수 있음 왜냐면 reader가 writer를 starve할 확률이 매우 크거든 -> 어떻게 해결??

Dining Philosophers 문제
먹을떄 2개의 젓가락이 필요해! (자신의 양옆에)
![](31_7.jpg)

여기선 putforks(), getforks()를 잘 구현해서 이 문제를 해결해야 함

![](31_8.jpg)
이런식으로 작성하게 되면 문제가 있음 -> deadlock
모든 사람이 자신의 왼쪽을 집게 되면 아무도 먹을 수 없음

매우 간단하게 이 문제 해결 가능 -> 최소 한 사람이 반대로 포크를 집어주면 된다. -> 사이클이 깨짐

![](31_9.jpg)

