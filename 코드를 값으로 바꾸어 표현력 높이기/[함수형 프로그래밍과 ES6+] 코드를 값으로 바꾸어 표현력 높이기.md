![[Pasted image 20240520152924.png]]

### 📗이터러블/이터레이터 프로토콜 규약을 따른 map, filter, reduce

```js
const log = console.log;

const map = (f, iter) => {  
  let res = [];  
  for (const a of iter) {  
    res.push(f(a));  
  }  
  return res;  
};  
  
const filter = (f, iter) => {  
  let res = [];  
  for (const a of iter) {  
    if (f(a)) res.push(a);  
  }  
  return res;  
};  
  
const reduce = (f, acc, iter) => {  
  if (!iter) {  // 3번째 인자까지 작성하지 않았다면 acc를 iter로 간주한다.
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;  
  }  
  for (const a of iter) {  
    acc = f(acc, a);  
  }  
  return acc;  
};
```

위는 이터러블/이터레이터 프로토콜 규약을 따른 map, filter, reduce 함수이다. 

함수와 이터레이터를 전달 받아서 결과값을 반환을 해주는 함수들로 함수형 프로그래밍의 가장 기본이 되는 함수들이라고 할 수 있다.

아래와 같이 함수를 합성하여 사용이 가능하다.

```js

const products = [  
  {name: '반팔티', price: 15000},  
  {name: '긴팔티', price: 20000},  
  {name: '핸드폰케이스', price: 15000},  
  {name: '후드티', price: 30000},  
  {name: '바지', price: 25000}  
];  
  
const add = (a, b) => a + b;  
  
log(  
  reduce(  
    add,  
    map(p => p.price,  
      filter(p => p.price < 20000, products))));  
  
log(  
  reduce(  
    add,  
    filter(n => n >= 20000,  
      map(p => p.price, products))));
```


> __Array.prototype.map 과 같은 함수 사용하면 되는거 아냐?__
> Array.prototype.map 과 같은 함수는 배열에만 국한되어 사용이 가능하기 때문에, 이터러블/이터레이터 프로토콜 규약을 따른 위와 같은 함수들이 더 다형성이 높다.

위 예시 코드에서 문제는 코드순서가 반대로 실행된다. 이는 가독성을 낮추는데, 함수형 프로그래밍에서는 `go`와 `pipe` 그리고 `curry`로 해당 문제들을 해결한다.
### 📗코드를 값으로 다루어 표현력 높이기.

### go

`go`는 각 **`args`의 인자가 값으로 평가**되어 **다음 인자로 전달해주는** 함수이다.

```js

// reduce를 통해 args가 이터레이터로 간주되고 순회하며 함수를 실행한다.
const go = (...args) => reduce((a, f) => f(a), args);

const add = (a, b) => a + b;

go(  
  add(0, 1),  // 평가된 값이 (1 + 0) 다음으로 넘어간다 
  a => a + 10, // 1 + 10
  a => a + 100,  // 11 + 100
  log); // 111
```

아래와 같이 이전 실행순서가 반대였던 코드가 깔끔하게 정리된다.

```js
// 이전코드
log(  
  reduce(  
    add,  
    map(p => p.price,  
      filter(p => p.price < 20000, products))));  
      
// 정리된 코드
go(  
  products,  
  products => filter(p => p.price < 20000, products),  
  products => map(p => p.price, products),  
  prices => reduce(add, prices),  
  log);
```

### pipe

`pipe`는 `go` 와는 다르게 **함수를 리턴**한다. 즉 **함수들을 합성**하여 새로운 함수로 만들어준다.

```js
const pipe = (f, ...fs) => (...as) => go(f(...as), ...fs);

const f = pipe(  
  (a, b) => a + b,  
  a => a + 10,  
  a => a + 100);  
  
log(f(0, 1));
```

### curry

curry는 함수를 받아서 함수를 리턴하는 함수인데,

함수의 인자가 **두 개 이상일 때 함수를 즉시 실행**하고 아니면 다시 함수를 리턴한다.

이제 글만봐서는 이제 슬슬 헷갈리기 시작한다. 예제 코드를 봐보자

```js

// 함수의 인자가 두 개 이상일때 함수를 즉시 실행한다
const curry = f =>  
  (a, ..._) => _.length ? f(a, ..._) : (..._) => f(a, ..._);

const mult = curry((a, b) => a * b);  
log(mult(3)(2));  
  
const mult3 = mult(3);  
log(mult3(10));  
log(mult3(5));  
log(mult3(3));
```

처음에 딱 보았을때 `mult(3)(2)`의 실행 순서가 헷갈려서 콘솔을 찍어봤다.

![[Pasted image 20240520171842.png]]

`mult(3)`에서 인자가 하나 이기 때문에,  함수를 리턴한다.
해당 리턴 된 함수는 a는 3으로 할당되어 있는 상태이다. 따라서 두개의 인자가 곱해져서 6이 반환된다. `mult3` 함수를 보면 이해가 편하다.

### go + curry

기존 함수들에 전부 curry를 적용하면

```js
const map = curry((f, iter) => {  
  let res = [];  
  for (const a of iter) {  
    res.push(f(a));  
  }  
  return res;  
});  
  
const filter = curry((f, iter) => {  
  let res = [];  
  for (const a of iter) {  
    if (f(a)) res.push(a);  
  }  
  return res;  
});  
  
const reduce = curry((f, acc, iter) => {  
  if (!iter) {  
    iter = acc[Symbol.iterator]();  
    acc = iter.next().value;  
  }  
  for (const a of iter) {  
    acc = f(acc, a);  
  }  
  return acc;  
});
```

이제 아래와 같이 코드의 정리가 가능하다.

```js
const go = (...args) => reduce((a, f) => f(a), args);

// 이전코드
go(  
  products,  
  products => filter(p => p.price < 20000, products), // 1-1
  products => map(p => p.price, products),  
  prices => reduce(add, prices),  
  log);  
  
// 현재코드
go(  
  products,  
  filter(p => p.price < 20000), // 1-2.
  map(p => p.price),  
  reduce(add),  
  log);
```

현재 `go` 함수에는 첫번째를 제외한 나머지 인자에는 함수가 들어가야 한다. 

1-1 에서는 함수 호출에 인자를 두 개 사용했으므로 `curry`를 감싼 `filter`는 값을 바로 리턴하기 때문에 `go`와의 호환을 위해서 함수로 감싸주어야 한다.

1-2 에서 함수 호출에 인자를 하나만 쓴다면 `curry`에 의해 함수를 리턴 하기 때문에 go와 호환이 가능하다.

위 `go`의 실행 과정을 차근히 살펴보자.

```js

// 현재 아래와 같은 go 함수가 있다고 할때
go(  
  products,  
  filter(p => p.price < 20000),
  map(p => p.price),  
  reduce(add),  
  log);

// go 는 reduce를 실행한 결과를 반환한다.
(...args) => reduce((a, f) => f(a), args)

// 이때 reduce는 아래와 같이 실행된다.
reduce((a, f) => f(a), [
  products,  
  filter(p => p.price < 20000),
  map(p => p.price),  
  reduce(add),  
  log
])

// go 안의 reduce로 인해 앞의 products가 filter 실행당시에 인자로 전달된다.
filter(p => p.price < 20000, products);

// 위 filter는 아래와 같이 실행되어
curry((p => p.price < 20000, products) => {  
  let res = [];  
  for (const p of products) {  
		// if (p => p.price < 20000) res.push(a);  
		if (f(p)) res.push(a);  
  }  
  return productsOver20000;  
});  

// curry에서 인자가 두개이기 때문에 즉시 실행되어 바로 값으로 평가된다.
(p => p.price < 20000, products) => {  
  let res = [];  
  for (const p of products) {  
		// if (p => p.price < 20000) res.push(a);  
		if (f(p)) res.push(a);  
  }  
  return productsOver20000;  
}
// 위의 return된 productsOver20000 은 다시 go의 reduce 함수에 의해 다음 함수로 전달되고
// 그 다음 함수인 map에 인자로 전달된다.

map(p => p.price, productsOver20000)

// 이전 함수의 결과들이 다음 함수의 인수로 전달되며 go의 실행이 계속된다.
```

