---
title : "Tcache Poisioning"
excerpt : ""
categories :
  - Pwnable
tags:
  - Heap
  - Tcache Poisoning
  - Pwnable
date:               2023-01-21 01:00:0 +0000
last_modified_at:   2023-01-21 01:00:0 +0000
---


## 1. Tcache Poisoning

Tcache Poisoning은 tcache를 조작해 임의의 주소에 chunk를 할당시키는 공격 기법을 말한다. 

Tcache poisoning은 중복으로 연결된 청크를 재할당하면, 그 청크가 freed chunk인 동시에, Allocated chunk라는 특징을 이용한다.

Freed Chunk에서 fd와 bk는 Allocated Chunk에서 data를 저장하는데 사용되는 영역 내에 속한다.

따라서 공격자가 중첨 상태인 chunk 내에 임의의 값을 쓸 수 있다면, 해당 chunk의 fd와 bk 값을 조작할 수 있게 되어, ptmalloc2의 free list에 임의의 주소를 추가할 수 있게 된다.

ptmalloc2는 동적 할당 요청에 대해 free list의 chunk를 먼저 반환하므로, 이를 이용하면 공격자는 임의 주소에 chunk를 할당할 수 있다. 

Tcache Poisoning으로 할당한 chunk에 대해 값을 출력하거나, 조작할 수 있다면 공격자는 임의의 주소에 chunk를 할당할 수 있으며, 그 청크를 이용하여 임의 주소의 데이터를 읽거나 조작할 수 있게 된다.


