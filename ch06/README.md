# 6 processor 기본 동작

microprocessor는 instruction 하나하나를 처리하므로, **Instruction Set Processor**라고 부른다. processor가 하는 일이라곤 그저 program에 있는 instruction의 flow를 지시대로 처리해 **architectural state**(구조적 상태, process의 state를 가진 각종 register들의 값)만 약속대로 반영하는 것뿐이다.

ch01에서 본 예제를 다시 살펴보며 processor의 동작을 살필 것이다. 예제는 다음과 같았다.

- 4byte integer variable A에 constant 7을 더해 다시 A에 쓰라"

  - add dword ptr [A], 0x07

  - (편의상 16진수로 표현하면) 0x83 0x45 0xF8 0x07

주어진 memory 공간에 이런 이진수 흐름이 적혀 있다면, processor는 어떻게 이런 instruction을 처리할까? 어떻게 해당 memory와 register 값을 지시대로 바꿀까? 또한 if문에 의한 branch 등은 어떻게 처리할까?

---

## 6.1 instruction 처리의 기본 다섯 단계

일단 살펴볼 instruction을 특성에 따라 4가지로 구분할 수 있다.

1. ALU 연산: 사칙 연산, AND, XOR 같은 비트 논리 연산

2. memory load: operand에 기록된 memory address를 구해 그 내용을 읽고 지정된 register에 value를 쓴다.

3. memory store: 주어진 register의 내용을 지정된 memory address에 쓴다.

4. branch: 주어진 condition을 계산해 다음에 수행될 PC를 얻는다.(조건 분기문일 때)

이런 종류마다 구현하는 로직은 다르지만, 중복되는 작업 단계도 존재한다. 이 책은 공통적인 단계를 세부 다섯 단계로 나눈다.

1. **Instruction Fetch**(IF. 명령어 인출)

2. **Instruction Decoding**(ID. 명령어 해독)

3. **Operand(s) Fetch**(OF. 피연산자 인출)

4. **Instruction Execution**(EX. 명령어 실행)

5. **Operand Store**(OS 또는 Write Back. 결과 저장)

이 과정을 code로 작성하면 다음과 같은 구조가 된다.

```c
while (실행할 instruction) {
    fetch(...);
    decode(...);
    operands_fetch(...);
    execution(...);
    oprand_store(...);
}
```

> 실제 pipeline processor나 out-of-order processor는 좀 다른 단계로 구성되어 있지만, 전체적인 방향성은 같다.

우선 한 예시로 DRAM 모듈을 보자. 

```c++
// dram_t 클래스
class dram_t{
    // 몇몇 통계값을 저장한 variable 정의
    protected:
        counter_t total_accesses;
        counter_t total_burst;
        int       best_latency;
        int       worst_latency;
    
    // 핵심 interface 정의
    public:
        dram_t(void);
        virtual ~dram_t(void);
        virtual int access(enum cache_command cmd, md_paddr_t baddr, int bsize);
        virtual void refresh(void);
        static void dram_create(void);
};
```

DRAM에 접근하는 access function이나, DRAM 특성상 주기마다 전력을 공급해주는 것을 모사하는 refresh function을 볼 수 있다. 이 C++ abstract class(또는 interface)를 inheritance(상속) 받으며 특정 DRAM 행동을 시뮬레이션한다.

---

### 6.1.1 Instruction Fetch

IF에서는 우선 처리할 instruction을 memory에서 가져온다. 이 instruction은 PC 혹은 IP register가 가리키는 address에 있다.

이때 거의 모든 processor는 **Instruction cache**(명령어 캐시)를 두며 instruction fetch latency를 줄인다.

이 다음 instruction cache에서 현재 PC가 가리키는 address의 내용, 즉 아직 해독되지 않은 raw byte data 뭉치를 읽어와서 byte queue에 넣는다.

이 과정을 code로 구현하면 다음과 같다.

```c++
void CoreFetch::step() {
    /* intruction TLB를 fetch한다. */
    core_->memory_->ITLB_.CacheProcess();
    /* instruction cache를 읽는 작업을 수행한다. */
    core_->memory_->IL1_.CacheProcess();
    /* byteQ를 채우라고 지시한다. */
    byteQ_.Request(pc_ & byteQ_line_mask);

    /* PC를 갱신하며 다음 instruction을 가져올 곳을 계산한다. */
    UpdatePC();

    /* instruction TLB와 cache에 다음 내용을 요청한다. */
    core_->memory_->ITLB_.Enqueue(CACHE_READ, pc_, ITLB_Callback, ...);
    core_->memory_->IL1_.Enqueue(CACHE_READ, pc_, IL1_Callback, ...);
}
```

> programmer가 보는 memory는 <U>process마다 가상으로 주어진 **address space**(주소 공간)</U>였다.(virtual address) 이렇게 vitual address를 구분하고, 다시 physical address로 translation하는 unit이 바로 **MMU**(Memory Management Unit. 메모리 관리 유닛)이었다.

> 하지만 translation도 적지 않은 시간이 걸릴 수 있다. virtual address의 상위 bit 중 일부, 즉 **VPN**(Virtual Page Number. 가상 페이지 번호)를 가져온 뒤, process의 page table에서 이와 대응되는 **PPN**(Physical Page Number)를 찾아야 한다. 이를 cache를 도입해서 처리하는 장치가 **TLB**(Translation Lookaside Buffer)이다.

> 보통 L1 cache가 data cache와 instruction cache로 분리하듯, TLB도 instruction TLB(ITLB)와 data TLB(DTLB)로 나눈다.

UpdatePC() 전까지는 cache가 주어진 일을 하도록 진행시키고, cache가 data를 불러오면 그 내용을 byte queue에 넣는 과정에 해당된다. 이처럼 fetch 단계는 byte의 flow만 queue에 넣어주면 된다.

참고로 이렇게 넣은 bytes를 **pre-fetched bytes**라고 한다.

> 실제 simulation code는 예시보다 훨씬 더 복잡하다.

> Execution Unit이 instruction을 decode하거나 execute하는 동안, Bus Interface Unit(BIU)을 통해 instruction bytes를 fetch하며 다음 instructions를 준비하게 된다.

---

#### 6.1.1.1 PC 값 갱신

UpdatePC() 구문은 말 그대로 PC를 갱신하는 작업이다. 즉, <U>바로 다음에 읽어야 할 instruction의 address를 구하는 작업</U>이다.

만약 모든 instruction이 4byte로 구성되는 간단한 RISC 구조라면, UpdatePC() function은 간단히 'PC += 4' 연산을 진행할 것이다.

하지만 x86 같이 가변 길이를 갖는 instruction set은, 현재 instruction이 몇 byte인지 알아내야 하므로 더 복잡하다.

또 고려해야 할 점은 branch(분기문)이다. 만약 instruction이 'if, goto, switch 또는 function call' 구문이라면, 다음에 실행될 program은 단순한 instruction이 아니라, branch가 지정하는 address가 된다. 따라서 branch에서는 PC 수정이 간단치 않다. 

-  goto와 function call과 같은 **unconditional branch statement**(무조건 분기문)과 그렇지 않고 condition에 따라 결정되는 **conditional branch statement**(조건 분기문)에 따라 다르게 적용된다.

- 또한 branch target address(분기 목적지)를 바로 알 수 있는 경우인 **direct branch**(직접 분기문)과, 그렇지 않은 **indirect branch**(간접 분기문)으로도 나뉜다.

> 일반적인 ISA 수준에서 high-level language가 제공하는 switch-case를 지원하지는 않는다. 또한 각종 loop control flow statements(do, for, while, break)도 결국 <U>모두 goto, jump 같은 uncoditional branch statement로 compiler가 translation</U>한다.

대다수 branch는 program을 실행하지 않으면, 분기 여부나 분기할 지점을 알 수 없다. 특히 지금 막 instruction fetch를 진행했다면, 이 control flow statement의 결과와 일부 branch의 target address를 바로 아는 것은 어려운 일이다. 에초에 이 단계에서는 단순히 byte flow만 읽었기 때문에 instruction이 branch인지도 모른다.

하지만 지금은 UpdatePC() function에서 Oracle이라는 모듈이 어디로 갈지 알려준다고 가정하자.

> 이런 식으로 시뮬레이터를 만들 땐 앞으로 일어날 모든 일을 알려주는 신의 역할을 하는 모듈을 가지고 작업한다. 이 신을 보통 **Oracle**(오라클)이라고 한다.

```c
Mop_t* Mop = Oracle_->Execute(pc_);
if (Mop->IsBranchOrCall()) {
    pc_ = Oracle_.어디로_가야_해요(Mop);
} else {
    pc_ = pc_ + Mop->Length();
}
```

- Mop_t는 x86 CISC instruction을 나타내는 Macro-op(매크로 옵. Micro-op과 반대) 자료구조이다.

---

#### 6.1.1.2 PC 값 갱신 이후

그 다음 line은 update된 PC가 가리키는 address의 내용을 읽어오라고 cache에 의뢰한다.

그런데 이때 cache 내에서 어떤 event가 발생하면 이를 처리해야 한다. 그래서 event handler callback function인 IL1_Callback/ITLB_Callback을 넣어 처리한다.

즉, 지금까지의 instruction fetch 과정을 요약하면 다음과 같다.

1. virtual address인 PC를 받아 이를 physical address로 바꾸면서, instruction cache에서 raw byte flow를 byte queue에 저장한다.

2. 다음에 옮겨갈 PC를 구해 update한다.

3. 이 update한 PC가 가리키는 내용을 읽어오도록 cache에 의뢰한다.

---

## 6.2 Instruction Decoding

instruction fetch를 통해 byte queue에 저장된 instruction은 앞서 본 '0x83 0x45 0xF8 0x07'과 같은 숫자 흐름이다.

> 구체적으로 opcode, operand, destination address, addressing mode, 또는 간단한 branch라면 branch target address도 읽는다.

ID는 이런 flow를 **parsing**(파싱. 어떤 문장을 분석하거나 문법적 관계를 해석하는 과정)하는 과정이라고 할 수 있다.

이때 RISC라면 instruction 길이가 일정하고 opcode 위치도 정해져 있기 때문에 decoding이 편리하다. 예를 들어 MIPS에서는 최상위 6bit에 opcode가 저장되므로 instruction이 덧셈인지 분기문인지 한 번에 알 수 있다.

반면 x86 instruction set은 decoding 수행이 어렵다. 극단적으로 x86 CISC는 REP(repeat) 접두어, 즉 loop를 내장한 instruction을 사용하기도 한다. 게다가 x86-64로 확장되면서 이런 경향은 더 심해졌다.

> 대체로 이런 부분을 테크닉으로 개선하려는 노력을 한다. 예를 들어 일부만 먼저 decoding하는 방식으로 처리하며 시간을 단축한다.

> 사실 최신 processor는 고속화를 위해 CISC instruction을 RISC 형태의 uop(마이크로 명령어)로 분해하도록 설계된다. 이때 decoder가 uop로 변환을 해준다.

0x83 0x45 0xF8 0x07 instruction을 decoding하면 다음과 같은 덧셈 instruction이 나온다.

- add dword ptr [A], 0x07

그런데 이 instruction이 받는 argument는 variable A가 있는 address, 즉 address type이다. 이런 상황이라면 memory를 읽어오는 작업을 한 cycle이 아닌 여러 cycle로 구현하는 것이 더 효율적이다.

> 한 cycle로 구성하게 되면, memory를 취하지 않고 간단한 register를 상대로 계산하는 ALU instruction들은 시간을 낭비하게 된다. 결국 이로 인해 성능이 저하된다.

이것이 복잡한 CISC instruction을 RISC 형식으로 나누는 이유다. 작업들을 나눠서 간단한 instruction은 빠르게 완료시키는 방식으로 latency와 throughput을 향상시키는 것이다.

예를 들어 위 예제인 add dword ptr [A], 0x07은 decoder에 의해 다음과 같이 RISC 형식으로 (uop 단위로) 쪼개진다. 그리고 각 작업은 한 cycle씩 차지하며 실행된다.

1. load temp_reg, dword ptr[A]

2. add temp_reg, temp_reg, 7

3. store dword ptr[A], temp_reg

> 지금까지 살핀 Instruction Fetch와 Instruction Decoding은, 실제 계산 장치가 처리할 instruction flow를 만들기 때문에 **front end**라고 부르기도 한다.

> 다만 몇몇 초저전력 processor는 이처럼 uop로 쪼개지 않고 CISC instruction을 바로 처리하는 전략을 취하기도 한다.(하나의 uop로 처리)

---

## 6.3 Operand(s) Fetch

instruction decoding까지 마쳤으므로, 이제 operand를 읽어야 한다. operand는 addressing mode에 따라 register, immediate(constant), memory address로 translation된다.

1. ld r0, [sp + 8]

   - memory address 계산과 memory 내용을 읽는다. 

2. add r1, r0, 10

   - register와 immediate를 읽는다.

3. st [sp + 4], r1

   - memory address 계산과 register를 읽는다.

4. jz r1, 100

   - (jump if zero) register와 immediate를 읽는다.

그렇다면 operand의 종류에 따라 어떻게 값을 읽어야 할까?

- immediate: ID 과정에서 읽은 내용을 바로 쓸 수 있다.

- register: register file에서 읽어오면 된다.(입출력이 지원되는 array처럼 생각하라.) 하지만 한 cycle만에 원하는 data를 읽어올 수 있는지, 또한 동시에 몇 개의 load/store가 지원되는지 여부 등을 살펴야 한다.

  > 참고로 register는 그 목적에 따라 GPR(General Purpose Register. 범용 목적 레지스터), floating point register, control register, debug register, PC가 있다.

- address: 위 예제의 1번 instruction은 stack pointer, sp에서 8만큼 떨어진 address에 저장된 값을 읽는다. 이 작업에서 수행되는 일은 다음과 같다.

   - 가져올 memory의 address를 구한다. 이 address를 주로 **effective address**(유효 주소)라고 부른다.

   - 저장된 value를 읽는다.

   - destination register에 저장한다.(operand fetch 단계에서 수행하지는 않는다.)

   > 대다수 processor는 address 계산에 특화된 **AGU**(Address Generation Unit)을 가지고 있다. AGU에서 effective address를 구해서 memory에 data를 읽어오도록 요청하게 된다.

   > 마찬가지로 data가 저장된 address도 virtual address이므로, DTLB(data address를 관리하는 TLB)에서 translation한 뒤, L1 data cache부터 DRAM까지 memory hierarchy를 거쳐 data를 가져오게 된다.

operand를 읽는 과정은 이와 같았다. memory store와 branch일 때는 다음과 같다.

- memory store

  1. memory에 쓸 source register의 내용을 읽는다.(이 단계가 operands fetch에 해당한다. 2번 3번은 operand store 단계에서 수행된다.)

  2. effective address를 구한다.

  3. memory access
   
- branch: ALU 연산처럼 보통 비교할 register나 immediate를 읽는다.

---

## 6.4 Instruction Execution

지금까지 instruction fetch, decoding, operands fetch 단계까지 완료되었다. 이제 지시에 따라 실제 계산을 Instruction Execution 단계(**EX** 또는 **EXE**)에서 수행한다. 

계산은 instruction 종류에 따라 다소 차이가 있는데, 앞서 본 add r1, r0, 10와 같은 경우, 지정한 연산 타입에 따라 계산만 수행하면 된다.

branch의 경우 ISA마다 형태가 달라 약간씩 다르게 처리된다.

x86은 보통 비교를 담당하는 연산으로 test와 cmp를 제공한다. 이 연산은 일반 산술 연산처럼 처리되고 그 결과를 **flag**(플래그) register가 받는다. 그리고 이 flag 내용을 기반으로 branch instruction이 분기 여부를 결정한다.

> memory load/store는 이 단계에서 수행되지는 않으나, 이 단계 자체가 어디까지나 가상의 분류에 해당한다는 점을 유념하자.

> 그러나 실제로는 processor가 갖는 ALU 자원은 한정되어 있기 때문에, 리소스 할당 문제를 추가로 고려해야 한다.

---

## 6.5 Operand Store

마지막으로 연산으로 구한 결과를 register에 써야 한다. 이를 **Operand Storing**(OS 혹은 Write Back, WB)라고 한다.

먼저 ALU 연산이라면 결과를 instruction에 지시된 destination register에 쓴다.

memory load도 마찬가지로 지시된 destination register에 써야 한다.

memory store는 effective address를 계산하고, 앞서(operand fetch 단계에서) 읽은 register 내용을 memory hierarchy로 전송한다.

branch는 직접적으로 연산을 저장할 register를 가지지는 않으나, 그 결과로 PC 값을 바꾸기도 한다. 앞서 Instruction Fetch 단계에서 PC를 update했지만, 직관적으로 본다면 이 과정에서 PC를 update하는 것이 더 합리적이다.

---

## 6.6 예외 처리

instruction을 처리하는 processor는 마땅히 여러 **exception**(예외)을 만나게 된다. 예를 들어 읽을 권한이 없는 엉뚱한 memory 영역에 읽기를 시도하는 경우가 있다.(segmentation fault) 

processor는 보통 exception과 interrupt로 구분해서 예외적인 상황을 외부로 알린다. 여기서 exception은 다시 **trap**, **fault**, **abort**로 나눌 수 있다.(정의는 processor마다 다를 수 있다.)

> interrupt는 hardware/software interrupt로 나뉜다. hardware interrupt는 instruction 처리와 상관이 없는 asynchoronous(비동기적인) 사건을 가리킨다. 예를 들어 키보드를 누르면 신호가 hardware intterrupt로 전달된다. 그러면 processor는 현재 실행 중인 instruction flow를 잠시 멈추고, interrupt를 처리하는 function routine으로 context를 전환한다. 반면 software interrupt는 instruction으로 발생한다.

> Interrupt를 처리하는 routine을 **Interrupt Service Routine**, 또는 **interrupt vector**라고 한다. interrupt 처리가 끝나면 원래 program으로 돌아가서 실행이 재개된다.

exception은 program 실행 중 synchronous(동기적)으로 발생하는 일이다. 대표적으로 division by zero(0으로 나누기)와 같은 경우가 해당된다. processor는 context를 전환해 이 exception을 처리하는 handler를 실행시킨다.

또한 가장 대표적인 exception으로 **page fault**가 있다. 이는 processor가 instruction cache에서 instruction을 읽었지만, 이 page가 아직 page table에 없는 상황에 해당한다. processor는 exception을 발생시키고 page fault를 처리하게 된다.

> 이외에도 데이터 정렬이 어긋날 때, 알 수 없는 opcode일 때, 메모리 접근 권한이 없을 때 등 exception이 발생한다.

앞서 trap, fault, abort로 나눌 수 있다고 언급했는데, 이들은 기본적인 개념은 동일하고 형태만 약간 다르다. 아래는 intel IA-32 구조에서의 정의다.

- trap: exception이 일어난 instruction의 '다음부터' 실행된다.(breakpoint trap)

- fault: exception이 일어난 instruction부터 다시 실행된다.

- abort: exception이 일어난 이유를 알려주고, 다시 program을 실행시키는 것도 허용하지 않는다.

   > 이런 이유 때문에 대체로 abort는 심각한 hardware problem일 때 발생한다.

exception이 일어난 위치를 정확하게 알려주는 것, 즉 **precise exception**은 processor와 OS 설계에서 매우 중요한 문제다. 만약 page fault가 발생했는데, 어느 instruction이나 data address가 문제를 일으켰는지 알 수 없다면 문제를 해결할 수 없을 것이다. 

> 그럼에도 processor 구조에 따라 precise exception이 어려운 때가 있다. 특히, asynchoronous processor에서 이를 위해 고민해야 할 부분이 굉장히 많다.(아예 virtual memory를 사용하지 않는 방법을 적용하기도 한다.)

아래 code는 division by zero에 따른 exception 처리를 보여준다.

```c
void OnInterruptOrException(int code, ...) {
    SaveContext(...);    // 현재 Context를 저장
    pc_ = interrupt_vector_[code];    // 해당 interrupt를 처리하는 routine으로 간다.
}

// 실행 단계 중
if (inst->IsDivision() ** operand_B_ == 0) {
    raise_exception(0);    // argument의 0은 divided by zero를 의미한다.
}
```

---
