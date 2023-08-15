---
title : "TypeScript 7"
excerpt : "배열"
categories :
  - TypeScript
tags:
  - TypeScript
  - Web
date:               2023-08-15 01:00:0 +0000
last_modified_at:   2023-08-15 01:00:0 +0000
---

## 0. 배열

내부에 모든 타입의 값을 혼합해서 저장이 가능했던 자바스크립트 배열과는 달리, 타입스크립트는 하나의 배열이 특정 데이터 타입으로만 동작하도록 제한한다. 

## 1. 배열 타입
배열에 초깃값을 넣으면, 자동으로 해당 배열의 요소의 타입을 유추할 수 있다. 

그러나 다음과 같은 배열에 대한 타입 애너테이션을 통해 배열의 요소 타입이 무엇인지 명시할 수도 있다.

```typescript
let arrayOfNumbers : number [];
arrayOfNumbers = [1,2,3,4,5];
```

#### 1.1 배열과 함수 타입 
```typescript
let createStrings: () => string [];
let stringCreators: (()=> string)[];
```
위의 예시에서 createStrings는 string 배열을 반환하는 함수이고,
stringCreators는 string을 반환하는 함수의 배열이다. 

치어럼 괄호를 통해 애너테이션의 어느 부분이 함수 반환 부분이고 어느 부분이 배열 타입 묶음인지를 잘 명시해야 한다. 


#### 1.2 유니언 타입 배열
배열의 각 요소가 여러 타입 중 하나일 수 있을 때는 유니언 타입을 사용한다. 

```typescript
let stringOrArrayOfNumbers: string | number [];
let arrayOfStringOrNumbers: (string|number)[];
```

유니언 타입을 배열 타입 애너테이션과 함께 사용할 때는 어느 부분이 배열의 컨텐츠이고 어느 부분이 유니언 타입 묶음인지를 나타내기 위해서 위와 같이 괄호를 사용해야 할 수도 있다. 

stringOrArrayOfNumbers는 해당 타입이 string 혹은 number의 배열일 수 있는 유니언 타입 변수이고,

arrayOfStringOrNumbers는 배열의 요소가 string 혹은 number임을 나타낸다. 

#### 1.3 any 배열의 진화

초기에 초깃값이 설정되지 않거나, 타입 애너테이션이 붙어있지 않은 빈 배열의 경우, 타입스크립트는 이를 any[]로 취급하고 모든 타입의 값을 받을 수 있다. 

```typescript
let values = []; //타입 : any[]

values.push(''); // 타입 : string[]

values[0] = 0; // 타입 : (string | number) []
```

위와 같이 어떤 타입의 값이든 받을 수 있기 때문에, 잠재적으로 잘못된 값 추가를 허용할 수 있어, 주의해야 한다. 

#### 1.4 다차원 배열
``` typescript
let arrayOfArraysOfNumbers: number[][];

arrayOfArraysOfNumbers = [
    [1,2,3],
    [2,3,4],
    [3,4,5],
];

```
위와 같은 2차원 이상의 배열의 경우, 해당 타입의 배열을 요소로 가지는 배열로 해석할 수 있으며, 이는 결과적으로 다음 예시의 두 변수의 타입이 동일함을 나타낸다. 

```typescript
let test: number[][];
let test2: (number[])[];
```

## 2. 배열 멤버
타입스크립트는 배열의 멤버를 찾아서 해당 배열의 타입 요소를 반환하는 전형적인 인덱스 기반 접근를 지원하는 언어이다. 

```typescript
const testStr = ["test1","test2"];
const testString = testStr[0] // type : string

const strOrDates = ["test",new Date(2023,8,15)];
const ret = strOrDates[0]; // 타입 : string | Date
```

해당 배열의 타입 요소를 그대로 반환하기 때문에, 위의 strOrDates와 같이 유니언 타입 배열의 경우, 그 반환값 역시 유니언 타입이다. 

#### 2.1 불안정한 멤버
```
function withElements(elements : string[]) {
    console.log(elements[9001].length); // 타입 오류가 없음
}

withElements(["test","test2"]);
```
자바스크립트의 경우 배열의 길이보다 큰 인덱스로 접근하면 undefined로 인식한다.

그러나 타입스크립트에서 위와 같은 사항일 경우, 코드 스니펫에서 elements[9001]은 undefined가 아닌 string 타입으로 간주된다. 

이처럼 타입스크립트는 검색된 배열의 멤버가 존재하는지 등을 확인하지 않는 경우가 있으므로, 배열의 길이 등을 체크하는 코드가 필요하다. 

cf. noUncheckedIndexedAccess 등과 같은 배열 조회를 더 제한하고 타입을 안전하게 만드는 플래그가 있으나, 너무 엄격해서 대부분의 프로젝트에서는 사용하지 않는다.

## 3. 스프레드

... 스프레드 연산자를 사용해 배열을 결합할 수 있다. 

```typescript

const str = ["test","test2"];
const nums = [10,20,30];

const conjoined = [...str, ...nums];
// type : (string|number)[]

console.log(conjoined[0]) // 출력값 : test

```
입력한 배열이 동일한 타입이라면 출력 배열도 동일한 타입이다.

그러나, 서로 다른 타입의 두 배열을 스프레드해 새로운 배열을 생성 시, 새로운 배열은 유니언 타입 배열이 된다.


## 4. 튜플

튜플 배열은 각 인덱스에 특정 타입을 가지면, 배열의 모든 가능한 멤버를 갖는 유니언 타입보다 더 구체적이다. 

```typescript
let numberAndStr: [number,string];

numberAndStr = [530, "test"]; // OK
numberAndStr = [false,"testt"] // Error
numberAndStr = ["test"] // Error
```

다음과 같은 형태의 튜플 배열도 선언이 가능하다.
``` typescript
const test: [string,[number,boolean]][] =[
    ["test",[1,true]],
    ["test2",[2,false]],
]
```


#### 4.1 튜플의 할당 가능성

튜플 타입은 가변길이의 배열 타입보다 더 구체적으로, 가변 길이의 배열 타입은 튜플 타입에 할당이 불가능하다. 

```typescript
const test = [false, 123]; //타입 :(boolean | number) [];

const testVar : [boolean, number] = test //Error : 가변길이의 배열 타입은 튜플 타입에 할당이 불가능

```

#### 나머지 매개변수로서의 튜플

튜플은 구체적인 길이와 요소 타입 정보를 가지는 배열로 간주된다.

따라서 함수에 전달한 인수를 저장하는 등에 유용하게 사용될 수 있다. 

```typescript
function logPair(name: string , value: number) {
    console.log(`${name} has ${value}`);
}

const pairArray = ["JDM",1];

logPair(...pairArray) // Error

const pairArrayError : [number, string] = [ 1,"JDM"];

logPair(...pairArrayError) // Error

const pairArrayCorrect : [string, number] = ["JDM", 1];

logPair(...pairArrayCorrect); // OK
```

위의 경우와 같이 타입스크립트는 ... 나머지 매개변수로 전달된 튜플에 정확한 타입 검사를 제공할 수 있다. 

이를 다음과도 같이 응용 가능하다.


``` typescript
function printTest(name: string, value: [number,boolean]){
    console.log(`${name} has ${value[0]}, ${value[1]}`);
}


const test: [string,[number,boolean]][] =[
    ["test",[1,true]],
    ["test2",[2,false]],
]

test.forEach(t => printTest(...t)); //OK
```
#### 4.2 튜플 추론
타입스크립트는 명시적으로 튜플임을 나타내지 않는 이상, 생성된 배열을 튜플이 아닌 가변 길이 배열로 취급한다. 

타입스크립트에서는 가변 길이 배열이 아닌 좀 더 구체적인 튜플 타입이어야 함을 명시적 튜플 타입과 const 어서션을 사용해 나타낸다. 

#### 명시적 튜플 타입

```typescript
function test(input: string): [string,number]{
    return [input[0],input.length];
}

const [testVar, size] = test("testStr");
// testVar 타입 : string
// size 타입 : number
```

함수가 튜플 타입을 반환한다고 선언되고, 배열 리터럴을 반환 시, 해당 배열 리터럴은 일반적인 가변 길이의 배열이 아닌 튜플로 간주된다. 


#### const 어서션
타입스크립트는 값 뒤에 넣을 수 있는 const 어서션인 as const 연산자를 제공한다. 
const 어서션은 타입스크립트가 타입을 유추할 때, 읽기 전용 형식을 사용하도록 지시한다. 

```typescript
const unionArray = [11,"test"];
const readOnlyTuple = [11,"test"] as const
```

위와 같이 배열 리터럴 뒤에 as const가 배치되면 배열이 튜플로 처리되어야 함을 나타낸다. 

그러나 이러한 as const를 사용하면, 해당 배열을 고정된 크기의 튜플로 전환하는 것을 넘어, 읽기 전용이 되고, 값 수정이 불가능하게 된다. 

(return 문의 마지막에 as const를 사용시 반환된 값을 받는 변수도 const 값으로 수정이 불가능)
