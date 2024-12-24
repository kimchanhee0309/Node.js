# 2장. 자바스크립트/Node.js의 문법

* Node.js의 실행 환경을 구축하고 간단한 자바스크립트 문법을 다룸
  * 자바스크립트를 처음 다룰 때 필요한 기초 문법을 이해하기 위한 장
  * 자바스크립트에 익숙한 사람은 건너뛰어도 되는 장
 

<details>
<summary>2.1 개발 환경 도입</summary>
<div markdown="1">    

### 2.1.1 Node.js의 버전

</div>
</details>

___

<details>
<summary>2.2 자바스크립트 기초</summary>
<div markdown="1">    

### 2.2.1 변수

선언 시 사용 | 내용 
----------------- | ------------------ 
var | 변수를 선언, 초기화할 수 있다.
const | 범위 안에서 유효하고 **재할당 불가능한 변수를 선언** 할 수 있다.
let | 범위 안에서 유효한 변수를 선언, 초기화할 수 있다.

* **var**
  * let/const와 달리 **선언한 범위를 넘어 참조** 할 수 있음
  * 예시 코드
```javascript
if (true)
{
  var foo = 5;
}
console.log(foo); // 5
```

* **let/const**
  * **선언한 범위 안에서 유효** 함
  * 예시 코드
```javascript
if (true)
{
  const bar = 5;
}
console.log(bar); // ReferenceError : bar is not defined
```

* 특징 및 유의사항
  * 변수의 영향 범위가 넓어질수록 코드를 추적하기 어려워짐
    * 이는 곧 유지보수성이 낮아짐 + 버그의 원인이 됨 -> So, let/const 를 주로 사용함
    * So, 변수가 유효한 범위는 가능한 한 좁히는 것이 바람직함
    * 이용 우선순위 : const > let > var

* let/const의 차이
  * **'재할당 가능 여부'**
    * let은 재할당 가능, const는 재할당 불가능
    * 예시 코드
```javascript
//const
const foo = 5;
console.log(foo);

foo = 'test';
console.log(foo);
foo = 'test'; // TypeError: Assignment to constant variable.

//let
let foo = 5;
console.log(foo); //5
foo = 'test';
console.log(foo); //test
```

* 함수 이름(식별자)은 문자, 언더스코어(_), 달러 기호($)로만 시작할 수 있음. 이후에 숫자 사용 가능
* 예시 사례
```javascript
const abc = 'abc'; // OK
const _abc = '_abc'; // OK
const abc123 = 'abc123' // OK
const 123 = '123'; // NG, 숫자는 맨 앞에 올 수 없음
```

### 2.2.2 연산자
* 할당, 비교, 산술, 문자열 등의 연산자 사용 가능
* 예시 코드
```javascript
const a = 2; // 2
const b = a * 2 + 1; // 5

const less = a < b; // true
const equal = a === b // false
```

* 동등 비교
  * 동등 연산자(==) 보다 일치 연산자(===)가 더 엄격한 비교를 수행함
  * 예시 코드
```javascript
const a = 1;
const b = 1;
const equal = a == b; // true
const euqal2 = a === b; // true

===================================

const a = 1;
const b = '1';
const equal = a == b; // true
const equal2 = a === b; // false
```
* 모호한 비교(==)에서는 같다고 판단하지만, 엄격한 비교(===)에서는 타입을 비교하기 때문에 같지 않다고 판단함
* 기본적으로는 엄격한 비교를 사용하고, 반드시 필요할 때만 모호한 비교를 이용하는 것이 안전함
* 부등 연산자에도 모호한 부등 연산자(!=)와 엄격한 부등 연산자(!==)가 있음

### 2.2.3 데이터 타입
* 자바스크립트는 동적 타입 언어로, **명시적으로 타입을 선언하지 않음**
* But, 각각의 값을 연산하거나 비교하는 경우에는 종종 변수 내부에 들어 있는 타입을 의식해야 할 때가 있음

데이터 타입 | 설명 
----------------- | ------------------ 
String | 문자열을 나타내는 타입
Nuimber | 정수나 부동 소수점과 같은 숫자를 다루는 타입
BigInt | 큰 자리를 다루는 정숫값
Boolean | true 또는 false를 가지는 불리언값
Symbol | 유일한 값이 되는 심벌 타입
undefined | 정의되지 않았음을 나타내는 타입
null | 데이터가 없는 것을 나타내는 타입
Object | 객체, 배열, 정규 표현, 함수 등 원시 이외의 타입

* 데이터 타입을 확인할 때는 typeof 연산자를 사용할 수 있음
  * typeof가 반환하는 데이터 타입의 일부는 위 표와 다름
* 예시 사례
```javascript
typeof 데이터 // 문자열로 데이터 타입을 반환함

typeof 'string' // 문자열은 'string'
typeof [] // 배열은 'object'
typeof console.log // 함수는 'function
typeof null // null은 'object'
```

#### String
```javascript
$ node
> "안녕하세요"
'안녕하세요'
>'안녕하세요'
'안녕하세요'
>`안녕하세요`
'안녕하세요'

=====================

$ node
> console.log('첫 번째 행\n두 번째 행')
첫 번째 행
두 번째 행
undefined

====================

// 여러 행
`첫 번째 행
두 번째 행
세 번째 행`

// 변수 전개, ${변수}를 템플릿 리터럴 안에 기재함
const one = '첫 번째';
const two = '두 번째';
const line = `'${one}' '${two}'`; // '첫 번째' '두 번째'
```

#### Number
```javascript
const int = -5;
const double = 3.4;
```

#### Boolean
```javascript
const okFlag = true;
const ngFlag = false;

// ok라고 출력됨
if (okFlag)
{
  console.log('ok');
}
else
{
  console.log('ng');
}
```

#### undefined와 null
* undefined는 변수가 정의되지 않았음을 나타냄
```javascript
$ node
> x
undefined
```

* null은 '존재하지 않음'을 나타내는 데이터 타입
```javvascript
const data = null
```

* undefined와 null은 에러를 일으키기 쉬운 값들임
* ex. 객체를 null로 덮어쓴 뒤 해당 객체의 속성에 접근하는 경우 런타임 에러 발생
* 예시 코드
```javascript
let obj = { foo: 'hello' };
console.log(obj.foo); // hello

obj = null;
console.log(obj.foo); // Uncaught TypeError: Cannot read properties of null (reading 'foo')
```
* 타임스크립트를 도입하면 null이나 undefined를 컴파일 시점에 체크할 수 있어 런타임 에러를 미리 감지할 수 있음

### 2.2.4 Object
* 원시 타입(String, Number, Boolean 등) 이외의 타입을 말함
* 배열, Date, 정규 표현, 함수 등 다양한 타입이 Object를 상속해 구현됨
* 다양한 값을 하나의 그룹으로 취급하는 용도로 이용됨
* Object 초기화는 { }로 감싼 뒤 **속성_이름: 값** 으로 표현되고, 콤마(,) 구분자를 이용해 여러 요소를 넣을 수 있음
```javascript
{
  속성_이름: 값,
  속성_이름2: 값
}
```

* 속성 이름에는 문자열 타입, 숫자 타입, Symbol 타입 등을 지정할 수 있음
```javascript
const obj =
{
  key: 'value',
  key2: 'value2'
};
```

* 속성값에는 원시 타입, Object의 중첩, Date 객체나 정규 표현, 함수 등 다양한 값을 넣을 수 있음
```javascript
const obj =
{
  foo: {
    bar: 'baz'
  },
  now: new Date(),
  func: function() {
    console.log('function');
  }
}
```

* ES6부터는 새로운 표기법이 등장함(속성 이름을 생략하고 작성하는 표기법)
```javascript
const key = 'value';
const key2 = 'calue2';

const obj =
{
  key, // key의 값(value)이 들어감
  key2 // key2의 값(value2)이 들어감
};
```

* 속성 이름을 객체 바깥에서 정의한 값을 이용해 초기화할 수도 있음
```javascript
const key = 'keyName';
const obj = { [key]: 'value'}; //keyName과 같은 속성에 값(value)이 들어감
console.log(obj); // { keyName: 'value' }
```

* Object로 선언한 속성에는 마침표 또는 []로 접근할 수 있음
```javascript
const obj =
{
  foo: 'hello',
  bar: {
    baz: 'world'
  }
};

console.log(obj.foo); // hello
console.log(obj['foo']); // hello
console.log(obj.bar.baz); // world
console.log(obj['bar']['baz']); // world
```

* 마침표는 속성 이름이 변수의 식별자와 같은 패턴에서만 사용 가능
* So, 속성 이름은 되도록 변수와 같은 방식으로 이름을 짓는 것이 좋음
```javascript
const obj =
{
  123: '숫자'
  '': '빈 문자열'
};

console.log(obj.123); // SyntaxError
console.log(obj[123]); // 숫자
console.log(obj.''); // SyntaxError
console.log(obj['']); // 빈 문자열
```

* Object의 속성은 선언 후에 바꿔쓸 수도 있음
```javascript
const obj =
{
  foo: 'hello'
};

console.log(obj.foo); // hello

obj.foo = 'good bye';

console.log(obj.foo); // good bye
```

* const는 어디까지나 재할당 금지를 의미하는 것! Object의 속성을 고정하지는 않음
* 다음과 같이 obj에 대해 재할당 시도할 시 에러가 발생함
```javascript
const obj =
{
  foo: 'hello'
};

obj =
{
  foo: 'good bye'
};
// Uncaught TypeError: Assignment to constant variable.
```

* 이러한 경우, Object.freeze()로 객체를 고정(동결)해 속성을 덮어쓰는 것을 막을 수 있음
```javascript
const obj = {
  foo: 'hello'
};

Object.freeze(obj);

console.log(obj.foo); // hello

obj.foo = 'good bye'; // 할당 자체에서는 에러가 발생하지 않음

console.log(obj.foo); // hello
```

### 2.2.5 배열
* []로 선언된 배열은 Array 객체로 취급되며, 몇 가지 속성과 함수를 이용할 수 있음
* 배열이 가진 함수의 예
  * 배열의 요소를 반복하면서 새로운 배열로 변환하는 **Array.prototype.map**
  * 조건에 일치하는 요소만 추출하는 **Array.prototype.filter**
```javascript
const students = [
  { name: 'Alice', age: 10 },
  { name: 'Bob', age: 20 },
  { name: 'Catherine', age: 30 }
];

const nameArray = students.map(function(person) {
  return person.name;
});
console.log(nameArray); // ['Alice', 'Bob', 'Catherine']

const under20 = students.filter(function(person) {
  return person.age <= 20;
});

console.log(under20); // [{ name: 'Alice', age: 10 }, { name: 'Bob', age: 20 }]
```

### 2.2.6 함수
* 함수의 인수에 객체를 전달할 때 참조로 전달함
```javascript
function setName(obj) {
  obj.name = 'Bob';
}

const person = { name: 'Alice' };
console.log(person.name); // Alice

setName(person);
console.log(person.name); // Bob
```

* 함수 표현식으로 별도의 변수에 할당할 수도 있음. 이때 함수 이름을 생략할 수 있음(익명 함수라고도 함)
* 익명 함수는 콜백이나 즉시 실행 함수에도 사용할 수 있음
```javascript
const add = function(a, b) {
  return a + b;
}

// 콜백에 익명 함수
setTimeout(function() {
  console.log('1s')
}, 1000);

// 즉시 실행 함수, 곧바로 실행됨
(function() {
  console.log('executed')
})();
```

#### ES6 이후의 함수
* 기본 인수와 화살표 함수라는 새로운 표기법 등장
* **기본 인수** : 함수의 인수를 생략했을 때 기본값을 전달
```javascript
function add(1, b = 2) {
  return a + b;
}

const total = add(1);
console.log(total); // 3

const total2 = add(1, 3);
console.log(total2); // 4
```

* **화살표 함수** : 인수가 하나뿐인 (), 혹은 함수 내부 처리가 1행인 경우 {}와 return을 생략할 수 있음
```javascript
// 함수
function add(a, b) {
  return a + b;
}

// 화살표 함수
const add = (a, b) => {
  return a + b;
};

=========================

const double = a => a * 2;
console.log(double(3)); // 6
```

</div>
</details>

___

<details>
<summary>2.3 자바스크립트와 상속</summary>
<div markdown="1">    

### 2.3.1 자바스크립트와 class

</div>
</details>

___

<details>
<summary>2.4 자바스크립트와 this</summary>
<div markdown="1">    

</div>
</details>

___

<details>
<summary>2.5 ES6 이후의 중요한 문법</summary>
<div markdown="1">    

### 2.5.1 전개 구문

### 2.5.2 분할 대입

### 2.5.3 루프

</div>
</details>
