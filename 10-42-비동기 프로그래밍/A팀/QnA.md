## 1. 태스크 큐의 종류와 차이점

둘 다 콜백 함수나 이벤트 핸들러를 일시 저장한다는 점에서 동일하지만, 태스크 큐보다 마이크로태스크 큐가 더 우선순위가 높다.
즉, 이벤트 루프는 콜 스택이 비면 먼저 마이크로태스크 큐에서 대기하고 있는 함수를 가져와 실행한다.

## 1-1. 그럼 실행 순서는 어떻게 될까요?

마이크로 태스크 큐가 더 먼저 실행된다.

## 2. 이벤트 루프의 역할은 무엇인가요?

태스크 큐와 콜스택의 실행 순서를 제어합니다.

## 3. 비동기 처리 방식의 정의와 장단점을 설명해주세요.

비동기 처리는 현재 실행 중인 태스크가 종료되지 않은 상태라 해도 다음 태스크를 곧바로 실행하는 방식을 말합니다.

장점은 블로킹(작업 중단)이 발생하지 않는다는 것이고, 단점은 실행 순서가 보장되지 않는다는 것입니다.

이런 비동기 처리는 콜백 패턴을 사용하는데, 콜백 패턴은 콜백 헬을 발생시켜 가독성을 나쁘게 하고, 비동기 처리 중 발생한 에러의 예외 처리가 곤란하고, 여러 비동기 처리를 한 번에 처리하는 데 한계가 있습니다.

## 4.

```js
function foo() {
  console.log("foo");
}

function bar() {
  console.log("bar");
}

setTimeout(foo, 0);
bar();
```

우선, 자바스크립트는 콜스택(실행 컨텍스트 스택)이 하나라 한 번에 하나의 작업만 처리할 수 있는 싱글 스레드 언어입니다.

그러나 서버 등에선 동시에 많은 작업을 처리해야 하는 경우가 있는데, 이때 브라우저 내장 기능인 이벤트 루프가 동시성을 지원합니다.

위 코드에서 setTimeout으로 호출되는 foo 함수는 비동기 함수이기 때문에 바로 실행되지 않고, 콜스택에 있다가 타이머 만료 후 태스크 큐에 푸시됩니다.

bar가 콜 스택에서 실행된 후 콜 스택에 아무 실행 컨텍스트도 존재하지 않을 때 이벤트 루프는 태스크 큐에 대기 중인 foo 함수를 콜 스택으로 푸시합니다.

태스크 큐에 여러 비동기 함수가 있더라도 콜 스택에 있는 함수가 팝된 후 하나 씩 푸시됩니다.

```
setTimeout(foo, 0) 스택에서 => (매크로) 테스크 큐에 저장 => bar() 실행 => 콜스택에서 제거 => setTimeout(foo, 0) 다시 호출 => 실행
setTimeout(foo, 0) => 브라우저 넘겨
```
