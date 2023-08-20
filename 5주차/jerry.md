# Chapter 10. 조건문 분해하기

> 조건부 로직은 프로그램을 복잡하게 만드는 요인이기도 함 -> 리팩토링의 대상

## 10.1 조건문 분해하기

> 복잡한 조건들을 하나로 묶어 함수로 만들면, 그 함수의 이름을 통해 의도를 쉽게 파악할 수 있다.

### 리팩토링 전

```ts
if (조건 1 && 조건 2) {}
```

### 리팩토링 후

```ts
if (조건()) {}

function 조건() {
  return 조건 1 && 조건 2;
}
```

## 10.2 조건식 통합하기

> 비교하는 조건은 다르지만 결과적으로 수행하는 동작은 똑같다면 하나로 통합하는 것이 좋다.

### 리팩토링 전

```ts
if (조건 1) return 0;
if (조건 2) return 0;
if (조건 3) return 0;
```

### 리팩토링 후

```ts
if (조건()) return 0;

function 조건() {
  return 조건 1 || 조건 2 || 조건 3;
}
```

## 10.3 중첩 조건문을 보호 구문으로 바꾸기

> 코드에는 명확함이 핵심이다.

- if-else로 중첩이 되는 순간 깊이가 깊어지면서 읽기가 어려운 코드가 된다.
- if와 else 구문으로 작성하면 if와 else 양 갈래가 암시적으로 중요하다는 의미를 전달하는데, 짧은 중첩 조건문이 있을땐 보호 구문으로 바꾸는게 좋다.

### 리팩토링 전

> 가변 변수를 없앨 수 있다는 장점도 있다.

```ts
function getPayAmount() {
  let result;
  if (조건 1) {
    result = 값 1;
  } else {
    if (조건 2) {
      result = 값 2;
    } else {
      result = 값 3;
    }
  }

  return result;
}
```

### 리팩토링 후

```ts
function getPayAmount() {
  if (조건 1) return 값 1;
  if (조건 2) return 값 2;

  return 값 3;
}
```

### 조건 반대로 만들기: 리팩토링 전

```ts
function getPayAmount() {
  let result = 0;

  if (값 1 > 0) {
    if (값 2 > 0 && 값 3 > 0) {
      return result = 계산된 값;
    }
  }

  return result;
}
```

### 조건 반대로 만들기: 리팩토링 전

```ts
function getPayAmount() {
  if (값 1 <= 0 || 값 2 <= 0 || 값 3 <= 0) {
    return 0;
  }

  return 계산된 값;
}
```

## 10.4 조건부 로직을 다형성으로 바꾸기

> 복잡한 조건부 로직을 클래스와 다형성을 이용하면 직관적으로 구조화할 수 있다.

- switch case별로 클래스를 하나씩 만들어 코드 중복을 없앨 수 있다.

### [자바스크립트에서의 팩토리 함수란?](https://ui.toast.com/weekly-pick/ko_20160905)

### 절차

1. 다형적 동작을 하는 클래스를 만든다. 이왕이면 적합한 인스턴스를 알아서 만들어서 반환하는 팩토리 함수도 함께 만든다.
2. 호출하는 코드에서 팩토리 함수를 사용하게 한다.
3. 조건부 로직 함수를 슈퍼클래스로 옮긴다.
4. 서브클래스 중 하나를 선택한다. 서브클래스에서 슈퍼클래스의 조건부 로직 메서드를 오버라이드한다. 조건부 문장 중 선택된 서브클래스에 해당하는 조건절을 서브클래스로 복사한 다음 적절히 수정한다.
5. 같은 방식으로 각 조건절을 해당 서브클래스에서 메서드로 구현한다.
6. 슈퍼클래스 메서드에는 기본 동작 부분만 남긴다.

### 리팩토링 전

```ts
switch (bird.type) {
  case '유럽':
    return 값 1;
  case '아프리카':
    return 값 2;
  default:
    return '알 수 없다.';
}
```

### 리팩토링 후

> 자바스크립트에서는 타입 계층 구조가 없이도 다형성을 표현할 수 있기에 슈퍼클래스인 Bird는 없어도 상관없다. 객체가 적절한 이름의 메서드를 구현하고 있다면 아무 문제없이 같은 타입으로 취급하기 때문이다. 이를 `덕 타이핑`이라고 한다.

```ts
function plumage(bird) {
  return createBird(bird).plumage;
}

function createBird(bird) {
  switch (bird.type) {
    case '유럽':
      return new 유럽새(bird);
    case '아프리카':
      return new 아프리카새(bird);
    default:
      return new 새(bird);
  }
}

class 새 {
  construcotr(birdObject) {
    //
  }

  get plumage() {
    return '알 수 없다.'
  }
}

class 유럽새 extends 새 {
  get plumage() {
    return 값 1;
  }
}

class 아프리카새 extends 새 {
  get plumage() {
    return 값 2;
  }
}
```

## 10.5 특이 케이스 추가하기

> 데이터 구조의 특정 값을 확인한 후 같은 동작을 수행하는 코드가 여러 곳에서 등장한다면 하나의 함수로 빼내 중복 코드를 줄일 수 있다.

### 리팩토링 전

> 다음 문자열을 이용한 조건을 여러 곳에서 사용하고 있는 경우

```ts
class Site {
  get customer() {
    return this._customer;
  }
}

class Customer {
  constructor() {}

  get name() {}
  get billingPlan() {}
  set billingPlan(arg) {}
  get paymentHistory() {}
}

// 클라이언트 1
const aCustomer = site.customer;

let customerName;

if (aCustomer === "미확인 고객") {
  customerName = "거주자";
} else {
  customerName = aCustomer.name;
}

// 클라이언트 2
const plan =
  aCustomer === "미확인 고객"
    ? registry.billingPlans.basic
    : aCustomer.billingPlan;

// 클라이언트 3
if (aCustomer !== "미확인 고객") {
  aCustomer.billingPlan = newPlan;
}

// 클라이언트 4
const weeksDelinquent =
  aCustomer === "미확인 고객"
    ? 0
    : aCustomer.paymentHistory.weeksDelinquentInLastYear;
```

### 리팩토링 후

- UnknownCustomer 클래스가 Customer 클래스의 서브 클래스가 아닌 이유: 자바스크립트의 서브 클래스 규칙과 동적 타이핑 능력을 가지므로 이 경우에는 이렇게 작성하는 편이 낫다.

```ts
class Site {
  get customer() {
    return this._customer === "미확인 고객"
      ? new UnknownCustomer()
      : this._customer;
  }
}

class Customer {
  constructor() {}

  get name() {}
  get billingPlan() {}
  set billingPlan(arg) {}
  get paymentHistory() {}
  get isUnknown() {
    return false;
  }
}

class UnknownCustomer {
  get isUnknown() {
    return true;
  }
  get name() {
    return "거주자";
  }
  get billingPlan() {
    return registry.billingPlans.basic;
  }
  set billingPlan(arg) {}
  get paymentHistory() {
    return new NullPaymentHistory();
  }
}

class NullPaymentHistory {
  get weeksDelinquentInLastYear() {
    return 0;
  }
}

function isUnknown(arg) {
  if (!(arg instanceof Customer || arg instanceof UnknownCustomer))
    throw new Error("에러");

  return arg.isUnknown;
}

// 또는 간단하게 조건 검사 부분을 함수로 빼낼 수 있다.
function isUnknown(arg) {
  return arg === "미확인 고객";
}

// 사용할 때
const aCustomer = site.customer;

// 클라이언트 1
const customerName = aCustomer.name;

// 클라이언트 2
const plan = aCustomer.billingPlan;

// 클라이언트 3
aCustomer.billingPlan = newPlan;

// 클라이언트 4
const weeksDelinquent = aCustomer.paymentHistory.weeksDelinquentInLastYear;
```

## 10.6 어서션 추가하기

> 특정 조건이 참일 때만 제대로 동작하는 코드 영역에 어서션을 추가하면 된다.

> 어서션: 항상 참이라고 가정하는 조건부 문장이다. 어서션의 존재 여부에 따라 동작에 아무런 영향이 없도록 작성되어야 한다. 어서션은 오류의 출처를 파악하기에 용이하다. 하지만 남발하는 것은 위험하며 `반드시 참이어야 하는` 곳에만 작성하는 것이 좋다.ㄴ

### 예시

```ts
// Customer 클래스 내부
applyDiscount(price) {
  if(!this.discountRate) return price;
  else {
    assert(this.discountRate >= 0);
    return price - (this.discountRate * price);
  }
}
```

## 10.7 제어 플래그를 탈출문으로 바꾸기

> 흔히 반복문 안에서 제어 플래그를 사용하는데, 제어 플래그 대신 제어문인 break나 continue, return을 사용해 명확하게 작성할 수 있다.

### 리팩토링 전

> 제어 흐름을 변경하는데 쓰이는 제어 플래그를 이용한 코드

```ts
let flag = false;

for (const p of people) {
  if (!flag) {
    if (p === "값 1") {
      // ...
      flag = true;
    }
    if (p === "값 2") {
      // ...
      flag = true;
    }
  }
}
```

### 리팩토링 후

> 제어 플래그가 참이면 반복문에서 더 이상 할 일이 없으므로 break나 return문으로 바로 빠져나오게 하자.

```ts
for (const p of people) {
  if (!flag) {
    if (p === "값 1") {
      // ...
      return;
    }

    if (p === "값 2") {
      // ...
      return;
    }
  }
}

// 또는
if (people.some((p) => ["값 1", "값 2"].includes(p))) {
  // 특정 동작
}
```
