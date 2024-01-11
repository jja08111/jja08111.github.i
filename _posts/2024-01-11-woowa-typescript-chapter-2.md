---
 title: "우아한 타입스크립트 with 리액트 - 2장 타입"
 date: 2024-01-11 11:00:00 +0900
 categories: typescript
 tags:
   - typescript
   - woowahan
---

# 타입이란

자바스크립트는 다음과 같은 자료형을 정의한다.

- undefined
- null
- Boolean(불리언)
- String(문장열)
- Symbol(심볼)
- Numeric(Number와 BigInt)
- Object

자료형이란 변수에 저장할 수 있는 값의 종류이며, 자료형마다 메모리를 차지하는 공간이 정해져있다.
예를 들어 메모리에 숫자 타입이 할당된 경우 자바스크립트 엔진이 값을 숫자로 인식하여 8바이트 단위로 메모리 공간에 저장된 값을 읽어온다.

자바스크립트에도 타입은 존재한다. 다만 컴파일 이전에는 타입이 정해지지 않고 런타임에 결정된다. 
타입을 결정하는 시점에 따라 정적 타입과 동적 타입으로 분류할 수 있다.

정적 타입 시스템에서는 모든 변수의 타입이 컴파일 타임에 결정된다. 이와 다르게 동적 타입 시스템에서는 변수 타입이 런타임에 결정된다.
이러한 부분은 개발자가 코드를 작성하는 때에는 확실히 어떤 타입인지 결정되지 않아 불안감을 제공한다.

타입스크립트는 아래와 같이 잘못된 타입으로 인자를 전달하는 경우 컴파일 타임에 오류를 보여준다.

```typescript
function double(n: number) {
  return n * 2;
}

double(2);
double("2"); // Error: Argument of type 'string' is not assignable to parameter of type 'number'(2345)
```

**강타입과 약타입**

암묵적 타입 변환 여부에 따라 타입 시스템을 강타입과 약타입으로 분류할 수 있다. 
강타입 특징을 가진 언어는 서로 다른 타입을 갖는 값끼리 연산을 시도하면 컴파일러 혹은 인터프리터에서 에러가 발생한다.
이와 다르게 약타입 특징을 갖는 언어의 경우는 서로 다른 타입을 갖는 값끼리 연산을 시도하면 컴파일러 혹은 인터프리터가 내부적으로 판단하여 특정 값의 타입을 변환하여 연산을 수행한 후 값을 도출한다.

타입스크립트의 경우 강타입 특징을 가져 아래와 같은 연산을 수행하면 컴파일 오류를 보여준다.

```typescript
console.log("2" - 1) // Error: The left-hand side of an arithmetic operation must be of type 'any', 'number', 'bigint' or an enum type.
```

암묵적 변환은 개발자가 명시적으로 타입을 변환하지 않아도 다른 데이터 타입끼리 연산을 진행할 수 있는 편리함을 제공하지만, 개발자의 의도와 다르게 동작할 수 있기 때문에 예기치 못한 오류를 발생할 가능성도 높아진다.

**컴파일 방식**

보통 고수준->저수준으로 코드를 변환하는 과정을 컴파일이라고 부른다.

타입스크립트의 경우는 컴파일을 거쳐 타입들이 제거된 후 자바스크립트 파일을 생성한다. 그렇기 때문에 타입스크립트를 템플릿 언어 또는 확장 언어로 해석하는 의견도 있다. 

# 타입스크립트의 타입 시스템

타입스크립트는 아래와 같이 타입 애너테이션 방식을 사용할 수 있다.

```typescript
let isDone: boolean = false;
let decimal: number = 7;
let x: [string, number];
```

위의 예시에서 : type 선언부를 제거해도 코드가 정상적으로 동작한다. 이는 타입 추론 때문이다.

**구조적 타이핑**

타입스크립트가 다른 언어와 차이를 가진 부분이다. 다른 언어의 경우는 타입 이름을 가지고 구분하며, 컴파일 타임 이후에도 이러한 타입 이름은 남아있다. 
이것을 명목적으로 구체화한 타입 시스템이라 부른다.

타입스크립트에서는 구조로 타입을 구분한다. 이를 구조적 타이핑이라고 한다.

```typescript
interface Developer {
  faceValue: number;
}

interface BankNote {
  faceValue: number;
}

let developer: Developer = { faceValue: 52 };
let bankNote: BankNote = { faceValue: 10000 };

developer = bankNote; // OK
bankNote = developer; // OK
```

**구조적 서브타이핑**

타입스크립트에서 타입은 집합이다. 따라서 아래와 같이 string 또는 number인 타입이 가능하다.

```typescript
type stringOrNumber = string | number;
```

이처럼 타입스크립트의 타입 시스템을 지탱하는 개념이 구조적 서브타이핑이다.

구조적 서브타이핑이란 객체가 가지고 있는 속성을 바탕으로 타입을 구분하는 것이다. 이름이 다른 객체라도 속성이 동이랗다면 타입스크립트는 서로 호환이 가능한 동일한 타입으로 여긴다.

인터페이스 뿐만 아니라 클래스도 마찬가지이다. 아래의 코드를 살펴보자

```typescript
class Person {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}

class Developer {
  name: string;
  age: number;
  sleepTime: number;

  constructor(name: string, age: number, sleepTime: number) {
    this.name = name;
    this.age = age;
    this.sleepTime = sleepTime;
  }
}

function greet(person: Person) {
  console.log(`Hello, I'm ${person.name}`);
}

const developer = new Developer("zig", 20, 7);
greet(developer);
```

Developer 클래스가 Person 클래스를 상속받지 않아도 위 코드는 동작한다. Developer는 Persondl 가진 속성을 모두 가졌기 때문이다.

타입스크립트는 자바스크립트의 덕 타이핑 동작을 그대로 모델링한다. 덕 타이핑과 구조적 타이핑은 타입을 검사하는 시점에 차이가 있다.
덕 타이핑은 런타임에 타입을 검사하며 구조적 타이핑은 컴파일 타임에 타입체커가 타입을 검사한다.

**점진적 타입 확인**

타입스크립트는 점진적으로 타입을 확인하는 언어이다. 점진적 타입 검사란 컴파일 타임에 타입을 검사하면서 필요에 따라 타입 선언 생략을 허용하는 방식이다.

```typescript
function add(x, y) {
  return x + y;
}

// 위 코드는 아래와 같이 암시적으로 타입 변환이 일어난다.
function add(x: any, y: any): any;
```

타입스크립트는 자바스크립트의 슈퍼셋 언어이기 때문에 모든 자바스크립트는 타입스크립트라고 봐도 무방하다.

그러나 이러한 특징 때문에 타입스크립트의 타입 시스템은 정적 타입의 정확성을 100% 보장하지 않는다. 
때문에 가능하면 모든 타입을 명시하는 것이 타입스크립트의 안정성을 높일 수 있다.

**값 vs 타입**

값은 프로그램이 처리하기 위해 메모리에 저장되는 모든 데이터다. 문자열, 숫자, 변수, 객체 함수 등이 존재한다.
어떠한 식을 연산한 것으로 변수에 할당할 수 있다.

타입은 변수, 매개변수, 객체 속성 등에 : type 형태로 타입을 명시한다. 또는 type이나 interface 키워드로 타입을 정의할 수 있다.

값 공간과 타입 공간은 서로 충돌하지 않는다. 그 이유는 타입스크립트 문법인 type으로 선언한 내용은 자바스크립트 코드로 컴파일 되는 과정에서 지워지기 때문이다.

아래처럼 값과 타입 공간이 혼용될 수 있다.

```typescript
function email({
  person,
  subject,
  body // <-- 여기까지가 값
}: {
  person: Person;
  subject: string;
  body: string; // <-- 여기까지가 타입
})
```

클래스와 enum은 값과 티입 공간에 동시에 존재한다. 아래의 코드를 보면 변수명 me 뒤에 등장하는 Developer는 타입에 해당되지만, new 키워드 뒤의 Developer는 생성자 함수인 값에 해당한다.

```typescript
class Developer {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}

const me: Developer = new Developer("zig", "frontend");
```

정리하자면, 타입스크립트에서 어떠한 심볼이 값으로 사용된다는 것은 컴파일 이후 자바스크립트 코드에서도 해당 정보가 남아있다는 것을 의미한다. 
반면 타입으로 사용되는 요소는 컴파일 타임 이후 해당 정보가 사라진다.

**타입을 확인하는 방법**

typeof 연산자가 반환하는 타입은 자바스크립트의 7가지 기본 데이터 타입(Boolean, null, undefined, Number, BigInt, String, Symbol)과 Function(함수), 호스트객체 그리고 object 객체이다.

```typescript
typeof 2022; // "number"
typeof "hi"; // "string"
typeof {}; // "object"
```

위와 다르게 타입에서 사용된 typeof는 타입을 반환한다.

```typescript
type T1 = typeof person // T1의 타입은 Person
```

**타입 필터링, 타입 가드**

타입 필터링

```typescript
let error = unknown;

if (error instanceof Error) {
  showAlert(error.message);
} else {
  throw Error(error);
}
```

타입 단언

```typescript
const loadedText: unknown;

function validateInput(text: string) { /* ... */ }

validateInput(loadedText as string);
```

# 원시 타입

타입스크립트에서는 자바스크립트와 다르게 특정 타입을 지정한 변수에 해당 타입은 값만 할당할 수 있다.

boolean, undefined, null, number, bigint, string, symbol

# 객체 타입

앞서 언급한 7가지 원시 타입에 속하지 않는 값은 모두 객체 타입으로 분류할 수 있다.

object, `{}`, array, type과 interface, function

호출 시그니쳐는 함수 타입을 정의할 때 사용하는 문법이다. 매개변수와 반환 값을 명시하면 된다.

```typescript
type add = (a: number, b: number) => number;
```