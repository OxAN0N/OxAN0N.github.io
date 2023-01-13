---
title : "[Heap] Heap basic #1"
excerpt : "메모리 Heap 영역과 그 동작 방식"
categories :
  - Pwnable
tags:
  - Heap
  - Memory
  - Pwnable
last_modified_at : 2023-01-13
---

# 1. 개요

개발자가 사전에 필요한 변수와 그 크기를 아는 경우, 일반적으로 전역 변수 (초기화 된 전역변수는 .data 영역, 초기화되지 않은 전역 변수는 .bss 영역에 할당됨)나 함수 내부에서 지역 변수 (stack 영역에서 할당됨)로 선언하여 해당 변수명, 혹은 해당 변수를 가리키는 포인터를 사용하여 접근한다.

이러한 지역변수나 전역 변수의 경우, 데이터 타입에 따라 변수의 사이즈가 컴파일 시에 결정되고, 이때 선언되는 변수의 사이즈는 고정으로 변경하지 못한다.

그러나, 컴파일 시가 아닌, 런타임 시에 변수의 크기를 결정해야 되는 경우가 있다. (ex. 사용자의 입력값에 따라)

이러한 경우, 해당 데이터는 프로세스의 Heap 영역에 할당이 되고, 이러한 데이터는 일반적으로 해당 메모리 영역의 주소를 저장하고 있는 포인터 변수를 통해서 접근한다.

```c
int main(void){
    int * pData = NULL;

    pData=(int *)malloc(sizeof(int));
    *pData=5;
    printf("Data Stored in %p : %d\n",pData,*pData);
    return 0;
}
```

# 2. Heap 기초
### 2.1 Dynamic Memory Allocator
위와 같이 Heap 영역에서의 동적으로 할당된 메모리를 관리하기 위해서 Dynamic Memory Allocator가 사용된다.

이러한 Allocator는 크게 두 가지 종류가 존재한다.
1. Explicit Allocator : 개발자가 명시적으로 공간의 할당/해제를 지시<br>
    ex)libc의 malloc, free
2. Implicit Allocator : 개발자가 공간의 할당을 지시 / free의 경우 내부적으로 처리<br>
    ex) Java의 Garbage Collection

Explicit Allocator에는 다양한 종류가 존재하나, 해당 포스팅에서는 glibc에서 사용하는 ptmalloc2에 대해서만 자세하게 다루도록 하겠다.

### 2.2 ptmalloc2
ptmalloc2 이전의 dlmalloc의 경우, 동시에 2개 이상의 thread가 메모리 할당을 요청할 때, freelist(동적 메모리 할당을 위해서 계획적으로 사용된 데이터 구조)가 사용 가능한 thread에 둘러싸인 상태로 공유되기 때문에, race condition을 막기 위해, 한번에 한 thread만 critical section에 접근할 수 있도록 처리되었다. 따라서 이러한 dlmalloc은 multithreaded 환경에서 성능 저하가 발생했다.

ptmalloc2는 dlmalloc에 multithreaded 환경에서 효율적으로 동작하는 기능을 추가한 Allocator이다. 

ptmalloc2는 동시에 다수의 thread가 malloc을 호출할 경우, 메모리는 각각의 thread가 분배된 heap 영역을 일정하게 유지하고, 이러한 heap을 유지하기 위한 freelist 또한 분배되기 때문에, 메모리 할당이 즉시 이뤄진다. 

Multithreaded 환경에서 각각의 thread가 서로 간섭하지 않고 서로 다른 메모리 영역에 access 할 수 있도록 도와주는 heap 영역을 Arena라고 한다. (위에서 기술한 분배된 heap 영역을 말한다.)

Arena에 대한 자세한 설명은 다음 [포스팅][1]에서 기술한다.

### 2.3 함수 호출 알고리즘

- **malloc 함수 호출 순서 : libc_malloc() → int_malloc() → sysmalloc()**
1. libc_malloc() 함수에서 사용하는 Thread에 맞게 Arena를 설정한 후, int_malloc() 함수 호출
2. int_malloc() 함수에서는 재사용할 수 있는 bin을 탐색하여 재할당하고, 마땅한 bin이 없다면 top chunk에서 분리해서 할당
3. top chunk가 요청한 크기보다 작은 경우, sysmalloc() 함수 호출
4. sysmalloc() 함수를 통해 시스템에 메모리를 요청해서 top chunk의 크기를 확장하고 대체
<br>※ sysmalloc() 함수는 기존의 영역을 해제한 후, 새로 할당함<br>
<center><img src="https://oopy.lazyrockets.com/api/v2/notion/image?src=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F98d594a8-8a72-4116-a2d8-822871fc4f2e%2FUntitled.png&blockId=84dc7cbe-a6f0-4862-b876-a39325d83dc8" width="70%"></center>

- **free 함수 호출 순서 :  libc_free() -> int_free() -> systrim() or heap_trim() or munmap_chunk()**
1. libc_free() 함수에서 mmap으로 할당된 메모리인지 확인한 후, 맞을 경우 munmap_chunk() 함수를 통해 메모리 해제
2. 아닌 경우, 해제하고자 하는 chunk가 속한 Arena의 포인터를 획득한 후, int_free() 함수 호출
3. chunk를 해제한 후, 크기에 맞는 bin을 찾아 저장하고 top chunk와 병합을 할 수 있다면 병합 수행
4. 병합된 top chunk가 너무 커서 Arena의 크기를 넘어선 경우, top chunk의 크기를 줄이기 위해 systrim() 함수 호출
5. 문제가 없다면, heap_trim() 함수 호출
6. mmap으로 할당된 chunk라면 munmap_chunk()를 호출 <br>
<center><img src="https://oopy.lazyrockets.com/api/v2/notion/image?src=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F9f25f0d7-96df-4bd3-8643-4e348d72d236%2FUntitled.png&blockId=3c284f69-65aa-4604-8a5e-866194bfeada" width="70%"></center>


## 참고 & 출처
[**출처 1**](https://jeongzero.oopy.io/bcb0067a-3d2d-4e00-b8e7-499fba15e1bb#bcb0067a-3d2d-4e00-b8e7-499fba15e1bb)

[1]: /pwnable/heap-arena/





