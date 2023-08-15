---
title : "TypeScript 6"
excerpt : "함수"
categories :
  - TypeScript
tags:
  - TypeScript
  - Web
date:               2023-08-15 01:00:0 +0000
last_modified_at:   2023-08-15 01:00:0 +0000
---

## 1. 함수 매개 변수

```
function sing(song) {
    console.log("Singing : ${song}!");
}
```
위와 같은 자바스크립트 함수가 있다고 할 때, 코드만 보고서는 song이라는 매개변수가 어떤 타입인지를 파악하는 것은 불가능하다. 

해당 타입이 string인지, 재정의된 toString() 메서드를 가진 객체인지 아는 것은 불가능하기 때문이다.

즉, 명시적 타입 정보가 매개변수에 선언되지 않으면, 어떤 타입인지 코드만 보고 아는 것은 불가능에 가깝다. 

타입스크립트는 이를 any 타입으로 간주하며, 해당 매개변수의 타입은 무엇으로든 진화할 수 있다. 

변수에 사용한 타입 애노테이션 기능을 이러한 함수 매개변수에 사용하면, 이러한 문제를 해결할 수 있다. 

```
function sing(song : string){
    console.log('singing : ${song}');
}
```

위의 코드를 통해서, song이 정확히 어떤 타입인지 알 수 있고, 이를 참고해서 코드를 변경하거나 추가하는 것이 가능해진다. 

#### 1.1 필드 매개변수
자바스크립트의 경우, 함수에 정의된 인수의 수와 상관없이(!) 함수를 호출할 수 있었다. 

타입스크립트는 함수에 선언된 모든 매개변수가 필수라고 가정한다. 

함수가 잘못된 수의 인수로 호출되면, 타입스크립트는 타입 오류를 발생시킨다. 

이처럼 함수에 필수 매개변수를 제공하도록 강제하면, 예상되는 모든 인숫값을 함수 내에 존재하도록 만들기 때문에, 타입 안정성을 강화하는데 도움이 된다. 

cf) 매개변수는 인수로 받을 것으로 예상되는 함수의 선언을 말하고, 인수는 함수를 호출할 때 매개변수에 제공되는 값을 말한다. 

#### 1.2 선택적 매개변수

자바스크립의 경우, 명시된 함수의 매개변수가 제공되지 않으면 함수 내부의 인숫값은 undefined으로 기본값이 설정된다. 

함수에 매개변수를 제공할 필요가 없는 경우, undefined 값을 위해 의도적으로 사용되는 경우를 위해 이러한 기능이 필요한 경우가 있다. 

이를 위해, 타입스크립트는 선택적 객체 타입 속성과 유사하게, 매개변수의 타입 애너테이션 : 앞에 ?를 추가함으로써 선택적 매개변수를 나타내는 기능을 제공한다.


```
function sing(song : string, singer? : string){
    console.log('singing : ${song}');

    if(singer) {
        console.log('singer : ${singer}');
    }
}
```
이러한 선택적 매개변수는 함수호출에 항상 제공될 필요는 없으며, 선택적 매개변수에는 항상 **| undefined**가 유니언 타입으로 추가되어 있다. 

따라서 선택적 매개변수는 항상 암묵적으로 undefined가 될 수 있으며, 이러한 선택적 매개변수는 if문에서 내로잉된다. 

선택적 매개변수는 **| undefined**를 포함하는 유니언 타입의 매개변수와는 다르다. 

?로 표시된 선택적 매개변수가 아닌 매개변수는 값이 undefined일지라도 항상 제공되어야 한다. 


```
function sing(song : string, singer : string | undefined){
    console.log('singing : ${song}');

    if(singer) {
        console.log('singer : ${singer}');
    }
}

sing("test", undefined); // ok
sing("test") //Error : 선택적 매개변수가 아닌 이상 항상 값이 제공되어야
```

함수에서 사용되는 모든 선택적 매개변수는 마지막 매개변수이어야 한다. 
필수 매개변수 이전에 선택적 매개변수를 위치시키면 타입스크립트 구문 오류가 발생한다.

#### 1.3 기본 매개변수 

자바스크립트에서는 선택적 매개변수를 선언시, =를 통해서 기본값을 제공할 수 있다. 

즉, 선택적 매개변수에는 기본적으로 값이 제공되기 때문에 해당 타입스크립트 타입에는 암묵적으로 함수 내부에 **| undefined** 유니언 타입이 추가된다. 

매개변수에 기본값이 있고, 애너테이션이 없는 경우, 타입스크립트는 해당 기본값을 기반으로 매개변수 타입을 유추한다. 

```
function rateSong(song : string, rating = 0) {
    ...
};

rateSong("test"); //ok
rateSong("test",4); //ok
rateSong("test",undefined); //ok

```
위와 같은 코드에서, 기본 인숫값으로 0을 받은 rating은 number 타입으로 취급되지만, 함수를 호출하는 함수의 외부에서는 number | undefined로 취급된다.

#### 1.4 나머지 매개변수

자바스크립트의 경우와 마찬가지로, 타입스크립트도 임의의 수의 인수를 받을 수 있는 함수를 만들 수 있다. 

... 스프레드 연산자는 함수 선언의 마지막 매개변수에 위치하고, 해당 매개변수에서 시작해 함수에 전달된 나머지 인수가 모두 단일 배열에 저장되어야 함을 나타난다. 

타입스크립트는 해당 나머지 인수들이 어떤 타입인지 나타내기 위해서 **[]** 구문이 타입의 마지막에 추가된다.

```
function singAllTheSongs(singer : string, ...songs : string[]) {
    ...
};
```

## 2. 반환 타입
타입스크립트는 함수가 반환할 수 있는 모든 값을 이해하면, 함수가 반환하는 타입을 자체적으로 알아낸다. 

만약 함수에 다른 값을 가진 여러 개의 반환문을 포함하고 있다면, 타입스크립트는 반환 타입을 가능한 모든 반환타입의 조합(유니언)으로 유치한다. 

#### 2.1 명시적 반환 타입
초깃값이 명시되어있거나, 변하지 않는 변수에 꼭 타입애너테이션을 적지 않아도 되는 것처럼, 함수의 반환 타입을 꼭 명시적으로 적지 않아도 된다. 

그러나 다음과 같은 경우, 함수의 반환 타입을 명시적으로 선언하는 것이 매우 유용하다. 

* 가능한 반환값이 많은 함수가 항상 동일한 타입의 값을 반환하도록 강제
* 타입스크립트는 재귀함수의 반환 타입을 통해, 타입을 유추하는 것을 거부
* 수백 개 이상의 타입스크립트 파일이 있는 경우, 반환 타입의 명시가 타입스크립트의 타입 검사 속도를 높일 수 있다. 

다음은 명시적 반환 타입을 적는 예시이다.

```
function singSongsRecursive(songs: string[], count = 0) : number {
    return songs.length ? singSongsRecursive(songs.slice(1), count+1) : count;
}
```

arrow function expression의 경우, 다음과 같이 사용한다. 

```
const singSongsRecursive = (songs : string[], count=0) : number => songs.length? singSongsRecursive(songs.slice(1),count+1) :count;
```

## 3. 함수 타입
자바스크립트에서는 함수를 값으로 전달하는 것이 가능하다. 

타입스크립트에서 이를 위해서는 함수를 가지기 위한 매개변수 또는 변수의 타입을 선언하는 별도의 방법이 필요하다.

함수 타입 구문은 이러한 방법 중 하나이다. 
```
let test : ()=>string;
```
위는 test라는 변수의 타입이 매개변수는 없고, string 타입을 반환하는 함수임을 나타낸다. 

위와 마찬가지로, 함수의 매개변수에도 함수 타입 구문을 사용할 수 있다. 
```
function testFunc(test : (index :number) => string ) : string{
    ...
}
```
위에선 testFunc은 인숫값으로 number 타입의 값을 인숫값으로 받고, string 타입의 값을 반환하는 함수를 받는다.


#### 3.2 매개변수 타입 추론
타입스크립트는 선언된 타입의 위치에서 제공된 함수의 매개변수 타입을 유추할 수 있다. 

```
let singer : (song : string) => string;
singer = function(song) {
    return song;
}
```
위와 같이 선언된 타입의 위치에서 함수의 매개변수 타입을 유추하기 때문에, 실제 함수를 할당할 때, 매개변수의 타입을 적지 않아도 된다.

함수를 매개변수로 갖는 함수에 인수로 전달된 함수의 경우에도 마찬가지이다. 

```
const songs = ["tst","test","test2"];

songs.forEach((song,index)=>{
    console.log('${song} is at index ${index}');
});
```
위의 song과 index 매개변수는 타입스크립트에 따라 각각 string과 number로 유추된다. 

#### 3.3 함수 타입 별칭 

함수 타입에도 타입 별칭 기능을 사용할 수 있다. 
```
type StringToNumber = (input:string) => number;
let stringToNumber : StringToNumber;
```
위에서 정의한 함수 타입 별칭의 경우, 다른 함수의 매개변수 타입으로도 지정할 수 있다. 

이와 같은 타입 별칭 기능을 사용하면, 함수 타입의 경우, 반복적으로 적는 매개변수의 타입과 반환타입의 양을 줄일 수 잇다. 

## 4. 그 외 반환 타입

#### 4.1 void 반환 타입
일부 함수의 경우 어떤 값도 반환하지 않는다. (return문이 없는 함수, 혹은 값을 반환하지 않는 return 문을 가진 경우)

타입스크립트는 void 키워드를 사용하여 반환값이 없는 함수의 반환 타입을 확인할 수 있으며, 값을 반환하지 않는 함수의 반환 타입을 적을 수 있다.

```
let singer : (song:string) => void;
function test(song: string | undefined) : void {
    ...
}
```

자바스크립트 함수의 경우 실제로 값이 반환되지 않으면, 기본으로 모두 undefined를 반환하지만, void의 경우 undefined와 동일하지 않다. 

void는 함수의 반환 타입이 무시된다는 것을 의미하고, undefined는 반환되는 리터럴 값이다.
```
function returnVoid() :void{
    return;
}

let test : string | undefined;
test = returnVoid() // Error : void를 string | undefined에 할당 할 수는 없다.

```
이러한 undefined와 void를 구분해서 사용하면 매우 유용한 경우가 많다. 

특히 void를 반환하도록 선언된 타입 위치에 전달된 함수가 반환된 모든 값을 무시하도록 설정할 때 유용하다. 

(인수로 전달된 함수가 반환하는 모든 값을 무시하도록 설정할때 유용하다.)

```
const records : string [] = [];
function saveRecords(newRecords : string[]) {
    newRecords.forEach(record => records.push(record));
}

saveRecords(['32','trest']);
```

배열의 내장 forEach 메서드는 void를 반환하는 callback함수를 받는다. 
forEach에 전달되는 함수는 모든 타입의 값을 반환할 수 있다.

그러나 forEach는 자체적으로 void를 반환하는 함수를 받는다고 선언되어 있으므로, 이러한 반환값들은 모두 무시된다. 

void 타입은 자바스크립트가 아닌 함수의 반환 타입을 선언하는 데 사용하는 타입스크립트 키워드이다. 

void 타입은 함수의 반환값이 자체적으로 반환될 수 있는 값도 아니고, 사용하기 위한 타입도 아니다. 

#### 4.2 never 반환 타입

never 반환 함수는 (의도적으로) 항상 오류를 발생시키거나 무한 루프를 실행하는 함수이다. 

함수가 절대 반환되지 않도록 의도하려면 명시적으로 never 키워드를 반환 타입 위치에 적으면 된다. 

```
function fail(message: string) :never {
    throw new Error('Invariant Failure : ${message}');
}
function workWithUnsafeParam(param : unknown) {
    if (typeof param != "string") {
        fail('param should be string');
    }
    param.toUpperCase();
}
```

위와 같이 fail함수는 오류만 발생시키고, 반환은 하지 않는다. 

cf) void는 아무 값도 반환하지 않는 함수를 위한 것이고, never는 절대 반환하지 않는 함수 (에러 발생 등)을 위한 것이다.

## 5. 함수 오버로드

타입스크립트는 여타의 다른 프로그래밍 언어와 같이 함수 오버로딩을 지원한다.

다만 다른 프로그래밍 언어와는 조금 상이한 방식으로 동작한다. 

타입스크립트의 함수 오버로딩은 오버로드 시그니처와 구현 시그니처라고 불리는 타입스크립트 구문으로 표현할 수 있다. 

오버로드 시그니처는 하나의 최종 구현 시그니처와 그 함수의 본문 앞에 서로 다른 버전의 함수 이름, 매개변수, 반환 타입을 여러 번 선언한다. 

```
function createDate(timestamp : number) : Date;
function createDate(month : number, day: number, year : number) : Date;
function createDate(monthOrTimeStamp : number, day? : number, year? : number) {
    return day === undefined || year === undefined 
            ? new Date(monthOrTimeStamp)
            : new Date(year, monthOrTimeStamp, day);
}
```
위의 예시에서 위의 두 줄은 오버로드 시그니처이고, 아래 마지막 함수는 구현 시그니처 코드이다. 

타입스크립트를 컴파일해서 자바스크립트로 출력 시, 오버로드 시그니처는 지워지고 다음과 같이 컴파일된다. 

```
function createDate(monthOrTimestamp, day, year) {
    return day === undefined || year === undefined
            ? new Date(monthOrTimestamp)
            : new Date(year, monthOrTimeStamp, day);
}
```

다른 프로그래밍 언어의 함수 오버로딩의 경우, 함수의 반환 타입, 매개변수 타입 등을 다르게 하고, 심지어 내부 구현 방식도 아예 다르게 만들 수 있다. (컴파일 시점에 컴파일러가 적절한 함수를 선택해서 빌드하는 방식으로 동작하는 것 같다.)

반면, 타입스크립트의 함수 오버로딩은 구현 시그니처의 반환 타입과 내부 구현에 종속적이다. 
(반대로 말하면, 구현 시그니처는 모든 오버로드 시그니처와 호환이 가능해야 한다. )

결국, 매개변수의 타입에만 선택 가능성을 준다는 것인데, 이는 결국 매개변수 타입을 유니언을 사용하는 방식과 동일하다. (사용 안하는 매개변수의 경우, undefined와 유니언을 하면 된다.)
