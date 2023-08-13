---
title: TypeScript 고급 타입1
date: 2023-08-13 00:00:30 +0900
categories: [TypeScript]
tags: [study, TypeScript]     # TAG names should always be lowercase
published: true
---
# 교차 타입(Intersection Types)

```tsx
function extend<First, Second>(first: First, second: Second): First & Second {
    const result: Partial<First & Second> = {};
    for (const prop in first) {
        if (first.hasOwnProperty(prop)) {
            (result as First)[prop] = first[prop];
        }
    }
    for (const prop in second) {
        if (second.hasOwnProperty(prop)) {
            (result as Second)[prop] = second[prop];
        }
    }
    return result as First & Second;
}

class Person {
    constructor(public name: string) { }
}

interface Loggable {
    log(name: string): void;
}

class ConsoleLogger implements Loggable {
    log(name) {
        console.log(`Hello, I'm ${name}.`);
    }
}

const jim = extend(new Person('Jim'), ConsoleLogger.prototype);
jim.log(jim.name);
```

`Partial<First & Second>`

- 교차타입은 여러 타입을 하나로 연결한다. 기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입을 얻을 수 있다.
- 예를 들어, `Person & Serializable & Loggable`은 `Person` *과* `Serializable` *그리고* `Loggable`이다. 즉, 이 타입의 객체는 세 가지 타입의 모든 멤버를 갖게 됩니다.
- 기존 객체-지향 틀과는 맞지 않는 믹스인이나 다른 컨셉들에서 교차 타입이 사용되는 것을 볼 수 있다.

# 유니언 타입(Union Types)

```tsx
/**
 * 문자열을 받고 왼쪽에 "padding"을 추가합니다.
 * 만약 'padding'이 문자열이라면, 'padding'은 왼쪽에 더해질 것입니다.
 * 만약 'padding'이 숫자라면, 그 숫자만큼의 공백이 왼쪽에 더해질 것입니다.
 */
function padLeft(value: string, padding: any) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}

padLeft("Hello world", 4); // "    Hello world"를 반환합니다.
```

`padLeft`의 문제는 매개변수 `padding`이 `any` 타입으로 되어있다는 것. 

즉, `숫자`나 `문자열` 둘 다 아닌 인수로 호출할 수 있다는 것이고, TypeScript는 이를 괜찮다고 받아들일 것이다.

```tsx
let indentedString = padLeft("Hello world", true); // 컴파일 타임에 통과되고, 런타임에 오류.
```

*유니언 타입*을 매개변수 `padding`에 사용할 수 있다.

```tsx
/**
 * 문자열을 받고 왼쪽에 "padding"을 추가합니다.
 * 만약 'padding'이 문자열이라면, 'padding'은 왼쪽에 더해질 것입니다.
 * 만약 'padding'이 숫자라면, 그 숫자만큼의 공백이 왼쪽에 더해질 것입니다.
 */
function padLeft(value: string, padding: string | number) {
    // ...
}

let indentedString = padLeft("Hello world", true); // 컴파일 중에 오류
```

유니언 타입은 값이 여러 타입 중 하나임을 설명한다. 세로 막대 (`|`)로 각 타입을 구분하여 `number | string | boolean`은 값의 타입이 `number`, `string` 혹은 `boolean`이 될 수 있음을 나타낸다.

유니언 타입을 값으로 가지고 있으면, 유니언에 있는 모든 타입에 공통인 멤버에만 접근할 수 있다.

```tsx
interface Bird {
    fly();
    layEggs();
}

interface Fish {
    swim();
    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // 성공
pet.swim();    // 오류
```

*확신할* 수 있는 것은 `A` *와* `B` 둘 다 가지고 있는 멤버가 있다는 것 뿐. 

이 예제에서, `Bird`는 `fly`를 멤버로 가지고 있습니다. `Bird | Fish`로 타입이 지정된 변수가 `fly` 메서드를 가지고 있는지 확신할 수 없습니다 만약 변수가 실제로 런타임에 `Fish`이면, `pet.fly()`를 호출하는 것은 오류이다..

# 타입 가드와 차별 타입 (Type Guards and Differentiating Types

유니언 타입은 값의 타입이 겹쳐질 수 있는 상황을 모델링하는데 유용하다. 

`Fish`가 있는지 구체적으로 알고 싶을 때, 무슨일이 벌어질까? JavaScript에서 가능한 두 값을 구분하는 흔한 방법은 멤버의 존재를 검사하는 것. 앞에서 말했듯이, 유니언 타입의 모든 구성 성분을 가지고 있다고 보장되는 멤버에만 접근할 수 있다.

```tsx
let pet = getSmallPet();

// 이렇게 각 프로퍼티들에 접근하는 것은 오류를 발생시킵니다
if (pet.swim) {
    pet.swim();
}
else if (pet.fly) {
    pet.fly();
}
```

같은 코드를 동작하게 하려면, 타입 단언을 사용해야 합니다:

```tsx
let pet = getSmallPet();

if ((pet as Fish).swim) {
    (pet as Fish).swim();
} else if ((pet as Bird).fly) {
    (pet as Bird).fly();
}
```

# 사용자-정의 타입 가드 (User-Defined Type Guards

타입 가드는 스코프 안에서의 타입을 보장하는 런타임 검사를 수행한다는 표현식이다.

## 타입 서술어 사용하기 (Using type predicates) 

타입 가드를 정의하기 위해, 반환 타입이 *타입 서술어*인 함수를 정의만 하면 된다.

```tsx
function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
}
```

`pet is Fish`는 이 예제에서의 타입 서술어다. 

서술어는 `parameterName is Type` 형태이고, `parameterName`는 반드시 현재 함수 시그니처의 매개변수 이름이어야 한다.

`isFish`가 변수와 함께 호출될 때마다, TypeScript는 기존 타입과 호환된다면 그 변수를 특정 타입으로 *제한*할 것.

```tsx
// 이제 'swim'과 'fly'에 대한 모든 호출은 허용됩니다

if (isFish(pet)) {
    pet.swim();
}
else {
    pet.fly();
}
```

## in` 연산자 사용하기 (Using the `in` operator)

`in` 연산자는 타입을 좁히는 표현으로 작용한다.

`n in x` 표현에서, `n`은 문자열 리터럴 혹은 문자열 리터럴 타입이고 `x`는 유니언 타입. 

"true" 분기에서는 선택적 혹은 필수 프로퍼티 `n`을 가지는 타입으로 좁히고, "false" 분기에서는 선택적 혹은 누락된 프로퍼티 `n`을 가지는 타입으로 좁혀진다.

```tsx
function move(pet: Fish | Bird) {
    if ("swim" in pet) {
        return pet.swim();
    }
    return pet.fly();
}
```

## typeof` 타입 가드 (`typeof` type guards)

유니언 타입을 사용하는 버전의 `padLeft` 코드를 작성

TypeScript는 `typeof`를 타입 가드로 인식하기 때문에 `typeof x === "number"`를 함수로 추상할 필요가 없다. 즉 타입 검사를 인라인으로 작성할 수 있다.

```tsx
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}

```

*`typeof` 타입 가드*는 두 가지 다른 형식인 `typeof v === "typename"` 와 `typeof v !== "typename"`이 있다. 여기서 `typename`은 `"number"`, `"string"`, `"boolean"` 그리고 `"symbol"`이여야 한다.

TypeScript에서 위에 없는 다른 문자열과 비교하는 것을 막지는 않지만, 타입 가드의 표현식으로 인지되지 않다.

## **`instanceof` 타입 가드 (`instanceof` type guards)**

위의 `typeof` 타입 가드를 읽었고 JavaScript의 `instanceof` 연산자에 익숙하다면 이미 알고 있을 것입니다.

*`instanceof` 타입 가드* 는 생성자 함수를 사용하여 타입을 좁히는 방법이다. 

```tsx
interface Padder {
    getPaddingString(): string
}

class SpaceRepeatingPadder implements Padder {
    constructor(private numSpaces: number) { }
    getPaddingString() {
        return Array(this.numSpaces + 1).join(" ");
    }
}

class StringPadder implements Padder {
    constructor(private value: string) { }
    getPaddingString() {
        return this.value;
    }
}

function getRandomPadder() {
    return Math.random() < 0.5 ?
        new SpaceRepeatingPadder(4) :
        new StringPadder("  ");
}

// 타입은 'SpaceRepeatingPadder | StringPadder' 입니다
let padder: Padder = getRandomPadder();

if (padder instanceof SpaceRepeatingPadder) {
    padder; // 타입은 'SpaceRepeatingPadder'으로 좁혀집니다
}
if (padder instanceof StringPadder) {
    padder; // 타입은 'StringPadder'으로 좁혀집니다
}
```

`instanceof`의 오른쪽은 생성자 함수여야 하며, TypeScript는 다음과 같이 좁힌다

1. 함수의 `prototype` 프로퍼티 타입이 `any`가 아닌 경우
2. 타입의 생성자 시그니처에서 반환된 유니언 타입일 경우

위와 같은 순서대로 진행된다.

# 널러블 타입 (Nullable types)

TypeScript는 각각 값이 null과 undefined를 갖는 특수한 타입인 `null`과 `undefined`가 있다.
기본적으로, 타입 검사 시 `null`과 `undefined`를 아무것에나 할당할 수 있다고 간주한다. 

실제로 `null`과 `undefined`는 모든 타입에서 유효한 값. 

즉, 방지하고 싶어도 어떤 타입에 할당되는 것을 *방지할* 없다. 

이건 `--strictNullChecks` 플래그로 해결한다: 변수를 선언할 때, 자동으로 `null`이나 `undefined`를 포함하지 않는다. 유니언 타입을 사용하여 명시적으로 포함할 수 있습니다.

```tsx
let s = "foo";
s = null; // 오류, 'null'은 'string'에 할당할 수 없습니다
let sn: string | null = "bar";
sn = null; // 성공

sn = undefined; // 오류, 'undefined'는 'string | null'에 할당할 수 없습니다.
```

TypeScript는 JavaScript와 맞추기 위해 `null`과 `undefined`를 다르게 처리한다. `string | null`은 `string | undefined`와 `string | undefined | null`과는 다른 타입입니다.

TypeScript 3.7 이후부터는 널러블 타입을 간단하게 다룰 수 있게 optional chaining를 사용할 수 있다.

## 선택적 매개변수와 프로퍼티 (Optional parameters and properties)

- `-strictNullChecks`를 적용하면, 선택적 매개변수가 `| undefined`를 자동으로 추가합니다:

```tsx
function f(x: number, y?: number) {
    return x + (y || 0);
}
f(1, 2);
f(1);
f(1, undefined);
f(1, null); // 오류, 'null'은 'number | undefined'에 할당할 수 없습니다

```

선택적 프로퍼티도 마찬가지입니다:

```tsx
class C {
    a: number;
    b?: number;
}
let c = new C();
c.a = 12;
c.a = undefined; // 오류, 'undefined'는 'number'에 할당할 수 없습니다
c.b = 13;
c.b = undefined; // 성공
c.b = null; // 오류, 'null'은 'number | undefined'에 할당할 수 없습니다.
```

## 타입 가드와 타입 단언 (Type guards and type assertions)
널러블 타입이 유니언으로 구현되기 때문에, `null`을 제거하기 위해 타입 가드를 사용할 필요가 있다. 

```tsx
function f(sn: string | null): string {
    if (sn == null) {
        return "default";
    }
    else {
        return sn;
    }
}
```

간단한 연산자를 사용할 수도 있다.

```tsx
function f(sn: string | null): string {
    return sn || "default";
}
```

컴파일러가 `null`이나 `undefined`를 제거할 수 없는 경우, 타입 단언 연산자를 사용하여 수동으로 제거할 수 있다. 

구문은 `!`를 후위 표기하는 방법입니다: `identifier!`는 `null`과 `undefined`를 `identifier`의 타입에서 제거한다..

```tsx
function broken(name: string | null): string {
  function postfix(epithet: string) {
    return name.charAt(0) + '.  the ' + epithet; // 오류, 'name'은 아마도 null 입니다
  }
  name = name || "Bob";
  return postfix("great");
}

function fixed(name: string | null): string {
  function postfix(epithet: string) {
    return name!.charAt(0) + '.  the ' + epithet; // 성공
  }
  name = name || "Bob";
  return postfix("great");
}
```

# 타입 별칭 (Type Aliases)

타입 별칭은 타입의 새로운 이름을 만든다. 

타입 별칭은 때때로 인터페이스와 유사하지만, 원시 값, 유니언, 튜플 그리고 손으로 작성해야 하는 다른 타입의 이름을 지을 수 있다.

```tsx
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === "string") {
        return n;
    }
    else {
        return n();
    }
}
```

별칭은 실제로 새로운 타입을 만드는 것은 아니다 - 그 타입을 나타내는 새로운 *이름* 을 만드는 것. 

원시 값의 별칭을 짓는 것은 문서화의 형태로 사용할 수 있지만, 별로 유용하지 않다.

인터페이스처럼, 타입 별칭은 제네릭이 될 수 있다 - 타입 매개변수를 추가하고 별칭 선언의 오른쪽에 사용하면 된다:

```tsx
type Container<T> = { value: T };
```

프로퍼티 안에서 자기 자신을 참조하는 타입 별칭을 가질 수 있다:

```tsx
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}
```

교차 타입과 같이 사용

```tsx
type LinkedList<T> = T & { next: LinkedList<T> };

interface Person {
    name: string;
}

var people: LinkedList<Person>;
var s = people.name;
var s = people.next.name;
var s = people.next.next.name;
var s = people.next.next.next.name;
```

하지만, 타입 별칭을 선언의 오른쪽 이외에 사용하는 것은 불가능

```tsx
type Yikes = Array<Yikes>; // 오류
```

## 인터페이스 vs. 타입 별칭 (Interfaces vs. Type Aliases

한 가지 차이점은 인터페이스는 어디에서나 사용할 수 있는 새로운 이름을 만들 수 있다. 

타입 별칭은 새로운 이름을 만들지 못한다 

예를 들어, 오류 메시지는 별칭 이름을 사용하지 않는다. 

아래의 코드에서, 에디터에서 `interfaced`에 마우스를 올리면 `Interface`를 반환한다고 보여주지만 `aliased`는 객체 리터럴 타입을 반환한다고 보여준다.

```tsx
type Alias = { num: number }
interface Interface {
    num: number;
}
declare function aliased(arg: Alias): Alias;
declare function interfaced(arg: Interface): Interface;
```