---
title : "[Heap] Heap basic #3"
excerpt : "Chunk의 구조"
categories :
  - pwnable
tags:
  - Heap
  - Memory
  - pwnable
date:               2023-01-14 02:00:0 +0000
last_modified_at:   2023-01-14 02:00:0 +0000
---

## 1. Chunk의 개념
덩어리라는 뜻의 chunk는 ptmalloc이 호출되면서 할당받는 영역을 공간을 의미한다.

chunk는 header와 data(payload)로 구성되는데, header는 chunk 관리에 필요한 정보를 담고 있고, data 영역에는 사용자가 입력한 데이터가 저장된다.

일반적으로 malloc()을 호출했을 때, 반환되는 주소를 이용해 데이터를 입력하게 되는데, 반환되는 주소는 chunk의 시작 부분인 header가 아닌 data(payload) 영역의 주소이다. 

chunk는 32bit에서는 8byte 단위로 할당이 되고, 64bit에서는 16byte 단위로 할당된다.

```c
struct malloc_chunk {
  INTERNAL_SIZE_T      prev_size;  /* Size of previous chunk (if free).  */
  INTERNAL_SIZE_T      size;       /* Size in bytes, including overhead. */

  struct malloc_chunk* fd;         /* double links -- used only if free. */
  struct malloc_chunk* bk;

  /* Only used for large blocks: pointer to next larger size.  */
  struct malloc_chunk* fd_nextsize; /* double links -- used only if free. */
  struct malloc_chunk* bk_nextsize;
};
```

chunk에는 크게 3가지의 타입이 존재한다.

## 2. Allocated Chunk 
<br>
![Allocated_chunk][1]

- prev_size : 인접한 직전 청크의 크기. 청크를 병합할 때 직전 청크를 찾는 데 사용된다. 만약 해당 chunk 바로 이전의 chunk가 해제된 경우, 해당 필드는 이전 chunk의 크기를 저장한다. 만약 해제된 chunk가 없고 모두 allocated chunk인 경우, 해당 필드는 기본값인 0으로 설정된다.
<br>(현재 chunk의 prev_size는 이전 chunk의 free()시에 설정되며, 현재 chunk의 시작주소에서 prev_size를 빼면 이전 chunk의 시작 주소를 구할 수 있다.)
- size : 현재 할당된 chunk의 크기를 저장한다.(헤더의 크기를 포함한다.) 64bit에서 chunk는 16byte 단위로 할당되므로 하위 3bit는 항상 0으로 고정된다. 따라서 이러한 3bit는 다음과 같이 chunk 관리에 필요한 3개의 flag 값을 표시하는데 사용된다.
  - NON_MAIN_ARENA(A) - 현재 청크가 thread_arena에 위치하는 경우, 1로 세팅됨
  - IS_MMAPPED(M) - 현재 청크가 mmap을 통해 할당된 경우 1로 세팅됨. 큰 메모리를 요청하는 경우에는 heap을 이용하지 않고, mmap() 시스템 콜을 통해 별도의 메모리 영역을 할당함. 이러한 청크들은 bin 내에 속하지 않음. free시 그냥 munmap() 호출로 해지함
  - PREV_INUSE(P) - 현재 청크 바로 이전의 청크가 할당되어 있는 경우 1로 세팅됨. ptmalloc은 해당 플래그를 참조해 병합이 필요한지 아닌지 판단할 수 있음

cf) 다음 청크의 prev_size 필드도 현재 청크의 페이로드 필드로 사용된다. 따라서 실질적인 청크는 다음 청크의 prev_size까지 포함된다. free시에는 해당 영역은 현재 청크의 크기(size 필드의 값)를 다음 chunk의 prev_size에 중복하여 저장하는 용도로 사용한다.

## 3. Freed Chunk
<br>
![Freed Chunk][2]
<br>

free된 chunk들은 단일한 freed chunk로 결합된다는 특징을 가지고 있다. (논리적으로는 하나의 free chunk가 되지만 실질적으로는 비순차적으로 free된 영역들을 모아둔 것으로 freed chunk의 physical한 chunk 단위는 연속적이지 않다. - 단일/이중 연속 리스트 형태로 관리)

- prev_size 
  
  ![boundary_tags][3]
  - 위와 같은 상황에서 chunk B를 free하려고 할 때, chunk B는 이전에 free된 chunk A와 결합될 것이다.
  - 이때, chunk들을 병합하기 위해서 chunk B는 chunk A의 크기를 필요로 한다. chunk A가 이전에 free될 때, chunk A는 자신의 size 필드의 값을 다음 chunk (이 경우 chunk B)의 prev_size 필드에 복사한다.(위에서 설명한 대로, 특정 chunk의 data 영역에 그 다음 chunk의 prev_size 필드가 포함되는 이유가 여기에 있다.)
  - 따라서 chunk B는 자신의 chunk 주소에서 해당 prev_size 필드의 값을 빼서 이전 chunk A의 header 주소의 시작값을 구해서 결합을 수행할 수 있다. 

- fd (forward pointer) / bk (backward pointer)

  fd와 bk는 각각 해제된 다음 chunk와 해제된 이전 chunk를 가리키는 포인터이다. 이러한 포인터는 단일/이중 연결 리스트 형태로 구현되어잇다. `bins`들에서 freed chunk들이 관리된다. (bins에 대한 자세한 설명은 다음 [포스팅][7]에서 다룬다.)

  ![bins_1][4]

  bin에 free된 chunk들은 이중 리스트 형태로 연결되어 있다. (fastbins의 경우, 단일 리스트 형태)
  fd,bk가 이전과 다음 freed chunk들을 가리키기 때문에, 전체 free된 chunk들을 순회하면서 접근할 수 있다. 

  이와 같이 연결된 chunk들은 물리적으로 붙어있지 않고 다음과 같이 불연속적이다. 
  
  ![bins_2][5]

- fd_nextsize, bk_nextsize

  bins는 freed chunk들을 관리한다. 해당 bin들도 freed chunk의 size에 따라서 fastbin, unsorted bin, small bin, large bin으로 나뉘어진다. (이후 포스팅에서 다룸) 

  이중 fd_nextsize, bk_nextsize 포인터는 가장 큰 freed chunk를 관리하는 large bin을 위해서만 사용된다.

 large bin을 제외한 다른 bins의 각 index에는 동일 사이즈의 chunk들로 리스트가 형성된다. 그러나 large bin은 동일한 size가 아닌 chunk들을 관리하기 때문에, chunk들의 size를 알아야 한다.



## 4. Top Chunk

Top Chunk는 [Arena][6]의 가장 상위 영역에 존재하는 chunk이다. 처음 malloc()을 호출 시 사용자가 요청한 사이즈 만큼이 아닌 충분한 크기의 메모리를 받아온다. 그리고 해당 size를 Top chunk에 넣는다.

freed chunk들이 메모리 할당 요청을 만족하지 못하는 경우, Top chunk를 분할하여 해당 요청을 처리하는데 이 경우 Top chunk는 다음의 2개로 분할 된다.
1. User chunk : 사용자가 요청한 크기
2. Remainder chunk : 나머지 크기, 새로운 Top chunk가 된다. 

만약 현재의 Top chunk의 크기보다 더 큰 사이즈를 요청하는 경우, main_arena의 경우 sbrk로 Top chunk의 크기를 확장시키고, thread_arena의 경우 mmap으로 새롭게 메모리 영역을 받아온다.

만약 Top chunk와 인접한 chunk가 free 된다면, 해당 chunk는 Top chunk와 병합된다. (Chunk consolidation)


## 출처 & 참조
[**참조 1**](https://jeongzero.oopy.io/b7a95ad0-a76b-4cb9-a431-7f2a5cd95258)<br>
[**참조 2**](https://dreamhack.io)


[1]: ../../assets/images/pwnable/allocated_chunk.png
[2]: ../../assets/images/pwnable/freed_chunk.png
[3]: ../../assets/images/pwnable/boundary_tag.png
[4]: ../../assets/images/pwnable/bin_1.png
[5]: ../../assets/images/pwnable/bin_2.png

[6]: /pwnable/Heap-arena/
[7]: /pwnable/Heap-bins/