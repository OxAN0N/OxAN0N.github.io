---
title : "[Heap] Heap basic #2"
excerpt : "Arena의 개념"
categories :
  - Pwnable
tags:
  - Heap
  - Memory
  - Pwnable
last_modified_at : 2023-01-14
---
# 1. Arena
Multithreaded 환경에서 각각의 thread가 서로 간섭하지 않고 서로 다른 메모리 영역에 access 할 수 있도록 도와주는 heap 영역을 Arena라고 한다. 

모든 thread마다 각각의 Arena를 할당한다면, 메모리 자원이 고갈되기 쉬우므로 시스템의 core 개수에 따라서 Arena 개수가 제한되어 있다. 

```c
[glibc 2.23 malloc.c line 1176]
#define NARENAS_FROM_NCORES(n) ((n) * (sizeof (long) == 4 ? 2 : 8))

// 32bit system인 경우 long타입 크기가 4bytes이므로 (core 갯수 * 4)만큼 arena를 가짐
// 64bit system인 경우 long타입 크기가 8bytes이므로 (core 갯수 * 8)만큼 arena를 가짐
```
제한된 크기만큼 Arena가 이미 존재해서 더 이상 늘릴 수 없는 경우에는, 여러 thread가 하나의 Arena를 공유해야 한다.
(이 경우 병목 현상이 발생할 수 있어, 이후 tcache라는 개념이 도입되었다.)

따라서, 각각의 Arena 안에는 여러 개의 thread가 존재할 수 있으며, 이 경우 mutex를 이용해서 접근을 제어한다.

- Arena는 multithreaded 환경을 지원하기 위해 도입된 개념 
- multithreaded process의 경우, 하나의 이상의 Arena를 가짐
- 서로 다른 Arena는 서로 간섭을 받지 않고 Heap에 관련된 작업 (동적 할당)을 수행할 수 있음
- 자원 고갈 방지를 위해 시스템 환경에 따라 Arena 개수가 제한됨.
- 하나의 Arena에 여러 개의 thread가 존재할 수 있으며, mutex를 통해 collision을 방지함 
<br>
<br>
<br> 

# 2. Arena의 종류
## 2.1 main_arena
main thread에서 생성된 arena이다. 단일 thread 환경인 경우, main_arena만이 사용된다. 별도로 heap을 생성하지 않아도 기본적으로 선언되어 있으며, 기본적으로 132KB의 initial heap을 가진다. (프로그램이 실행되면서 최초로 malloc(1000)만을 요청해도 132KB의 heap 영역이 매핑됨)

main arena가 감당할 수 있을 만큼의 크기에 대한 동적 할당 요청이 들어오면 sbrk()를 통해 heap segmenent를 확장한다. (따라서 main arena는 start_brk와 brk 사이 공간에 존재한다.) 너무 큰 크기의 동적 할당 요청이 들어오면 mmap()을 통해서 새로운 heap 메모리를 할당한다.

main_arena의 경우, 하나의 heap segment만을 가지게 되며, 이 때문에 heap_info 구조체를 가지지 않는다.

하나의 heap은 여러 개의 chunk로 나누어지며, 각 chunk는 각각의 header를 갖는다.

## 2.2 thread_arena
새로운 thread가 생성되어 힙 작업을 수행하고자 할 때, 다른 thread를 기다리는 것을 줄이기 위해 새로운 Arena를 생성한다. 이러한 arena를 thread_arena라고 한다. 

sbrk()를 통해서 할당되는 main_arena와는 달리 mmap()을 통해서 새로운 heap 메모리에 할당되며, mprotect()를 사용해 확장된다. 

main_arena와는 달리 여러 개의 sub heap과 heap_info 구조체를 가질 수 있다.
<br>
<br>
<br> 


# 3. Arena 구조
![Arena](https://core-analyzer.sourceforge.net/index_files/image702.png)

## 3.1 main_arena

```c
static struct malloc_state main_arena =
{
  .mutex = _LIBC_LOCK_INITIALIZER,
  .next = &main_arena,
  .attached_threads = 1
};
```

## 3.2 heap_info(Heap Header)
```c
typedef struct _heap_info
{
  mstate ar_ptr; /* 현재 heap을 담당하고 있는 Arena */
  struct _heap_info *prev; /* 이전 heap 영역 */
  size_t size;  /* 현재 size(bytes) */
  size_t mprotect_size; /* Size in bytes that has been mprotected
                           PROT_READ|PROT_WRITE.  */
  /* Make sure the following data is properly aligned, particularly
     that sizeof (heap_info) + 2 * SIZE_SZ is a multiple of
     MALLOC_ALIGNMENT. */
  char pad[-6 * SIZE_SZ & MALLOC_ALIGN_MASK];/* 메모리 정렬 */
} heap_info;
```
thread_arena는 각 쓰레드들에 대한 힙 영역이기 때문에 힙 영역의 공간이 부족하면 새로운 영역에 추가로 할당받기 때문에(mmap사용) 여러 개의 힙 영역을 가질 수 있다.

(반면, main_arena는 여러 개의 힙을 가질 수 없다. main_arena의 공간이 부족한 경우, sbrk 힙 영역은 메모리가 매핑된 영역까지 확장된다.)

이런 힙 영역은 어떤 arena가 관리하고 있는지, 힙 영역의 크기가 어느 정도인지, 이전에 사용하던 힙 영역의 정보가 어디에 있는지를 저장할 필요가 있다.
이런 정보를 저장하기 위한 구조체가 바로 위 구조체인 heap_info이며, 힙에 대한 정보를 저장하기 때문에 Heap_Header라고도 한다. (main_thread는 확장을 해서 쓰기 때문에 제외)

- thread_arena는 여러 개의 heap을 가질 수 있고 각각의 heap은 각각의 header를 가짐
- 힙 영역의 공간이 부족해지는 경우 Arena에 새로운 힙(인접하지 않은 영역)을 할당해줘야 하기 때문에 여러 개의 힙이 필요
- 새로운 heap 공간은 기존의 heap 공간과 연속적으로 생성되지 않고 새로운 공간에 생성되며 arena에 mapping됨


## 3.3 malloc_state (Arena Header)
```c
struct malloc_state
{
  /* Serialize access.  */
  mutex_t mutex;
  /* Flags (formerly in max_fast).  */
  int flags;
  /* Fastbins */
  mfastbinptr fastbinsY[NFASTBINS];
  /* Base of the topmost chunk -- not otherwise kept in a bin */
  mchunkptr top;
  /* The remainder from the most recent split of a small request */
  mchunkptr last_remainder;
  /* Normal bins packed as described above */
  mchunkptr bins[NBINS * 2 - 2];
  /* Bitmap of bins */
  unsigned int binmap[BINMAPSIZE];
  /* Linked list */
  struct malloc_state *next;
  /* Linked list for free arenas.  Access to this field is serialized
     by free_list_lock in arena.c.  */
  struct malloc_state *next_free;
  /* Number of threads attached to this arena.  0 if the arena is on
     the free list.  Access to this field is serialized by
     free_list_lock in arena.c.  */
  INTERNAL_SIZE_T attached_threads;
  /* Memory allocated from the system in this arena.  */
  INTERNAL_SIZE_T system_mem;
  INTERNAL_SIZE_T max_system_mem;
};
```
[glibc 2.23 malloc.c / line 1686 - 1724]

arena는 힙 영역에 대한 정보 중에서도 어떤 부분을 사용하면 되는지를 관리하기 때문에 해당 정보를 알고 있어야 한다. malloc_state 구조체는 각 arena에 하나씩 주어지고, 해제된 chunk를 관리하는 연결리스트 bin과 최상위 Chunk인 Top Chunk와 같은 arena에 대한 정보를 저장한다. 이를 arena_header라고도 부른다

thread_arena와 달리, main_arena의 arena header는 sbrk 힙 영역의 일부가 아니다. main_arena는 전역변수이며, libc.so의 데이터 영역에서 찾을 수 있다.

- bin, top chunk, last remainder chunk 등에 대한 정보를 가짐
- (thread_arena가 여러 개의 heap을 가질 수 있어도) 모든 Arena는 단 하나의 Areana Header를 가짐

## 3.4 malloc_chunk (Chunk Header)
```c
struct malloc_chunk {
 
  INTERNAL_SIZE_T      prev_size;  /* Size of previous chunk (if free).  */
  INTERNAL_SIZE_T      size;       /* Size in bytes, including overhead. */
 
  struct malloc_chunk* fd;         /* double links -- used only if free. */
  struct malloc_chunk* bk;
 
  /* large block에서만 사용하고 해당 bin list의 크기 순서를 나타냄  */
  struct malloc_chunk* fd_nextsize; /* double links -- used only if free. */
  struct malloc_chunk* bk_nextsize;
};
```

- 각각의 Chunk마다 Header를 포함
- 사용자의 요청으로 할당된 힙에서 여러 개의 Chunk로 나뉨
- 각 Chunk들은 double linked list로 구성


힙 영역은 사용자에 의해 할당되거나, 해제되거나 하면 Chunk라는 단위로 관리된다.
1. prev_size : 바로 이전 청크의 크기를 저장
2. size : 현재 청크 크기를 저장
3. fd, bk : malloc시 데이터가 들어가고, free시 fd, bk포인터로 사용
4. fd(bk)_nextsize : large bin을 위해서 사용되는 포인터



## Cf)
**brk()**
- 프로세스의 data segment에 할당 메모리량을 변경하기 위해 사용
- program break 위치를 이동시킴으로써 메모리를 획득하며 성공 시 0, 실패 시 -1을 반환
- 참고로 break는 BSS영역 끝의 다음 위치에 존재함
- ex) brk(주소 값) : 요청한 주소 값까지 메모리 할당
<br>

**sbrk()**
- sbrk의 기능은 내부적으로 brk system call을 사용
- brk와 동일하게 작업하며 성공 시 세그먼트 break주소를 반환하며 실패 시 -1을 반환
- ex) sbrk(메모리 크기) : 이전 세그먼트부터 메모리 크기만큼 메모리 할당
<br>

**mmap()**
- 새로운 메모리를 할당하고 호출한 프로세스에서 해당 메모리를 사용

사용자가 요청한 크기가 현재 arena(main_arena이든 thread_arene이든)에 사용자의 요청을 
만족시킬 수 있는 충분한 공간이 없는 경우, mmap syscall(brk 미사용)을 사용하여 부족한 메모리를 할당한다.


## 참고 및 출처
 [**참고 1**](https://rninche01.tistory.com/entry/heap2-glibc-malloc1-feat-Arena) <br>
 [**참고 2**](https://jeongzero.oopy.io/bcb0067a-3d2d-4e00-b8e7-499fba15e1bb#bcb0067a-3d2d-4e00-b8e7-499fba15e1bb)<br>
[**이미지 출처**](https://core-analyzer.sourceforge.net/index_files/image702.png)
