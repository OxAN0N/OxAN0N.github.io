---
title : "Double Free Bug"
excerpt : "Double Fee Bug와 보호 기법의 우회 원리"
categories :
  - pwnable
tags:
  - Heap
  - Double Free Bug
  - pwnable
date:               2023-01-21 00:00:0 +0000
last_modified_at:   2023-01-21 00:00:0 +0000
---

## 1. 개요

tcache와 bins를 free list라고 통칭한다면, free list의 관점에서 free는 청크를 추가하는 함수, malloc은 청크를 꺼내는 함수이다. 그러므로, 임의의 청크에 대해 free를 두 번이상 적용할 수 있다는 것은, 청크를 free list에 여러 번 추가할 수 있음을 의미한다.

free list에 chunk가 중복되어 존재하는 것을 duplicated라고 표현하다. 이러한 duplicated free list를 이용하면 임의 주소에 청크를 할당할 수 있다. 이렇게 할당한 청크의 값을 읽거나 조작함으로써 임의 주소 읽기 또는 쓰기를 할 수 있다. 

이처럼 chunk를 중복해서 해제할 수 있는 코드는 보안상 약점으로 분류되어 Double Free Bug라고 불린다. 

## 2. Double Free Bug

Double Free Bug (DFB)는 같은 청크를 두 번 해제할 수 있는 버그를 말한다. ptmalloc2에서 발생하는 버그 중 하나이며, 공격자에게 임의 주소 쓰기, 임의 주소 읽기, 임의 코드 실행, 서비스 거부 등의 수단으로 활용될 수 있다. 

유효하지 않은 메모리 영역을 가리키는 포인터를 Dangling Pointer라고 한다.  Dangling Pointer는 ptmalloc의 free함수가 chunk를 ptmalloc에 반환하기만 할 뿐, chunk의 주소를 담고 있던 포인터를 초기화하지 않아서 발생할 수 있다. 
이러한 Dangling Pointer는 Double Free Bug를 유발하는 대표적인 원인이다. 

Double Free Bug를 이용하면 duplicated free list를 만드는 것이 가능하다. 

free list의 각 chunk들은 fd와 bk로 연결된다. fd는 자신 이후에 해제된 chunk를, bk는 자신 이전에 해제된 chunk를 가리키는데, 이전 [포스팅][1]에서 확인할 수 있듯이, freed chunk에서 fd와 bk 값을 저장하는 공간은 Allocated chunk에서 data를 저장하는데 사용된다. 

따라서 만약 어떤 chunk가 free list에 중복해서 포함된다면, 첫번째 재할당에서 fd와 bk를 조작해서 free list에 임의의 주소를 포함시킬 수 있다.

ptmalloc이 도입된 초기에는 Double Free에 대한 보안 검사가 미흡해 Double Free Bug가 존재 할 시 손쉽게 트리거 시킬 수 있었으나, 최근에는 관련된 보호 기법이 gblic에 구현되어 이를 별도로 우회하지 않고 같은 chunk를 두번 해제시 즉시 프로세스가 종료되게 되었다.

## 3. 정적 패치 분석

이전 [포스팅][2]에서 살펴본 구조체와 함수 등에 보호 기법을 위해 어떤 코드 패치가 적용되었는지 다음 내용에서 서술하였다.

### 3.1 tcache_entry
```c
typedef struct tcache_entry {
  struct tcache_entry *next;
+ /* This field exists to detect double frees.  */
+ struct tcache_perthread_struct *key;
} tcache_entry;
```
double free를 탐지하기 위해서 tcache_perthread_struct를 가리키는 key 포인터 변수가 tcache_entry에 추가되었다.

실제 freed chunk에서 해당 key 포인터 변수가 가리키는 주소의 메모리 값(tcache_perthread_struct 변수)을 조회하면, 해제한 chunk의 주소가 (tcache_perthread_struct의 필드) entry에 포함되어 있음을 알 수 있다. 이는 `tcache_perthread`에 tcache들이 저장되기 떄문이다. 

### 3.2 tcache_put
```c
tcache_put(mchunkptr chunk, size_t tc_idx) {
  tcache_entry *e = (tcache_entry *)chunk2mem(chunk);
  assert(tc_idx < TCACHE_MAX_BINS);
  
+ /* Mark this chunk as "in the tcache" so the test in _int_free will detect a
       double free.  */
+ e->key = tcache;
  e->next = tcache->entries[tc_idx];
  tcache->entries[tc_idx] = e;
  ++(tcache->counts[tc_idx]);
}
```

해제한 chunk를 tcache에 추가하는 함수인 tcache_put에 chunk의 key에 tcache라는 값을 대입하는 것이 추가되었다. 

이때 tcache는 tcahce_perthread라는 구조체 변수를 가리킨다.

### 3.3 tcache_get
```c
tcache_get (size_t tc_idx){
   assert (tcache->entries[tc_idx] > 0);
   tcache->entries[tc_idx] = e->next;
   --(tcache->counts[tc_idx]);
+  e->key = NULL;
   return (void *) e;
 }
 ```

 tcache에 연결된 chunk를 재사용할 때 사용하는 함수인 tcache_get에 재사용하는 chunk의 key값에 NULL을 대입하는 코드가 추가되었다.

### 3.4 _int_free

 ```c
 _int_free (mstate av, mchunkptr p, int have_lock)
 #if USE_TCACHE
   {
     size_t tc_idx = csize2tidx (size);
-
-    if (tcache
-       && tc_idx < mp_.tcache_bins
-       && tcache->counts[tc_idx] < mp_.tcache_count)
+    if (tcache != NULL && tc_idx < mp_.tcache_bins)
       {
-       tcache_put (p, tc_idx);
-       return;
+       /* Check to see if it's already in the tcache.  */
+       tcache_entry *e = (tcache_entry *) chunk2mem (p);
+
+       /* This test succeeds on double free.  However, we don't 100%
+          trust it (it also matches random payload data at a 1 in
+          2^<size_t> chance), so verify it's not an unlikely
+          coincidence before aborting.  */
+       if (__glibc_unlikely (e->key == tcache))
+         {
+           tcache_entry *tmp;
+           LIBC_PROBE (memory_tcache_double_free, 2, e, tc_idx);
+           for (tmp = tcache->entries[tc_idx];
+                tmp;
+                tmp = tmp->next)
+             if (tmp == e)
+               malloc_printerr ("free(): double free detected in tcache 2");
+           /* If we get here, it was a coincidence.  We've wasted a
+              few cycles, but don't abort.  */
+         }
+
+       if (tcache->counts[tc_idx] < mp_.tcache_count)
+         {
+           tcache_put (p, tc_idx);
+           return;
+         }
       }
   }
 #endif
 ```

> if (__glibc_unlikely (e->key == tcache)) 

위의 코드를 통해서 재할당하려는 chunk의 key값이 tcache이면 Double Free가 발생했다는 것을 감지하여, 프로그램을 abort 시킨다.
(따라서 해제된 chunk의 key 값을 1비트만 바꾸어 위의 코드만 우회한다면, DFB를 발생시킬 수 있다.)

## 4. 보호 기법 우회

```c
#include <stdio.h>
#include <stdlib.h>
int main() {
  void *chunk = malloc(0x20);
  printf("Chunk to be double-freed: %p\n", chunk);
  free(chunk);
  
  *(char *)(chunk + 8) = 0xff;  // manipulate chunk->key
  free(chunk);                  // free chunk in twice
  
  printf("First allocation: %p\n", malloc(0x20));
  printf("Second allocation: %p\n", malloc(0x20));
  return 0;
}
```

위의 코드를 통해서 Double Free Bug를 방지하는 보호 기법을 우회하는 기본적인 방법을 확인 할 수 있다.
처음 할당한 chunk를 free한 후에, dangling pointer를 통해서 해제된 chunk의 key값을 변경한다.
그 후에 chunk를 다시 free하면, Double Free Bug가 발생한다.

```c
  printf("First allocation: %p\n", malloc(0x20));
  printf("Second allocation: %p\n", malloc(0x20));
```

이후 해당 코드를 통해서 할당되는 chunk를 확인하면, tcache에 중복 연결된 chunk가 연속으로 재할당 되는 것을 확인할 수 있다.
이처럼 tcache에 같은 chunk가 두 번 연결되는 것을 Tcache Duplication이라고 하며, 이는 double free bug를 통해서 발생시킬 수 있다.

## 출처 & 참고
[출처](https://dreamhack.io)

[1]: ./2023-01-14-Heap-chunk.md
[2]: ./2023-01-17-Heap-tcache.md