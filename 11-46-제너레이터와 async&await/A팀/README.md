# 46장 제너레이터와 async/await

## 1. 제너레이터란?

- 코드 블록의 실행을 일시 중지했다가 필요한 시점에 재개할 수 있는 특수한 함수
- 제너레이터와 일반 함수의 차이
  - 제너레이터 함수는 함수 호출자에게 함수 실행의 제어권을 양도(yield)할 수 있습니다.
  - 제너레이터 함수는 함수 호출자와 함수의 상태를 주고받을 수 있습니다.
  - 제너레이터 함수를 호출하면 제너레이터 객체를 반환합니다.

<br />

## 2. 제너레이터 함수의 정의

- `function` 키워드로 선언하고, 하나 이상의 `yield` 표현식을 포함

```jsx
// 제너레이터 함수 선언문
function* genDecFunc() {
  yield 1;
}

// 제너레이터 함수 표현식
const genExpFunc = function* () {
  yield 1;
};

// 제너레이터 메서드
const obj = {
  *genObjMethod() {
    yield 1;
  },
};

// 제너레이터 클래스 메서드
class MyClass {
  *genClsMethod() {
    yield 1;
  }
}
```

- 애스터리스크(`*`)의 위치는 `function` 키워드와 함수 이름 사이라면 어디든지 상관없지만 일관성을 유지하기 위해 `function` 키워드 바로 뒤에 붙이는 것을 권장합니다.

```jsx
function* genFunc() {
  yield 1;
}
```

---

화살표 함수로 정의할 수 없습니다.

```jsx
const genArrowFunc = * () => {
	yield 1;
}; // SyntaxError: Unexpected token '*'
```

---

`new` 연산자와 함께 생성자 함수로 호출할 수 없습니다.

```jsx
function* genFunc() {
  yield 1;
}

new genFunc(); // TypeError: genFunc is not a constructor
```

<br />

## 3. 제너레이터 객체

- 제너레이터 함수를 호출하면 일반 함수처럼 함수 코드 블록을 실행하는 것이 아니라 제너레이터 객체를 생성해 반환합니다.

- 제너레이터 객체는 `Symbol.iterator` 메서드를 상속받는 이터러블입니다. (즉, 이터러블 객체는 자바스크립트의 for...of 루프에서 순회할 수 있으며, 이러한 객체의 예로는 배열(Array), 문자열(String), 맵(Map), 세트(Set) 등이 있습니다.)
- 동시에, 제너레이터 객체는 `value`, `done` 프로퍼티를 갖는 이터레이터 리절트 객체를 반환하는 `next` 메서드를 소유하는 이터레이터입니다.
- 제너레이터 객체는 이터레이터이므로 `Symbol.iterator` 메서드를 호출해서 별도로 이터레이터를 생성할 필요가 없습니다.

### 제너레이터 객체 메서드

제너레이터 객체는 `next` 메서드를 갖는 이터레이터이지만 이터레이터에는 없는 `return`, `throw` 메서드를 갖습니다.

#### `next`

- 호출시 제너레이터 함수의 `yield` 표현식까지 코드 블록을 실행합니다.
- `yield`된 값을 `value` 프로퍼티 값으로, `false`를 `done` 프로퍼티 값으로 갖는 이터레이터 리절트 객체를 반환합니다.

#### `return`

- 호출시 인수로 전달받은 값을 `value` 프로퍼티 값으로, `true`를 `done` 프로퍼티 값으로 갖는 이터레이터 리절트 객체를 반환합니다.

#### `throw`

- 호출시 인수로 전달받은 에러를 발생시킵니다.
- `undefined`를 `value` 프로퍼티 값으로, `true`를 `done` 프로퍼티 값으로 갖는 이터레이터 리절트 객체를 반환합니다.

<br />

## 4. 제너레이터의 일시 중지와 재개

- 제너레이터는 `next` 메서드와 `yield` 표현식을 통해 실행을 일시 중지했다가 필요한 시점에 다시 재개할 수 있으며, 함수 호출자와 함수의 상태를 주고받을 수 있습니다.
- 함수 호출자는 `next` 메서드를 통해 `yield` 표현식까지 함수를 실행시켜 제너레이터 객체가 관리하는 상태를 꺼내올 수 있고, `next` 메서드에 인수를 전달해서 제너레이터 객체에 상태를 밀어넣을 수 있습니다.

- `yield` 키워드: 제너레이터 함수의 실행을 일시 중지시키거나 키워드 뒤에 오는 표현식의 평가 결과를 제너레이터 함수 호출자에게 반환합니다.

### 제너레이터의 동작

- 제너레이터 함수를 호출 시, 반환하는 제너레이터 객체의 `next` 메서드를 호출하면 제너레이터 함수의 코드 블록을 실행합니다.
- 단, 일반 함수처럼 한 번에 코드 블록의 모든 코드를 일괄 실행하는 것이 아니라 `yield` 표현식까지 실행되고 일시 중지됩니다.
- 이때 함수의 제어권이 호출자로 양도됩니다. 이후 필요한 시점에 호출자가 또 다시 `next` 메서드를 호출하면 일시 중지된 코드부터 실행을 재개하기 시작하여 다음 `yield` 표현식까지 실행되고 또 다시 일시 중지됩니다.
- 이때 제너레이터 객체의 `next` 메서드는 이터레이터 리절트 객체를 반환합니다.
- `next` 메서드가 반환한 이터레이터 리절트 객체의 `value` 프로퍼티에는 `yield` 표현식에서 `yield`된 값이 할당되고 `done` 프로퍼티에는 제너레이터 함수가 끝까지 실행되었는지를 나타내는 불리언 값이 할당됩니다.

- 정리: 이처럼 `next` 메서드를 반복 호출하여 `yield` 표현식까지 실행과 일시 중지를 반복하다가 제너레이터 함수가 끝까지 실행되면 `next` 메서드가 반환하는 이터레이터 리절트 객체의 `value` 프로퍼티에는 제너레이터 함수의 반환값이 할당되고 `done` 프로퍼티에는 제너레이터 함수가 끝까지 실행되었음을 나타내는 `true`가 할당됩니다.

<br />

## 5. 제너레이터의 활용

### 5-1. 이터러블의 구현

제너레이터 함수를 사용하면 이터레이션 프로토콜을 준수해 이터러블을 생성하는 방식보다 간단히 이터러블을 구현할 수 있습니다.

### 5-2. 비동기 처리

- 제너레이터 함수는 `next` 메서드와 `yield` 표현식을 통해 함수 호출자와 함수의 상태를 주고받을 수 있습니다. 
- 이러한 특성을 활용하면 프로미스의 후속 처리 메서드없이 마치 동기 처리처럼 프로미스가 비동기 처리 결과를 반환하도록 구현할 수 있습니다.

<br />

## 6. `async`/`await`

- 극단적으로 말하면, 프로미스와 제너레이터의 조합입니다.

- 제너레이터보다 간단하고 가독성 좋게 비동기 처리를 동기 처리처럼 동작하도록 구현 가능합니다.

- `async`/`await`는 프로미스를 기반으로 동작하며, 프로미스의 후속 처리 메서드 없이 마치 동기 처리처럼 프로미스가 처리 결과를 반환하도록 구현할 수 있습니다.

### 6-1. `async`함수

- `async` 키워드를 사용해 정의하여 언제나 프로미스를 반환 (명시적으로 프로미스를 반환하지 않더라도 암묵적으로 반환값을 `resolve`하는 프로미스를 반환)

- `await` 키워드는 반드시 `async` 함수 내부에서 사용해야 합니다.
- 클래스의 `constructor` 메서드는 인스턴스를 반환해야 하지만 `async` 함수는 언제나 프로미스를 반환해야 하기 때문에 클래스의 `constructor` 메서드는 `async` 메서드가 될 수 없습니다.

### 6-2. `await`키워드

- 프로미스가 `settled` 상태가 될 때까지 대기하다가 `settled` 상태가 되면 프로미스가 `resolve`한 처리 결과를 반환합니다

- `await` 키워드는 반드시 프로미스 앞에서 사용해야 합니다.
- 모든 프로미스에 `await` 키워드를 사용하는 것은 주의해야 합니다.

### 6-3. 에러 처리

#### `try … catch`문 사용

- 콜백 함수를 인수로 전달받는 비동기 함수와는 달리, 프로미스를 반환하는 비동기 함수는 명시적으로 호출할 수 있기 때문에 호출자가 명확합니다.

#### `Promise.prototype.catch`후속 처리 메서드 사용

- `async` 함수 내에서 `catch` 문을 사용해서 에러 처리를 하지 않으면 `async` 함수는 발생한 에러를 `reject`하는 프로미스를 반환하기 때문에 `Promise.prototype.catch` 후속 처리 메서드를 사용해 에러를 캐치할 수 있습니다.
