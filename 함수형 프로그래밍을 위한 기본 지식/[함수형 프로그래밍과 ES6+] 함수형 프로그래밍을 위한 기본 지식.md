![](https://blog.kakaocdn.net/dn/tyz0k/btsHucq0bgw/FhyJq3MW0EaCQnxUT25l2k/img.jpg)

### 📗함수형 프로그래밍을 위한 자바스크립트 기본기
### 함수형 프로그래밍은 어떤 이론을 토대로 가능할까

**평가**란 코드가 계산(Evaluation) 되어 **값**을 만드는 것 이다.
  
**일급**이란 아래 4개의 조건을 충족할 수 있어야 한다.
- 값으로 다룰 수 있다.  
- 변수에 담을 수 있다.  
- 함수의 인자로 사용될 수 있다.  
- 함수의 결과로 사용될 수 있다.  

함수는 **일급**이며 **일급 함수**라 부를 수 있다. 일급 함수는 조합성과 추상화의 도구가 된다

 자바스크립트는 함수를 **값**으로 **평가**하며, 함수 자체가 **일급**이기 때문에 함수의 인자로 받아서 실행하거나 함수를 반환할 수 있다. 이 특성은 함수를 값으로 다루는 함수인 **고차 함수**의 작동을 가능하게 하며 함수형 프로그래밍의 토대가 된다.
 
```js
const apply1 = f => f(1);
const add2 = a => a + 2;
log(apply1(add2));
log(apply1(a => a - 1));

const times = (f, n) => {
  let i = -1;
  while (++i < n) f(i);
};

times(log, 3);
times(a => log(a + 10), 3);

const addMaker = a => b => a + b;
const add10 = addMaker(10);
log(add10(5));
log(add10(10));
// apply1과 times 함수는 함수를 인자로 받아서 실행한다.
// addMaker는 함수를 리턴한다.
```

### 📗ES6에서의 순회와 이터러블 이터레이터 프로토콜

ES6에서의 순회 방식은 기존과 조금 달라졌다. `for i++` -> `for of` 

```js
const list = [1, 2, 3];
// 기
for (var i = 0; i < list.length; i++) {
  // log(list[i]);
}
const str = 'abc';
for (var i = 0; i < str.length; i++) {
  // log(str[i]);
}
// ES6
for (const a of list) {
  // log(a);
}
for (const a of str) {
  // log(a);
}
```

`Array` 뿐만 아니라 `Set` 과 `Map` 객체도 순회가 가능하다. 이게 어떻게 가능할까?

```js

### Array를 통해 알아보기

log('Arr -----------');
const arr = [1, 2, 3];
let iter1 = arr[Symbol.iterator]();
for (const a of iter1) log(a);

### Set을 통해 알아보기

log('Set -----------');
const set = new Set([1, 2, 3]);
for (const a of set) log(a);

### Map을 통해 알아보기

log('Map -----------');
const map = new Map([['a', 1], ['b', 2], ['c', 3]]);
for (const a of map.keys()) log(a);
for (const a of map.values()) log(a);
for (const a of map.entries()) log(a);
console.clear();

```

여기서 `console.log`로 `set[0]` 을 실행하고 `map[0]` 을 실행해도 `undefined`만이 나올 뿐이다. 
`Map` 과 `Set` 이 `array` 처럼 숫자로 된 `key`와 대응하는 `value`로 순회하는 방식으로 동작하지 않고, **이터러블/이터레이터 프로토콜**을 따르기 때문에 가능하다.

> __이터러블/이터레이터 프로토콜__
> - 이터러블: 이터레이터를 리턴하는 `[Symbol.iterator]() `를 가진 값
> - 이터레이터: { value, done } 객체를 리턴하는 next() 를 가진 값
> - 이터러블/이터레이터 프로토콜: 이터러블을 for...of, 전개 연산자 등과 함께 동작하도록 한 규약

실제로 아래와 같이 객체 내 `[Symbol.iterator]()` 메소드를 `null` 로 만든다면 순회가 작동하지 않음을 알 수 있다.

![[Pasted image 20240519213426.png]]

사용자 정의 이터레이터

```js
const iterable = {
  [Symbol.iterator]() {
    let i = 3;
    return {
      next() {
        return i == 0 ? {done: true} : {value: i--, done: false};
      },
      [Symbol.iterator]() {
        return this; 
      }
    }
  }
};
```

> __well-formed 이터레이터__
> 이터레이터가 자기 자신을 반환하는 심볼 이터레이터 메소드를 가지고 있는 이터레이터
### 📗제너레이터와 이터레이터

**제너레이터**는 **이터레이터이자 이터러블을 리턴하는 함수** 이며, 아래와 같이 선언한다.

```js
function* gen() { // *을 붙인다.
	yield 1;
    yield 2;
    yield 3;
}
```

제너레이터는 문장(코드)을 통해 순회할 수 있는 값으로 만들 수 있기 때문에, **자바스크립트에서는 어떠한 상태나 어떠한 값이든 순회 할 수 있게 된다**. 이 특성 덕분에 다형성이 높게 함수형 프로그래밍을 구현 할 수 있다.

```js
function* infinity(i = 0) {
    while (true) yield i++;
}

function* limit(l, iter) {
  for (const a of iter) {
    yield a;
    if (a == l) return;
  }
}

function* odds(l) {
  for (const a of limit(l, infinity(1))) {
    if (a % 2) yield a;
  }
}

let iter2 = odds(10);
log(iter2.next());
log(iter2.next());
log(iter2.next());
log(iter2.next());
log(iter2.next());
log(iter2.next());
log(iter2.next());

for (const a of odds(40)) log(a);

log(...odds(10));
log([...odds(10), ...odds(20)]) // 제너리이터로 생성된 이터레이터를 통해 다양한 함수 조합을 할 수 있다.

const [head, ...tail] = odds(5);
log(head);
log(tail)

const [a, b, ...rest] = odds(10);
log(a);
log(b);
log(rest);
```

정리하면,
자바스크립트는 함수를 **값**으로 **평가**하며, 함수 자체가 **일급**이기 때문에 함수의 인자로 받아서 실행하거나 함수를 반환할 수 있다. 

ES6에서의 순회 방식은 **이터러블/이터레이터 프로토콜**을 따르는 방식으로 순회가 가능하며, 이터레이터를 리턴하는 제너레이터 함수를 통해 손쉽게 이터레이터를 만들 수 있다.

**일급 함수**의 특성은 이렇게 손쉽게 생성된 이터레이터를 손쉽게 인자로 받거나 반환할 수 있어 함수형 프로그래밍의 다형성에 기여한다.