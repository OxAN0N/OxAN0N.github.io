---
title : "[Heap] Heap basic #5"
excerpt : "tcache의 개념과 특징"
categories :
  - Pwnable
tags:
  - Heap
  - Memory
  - Pwnable
date:               2023-01-18 00:00:0 +0000
last_modified_at:   2023-01-18 00:00:0 +0000
---

## 1. tcache 개요
tcache (thread local cache)는 각 쓰레드에 독립적으로 할당되는 캐시 저장소를 지칭한다. 

tcache는 GLibc 버전 2.26에서 도입되어 멀티 쓰레드 환경에 더 최적화된 메모리 관리 메커니즘을 제공한다. 

tcache는 각 thread가 고유하게 갖는 캐시이기 때문에, ptmalloc은 race condition (공유 rsc에 대한 동시 접근에 의해 일어날 수 있는 문제)을 고려하지 않고 접근할 수 있다. 

arena의 bin에 접근하기 전에 이러한 tcache를 먼저 사용하므로 arena에서 발생할 수 있는 병목 현상을 완화하는 효과가 있다.

(thread_arena는 코어 개수 등 각 시스템의 환경에 따라서 그 수에 따라 제한이 있다. 따라서 하나의 arena가 여러 thread를 가질 수 있기 때문에, race condition을 막기 위한 mutex 등의 사용으로 병목 현상이 발생할 수 있다.)

tcache의 경우, 보안 검사가 많이 생략되어 있어서, heap exploit 등에 많이 사용된다. 

## 2. tcache 특징

각각의 thread는 64개의 tcache를 가지고 있다. 

tcache는 fastbin과 마찬가지로 LIFO 방식으로 사용되는 단일 연결리스트 방식을 사용한다. 

리눅스는 각각의 tcache가 보관할 수 있는 chunk의 개수를 7개로 제한하고 있다. (이는 thread 마다 정의되는 tcache의 특성상, 무제한으로 chunk를 연결한다면 메모리가 낭비될 수 있기 때문이다.)
chunk가 보관될 tcache가 가득찼을 경우에는 적절한 bin으로 chunk를 분류한다. 

fastbin과 마찬가지로 **tcache에 들어간 chunk들은 병합되지 않는다.**

tcache에는 32 바이트 (0x20) 이상, 1040 바이트 (0x410) 이하의 크기를 가지는 chunk들이 보관된다.

해당 범위에 속하는 chunk들은 할당 혹은 해제될 때, tcache를 가장 먼저 조회한다.  

각각의 tcache들은 동일한 크기의 chunk들을 보관한다. 

## 3. tcache 구조

![tcache struct][1]

### 3.1 tcache_perthread_struct

```c
typedef struct tcache_perthread_struct
{
  char counts[TCACHE_MAX_BINS];
  tcache_entry *entries[TCACHE_MAX_BINS];
} tcache_perthread_struct;

# define TCACHE_MAX_BINS                64

static __thread tcache_perthread_struct *tcache = NULL;
```

- 전체적인 tcache list를 관리하는 구조체이다.
- tcache는 다른 bin들과는 달리 main_arena에 존재하지 않고, 해당 구조체에 존재하다.
- tcache_entry[tc_idx] 
  각 배열의 인덱스마다 동일한 사이즈 별로 freed chunk들이 단일 연결 리스트 형태로 연결된다. (LIFO 방식으로 재할당이 된다.)
- count[tc_idx]
  tcache_entry[tc_idx]에 연결되어 있는 chunk의 개수를 저장한다.(기본적으로 각각의 단일 연결 리스트 당 최대 7개로 제한이 있다.)


### 3.2 tcache_entry

```c
typedef struct tcache_entry
{
  struct tcache_entry *next;
} tcache_entry;
```

- 동일한 크기의 freed chunk를 관리하는 구조체
- next 포인터를 통해서 다음 동일한 크기의 freed chunk를 연결한다.

cf) fastbin과 tcache의 차이
- tcache는 fd가 fd를 가리키는 반면, fastbin은 prev_size를 가리킨다.
- tcache에서는 prev_size와 prev_inuse bit을 설정하지 않는다. 
- tcache는 tcache_pthread_struct가 관리하고 fastbin은 libc에 존재하는 main_arena가 관리한다.


## 4. tcache 사용 과정

tcache와 관련된 주요 함수
- tacache_init : tcache 초기화 관련 함수
- tcache_put : tcache list에 freed chunk를 삽입하는 함수
- tcache_get : tcache list에서 chunk를 제거하는 함수 (재할당)

### 4.1 malloc의 전체적 흐름
![tcache_malloc][2]  

1. malloc 호출 시, __libc_malloc이 호출된다.
  - 만약 malloc_hook이 존재 시, 이를 먼저 실행
  - 존재하지 않을 시, 다음 로직 실행
2. tcache가 비어있을 시, MAYBE_INIT_TCACHE가 호출
  - MAYBE_INIT_TCACHE 내에서 (tcache가 NULL일 시) tcache_init 함수를 호출 
  - tcach_init 내에서 _int_malloc을 호출해 heap 영역을 할당받아, tcache_pthread_struct를 청크로 할당
3. 현재 요청한 크기에 맞는 chunk가 tcache 내에 존재시, tcache_get을 호출해 재할당
4. tcache에 재할당 해줄 수 있는 chunk가 없을 시, _int_malloc을 호출
5. fastbin, smallbin, unsortedbin 등을 확인하고 재할당 가능한 chunk가 있다면 재할당한다.  
    -  현재 tcache_entry[tc_idx]에 자리가 비여있다면, 빈들에 있는 청크를 꺼내 tcache에 넣을 수 있을 만큼 넣는다.
6. 재할당한 chunk의 주소를 반환한다.


### 4.2 free의 전체적 흐름

![tcache_free][3]

1. 처음에 _libc_free 함수가 호출되면서 free_hook을 검사한다. 있으면 (free_hook이 NULL이 아니면) 호출되고 없으면 넘어간다.
2. tcache가 비어있을 시, MAYBE_INIT_TCACHE가 호출
  - MAYBE_INIT_TCACHE 내에서 (tcache가 NULL일 시) tcache_init 함수를 호출 
  - tcach_init 내에서 _int_malloc을 호출해 heap 영역을 할당받아, tcache_pthread_struct를 청크로 할당
3. _int_free 함수가 호출된다
4. tcache에 넣을 수 있는 chunk이면, tcache_put함수를 호출하여 넣는다.

## 출처 & 참고
[**출처 1**] (https://jeongzero.oopy.io/2c5b7648-5f96-42c4-8366-300e7b5ebac4)
[**출처 2**] (https://dreamhack.io)

[1]: ../../assets/images/pwnable/tcache.jpeg
[2]: ../../assets/images/pwnable/tcache_malloc.png
[3]: ../../assets/images/pwnable/tcahce_free.png