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

