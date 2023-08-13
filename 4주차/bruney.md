# 7장
**모듈을 분리하는 가장 중요한 기준**: 각 모듈이 자신을 제외한 다른 부분에 드러내지 않아야 할 비밀을 얼마나 잠 숨기느냐
- 레코드 캡슐화하기
- 컬렉션 캡슐화하기
- 기본형을 객체로 바꾸기(기본형 데이터를 캡슐화할 수 있음.)

리팩토링할 때 임시 변수가 자주 걸리적거리는데, 정확한 순서로 계산해야 하고 리팩터링후에도 그 값을 사용하는 코드에서 접근할 수 있어야 하기 때문. 이럴 때는 **임시 변수를 질의 함수로 바꾸기**가 상당히 도움된다. 특히 길이가 너무 긴 함수를 쪼개는 데 유용함.

클래스는 본래 정보를 숨기는 용도로 설계됨. **여러 함수를 클래스로 묶기, 클래스 추출하기, 클래스 인라인하기**로 클래스를 리팩터링할 수 있다.

클래스는 내부 정보뿐 아니라 클래스 사이의 연결 관계를 숨기는 데도 유용하다. 이 용도로는 **위임 숨기기(7장)**가 있다. but 너무 많이 숨기려다 보면 인터페이스가 비대해질 수 있으니 반대 기법인 **중개자 제거하기**도 필요하다.

가장 큰 캡슐화 단위는 클래스와 모듈이지만 함수도 구현을 캡슐화한다. 때로는 알고리즘을 통째로 바꿔야 할 때가 있는데, **함수 추출하기**로 알고리즘 전체를 함수 하나에 담은 뒤 **알고리즘 교체하기**를 적용하면 된다.

## 7.1 레코드 캡슐화하기
객체를 사용하면 어떻게 저장했는지를 값 각각의 메서드로 제공할 수 있다. 사용자는 무엇이 저장된 값이고 무엇이 계산된 값인지 알 필요가 없다. 캡슐화하면 이름을 바꿀 때도 좋다. 필드 이름을 바꿔도 기존 이름과 새 이름 모두를 각 메서드로 제공할 수 있어서 사용자 모두가 새로운 메서드로 옮겨갈 때까지 점진적으로 수정할 수 있다.

- 가변 데이터: 객체
- 불변 데이터: 시작, 끝, 길이를 모두 구해서 레코드에 저장한다.

레코드 구조
- 필드 이름을 노출하는 형태
- 내가 원하는 이름을 쓸 수 있는 형태(주로 라이브러리에서 해시, 맵, 해시맵, 딕셔너리, 연관 배열 등의 이름으로 제공한다.)

**절차**
1. 레코드를 담은 변수 캡슐화
2. 레코드를 감산 단순한 클래스로 해당 변수의 내용 교체. 이 클래스에 원본 레코드를 반환하는 접근자도 정의하고, 변수를 캡슐화하는 함수들이 이 접근자를 사용하도록 수정
3. 테스트
4. 원본 레코드 대신 새로 정의한 클래스 타입의 객체를 반환하는 함수들을 새로 만든다.
5. 레코드를 반환하는 예전 함수를 사용하는 코드를 4에서 만든 새 함수를 사용하도록 바꾼다. 필드에 접근할 대는 객체의 접근자를 사용한다. 적절한 접근자가 없다면 추가한다. 한 부분을 바꿀 때마다 테스트
6. 클래스에서 원본 데이터를 반환하는 접근자와 원본 레코드를 반환하는 함수들을 제거
7. 테스트
8. 레코드의 필드로 데이터 구조인 중첩 구조라면 레코드 캡슐화하기와 컬렉션 캡슐화하기(7.2절)를 재귀적으로 적용

### 예시: 간단한 레코드 캡슐화
```js
const organization = { name: '애크미 구스베리', country: 'gb'}; // 정의
result += `<h1>${organization.name}</h1>`; // 읽기 예
organization.name = newName; // 쓰기 예
```

1. 상수(변수) 캡슐화
```js
function getRawDataOfOrganization() {return organization};
result += `<h1>${getRawDataOfOrganization().name}</h1>`; // 읽기 예
getRawDataOfOrganization().name = newName; // 쓰기 예
```

2. 레코드를 클래스로 바꾸기
4. 새 클래스의 인스턴스를 반환하는 함수 새로 만들기
```js
class Organization {
  contructor(data) [
    this._data = data;
  ]
}

const organization = new Organization({name: '애크미 구스베리', country: 'gb'})
function getRawDataOfOrganization() {return organization._data};
function getOrganization() {return organization};
```

5. 레코드를 갱신하던 코드를 모두 getterm, setter를 사용하도록 고치기
```js
set name(aString) {this._data.name = aString}
getOrganization().name = newName;

get name() {return this._data.name}
result += `<h1>${getOrganization().name}</h1>`;
```
6. 임시 함수(getRawDataOfOrganization) 제거
7. _data 안의 필드를 class안에 펼쳐놓으면 더 깔끔함.

-> 입력 데이터 레코드와의 연결을 끊어준다는 이점이 있음. 특히 이 레코드를 참조하여 캡슐화를 깰 우려가 있는 코드가 많을 때 좋음. 데이터를 개별 필드로 펼치지 않았다면 _data를 대입할 때 복제하는 식으로 처리하면 됨.

### 예시: 중첩된 레코드 캡슐화하기
중첩 정도가 심할수록 읽거나 쓸 때 데이터 구조 안으로 더 깊숙히 들어가야 한다.

```js
customerData[customerID].usages[year][month] = amount; // 쓰기 예
function compareUsage(customerID, laterYear, month) {
  const later = customerData[customerID].usages[laterYear][month];
  const earlier = customerData[customerID].usages[laterYear - 1][month];
  return {laterAmount: later, change: later - ealier};
}
```

이전 예시와 절차는 비슷하지만 달라지는 점은 데이터를 쓰는 코드이다.
```js
getRawDataOfCustomers()[customerID].usages[year][month] = amount;
```

기본 절차에 따르면 객체를 반환하고 필요한 접근자를 만들어서 사용하게 하면 된다. 세터를 만들고 클래스로 옮긴다.ㄴ

```js
function setUsage(customerID, year, month, amount){
  getRawDataOfCustomers()[customerID].usages[year][month] = amount;
}
```

캠슐화에서는 값을 수정하는 부분을 명확하게 드러내고 한 곳에 모아두는 일이 굉장히 중요하다.

이렇게 쓰는 부분을 찾아 수정하다 보면 빠진 것이 없는지 궁금할 때,
1. 데이터를 깊은 복사하여 반환하는 방법으로 확인할수 있다.(lodash cloneDeep 사용하면 됨.)
2. 데이터 구조의 읽기전용 프락시를 반환하는 방법도 있다. 클라이언트에서 내부 객체를 수정하려 하면 프락시가 예외를 던지도록 하는 것이다. js로 구현하기에는 쉽지 않아 독자에게 연습 문제로 남긴다..
3. 복제본을 만들고 이를 재귀적으로 동결(freeze)해서 쓰기 동작을 감지하는 방법도 있다.
- setter와 같은 방법 적용. 읽는 코드를 모두 독립 함수로 추출하여 클래스로 옮기는 것
```js
usage(customerID, year, month) {
  return this._data[customerID].usages[year][month];
}

function compareUsage(customerID, laterYear, month) {
  const later = customerData[customerID].usage(customerID, laterYear, month);
  const earlier = customerData[customerID].usage(customerID, laterYear - 1, month);
  return {laterAmount: later, change: later - ealier};
}
```

가장 큰 장점은 customerdata의 모든 쓰임을 명시적인 API로 제공한다는 것. 데이터 사용 방법을 모두 파악할 수 있다. but 읽는 패턴이 다양하면 그만큼 작성할 코드가 늘어남.

4. 클라이언트가 데이터 구조를 요청할 때 실제 데이터를 제공해도 된다. but 클라이언트가 데이터를 직접 수정하지 못하게 막을 방법이 없어서 **모든 쓰기를 함수 안에서 처리한다**는 캡슐화의 원칙이 깨진다. 복제해서 제공하면 된다.
```js
get rawData() {
  return _.cloneDeep(this._data);
}
```

but 데이터 구조가 클수록 복제 비용이 커져서 성능이 느려진다. but 성능 비용을 감당할 수 있는 상황일 수도 있다. 걱정만 하지 않고 얼마나 영향을 주는지 실제로 측정을 해보자. 또 다른 문제는 **클라이언트가 원본을 수정한다고 착각할 수 있다는 것이다.** 이럴 때는 읽기전용 프락시를 제공하거나 복제본을 동결시켜서 데이터를 수정하려 할 때 에러를 던지도록 만들어라.(2번처럼)

## 7.2 컬렉션 캡슐화하기
### 배경
필자는 가변 데이터를 모두 캡슐화하는 편이다. 그러면 데이터 구조가 언제 어떻게 수정되는지 파악하기 쉬워서 필요한 시점에 데이터 구조를 변경하기도 쉬워지기 때문이다. 객체 지향 개발자들은 캡슐화를 적극 권장하는데 컬렉션을 다룰 때는 곧잘 실수를 저지른다. 컬렉션 변수로의 접근을 캡슐화하면서 getter가 컬렉션 자체를 반환하도록 한다면, 그 컬렉션을 감싼 클래스가 눈치채지 못하는 상태에서 컬렉션의 원소들로 바뀌어버릴 수 있다.

이 문제를 방지하기 위해 add, remove라는 이름의 컬렉션 변경자 메서드를 만든다. 항상 컬렉션을 소유한 클래스를 통해서만 원소를 변경하도록 하면 프로그램을 개선하면서 컬렉션 변경 방식도 원하는 대로 수정할 수 있다.

but 컬렉션 getter가 원본 컬렉션을 반환하지 않게 만들어서 클라이언트가 실수로 바꿀 가능성을 차단하는 게 낫다.

**내부 컬렉션을 직접 수정하지 못하게 컬렉션 값을 반환하지 않게 할 수 있다.** 컬렉션을 접근하려면 컬렉션이 소속된 클래스의 적절한 케서드를 반드시 거치게 하는 것이다. aCustomer.orders.size()를 aCustomer.numberOfOrder()로 바꾸는 것이다. 필자는 이 방식에 동의하지 않는다. 최신 언어는 다양한 컬렉션 클래스들을 표준화된 인터페이스로 제공하며, 컬렉션 파이프라인과 같은 패턴을 적용하여 다채롭게 조합할 수 있다. 표준 인터페이스 대신 전용 메서드들을 사용하게 하면 부가적인 코드가 상당히 늘어나며 컬렉션 연산들을 조합해 쓰기도 어려워진다.

또 다른 방법은 **컬렉션을 읽기전용으로 제공할 수 있다.** 프락시가 내부 컬렉션을 읽는 연산은 그대로 전달하고, 쓰기는 예외 처리하여 모두 막는 것이다. 이터레이터나 열거형 객체를 기반으로 컬렉션을 조합하는 라이브러리들은 이터레이터에서 내부 컬렉션을 수정할 수 없게 한다.

가장 흔한 방식은 컬렉션 getter를 제공하되 내부 컬렉션의 복제본을 반환하는 것이다. 컬렉션이 상당히 크다면 성능 문제가 발생할 수 있다. but 성능에 지장을 줄만큼 컬렉션이 큰 경우는 별로 없으니 성능에 대한 일반 규칙을 따르도록 하자.

중요한 점은 **코드베이스에서 일관성을 주는 것이다.** 앞에 나온 방식 중에서 한 가지만 적용해서 컬렉션 접근 함수의 동작 방식을 통일해야 한다.

### 절차
1. 아직 컬렉션을 캡휼화하지 않았다년 7.1의 변수 캡슐화하기부터 한다.
2. 컬렉션에 원소를 add/remove하는 함수를 추가한다.
3. 정적 검사 수행
4. 컬렉션을 참조하는 부분을 모두 찾는다. 컬렉션의 변경자를 호출하는 코드가 모두 앞에서 추가한 add/remove 함수를 호출하도록 수정한다. 하나씩 수정할 때마다 테스트
5. 컬렉션 getter를 수정해서 원본 내용을 수정할 수 없는 읽기전용 프락시나 본제본을 반환하게 한다.
6. 테스트

### 예시
```js
// Person 클래스
constructor(name) {
  this._name = name;
  this._courses = [];
}
get name() {return this._name;}
get courses() {return this._courses;}
set courses(aList) {this._courses = aList;}

// Course 클래스
constuctor(name, isAdvanced) {
  this._name = name;
  this._isAdvanced = [];
}
get name() {return this._name;}
get isAdvanced() {return this._isAdvanced;}
```

```js
numAdvancedCourses = aPerson.courses
    .filter(c => c.isAdvanced)
    .length
```
모든 필드가 접근자 메서드로 보호받고 있으니 이렇게만 해도 데이터를 제대로 캡슐화했다고 생각하기 쉽다. 하지만 허점이 있다. setter를 이용해 수업 컬렉션을 통째로 설정한 클라이언트는 누구든 이 컬렉션을 마음대로 수정할 수 있기 때문이다.
```js
const basicCouseNames = readBasicCourseNames(filename);
aPerson.courses = basicCouseNames.map(name => new Cousre(name, false))
```

클라이언트 입장에서는 다음처럼 수업 목록은 직접 수정하는 것이 훨씬 편할 수 있다.
```js
for(const name of readBasicCourseNames(filename)) {
  aPerson.courses.push(new Course(name, false));
}
```
이런 식으로 목록을 갱신하면 Person 클래스가 더는 컬렉션을 제어할 수 없으니 캡슐화가 깨진다. 필드를 참조하는 과정만 캡슐화했을 뿐 필드에 담긴 내용은 캡슐화하지 않은 게 원인이다.

2. 클라이언트가 수업을 하나씩 추가하고 제거하는 메서드를 Person에 추가
```js
addCourse(aCourse) [
  this._courses.push(aCourse);
]
removeCousre(aCousre, fnIfAbsent = () => {throw new RangeError();}) {
  const index = this._courses.indexOf(aCourse);
  if(index === -1) fnIfAbsent();
  else this._courses.splice(index, 1);
}
```
4. 컬렉션의 변경자를 직접 호출하던 코드를 모두 찾아서 방금 추가한 메서드를 사용하도록 바꾼다.
```js
for(const name of readBasicCourseNames(filename)) {
  aPerson.addCourse(new Course(name, false));
}
```

2번처럼 개별 원소를 추가하고 제거하는 메서드를 제공하기 때문에 setter를 제거. setter를 제공해야 할 특별한 이유가 있다면 인수로 받은 컬렉션의 복제본을 필드에 저장
```js
set courses(aList) {this._courses = aList.slice();}
```
아무런 목록을 변경할 수 없도록 복제본을 제공하면 된다.
```js
get courses() {return this._courses.slice() }
```

컬렉션에 대해서는 어느 정도 강박증을 갖고 불필요한 복제본을 만드는 편이, 예상치 못한 수정이 촉발한 오류를 디버깅하는 것보다 낫다. 컬렉션 관리를 책임지는 클래스라면 항상 복제본을 제공해야 한다.

## 7.3 기본형을 객체로 바꾸기
### 배경
개발이 진행되면서 간단했던 정보들이 더 이상 간단하지 않게 변한다. 금세 중복 코드가 늘어나서 사용할 때마다 드는 노력도 늘어나게 된다.

필자는 단순한 출력 이상의 기능이 필요해지는 순간 그 데이터를 표현하는 전용 클래스를 정의하는 편이다. 시작은 기본형 데이터를 단순히 감싼 것과 큰 차이가 없을 것이라 효과가 미미하지만 나중에 특별한 동작이 필요해지면 클래스에 추가하면 되니 프로그램이 커질수록 점점 유용한 도구가 된다. 코드베이스에 미치는 효과는 놀라울 만큼 크다.

### 절차
1. 7.2.1 할 것
2. 단순한 값 클래스를 만든다. 생성자는 기존 값을 인수로 받아서 저장하고, 이 값을 반환하는 getter를 추가
3. 정적 검사 수행
4. 값 클래스의 인스턴스를 새로 만들어서 필드에 저장하도록 getter를 수정. 이미 있다면 필드의 타입을 적절히 변경
5. 새로 만든 클래스의 getter를 호출한 결과를 반환하도록 getter를 수정
6. 테스트
7. 함수 이름을 바꾸면 원본 접근자의 동작을 더 잘 드러낼 수 있는지 검토

### 예시
레코드 구조에서 데이터를 읽어 들이는 단순한 주문 클래스. 클래스의 우선 순위 속성은 값을 간단히 문자열 표현함.

```js
constructor(data) {
  this.priority = data.priority;
  // ...
}

highPriorityCount = orders.filter(o => 'high' === o.priority || 'rush' === o.priority).length;
```
1. 데이터 값을 다루기 전에 항상 변수부터 캡슐화한다.
```js
get priority() {return this._priority;}
set priority(aString) {this._priority = aString;}
```
우선순위 속성을 초기화하는 생성자에서 방금 정의한 setter를 사용할 수 있다. 필드를 자가 캡슐화하면 필드 이름을 바꿔도 클라이언트 코드는 유지할 수 있다.

2. 우선순위 속성을 표현하는 값 클래스 Priority를 만든다. 표현할 값을 받는 생성자와 그 값을 문자열로 반환하는 변환 함수로 구성된다.ㄴ
```js
class Priority {
  constructor(value) {this._value = value;}
  toString() {return this._value;}
}
```
필자는 이 상황에서는 개인적으로 getter보다는 변환 함수(toString)을 선호한다. 클라이언트 입장에서 보면 속성 자체를 받은 게 아니고 해당 속성을 문자열로 표현한 값을 요청한 게 되기 때문
4. 새로 만든 클래스 사용하도록 접근자 수정
7. Order클래스의 함수 이름 바꿔주기
```js
get priorityString() {return this._priority.toString();}
set priority(aString) {this._priority = new Priority(aString);}

highPriorityCount = orders.filter(o => 'high' === o.priorityString || 'rush' === o.priorityString).length;
```

## 7.4 임시 변수를 질의 함수로 바꾸기
### 배경
함수 안에서 어떤 코드의 결과값을 뒤에서 다시 참조할 목적으로 임시 변수를 쓰기도 한다. 임시 변수를 사용하면 값을 계산하는 코드가 반복되는 걸 줄이고 값의 의미를 설명할 수도 있어서 유용하다. 그런데 아예 함수로 만들어 사용하는 편이 나을 때가 많다.

긴 함수의 한 부분을 별도 함수로 추출하고자 할 때 먼저 변수들을 각각의 함수로 만들면 일이 수월해진다. 추출한 함수에 변수를 따로 전달할 필요가 없어지기 때문이다. 이 덕분에 추출한 함수와 원래 함수의 경계가 더 분명해지기도 하는데, 부자연스러운 의존 관계나 부수효과를 찾고 제거하는 데 도움이 된다.

변수 대신 함수로 만들어두면 비슷한 계산을 수행하는 다른 함수에서도 사용할 수 있어 코드 중복이 줄어든다.

이번 리팩터링은 클래스 안에서 적용할 때 효과가 가장 크다. 클래스는 추출할 메서드들에 공유 컨텍스트를 제공하기 때문이다. 클래스 바깥의 최상위 함수로 추출하면 매개변수가 너무 많아져서 함수를 사용하는 장점이 줄어든다. 중첩 함수를 사용하면 이런 문제는 없지만 관련 함수들과 로직을 널리 공유하는 데 한계가 있다.

임시 변수를 질의 함수로 바꾼다고 다 좋아지는 건 아니다. 변수는 값을 한 번만 게산하고, 그 뒤로는 읽기만 해야 한다. 변수에 값을 한 번 대입한 뒤 더 복잡한 코드 덩어리에서 여러 차례 다시 대입하는 경우는 모두 질의 함수로 추출해야 한다. 또한, 이 계산 로직은 변수가 다음번에 사용될 때 수행해도 똑같은 결과를 내야 한다. 그래서 "옛날 주소"처럼 스냅샷 용도로 쓰이는 변수에는 이 리팩터링을 적용하면 안된다.

### 절차
1. 변수가 사용되기 전에 값이 확실히 결정되는지, 변수를 사용할 때마다 계산 로직이 매번 다른 결과를 내지는 않는지 확인한다.
2. 읽기전용으로 만들 수 있는 변수는 읽기전용으로 만든다.
3. 테스트
4. 변수 대입문을 함수로 추출
5. 테스트
6. 변수 인라인하기로 임시 변수 제거

### 예시
```js
constructor(quantity, item) {
  this._quantity = quantity;
  this._item = item;
}

get price() {
  var basePrice = this._quantity * this._item.price;
  var discountFactor = 0.98;

  if(basePrice > 1000) discountFactor -= 0.03;
  return basePrice * discountFactor;
}
```

2. basePrice을 const로 만들고 3. 테스트
4. 대입문의 우변을 getter로 추출
```js
get basePrice() {
  return this._quantity * this._item.price;
}
get price() {
  const basePrice = this.basePrice;
  var discountFactor = 0.98;
  if(basePrice > 1000) discountFactor -= 0.03;
  return basePrice * discountFactor;
}
```
5. 테스트한 다음 6. 변수를 인라인

discountFactor변수로 같은 순서로 처리

## 7.5 클래스 추출하기
### 배경
클래스는 반드시 명확하게 추상화하고 소수의 주어진 역할만 처리해야 한다는 가이드라인을 들어봤을 것이다. but 실무에서는 몇 가지 연산을 추가하고 데이터도 보강하면서 클래스가 점점 비대해지곤 한다. 기존 클래스를 굳이 쪼갤 필요까지는 없다고 생각하여 새로운 역할을 덧씌우기 쉬운데, 역할이 갈수록 많아지고 새끼를 치면서 클래스가 굉장히 복잡해진다.

메서드와 데이터가 너무 많은 클래스는 적절히 분리하는 것이 좋다. 특히 일부 데이터와 메서드를 묶을 수 있다면 어서 분리하라는 신호다. 함께 변경되는 일이 많거나 서로 의존하는 데이터들도 분리한다. 특정 데이터나 메서드 일부를 제거하면 어떤 일이 일어나는지 자문해보면 판단에 도움이 된다. 제거해도 다른 필드나 메서드들이 논리적으로 문제가 없다면 분리할 수 있다는 뜻이다.

### 절차
1. 클래스의 역할을 분리할 방법을 정한다.
2. 분리될 역할을 담당할 클래스를 새로 만든다;.
3. 원래 클래스의 생성자에서 새로운 클래스의 인스턴스를 생성하여 필드에 저장해둔다.
4. 분리될 역할에 필요한 필드들을 새 클래스로 옮긴다. 하나씩 옮길 때마다 테스트
5. 메서드들도 새 클래스로 옮긴다. 이때 저수준 메서드, 즉 다른 메서드를 호출하기보다는 호출을 당하는 일이 많은 메서드부터 옮긴다. 하나씩 옮길 때마다 테스트
6. 양쪽 클래스의 인터페이스를 살펴보면서 불필요한 메서드를 제거하고, 이름도 새로운 환경에 맞게 바꾼다.
7. 새 클래스를 외부로 노출할지 정한다. 노출하려거든 새 클래스에 참조를 값으로 바꾸기를 적용할지 고민해본다.ㄴ

### 예시
```js
get name() {return this._name;}
set name(arg) {this._name = arg;}
get telephoneNumber() {return `(${this.officeAreaCode}) ${this.officeNumber}`;}
get officeAreaCode() {return this._officeAreaCode;}
set officeAreaCode(arg) {this._officeAreaCode = arg;}
get officeNumber() {return this._officeNumber;}
set officeNumber(arg) {this._officeNumber = arg;}
```
1. 전화번호 관련 동작을 별도 클래스로 뽑아보기
2. 먼저 빈 전화번호를 표현하는 TelephoneNumber클래스 정의
3. Person 클래스의 인스턴스를 생성할 때 전화번호 인스턴스도 함께 생성해 저장
```js
// Person
constructor() {
  this._telephoneNumber = new TelephoneNumber();
}

// TelephoneNumber
get officeAreaCode() {return this._officeAreaCode;}
set officeAreaCode(arg) {this._officeAreaCode = arg;}
get officeNumber() {return this._officeNumber;}
set officeNumber(arg) {this._officeNumber = arg;}

// Person
get officeAreaCode() {return this._telephoneNumber.officeAreaCode;}
set officeAreaCode(arg) {this._telephoneNumber.officeAreaCode = arg;}
...
```
6. 정리하기. 새로 만든 클래스는 순수한 전화번호를 뜻하므로 사무실이란 단어를 쓸 이유가 없다. 마찬가지로 전화번호라는 뜻도 메서드 이름에서 다시 강조할 이유가 없다.
- officeAreaCode -> areaCode
- officeNumber -> number

## 7.6 클래스 인라인하기
### 배경
**클래스 추출하기를 거꾸로 돌리는 리팩터링이다.** 필자는 더 이상 제 역할을 못 해서 그대로 두면 안 되는 클래스는 인라인해버린다. 역할을 옮기는 리팩터링을 하고나니 특정 클래스에 남은 역할이 거의 없을 때 이런 현상이 자주 생긴다.

두 클래스의 기능을 지금과 다르게 배분하고 싶을 때도 클래스를 인라인한다. **클래스를 인라인해서 하나로 합친 다음 새로운 클래스를 추출하는 게 쉬울 수도 있기 때문이다.**

### 절차
1. 소스 클래스의 각 public 메서드에 대응하는 메서드들을 타깃 클래스에 생성한다. 이 메서드들은 단순히 작업을 소스 클래스로 위임해야 한다.
2. 소스 클래스의 메서드를 사용하는 코드를 모두 타깃 클래스의 위임 메서드를 사용하도록 바꾼다. 하나씩 바꿀 때마다 테스트한다.
3. 소스 클래스의 메서드와 필드를 모두 타깃 클래스로 옮긴다. 하나씩 옮길 때마다 테스트한다.
4. 소스 클래스를 삭제하고 조의를 표한다.

```js
class TrackingInformation {
  get shippingCompany() {return this._shippingCompany;}
  set shippingCompany(arg) {this._shippingCompany = arg;}
  get trackingNumber() {return this._trackingNumber;}
  set trackingNumber(arg) {this._trackingNumber = arg;}
  get display() {
    return `${this.shippingCompany}: ${this.trackingNumber}`
  }
  //...
}

// Shipment 클래스
get trackingInfo() {return this._trackingInformation.display;}
get trackingInformation() {return this._trackingInformation;}
set trackingInformation(arg) {this._trackingInformation = arg;}
```

TrackingInformation이 제 역할을 하지 못 하고 있어 Shipment 클래스로 인라인한다.
```js
aShipment.trackingInformation.shippingCompany = request.vendor
```
1. 외부에서 직접 호출하는 TrackingInformation의 메서드들을 모조리 Shipment로 옮긴다. 보통 때의 함수 옮기기(8.1절)와는 약간 다르게 진행해본다. Shipment의 위임 함수를 만들고
2. 클라이언트가 이를 호출하도록 수정한다.
```js
// Shipment
set shippingCompany(arg) {this._trackingInformation.shippingCompany = arg;}

aShipment.shippingCompany = request.vendor;
```

클라이언트에서 사용하는 TrackingInformation의 모든 요소를 이런 식으로 처리한다.

3. TrackingInformation의 모든 요소를 Shipment로 옮긴다.

여기서는 이동할 목적지인 Shipment에서 shippingCompany()만 참조하므로 **필드 옮기기(8.2절)**의 절차를 모두 수행하지 않아도 된다. 그래서 참조하는 링크를 소스에 추가하는 단계는 생략한다.

4. 이 과정을 반복하고 다 옮겼다면 TrackingInformation 클래스를 삭제한다.
```js
// Shipment
get trackingInfo() {return `${this.shippingCompany}: ${this.trackingNumber}`;}
get shippingCompany() {return this._shippingCompany;}
set shippingCompany(arg) {this._shippingCompany = arg;}
get trackingNumber() {return this._trackingNumber;}
set trackingNumber(arg) {return this._trackingNumber = arg;}
```

## 7.7 위임 숨기기
### 배경
모듈화 설계를 제대로 하는 핵심은 캡슐화이다. 캡슐화는 모듈들이 시스템의 다른 부분에 대해 알아야 할 내용을 줄여준다. 캡슐화가 잘 되어 있다면 무언가를 변경해야 할 때 함께 고려해야 할 모듈 수가 적어져서 코드를 변경하기가 훨씬 쉬워진다.

서버 객체의 필드가 가리키는 객체(위임 객체)의 메서드를 호출하려면 클라이언트는 이 위임 객체를 알아야 한다. 위임 객체의 인터페이스가 바뀌면 이 인터페이스를 사용하는 모든 클라이언트가 코드를 수정해야 한다. 이러한 의존성을 없애려면 서버 자체에 위임 메서드를 만들어서 위임 객체의 존재를 숨기면 된다. 그러면 위임 객체가 수정되더라도 서버 코드만 고치면 되며, 클라이언트는 아무런 영향을 받지 않는다.

### 절차
1. 위임 객체의 각 메서드에 해당하는 위임 메서드를 서버에 생성한다.
2. 클라이언트가 위임 객체 대신 서버를 호출하도록 수정한다. 하나씩 바꿀 때마다 테스트한다.
3. 모두 수정했다면, 서버로부터 위임 객체를 얻는 접근자를 제거한다.
4. 테스트

### 예시
```js
// Person
constructor(name) {
  this._name = name;
}
get name() {return this._name;}
get department() {return this._department;}
set department(arg) {this._department = arg;}

// Department
get chargeCode() {return this._chargeCode;}
set chargeCode(arg) {this._chargeCode = arg;}
get manager() {return this._manager;}
set manager(arg) {this._manager = arg;}
```
클라이언트에서 어떤 사람이 속한 부서의 관리자를 알고 싶을 때, 부서 객체부터 얻어와야 한다.

```js
manager = aPerson.department.manager;
```
보다시피 클라이언트는 부서 클래스의 작동 방식, 부서 클래스가 관리자 정보를 제공한다는 사실을 알아야 한다.
1. 의존성을 줄이려면 클라이언트가 부서 클래스를 볼 수 없게 숨기고, 대신 사람 클래스에 간단한 위임 메서드를 만들면 된다.
```js
// Person
get manager() {this._department.manager};
```
2. 모든 클라이언트가 이 메서들를 사용하도록 고치기
3. department 접근자를 삭제하기

## 7.8 중개자 제거하기
### 배경
위임 숨기기(7.7)의 배경에서 객체를 캡슐화는 이점을 설명했다. but 그 이점이 거져 주어지는 건 아니다. 클라이언트가 위임 객체의 또 다른 기능을 사용하고 싶을 때마다 서버에 위임 메서드를 추가해야 하는데, 이렇게 기능을 추가하다 보면 단순히 전달만 하는 위임 메서드들이 점점 성가셔진다. 그러면 서버 클래스는 그저 **중개자 역할**로 전락하여, 차라리 클라이언트가 위임 객체를 직접 호출하는 게 나을 수도 있다.

어느 정도까지 숨겨야 적절한지를 판단하기란 쉽지 않지만, 우리에게는 다행히 위임 숨기기(7.7)와 중개자 제거하기 리팩터링이 있으니 크게 문제가 되지 않는다. 상황에 맞게 트레이드오프를 고려하면 된다.

### 절차
1. 위임 객체를 얻는 getter를 만든다.
2. 위임 메서드를 호출하는 클라이언트가 모두 이 getter를 거치도록 수정한다. 하나씩 바꿀 때마다 테스트
3. 모두 수정했다면 위임 메서드 삭제

### 예시
자신이 속한 부서 객체를 통해 관리자를 찾는 사람 클래스를 살펴보자.
```js
// client
manager = aPerson.manager;

// Person
get manager() {return this._department.manager;}

// Department
get manager() {return this._manager;}
```

사용하기 쉽고 부서는 캡슐화되어 있다. but 이런 위임 메서드가 많아지면 사람 클래스의 상당 부분이 그저 위임하는 데만 쓰일 것이다. 그럴 때는 중개자를 제거하는 편이 낫다.
1. 위임 객체(부서)를 얻는 getter를 만들자.
```js
// Person
get department() {return this._department;}
```
2. 각 클라이언트가 부서 객체를 직접 사용하도록 고친다.
```js
// client
manager = aPerson.department.manager;
```
3. Person의 manager 메서드 삭제. Person에 단순한 위임 메서드가 더는 남지 않을 때까지 이 작업 반복

위임 숨기기(7.7)나 중개자 제거하기를 적당히 섞어도 된다. 자주 쓰는 위임은 그대로 두는 편이 클라이언트 입장에서 편하다. 둘 중 하나를 반드시 해야 한다는 법은 없다. 상황에 맞게 처리하면 된다.

### 자동 리팩터링을 사용한다면
다른 방식도 생각해볼 수 있다. 먼저 부서 필드를 캡슐화한다. 그러면 관리자 getter에서 부서의 public getter를 사용할 수 있다.

```js
// Person
get manager() {return this.department.manager;}
```
js에서는 이 변화가 잘 드러나지 않지만, department 앞의 밑줄(_)을 빼면, 더 이상 필드를 직접 접근하지 않고 새로 만든 getter를 사용한다는 뜻이다. 그런 다음 manager() 메서드를 인라인하여 모든 호출자를 한 번에 교체한다.

## 7.9 알고리즘 교체하긴
### 배경
어떤 목적을 달성하는 방법은 여러 가지가 있게 마련이다. 그중에서도 다른 것보다 더 쉬운 방법이 분명히 존재한다. 알고리즘도 마찬가지다.

리팩터링하면 복잡한 대상을 단순한 단위로 나눌 수 있지만, 때로는 알고리즘 전체를 걷어내고 훨씬 간결한 알고리즘으로 바꿔야 할 때가 있다. 문제를 더 확실히 이해하고 훨씬 쉽게 해결하는 방법을 발견했을 때 이렇게 한다.

알고리즘을 살짝 다르게 동작하도록 바꾸고 싶을 때도 이 변화를 더 쉽게 가할 수 있는 알고리즘으로 통재로 바꾼 후에 처리하면 편할 수 있다.

이 작업에 착수하려면 반드시 메서드를 가능한 한 잘게 나눴는지 확인해야 한다. 거대하고 복잡한 알고리즘을 교체하기란 상당히 어려우니 알고리즘을 간소화하는 작업부터 해야 교체가 쉬워진다.

### 절차
1. 교체할 코드를 함수 하나에 모은다.
2. 이 함수만을 이용해 동작을 검증하는 테스트를 마련한다.
3. 대체할 알고리즘 준비
4. 정적 검사 수행
5. 기존 알고리즘과 새 알고리즘의 결과를 비교하는 테스트 수행. 두 결과가 같다면 리팩터링이 끝난다. 그렇지 않다면 기존 알고리즘을 참고해서 새 알고리즘을 테스트하고 디버깅한다.
