<div width="100%" height="100%" align="center">
  
<h1 align="center">
  <p align="center">Blog2Book, 프로그래머가 몰랐던 멀티코어 CPU 이야기</p>
  <a href="https://m.hanbit.co.kr/store/books/book_view.html?p_code=B2165426861">
    <img width="50%" src="cover.jpg" />
  </a>
</h1>
  

<b>김민장 저</b></br>
한빛미디어 · 2010년 05월 20일 출시</b> 

</div>

## :bulb: 목표

- **CPU의 다양한 동작을 심화하여 이해한다.**

  > out-of-order execution(비순차 실행), ILP(명령어 수준 병렬성) 등을 공부하기

- **멀티코어 프로세서의 동작 원리와 알고리즘 이해하기**

  > multicore processor 동작이 어떻게 다른지, 또한 threading, pararllel programming 등을 공부한다.

</br>

## 🚩 정리한 문서 목록

### 📔 processor 기초

 - [ISA와 architectural state](https://github.com/erectbranch/Multicore_Basic/tree/master/ch01)

   > computer architectural state, ISA

   > CISC와 RISC: instruction의 구조, 설계 차이(stack memory 조작, addressing mode, array 접근 방식), compile로 보는 차이

 - [Amdahl's Law와 processor 성능 향상](https://github.com/erectbranch/Multicore_Basic/tree/master/ch04)

   > Amdahl's Law, 병렬 처리에서의 Amdahl's Law, latency, throughput, processor 성능 향상을 위한 접근 방법

 - [dependence](https://github.com/erectbranch/Multicore_Basic/tree/master/ch05)

   > data dependence(RAW/WAR/WAW), control dependence, memory dependence, loop-carried dependence

 - [processor의 기본 동작](https://github.com/erectbranch/Multicore_Basic/tree/master/ch06)

   > (가상의) instruction 처리 단계: Instruction Fetch(IF), Instruction Decoding(ID), Operand Fetch(OF), Instruction Execution(EXE), Operand Store(OS, WB)

   > exception(trap, falut, abort)과 interrupt

 - [cache](https://github.com/erectbranch/Multicore_Basic/tree/master/ch12)

   > temporal/spatial locality, cache hit, cache miss(cold/conflict/capacity/coherence), miss penalty, hit latency, software cache, cache line, cache block

   > cache lookup policy, cache placement policy(direct-mapped/set-associative/fully-associative cache), cache replacement policy(LRU/LFU/FIFO/pseudo LRU), cache write policy(write-through/write-back)

   > cache inclusion policy(L1/L2/L3 hierarchy), victim cache, private/shared cache, cache coherence(MSI/MESI/MOESI protocol)

### 🗜 pipeline

 - [Instruction Pipeline](https://github.com/erectbranch/Multicore_Basic/tree/master/ch07)

   > pipeline stall, 이상적인 pipeline 조건, 다섯 단계(IF/ID/OF/EXE/OS)으로 구성한 instruction pipeline, fragmentation 줄이기

   > structural/control/data hazard, pipeline을 통한 software parallelize

 - [out-of-order processor pipeline](https://github.com/erectbranch/Multicore_Basic/tree/master/ch08)

   > Instruction Level Parallelism, critical path, superscalar 구조, Tomasulo algorithm, Instruction Window, Register Renaming, Reservation Station, Re-Order Buffer

 - [Simultaneous Multi-Threading(SMT)](https://github.com/erectbranch/Multicore_Basic/tree/master/ch09)

   > thread context, context switching, TLP(Thread-Level Parallelism)

   > Temporal Multi-Threading(fine-grained/coarse-grained threading), Simultaneous Multi-Threading(SMT)

<br/>

## :mag: 목차

### Story 01. 프로그래머가 프로세서도 알아야 해요?
    들어가며 
    고달픈 프로그래머 
    누구를 위해 이 책을 썼는가? 
    참고문헌 
 
### Story 02. 프로세서의 언어 : 명령어 집합 구조
    들어가며
    프로그래머가 보는 프로세서
    프로세서의 언어 : 명령어 집합 구조 
    RISC와 CISC로 알아보는 명령어 집합 구조
    간단한 코드로 보는 RISC와 CISC의 차이
    아직도 RISC vs. CISC ?
    결론
    참고문헌 
 
### Story 03. 프로세서의 기본 부품과 개념들
    들어가며 
    마이크로아키텍처란? 
    산술 논리 장치 : 프로세서 속의 계산기 
    클록, 1사이클이 가지는 의미
    메모리 계층
    컨트롤 장치
    프로세스와 스레드
    가상 메모리 
    결론
    참고문헌 
 
### Story 04. 암달의 법칙과 프로세서의 성능 지표
    들어가며 
    암달의 법칙 
    병렬 처리에서의 암달의 법칙 
    프로그램의 수행 시간 
    성능 향상을 위해 해야 할 일 
    결론 
    참고문헌 
 
### Story 05. 프로그램의 의미를 결정 짓는 의존성
    들어가며 
    데이터 의존성 
    컨트롤 의존성 
    메모리 의존성 
    루프에서의 데이터 의존성 
    결론 
 
### Story 06. 프로세서 기본 동작
    들어가며 
    명령어 처리의 기본적인 다섯 단계 
    명령어 인출 
    명령어 해독 
    피연산자 인출 
    명령어 실행 단계 
    연산 결과 저장 
    예외 처리 
    결론 
    참고문헌 
 
### Story 07. 고성능 프로세서의 시작 : 명령어 파이프라인
    들어가며 
    파이프라인의 기본 개념 
    파이프라인의 효율적인 설계 
    파이프라인 프로세서의 구현 
    파이프라인 해저드 
    파이프라인 : 소프트웨어 병렬화의 한 가지 방법 
    결론 
    참고문헌 
 
### Story 08. 또 하나의 혁명 : 비순차 실행
    들어가며 
    비순차 슈퍼스칼라 프로세서가 필요한 이유 
    비순차 실행의 원리 : 명령어 수준 병렬성 
    슈퍼스칼라 파이프라인 구조 
    비순차 실행의 구현 : 토마슐로 알고리즘 
    비순차 프로세서 파이프라인 
    결론 
    참고문헌 
 
### Story 09. 하이퍼스레딩 : 병렬성의 극대화
    들어가며 
    하이퍼스레딩이 뭐야? 
    동시 멀티스레딩의 구현과 성능 
    결론 
    참고문헌 
 
### Story 10. 멀티코어 혁명 : 칩 멀티프로세서
    들어가며 
    멀티코어 시대 
    싱글코어의 한계 : 에너지 장벽 
    싱글코어의 한계 : ILP의 한계 
    병렬 컴퓨터의 개념 
    병렬 컴퓨터 구조 
    멀티코어의 구성 방식 
    멀티코어의 한계 : 메모리 장벽과 병렬 프로그래밍 
    여전히 중요한 싱글코어 성능 
    결론 
    참고문헌 
 
### Story 11. 데이터 병렬성 : SIMD와 GPU
    들어가며 
    데이터 병렬성 
    GPU : 또 하나의 병렬 프로세서 
    CUDA 프로그래밍 모델 : 스레드와 메모리 모델 
    CUDA 프로그래밍의 예 : 행렬 곱셈 
    nVidia GPU의 자세한 스레드 실행 구조 : 워프(Warp) 
    결론 
    참고문헌 
 
### Story 12. 고성능 프로세서의 필수 조건 : 똑똑한 캐시
    들어가며 
    왜 캐시가 필요하고 잘 작동할 수 있을까? 
    일반적인 캐시 구조 
    CPU 캐시의 기본적인 설계 
    고성능 캐시를 위한 알고리즘 
    멀티코어에서의 캐시
    결론 
    참고문헌 
 
### Story 13. if 문은 그냥 실행되는 것이 아니다
    들어가며 
    분기문 명령어와 프로그래밍 언어 
    분기 예측이 필요한 이유 
    분기 예측에 기반한 투기적 실행 
    기본적인 분기 예측 방법 
    더 똑똑한 과거 기반의 미래 예측 
    히스토리를 이용한 분기 예측
    프리디케이션
    결론 
    참고문헌
 
### Story 14. 가상 함수에 담긴 복잡함
    들어가며 
    분기 목적지 예측 
    간접 분기문의 분기 목적지 예측 
    결론
    참고문헌 
 
### Story 15. 효율적인 메모리 명령 실행 알고리즘
    들어가며 
    효율적인 메모리 연산의 실행
    컴파일러 최적화의 장애물 : 포인터
    결론 
    참고문헌 
 
### Story 16. 메모리 레이턴시 감추기 : 프리펫처
    들어가며
    필요한 데이터를 미리 잘 가져오자 
    기본적인 소프트웨어 프리펫칭 
    포인터 기반 자료구조의 소프트웨어 프리펫칭 
    하드웨어 프리펫칭 알고리즘 
    결론 
    참고문헌 
 
### Story 17. VLIW로 살펴보는 두 변수 교환 방법
    들어가며 
    VLIW의 철학 
    두 변수를 교환하는 방법에 대한 고찰 
    결론 
    참고문헌
 
### Story 18. 프로그래머의 새로운 과제 : 병렬 프로그래밍
    들어가며 
    병렬 프로그래밍은 선택이 아니라 필수 
    기본 개념 : 원자적 실행과 동기화 연산 
    멘델브로 집합으로 보는 병렬 프로그래밍 
    결론 
    참고문헌 
 
### Story 19. 골치 아픈 멀티스레드 버그 : 하이젠버그
    들어가며 
    재현이 어려운 골치 아픈 버그 
    대표적인 병행성 버그 : 원자성 위반과 순서 위반 
    결론 
    참고문헌 
 
### Story 20. 어려운 병렬 프로그래밍, 그리고 그 미래는?
    들어가며 
    비효율적인 병렬 프로그래밍 : 가짜 공유 문제 
    미래의 병렬 프로그래밍 방법론 
    결론 
    참고문헌 