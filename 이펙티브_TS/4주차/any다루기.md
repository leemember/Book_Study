
# 5장. any 다루기

타입스크립트의 타입 시스템은 선택적(Optional)이고 점진적 이기 때문에 정적이면서도 동적인 특성을 동시에 가진다.

- 프로그램 일부에서만 타입 시스템을 적용할 수 있다는 특성 덕분에 점진적인 마이그레이션이 가능하다. 
(JS → TS 전환)
- 마이그레이션을 할 때 코드의 일부분에 타입 체크를 `비활성화`시켜주는 `any` 타입이 중요한 역할을 한다.
- any를 현명하게 사용하는 방법을 익혀야, 효과적인 TS 코드를 작성할 수 있다.
    - any가 매우 강력한 힘을 가지므로 남용하게 될 소지가 높기 때문이다.

### 이번 챕터에서는 any의 장점은 살리면서 단점을 줄이는 방법들을 알아봅시다 ‼️

---

### any 타입은 가능한 좁은 범위에서만 사용하기

안좋은 Bad

```jsx
function f1() {
	const x: any = expressionReturningFoo();
	processBar(x);
}
```

좋은 GOOD

```jsx
function f1() {
	const x = expressionReturningFoo();
	processBar(x as any);
}
```

`x as any` 의 형태가 더 권장된다. 그 이유는 any 타입이 processBar 함수의 매개변수에서만 사용된 표현식이므로 다른 코드에는 영향을 미치지 않기 때문이다.

```jsx
function f1() {
	const x:any = hello();
	processBar(x);
	return x;
}

function g() {
	const foo = f1();
	foo.fooMethod();
}
```

g 함수 내에서 f1이 사용되므로 f1의 반환타입인 any 타입이 foo 타입에 영향을 미친다.

이렇게 함수에서 any를 반환하면 그 영향력은 프로젝트 전반에 전염병처럼 퍼진다.

반면에, any의 사용 범위를 좁게 제한하는 `x as any` 방식의 함수를 사용한다면 any 타입이 함수 바깥으로 영향을 미치지 않는다.

타입스크립트가 함수의 반환 타입을 추론할 수 있는 경우에도 함수의 반환 타입을 명시하는 것이 좋다.

함수의 반환 타입을 명시하면 any 타입이 함수 바깥으로 영향을 미치는 것을 방지할 수 있다.

```jsx
//@ts-ignore
processBar(x);
```

위와 같은 주석을 사용하면 any를 사용하지 않고 오류를 제거할 수 있다. `@ts-ignore` 를 사용한 다음 줄의 오류가 무시된다. 

안좋은예)

```jsx
const config:Config = {
	a:1,
	b:2,
	c: {
		key:value
	}
} as any; // 이렇게 하지 맙시다 !
```

객체 전체를 any로 단언하면 다른 속성들 역시 타입 체크가 되지 않는 부작용이 발생한다.

좋은예)

```jsx
const config:Config = {
	a:1,
	b:2,
	c: {
		key:value as any;
	}
}
```

### ✅ 요약

- 의도치 않은 타입 안전성의 손실을 피하기 위해 any의 사용 범위를 최소한으로 좁혀야 한다.
- 함수의 반환 타입이 any인 경우 타입 안정성이 나빠진다. 따라서 any 타입을 반환하면 절대 안된다.
- 강제로 타입 오류를 제거하려면 any 대신에 `@ts-ignore` 사용하는 것이 좋다.

### any를 구체적으로 변형하여 사용하기

- any는 자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입입니다.
    - 숫자, 문자열, 배열, 객체, 정규식, 함수, 클래스, DOM 엘리먼트, null, undefined 전부 포함
- any보다 더 구체적으로 표현할 수 있는 타입이 존재할 가능성이 높기 때문에 더 구체적인 타입을 찾아 타입 안전성을 높이도록 해야 한다.

안좋은예)

```jsx
function getLengthBad(array:any) {
	return array.length;
}
```

좋은예)

```jsx
function getLength(array:any[]) {
	return array.length;
}
```

이처럼 `any[]` 를 사용하는 함수가 더 좋은 함수다.

### ✅ 이유

1. 함수 내의 **array.length** 타입이 체크된다.
2. 함수의 반환 타입이 **`any`** 대신 **`number`**로 추론된다.
3. 함수 호출될 때 매개변수가 **배열인지 체크된다.**

배열이 아닌 값을 넣어서 실행해 보면, `getLength` 는 제대로 오류를 표시하지만 `getLengthBad` 는 오류를 잡아내지 못한다.

함수의 매개변수를 구체화할 때, 배열의 배열 형태라면 `any[][]` 처럼 선언하면 안된다.

그리고 함수의 매개변수가 객체이긴 하지만 값을 알 수 없다면 `{[key: stsring]: any}` 처럼 선언하면 된다.

### 함수 안으로 타입 단언문 감추기

내부 로직이 복잡해서 안전한 타입으로 구현하기 어려운 경우가 많다.

- 제대로 타입이 정의된 함수 안으로 타입 단언문을 감추는 것이 더 좋은 설계이다.
- 함수 내부에는 타입 단언을 사용하고 함수 외부로 드러나는 타입 정의를 정확히 명시하는 정도로 끝내자.
- 함수 캐싱은 리액트같은 프레임워크에서 실행 시간이 오래 걸리는 함수 호출을 개선하는 일반적인 기법이다.

```jsx
declare function cacheLast<T extends Function>(fn: T): T;
```

구현체는 위와 같다.

```jsx
declare function shallowEqual(a: any, b:any): boolean;

function cacheLast<T extands Function>(fn: T):T {
	let lastArgs: any[]|null = null;
	let lastResult: any;
}
```

어떤 함수든 캐싱할 수 있도록 래퍼 함수이다.

```jsx
return function(...args: any[]) { // -> 이 형식은 'T'형식에 할당할 수 없다.
	if(!lastArg || !shallowEqual(lastArgs, args) {
		lastResult = fn(...args);
		lastArg = args;
	}
	return lastResult;
}
```

- 타입스크립트는 반환문에 있는 함수와 원본 함수 T 타입이 어떤 관련이 있는지 알지 못해 오류가 발생
- 결과적으로 원본 함수 T 타입과 동일한 매개변수로 호출되고 반환값 역시 예상 결과가 되기 때문에 타입 단언문을 추가해서 오류를 제거하는 것이 큰 문제가 되지는 않는다ㅏ.

```jsx
function cacheLast<T extends Function>(fn: T):T {
	let lastArgs: any[]|null = null;
	let lastResult: any;
	return function(...args: any[]) {
		if(!lastArgs || !shallowEquls(lastArgs, args)) {
			lastResult = fn(...args);
			lastArgs = args;
		}
		return lastResult;
	} as unknown as T;
}
```

실제로 함수 실행해보면 잘 동작한다. 함수 내부에는 any가 꽤 많이 보이지만 타입 정의에는 any가 없기 때문에, chcheLast를 호출하는 쪽에서는 any가 사용됐는지 알지 못한다.

```jsx
declare function shallowObjectEqual<T extends object>(a: T, b: T):boolean;
```

객체 매개변수 a와 b가 동일한 키를 가진다는 보장이 없기 때문에 구현할 때는 주의해야 한다.

### ✅ 요약

- 타입 선언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도 한다.
- 불가피하게 사용해야 한다면, 정확한 정의를 가지는 함수 안으로 숨기도록 하자.

### any의 진화를 이해하기

타입스크립트에서 일반적으로 변수의 타입은 변수를 선언할 때 결정된다.

새로운 값이 추가되도록 확장할 수는 없다. 그러나 any 타입과 관련해서 예외인 경우가 존재한다.

```jsx
function range(start, limit){
	const out = [];
	for (let i = startl i < limit; i++) {
		out.push(i);
	}
	return out;
}
```

이 코드를 타입 스크립트로 변환하면 예상한 대로 정확히 동작한다.

```jsx
function range(start: number, limit:number){
	const out = [];
	for (let i = startl i < limit; i++) {
		out.push(i);
	}
	return out; // 반환 타입이 number[]로 추론됨.
}
```

out의 타입이 처음에는 any 타입 배열인 [] 로 초기화 되었는데, 마지막에는 number[] 로 추론되고 있다.

코드에 out이 등장하는 세 가지 위치를 조사해 보면 이유를 알 수 있다.

```jsx
function range(start: number, limit:number){
	const out = []; // 📍타입이 any[]
	for (let i = startl i < limit; i++) {
		out.push(i); // 📍out의 타입이 any[]
	}
	return out; // 📍타입이 number[]
}
```

out의 타입은 any[]로 선언되었지만 number 타입의 값을 넣는 순간부터 타입은 number[] 로 진화 한다.

타입의 진화는 타입 좁히기와 다르다. 배열에 다양한 타입의 요소를 넣으면 배열의 타입이 **확장되며 진화한다.**

```jsx
const result = []; // 타입 any[]
result.push('a');
result // 📍타입 string
result.push(1);
result // 📍타입 number
```

또한 조건문에서는 분기에 따라 타입이 변할 수도 있다.

```jsx
let val;
if(Math.random() < 0.5) {
	val = /hello/;
	val // 📍타입 RegExp
} else {
	val = 12;
	val // 📍타입 number
}
val // 📍타입 number|RegExp
```

변수의 초깃값이 null인 경우도 any의 진화가 일어난다. 보통은 `try/catch` 블록 안에서 변수를 할당하는 경우에 나타난다.

```jsx
let val = null;
try {
	somethingDangerous();
	val = 12;
	val // 타입이 number
} catch (e) {
	console.warn('alas!');
}
val // 타입이 number|null
```

any 타입의 진화는 `noImplicitAny`가 설정된 상태에서 변수의 타입이 암시적 any인 경우에만 일어난다. 그러나 다음처럼 명시적으로 any를 선언하면 타입이 그대로 유지된다.

```jsx
let val: any;
if(Math.random() < 0.5) {
	val = /hello/;
	val // 타입 any
} else {
	val = 12;
	val // 타입 any
}
val // 타입 any
```

any 타입의 진화는 암시적 any 타입에 어떤 값을 할당할 때만 발생한다. 그리고 어떤 변수가 암시적 any 상태일 때 값을 읽으려고 하면 오류가 발생한다. 암시적 any 타입은 함수 호출을 거쳐도 진화하지 않는다.

```jsx
function makeSquares(start: number, limit: number) {
	const out = [];
	range(start, limit).forEach(i => {
		out.push(i * i);
	})
	reteurn out;
}
```

- 루프로 순회하는 대신 배열의 map과 filter 메서드를 통해 단일 구문으로 배열을 생성하여 any 전체를 진화시키는 방법을 생각해볼 수 있다.
- any가 진화하는 방식은 일반적인 변수가 추론되는 원리와 동일하다.
- 진화한 배열의 타입이 `(string|number)[]` 라면, 원래 `number[]` 타입이어야 하지만 실수로 string이 섞여서 잘못 진화한 것일 수도 있다.
- 타입을 안전하게 지키기 위해서는 암시적 `any`를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 더 좋은 설계이다.

### ✅ 요약

- 일반적인 타입들은 정제되기만 하는 반면, 암시적 any와 any[] 타입은 진화할 수 있다.
    - 이러한 동작이 발생하는 코드를 인지하고 이해할 수 있어야 한다.
- any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 안전한 타입을 유지하는 방법이다.

### 모르는 타입의 값에는 `any` 대신 `unknown` 을 사용하기

```jsx
function parseYAML(yaml: string): any {
 //...
}
```

함수의 반환 타입으로 any를 사용하는 것은 좋지 않은 설계이다.

대신 `parseYAML` 를 호출한 곳에서 반환값을 원하는 타입으로 할당하는 것이 이상적이다.

```jsx
interface Book {
	name: string;
	author: string;
}

const book: Book = parseYAML(`
	name: Wuthering Heights;
	author: Emily Bronte
`);
```

함수 반환값에 타입 선언을 강제할 수 없기 때문에, 호출한 곳에서 타입 선언을 생략하게 되면 book 변수는 암시적 any 타입이 되고, 사용되는 곳마다 타입 오류가 발생하게 된다.

```jsx
const book: Book = parseYAML(`
	name: Wuthering Heights;
	author: Emily Bronte
`);

alert(book.title); // 오류 없음. 런타입에 undefined 경고
book('read'); // 오류 없음. 런타임에 TypeError: book은 함수가 아닙니다. 예외 발생
```

대신 parseYAML 이 `unknown` 타입을 반환하게 만드는 것이 더 안전하다.

```jsx
function safeParseYAML(yaml: string): unknown { ✅ 더 안전하다.
	return parseYAML(yaml);
}

const book = safeParseYAML(`
	name: The Tenant of Wildfell Hall
	author: Anne Bronte
`);

alert(book.title);
book("read");
```

`unknown` 타입을 이해하기 위해서는 할당 가능성의 관점에서 `any`를 생각해 볼 필요가 있다.

any가 강력하면서도 위험한 이유는 두 가지 특징이 있다.

1. 어떠한 타입이든 `any` 타입에 할당 가능
2. any 타입은 어떠한 타입으로도 할당 가능

- 타입을 값의 집합으로 생각하기의 관점에서, 한 집합은 다른 모든 집합의 부분 집합이면서 동시에 상위 집합이 될 수 없기 때문에, 분명히 any는 타입 시스템과 상충되는 면을 가지고 있다.
- 이러한 점이 any의 강력함의 원천이면서 동시에 문제를 일으키는 원인이 된다.
- 타입 체커는 집합 기반이기 때문에 any를 사용하면 타입 체커가 무용지물이 된다는 것을 주의해야 한다.
- unknown은 any 대신 쓸 수 있는 타입 시스템에 부합하는 타입이다.
- unknown 타입은 any의 첫 번째 속성을 만족하지만 두 번째 속성은 만족하지 않는다.
- never 타입은 unknown과 정반대이다. 첫 번째 속성은 만족하지 않지만 두 번째 속성은 만족한다.
- unknown 타입인 채로 값을 사용하면 오류가 발생한다. unknown 상태로 사용하려고 하면 오류가 발생하기 때문에 적절한 타입으로 변환하도록 강제할 수 있다.

```jsx
const book = safeParseYAML(`
	name: Villette
	author: Charlotte Bronte
`) as Book;
alert(book.title) // ---> Book 형식에 title 속성 없음.

book('read') // 이 식은 호출할 수 없다.
```

타입을 모르는 경우에는 unknown을 사용한다.

타입 단언문이 unknown에서 원하는 타입으로 변환하는 유일한 방버은 아니다.

instanceof를 체크한 후 unknown에서 원하는 타입으로 변환할 수 있다.

```jsx
function processValue(val: unknown) {
	if (val instanceof Date) {
		val // 타입이 Date
	}
}
```

타입 가드도 unknown에서 원하는 타입으로 변환할 수 있다.

```jsx
function isBook(val: unknown):val is Book {
	return (
		typeof(val) === 'object' && val !== null &&
		'name' in val && 'author' in val
	);
}

function processValue(val: unknown) {
	if(isBook(val)) {
		val; // 타입이 Book
	}
}
```

unknown 타입의 범위를 좁히기 위해서는 상당히 많은 노력이 필요하다. in 연산자에서 오류를 피하기 위해 먼저 val이 객체임을 확이해야 하고, `typeof null === 'object'` 이므로 별도로 val이 null 아님을 확인해야 한다.

가끔 `unknown` 대신 제너릭 매개변수가 사용되는 경우도 있다. 제너릭을 사용하기 위해 다음 코드처럼 `safeParseYAML` 함수를 선언할 수 있다.

```jsx
function safeParseYAML<T>(yaml: string):T {
	return parseYAML(yaml);
}
```

이 코드는 일반적으로 타입스크립트에서 좋은 코드는 아니다.

제너릭을 사용한 스타일은 타입 단언문과 달라 보이지만 기능적으로는 동일하다.

제너릭보다는 unknown을 반환하고 사용자가 직접 단언문을 사용하거나 원하는 대로 타입을 좁히도록 강제하는 것이 좋다.

```jsx
declare const foo: Foo;
let barAny = foo as any as Bar;
let barUnk = foo as unknown as Bar;
```

나중에 두 개의 단언문을 분리하는 리팩터링을 한다면 unknown 형태가 더 안전하다.

any의 경우는 분리되는 순간 그 영향력이 전염병처럼 퍼지게 된다.

그러나 unknown의 경우는 분리되는 즉시 오류를 발생하게 되므로 더 안전하다.

`unknown`에 대해서 설명한 것과 비슷한 방식으로 `object` 또는 `{ }` 를 사용하는 코드들이 존재한다.

`object` 또는 `{ }` 를 사용하는 방법 역시 `unknown`만큼 범위가 넓은 타입이지만, `unknown` 보다는 범위가 약간 좁다.

- `{ }` 타입은 `null` 과 `undefined`를 제외한 모든 값을 포함한다.
- `object` 타입은 모든 비기본형 타입으로 이루어진다. → 여기에는 true 또는 12 또는 ‘foo’ 가 포함되지 않지만 객체와 배열은 포함된다.

`unknown` 타입이 도입되기 전 까지는 { } 가 더 일반적으로 사용되었지만, 최근에는 { } 를 사용하는 경우가 꽤 드물다. `null`과 `undefined`가 불가능하다고 판단되는 경우만 unknown 대신 { } 를 사용하면 된다.

### ✅ 요약

- `unknown` 은 `any` 대신 사용할 수 있는 안전한 타입이다.
- 어떠한 값이 있지만 그 타입을 알지 못하는 경우 `unknown`을 사용하면 된다.
- 사용자가 타입 단언문이나 타입 체크를 사용하도록 강제 하려면 `unknown`을 사용하면 된다.
- `{}, object, unkonwn` 의 차이점을 이해 해야한다.

### `{}, object, unkonwn` 차이점 정리

| 타입 | 허용되는 값 | 주의할 점 |
| --- | --- | --- |
| `{}` | 모든 값 (`null`, `undefined` 포함) | 타입이 너무 넓어서 안전하지 않음 |
| `object` | 객체만 허용 (`null`, `undefined` 불가) | 객체 내부 구조를 알 수 없음 |
| `unknown` | 모든 값 허용 | 타입 검사를 해야만 조작 가능 (안전함) |

### 1. `{}` (빈 객체 타입)

- `{}`는 "빈 객체" 타입을 의미하지만, 실제로는 **모든 값**을 허용한다.
- `null`과 `undefined`도 허용하기 때문에 예상치 못한 동작이 발생할 수 있다.

```jsx
let a: {} = {}; // 가능
let b: {} = 42; // 가능
let c: {} = "hello"; // 가능
let d: {} = []; // 가능
let e: {} = null; // 가능 (주의)
let f: {} = undefined; // 가능 (주의)
```

👉 안전한 코드 작성을 원한다면 `unknown`을 사용하고, 객체를 제한적으로 허용하려면 `object`를 사용하자. `{}`는 지양하는 것이 좋다.

### 2. `object` (객체 타입)

- `object`는 **객체만 허용**하고, 원시 타입은 허용하지 않는다.
    - 원시타입 → string, number, boolean, null, undefined, bigint, symbol
    - 객체타입 → {}, [], function
- `null`과 `undefined`도 허용하지 않는다.

```jsx
let obj1: object = {}; // 가능
let obj2: object = { key: "value" }; // 가능
let obj3: object = []; // 가능
let obj4: object = new Date(); // 가능

let obj5: object = 42; // ❌ 에러 (number는 객체가 아님)
let obj6: object = "hello"; // ❌ 에러 (string은 객체가 아님)
let obj7: object = null; // ❌ 에러 (null은 객체가 아님)
let obj8: object = undefined; // ❌ 에러 (undefined는 객체가 아님)

```

- 객체만 허용하고, 원시 타입을 방지할 때 사용 가능.
- 하지만 객체의 프로퍼티 타입을 알 수 없기 때문에 정확한 타입 지정이 어려움.

### 3. `unknown` (알 수 없는 타입)

- `unknown`은 **어떤 값이든 할당 가능하지만, 직접 사용할 수 없음.**
- `any`와 달리, 타입 검사를 통과해야만 조작할 수 있다.

```tsx
let value: unknown;

value = 42; // 가능
value = "hello"; // 가능
value = {}; // 가능
value = null; // 가능
value = undefined; // 가능

let str: string = value; // ❌ 에러 (unknown은 직접 할당 불가능)
let num: number = value; // ❌ 에러
let obj: object = value; // ❌ 에러

// 타입 검사를 통해 사용해야 함
if (typeof value === "string") {
  let str2: string = value; // 가능
}
```

- 안전성을 높이기 위해 사용하며, 타입 검사를 통해서만 사용 가능.
- any 보다 타입을 엄격하게 관리할 수 있음.

### 몽키 패치보다는 안전한 타입 사용하기

- JS의 가장 유명한 특징 중 하나는 객체와 클래스에 임의의 속성을 추가할 수 있을 만큼 유연하다는 점이다.
- 객체에 속성을 추가할 수 있는 기능은 종종 웹 페이지에서 `window`나 `document`에 값을 할당하여 전역 변수를 만드는 데 사용한다.

```tsx
window.monkey = 'Tamarin';
document.monkey = 'Howler';
```

또는 DOM 엘리먼트에 데이터를 추가하기 위해서도 사용된다.

```tsx
const el = document.getElementById('colobus');
el.home = 'tree';
```

객체에 속성을 추가하는 코드 스타일은 특히 제이쿼리를 사용하는 코드에서 흔히 볼 수 있다.

심지어 내장 기능의 프로토타입에도 속성을 추가할 수 있다. 그런데 이상한 결과를 보일 때가 있다.

함수를 호출할 때마다 부작용 (side effect)를 고려해야한다.

타입스크립트까지 더하면 또 다른 문제가 발생한다. 타입 체커는 document와 HTMLElement 의 내장 속성에 대해서는 알고 있지만, 임의로 추가한 속성에 대해서는 알지 못한다.

```tsx
document.monkey = 'Tamarin'; // ---> Document 유형에 monkey 속성이 없다.
```

이 오유를 해결하기 위해서 가장 간단한 방법은 `any` 단언문을 사용하는 것이다.

```tsx
(document as any).monkey = 'Tamarin'; // 정상
```

타입 체커는 통과해도 단점이 있다.

any를 사용함으로써 타입 안전성을 상실하고 언어 서비스를 사용할 수 없게 된다는 것이다.

```tsx
(document as any).monkey = 'Tamarin'; // 정상, 오타
(document as any).monkey = /Tamarin/; // 정상, 잘못된 타입
```

최선의 해결책은 document 또는 DOM으로부터 데이터를 분리하는 것이다.

분리할 수 없는 경우 두 가지 차선책이 존재한다.

1. interface의 특수 기능 중 하나인 보강을 사용하는 것이다.

```tsx
interface Document {
	monkey: string;
}

document.monkey = 'Tamarin' // 정상
```

보강을 사용한 방법이 any 보다 나은 점은 이와 같다.

- 타입이 더 안전하다. 타입 체커는 오타나 잘못된 타입의 할당을 오류로 표시한다.
- 속성에 주석을 붙일 수 있다.
- 속성에 자동완성을 사용할 수 있다.
- 몽키 패치가 어떤 부분에 적용됐는지 정확한 기록이 남는다.

그리고 모듈의 관점에서 (타입스크립트 파일이 import / export를 사용하는 경우), 제대로 동작하게 하려면 global 선언을 추가해야 한다.

```tsx
export {};
declare global {
	interface Document {
		monkey: string;
	}
}

document.monkey = 'Tamarin' // 정상
```

보강을 사용할 때 주의할 점은 모듈 영역과 관련이 있다.

보강은 전역적으로 적용되기 때문에, 코드의 다른 부분이나 라이브러리로부터 분리할 수 없다.

애플리케이션이 실행되는 동안 속성을 할당하면 실행 시점에서 보강을 적용할 방법이 없다.

웹페이지 내의 HTML 엘리먼트를 조작할 때, 어떤 엘리먼트는 속성이 있고 어떤 엘리먼트는 속성이 없는 경우 문제가 된다.

이러한 이유로 속성을 string | undefined로 선언할 수도 있다.

2번째, 더 구체적인 타입 단언문을 사용하는 것이다.

```tsx
interface MonkeyDocument extends Document {
	monkey: string;
}

(document as MonkeyDocument).monkey = 'Macaque';
```

`MonkeyDocument`는 `Document`를 확장하기 때문에 타입 단언문은 정상이며 할당문의 타입은 안전하다.

또한 Document 타입을 건드리지 않고 별도로 확장하는 새로운 타입을 도입했기 때문에 모듈 영역 문제도 해결할 수 있다.

따라서 몽키 패치 된 속성을 참조하는 경우에만 단언문을 사용하거나 새로운 변수를 도입하면 된다.

그러나 몽키 패치를 남용 해서는 안 되며 궁극적으로 더 잘 설계된 구조로 **리팩터링** 하는 것이 좋다.

### ✅ 요약

- 전역 변수나 DOM에 데이터를 저장하지 말고, 데이터를 분리하여 사용해라
- 내장 타입에 데이터를 저장해야 하는 경우, 안전한 타입 접근법 중 하나 (보강 OR 사용자 정의 인터페이스로 정의) 를 사용해야 합니다.
- 보강의 모듈 영역 문제를 이해해야 합니다.

### 타입 커버리지를 추적하여 타입 안전성 유지하기

**noImplicitAny** 를 설정하고 모든 암시적 any 대신 명시적 타입 구문을 추가해도 any 타입과 관련된 문제들로부터 안전하다고 할 수 없다.

any 타입이 여전히 프로그램 내에 존재할 수 있는 두 가지 경우가 있다.

- 명시적 any 타입
    - any 타입의 범위를 좁히고 구체적으로 만들어도 여전히 any 타입이다. 특히 `any[]`와 `{[key: string]:any}` 같은 타입은 인덱스를 생성하면 단순 any가 되고 코드 전반에 영향을 미친다.
    
- 서드파티 타입 선언
    - 이 경우는 `@type` 선언 파일로부터 any 타입이 전파되기 때문에 특별히 조심해야 한다.
    - **noImplicitAny**를 설정하고 절대 any를 사용하지 않았다 하더라도 여전히 any 타입은 코드 전반에 영향을 미친다.

any 타입은 타입 안전성과 생산선에 부정적 영향을 미칠 수 있으므로 프로젝트에서 any의 개수를 추적하는 것이 좋다. `npm` 의 `type-coverage` 패키지를 활용하여 any를 추적할 수 있는 방법이 있다.

```tsx
$npx type-coverage

=> 9985  10117 98.69%
```

결과를 통해 이 프로젝트의 10,117개 심벌 중 9,985개(98.69%)가 any가 아니거나 any의 별칭이 아닌 타입을 가지고 있음을 알 수 있다. 실수로 any 타입이 추가된다면, 백분율이 감소하게 된다.

저 백분율은 위에 설명한 any, unknown, naver 등등을 상황에 따라 쓰임에 대한 조언들을 얼마나 잘 따랐는지에 대한 점수라고 할 수 있다.

점수를 추적함으로써 시간이 지남에 따라 코드의 품질을 높일 수 있다.

타입 커버리지 정보를 수집해 보는 것도 유용하다.

`type-coverage` 를 실행할 때 `--detail` 플래그를 붙이면 any 타입이 있는 곳을 모두 출력해 준다.

```tsx
$npx type-coverage --detail

=> ex) page/index.tsx:1:10 이런식으로 출력해준다.
```

### noImplicitAny란?

TypeScript의 `tsconfig.json` 설정 중 하나로, **암시적 `any` 타입을 허용할지 여부**를 결정하는 옵션

### **`noImplicitAny: true`**

- 암시적으로 타입이 `any`로 설정되는 것을 **금지**
- 명확한 타입을 지정하지 않으면 TypeScript가 오류를 발생
- 타입 안전성을 높일 수 있다.

```tsx
function add(a, b) {
return a + b;
}
```

🚨 오류 발생! (noImplicitAny: true 일 때)

Parameter 'a' implicitly has an 'any' type.

### 해결 방법: 타입을 명시적으로 선언하기

```tsx
function add(a: number, b: number): number {
return a + b;
}
```

### **`noImplicitAny: false` (기본값)**

- 타입을 명시하지 않으면 자동으로 `any` 타입이 적용돼.
- 타입 체크가 느슨해지고, 런타임 오류가 발생할 가능성이 높아.

```tsx
typescript
복사편집
function add(a, b) {
  return a + b;
}

const result = add(5, "10"); // 런타임 오류 발생 가능
```

---

### **결론**

`noImplicitAny: true`로 설정하는 게 좋고, TypeScript를 제대로 활용하려면 항상 명시적인 타입을 지정하는 습관을 들이는 게 좋다.
