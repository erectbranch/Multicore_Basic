# 8 비순차 실행

이전 장까지의 pipeline은 program이 순서대로 실행될 것이라고 생각했다. 이런 방식을 **in-order** processor라고 부르며, 직관적이나 큰 단점을 하나 가지고 있다.

아래 예시를 보자.

```c
x = data[10];    // 1번
y = x + 10;      // 2번
a = b / c;       // 3번
d = e * f;       // 4번
```

1번 load instruction이 CPU 내 cache에서 data를 찾지 못하고, main memory에서 data를 가져와야 한다고 하자. 그렇다면 보통 컴퓨터에서 이 작업은 100 cycle이 넘는 시간이 걸리게 된다.

> 덧셈 같은 단순한 integer 연산이 한 cycle마다 완료될 수 있다는 사실과 비교하면 매우 긴 시간이다.

게다가 3번과 4번 instruction은 1번과 아무 연관이 없다. 따라서 1번 instruction이 완료되지 못하고 기다리는 상황에 3번, 4번을 처리할 수 있으면 시간을 단축시킬 수 있을 것이다.

따라서 processor가 직접 instruction 사이의 dependence를 분석하고, 먼저 실행할 수 있는 instruction을 미리 실행시키는 방법을 바로 **out-of-order** execution(OOO 혹은 OOOE. 비순차 실행)이라고 한다.

더 나아가서 instruction throughput을 높일 수 있는 방법이 있다. 4번도 나머지 instruction과 무관하므로 pipeline을 늘려서 3, 4번을 parallel로 처리하면 된다. 이런 구조를 **superscalar**(슈퍼스칼라) processor라고 한다.

---

## 8.1 Instruction Level Parallelism

out-of-order execution은 **ILP**(Instruction Level Parallelism. 명령어 수준 병렬성)을 찾아서, 원래 순서가 아닌 data의 flow에 따라 instruction을 처리한다.

우선 **parallelism**(병렬성)의 대표적인 예로 multi thread나 multicore를 먼저 떠올릴 수 있다. 두 개의 independent한 thread가 있다면, 이 두 thread는 동시에 작동한다. single core processor에서도 OS의 time division(시분할) multitasking 기능 덕분에 논리적으로는 동시에 실행되는 것처럼 보인다. 이런 parallelism을 **TLP**(Thread-level parallelism. 스레드 수준 병렬성)이라고 한다.

> 요약하자면 TLP는 프로그래머가 명시적으로 multi thread로 program을 작성했을 때 얻는 parallelism이다.

하지만 프로그래머가 multi thread로 작성하지 않은 일반적인 single thread program에서도, 동시에 실행 가능한 instruction을 찾을 수 있다. 다시 말해 TLP보다 규모가 작은 parallelism이 있다는 뜻이다. 아래 예시를 보자.

```c
x = y + 1;    // 1번
a = b * 2;    // 2번
```

- 모든 variable은 pointer가 아닌 int/float 같은 scalar type이라고 가정한다.

1번과 2번 instruction은 동시에 실행 가능하다. 둘 중 어느 것이 먼저 실행되든, 동시에 실행되든 program의 context에는 차이가 없다.

> **context**란 1번, 2번 instruction을 완료했을 때의 computer architectural state(컴퓨터 구조 상태)를 가리킨다.

다시 말해 1, 2번을 어떤 순서로 실행해도 이 program의 **semantic**(의미)를 정확하게 실행한다. data dependence만 지키면 실제 실행 순서는 바뀌어도 큰 문제가 없다. 바로 이것이 ILP이다.

> 하지만 주의할 점은 어떤 연산은 flag register의 value를 변경시키기 때문에, <U>flag value를 변경시키거나 이를 이용하는 연산은 별개의 dependence를 유발</U>한다.

> 일반적으로 과학 계산 program에서 ILP를 매우 많이 발견할 수 있지만, 그렇지 않은 program 역시 다양하다. 

이번 ch08 첫 예시로 본 code를 graph로 그려서 ILP를 나타내 보자.

![ILP graph 1](images/ILP_ex_1.png)

- 1번, 2번: variable x에 RAW data dependence를 갖는다.

   - 따라서 서로 edge(간선)으로 dependence를 표현한다.

- 3번, 4번은 아무런 관련이 없다.

   - 따라서 그냥 vertex(정점)으로만 그려진다.

만일 모든 instruction이 한 cycle 안에 달성되고, ILP를 찾아서 parallel로 실행한다면 2 cycle이면 모두 완료할 수 있을 것이다.(in-order로 처리했다면 4 cycle이 걸릴 것이다.) 즉, 이 program의 ILP는 4/2 = 2라고 할 수 있다.

> 만약 모든 instruction이 dependence로 묶여 있었다면 ILP는 4/4 = 1이었을 것이다.

ILP를 계산하려면 위처럼 dependence graph를 그린 뒤, root node에서 leaf node까지 가장 긴 경로의 길이를 구한다. 즉, 우선 **critical path**(임계 경로)를 구한다. 그 다음 전체 instruction 개수에 나누면 된다.

> IPC(Instruction Per Cycle)와 혼동해서는 안 된다. 에초에 ILP는 program의 고유한 성격에 해당하고, IPC는 processor의 구현 방식에 좌우된다.

OOOE는 **scheduling**(스케줄링) 관점에서 볼 수 있다. OOOE는 instruction 사이의 dependence를 파악해서 dynamic하게 scheduling한다. 그래서 OOOE를 **dynamic scheduling**(동적 스케줄링) 기법이라고 부른다. 또한 ILP는 hardware가 아닌 software(compiler)가 대신 찾아줘도 된다.

> 한편 computer architecture 중에서 **dataflow**(데이터플로우)라는 전통적인 방식과 매우 상이한 처리 방식이 있다. dataflow는 OOOE의 이상적인 모습에 해당한다.

---

## 8.2 superscalar pipeline

**superscalar**(슈퍼스칼라)를 간단히 말하면 pipeline이 여러 개 있어서 동시에 instruction을 처리할 수 있는 구조를 뜻한다.

![슈퍼스칼라 구조](images/superscalar.png)

이전까지 본 pipeline processor는 pipeline이 하나밖에 없어서, 1 cycle에 최대 instruction 1개만 완료할 수 있었다.(IPC = 1)

하지만 위 그림처럼 pipeline이 두 개가 되면서 이상적인 상황이라면 cycle마다 최대 두 개의 instruction이 **issue**(투입)과 **graduation**(retirement. 완료)가 가능하다.(IPC = 2)

> 보통 superscalar processor는 동시에 issue/graduation 가능한 instruction 개수에 따라 N-issue, N-wide, N-way superscalar로 표현한다.

이상적으로는 N개의 instruction을 fetch하여 pipeline에 넣을 수 있지만, 실제로는 instruction cache의 제약, dependence, branch 때문에 대체로 N개보다 작은 instruction을 fetch할 수 있다.

그런데 superscalar processor라는 개념은 오직 instruction pipeline이 여러 개 복제되어 있는 것만을 의미한다. in-order인지, out-of-order인지 여부는 무관하다. 하지만 out-of-order processor로 좋은 성능을 내려면 superscalar로 만들어야 한다.(따라서 오인하기 쉽다.)

> 다시 말해 out-of-order processor도 superscalar가 아닌 1-wide processor일 수도 있다. 반면 in-order processor가 superscalar일 수 있다.

superscalar processor 구현에는 어려움이 따른다. N-wide processor라면 최대 N개의 instruction을 fetch해야 하지만, instruction cache가 이를 손쉽게 해결하지는 못한다. instruction이 제대로 정렬되지 않았거나, 다른 cache line에 걸쳐 있는 등의 상황이 발생하기 때문이다. 또한 앞서 말한 것처럼 branch가 끼어 있다면 문제가 더 어려워진다. bypass logic 역시 복잡해진다. superscalar는 이런 문제도 N배로 커지는 것이다.

---

## 8.3 OOOE 구현: Tomasulo Algorithm

out-of-order superscalar processor는 주어진 instruction에서 ILP를 찾아 실행 가능한 instruction부터 처리한다. 그런데 이를 처리하는 algorithm은 어떻게 가능할까? 핵심은 instruction scheduling에 있다는 점을 상기해 보자.

```c
x = data[10];    // 1번
y = x + 10;      // 2번
a = b / c;       // 3번
d = e * f;       // 4번
```

앞서 본 예제를 다시 살펴보자. 편의상 모든 variable(x, y, a, b, c, d, e, f)를 모두 register로 가정하자. data[10]은 어떤 data가 담긴 address를 뜻한다. 

![토마슐로 알고리즘](images/tomasulo_algorithm.png)

1. 일단 instruction을 모두 fetch, decoding하여 어떤 queue에 넣어야 한다.(IF, ID 단계)

2. queue에서 대기하며 operand가 준비되는 것을 검사하며 기다린다.(scheduling 단계)

3. 준비가 다 되었으면 실제 계산을 수행한다.(EXE 단계)

4. 완료가 됐을 때 만약 지금의 연산 결과를 필요로 하는 instruction이 있다면 알려준다. 끝으로 instruction은 queue를 떠나고 프로그래머는 과정이 완료가 된 것으로 보인다.(OS 단계)

위 4단계를 예제에 맞춰서 설명하면 다음과 같다.

1. 1번 instruction은 queue를 하나 할당받아서 들어간다. 1번 register x에 value을 쓸 것이므로, 뒤에 따라오는 instruction들 중 x를 쓰는 instruction들에게 반드시 자신을 기다리라고 당부한다.

   - 예들 들어 register file에서 x의 value를 바로 읽지 못하게 한다.

   - 1번은 memory load이므로 cache에 이 data를 가져와달라고 요청한다. 허나 cache miss가 발생하면 긴 시간이 소요될 것이다.(지금 예제에서 발생했다고 가정하자.)

2. processor는 다음 instruction인 2번을 읽어 queue에 넣는다. 그런데 register x의 value가 필요하고, 이는 1번이 완료되어야 쓸 수 있도록 되어 있다. 따라서 1번을 계속 기다린다.

3. 3번 instruction이 queue에 들어온다. register b와 c를 읽어야 하는데 이번에는 그냥 읽을 수 있다. 따라서 processor는 ALU 하나를 할당해서 계산을 수행한다. queue에서는 이 instruction을 이제 빼내고 완료되었다는 사실을 전한다.

4. 4번 instruction도 3번처럼 처리된다.

5. cache miss에 의해 긴 시간이 소요되며 마침내 1번 instruction의 data가 도달한다. 1번은 이제 register file에 값을 갱신하고, 완료가 되면 이 사실을 queue에서 기다리는 instruction에게 알린다. 그러면 2번은 비로소 register x의 값을 읽울 수 있게 된다.

위 과정을 보면 data dependence에 따라 instruction이 scheduling 되었음을 알 수 있다. 이 algorithm이 OOOE의 기원인 **Tomasulo algorithm**(토마슐로 알고리즘)이다.

> 1967년 이 알고리즘을 구현한 Robert Tomasulo의 이름을 딴 것이다. 지금에 와서는 어느 정도 바뀌었지만 핵심은 그대로 남아 있다.

---

## 8.4 Instruction Window(명령어 윈도우)

> [비순차 프로세서 파이프라인의 핵심 기술](http://cloudrain21.com/out-of-order-processor-pipeline-1)

현실에서 실제 program의 크기는 매우 크기 때문에, code 전체를 분석해 이상적인 ILP를 찾는 것은 hardware의 제약으로 불가능에 가깝다. 따라서 out-of-order processor에서 찾는 ILP는 상대적으로 매우 제한된 범위 내에서 찾는다. **Instruction Window**(명령어 윈도우)가 바로 이 범위를 가리키는 개념이다.

예를 들어 128개의 instruction window를 갖는다는 말은 128개의 instruction 속에서 ILP를 찾을 수 있다는 뜻이다. 다르게 말하면 128개의 instruction을 잠시 들고 있으면서 OOOE를 수행한다는 말이기도 하다.

![명령어 윈도우](images/instruction_window.png)

instruction window는 보통 연속적으로 움직인다. 내부에서 가장 오래된 instruction이 완료되면 한 칸씩 이동하게 된다. 이 instruction window 안에 있는 instruction을 **in-flight**(인-플라이트) instruction이라고 표현하기도 한다.

> instruction window는 hardware의 제약을 받아 보통 100여개가 넘는 정도에서 ILP를 탐색하지만, compiler가 software 관점에서 수천 개의 instruction 사이에서 최적의 ILP을 찾는다.

---

## 8.5 register renaming을 이용한 false dependence 제거

기계어상에서 data dependence는 register 이름이 겹치는지 파악하면 간단히 알 수 있다. 그 중에서도 RAW는 true dependence, WAW, WAR는 false dependence였다. in-order pipeline processor에서는 일반적으로 WAW, WAR dependence로 pipeline hazard가 일어나지 않았다. 아래 예시를 보자.

```c
r2 = r1 + 5;    // 1번
r1 = r0 + 1;    // 2번: WAR
r1 = r0 + 2;    // 3번: WAW
```

![순차 프로세서에서의 false dependence](images/in-order_false_dependence.png)

- 2번 instruction이 r1에 value를 store하는 시점은 WB, 즉 t7 시점이다.

- 그런데 1번 instruction은 이보다 **RR**(Register Read)을 빠른 시간(t3)에 마치므로 아무런 문제가 일어나지 않는다.

- 3번 instruction도 동일한 이유로 dependence로 인한 hazard가 발생하지 않는다.

하지만 OOOE가 되면 WAW/WAR 같은 false dependence도 문제를 일으킨다. 따라서 ILP를 높이기 위해 한 차례 **Register Renaming** 과정을 거쳐서 false dependence를 제거해야 한다.(이 작업은 IF 단계에서 진행된다.) 아래 예시를 보자.

```c
r2 = r1 + 5;    // => F0 = r1 + 5;
r0 = r2 + 4;    // => F1 = F0 + 4;
r1 = r0 + 1;    // => F2 = F1 + 1;
r1 = r0 + 2;    // => F3 = F1 + 2;
r3 = r1 + 3;    // => F4 = F3 + 3; 
```

instruction마다 주석으로 RR을 적용한 버전을 작성하였다. 하지만 이런 방법을 실현하기 위해서는 ISA가 정의하는 register 이외에 추가 register가 있어야 RR을 효과적으로 사용할 수 있다.(부족하면 제약이 생기기 때문이다.) 따라서 register file을 다음과 같이 나눈다.

- **ARF**(Architecture Register File): 구조 레지스터 파일. ISA가 정의하는 register file이다. 함부로 바꾸면 하위호환성이 어긋나기 때문에 쉽게 바꿀 수 없다.

- **PRF**(Physical Register File): 물리 레지스터 파일. 프로그래머가 자유롭게 사용할 수 있다.

- **RAT**(Register Alias Table): 위 register file의 변경 내역을 기억하는 장소.

위 예제에서 r0 ~ r3은 ARF, F0 ~ F4는 PRF에 해당된다. 아래와 같은 변경 규칙으로 RR이 진행되었다.

1. instruction의 target register name을 현재 가능한 PRF 중 하나로 바꾼다. 이 변경 내역을 RAT에 기록한다.

   - 예를 들어 예제에서는 r2가 F0로 대체되었다고 기록한다.

2. 다음 instruction이 register를 읽을 때는 RAT를 살펴보고, 현재 어떤 PRF에 mapping이 되었는지 확인하고 읽는다.

   - 예제에서 2번 instruction은 r2가 F0로 대체되었다는 기록을 읽는다. 또한 r0를 F1으로 대체한다.

3. instruction이 완료되고 자신이 이 논리 register를 쓰는 마지막 instruction이라면 RAT value를 default로, 즉 ARF로 바꾼다.

   - RAT에서 r2 항목을 F0에서 다시 r2로 바꿔둔다.

> 다만 과정 중에 PRF에 여유가 없다면 stall이 발생한다.

> RR은 superscalar 구조에서 더욱 구현이 복잡하다는 문제를 안고 있다.

---

## 8.6 Reservation Station

OOOE의 핵심은 dynamic instruction scheduling에 있다. program에 적힌 순서대로가 아닌, operand가 완료되는 순서대로 처리하기 위해서는 scheduling device가 필요하게 된다. 이 device가 바로 **RS**(Reservation Station. 대기소)이다.

OOOE에서는 instruction decoding 이후 RS device에서 queue 슬롯 하나를 할당받는다. instruction은 RS에 자리 잡고 **Wake-up**(깨우기) 작업과 **Select**(선택) 작업 두 단계를 거치게 된다.

Wake-up은 어떤 instruction의 모든 operand가 완료되었는지 감시하는 device이다. 모두 완료되면 instruction execution 준비가 되었다는 뜻이며, 이제 필요한 계산 장치를 할당하면 된다. 여기서도 scheduling이 필요한데, 이 작업이 Select에 해당한다.

![인텔 네할렘 구조의 RS](images/nehalem_rs.png)

- 이 구조에서 RS는 36개의 instruction을 담을 수 있으며, 15개의 실행 장치를 가진다.

- 15개 실행 장치를 한 단위, **Post**로 묶는다. 위 그림에서는 6개의 Port가 있으며, 따라서 36대 6 scheduling만 구현하면 된다.

위 그림은 intel nehalem microarchitecture의 RS을 나타낸 것이다. 총 36개의 RS가 있으며, 말머리에 붙는 Unified란 instruction type과 상관없이 RS를 통합했다는 의미다.

> processor에 따라 integer/floating point/load/store에 따라 RS를 독립적으로 관리하기도 한다.

> 하지만 wake-up 작업이 매우 복잡하기 때문에 RS를 늘리기는 굉장히 어렵다. 이 부분이 현대 out-of-order microprocessor pipeline 구현의 bottleneck 지점이다.

이어서 아래 4-wide superscalar 구조 예시를 보자.

- 4-wide superscalar 구조라면 최대 instruction을 4개까지 완료할 수 있다.

- cycle마다 RS에 있는 instruction을 자신의 operand가 완료되었는지 검사해야 했다. 

- 이때 nehalem 구조처럼 instruction 36개가 RS에 있다면, uop가 최대 3개의 argument를 취할 수 있으므로, 총 108개의 operand를 대기하는 상황이 발생할 수 있다.

- 따라서 완료된 instruction 4개의 연산 결과가 108개의 operand와 일치하는지 동시에 검사해야 한다.

이 작업은 보통 hardware device 중 **CAM**(Content-Addressable Memory)라는 것으로 구현된다. CAM은 매우 빠르게 어떤 data를 찾는 데 쓰이는 device인데, 이 device 역시 매우 복잡하기 때문에 늘리기 힘들다.

---

## 8.7 Re-Order Buffer

out-of-order processor라고 해서 모든 것을 뒤죽박죽 실행하지는 않는다. 예를 들어 instruction fetch, decoding를 out-of-order로 시행하지는 않는다. instruction fetch는 여전히 program에서 정한 순서대로 진행된다.

> out-of-order란 <U>오직 execution 단계에서 operand가 먼저 완료되는 순서대로 실행</U>하는 것이다.

따라서 instruction fetch, decoding은 in-order하지만, execution은 out-of-order로 진행된다. 따라서 결과적으로 instruction는 out-of-order하게 완료될 것이다. 하지만 이런 상황은 프로그래머가 굉장히 혼동을 겪게 만들 수 있다. 따라서 **ROB**(Re-Order Buffer)를 둬서 instruction이 in-order하게 완료될 수 있도록 만든다.

앞서 instruction decoding이 되면 scheduling을 위해 RS에 머무는 것을 보았는데, 사실 이때 RS만이 아니라 ROB에도 같이 할당된다. RS에서는 instruction이 자신의 operand가 모두 충족되면 execution할 수 있는 시점에 떠날 수 있었다. 하지만 ROB는 execution이 완료된 시점에서도 바로 밖으로 노출되지 않는다.

> 이런 점 때문에 사실상 ROB의 크기가 결국 out-of-order processor의 instruction window를 결정짓는다.

> nehalem 구조에서 RS는 36개를 가진 반면, ROB은 128개를 가지고 있었다.

![ROB](images/ROB.png)

- 3번, 4번은 instruction 1,2와 무관하여 이미 execution을 마쳤다.

- 하지만 3번, 4번은 ROB에서 가장 오래된 instruction이 아니기 때문에, 지금 바로 commit을 하면 프로그래머는 out-of-order한 결과를 보고 혼동을 겪을 것이다. 따라서 앞서 있는 instruction의 commit을 대기한다.

ROB는 일종의 circular queue다. instruction decoding 후 ROB에 instruction이 자리를 잡고, execution이 완료되면 그 사실을 기록한다. 그리고 그 instruction이 가장 오래된 instruction에 해당될 때 ROB를 떠날 수 있다. 이 과정을 **commit**(커밋), **retirement**(퇴장), **graduation**(마침)으로 표현한다. 

> 또한 동시에 ARF에 value를 최종적으로 갱신하는 과정이나 memory에 value를 store하는 과정도 commit이 되어야만 가능하다. 만약 commit 전이라면 보류된다.

ROB로 in-order하게 instruction을 commit해야 하는 또 다른 중요한 이유는 fault와 exception 처리에 있다. 만약 out-of-order하게 execution한 program에서, 앞선 instruction보다 먼저 execution한 instruction이 page fault를 일으키켰다고 하자. 그렇게 OS로 제어가 넘어가게 되면 프로그래머는 혼란을 겪을 수밖에 없다.

> 따라서 out-of-order processor에서는 fault와 exception도 하나의 결과로 간주하고, 발생하는 시점에서 즉시 조치를 취하지는 않는다. ROB에서 기다리고 있다가, instruction의 commit 시점에서 외부(프로그래머)에게 알리게 된다.

---

## 8.8 out-of-order processor pipeline 정리

정리하자면 out-of-order processor의 주요 pipeline 단계는 다음과 같다. 크게 분류하면 (1) IF,ID (2) 자원 할당(ROB와 RS 할당) (3) out-of-order execution (4) commit으로 나눌 수 있으며, 세부적인 단계는 다음과 같다.

| Fetch | Decode | Rename | Issue | Schedule | Exec | Writeback | Commit |

1. instruction fetch: 보통 superscalar 구조이므로 in-order processor보다 더 복잡하다.

2. instruction decode: 만약 x86 processor라면 RISC 형식의 uop로 decode한다.

3. register renaming: false dependence를 제거하기 위해 RR 작업을 수행한다.

4. issue(혹은 allocation): 준비된 uop를 각종 장치, 예를 들어 RS, ROB에 할당한다.

5. scheduling: operand 완료를 매 cycle마다 검사하고, 준비되면 실행 장치를 scheduling한다.

6. execution: 연산한다.

7. Writeback: 연산 결과를 ROB나 PRF 같은 곳에 쓴다.(아직 commit 전이므로 외부에 노출되지는 않는다.)

8. commit: 차지한 자원을 반환하고 결과를 최종적으로 반영한다. ARF나 memory를 갱신한다.

---

