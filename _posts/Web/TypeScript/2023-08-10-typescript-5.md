---
title : "TypeScript 5"
excerpt : "객체"
categories :
  - TypeScript
tags:
  - TypeScript
  - Web
date:               2023-08-10 01:00:0 +0000
last_modified_at:   2023-08-10 01:00:0 +0000
---

## 1. 객체 타입

{...} 구문을 사용해서 객체 리터럴을 생성 시, 타입스크립트는 선언된 객체 리터럴의 속성을 기반으로 새로운 객체 타입 또는 타입 형태를 고려한다. 

```
const poet = { 
    born : 1999,
    name : "JDM".
};

poet['born']
poet.name;
```

위에서 poet이라는 변수에, born : number와 name : string이라는 속성들을 가지는 객체 리터럴을 생성했는데, 이를 보고 타입스크립트는 동일한 속성과 타입을 가지는 새로운 객체 타입 또는 타입 형태를 내부적으로 정의하는 것이다. 

해당 객체 타입은 객체의 값과 동일한 속성명과 원시 타입을 갖는다. 

이러한 값의 속성에 접근하려면 value.멤버 혹은 value['멤버'] 구문을 사용한다. 

#### 1.1 객체 타입 선언
위의 내용과 같이 직접 타입을 유추할 수도 있지만, 다음과 같이 명시적으로 타입을 선언할 수도 있다. 

```
let poetLater : {
    born : number;
    name : string;
};

```
위와 같이 선언 시, poetLater라는 변수는 born:number와 name : string을 속성으로 가지는 객체 타입으로 명시적으로 선언된 것이다.

따라서 해당 변수에 다른 타입의 값을 할당하려고 하면 에러 메시지를 보인다.

#### 1.2 별칭 객체 타입
위와 같이 객체 타입을 매번 특정 변수에 적기보다는 객체 타입에 별도의 타입 별칭을 할당해서 사용하는 다음과 같은 방법이 더 일반적이다. 

```
type Poet = {
    born : number;
    name : string;
};

let poetLater : Poet;

poetLater = {
    born : 1999,
    name : "JDM"
};
```

대부분의 타입스크립트 프로젝트는 객체 타입을 설명시 interface 키워드를 사용하는 것을 더 선호한다. 관련 내용은 추후 포스팅에서 더 자세히 다루겠다.

## 2. 구조적 타이핑
타입 스크립트의 타입 시스템은 구조적으로 타입화 (structurally typed)되어 있다. 

즉, 아래와 같이 타입을 충족하는 모든 값을 해당 타입의 값으로 사용할 수 있다. 

매개변수나 변수가 특정 객체 타입으로 선언 시, 타입스크립트에 어떤 객체를 사용하든 해당 속성이 있어야 한다고 말해야 한다. 

```
type WithFirstName = {
    firstName : string;
};

type WithLastName = {
    lastName : string;
};

const hasBoth = {
    firstName : "DM",
    lastName : "J"
};

let withFirstName : WithFirstName = hasBoth;
// Ok : hasBoth 변수는 내부적으로 WithFirstName의 속성을 가지고 있음

let withLastName : WithLastName = hasBoth;
// Ok : hasBoth 변수는 내부적으로 WithLastName의 속성을 가지고 있음

```

이와 같은 구조적 타이핑은 정적 시스템이 타입을 검사하는 경우인 반면, 
덕 타이핑 (duck typing)은 런타임에서 사용될 때까지 객체 타입을 검사하지 않는다. 

(덕 타이핑은 동적 타이핑의 한 종류로, 객체의 변수 및 메서드의 집합이 객체의 타입을 결정하는 것을 말한다. 그런 속성을 가지고, 그런 행동을 한다면, 해당 객체일 것이다라고 이해하면 된다. - 어떤 타입을 가지고 어떤 타입을 반환하는지 등은 명시되지 않음)

자바스크립트는 덕타입인 반면, 타입스크립트는 구조적으로 타입화되어있다. 

#### 2.1. 사용 검사
객체 타입으로 애너테이션된 위치에 값을 제공할 때, 타입스크립트는 값을 해당 위치에 할당할 수 있는지 확인한다. 

할당하는 값에는 애너테이션에 명시된 객체 타입의 필수 속성이 있어야 한다. 만약 객체 타입에 필요한 필수 속성이 값에 없다면, 타입스크립트는 오류를 발생시킨다. 


```
type FirstAndLastNames = {
    first : string;
    last : string;
};

const hasBoth : FirstAndLastNames = {
    first : "DM",
    last : "J",
};

const hasOnlyOne :  FirstAndLastNames = {
    // Error : last 속성이 없음
    first : "DM",
};
```

속성은 같지만 다른 타입인 경우에도 에러를 발생시킨다. 

즉, 특정 객체 타입으로 선언된 변수에는 해당 객체의  속성(동일한 타입)을 만족하는 객체들만이 할당 될 수 있다. 

#### 2.2. 초과 속성 검사
변수가 객체 타입으로 선언되었을 때, 초깃값에 객체 타입에서 정의된 것보다 많은 필드가 존재한다면 타입스크립트에서는 타입 오류가 발생된다. 

```
type Poet = {
    born :number;
    name :string;
};

const test : Poet = {
    activity : "walking", // Error
    born :1999,
    name : "JDM",
};

```

앞서서, 구조적 타이핑에서 설명했던 것과 충돌하는 내용이 있다. 

초과 속성 검사는 **객체타입으로 선언된 위치에서 생성되는 객체 리터럴에 대해서만 발생한다.**

기존 객체 리터럴을 제공시에는 이러한 초과 속성 검사를 우회하기 때문에 앞선 구조적 타이핑에서의 예시가 가능하다. 

즉, 다음과 같은 예시는 에러가 발생하지 않는다.

```
const existingObject = {
    activity : "walking", // Error
    born :1999,
    name : "JDM",
};

const extraPropertyTest : Poet = existingObject // ok
```

이러한 초과 속성 금지를 통해서, 코드를 깨끗하게 유지하거나 예상 가능한 동작을 하도록 제한한다. 

(객체 타입에 선언되지 않은 초과 속성의 경우 종종 잘못 입력된 속성 이름이거나, 사용되지 않는 코드일 수 있기 때문이다.)


#### 2.3. 중첩된 객체 타입

타입스크립트의 객체는 자바스크립트의 객체와 마찬가지로, 객체 타입을 속성으로 가지는 객체 타입, 즉 중첩된 객체 타입을 지원한다.

```
type Poem = {
    author : {
        firstName : string;
        lastName : string;
    };
    name : string;
};

const poetMatch : Poem = {
    author : {
        firstName : "DM",
        lastName : "J",
    },
    name : "test",
}; // OK


const poetMisMatch : Poem = {
    author : {
        name : "DM",
        //Error : { firstName : string; lastName : string}
    },
    name : "test",
};

```

위와 같은 방법보다는 타입 별칭을 이용하면 더 간단 명료하게 표시 가능하다. 

```
type Author = {
        firstName : string;
        lastName : string;
};

type Poem = {
    author : Author;
    name : string;
};

```
이렇게 하면 에러 메시지도 {...}의 객체 형태 보다는 해당 객체명을 명시하기 때문에, 더 자세하고 간단하게 에러를 파악할 수 있다. 

#### 2.4. 선택적 속성
객체 타입의 속성이 항상 필수적인 것은 아니다. 타입의 속성 애너테이션에서 :앞에 ?를 추가시 해당 속성이 선택적 속성임을 나타낼 수 있다. 

```
type Name = {
        firstName : string;
        lastName? : string;
};

```
위와 같은 경우, lastName에 해당하는 속성은 Name 타입의 변수에 할당하는 값에서 필수적인 속성이 아니다. 

## 3. 객체 타입 유니언

객체 타입도 타입이기 때문에, 기존 원시 타입과 리터럴 유니언과 마찬가지롤 유니언이 가능하다. 

#### 3.1. 유추된 객체 타입 유니언
변수에 여러 객체 타입 중 하나가 될 수 있는 초깃값이 주어지면 타입스크립트는 해당 타입을 객체 타입 유니언으로 유추한다. 

```
const poem =  Math.random() > 0.5
            ? { name: "The Double Image", pages: 7 }
            : { name: "Her Kind", rhymes: true };

```

이때 주의할 점은 아래와 같이, 객체 타입에 정의된 각각의 가능한 속성은 초깃값이 없는 선택적 타입(?)으로 각 객체 타입의 구성 속성으로 주어진다는 점이다. 

```
//  {
//    name : string;
//    pages : number;
//    rhymes? : undefined;
//  } 
//  |
//  {
//    name : string;
//    pages? : number;
//    rhymes : boolean;       
//  }
// 위의 유니언 타입이 해당 변수의 타입

poem.name; // string
poem.pages; // number | undefined
poem.rhymes; // booleans | undefined

```


#### 3.2. 명시된 객체 타입 유니언

객체 타입을 더 명확하게 정의하기 위해서, 객체 타입의 조합을 **명시** 할 수  있다.

코드는 늘어나지만 객체 타입을 더 많이 제어할 수 있다. 

```
type PoemWithPages = {
    name : string;
    pages : number;
};

type PoemWithRhymes = {
    name : string;
    rhymes : boolean;
};

type Poem = PoemWithPages | PoemWithRhymes ;
// 명시적으로 객체 타입의 조합을 명시


const poem : Poem = Math.random() > 0.5 
            ? { name : "test" ,pages :7}
            : { name : "test2", rhymes: true};

poem.name; // OK : PoemWithPages와 PoemWithRhymes에 모두 존재

poem.pages; // Error : PoemWithRhymes에는 존재 x
poem.rhymes; // Error : PoemWithPages에는 존재 x

```
앞서 유추된 객체 타입 유니언과는 달리, 명시된 객체 타입 유니언은 위와 같이 잠재적으로 존재하지 않는 객체의 멤버에 대한 접근을 제한하기 때문에, 의도치 않은 동작 등을 사전에 막을 수 있다. 

(값이 여러 타입 중 하나일 경우, 모든 타입에 존재하지 않는 속성이 객체에 존재할 거라 보장하지 않는다.) 


유니언 타입 상의 모든 타입에 존재하지 않는 속성 (특정 타입에만 존재하는 속성)에 접근하기 위해서는 리터럴이나 원시 타입의 경우와 마찬가지로 객체 타입 유니언도 내로잉을 수행해야 한다. 

#### 3.3. 객체 타입 내로잉

타입스크립트의 타입 검사기는, 유니언에서 특정 타입의 **특정 속성이 포함된 경우에만 코드 영역을 실행할 수 있음**을 알게 되면, 값의 타입을 해당 속성을 포함하는 구성 요소로만 좁힌다. 

즉, 코드에서 객체의 형태를 확인하고 나서 타입 내로잉이 객체에 적용된다.

```
if("pages" in poem) {
    console.log(poem.pages); // PoemWithPages로 내로잉됨
}
else {
    console.log(poem.rhymes) // PoemWithRhymes로 내로잉됨
}

```

주의할 점은 다음과 같은 참 여부 확인 방식은 허용되지 않는다는 것이다.
```
if(poem.pages){ // Error 
    ...
}
```
이는 비록 타입 가드의 형태로 사용하더라도 존재하지 않을 수 있는 객체의 속성에 접근하려고 시도하면 타입 오류로 간주됨을 나타낸다. 


#### 3.4. 판별된 유니언

위에서 타입 가드의 형태라도, 존재하지 않을 수 있는 객체의 속성에 접근하려고 시도하면 타입 오류로 간주될 수 있는 문제가 있었다. 

이러한 문제를 해결할 수 있는 하나의 방법이 바로 객체의 속성에 객체의 타입을 나타내도록 명시하는 판별된 유니언의 형태이다. 

이때 객체의 타입을 가리키는 이러한 속성을 판별값이라고 부르며, 이러한 판별값을 이용해 타입 내로잉을 다음과 같이 수행할 수 있다. 

```
type PoemWithPages = {
    name : string;
    pages : number;
    type : 'pages;
};

type PoemWithRhymes = {
    name : string;
    rhymes : boolean;
    type : 'rhymes';
};

type Poem = PoemWithPages | PoemWithRhymes ;


const poem : Poem = Math.random() > 0.5 
            ? { name : "test" , pages : 7, type : 'pages'}
            : { name : "test2", rhymes: true, type :'rhymes'};

poem.type // 'pages' | 'rhymes'
poem.pages // Error 

if(poem.type === "pages"){
    poem.pages // OK
}
else {
    poem.rhymes // OK
}

```

## 4. 교차 타입

타입스크립트의 & 교차 타입 연산자는 여러 타입을 동시에 나타낼 수 있다. 
교차 타입은 일반적으로 여러 기존 객체 타입을 별칭 객체 타입으로 결합, 새로운 타입을 생성한다. 

```
type ArtWork = {
    genre : string;
    name : string;
};

type Writing = { 
    pages : number;
    name : string;
};

type WrittenArt = ArtWork & Writing;

```
위의 WrittenArt 교차 타입은 결과적으로 다음과 같은 형태이다. 

```
{
    genre : string;
    name : string;
    pages : number;
};
```

교차 타입은 유니언 타입과 결합이 가능하다.
```
type ShortPoem= { author: string } & (
        | { kigo: string; type: "haiku"; } 
        | {meter: number; type: "villanelle"; }
);

// Ok
const morningGlory: ShortPoem = {
    author: "Fukuda Chiyo-ni",
    kigo: "Morning Glory",
    type: "haiku",
};

```

#### 4.1. 교차 타입의 위험성

위와 같은 교차 타입은 유용한지만, 타입스크립트 컴파일러를 혼동 시킬 수 있다. 

##### 긴 할당 가능성 오류
유니언 타입과 결합하는 것처럼 복잡한 교차 타입을 만들게 되면 할당 가능성 오류 메시지가 복잡해진다. 복잡할수록 타입 검사기의 메시지가 이해하기 어려운 것.

따라서 별칭 타입의 형태로 타입을 분할하여 코드를 작성하는 편이 가독성이 더 좋다.

##### never
교차 타입은 잘못 사용하기 쉽고 불가능한 타입을 생성하는 경우도 종종 있다. 

원시 타입의 값은 동시에 여러 타입이 될 수 없기 때문에, 교차 타입의 구성요소로 결합할 수 없다. 

두 개의 원시 타입을 교차하려고 시도하면 never 타입이 된다.

never 타입은 프로그래밍 언어에서 bottom 타입 또는 empty 타입을 뜻하는데, bottom 타입은 값을 가질 수 없고, 참조할 수 없는 타입으로 해당 타입에는 어떤 타입도 제공할 수 없다. 



