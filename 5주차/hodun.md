# Chapter 09. 데이터 조직화


## 9.1 변수 쪼개기

```js
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);
```

```js
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area):
```

### 배경

변수는 다양한 용도로 사용된다.
- 반복문에 사용되는 루프 변수
- 중간 중간 값을 저장하는 수집 변수
- 코드의 결과를 저장하는 변수 등
<br />

루프 변수, 수집 변수를 제외한 변수는 값을 단 한번만 대입해야 한다.  
<br />
대입이 두 번 이상 이뤄진다면 여러가지 역할을 수행한다는 신호다.  
<br />
역할이 둘 이상인 변수가 있다면 쪼개야 한다.  
<br />
역할 하나당 변수 하나다.  



### 절차

<br />

```js
function distanceTrabelled(scenario, time) {
  let result;
  let acc = scenario.primaryForce / scenario.mass;
  let primaryTime = Math.min(time, scenario.delay);
  result = 0.5 * acc * primaryTime * primaryTime;
  let secondaryTime = time - scenario.delay;
  if(secondaryTime > 0) {
    let primaryVelocity = acc * secondaryTime
    acc = (scenario.primaryForce + scenario.secondaryForce) / scenario.mass;
    result += primaryVelocity * secondaryTime + 0.5 * acc * secondaryTime * secondaryTime;
  }
  return result;
}
```

<br />


1. 변수를 선언한 곳과 값을 처음 대입하는 곳에서 변수 이름을 바꾼다.
2. 가능하면 이때 불변으로 선언한다.
3. 이 변수에 두 번째로 값을 대입하는 곳까지 모든 참조를 새로운 변수 이름으로 바꾼다.
4. 두 번째 대입 시 변수를 원래 이름으로 다시 선언한다.

<br />

```js
function distanceTrabelled(scenario, time) {
  let result;
  const primaryAcceleration = scenario.primaryForce / scenario.mass; // 1, 2
  let primaryTime = Math.min(time, scenario.delay);
  result = 0.5 * primaryAcceleration * primaryTime * primaryTime; // 3
  let secondaryTime = time - scenario.delay;
  if(secondaryTime > 0) {
    let primaryVelocity = primaryAcceleration * secondaryTime // 3
    let acc = (scenario.primaryForce + scenario.secondaryForce) / scenario.mass; // 4
    result += primaryVelocity * secondaryTime + 0.5 * acc * secondaryTime * secondaryTime;
  }
  return result;
}
```

<br />

5. 테스트한다.
6. 반복한다.

<br />

```js
function distanceTrabelled(scenario, time) {
  let result;
  const primaryAcceleration = scenario.primaryForce / scenario.mass;
  let primaryTime = Math.min(time, scenario.delay);
  result = 0.5 * primaryAcceleration * primaryTime * primaryTime; 
  let secondaryTime = time - scenario.delay;
  if(secondaryTime > 0) {
    let primaryVelocity = primaryAcceleration * secondaryTime 
    const secondaryAcceleration = (scenario.primaryForce + scenario.secondaryForce) / scenario.mass; // here
    result += primaryVelocity * secondaryTime + 0.5 * secondaryAcceleration * secondaryTime * secondaryTime; // here
  }
  return result;
}
```

<br />

## 9.2 필드 이름 바꾸기

```js
class Organization {
  get name() { ... }
```

```js
class Organization {
  get title() { ... }
```

<br />

### 절차

<br />

```js
const organization = { name: "애크미 구스베리", country: "GB" };
```

<br />

1. 레코드의 유효 범위가 제한적이라면 필드에 접근하는 모든 코드를 수정한 후 테스트한다. 이후 단계는 필요없다.
2. 레코드가 캡슐화되지 않았다면 우선 레코드를 캡슐화한다.

<br />

```js
class Organization {
  constructor(data) {
    this._name = data.name;
    this._country = data.country;
  }
  get name() {return this._name;}
  set name(aString) {this._name = aString;}
  get country() {return this._country;}
  set country(aCountryCode} {this._country = aCountryCode;}
}

const organization = new Organization({name: "애크미 구스베리", country: "GB"});
```

<br />


3. 캡슐화된 객체 안의 private 필드명을 변경하고, 그에 맞게 내부 메서드들을 수정한다.

<br />

```js
class Organization {
  constructor(data) {
    this._title = (data.title !== undefined) ? data.title : data.name;
    this._country = data.country;
  }
  get name() {return this._title;}
  set name(aString) {this._title = aString;}
  get country() {return this._country;}
  set country(aCountryCode} {this._country = aCountryCode;}
}

```

<br />

```js
const organization = new Organization({name: "애크미 구스베리", country: "GB"});
```
  - 이제 생성자를 호출하는 쪽에서 `"name"`과 `"title"`을 모두 사용할 수 있게 되었다.

<br />

모두 수정했다면 생성자에서 `"name"`을 사용할 수 있게 하던 코드를 제거한다.
```js
class Organization {
  constructor(data) {
    this._title = data.title
    this._country = data.country;
  }
  get name() {return this._title;}
  set name(aString) {this._title = aString;}
  get country() {return this._country;}
  set country(aCountryCode} {this._country = aCountryCode;}
}

```

<br />

4. 테스트한다.
5. 생성자의 매개변수 중 필드와 이름이 겹치는게 있다면 함수 선언 바꾸기로 변경한다.
6. 접근자들의 이름도 바꿔준다.

<br />

모두 수정했다면 생성자에서 `"name"`을 사용할 수 있게 하던 코드를 제거한다.
```js
class Organization {
  constructor(data) {
    this._title = data.title
    this._country = data.country;
  }
  get title() {return this._title;}
  set title(aString) {this._title = aString;}
  get country() {return this._country;}
  set country(aCountryCode} {this._country = aCountryCode;}
}

```

<br />

## 9.3 파생 변수를 질의 함수로 바꾸기

```js
get discountedTotal() {return this._discountedTotal;}
set discount(aNumber) {
  const old = this._discount;
  this._discount = aNumber;
  this._discountedTotal += old - aNumber;
}
```
```js
get discountedTotal() {return this._baseTotal - this._discount;}
set discount(aNumber) {this._discount = aNumber;}
```

### 절차


1. 변수 값이 갱신되는 지점을 모두 찾는다. 필요하면 변수 쪼개기를 활용해 각 갱신 지점에서 변수를 분리한다.
2. 해당 변수의 값을 계산해주는 함수를 만든다.
3. 해당 변수가 사용되는 모든 곳에 어서션을 추가하여 함수의 계산 결과가 변수의 값과 같은지 확인한다.
4. 테스트한다.
5. 변수를 읽는 코드를 모두 함수 호출로 대체한다.
6. 테스트한다.
7. 변수를 선언하고 갱신하는 코드를 죽은 코드 제거하기로 없앤다.

### 예시

<br />

```js
get production() {return this._production;}
applyAdjustment(anAdjustment) {
  this._adjustments.push(anAdjustment);
  this._production += anAdjustment.amount;
}
```

<br />

adjustment를 적용하는 과정에서 직접 관련이 없는 누적 값 production까지 갱신했다.  
이 누적 값은 매번 갱신하지 않고도 계산할 수 있다.

<br />

```js
get production() {
  assert(this._production === this.calculatedProduction);
  return this._production;
}

get calculatedProduction() {
  return this._adjustments.reduce((sum, a) => sum + a.amount, 0);
}
```

<br />

어서션이 실패하지 않으면 필드를 반환하던 코드를 수정해 직접 계산 결과를 반환하도록 한다.

<br />

```js
get production() {
  return this.calculatedProduction;
}
```

<br />

그런 다음 calculatedProduction() 메서드를 인라인한다.

<br />

```js
get production() {
  return this._adjustments.reduce((sum, a) => sum + a.amount, 0);
}
```

<br />

마지막으로 옛 변수를 참조하는 모든 코드를 죽은 코드 제거하기로 정리한다.

<br />

```js
applyAdjustment(anAdjustment) {
  this._adjustments.push(anAdjustment);
}
```

<br />

## 9.4 참조를 값으로 바꾸기

```js
class Product {
  applyDiscount(arg) {this._price.amount -= arg;}
```
```js
class Product {
  applyDiscount(arg) {
    this._price = new Money(this._price.amount - arg, this._price.currency);
  }
```

### 배경

참조냐 값이냐의 차이는 내부 객체의 속성을 갱신하는 방식에서 가장 극명하게 드러난다.

값으로 사용하는 경우 불변이기 때문에 값 객체는 대체로 자유롭게 활용하기 좋다. 몰래 바뀔 염려를 하지 않아도 된다.

### 절차

1. 후보 클래스가 불변인지, 혹은 불변이 될 수 있는지 확인한다.
2. 각각의 세터를 하나씩 제거한다.
3. 이 값 객체의 필드들을 사용하는 동치성 비교 메서드를 만든다.

### 예시

<br />

```js
//Person

constructor() {
  this._telephoneNumber = new TelephoneNumber();
}

get officeAreaCode() {return this._telephoneNumber.areaCode;}
set officeAreaCode(arg) {this._telephoneNumber.areaCode = arg;}
get officeNumber() {return this._telephoneNumber.number;}
set officeNumber(arg) {this._telephoneNumber.number = arg;}

// TelephoneNumber

get areaCode() {return this._areaCode;}
set areaCode(arg) {this._areaCode = arg;}
get number() {return this._number;}
set number(arg) {this._number = arg;}
```

<br />

세터를 제거해 전화번호를 불변으로 만들자.

```js
// TelephoneNumber

constructor(areaCode, number) {
  this._areaCode = areaCode;
  this._number = number;
}
```

```js
// Person

get officeAreaCode() {return this._telephoneNumber.areaCode;}
set officeAreaCode(arg) {
  this._telephoneNumber = new TelephoneNumber(arg, this.officeNumber);
}
get officeNumber() {return this._telephoneNumber.number;}
set officeNumber(arg) {
  this._telephoneNumber = new TelephoneNumber(this.officeAreaCode, arg);
}
```

값 객체로 인정받으려면 동치성를 값 기반으로 평가해야 한다.

직접 equals 메서드를 작성하자.

```js
// TelephoneNumber
equals(other) {
  if(!(other instanceof TelephoneNumber)) return false;
  return this.areaCode === other.areaCode &&
         this.number == other.number;
}
```

## 9.5 값을 참조로 바꾸기

```js
let customer = new Customer(customerData);
```
```js
let customer = customerRepository.get(customerData.id);
```

### 배경

예를 들어 주문 목록을 읽다 보면 같은 고객이 요청한 주문이 여러개 있을 수 있다.  
  
값으로 다룬다면 고객 데이터가 각 주문에 복사되고, 참조로 다룬다면 여러 주문이 단 하나의 데이터 구조를 참조하게 된다.  
  
고객 데이터를 갱신해야 하는 상황에 고객 데이터를 값으로 다룬다면 모든 복제본을 찾아서 빠짐없이 갱신해야 하므로 참조로 바꿔주는게 좋다.

값을 참조로 바꾸면 엔티티 하나당 객체도 단 하나만 존재하게 되는데, 데이터들을 관리할 저장소가 필요해진다.

### 절차

1. 같은 부류에 속하는 객체들을 보관할 저장소를 만든다.
2. 생성자에서 이 부류의 객체들 중 특정 객체를 정확히 찾아내는 방법이 있는지 확인한다.
3. 호스트 객체의 생성자들을 수정하여 필요한 객체를 이 저장소에서 찾도록 한다. 하나 수정할 때마다 테스트한다.

### 예시

```js
// Order

constructor(data) {
  this._number = data.number;
  this._customer = new Customer(data.customer);
}


// Customer

constructor(id) {
  this._id = id;
}

get id() {return this._id;}
```

이런 방식으로 생성한 고객 객체는 **값**

항상 물리적으로 똑같은 고객 객체를 사용하고 싶다면 저장소를 만들자.

```js
let _repositoryData;

export function initialize() {
  _repositoryData = {};
  _repositoryData.customers = new Map();
}

export function registerCustomer(id) {
  if(!_repositoryData.customers.has(id))
    _repositoryData.customers.set(id, new Customer(id));
  return findCustomer(id);
}

export function findCustomer(id) {
  return _repositoryData.customers.get(id);
}
```

만든 저장소를 적용하면

```js
// Order

constructor(data) {
  this._number = data.number;
  this._customer = registerCustomer(data.customer);
}

get customer() {return this._customer;}
```

⚠️ 전역 객체는 독한 약처럼 신중히 다뤄야 한다. 과용하면 독이 된다.

### 9.6 매직 리터럴 바꾸기

```js
function potentialEnerge(mass, height) {
  return mass * 9.81 * height;
}
```

```js
const STANDARD_GRAVITY = 9.81;
function potentialEnergy(mass, hegight) {
  return mass * STANDARD_GRAVITY * height;
}
```

### 배경

코드 자체가 뜻을 분명하게 드러내는게 좋다.  

상수로 바꾸는 방법 말고도 다른 선택지가 있는데 예를 들어 비교 로직에 쓰이는 경우 고려해볼 수 있다.

```js
aVaule === "M"

aVaule === MALE_GENDER // 상수로

isMale(aVaule) // 함수 호출로 
```

⚠️ `const ONE = 1`같은 선언은 의마가 없다.
⚠️ 리터럴이 함수 하나에서만 사용되는 경우, 그 함수가 맥락 정보를 충분히 제공해준다면 상수로 바꿔 얻는 이득이 줄어든다.

### 절차

1. 상수를 선언하고 매직 리터럴을 대입한다.
2. 해당 리터럴이 사용되는 곳을 모두 찾는다.
3. 찾은 곳 각각에서 리터럴이 새 상수와 똑같은 의미로 쓰였는지 확인하여, 같은 의미라면 상수로 대체한 후 테스트한다.
