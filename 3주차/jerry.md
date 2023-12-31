# Chapter 04. 테스트 구축하기

> 테스트 스위트(text suite): 테스트 케이스들을 하나로 묶은 것을 말한다.

## 4.1 자가 테스트 코드의 가치

- `회귀 버그`를 잡는데에 시간이 단축된다.

> 회귀 버그: 잘 작동하던 기능에 문제가 생기는 현상

> 회귀 테스트: 잘 작동하던 기능이 여전히 잘 작동하는지 확인하는 테스트

- `TDD(테스트 주도 개발)`: 테스트-코딩-리팩터링 과정을 여러 차례 진행해 생산적으로 코드를 작성할 수 있다.

- 리팩터링에는 테스트가 필요하다.

## 4.2 테스트할 샘플 코드

## 4.3 첫 번째 테스트

- 픽스처: 테스트에 필요한 데이터와 객체

```js
describe('province', function() {
  if('shortfall', function() {
    const asia = new Province(sampleProvinceData()); // 픽스처
    expect(asia.shortfall).equal(5); // 검증
  })
})
```

- 오류 상황에서도 올바르게 오류를 반환하는지 확인할 필요가 있다.
- 실패해야 할 상황에서는 반드시 실패하게 만들자.
- 코드에서 어떤 부분이 훼손될 때 그리고 오직 훼손된 경우에만 테스트가 실패하도록 하는 것이 중요하다.

## 4.4 테스트 추가하기

> 여러 테스트에서 똑같은 픽스처를 사용하는 경우, 표준 픽스처 설정하기

### ❌ 테스트끼리 상호작용하게 되는 공유 픽스처를 만든 경우

asia는 참조값을 갖기 때문에 특정 테스트에서 이 객체 값을 수정하면 이 픽스처를 사용하는 다른 테스트에서 실패할 수 있다.

```js
describe("province", function () {
  const asia = new Province(sampleProvinceData()); // 공유 픽스처

  it("shortfall", function () {
    expect(asia.shortfall).equal(5);
  });

  it("profit", function () {
    expect(asia.profit).equal(230);
  });
});
```

### ✅ 독립적인 픽스처

beforeEach 구문을 작성하면 각각의 테스트 전에 실행되어 asia를 초기화하기 때문에 모든 테스트를 독립적으로 구성할 수 있다.

각 it 블록마다 픽스처를 작성할 수도 있지만, beforeEach 구문을 작성하는 이유는 모든 테스트들이 똑같은 픽스처에 기초해 검증을 수행한다는 것을 명시적으로 개발자에게 알릴 수 있기 때문이다.

```js
describe("province", function () {
  let asia;
  beforeEach(function () {
    asia = new Province(sampleProvinceData()); // 표준 픽스처
  });

  it("shortfall", function () {
    expect(asia.shortfall).equal(5);
  });

  it("profit", function () {
    expect(asia.profit).equal(230);
  });
});
```

## 4.5 픽스처 수정하기

- 픽스처의 수정은 대부분 세터에서 발생하며, 세터는 보통 단순해 버그가 생길일이 없기에 잘 테스트하지 않는다.

- it 구문 하나당 검증은 하나만 수행하는 것이 좋다. 앞 검증이 실패하면 다음 검증을 수행하지 못하기 때문에 실패 원인을 파악하는데에 유용한 정보를 놓치기 쉽다.

## 4.6 경계 조건 검사하기

> 범위를 벗어나는 경계 지점에서 문제가 생기는 경우의 테스트도 작성하면 좋다.

- 입력값의 최소값은 0이지만 의도적으로 음수값을 넣어 테스트하는 경우
- 테스트 대상 값이 의도한 자료형으로 출력되는지

```js
describe("negative demand", function () {
  asia.demand = -1;
  expect(asia.shortfall).equal(-26);
  expect(asia.profit).equal(-10);
});
```

수요가 음수일 때, 수익이 음수가 나온다는 상황은 없겠지만 문제가 생길 가능성이 있는 경계 조건을 생각해보고 테스트해보아야 한다.

```js
describe('string for producers', function() {
  if('', function() {
    const data = {
      name: "String producers",
      producers: "",
      demand: 30,
      price: 20,
    };
    const prov = new Province(data);
    expect(prov.shortfall).equal(0); // 검증
  })
})

1 failing
string for producers:
TypeError: doc.producers.forEach is not a function...
```
단순히 shortfall이 0이 아니라는 실패 메시지를 출력하는 대신 검증 단계 이전에 발생한 예외 상황을 출력한다.

## 좋은 단위 테스트가 가져야 할 5가지

- `훼손의 정확한 감지`: 테스트는 코드가 실제로 훼손된 경우에만 실패해야 한다.
- `세부 구현 사항에 독립적`: 세부 구현 사항을 변경하더라도 테스트 코드는 변경하지 않는 것이 이상적이다.
- `잘 설명되는 실패`: 테스트 실패의 원인과 문제점을 명확하게 설명해야 한다.
- `이해할 수 있는 테스트 코드`: 테스트 코드가 무엇을 테스트하는지와 어떻게 테스트가 수행되는지 이해할 수 있어야 한다.
- `쉽고 빠르게 실행`: 단위 테스트가 느려서는 개발 시간이 낭비되지 않도록 해야한다.
