---
title : "TypeScript 4"
excerpt : "리터럴과 타입의 기타 성질"
categories :
  - TypeScript
tags:
  - TypeScript
  - Web
date:               2023-08-09 01:00:0 +0000
last_modified_at:   2023-08-09 01:00:0 +0000
---

## 1. 리터럴 타입

```
const test = "test";
```

위에서 test의 타입은 string타입이라고 말할 수 있다. 

그러나 좀 더 기술적으로 접근하면 test는 단지 string 타입이 아닌 "test"라는 특별한 값이다.

따라서 변수 test타입은 기술적으로 더 구체적인 "test"이다. 

원시 타입 값 중 어떤 것이 아닌 특정 원싯값으로 알려진 타입이 리터럴 타입니다.

원시 타입 string은 존재할 수 있는 모든 가능한 문자열의 집합을 나타내는 반면, 리터럴 타입인 "test"는 하나의 문자열만을 나타낸다. 

만약 변수를 const로 선언하고 직접 리터럴 값을 할당 시, 타입스크립트는 해당 변수를 할당된 리터럴 값으로 유추하고 원시 타입 대신 해당 리터럴을 표시한다. 


각각의 원시 타입은 해당 타입의 가능한 모든 리터럴 값의 집합으로도 해석할 수 있다. 

* boolean : true | false
* null과 undefined : 둘 다 자기 자신, 즉 하나의 리터럴 값만을 가짐
* number : 0 | 1 | 2 | ... | 0.1 | ...
* string : "" | "a" | "b" | ... | "ab" | ...


유니언 타입 애너테이션에서는 리터럴과 원시 타입을 섞어서 사용할 수도 있다.

```
let test : number | "test" | "success" ;

test = 89; // ok
test = "test"; // ok

test = true // Error

```

#### 1.1 리터럴 할당 가능성

0과 1과 같은 동일한 원시 타입일 지라도 서로 다른 리터럴 타입은 서로 할당할 수 없다. 

```
let test : 0 | string;

test = 0; // ok 
test = 1; // Error
```

그러나 리터럴 타입은 그 값이 해당하는 원시 타입에는 할당 할 수 있다. 모든 특정 리터럴 타입은 여전히 해당 원시 타입의 부분이기 때문이다.


## 2. 참 검사를 통한 내로잉

자바스크립트에서 참 또는 참으로 평가되는 것(truthy)들은 && 연산자 또는 if문처럼 boolean 문맥에서 true로 간주된다. 
(false, 0, -0,0n, "",null,undefined,NaN와 같이 falsy로 정의된 값을 제외한 모든 값은 참이다.)

타입스크립트는 잠재적인 값 중 truthy로 확인된 일부에 한해서만 변수의 타입을 좁힐 수 있다. 

```
let test = Math.random()>0.5
    ? "test"
    : undefined
```

위의 코드에서 test는 string | undefined 타입인데, undefined는 항상 falsy 한 리터럴이자 원시 타입이므로, 타입스크립트의 if 문의 코드 블럭에서는 test가 항상 string 타입이 되어야 한다. 

즉, 위에서 정의한 test는 if문 코드 블럭에서는 항상 string으로 간주된다. 

```
if(test){ // 이때 test는 string으로
    test.toUpperCase();
}
```

논리 연산자인 &&과 ?도 참여부를 검사하는 일을 수행한다.
(그러나 해당 연산자들은 참 여부의 확인 외에는 다른 일, 예를 들어 string|undefined가 falsy일 때, 빈 문자열이기 때문에 falsy인지 undefined인지 알 수 없다.)

```
test && test.toUpperCase() // OK : String | undefined

test?.toUpperCase() // OK : string | undefined

if(test){
  test.toUpperCase(); //OK string
}
else {
  test // "" (string falsy) or undefined;
}
```

## 3. 초깃값이 없는 변수

자바스크립트에서 초깃값이 없는 변수는 기본적으로 undefined가 된다. 

```
let test : string;

test?.length; // Error : 값이 할당 안됨
```
만약 undefined를 포함하지 않는 타입으로 변수를 선언하고, 해당 변수에 값을 할당하기 전에 속성에 접근하려는 것처럼 해당 변수를 사용하면 오류가 난다.

반면 변수 타입에 undefined가 포함되어 있는 경우에는, 오류가 보고되지 않는다. 

```
let test : string | undefined;
test?.toUpperCase() // Ok
```
undefined는 유효한 원시 타입이기 때문에 사용되기 전에 정의가 될 필요 없다. 

## 4. 타입 별칭

긴 형태의 유니언 타입의 경우, 다음과 같이 타입 별칭 (type alias)를 할당하면, 추후에 더 용이하게 재사용 할 수 있다. 

```
type RawData = boolean | number | string | null | undefined

let dataFirst : RawData;
```
이와 같은 타입 별칭은 타입스크립트 타입 시스템이 해당 별칭을 발견 시, 마치 복사 붙여넣기 한 것처럼 해당 별칭이 참조하는 실제 타입을 입력한 것처럼 작동한다. 

실제 타입 별칭은 타입스크립트 타입 시스템에만 존재하기 때문에, 자바스크립로 컴파일 되지 않는다.
즉, 타입 별칭은 런타임 코드에서는 참조할 수 없으며, **'개발 시'** 에만 존재하기 때문에, 런타임 코드에서 이에 접근하려고 하면 타입 오류로 이를 알려준다. 

```
type Id = number | string;

console.log(Id) // Error : 타입오류 
```

#### 타입 별칭 결함
타입 별칭은 다음과 같이 다른 타입 별칭의 참조가 가능하다. 

```
type Id = number | string;

type IdMaybe = Id | undefined | null;
```

