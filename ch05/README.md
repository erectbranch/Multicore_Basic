# 5 dependence

올바른 parallel program은 순서를 지켜가며 실행된다. 이 순서를 결정짓는 요소가 바로 **dependence**(의존성)이다.

---

## 5.1 data dependence

**data dependence**(데이터 의존성)이란 instruction들 사이의 data flow를 의미한다. 

> 특히 compiler에서 optimization을 수행할 때, dependence 파악이 가장 중요한 작업 중 하나다.(compiler나 processor는 optimization 과정에서 instruction 순서를 뒤바꾸거나 아예 구문 자체를 없애버리기도 한다.)

두 instruction 사이에서 일어나는 data dependence는 read/write 순서에 따라 RAW, WAW, WAR라는 dependence로 구분할 수 있다.

> 간혹 computer architecture에 한정지어 이야기할 때는, 이런 dependence를 **hazard**(해저드)라고 표현하기도 한다.

1. **RAW**(Read-After-Write) dependence

아래 예제에서 variable x를 잘 보자.

```c
x = y + 1;    // 1번 계산
z = x * 2;    // 2번 계산
```

variable x는 1번 instruction이 계산 결과를 만들면, 2번 instruction이 이를 사용한다. 

다시 말해 2번 instruction은 절대 1번 instruction보다 앞서 계산될 수 없다. 이런 dependence가 바로 RAW이다.

> compiler에서는 RAW dependence를 flow dependence(흐름 의존성)이라고도 부른다.

2. **WAR**(Write-After-Read) dependence

```c
z = x * 2;    // 1번 계산
x = y + 1;    // 2번 계산
```

이번에는 variable x가 1번 계산으로 먼저 읽힌 뒤, 2번 계산에서 값이 쓰여진다. 이런 dependence가 바로 WAR dependence이다.

> compiler에서는 antidependence(반의존성)이라고도 부른다.

3. **WAW**(Write-After-Write) dependence

```c
x = z * 2;    // 1번 계산
x = y + 1;    // 2번 계산
```

이번에는 같은 대상에 값이 여러 번 쓰였다. 이런 dependence를 바로 WAW dependence라고 한다. 다른 말로 output dependence(출력 의존성)이라고 부른다.

<br/>

> 이 중에서도 RAW dependence를 **true dependence**(진짜 의존성), WAR, WAW를 **false dependence**(가짜 의존성)으로 분류한다.

이 세 가지 data dependence 중 하나라도 있으면, 순서대로 실행을 해야만 정확한 결과를 얻을 수 있다. 하지만 사실 WAR와 WAW는 성격이 많이 다른 RAW와 다르게, 어느 정도 속임수를 사용해 없앨 수 있다.

- WAR 제거하기

다음과 같은 WAR dependence가 있는 code를 보자.

```c
z = x * 2;    // 1번 instruction
x = y + 1;    // 2번 instruction
a = x / 2;    // 3번 instruction
```

아래와 같이 variable x의 이름을 x1으로 바꾸면, 1번과 2번 사이에 어떤 dependence도 존재하지 않게 된다.

```c
z = x * 2;
x1 = y + 1;
a = x1 / 2;
```

하지만 2번과 3번 사이에 존재하는 RAW dependence는 이런 방법으로도 절대 해결하지 못한다.

- WAW 제거하기

다음과 같은 WAW dependence가 있는 code를 보자.

```c
x = z * 2;
x = y + 1;
a = x / 2;
```

WAW도 WAR와 마찬가지로 variable의 이름을 바꾸는 방법으로 해결할 수 있다.

```c
x = z * 2;
x1 = y + 1;
a = x1 / 2;
```

하지만 2번과 3번에 존재하는 RAW dependence는 해결하지 못한다.

> 이처럼 WAR/WAW dependence는 이름 때문에 발생하는 점에서 **name dependence**로 분류하기도 한다.

---

## 5.2 control dependence

program의 의미는 조건 분기문으로도 결정된다. 아래 code를 보자.

```c
if (a == 10) {    // 1번 instruction
    b = 10;       // 2번
} else {          // 3번
    b = 20;       // 4번
}
a = b + 10;       // 5번
z = x / y;        // 6번
k = a + z;        // 7번
```

이 code에서 2번 instruction이 실행될지, 4번 instruction이 실행될지는 1번의 조건문 판단이 끝나야만 알 수 있다. 명시적인 data dependence는 1,2,4번 instruction 사이에 존재하지 않지만, **control flow**(실행 흐름) 때문에 dependence가 만들어졌다. 이런 dependence를 바로 **control dependence**(컨트롤 의존성)이라고 한다.

- 5번은 2, 4번 instruction과 RAW dependence를 가지고 있으므로, 반드시 조건문 판단이 끝나고 수행된다.

- 6번은 variable x, y가 이 코드에서는 나타나지 않았고, 조건문 control flow의 결과와 상관이 없으므로 어떠한 dependence도 없다.

  - 7번 instruction과는 variable z로 인한 RAW dependence는 있지만, 앞선 instruction과는 무관하므로 이들과는 무관하게 수행이 가능하다.

> 참고로 uncondition branch(무조건 분기문)은 control dependence를 만들지 않는다.

또 다른 분기문인 function call 이 자체는 uncondition branch를 하지만, function call이 완료되면 결국 다음 문장으로 이어지므로 겉으로는 control dependence를 만들지 않는 것으로 보인다. 하지만 argument를 받고, 또 내부에 복합한 control flow가 있는 등 쉽게 판단할 수는 없다.

> 통상 하나의 function range를 벗어나 function call을 추적하면서 분석하는 방식을 interprocedural analysis(프로시저간 분석)이라고 한다. 

> 원래 compiler optimization도 아주 간단한 경우 한 function 안에서만 이뤄지지만, 고차원 optimization은 **IPO**(Interprocedural Optimization)을 수행한다.

control dependence는 어떻게 보면 PC(Program Counter) register에 의한 RAW data dependence로도 볼 수 있다. (분기문의 경우 instruction을 실제로 수행해야 최종 PC 값을 쓸 수 있다. 그리고 instruction이 실제로 실행된다면 PC register를 읽게 되기 때문이다.)

---

## 5.3 memory dependence

data dependence에서 예시로 본 code들은 모두 int, float와 같이 pointer가 아닌 type이었다.(기계어 수준으로 이야기하면 모두 register type이다.) 하지만 pointer라면 이야기가 달라진다.

```c
*x = *y + 1;    // 1번
*a = *b + 2;    // 2번
```

pointer를 사용하는 위 code만으로는 1번과 2번 사이에 존재하는 data dependence를 파악할 수 없다. 만약 pointer x, y, a, b가 모두 다른 variable, 즉 다른 memory address를 가리킨다면 dependence도 없을 것이다. 하지만 같은 address를 가리킨다면 이야기는 달라진다.

예시 코드를 기계어로 다시 살펴보자.

```
1: mov [esi+0], eax    ; *(esi+0) = eax
2: mov ebx,  [edi+8]   ; ebx = *(edi+8)
3: mov [ebp-8], ecx    ; *(ebp-8) = ecx
4: add ebx,  1         ; ebx = ebx + 1;
```

- 1번: register eax의 값을 읽고, esi+0이 가리키는 address에 값을 쓴다.

- 3번: register ecx의 값을 읽고, ebp-8이 가리키는 address에 값을 쓴다.

- 2번: edi+8이 가리키는 address의 값을 읽고 ebx에 쓴다.

즉, esi+0, edi+8, ebp-8이 가리키는 value가 같은지 여부를 밝혀내야 한다. 이렇게 memory dependence 여부를 확인하는 과정을 **memory disambiguation**(메모리 명확화)라고 지칭한다.

> compiler에서는 pointer가 어떤 값을 가리키는지 알아내야 하므로 **pointer analysis**(포인터 분석)이라고도 한다.

물론 위 예제를 보면 2번과 4번에서 RAW dependence도 존재한다. 이런 data dependence는 이름을 보는 것만으로 파악이 가능하지만, memory dependence는 load/store 과정을 거치지 않으면 파악이 어려운 것이다.

> 따라서 실제 실행하지 않고서는 dependence를 완전히 파악할 수 없으므로, compiler는 보수적으로 서로 dependence가 있다고 판단할 때가 많다.

위와 같은 dependence들이 없다면 instruction들은 어떤 순서든, 혹은 병렬로 실행되어도 상관이 없다. 이렇게 dependence를 분석한 compiler나 processor는 최적의 instruction 실행 순서를 정한다. 이런 절차를 보통 **instruction reordering**(명령어 재배치) 혹은 **instruction scheduling**(명령어 스케줄링) 최적화라고 부른다.

> memory load 연산은 cache miss를 유발할 수 있는 작업이므로 시간이 오래 걸릴 수 있다. 따라서 memory load를 RAW dependence를 가진 instructions 앞이 아닌, dependence가 없는 instruction 앞에 배치하는 것으로 일종의 **latency hiding**(레이턴시 감추기) 같은 조치를 취할 수 있다.(다만 단일 processor에 해당하는 이야기로, multicore의 경우 이렇게 순서를 바꾸면 예측하기 어려운 사태가 발생할 수 있다.)

---

## 5.4 loop-carried dependence

data dependence는 loop에서 더 특이한 형태로 발생할 수 있다. 이를 **loop-carried dependence**(루프 전이 의존성)이라고 하며, 마찬가지로 RAW, WAR, WAW dependence로 분류된다.

```c
for (int i = 1; i < N; ++i) {    // 1번
  A[i] = A[i - 1] + 1;           // 2번
}
```

위 구문은 다음과 같은 절차로 진행된다.

1. i = 1: A[1] = A[0] + 1;

2. i = 2: A[2] = A[1] + 1;    // RAW dependence

3. i = 3: A[3] = A[2] + 1;    // RAW dependence

만약 2번 instruction이 한 번만 실행됐다면, 이 loop는 어떠한 dependence를 만들지 않을 것이다. 하지만 loop가 여러 번 반복된다면 한눈에 A[1], A[2] 등 RAW dependence가 만들어지는 것을 확인할 수 있다.

따라서 array A는 loop-carried dependence를 갖는다고 말할 수 있다.

> loop-carried dependence과 구분하여, loop 순환과 무관하게 발생하는 dependence를 **loop-independent dependence**(루프 무관 의존성)이라 부른다.

이번에는 다른 예제를 살펴보자.

```c
for (int i = 0; i < N-1; ++i) {    // 1번
  A[i] = A[i + 1] + 1;             // 2번
}
```

위 구문은 다음과 같은 절차로 진행된다.

1. i = 0: A[0] = A[1] + 1;

2. i = 1: A[1] = A[2] + 1;    // WAR dependence

3. i = 2: A[2] = A[3] + 1;    // WAR dependence

array A는 WAR dependence를 loop에 걸쳐서 갖게 된다. 이 또한 data dependence 때처럼 이름 바꾸기로 해결할 수 있다. 그러나 명시적으로 이름을 바꾸지는 않고, 복사본을 바꾸는 방식으로 해결한다.

```c
int oldA[N];
memcpy(oldA, A, sizeof(int)*N);
for (int i = 0; i < N-1; ++i) {
  A[i] = oldA[i + 1] + 1;
}
```

이렇게 복사본으로 바꾼 구문은 다음과 같이 진행된다.

1. i = 0: A[0] = oldA[1] + 1;

2. i = 1: A[1] = oldA[2] + 1;

3. i = 2: A[2] = oldA[3] + 1;

이렇게 oldA라는 임시 array로 loop-carried WAR dependence를 해결했다. 그런데 program에 따라 더 효율적인 방법으로 loop-carried dependence를 해결할 수 있다.

아래는 이해를 위해 조금 억지로 loop-carried WAW dependence 예시를 만들었다.

```c
for (int i = 0; i < N; i++) {    // 1번
  B[0] = A[i] + i;               // 2번
}
```

위 예시에서는 B[0]에 loop-carried WAW dependence가 발생한다. 하지만 아래와 같이 variable을 따로 declaration하면 이런 dependence를 해결할 수 있다.

```c
for (int i = 0; i < N; ++i) {
  tempB[i] = A[i] + 1;
}
B[0] = tempB[N - 1];
```

이렇게 <U>loop-carried dependence를 제거하면 loop를 parallelize할 수 있게 된다.</U> 즉, 각 loop의 iteration을 여러 thread에서 동시에 실행 가능하다. 

> 물론 real dependence에 해당하는 RAW dependence는 알고리즘을 뜯어고치는 수정 없이는 없앨 수 없어서 loop의 parallelize를 막고 만다.

이처럼 false dependence는 프로그래밍 테크닉으로 회피할 수 있으며, compiler는 이런 dependence 분석을 통해 자동으로 parallelize할 수도 있다. **loop fusion**(루프 병합)이나 **loop interchange**(루프 바꾸기) 같은 optimization도 이런 dependence 분석을 반드시 거쳐야 가능하다.

---
