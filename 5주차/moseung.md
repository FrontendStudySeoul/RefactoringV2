# API 리팩터링 
좋은 API는 데이터를 갱신하는 함수와 그저 조회만 하는 함수를 명확히 구분한다. 
## 11-1 질의 함수와 변경 함수 분리하기
우리는 외부에서 관찰할 수 이쓴 겉보기 부수효과가 전혀 없이 값을 반환해주는 함수를 추구해야 한다. 이를 위한 한가지 방법은 '질의 함수는 모두 부수효과가 없어야 한다'는 규칙을 따른다.<br>
```tsx
 function alertForMiscreant(people){
    for (const p of people){
      if(p==="조커"){
        setoffAlarms()
        return '조커';
      }
      if(p==="사루만"){
        setoffAlarms()
        return '사루만';
      }
      return ""
    }
  }
```
위와 같은 함수에서 한 함수에서 악당을 리턴하고 alarm을 끄는 두가지의 역할을 하고 있으므로 분리해줘야 할 필요가 있다.
```tsx
const foundMiscreant = findMisCreant(people);
alertForMiscreant(people)

function findMiscreant(people){
    for (const p of people){
      if(p==="조커"){
        return '조커';
      }
      if(p==="사루만"){
        return '사루만';
      }
      return ""
    }
  }

function alertForMiscreant(people){
    for (const p of people){
      if(p==="조커"){
        setoffAlarms()
        return;
      }
      if(p==="사루만"){
        setoffAlarms()
        return;
      }
      return;
    }
  }
```
이처럼 악당을 발견하면 악당이라는 string을 return 해주는 함수와 alert해주는 함수 두개로 분리됐다. 근데 위에도 people을 돌면서 반복문을 두번 실행하기에 나라면 한번 더 줄일것 같다.

```tsx
const foundMiscreant = findMisCreant(people);
if(foundMiscreant!=="") setoffAlarms()
```

## 11.2 함수 매개변수화하기
두 함수의 로직이 아주 비슷하고 단지 리터럴 값만 다르다면, 그 다른 값만 매개변수로 받아 처리하는 함수 하나로 합쳐서 중복을 없앨 수 있다.
```tsx
function baseCharge(usage){
  if(usage<0)return usd(0);
  const amount =bottomBand(usage)*0.03*middleBand(usage)*0.05+topBand(usage)*0.07;
  return usd(amount);
 }

 function bottomBand (usage){
  return Math.min(usage,100);
 }

 function middleBand (usage){
  return usage>100?Math.min(usage,200)-100:0
 }

 function topBand (usage){
  return usage>200?usage-200:0
 }
```
위 코드를 자세히보면 각각 다른 계산을 하고 있긴 하지만 Math.min이라는 함수와 usage라는 인자를 받고있다는점, 100,200이라는 매직넘버를 이용해서 계산하는 로직을 하고 있는점이 비슷하다.
<br> 그래서 위 함수 세개를 하나의 함수로 합친다.
```tsx
function withinBand(usage,bottom,top){
return usage>bottom?Math.min(usage,top) - bottom:0
}
```
위 세개의 함수를 하나로 합치면서 함수가 좀 더 명확해졌다. withinBand라는 함수는 usage와 bottom값 top값을 받아서 그 값이 usage보다 크다면 그 값을 bottom으로 뺀 값을 리턴하고 아니라면 0을 리턴하는 함수로 바뀌었다.
### 나의 생각
위에서는 단순 삼항연산자로 바꿀 수 있기에 괜찮았지만 만약에 여러개의 함수를 합쳤는데 오히려 여러개의 if문이나 중첩삼항연산자로 코드가 만들어진다면 나라면 그대로 코드를 둘 것 같다.

## 11.3 플래그 인수 제거하기
플래그 인수란 호출되는 함수가 실행할 로직을 호출하는 쪽에서 선택하기 위해 전달하는 인수이다.<br>
이런 플래그 인수가 존재하면 함수를 읽는 입장에서 오히려 가독성이 떨어지기 때문에 해당 플래그 인수를 기점으로 조건문이 나눠지는 하나의 함수보다는 함수의 의도를 드러내는 이름과 함께 조건문을 분리해주는것이 좋다.
```tsx
function deliveryDate(anOrder:Order,isRush:boolean){
if(isRush){
//isRush 값이 true일떄 실행되는 로직 anOrder를 이용해 값을 만듬
}else{
//isRush 값이 false일떄 실행되는 로직 anOrder를 이용해 값을 만듬
}
 }
```
위 함수는 isRush가 무엇인지 이해하는데 1차 이해가 필요하고 isRush가 true인건 뭔지 isRush가 false인건 뭔지에 대한 이해가 2차적으로 필요하다. 근데 가령 isRush가 회사에서 사용하는 특별한 값인 쀾꾺깕탉삵이라는 단어라면 이를 이해하는데 더 어려움을 겪을것이다.<br>
그런데 이러한 상황에서 기획에서 쀾꾺깕탉삵이라는 값을 어떤 특정값으로 바꿔달라고 했을떄 이를 고치는 개발자는 쀾꾺깕탉삵이라는 값을 바꾸기 위해 어떻게 접근해야하는지 더 어려울것이다.
<br>
그래서 이를 앞서 10장에서 적용한 조건문 분해하기를 이용하여 분리한다.
```tsx
function rushDeliveryDate(anOrder){
//로직
}
function regularDeliveryDate(anOrder){
//로직
}
```
이렇게 두개의 조건문을 두개의 함수로 분리하면서 더 명확하게 함수의 의도를 알게 됐다.

## 11.4 객체 통째로 넘기기
하나의 레코드에서 값 두어 개를 가져와 인수로 넘기는 코드를 보면, 필자는 레코드를 통쨰로 넘긴다고한다.
<br>
레코드를 통쨰로 넘기면 변화에 대응하기 쉽다 왜냐하면 어떤 함수가 어떤 레코드에 있는 더 많은 데이터를 쓰게 됐을때 코드 수정없이 사용이 가능하기 떄문이다. 하지만 특정 레코드를 넘기긴 하나 레코드 자체에 의존하지 않고 싶을때엔 그렇게 리팩토링 하지 않는다.
<br>
```tsx
interface ButtonPropsType
  extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  property?: 'solid' | 'primary' | 'default' | 'danger'
  small?: boolean
  active?: boolean
  normal?: boolean
  disabled?: boolean
  loading?: boolean
  type?: 'button' | 'submit' | 'reset' | undefined
  children?: React.ReactElement | string
}
```
자주 사용하는 컴포넌트를 만들때 그 컴포넌트가 자바스크립트 근본에 비롯된 html들로 만드는 경우가 있다.(button,input)
이 때 위처럼 나는 해당 html태그의 interface를 상속받아서 값을 사용하는데 이렇게 하면 기존 button안에 닮겨있는 모든 property를 사용 가능하다.

## 11.5 매개변수를 질의 함수로 넘기기
매개변수 목록은 함수의 변동 요인을 모아놓은 곳이다. 이떄 매개변수를 다시 질의함수로 바꿔주는것이 필요할 떄가 있다.
```tsx
get finalPrice(){
  const basePrice = this.quantity * this.itemPrice;
  let discountLevel ;
  if(this.quantity>100)discountLevel=2
  discountLevel=1
  return this.discountedPrice(basePrice,discountLevel)
}
```
discountLevel에 따라 discountedPrice를 최종값으로 리턴하는 함수가 있는데 여기서 discountLevel을 분리할만하다.
```tsx
get discountLevel(){
return this.quantity>100?2:1;
}
get finalPrice(){
  const basePrice = this.quantity * this.itemPrice;
  return this.discountedPrice(basePrice)
}
```

## 11.6 질의 함수를 매개변수로 바꾸기
만약 질의 함수가 전역 변수를 사용한다거나 값이 거북한 상황에서는 질의 함수를 다시 매개변수로 바꿔준다.

## 11.7 세터 제거하기
클래스내에 세터함수가 있다는것은 필드값이 바뀐다는 징조이다.하지만 세터가 있으면 안되는 고유한 id값같은 경우에는 세터로 값을 설정해주는것이 아니라 생성자로 값을 만들어준다.
```tsx

get name() return this._name
set name(arg) this._name = arg
get id() return this._id
set id(arg) this._id = arg

const person = new Person();
person.name = "모승"
person.id = getId("모승")
```
위와같이 id는 다시 바껴선 안되는 값이기에 set이 아니라 생성자로 값을 정한다.

```tsx
constructor(id){
this._id = id
}

get name() return this._name
set name(arg) this._name = arg
get id() return this._id
set id(arg) this._id = arg
```

## 11.8 생성자를 팩터리 함수로 바꾸기
생성자로 클래스를 만드는것을 함수로 한번더 감싸주는것이 좋다.
```tsx
const leadEngineer = new Employee(document.leadEngineer,"E")

/// 리팩토링

function createEngineer(name){
return new Employee(name,"E")
}
```
이처럼 리팩토링 하면 추후에 Enginner를 만드는 내부 로직이 바뀌더라도 혹은 다른 함수로 대체되더라도 변경하기에 용이하다.<br>
### 내 생각
위와 유사한 방법으로 전역관리 라이브러리를 다룰때 유용할 것 같다.예를들어 어떤 프로젝트에서 전역관리툴을 Redux로 사용하고 실제로 코드 불러오는 부분도 Redux를 불러와서 사용했다. 하지만 이 부분을 팩터리 함수로 바꿔준다면 추후에 Redux가 아닌 Recoil이나 Zustand같은 다른 상태관리 라이브러리를 사용하더라도 **팩터리 함수 내에 코드만 바꿔주면** 실행부에는 전혀 지장이 없을것이다

## 11.9 함수를 명령으로 바꾸기
함수는 프로그래밍의 기본적인 블록 중 하나이다. 그런데 사용하는 함수를 그 함수만을 위한 객체 안으로 캡슐화하면 유용해지는 경우가 있고 그 객체를 **명령** 이라고 한다. 이렇게 사용하는 함수가 굉장히 복잡한 경우라면 명령으로 바꿔주는것이 용이하다.
```tsx

  function score(candidate, medicalExam, scoringGuide){
    let result = 0
    let healthLevel = 0
    let highMedicalRiskFlag = false;
    
   if(healthLevel){
//로직
}
  if(highMedicalRiskFlag){
//로직
}
    return result
  }
```
이제 이 코드를 명령 객체로 바꾼다.
```tsx
 function score(candidate, medicalExam, scoringGuide){
    return new Scorer(candidate, medicalExam, scoringGuide).execute()
  }

class Scorer {

constructor(candidate, medicalExam, scoringGuide){
this.candidate =_candidate;
this.medicalExam =_medicalExam;
  this.scoringGuide =_scoringGuide;
}

execute(){
this._result=0
this.getHealthLevel()
this.getHighMedicalRiskFlag()
return this._result
}

getHealthLevel(){
//로직
}

getHighMedicalRiskFlag(){
//로직
}
}
```

## 11.10 명령을 질의 함수로 바꾸기
9장이라는 다르게 명령이 있고 이 함수가 복잡하지 않다면 다시 질의 함수로 바꿔주는것이 용이하다.

## 11.11 수정된 값으로 반환하기
데이터가 어떻게 수정되는지를 추적하는 일은 코드에서 이해하기 가정 어려운 부분 중 하나이다. 그래서 데이터가 수정된다면 그 사실을 명확히 알려주어서 어느 함수가 무슨 일을 하는지 쉽게 하는일이 중요하다.
<br>
그래서 가장 좋은 방법은 변수를 갱신하는 함수라면 수정된 값을 반환하여 호출자가 그 값을 변수로 갖고있는 것이다.
```tsx
 let totalAscent = 0;
  let totalTime = 0;
  calculateTotalAscent()
  calculateTotalTime()

  function calculateTotalAscent(){
    for(let i = 0;i<points.length<i++){
      totalAscent+= a>0?1:0
    }
  }
```
위와 같은 함수가 있다
```tsx
 const totalAscent = calculateTotalAscent();
  let totalTime = 0;
  calculateTotalTime()

  function calculateTotalAscent(){
let totalAscent;
    for(let i = 0;i<points.length<i++){
      totalTime+= a>0?1:0
    }
return totalAscent
  }
```

totalAscent를 수정된 값으로 반환함으로써 데이터가 어디서 처리되는지 해당 함수에서 명확히 확인이 가능해졌다.

## 11.12 오류 코드를 예외로 바꾸기
만약 오류 코드를 사용했을떄 복잡한 상황을 예외를 사용하면 그런일이 없게 해준다.
```tsx
function localShippingRules(country){
  const data = countryData.shippingRules[country]
  if(data) return new ShippingRules(data)
  else return -23
 }
```
이제 위와 같이 -23을 리턴하는것이 아니라 예외상황을 던져준다.
```tsx
class ErrorBoundary(){
//에러 핸들링 로직
}

function localShippingRules(country){
  const data = countryData.shippingRules[country]
  if(data) return new ShippingRules(data)
  else return new ErrorBoundary(-23)
 }

```

## 11.13 예외를 사전확인으로 바꾸기
예외를 던지는 경우를 사전에 있는 함수로 체크가 가능할경우 사전확인후 값을 컨트롤한다.
