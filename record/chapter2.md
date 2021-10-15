## 📌 1. 편집기를 사용하여 타입시스템 탐색하기

타입스크립트를 설치하면, 타입스크립트 컴파일러**(tsc)**, 단독으로 실행할 수 있는 타입스크립트 서버**(tsserver)** 가 있다. 
보통 타입스크립트는 컴파일러를 실행하는 것이 주된 목적이지만, 타입스크립트 서버 또한 '언어 서비스'를 제공한다는 점에서 중요하다.

**예제 코드**

```tsx
function getElement(elOrId: string|HTMLElement|null): HTMLElement {
  if (typeof elOrId === 'object') {
    return elOrId;
  } else if (elOrId === null) {
    return document.body;
  } else{
    const el = document.getElementById(elOrId);
    return el;
  }
}
```

첫 번째 if 분기문의 의도는 단지 HTMLElement라는 객체를 골라내는 것이다. 하지만 자바스크립트에서 typeof null은 'object' 이므로, elOrId는 여전히 분기문 내에서 null일 가능성이 있다. 그렇기 때문에 처음에 null 체크를 추가해서 잡는다. 두 번째 오류는 document.getElementById가 null을 반환할 가능성이 있어서 첫 번째 오류와 동일하게 null 체크를 추가 하고 예외를 던져야한다.

### ✅ 요약

- 편집기에서 타입스크립트 언어를 적극 활용해야한다.
- 편집기를 사용하면 어떻게 타입 시스템이 동작하는지, 그리고 타입스크립트가 어떻게 타입 추론을 하는지 개념을 잡을 수 있다.
- 타입스크립트가 동작을 어떻게 모델링하는 지 알기 위해 타입 선언 파일을 찾아보는 방법을 익숙하게 해야한다.

## 📌 2. 타입의 값들이 집합이라고 생각하기

```tsx
interface Person {
	name: string;
}

interface Lifespan {
	birth: Date;
	death: Date;
}

type PersonSpan = Person & Lifespan;
```

& 연산자는 두 타입의 교집합을 계산한다. person과 Lifespan이 공통으로 가지는 속성이 없기 때문에, PersonSpan 타입을 공집합(never) 로 예상하기 쉽다. 그러나 타입 연산자는 인터페이스 속성이 아닌, 값의 집합(타입의 범위)에 적용된다. 그와 더불어 추가적인 속성을 가지는 값도 여전히 그 타입에 해당된다.

즉 person과 Lifespan의 둘 다 가지는 값은 인터섹션(교집합) 타입에 속하게 된다. 

```tsx
const ps: PersonSpan = {
	name: 'jangwon',
	birth: new Date('19930829'),
	death: new Date('12344')
}; // 정상
```

당연하게도 앞의 세 가지보다 더 많은 속성을 가지는 경우에도 PersonSpan타입에 속한다. 

[❗ 타입스크립트 용어 & 집합 이론 용어 대응표
](https://www.notion.so/03e6759745e44e8abaf3a9c4f4342f21)

### ✅ **요약**

- 타입을 값의 집합으로 생각하면 이해가 편하다. 집합은 유한(boolean, 리터럴 타입)하거나 무한(number 또는 string)하다.
- 타입스크립트 타입은 엄격한 상속 관계가 아니라 겹쳐지는 집합으로 표현된다. 두 타입은 서로 서브타입이 아니면서도 겹쳐질 수 있다.
- 한 객체의 추가적인 속성이 타입 선언에 언급되지 않더라도 그 타입에 속할 수 있다.
- 타입 연산은 집합의 범위에 적용된다. 객체 타입에서는 A&B인 값이 A와 B의 속성을 모두 가짐을 의미한다.

## ❗ 3. 타입 공간과 값 공간의 심벌 구분하기

**타입과 값 사이에 다른 의미를 가지는 코드 패턴**

- 값으로 쓰이는 `this` 는 자바스크립트의 this 키워드이다. 타입으로 쓰이는 this는 일명 다형성 this라 불리는 this의 타입스크립트 타입이다. 서브 클래스의 메서드 체인을 구현할 때 유용하다.
- 값에서 &와 |은 AND와 OR 비트연산이다. 타입에서는 인터섹션과 유니온이다.
- const는 새 변수를 선언하지만 as const는 리터럴 또는 리터럴 표현식의 추론된 타입을 바꾼다.
- extends는 서브 클래스 또는 서브 타입 , 제너릭 타입의 한정자를 정의 할 수 있다.
- in은 루프 또는 매핑된 타입에 등장한다.

> 타입스크립트 코드가 잘 동작하지 않는다면, 타입 공간과 값 공간을 혼동해서 잘못 작성했을 가능성이 크다.
> 

자바스크립트에서는 객체 내의 각 속성을 로컬 변수로 만들어 주는 구조 분해 할당(destructuring)할당을 사용할 수 있다. 

```tsx
const email({person, subject, body}) {
	// ...
}
```

그런데 타입스크립트에서 구조 분해 할당을 하면, 이상한 오류가 발생한다. 

```tsx
function email({
	person: Person,
		// ~~ 바인딩 요소 'Person'에 암시적으로 any형식이 있습니다. 
	subject: string,
		// ~~ 바인딩 요소 'string'에 암시적으로 any 형식이 있습니다. 
	body: string // 'string' 식별자가 중복 되었씁니다,
});
```

값의 관점에서 Peson과 string이 해석되었기 때문에 오류가 발생한다. Person이라는 변수명과 string이라는 이름을 가지는 두 개의 변수를 생성하려한 것이다. 이러한 문제를 해결하기 위해서는 **`타입과 값을 구분지어야한다.`**

```bash
function email({person, subject, body}: {person: Person, subject: string, body: string}) {
	// ... 
}
```

> 이 코드는 장황하지만, 매개변수에 명명된 타입을 사용하거나 문맥에서 추론되도록 잘 동작합니다.
> 

### ✅ 요약

- 타입스크립트 코드를 읽을 때 타입인지 값인지 구분하는 방법을 터득해야한다.
- 모든 값은 타입을 가지지만, 타입은 값을 가지지 않는다. type과 interface같은 키워드는 타입 공간에만 존재한다.
- class나 enum과 같은 키워드는 타입과 값 두 가지로 사용될 수 있다.
- 'foo'는 문자열 리터럴이거나, 문자열 리터럴 타입일 수 있다. 차이점을 알고 구별할 수 있어야한다.
- typeof, this 그리고 많은 다른 연산자들과 키워드들은 타입 공간과 값 공간에서 다른 목적으로 사용될 수 있다.

## 📌 4. 타입 단언보다는 타입 선언을 사용하기

타입스크립트에서 변수에 값을 할당하고 타입을 부여하는 방법은 두 가지다. 

```tsx
interface Person { name: string };

const alice: Person = { name: 'Alice' }; // 타입은 person -> 타입 선언
const bob = { name: 'Bob' } as Person // 타입은 person -> 타입 단언
```

이 두 가지 방법은 결과가 같아 보이지만 그렇지 않다. 첫 번째 alice: Person은 변수에 타입 선언을 붙여 그 값이 선언된 타입임을 명시한다. 두 번째 as Person은 '타입 단언'을 수행한다. 그러면 타입스크립트가 추론한 타입이 있더라도 Person타입으로 간주한다.

타입 단언보다 타입 선언을 사용하는것이 좋다. 

```tsx
const alice: Person = {};
	// ~~ 'Person' 유형에 필요한 'name' 속성이 '{}' 유형에 없습니다. 
const bob = {} as Person; // 오류 없음
```

**타입 선언**은 할당 되는 값이 해당 인터페이스를 만족하는지 검사합니다. 앞의 예제에서는 그러지 못했지 때문에 타입스크립트가 오류를 표시했다. 

**타입 단언**은 강제로 타입을 지정했으니, 타입 체커에게 오류를 무시하라고 하는 것이다. 

타입 선언과 타입 단언의 차이는 속성을 추가하는 경우에도 마찬가지이다.

```tsx
const alice: Person = {
	name: 'Alice',
	occupation: 'TypeScirpt Developer'
// ~~ 개체 리터럴은 알려진 속성만 지정할 수 있으며 'person'형식에 'occupation'이 없다.
} 

const bob = {
	name: 'Bob',
	occupation: 'JavaScirpt Developer'
} as Person; // 오류 없음
```

타입 단언이 꼭 필요한 경우가 아니라면, 안전성 체크도 되는 타입 선언을 사용하는 것이 좋다. 

const bob = <Person>{} 같은 코드를 본적이 있을 것이다. 이 코드는 단언문의 원래 문법이며 {} as Person과 동일하다. 그러나 const bob = <person>{} 같은 코드는 <Person>이 .tsx파일(타입스크립트 + 리액트)에서 컴포넌트 태그로 인식되기 때문에 현재는 잘 쓰이지 않는다. 

화살표 함수의 타입 선언은 추론된 타입이 모호할 때가 있다. 

```tsx
const people = ['alice', 'bob', 'jan'].map(name => ({name}));
// Person[]을 원했지만 결과는 {name: string}[]...
```

타입 단언을 통해 해결할 수 있을 것 같지만앞선 예제 처럼 런타임에 문제가 발생할 수 있다.

```tsx
const people = ['alice', 'bob', 'jan'].map(name => ({} as Person)); // 오류 없음
```

단언문을 쓰지 않고, 화살표 함수 안에서 타입과 함께 변수를 선언하는 것이 가장 직관적이다.

```tsx
const people = ['alice', 'bob', 'jan'].map(name => {
 const person: Person = {name};
	return person
}); // 타입은 Person[]
```

위 코드는 꽤 복잡해 보이기 때문에 변수 대신 화살표 함수의 반환 타입을 선언해 본다.

```tsx
const people = ['alice', 'bob', 'jan'].map((name): Person => ({name})); // 타입은 Person[]
```

여기서의 소괄호는 매우 중요한 의미를 지난다.  (name): Person은 name의 타입이 없고, 반환 타입이 Person이라 명시한다. 그러나 (name: Person)은 name의 타입이 Person임을 명시하고 반환 타입은 없기 때문에 오류가 발생한다.

아래 코드는 최종적으로 원하는 타입을 직접 명시하고, 타입스크립트가 할당문의 유효성을 검사한다.

```tsx
const people: Person[] = ['alice', 'bob', 'jan'].map((name): Person => ({name}));
```

함수 체이닝이 연속되는 곳에서는 체이닝 시작에서 부터 명명된 타입을 가져야한다. 

타입 단언이 반드시 필요한 경우는 타입 체커가 추론한 타입보다 작성자가 판단하는 타입이 더 정확한 경우 의미가 있다. 예를 들어 Dom 엘리먼트가 있다.

```tsx
document.querySelector('#myButton').addEventListener('click', e => {
	e.currentTarget // 타입은 EventTarget
	const button = e.currentTarget as HTMLButtonElement;
	button // 타입은 HTMLButtonElement
});
```

타입스크립트는 DOM에 접근할 수 없기 때문에 #mybutton이 버튼 엘리먼트인지 알지 못한다. 그리고 이벤트의 currentTarget이 같은 버튼이어야하는 것 또한 알지 못한다. 이러한 부분에서 타입스크립트가 알지 못하는 정보를 작성자가 알고 있기 때문에 이러한 케이스에서는 타입 단언문을 쓰는것이 타당하다.

또한 자주 쓰이는 특별한 문법(!)을 사용해서 null이 아님을 단언하는 경우도 있다.

```tsx
const elNull = document.getElementById('foo'); // 타입은 HTMLElement | null
const el = document.getElementById('foo')!; // 타입은 HTMLElement
```

변수에 접두사로 쓰인 `!`는 boolean의 부정문이다. 그러나 접미사로 사용된 `!` 그 값이 null이 아니라는 단언문으로 해석된다. 그렇기 때문에 !를 일반적인 단언문처럼 생각해야한다. 

단언문은 컴파일 과정 중에 제거되므로, 타입체커는 알지 못하지만 그 값이 null이 아니라고 확신할 수 있는 경우에 사용해야한다. 만약 그렇지 않은 경우에는 null의 경우를 체크하는 조건문을 사용해야한다. 

### ✅ 요약

- 타입 단언(as Type)보다 타입 선언(: Type)을 사용해야 한다.
- 화살표 함수의 반환 타입을 명시하는 방법을 터득해야한다.
- 타입스크립트보다 타입 정보를 더 잘알고 있는 상황에서는 타입 단언문과 null이 아님 단언문을 사용하면 된다.

## 📌 5. 객체 래퍼 타입 피하기

  

자바스크립트에는 객체 이외에도 기본형 값들에 대한 일곱 가지 타입(string, number, boolean, null, undefined, symbol, bigint)가 있다. 기본형들은 불변(immutable)이며 메서드를 가지지 않는다는 점에서 객체와 구분된다. 

타입스크립트는 기본형과 객체 래퍼 타입을 별도로 모델링한다.

- string과 String
- number와 Number
- boolean과 Boolean
- symbol과 Symbol
- bigint와 Bigint

그렇지만 기본형 타입은 객체 리퍼에 할당할 수 있기 때문에 타입스크립트는 기본형 타입을 객체 리퍼에 할당하는 선언을 허용한다. 그러나 기본형 타입을 객체 리퍼에 할당하는 구문은 오해하기 쉽고 굳이 그렇게 할 필요 또한 없다. 

- 몽키패치: 런타임에 프로그램의 어떤 기능을 수정해서 사용하는 기법을 의미한다. 자바스크립트에서는 주로 프로토타입을 변경하는 것이 해당된다.

### ✅ 요약

- 기본형 값에 메서드를 제공하기 위해 객체 래퍼타입이 어떻게 쓰이는지 이해해야한다. 직접 사용하거나 인스턴스를 생성하는 것은 피해야 한다,
- 타입스크립트 객체 래퍼 타입은 지양하고, 대신 기본형 타입을 사용해야한다. String대신 string, Number 대신 number, Boolean대신 boolean, Symbol대신 symbol을 사용해야한다.

## 📌 6. 잉여 속성 체크의 한계 인지하기

**예제 코드 1**

```tsx
interface Room {
	numDoors: number;
	ceilingHeightFt: number;
}

const r: Room = {
	numDoors: 1,
	ceilingHeightFt: 10,
	elephant: 'present'

// ~~ 개체 리터럴은 알려진 속성만 지정할 수 있으며 'Room' 형식에 'elephant'가 없습니다. 
}
```

Room타입이 생뚱맞게 elephant 속성이 있는 것이 어색하지만, 구조적 타이핑 관점으로 생각해보면 오류가 발생하지 않아야한다. 임시 변수를 도입하면 알 수 있다.

**예제 코드 2**

```tsx
const obj = {
	numDoors: 1,
	ceilingHeightFt: 10,
	elephant: 'present'
};

const r: Room = obj; // 정상
```

obj의 타입은 `{ numDoors: number; ceilingHeightFt: number; elephant: string }` 으로 추론된다. obj 타입은 Room 타입의 부분 집합을 포함하므로, Room에 할당 가능하며 타입 체커 또한 통과한다. 

첫번째 코드는 구조적 타입시스템에서 발생할 수 있는 중요한 종류의 오류를 잡을 수 있도록 '잉여 체크' 라는 과정이 수행됐다. 그러나 잉여 속성 체크 역시 조건에 따라 동작하지 않는다는 한계가 있다. 또한 잉여 속성 체크가 할당 가능 검사와는 별도의 과정이라는 것을 알아야한다.  

타입이 명시된 변수에 객체 리터럴을 할당할 때 타입스크립트는 해당 타입의 속성이 있는지, 그리고 그 외 속성은 없는지 확인한다.

잉여 속성 체크를 원치 않는경우, 인덱스 시그니처를 사용해서 타입스크립트 추가적인 속성을 예상하도록 할 수 있다.  

```tsx
interface Options {
	title: string;
	darkMode?: boolean;
}

function createWindow(options: Options) {
	if (options.darkMode) {
		setDarkMode();
	}
}

createWindow({
	title: 'Spider Solitaire',
	darkmode: true
// ~~ 개체 리터럴은 알려진 속성만 지정할 수 있지만 
// 'Options' 형식에 darkMode이(가) 없습니다.
})
```

### ✅ 요약

- 객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때 잉여 속성 체크가 수행된다.
- 잉여 속성 체크는 오류를 찾는 효과적인 방법이지만, 타입스크립트 타입 체커가 수행하는 일반적인 구조적 할당 가능성 체크와 역할이 다르다.
- 잉여 속성 체크는 한계가 있다. 임시 변수를 도입하면 잉여 속성 체크를 건너뛸 수 있다.

## 📌 7. 함수 표현식에 타입 적용하기

자바스크립트에서는 함수 문장(statement)와 함수 표현식(expression)을 다르게 인식한다. 

```tsx
function rollDice1(sides: number): number {//...} 문장
const rollDice2 = function(sides: number): number {//} 표현식
const rollDice3 = (sides:number): number => {}; // 표현식 
```

타입스크립트에서는 **함수 표현식을 사용하는 것이 좋다**. 함수의 매개변수부터 반환 값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있다는 장점이 있기 때문이다. 

```tsx
type DiceRollFn = (sides: number) => number;
const rollDice: DiceRollFn = sides => {//};
```

편집기에서 sides에 마우스를 올려 보면, 타입스크립트에서는 이미 sides의 타입을 number로 인식하고 있다. 

**함수 타입 선언의 장점**

함수 타입의 선언은 불필요한 코드의 반복을 줄인다. 사칙연산을 하는 함수 네 개는 다음과 같이 작성할 수 있다.

```tsx
function add(a: number, b: number) { return a + b };
function sub(a: number, b: number) { return a - b };
function mul(a: number, b: number) { return a - b };
function div(a: number, b: number) { return a - b };
```

반복되는 함수 시그니처를 하나의 함수 타입으로 통합할 수도 있다.

```tsx
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a */ b; 
```

함수 타입 선언을 이용했던 예제보다 타입 구문이 적다. 함수 구현부도 분리되어 있어 로직이 보다 분명하다. 모든 함수 표현식의 반환 타입까지 number로 선언했다. 

**예시**

```tsx
const responsP = fetch('/quote?by=Mark+Twain'); // 타입이 Promise<Response>
```

그리고 `response.json()` 또는 `response.text()` 를 사용해 응답의 데이터를 추출한다. 

```tsx
async function getQuote() {
	const response = await fetch('/quote?by=Mark+Twain');
	const quote = await response.json();
	return quote;
}
```

여기에는 버그가 존재한다. /quote가 만약 존재하지 않는 API라면 '404 NotFound' 가 포함된 내용을 응답한다. 응답은 Json형식이 아니라는 새로운 오류 메세지를 담아 거절된(rejected) 프로미스를 반환한다. 호출한 곳에서는 새로운 오류 메세지가 전달되어 실제 오류인 404가 감춰진다. 

또한 fetch가 실패하면 거절된 프로미스를 응답하지는 않는다는 것을 간과하기 쉽다. 그렇기 때문에 상태 체크를 해줄 checkedFetch 함수를 작성해본다.

```tsx
declare function fetch(input: RequestInfo, init?: RequestInit): Promise<Response>;
```

```tsx
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
	const response = await fetch(input, init);
	if (!response.ok) {
		throw new Error('Request failed: ' + response.status);
	}
	return response;
}
```

위 코드도 잘 동작하지만, 더 간결하게 작성할 수 있다.

```tsx
const checkedFetch: typeof fetch = async (input, init) => {
	const response = await fetch(input, init);
	if (!response.ok) {
		throw new Error('Request failed: ' + response.status);
	}
	return response;
}
```

함수 문장을 함수 표현식으로 바꿨고 함수 전체에 `타입(typeof fetch)`를 적용했다. 이는 타입스크립트가 input과 init의 타입을 추론할 수 있게 해준다. 

타입 구문은 또한 checkedFetch의 반환 타입을 보장하며, fetch와 동일하다. 예를 들어 throw대신 return을 사용했다면, 타입스크립트는 그 실수를 잡아낸다.

```tsx
const checkedFetch: typeof fetch = async (input, init) => {
	// Promise <Response | Error>형식은 Promise<Response>형식에 할당할 수 없습니다.
	const response = await fetch(input, init);
	if (!response.ok) {
		return new Error('request' + response.status);
	}
	return response;
}
```

함수 문장으로 작성한 예제에서도 throw가 아니라 return을 사용할 경우 오류가 발생한다. 그러나 오류는 첫 번째와 달리 checkedFetch 구현체가 아닌, 함수를 호출한 위치에서 발생한다. 

함수의 매개변수에 타입 선언하는 것 보다 함수 표현식 전체 타입을 정의하는 것이 코드도 간결하고 안전하다. 

### ✅ 요약

- 매개변수나 반환 값에 타입을 명시하기 보다는 함수 표현식 전체에 타입 구문을 적용하는 것이 좋다.
- 만약 같은 타입 시그니처를 반복적으로 작성한 코드가 있다면 함수 타입을 분리해 내거나 이미 존재하는 타입을 찾아봐야한다. 라이브러리를 직접 만든다면 공통 콜백에 타입을 제공해야한다.
- 다른 함수의 시그니처를 참조하려면 `typeof fn` 을 사용하면 된다.

## 📌 8. 타입과 인터페이스의 차이점 알기

타입스크립트에서 명명된 타입을 정의하는 방법은 두 가지가 있다.

다음 처럼 타입을 사용할 수 있다. 

```tsx
type TState = {
	name: string;
	capital: string;
}
```

또는 인터페이스를 사용해도 된다.

```tsx
interface IState {
	name: string;
	capital: string;
}
```

대부분의 경우에는 타입을 사용해도 되고 인터페이스를 사용해도 된다. 
그러나 타입과 인터페이스 사이에 존재하는 차이를 분명히 알고, 같은 상황에서는 동인한 방법으로 명명된 타입을 정의해 일관성을 유지해야한다. 

### 타입과 인터페이스 선언의 비슷한 점

1. **명명된 타입은 인터페이스로 정의하든 타입으로 정의하든 상태에는 차이가 없다.**

```tsx
const jangwon: TState = {
	name: 'jangwon',
	capital: 'chchch',
	population: 50000

// ~~ 형식은 TState 형식에 할당할 수 없다. 
// 개체 리터럴은 알려진 속성에만 지정할 수 있으며, TState 형식에 population이 없다. 
}
```

1. **인덱스 시그니처는 인터페이스와 타입에서 모두 사용할 수 있다.**

```tsx
type TDict = { [key: string]: string };
interface IDict {
	[key: string ]: string;
}
```

1. **함수 타입 또한 인터페이스나 타입으로 정의할 수 있다.** 

```tsx
type TFn = (x: number) => string;
interface IFn {
	(x: number): string;
}
```

```tsx
const toStrT: TFn = x => '' + x; // 정상
const toStrI: IFn = x => '' + x; // 정상
```

```tsx
type TFnWithProperties = {
	(x: number): number;
	prop: string;
}
```

```tsx
interface IFnWithProperties {
	(x: number): number;
	prop: string;
}
```

1. **타입 별칭과 인터페이스는 모두 제네릭이 가능하다.**

```tsx
type Tpair<T> = {
	first: T;
	second: T;
}

interface IPair<T> {
	first: T;
	second: T;
}
```

1. **한편 클래스를 구현(implements) 하는 경우에는 타입과 인터페이스 둘 다 사용할 수 있다.**
    
    ```tsx
    class StateT implements TState {
    	name: string = '';
    	capital: string ='';
    }
    
    class StateI implements IState {
    	name: string = '';
    	capitatal: string = '';
    }
    ```
    
- **명명된 타입은 인터페이스로 정의하든 타입으로 정의하든 상태에는 차이가 없다.**
- **인덱스 시그니처는 인터페이스와 타입에서 모두 사용할 수 있다.**
- **함수 타입 또한 인터페이스나 타입으로 정의할 수 있다.**
- **타입 별칭과 인터페이스는 모두 제네릭이 가능하다.**
- **한편 클래스를 구현(implements) 하는 경우에는 타입과 인터페이스 둘 다 사용할 수 있다.**

**인터페이스는 타입을 확장할 후 있으며 타입은 인터페이스를 확장할 수 있다.**

```tsx
interface IStateWithPop extends TState {
	population: number;
}

type TStateWithPop = IState & { population: number };
```

`IStateWithPop` 과  `TStateWithPop` 은 동일하다. 여기서 **주의할 점은 인터페이스는 유니온 타입 같은 복잡한 타입을 확장하지는 못한다는 것이다**. 복잡한 타입을 확장하기 위해서는 타입과 &를 사용해야한다. 

### 인터페이스와 타입의 차이점

- 인터페이스는 유니온 타입 같은 복잡한 타입을 확장하지 못한다.
    - 즉 &를 통한 복잡한 확장을 할 수 없다.
- 복잡한 타입을 확장하고 싶다면 타입과 &를 사용해야한다.
- 유니온 타입은 있지만 유니온 인터페이스는 없다.
- 인터페이스에는 타입에 없는 몇 가지 기능이 있다.
    - 보강(agument)이 가능하다.

**유니온 타입은 있지만 유니온 인터페이스는 없다.**

```tsx
type NamedVariable = (Input | Output) & { name: string };
```

이 타입은 인터페이스로 표현 할 수 없다. type은 일반적으로 interface 보다 쓰임새가 많다. type 키워드는 유니온이 될 수 도 있고, 매핑된 타입 또는 조건부 타입과 같은 고급 기능에 활용된다. 

튜플과 배열 타입도 type 키워드를 이용하여 더 간결하게 표현할 수 있다. 

```tsx
type Pair = [number, number];
type StringList = string[];
type NaemNums = [string, ...number[]];
```

인터페이스로도 튜퓰을 비슷하게 구현할 수 있다.

```tsx
interface Tuple {
	0: number;
	1: number;
	length: 2;
}
const t: Tuple = [10, 20]; //정상
```

인터페이스로 튜플과 비슷하게 구현하면 튜플에서 사용할 수 있는 `concat` 같은 메서드들을 사용할 수 없다. 튜플은 type 키워드로 구현하는 것이 좋다. 

인터페이스는 타입과 다르게 몇 가지 기능이 있다.

1. **보강(agument)이 가능하다.**

```tsx
interface IState {
	name: string;
	capital: string;
}

interface IState {
	population: number;
}

const jan: IState = {
	name: 'jan',
	capital: 'jj',
	population: 50000
}; // 정상
```

위의 코드 처럼 속성을 확장하는 것을 **'선언 병합'** 이라한다. 

타입 선언 파일을 작성할 때는 선언 병합을 지원하기 위해 반드시 인터페이스를 사용해야 하며 표준을 따라야한다. 타입 선언에는 사용자가 채워야 하는 빈틈이 있을 수 있는데, 바로 이 선언 병합이 그렇다. 

병합은 선언과 마찬가지로 일반적인 코드에서도 지원되므로 언제 병합이 가능한지 알고 있어야 한다. 타입은 기존 타입에 추가적인 보강이 없는 경우에만 사용한다. 

아직 스타일이 확립되지 않은 프로젝트인 경우에는, 향후에 보강의 가능성이 있을지 생각해 봐야한다. 어떤 API에 대한 타입 선언을 작성해야 한다면 인터페이스를 사용하는 것이 좋다. API가 변경될 때 사용자가 인터페이스를 통해 새로운 필드를 병합할 수 있어 유용하다. 

### ✅ 요약

- 타입과 인터페이스의 차이점과 비슷한 점을 이해해야 한다.
- 프로젝트에서 어떤 문법을 사용할지 결정하는 경우 한 가지의 일관된 스타일을 확립하고, 보강 기법이 필요한지 고려해야한다.

### 타입 연산과 제너릭 사용으로 반복 줄이기

```tsx
interface Person {
	firstName: string;
	lastName: string;
}

interface PersonWithBirthDate {
	firstName: string;
	lastName: string;
	birth: Date;
}
```

타입 중복은 코드 중복만큼 많은 문제를 발생시킨다. 

타입에서 중복이 더 흔한 이유 중 하나는 공유된 패턴을 제거하는 매커니즘이 기존 코드에서 하던 것과 비교해 덜 익숙하기 때문이다. 

반복을 줄이는 가장 간단한 방법은 타입에 이름을 붙이는 것이다. 

```tsx
function distance(a: {x: number, y: number}, b: {x: number, y: number}) {
	return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y ,2));
}
```

코드를 수정해 타입에 이름을 붙여 본다.

```tsx
interface Point2D {
	x: number;
	y: number;
}

function distance(a: Point2D, b: Point2D) {/* ... */}
```

```tsx
interface Person {
	firstName: string;
	lastName: string;
}

interface PersonWithBirthDate extends Person {
	birth: Date;
}
```

일반적이지는 않지만 인터섹션 연산자(&)를 사용할 수 있습니다. 

```tsx
type PersonWithBirthDate = Person & { birth: Date };
```

**state** 타입과 단지 부분만 표현하는 **TopNavState**가 있는 경우를 보자

```tsx
interface State {
	userId: string;
	pageTitle: string;
	recentFiles: string[];
	pageContents: string;
}

interface TopNavState {
	userId: string;
	pageTitle: string;
	recentFiles: string[];
}
```

**TopNavState 를** 확장하여 state를 구성하기보다, State의 부분 집합으로 TopNavState를 정의하는 것이 바람직하다. 이러한 방법이 전체 앱의 상태를 하나의 인터페이스로 유지할 수 있게 해준다. 

State를 인덱싱하여 속성의 타입에서 중복을 제거할 수 있다.

```tsx
type TopNavState = {
	userId: State['userId'];
	pageTitle: State['pageTitle'];
	recentFiles: State['recentFiles'];
}
```

State 내의 pageTitle의 타입이 바뀌면 TopNavState도 반영된다. 

```tsx
type TopNavState = {
	[k in 'userId' | 'pageTitle' | 'recentFiles']: State[k]
};
```

매핑된 타입은 배열의 필드를 루프 노는 것과 같은 방식이다. 이 패턴은 표준 라이브러리에서도 일반적으로 찾을 수 있으며, Pick이라 한다.

```tsx
type Pick<T,L> = { [k in K ]: T[k] };
```

정의가 완전하지는 않지만 다음과 같이 사용할 수 있다.

```tsx
type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles' >;
```

여기서 `Pick`은 제너릭 타입이다. `Pick` 을 사용하는 것은 함수를 호출하는 것과 마찬가지이다. 마치 함수에서 두 개의 매개변수 값을 받아 결괏값을 반환하는 것처럼, Pick은 T와 K 두 가지 타입을 받아서 결과 타입을 반환 한다. 

생성하고 난 다음에 업데이트가 되는 클래스를 정의하면, update 메서드 매개변수의 타입은 생성자와 동일한 매개변수이면서, 타입 대부분이 선택적 필드가 된다.

```tsx
interface Options {
	width: number;
	height: number;
	color: string;
	label: string;
}

interface OptionsUpdate {
	width?: number;
	height?: number;
	color?: string;
	label?: string;
}

class UIWidget {
	constructor(init: Options) {/* ... */}
	update(options: OptionsUpdate) {/* ... */}
}
```

매핑된 타입과 keyof를 사용하면 options으로부터 OptionsUpdate를 만들 수 있다. 

```tsx
type OptionsUpdate = {[k in keyof Options]?: Options[k]};
```

keyof 타입을 받아서 속성 타입의 유니온을 반환합니다. 

```tsx
type OptionsKeys = keyof Options;
```

```tsx
class UIWidget {
	constructor(init: Options) {/* ... */}
	update(options: Partial<Options>) {/* ... */}
}
```

값의 형태에 해당하는 타입을 정의하고 싶을 때도 있다.

```tsx
const INIT_OPTIONS = {
	width: 640,
	height: 480,
	color: '#00FF00',
	label: 'VGA'
}

interface Options {
	width: number;
	height: number;
	color: string;
	label: string;
}
```

이런 경우 typeof를 사용하면 된다.

```tsx
type Options = typeof INIT_OPTIONS;
```

값으로 부터 타입을 만들어 내는 경우에는 선언의 순서를 주의 해야한다. 타입 정의를 먼저하고 값이 그 타입에 할당 가능하다고 선언하는 것이 좋다. 

제네릭 타입에서 매개변수를 제한할 수 있는 방법은 extends를 사용하는 것이다.

```tsx
interface Name {
	first: string;
	last: string;
}

type DancingDuo<T extends Name> = [T, T];

const couple1: DancingDuo<Name> = [
	{first: 'Fred', last: 'Astarie'},
	{first: 'Ginger', last: 'Rogers'}
];
```

### ✅ 요약

- DRY(don't repeat yourself) 원칙을 타입에도 최대한 적용해야한다.
- 타입에 이름을 붙여서 반복을 피해야 한다. extends를 사용해서 인터페이스 필드의 반복을 피해야한다.
- 타입들 간의 매핑을 위해 타입스크립트가 제공한 도구들을 공부하면 좋다. 여기에는 keyof, typeof, 인덱싱, 매핑된 타입들이 포함된다.
- 제너릭 타입은 타입을 위한 함수와 같다. 타입을 반복하는 대신 제너릭 타입을 사용하여 타입간의 매핑을 하는 것이 좋다. 제너릭 타입을 제한하기 위해서는 extends를 사용하면 된다.
- 표준 라이브러리에 정의 된 Pick, Partial, ReturnType같은 제너릭 타입에 익숙해져야한다.