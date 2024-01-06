---
 title: "[Typescript] any vs unknown"
 date: 2024-01-06 11:00:00 +0900
 categories: typescript
 tags:
   - typescript
---

Typescript에는 any와 unknown 타입이 존재한다. 처음 이 둘을 마주했을때는 마냥 비슷한 줄 알았지만 사실은 아니었다.

결론부터 말하자면 any 대신 unknown을 사용하는 것이 안전하다. 그 이유를 알아보자.

# any

any는 타입체크를 전혀 하지 않고 메소드 호출, 프로퍼티 접근을 모두 할 수 있다.
집합으로 표현하자면 어떤 집합도 될 수 있고, 어떤 부분 집합되 될 수 있는 마스터 키 같은 녀석이다.
때문에 any를 남발하는 경우 런타임에 문제가 발생할 위험성이 높다.

아래의 코드를 보자. `buyBook` 함수의 매개변수로 book을 any 타입으로 선언했다.

```typescript
interface Book {
  title: string;
  price: number;
}

function buyBook(book: any) {
  console.log(`Buying the ${book.titel}! (${book.price} won)`);
}

buyBook({ title: "Nice Book", price: 12000 }); // 출력 결과: Buying the undefined! (12000 won)
```

실행을 해보니 엉뚱한 결과가 출력된다. 그 이유는 `title`을 `titel`로 잘못 작성했기 때문이다.

협업을 하는 경우 다른 사람의 코드를 이해하고 사용해야하는 경우가 많다. 이러한 any 타입의 남발은 협업을 할 때 어려움을 제공한다.
대부분의 타입들이 any로 선언되어있다면? 벌써부터 머리가 아프다. "이 값은 무엇을 담고 있는거지?", "이 함수는 무엇을 반환하는 거지?"
이럴때마다 코드를 역추적하며 해당 값이 어떤 타입인지 직접 찾아야한다. 이는 큰 시간 낭비이며 런타임 오류를 만들어내기 쉽다.

또한 변화에 취약하다. 만약 특정 타입의 프로퍼티 이름을 수정하는 경우 하나씩 찾아가며 수정을 진행해야한다. 실수로 하나를 놓치게 되는 경우 프로그램에 문제가 생긴다.

# unknown

unknown은 any와 다르다. unknown은 전체 집합이라고 표현할 수 있으며, 타입이 특정되지 않는다면 메소드와 프로퍼티 접근이 제한된다.

앞선 any 대신 예제를 unknown으로 바꿔보겠다. 그러자 컴파일러가 book은 unknown 타입이라 title, price 프로퍼티에 접근할 수 없다고 경고를 보여준다.

```typescript
interface Book {
  title: string;
  price: number;
}

function buyBook(book: unknown) {
  // compile error! 'book' is of type 'unknown'.
  console.log(`Buying the ${book.titel}! (${book.price} won)`);
}

buyBook({ title: "Nice Book", price: 12000 });
```

이러한 경고를 제거하기 위해서는 type-guard를 통해 타입 범위를 좁히는 방법이 있다. `typeof`, `instaceof`부터, 아래와 같이 `isBookType` 함수를 만들 수도 있다.
`isBookType`을 사용하니 컴파일러에서 titel이 아닌 title이라고 경고를 보내준다.

```typescript
function isBookType(arg: any): arg is Book {
  return arg.title !== undefined && arg.price !== undefined;
}

function buyBook(book: unknown) {
  if (isBookType(book)) {
    // Property 'titel' does not exist on type 'Book'. Did you mean 'title'?
    console.log(`Buying the ${book.titel}! (${book.price} won)`);
  }
}
```

그렇다면 언제 unknown이 쓰이게 될까? 대표적으로 외부 반환 값에 대해 검증이 필요한 경우가 있다. 예를 들면 외부 라이브러리의 반환 값이 any인 경우이다.
