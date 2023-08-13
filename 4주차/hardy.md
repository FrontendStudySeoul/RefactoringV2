# 6장 기본적인 리팩터링

## 6.1 함수 추출하기

함수를 추출하는 기준은 길이, 재사용성 등이 있지만 목적과 구현을 분리하는 방식이 가장 합리적인 기준으로 보입니다. 코드를 이해하기 어렵다면 그 부분을 함수로 추출한 뒤 걸맞는 이름을 짓습니다. 이러한 방식은 나중에 코드를 읽을 때 함수명으로 함수의 목적이 눈에 확 들어오고 본문 코드를 신경 쓸 일이 거의 없습니다. 이 원칙은 함수의 길이와 상관없이 적용합니다. 단 몇 줄의 코드를 담은 함수여도 코드의 목적을 잘 전달할 수 있는게 중요하다고 합니다. 이런 방식은 당연하게도 이름을 잘 지어야 효과가 발휘되므로 이름 짓기에 특별히 신경 써야합니다.

**함수 추출 절차**

1. 함수를 새로 만들고 목적을 잘 드러내는 이름을 붙인다.
    - “어떻게”가 아닌 “무엇을” 하는지가 드러나야 합니다.
    - 간단한 코드여도 함수로 뽑아서 목적이 더 잘 드러나는 이름을 붙일 수 있다면 추출합니다.
    - 하지만 좋은 이름이 생각나지 않는다면 함수로 추출하면 안된다는 신호일 수 있습니다.
2. 추출할 코드를 원본 함수에서 복사하여  함수에 붙여넣는다.
3. 추출한 코드 중 원본 함수의 지역 변수를 참조하거나 유효범위를 벗어나는 변수는 없는지 검사한다. 있다면 매개변수로 전달합니다.
    - 추출한 함수에서만 사용하는 변수가 추출한 함수 밖에 선언되어 있다면 추출한 함수 안에서 선언하도록 한다.
    - 매개변수로 넘겨줄 변수가 많은 경우엔 함수 추출을 멈추고 다른 리팩터링을 적용해 변수를 사용하는 코드를 단순하게 바꿔본 후 함수 추출을 시도합니다.
4. 변수를 다 처리했다면 컴파일합니다.
5. 원본 함수에서 추출한 코드 부분을 새로 만든 함수를 호출하는 문장으로 바꿉니다
6. 테스트
7. 다른 코드에 추출한 함수와 중복된 코드가 있는지 살피고 있다면 새 함수를 호출할지 검토한다.

**예시 코드**

```jsx
function printOwing(invoice) {
	let outstanding = 0;
	
	// 콘솔로그 구분용 배너
	console.log("*******************")
	console.log("***** 고객 채무 *****")
	console.log("*******************")
	
	// 미해결 채무를 계산한다.
	for (const o of invoice.orders) {
		outstanding += o.amount;
	}
	
	// 마감일을 기록한다.
	const today = Clock.today;
	invoice.dueDate = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 30);

	// 세부 사항을 출력한다.
	console.log(`고객명: ${invoice.customer}`)
	console.log(`채무액: ${outstanding}`)
	console.log(`마감일: ${invoice.dueDate.toLocalDateString()}`)
}
```

**리팩터링**

```jsx
function printOwing(invoice) {
	// 로그 구분용 배너
	printBanner();

	// 미해결 채무를 계산
	// 가변적인 지역 변수를 사용하기 때문에 함수에서 값을 리턴하도록 수정
  let outstanding = calculateOutstnading(invoice) 

	// 마감일 기록
	// 지역 변수를 사용하기 때문에 두 변수를 매개변수로 받도록 수정
	recordDueDate(invoice)
	// 세부 사항 출력
	printDetails(invoice, outstanding)
}
```

## 6.2 함수 인라인하기

목적이 분명히 드러나는 이름의 짤막한 함수를 이용하면 코드가 명료해지고 이해하기 쉬워지지만 때로는 함수의 본문이 이름만큼 명확한 경우도 있습니다. 

**예시 코드**

```jsx
function rating(aDriver) {
	return moreThanFiveLateDeliveries(aDriver) ? 2 : 1;
}

function moreThanFiveLateDeliveries(aDirver) {
	return aDriver.numberOfLateDeliveries > 5;
}
```

**리팩터링**

```jsx
function rating(aDriver) {
	return aDriver.numberOfLateDeliveries > 5 ? 2 : 1;
}
```

## 6.3 변수 추출하기

표현식이 너무 복잡해서 이해하기 어려울 때가 있다. 이럴 때 지역 변수를 활용하면 표현식을 쪼개 관리하기 더 쉽게 만들 수 있습니다. 그러면 복잡한 로직을 구성하는 단계마다 이름을 붙일 수 있어 코드의 목적을 훨씬 명확하게 드러낼 수 있습니다.

**예시 코드**

```jsx
return order.quantity * order.itemPrice - Math.max(0, order.quantity - 500)
* order.itemPrice * 0.05 + Math.min(order.quantity * order.itemPrice * 0.1, 100);
```

**리팩터링**

```jsx
const basePrice = order.quantity * order.itemPrice;
const quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
const shipping = Math.min(basePrice * 0.1, 100);
return basePrice - quantityDiscount + shipping;
```

## 6.4 변수 인라인하기

때로 변수는 그 이름이 원래 표현식과 다를 바 없을 때도 있습니다. 그럴 경우 인라인하는 것이 좋습니다.

**예시 코드**

```jsx
let basePrice = anOrder.basePrice;
return basePrice > 1000;
```

**리팩터링**

```jsx
return anOrder.basePrice > 1000;
```

## 6.5 함수 선언 바꾸기

함수는 소프트웨어 시스템의 구성 요소를 조립하는 연결부 역할을 합니다. 연결부를 잘 정의하면 유지보수가 편해지는 반면 잘못 정의하면 코드 파악이 어려워지고 유지보수에 치명적일 수 있습니다. 이런한 연결부에서 중요한 요소는 함수의 이름과 매개변수 입니다. 이름의 중요성은 계속해서 나오듯이 잘 지은 이름은 코드를 살펴볼 필요 없이 호출문만 보고 무슨 일을 하는지 파악할 수 있습니다. 매개변수는 함수가 외부 세계와 어우러지는 방식을 정의합니다. 때문에 어떻게 활용하는지 함수의 활용 범위가 달라질 수 있습니다. 

함수 선언 바꾸기는 간단한 절차와 마이그레이션 절차 두 가지 방법이 있습니다. 간단한 절차는 함수를 리팩터링하고 그 함수를 참조하는 부분들 모두 찾아서 바꾸는 절차입니다. 반면 마이그레이션 절차는 단번에 참조되는 부분을 수정하지 않도록 새로운 함수를 만들고 기존의 함수에 새로운 함수를 인라인하는 절차입니다.

**예시 코드**

```jsx
function cricum(radius) {
	return 2 * Math.PI * radius;
}
```

**리팩터링**

```jsx
// 간단한 절차
function cricumference(radius) {
	return 2 * Math.PI * radius;
}

// 마이그레이션 절차
function cricum(radius) {
	return cricumference(radius);
} 

function cricumference(radius) {
	return 2 * Math.PI * radius;
}
```

## 6.6 변수 캡슐화하기

데이터는 함수보다 리팩터링하기 까다롭습니다. 데이터는 참조하는 모든 부분을 한 번에 바꿔야 코드가 제대로 동작하고 데이터의 유효범위가 넓어질수록 다루기 어려워집니다. 범위가 넓은 데이터를 옮길 때는 먼저 그 데이터로의 접근을 독점하는 함수를 만드는 식으로 캡슐화하는 것이 가장 좋은 방법입니다. 

```jsx
let defaultOwnerData = { firstName: "마틴", lastName: "파울러" };
export function defaultOwner() { return defaultOwnerData };
export function setDefaultOwner(arg) { defaultOwnerData = arg }
```

```jsx
const owner1 = defaultOwner();
assert.equal("파울러", owner1.lastName, "처음 값 확인");
const owner2 = defaultOwner();
owner2.lastName = "파슨스"
assert.equal("파슨스", owner1.lastName, "onwer2를 변경한 후") // 성공할까??

// 게터가 데이터의 복제본을 반환하도록 수정한다.
export function defaultOwner() { return Object.assign({}, defaultOwnerData) };
```

## 6.7 변수 이름 바꾸기

변수는 프로그래머가 하려는 일에 관해 많은 것을 설명합니다. 하지만 이름을 잘 지었을 때만 해당합니다. 언제나 이름을 잘지어야 합니다. 간단한 임시 변수나 유효범위가 좁은 변수라면 쉽게 이름을 바꿀 수 있지만 폭넓게 쓰이는 범용적인 변수라면 캡슐화(6.6절)를 고려하는 것이 좋습니다. 

```jsx
let tpHd = "untitled";
result += `<h1>${tpHd}</h1>` // 변수를 읽기만 한다.
tpHd = "title"; // 값을 변경하기도 한다.

// 이럴 땐 주로 변수 캡슐화로 처리한다.
result += `<h1>${tpHd}</h1>`
setTitle("title");

function title() { return tpHd }
function setTitle(arg) { tpHd = arg }
```

```jsx
// 변하지 않는 상수는 캡슐화하지 않고 복제하는 방식을 이용해 점진적으로 바꿀 수 있습니다.
const cpyNm = "애크미 구스베리";

// 리팩터링
const companyName = "애크미 구스베리";
const cpyNm = companyName;
```

## 6.8 매개변수 객체 만들기

데이터 항목 여러 개가 여러 함수에서 사용되는 경우 데이터 구조를 하나로 모아주곤 합니다. 테이터 뭉치를 데이터 구조로 묶으면 데이터 사이의 관계가 명확해진다는 이점이 있습니다. 여러 데이터를 전달하던 매개변수의 수도 줄어들게 됩니다.

**예시 코드**

```jsx
function redingsOutsideRange(station, min, max) {
	return station.readings
		.filter(r => r.temp < min || r.temp > max);
}
```

**리팩터링**

```jsx
class NumberRange {
	constructor(min, max) {
		this._data = { min, max };
	}
	get min() { return min }
	get max() { return max }
}

function redingsOutsideRange(station, range) {
	return station.readings
		.filter(r => r.temp < range.min || r.temp > range.max);
}
```

## 6.9 여러 함수를 클래스로 묶기

공동 데이터를 중심으로 긴민하게 엮여 작동하는 함수 무리는 클래스로 묶습니다. 클래스로 묶으면 함수들이 공유하는 공통 환경을 더 명확하게 표현할 수 있고, 각 함수에 전달되는 인수를 줄여서 객체 안에서의 함수 호출을 간결하게 만들 수 있습니다. 또한 클라이언트가 객체의 핵심 데이터를 변경할 수 있고, 파생 객체들을 일관되게 관리할 수 있습니다.

**예시 코드**

```jsx
// 클라이언트1
const aReading = acquireReading();
const baseCharge = baseRate(areadong.month, aReading.year) * aReading.quantity;

// 클라이언트2
const aReading = acquireReading();
const base = baseRate(areadong.month, aReading.year) * aReading.quantity;
const taxableCharge = Mtah.max(0, base - taxThreshole(aReading.year));

// 클라이언트3
const aReading = acquireReading();
const basicChargeAmount = calculateBaseCharge(aReading);

const calculateBaseCharge(aReading) {
	return baseRate(areadong.month, aReading.year) * aReading.quantity;
}

// 각 클라이언트마다 baseRate가 중복됨
```

**리팩터링**

```jsx
class Reading {
	constructor(data) {
		this._customer = data.customer;
		this._quantity = data.quantity;
		this._month = data.month;
		this._year = data.year;
	}

	get customer() { return this._custormer };
	get quantity() { return this._quantity };
	get month() { return this._month };
	get year() { return this._year };

	get baseCharge(aReading) {
		return baseRate(this.month, this.year) * this.quantity;
	}
}

// 클라이언트1
const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const baseCharge = aReading.baseCharge;;

// 클라이언트2
const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const taxableCharge = Mtah.max(0, aReading.baseCharge - taxThreshole(aReading.year));
// const taxableCharge = aReading.taxableCharge

// 클라이언트3
const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const basicChargeAmount = aReading.baseCharge;
```

## 6.10 여러 함수를 변환 함수로 묶기

클래스로 여러 함수를 묶은것처럼 함수들을 변환 함수로 묶을 수 있습니다. 장점은 클래스와 같지만 둘의 중요한 차이가 있습니다. 원본 데이터가 코드 안에서 갱신될 때는 변환 함수로 묶으면 가공한 데이터를 새로운 레코드에 저장하므로 클래스로 묶는편이 훨씬 낫습니다.

```jsx
// 코드는 클래스와 동일하여 변환 함수만 작성함
function enrichReading(original) {
	const result = _.cloneDeep(original) // lodash cloneDeep
	// const result = JSON.parse(JSON.stringify(original))
	
	result.baseCharge = calculateBaseCharge(result);
	result.taxableCharge = Mtah.max(0, aReading.baseCharge - taxThreshole(aReading.year));

	return result
}

const rawReading = acquireReading();
const aReading = enrichReading(rawReading);
const baseCharge = enrichReading.baseCharge;;
```

## 6.11 단계 쪼개기

서로 다른 대상을 한꺼번에 다루는 코드는 각각의 별개 모듈로 나누는 방법을 모색합니다. 나누는 간편한 방법은 동작을 연이은 두 단계로 분리하거나 순차적으로 단계를 분리하는 것입니다. 이때 각 단계는 서로 다른 일을 수행해야 합니다. 각 단계는 자신만의 문제에 집중하기 때문에 나머지 단계에 관해서는 자세히 몰라도 이해할 수 있습니다.

**예시 코드**

```jsx
function priceOrder(product, quantity, shippingMethod) {
	const basePrice = product.basePrice * quantity;
	const discount = Math.max(quntity - product.discountThreshold, 0)
					* product.basePrice * product.discountRate;
	const shippingPerCase = (basePrice > shippingMethod.discountThreshold)
					? shippingMethod.discountedFee : shippingMethod.feePerCase;
	const shippingCose = quantity * shippingPerCase;
	const price = basePrice - discount + shippingCost;
	return price;				
}
```

**리팩터링**
