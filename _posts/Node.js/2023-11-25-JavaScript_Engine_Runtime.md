---
title : "JavaScript 엔진과 런타임"
excerpt : ""
categories :
  - Node.js
tags:
  - RTE
  - JavaScript Runtime
  - JavaScript Engine
date:               2023-11-25 01:00:0 +0000
last_modified_at:   2023-11-25 01:00:0 +0000
---

## 1. 런타임 환경

### 런타임 환경 (Runtime Environment : RTE)

> 💡 애플리케이션이 운영체제의 시스템 자원(RAM, 시스템 변수 등)에 액세스 할 수 있도록 하는 실행환경.

- 런타임 환경을 통해 프로세서에 명령을 보내, 시스템 자원에 접근 사용.
- 런타임 환경은 OS 자체에 속할 수도 있고, OS 위에서 작동하는 소프트웨어인 경우도 있음.
    - ex. JRE, NodeJS
- 고수준 언어 기반의 애플리케이션은 런타임 환경이 아니면 시스템 리소스에 접근하기 어려움

 - cf. 
 런타임 : 애플리케이션이 필요한 시스템 자원을 할당받아 이를 이용해 어떤 처리를 하고 있는 상태.

## 2. JavaScript 엔진과 런타임


### JavaScript 런타임

> 💡 JavaScript 기반 프로그램 혹은 코드가 구동되는 환경

- 예시
    - 브라우저 - 크롬, 파이어폭스 등
    - NodeJS
    

### JavaScript 엔진
> 💡 JavaScript 코드를 실행하는 프로그램 또는 인터프리터

- 전통적으로 웹 브라우저에서 사용되었으나, Node.js처럼 브라우저 외부 런타임 환경에서도 사용됨.
- 예시
    - [V8](https://www.notion.so/V8-Engine-7ae089ea50f34275a2cab5f3671c1113?pvs=21) (Chrome)
    - WebKit (Safari)
    - SpiderMonkey (FireFox)
    - JerryScript (사물인터넷)

## Ref.

- [https://ko.wikipedia.org/wiki/런타임](https://ko.wikipedia.org/wiki/%EB%9F%B0%ED%83%80%EC%9E%84)
- [https://ko.wikipedia.org/wiki/런타임_시스템](https://ko.wikipedia.org/wiki/%EB%9F%B0%ED%83%80%EC%9E%84_%EC%8B%9C%EC%8A%A4%ED%85%9C)
- [https://ko.wikipedia.org/wiki/자바스크립트_엔진](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8_%EC%97%94%EC%A7%84)
- [https://velog.io/@ahsy92/기술면접-JavaScript-런타임-작동방식-비동기와-이벤트-루프](https://velog.io/@ahsy92/%EA%B8%B0%EC%88%A0%EB%A9%B4%EC%A0%91-JavaScript-%EB%9F%B0%ED%83%80%EC%9E%84-%EC%9E%91%EB%8F%99%EB%B0%A9%EC%8B%9D-%EB%B9%84%EB%8F%99%EA%B8%B0%EC%99%80-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84)