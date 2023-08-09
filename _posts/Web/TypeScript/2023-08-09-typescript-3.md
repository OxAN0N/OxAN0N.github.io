---
title : "TypeScript 3"
excerpt : "유니언과 내로잉"
categories :
  - TypeScript
tags:
  - TypeScript
  - Web
date:               2023-08-09 01:00:0 +0000
last_modified_at:   2023-08-09 01:00:0 +0000
---


## 0. 유니언과 내로잉

타입스크립트가 변수의 값을 바탕으로 추론을 하기 위한 핵심적인 개념이 바로 유니언과 내로잉이다. 

* 유니언 (union) : 값에 허용된 타입을 두 개 이상의 가능한 타입으로 확장
* 내로잉 (narrowing) : 값에 허용된 타입이 하나 이상의 가능한 탕입이 되지 않도록 좁힘.

유니언과 내로잉은 타입스크립트에서 '코드 정보에 입각한 추론'을 가능하게 하는 중요한 개념이다.

## 1. 유니언 타입

```
let test = Math.random() > 0.5
    ? undefined 
    : "success";
```

위의 test는 undefined 타입이거나, string 타입일 수 있다.  이와 같이 유니언 타입은 값이 정확히 어떤 타입인지 알 수 없지만, 두 개 이상의 옵션 중 하나라는 것을 알고 있는 경우에 코드를 처리하는 개념이다. 


타입스크립트는 위와 같은 유니온 타입을 **string | undefined** 형태로 나타낸다.


#### 1.1 유니언 타입 선언
```
let test : string | null = null

if(Math.random() > 0.5) {
    test = "Test";
}

```

위와 같이 변수의 초깃값이 존재하더라도 변수에 대한 명식적 타입 애너테이션을 제공하면, 초깃값이 null이지만 잠재적으로 해당 변수가 string 타입이 될 수 있다는 것을 알려줄 수 있다. 

이와 같이 유니언 타입 선언은 애너테이션으로 타입을 정의하는 모든 곳에서 사용할 수 있다.

#### 1.2 유니언 속성

값이 유니언 타입인 경우, 타입스크립트는 유니언으로 선언한 모든 가능한 타입에 존재하는 멤버 속성에만 접근할 수 있다.

두 속성 모두에 존재하는게 아닌 특정 타입에만 존재하는 멤버 속성의 경우에는 접근할 수가 없다.

```
let test = Math.random() > 0.5
    ? 10 
    : "success";


test.toString() ; //OK

test.toUpperCase(); // Error : number 존재 x

test.toFixed() // Error : string에 존재 x
```

이처럼 타입스크립트는 모든 유니언 타입에 존재하지 않는 속성에 대한 접근을 제한함으로써, 안전 조치를 가한다. 

## 2. 내로잉

내로잉은 값이 정의, 선언 혹은 이전에 유추된 것보다 더 구체적인 타입임을 코드에서 유추하는 것을 말한다. 

타입스크립트가 이와 같은 내로잉을 하게 되면, 값을 더 구체적인 타입으로 취급한다.

타입을 좁히는데 사용할 수 있는 논리적인 검사를 타입 가드 (type guard)라고 한다. 

#### 2.1 값 할당을 통한 내로잉
변수에 값을 직접 할당 시, 타입스크립트는 변수 타입을 할당된 값의 타입으로 좁힌다.

```
let test : number | string;
test = "test"; // 변수의 타입이 string으로 내로잉 됨
test.toUpperCase(); // OK 
test.toFixed(); // string에 존재 x
```

변수에 유니언 타입 애너테이션이 명시되고, 초깃값이 주어질 때, 값 할당 내로잉이 작동한다.


#### 2.2 조건 검사를 통한 내로잉
일반적으로 타입스크립트에서는 변수가 알려진 값과 같은지 확인하는 if문을 통해서 변수의 값을 좁히는 방법을 사용한다. 
```
let test : string | null = null

if(Math.random() > 0.5) {
    test = "Test";
}

if(test==="Test"){
    test.toUpperCase(); //OK
}

test.toUpperCase(); // Error

```

위와 같이 if문 내에서 변수가 알려진 값과 동일한 타입인지 확인한다.

#### 2.3 typeof 검사를 통한 내로잉

```
let test : string | number = Math.random() > 0.5 ? "Test" : 10

console.log(typeof test === "string" ? test.toUpperCaese() : test.toFixed());

```


위와 같이 typeof 검사를 이용해도 내로잉을 수행할 수 있다. 


타입스크립의 타입 검사기는 이외에도 더 많은 내로잉 형태를 인식한다.