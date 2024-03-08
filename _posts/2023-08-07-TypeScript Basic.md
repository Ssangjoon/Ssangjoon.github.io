---
title: TypeScript Basic
date: '2023-08-07 00:00:30 +0900'
categories:
  - TypeScript
tags:
  - study
  - TypeScript
published: false
---

## 타입

- 타입선언

```tsx
// 변수 foo는 string 타입이다.
let foo: string = 'hello';
```

| Type | JS | TS | Description |
| --- | --- | --- | --- |
| boolean | ◯ | ◯ | true와 false |
| null | ◯ | ◯ | 값이 없다는 것을 명시 |
| undefined | ◯ | ◯ | 값을 할당하지 않은 변수의 초기값 |
| number | ◯ | ◯ | 숫자(정수와 실수, Infinity, NaN) |
| string | ◯ | ◯ | 문자열 |
| symbol | ◯ | ◯ | 고유하고 수정 불가능한 데이터 타입이며 주로 객체 프로퍼티들의 식별자로 사용(ES6에서 추가) |
| object | ◯ | ◯ | 객체형(참조형) |
| array |  | ◯ | 배열 |
| tuple |  | ◯ | 고정된 요소수 만큼의 타입을 미리 선언후 배열을 표현 |
| enum |  | ◯ | 열거형. 숫자값 집합에 이름을 지정한 것이다. |
| any |  | ◯ | 타입 추론(type inference)할 수 없거나 타입 체크가 필요없는 변수에 사용. var 키워드로 선언한 변수와 같이 어떤 타입의 값이라도 할당 가능. |
| void |  | ◯ | 일반적으로 함수에서 반환값이 없을 경우 사용한다. |
| never |  | ◯ | 결코 발생하지 않는 값 |

## 타입 추론

- 만약 타입 선언을 생략하면 값이 할당되는 과정에서 동적으로 타입이 결정된다. 이를 타입 추론(Type Inference)이라 한다.
- TypeScript는 정적 타입 언어이므로 타입 추론으로 타입이 결정된 이후, 다른 타입의 값을 할당하면 에러가 발생한다.

```tsx
let foo = 123; // foo는 number 타입
foo = 'hi';    // error: Type '"hi"' is not assignable to type 'number'.
```

- 타입 선언을 생략하고 값도 할당하지 않아서 타입을 추론할 수 없으면 `any` 타입이 된다. `any` 타입의 변수는 자바스크립트의 var 키워드로 선언된 변수처럼 어떤 타입의 값도 재할당이 가능하다.
- 이는 TypeScript를 사용하는 장점을 없애기 때문에 사용하지 않는 편이 좋다.

```tsx
let foo; // let foo: any와 동치

foo = 'Hello';
console.log(typeof foo); // string

foo = true;
console.log(typeof foo); // boolean
```

## 타입 캐스팅

- 기존의 타입에서 다른 타입으로 타입 캐스팅하려면 as 키워드를 사용하거나 `<>` 연산자를 사용할 수 있다. 다음 예제를 살펴보자.

```tsx
const $input = document.querySelector('input["type="text"]');
// => $input: Element | null

const val = $input.value;
// TS2339: Property 'value' does not exist on type 'Element'.
```

- value 프로퍼티는 Element 타입의 하위 타입인 HTMLInputElement 타입에만 존재한다. 따라서 다음과 같이 타입 캐스팅이 필요하다.

```tsx
const $input = document.querySelector('input["type="text"]') as HTMLInputElement;
const val = $input.value;
```

- 또는 타입 캐스팅을 위해 as 키워드 대신 `<>` 연산자를 사용할 수도 있다.

```tsx
const $input = <HTMLInputElement>document.querySelector('input["type="text"]');
const val = $input.value;
```

## Class

- Vue.js 에서 사용을 지양하기 때문에 생략한다.

## interface

- 인터페이스는 일반적으로 **타입 체크를 위해 사용되며 변수, 함수, 클래스에 사용**할 수 있다.
- 인터페이스는 여러가지 타입을 갖는 프로퍼티로 이루어진 새로운 타입을 정의하는 것과 유사하다.
- 인터페이스에 선언된 프로퍼티 또는 메소드의 구현을 강제하여 일관성을 유지할 수 있도록 하는 것이다.
- ES6는 인터페이스를 지원하지 않지만 TypeScript는 인터페이스를 지원한다.
- 인터페이스는 프로퍼티와 메소드를 가질 수 있다는 점에서 클래스와 유사하나 직접 인스턴스를 생성할 수 없고 모든 메소드는 추상 메소드이다. 단, 추상 클래스의 추상 메소드와 달리 abstract 키워드를 사용하지 않는다.
- 변수와 인터페이스
    - 인터페이스는 변수의 타입으로 사용할 수 있다.
    - 이때 인터페이스를 타입으로 선언한 변수는 해당 인터페이스를 준수하여야 한다.
    
    ```tsx
    // 인터페이스의 정의
    interface Todo {
      id: number;
      content: string;
      completed: boolean;
    }
    
    // 변수 todo의 타입으로 Todo 인터페이스를 선언하였다.
    let todo: Todo;
    
    // 변수 todo는 Todo 인터페이스를 준수하여야 한다.
    todo = { id: 1, content: 'typescript', completed: false };
    ```
    
    - 인터페이스를 사용하여 함수 파라미터의 타입을 선언할 수 있다.
    - 이때 해당 함수에는 함수 파라미터의 타입으로 지정한 인터페이스를 준수하는 인수를 전달하여야 한다.
    - 함수에 객체를 전달할 때 복잡한 매개변수 체크가 필요없어서 매우 유용하다.
    
    ```tsx
    // 인터페이스의 정의
    interface Todo {
      id: number;
      content: string;
      completed: boolean;
    }
    
    let todos: Todo[] = [];
    
    // 파라미터 todo의 타입으로 Todo 인터페이스를 선언하였다.
    function addTodo(todo: Todo) {
      todos = [...todos, todo];
    }
    
    // 파라미터 todo는 Todo 인터페이스를 준수하여야 한다.
    const newTodo: Todo = { id: 1, content: 'typescript', completed: false };
    addTodo(newTodo);
    console.log(todos)
    // [ { id: 1, content: 'typescript', completed: false } ]
    ```
    
- 함수와 인터페이스
    - 인터페이스는 함수의 타입으로 사용할 수 있다.
    - 이때 함수의 인터페이스에는 타입이 선언된 파라미터 리스트와 리턴 타입을 정의한다.
    - 함수 인테페이스를 구현하는 함수는 인터페이스를 준수하여야 한다.
    
    ```tsx
    // 함수 인터페이스의 정의
    interface SquareFunc {
      (num: number): number;
    }
    
    // 함수 인테페이스를 구현하는 함수는 인터페이스를 준수하여야 한다.
    const squareFunc: SquareFunc = function (num: number) {
      return num * num;
    }
    
    console.log(squareFunc(10)); // 100
    ```
    
- 클래스와 인터페이스
    - 생략…
- 덕 타이핑(duck typing)
    - 생략…
- 선택적 프로퍼티
    - 인터페이스의 프로퍼티는 반드시 구현되어야 한다.
    - 하지만 인터페이스의 프로퍼티가 선택적으로 필요한 경우가 있을 수 있다.
    - 선택적 프로퍼티(Optional Property)는 프로퍼티명 뒤에 `?`를 붙이며 생략하여도 에러가 발생하지 않는다.
    
    ```tsx
    interface UserInfo {
      username: string;
      password: string;
      age?    : number;
      address?: string;
    }
    
    const userInfo: UserInfo = {
      username: 'ungmo2@gmail.com',
      password: '123456'
    }
    
    console.log(userInfo);
    ```
    
- 인터페이스 상속
    - 인터페이스는 extends 키워드를 사용하여 인터페이스 또는 클래스를 상속받을 수 있다.
    
    ```tsx
    interface Person {
      name: string;
      age?: number;
    }
    
    interface Student extends Person {
      grade: number;
    }
    
    const student: Student =  {
      name: 'Lee',
      age: 20,
      grade: 3
    }
    ```
    
    - 복수의 인터페이스를 상속받을 수도 있다.
    
    ```tsx
    interface Person {
      name: string;
      age?: number;
    }
    
    interface Developer {
      skills: string[];
    }
    
    interface WebDeveloper extends Person, Developer {}
    
    const webDeveloper: WebDeveloper =  {
      name: 'Lee',
      age: 20,
      skills: ['HTML', 'CSS', 'JavaScript']
    }
    ```
    
    - 인터페이스는 인터페이스 뿐만 아니라 클래스도 상속받을 수 있다. 단, 클래스의 모든 멤버(public, protected, private)가 상속되지만 구현까지 상속하지는 않는다.
    
    ```tsx
    class Person {
      constructor(public name: string, public age: number) {}
    }
    
    interface Developer extends Person {
      skills: string[];
    }
    
    const developer: Developer =  {
      name: 'Lee',
      age: 20,
      skills: ['HTML', 'CSS', 'JavaScript']
    }
    ```
    

```tsx
interface Todo{
	title: string;
	done: boolean;
}

...
data(){
	return {
		todoItems: [] as Todo[]
	}
}
```

## 타입 앨리어스

- 타입 앨리어스는 새로운 타입을 정의한다. 타입으로 사용할 수 있다는 점에서 타입 앨리어스는 인터페이스와 유사하다.
- 인터페이스는 아래와 같이 타입으로 사용할 수 있다.
    
    ```tsx
    interface Person {
      name: string,
      age?: number
    }
    
    // 빈 객체를 Person 타입으로 지정
    const person = {} as Person;
    person.name = 'Lee';
    person.age = 20;
    person.address = 'Seoul'; // Error
    ```
    
- 타입 앨리어스도 인터페이스와 마찬가지로 타입으로 사용할 수 있다.
    
    ```tsx
    // 타입 앨리어스
    type Person = {
      name: string,
      age?: number
    }
    
    // 빈 객체를 Person 타입으로 지정
    const person = {} as Person;
    person.name = 'Lee';
    person.age = 20;
    person.address = 'Seoul'; // Error
    ```
    
- 하지만 타입 앨리어스는 원시값, 유니온 타입, 튜플 등도 타입으로 지정할 수 있다.
    
    ```tsx
    // 문자열 리터럴로 타입 지정
    type Str = 'Lee';
    
    // 유니온 타입으로 타입 지정
    type Union = string | null;
    
    // 문자열 유니온 타입으로 타입 지정
    type Name = 'Lee' | 'Kim';
    
    // 숫자 리터럴 유니온 타입으로 타입 지정
    type Num = 1 | 2 | 3 | 4 | 5;
    
    // 객체 리터럴 유니온 타입으로 타입 지정
    type Obj = {a: 1} | {b: 2};
    
    // 함수 유니온 타입으로 타입 지정
    type Func = (() => string) | (() => void);
    
    // 인터페이스 유니온 타입으로 타입 지정
    type Shape = Square | Rectangle | Circle;
    
    // 튜플로 타입 지정
    type Tuple = [string, boolean];
    const t: Tuple = ['', '']; // Error
    ```
    
- 인터페이스는 extends 또는 implements될 수 있지만 타입 앨리어스는 extends 또는 implements될 수 없다.
- 즉, 상속을 통해 확장이 필요하다면 타입 앨리어스보다는 인터페이스가 유리하다.
- 하지만 인터페이스로 표현할 수 없거나 유니온 또는 튜플을 사용해야한다면 타입 앨리어스를 사용한는 편이 유리하다.

## **제네릭**

- TypeScript는정적 타입 언어이기 때문에 함수 또는 클래스를 정의하는 시점에 매개변수나 반환값의 타입을 선언하여야 한다.
- 그런데 함수 또는 클래스를 정의하는 시점에 매개변수나 반환값의 타입을 선언하기 어려운 경우가 있다.
- 아래의 예제를 살펴보자. FIFO(First In First Out) 구조로 데이터를 저장하는 큐를 표현한 것이다.
    
    ```tsx
    class Queue {
      protected data: any[] = [];
    
      push(item: any) {
        this.data.push(item);
      }
    
      pop() {
        return this.data.shift();
      }
    }
    
    const queue = new Queue();
    
    queue.push(0);
    queue.push('1'); // 의도하지 않은 실수!
    
    console.log(queue.pop().toFixed()); // 0
    console.log(queue.pop().toFixed()); // Runtime error
    ```
    
    - Queue 클래스의 data 프로퍼티는 any[] 타입이다. any[] 타입은 어떤 타입의 요소도 가질 수 있는 배열을 의미한다.
    - any[] 타입은 배열 요소의 타입이 모두 같지 않다는 문제를 가지게 된다. 위 예제의 경우 data 프로퍼티는 number 타입만을 포함하는 배열이라는 기대 하에 각 요소에 대해 [Number.prototype.toFixed](https://poiemaweb.com/js-number#36-numberprototypetofixed)를 사용하였다. 따라서 number 타입이 아닌 요소의 경우 런타임 에러가 발생한다.
    - 위와 같은 문제를 해결하기 위해 Queue 클래스를 상속하여 number 타입 전용 NumberQueue 클래스를 정의할 수 있다.
    
    ```tsx
    class Queue {
      protected data: any[] = [];
    
      push(item: any) {
        this.data.push(item);
      }
    
      pop() {
        return this.data.shift();
      }
    }
    
    // Queue 클래스를 상속하여 number 타입 전용 NumberQueue 클래스를 정의
    class NumberQueue extends Queue {
      // number 타입의 요소만을 push한다.
      push(item: number) {
        super.push(item);
      }
    
      pop(): number {
        return super.pop();
      }
    }
    
    const queue = new NumberQueue();
    
    queue.push(0);
    // 의도하지 않은 실수를 사전 검출 가능
    // error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'.
    // queue.push('1');
    queue.push(+'1'); // 실수를 사전 인지하고 수정할 수 있다
    
    console.log(queue.pop().toFixed()); // 0
    console.log(queue.pop().toFixed()); // 1
    ```
    
    - 이와 같이 number 타입 전용 NumberQueue 클래스를 정의하면 number 타입 이외의 요소가 추가(push)되었을 때, 아래와 같이 런타임 이전에 에러를 사전 감지할 수 있다.
    - 하지만 다양한 타입을 지원해야 한다면 타입 별로 클래스를 상속받아 추가해야 하므로 이 또한 좋은 방법은 아니다. 제네릭을 사용하여 이 문제를 해결하여 보자.
    
    ```tsx
    class Queue<T> {
      protected data: Array<T> = [];
      push(item: T) {
        this.data.push(item);
      }
      pop(): T | undefined {
        return this.data.shift();
      }
    }
    
    // number 전용 Queue
    const numberQueue = new Queue<number>();
    
    numberQueue.push(0);
    // numberQueue.push('1'); // 의도하지 않은 실수를 사전 검출 가능
    numberQueue.push(+'1');   // 실수를 사전 인지하고 수정할 수 있다
    
    // ?. => optional chaining
    // https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#optional-chaining
    console.log(numberQueue.pop()?.toFixed()); // 0
    console.log(numberQueue.pop()?.toFixed()); // 1
    console.log(numberQueue.pop()?.toFixed()); // undefined
    
    // string 전용 Queue
    const stringQueue = new Queue<string>();
    
    stringQueue.push('Hello');
    stringQueue.push('World');
    
    console.log(stringQueue.pop()?.toUpperCase()); // HELLO
    console.log(stringQueue.pop()?.toUpperCase()); // WORLD
    console.log(stringQueue.pop()?.toUpperCase()); // undefined
    
    // 커스텀 객체 전용 Queue
    const myQueue = new Queue<{name: string, age: number}>();
    myQueue.push({name: 'Lee', age: 10});
    myQueue.push({name: 'Kim', age: 20});
    
    console.log(myQueue.pop()); // { name: 'Lee', age: 10 }
    console.log(myQueue.pop()); // { name: 'Kim', age: 20 }
    console.log(myQueue.pop()); // undefined
    ```
    
    - **제네릭은 선언 시점이 아니라 생성 시점에 타입을 명시하여 하나의 타입만이 아닌 다양한 타입을 사용할 수 있도록 하는 기법이다. 한번의 선언으로 다양한 타입에 재사용이 가능하다는 장점이 있다.**
    - **T는 제네릭을 선언할 때 관용적으로 사용되는 식별자로 타입 파라미터(Type parameter)라 한다.** T는 Type의 약자로 반드시 T를 사용하여야 하는 것은 아니다.
    - 또한 함수에도 제네릭을 사용할 수 있다. 제네릭을 사용하면 하나의 타입만이 아닌 다양한 타입의 매개변수와 리턴값을 사용할 수 있다. 아래 예제를 살펴보자.
    
    ```tsx
    function reverse<T>(items: T[]): T[] {
      return items.reverse();
    }
    ```
    
    - reverse 함수는 인수의 타입에 의해 타입 매개변수가 결정된다. Reverse 함수는 다양한 타입의 요소로 구성된 배열을 인자로 전달받는다. 예를 들어 number 타입의 요소를 갖는 배열을 전달받으면 타입 매개변수는 number가 된다.
    
    ```tsx
    function reverse<T>(items: T[]): T[] {
      return items.reverse();
    }
    
    const arg = [1, 2, 3, 4, 5];
    // 인수에 의해 타입 매개변수가 결정된다.
    const reversed = reverse(arg);
    console.log(reversed); // [ 5, 4, 3, 2, 1 ]
    ```
    
    - 만약 {name: string} 타입의 요소를 갖는 배열을 전달받으면 타입 매개변수는 {name: string}가 된다.
    
    ```tsx
    function reverse<T>(items: T[]): T[] {
      return items.reverse();
    }
    
    const arg = [{ name: 'Lee' }, { name: 'Kim' }];
    // 인수에 의해 타입 매개변수가 결정된다.
    const reversed = reverse(arg);
    console.log(reversed); // [ { name: 'Kim' }, { name: 'Lee' } ]
    ```
