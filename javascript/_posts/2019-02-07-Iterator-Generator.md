---
layout: post
title: [ES6] 반복자(Iterator) 와 생성자(Generator)
---
Iterator와 Generator는 es6 부터 있던기능인데 필요성을 느끼지못하여 잘 살펴보지 않았었는데, 최근 제가 사용하고자 하는 라이브러리 들이 공통적으로 Generator를 사용하기에, 필요성을 느껴 스터디겸 정리를 해보고자합니다. 

## Iteratable Object
Iteratable Object는 말그대로 순회가 가능한 객체입니다. iterable하기위해 @@iterator 메소드를 정의해야하는데, 객체(또는 프로토타입 체인 내 객체중 하나)에 Symbole.iterator 속성을 가지고 있고. 그 값으로 Iterator 를 반환하는 함수를 할당 해야합니다. String, Array, Map, Set,TypedArray 이 객체들은 @@iterator method가 프로토타입내에 내장되어 있기때문에 바로 Iterator를 사용할 수 있습니다.

``` javascript
  for( let el of ['foo','bar','baz']){ // iterator값은 for ... of문으로 가져올수 있습니다.
    console.log(el) // 'foo', 'bar', 'baz' 차례대로력 출력
  }

  for( let char  of 'foo'){ 
    console.log(char) // 'f', 'o', 'o' 차례대로 출력
  }
```

## Iterator
Iterator는 Iterator porotocol을 따라는 객체로 다음과 같은 Obejct를 반환하는 메소드 ``next()``를 가지고 있어야합니다.
| key                | type          | description                       |
|:-------------------|:--------------|:----------------------------------|
| done               | boolean       | Iterator의 완료여부   |
| value              | any           | Iterator에 의해 반환되는 값이며 ``done`` 이 ``true`` 일경우 'undefind'를 반환합니다.  |

그럼 이터레이터블 객체를 한번 만들어 볼까요 ??

``` javascript
  let iterableCounter = { // Iteratable Object
    [Symbol.iterator]: function(){
      let count = 0;
      return { // Iterator
        next: () =>  
          ({value: count += 1, done: false})
      }
    }
  }

  for( let count of iterableCounter) { // 반복기는 for ... of문으로 값을 가져올 수 있습니다. 
    if(count > 5) // count의 값이 5이상이되면 종료
      break;

    console.log(count); // 1 ... 5 차례대로 출력
  }
```
위의 카운터는 무한하게 카운팅이 가능한 무한 이터레이터입니다. 기능에 비해 구현하기가 조금 귀찮습니다. 매번 이렇게 만들어야할까요? es6에 추가된 Generator로 쉽게 iterator를 사용할 수 있습니다.


## Generator function

Generator function은 Generator객체를 반환하는 함수인데, ``function*`` 문법을 사용하여 작성합니다. Generator function를 호출하면 함수내 어떠한 코드도 실행되지 않고 Generator를 반환하는데. Generator 의 ``next()`` method가 호출될때마다 함수내 다음 yeild를 만날때 까지 실행되고 그 값을 반환합니다. 더이상 함수내에 yeild가 없을경우 종료됩니다. 위에서 만든 카운터를 generator function 을 이용하여 작성해 볼까요?

```javascript
  function* counter(){
    let count = 0;
    while(true)
      yield count+=1; // yield문을 만나면 실행이 종료되고 우측의 값을 반환합니다.
  }

  const iter = counter();
  console.log(iter.next()) // {value: 0, done: false} 
  console.log(iter.next()) // {value: 1, done: false} 
  // ...
  console.log(iter.next()) // {value: 234, done: false} 
```
가독성도 좋고 작성하기도 훨씬 쉬어지지 않았나요?? 그러면 조금더 유용한 함수 range를 만들어 보겠습니다.range 함수는 인자로 start와 end의 값을 받고 그사이의 값을 차례대로 반환하는 함수입니다.

```javascript
  function* range(start, end){
    for(let cur = start; cur < end; cur += 1) {
      yield cur;
    }
  }

  for(let index of range(3,6)) { // 
    console.log(index) // 3, 4, 5 차례대로 출력
  }

  console.log(range(3,6)) // Iterator 객체는 전개연산자를 사용할수 있습니다. 3, 4, 5 한번에 출력

```

### yield* 를 사용한 반복기 위임
``yield*`` 를 사용하여 다른 반복기에 역활을 위임할 수 있습니다. 

``` javascript
  function* range(start, end){
    for(let cur = start; cur < end; cur += 1) {
      yield cur;
    }
  } 

  function* counter(){
    yield "start counter";
    yield* range(1,4);
    yield "end counter";
  } 

  let myCounter = counter();
  console.log(myCounter.next().value); // start counter 출력
  console.log(myCounter.next().value); // 1 출력
  console.log(myCounter.next().value); // 2 출력
  console.log(myCounter.next().value); // 3 출력
  console.log(myCounter.next().value); // end counter 출력 
```


### Generator에 인자값 삽입

또한 Generator ``next()``을 통해 인자값을 넘길수 있습니다. 이때 yield 우측의 값을 반환하고. 좌측의 값에 전달받은 인자값을 할당하는데 
주의 해야할점은 yield의 인자값은 가장 최근의 yield에 할당되고 반환값은 다음 yield의 우측 값이라는 겁니다. 
```javascript
  function *myGen(){
    let a = yield 1; // a = 10;
    let b = yield a + 2; // b = 20;
    let c = yield b + 3; // c = 30;
    yield a + b + c;
  }

  const gen = myGen();
  gen.next() // {value: 1, done: false} 
  gen.next(10) // {value: 12, done: false}
  gen.next(20) // {value: 23, done: false}
  gen.next(30) // {value: 60, done: false}
  gent.next() // {value: undefind, done: true} 

```

## 제네레이터의 의의

이렇게 사용하면 이터레이터를 적은코드로 가독성 높게 작성할수있다. 하지만 그것이 제네레이터의 모든 기능은 아니다. 제네레이터를 이용하여 동시성/비동기 프로그래밍이 가능하다.

### 코루틴
일반적인 함수는 실행될 때 콜스택에 등록되고 종료되면 콜스택에서 제거됩니다. 함수의 실행점은 항상 함수의 실행부분이고 한번 사라진 콜스택 함수를 다시 불러올 수 있는 방법은 없습니다. 하지만 Generator는 실행중 ``yield``문을 만나게되면 스택프레임을 복사해 놓고 콜스택에서 제거한후 ``next()`` 메소드를 호출하게되면 저장된 스택프레임을 복원하여 실행합니다. 즉 진입점을 개발자가 원하는데로 설정할 수 있고 한번 올라온 컨텍스트를 원하는 만큼 유지할 수 있다. 이런개념을 코루틴coroutine이라고 한다. 

### 비동기
이런 코루틴을 활용하면 비동기코딩을 효율적으로 표현할 수 있습니다. 기존의 자바스크립트에서 콜백방식으로 다중 비동기를 코드를 작성하는 경우 이른 바 콜백지옥이라고 불리는, 가독성이 떨어지고 에러 핸들링이 힘든 모습이 되곤하였습니다.

 ```javascript
  function getId(phoneNumber, callback) { /* … */ }
  function getEmail(id, callback) { /* … */ }
  function getName(email, callback) { /* … */ }
  function order(name, menu, callback) { /* … */ }

  function orderCoffee(phoneNumber, callback) {
      getId(phoneNumber, function(id) {
          getEmail(id, function(email) {
              getName(email, function(name) {
                  order(name, 'coffee', function(result) {
                      callback(result);
                  });
              });
          });
      });
  }
 ```

 가상의 코드지만 어지럽다... 위의 작업들을 Generator와 사용하여 해결해 보자

```javascript
  function* orderCoffee(phoneNumber) {
      const id = yield getId(phoneNumber);
      const email = yield getEmail(id);
      const name = yield getName(email);
      const result = yield order(name, 'coffee');
      return result;
  }

  const iterator = orderCoffee('010-1234-1234');
  iterator.next();

  function getId(phoneNumber) {
      // …
      iterator.next(result);
  }

  function getEmail(id) {
      // …
      iterator.next(result);
  }

  function getName(email) {
      // …
      iterator.next(result);
  }

  function order(name, menu) {
      // …
      iterator.next(result);
  }
```
조금 보기 쉬워졌지만. 매번 iterator를 호출하는게 귀찮고, 무엇보다 iterator가 전역에 있기때문에 side effect가 생길 가능성이있다.
각 자동으로 next를 호출하는 함수를 만들어 해결해보겠습니다.

```javascript
function generatorToAsync(fn){
  return (...args) => new Promise((resolve, reject) => { // 함수를 반환합니다.
    const gen = fn(...args); // Generaotr객체를 만듭니다.
    function step(val){ // 인자로 key(메소드)를 받는 내부함수를 만듭니다.
      try {
        var info = gen.next(val) // 반복기 실행 
        var value = info.value; // 값을 저장합니다.
      } catch(err) {
        reject(err); // 에러처리 
      }
      if (info.done) { // 반복기가 종료되었는지 확인
        resolve(value) 
      } else {
        Promise.resolve(value).then((val) => { // Promise를 이용한 비동기 처리
          step(val) // 재귀 실행
        })
      }
    }
    step() // 최초 실행
  })
}

let orderCoffe = generatorToAsync(function* (phoneNumber){
  const id = yield getId(phoneNumber);
  const email = yield getEmail(id);
  const name = yield getName(email);
  const result = yield order(name, 'coffee');
  return result;
})

orderCoffe('010-1234-5678').then((result) => { // Promise 객체가 반환됨으로 then으로 실행
  console.log(result);
});
```
실제로는 이렇게 구현하여 처리하지않고 ``co, koa``같은 coroutine 구현을 사용하여 쉽게 사용할 수 있습니다. 그리고 es7부터 추가된 ``async / await``문을 사용하면 보다 간단하게 비동기식을 작성할 수 있습니다.


## 참조

https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Iterators_and_Generators
https://gist.github.com/qodot/ecf8d90ce291196817f8cf6117036997
https://blog.qodot.me/post/es6-generator/