## 8-1 함수 옮기기
좋은 소프트웨어 설계의 핵심은 모듈화가 얼마나 잘되어 있느냐를 뜻하는 모듈성이다. 어떤 함수가 속한 A의 모듈보다 B의 모듈의 의존성을 더 사용한다면 해당 함수는 모듈 B로 옮겨줘야한다.
```jsx
//module
  function trachSummary(points){
    ...
    function calculateDistance(){
      let result = 0;
      for(let i=0;i<points.length;i++){
        result += distance(points[i-1],points[i])
      }
      return result
    }
    function distance(p1,p2){

    }
    function radians(degree){

    }
  }
```
위의 모듈안에 있는 calculateDistance를 상위로 빼서 다른 정보와는 독립적으로 계산하고싶다는 니즈가 있다.<br>
이때 가장 먼저 할일은 이 함수를 모듈 밖으로 뺀다. 이름은 일단 임시로 아무렇게 정한다.
```jsx
function top_calculateDistance(points){
      let result = 0;
      for(let i=0;i<points.length;i++){
        result += distance(points[i-1],points[i])
      }
      return result
    }
```
이제 내부 함수를 확인해본다.
```jsx
function distance(p1,p2){
      const EARTH_RADIUS = 3959;
      const dlat = radians(p1,p2)
      const dlon = radians(p1,p2)
      // dlat과 dlon을 이용한 순수 계산
      const c = 순수 계산
      return EARTH_RADLUS *c;
    }
    function radians(degree){
return degree* Math.PI/180
    }
```
distance 함수를 확인해보면 기존 모듈안에 있던 radains라는 값만 사용하고 있으므로 이 두 함수를 같이 뺸다.
```jsx
function top_calculateDistance(points){
      let result = 0;
      for(let i=0;i<points.length;i++){
        result += distance(points[i-1],points[i])
      }
      return result
function distance(p1,p2){
      const EARTH_RADIUS = 3959;
      const dlat = radians(p1,p2)
      const dlon = radians(p1,p2)
      // dlat과 dlon을 이용한 순수 계산
      const c = 순수 계산
      return EARTH_RADLUS *c;
    }
    function radians(degree){
return degree* Math.PI/180
    }
    }
```
그리고 임시로 지은 함수명을 좀 더 의미있는 이름인 totalDistance라고 불러주고 기존에 있던 모듈에서 뺀 함수를 totalDistance로 대체해준다.


## 8-2 필드 옮기기
어떤 함수에 클래스를 넘기고 있는데 그때 다른 클래스의 필드도 같이 넘기고 있다면 이때 필드의 위치를 옮겨야한다.

```jsx
class Customer {
    constructor(name,discountRate){
      this._name = name;
      this._discountRate = discountRate;
      this._contract = new CustomerContract(dateToday())
    }

    get discountRate(){return this.discountRate_}
    becomePreferred(){
      this._discountRate +=0.03;
    }
  }

class CustomerContract {
constructor(startDate){
this._startDate=startDate;
}
}

```
위에서 discountRate를 CustomerContract 옮기고 싶고 이때는 [캡슐화](https://www.codestates.com/blog/content/%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%ED%8A%B9%EC%A7%95)를 사용한다.
```jsx
class Customer {
    constructor(name,discountRate){
      this._name = name;
      this._discountRate = discountRate;
      this._contract = new CustomerContract(dateToday())
    }

    get discountRate(){return this.discountRate_}
    seDiscountRate(arg){this._discountRate = arg}
    becomePreferred(){
      this.seDiscountRate(this._discountRate +=0.03);
    }
  }

```
그 이후 이 값들을 CustomerContract로 옮겨준다.

```jsx
class CustomerContract {
constructor(startDate,discountRate){
this._startDate=startDate;
this._discountRate = discountRate;
}

    get discountRate(){return this.discountRate_}
    set discountRate(arg){this._discountRate = arg}
}
```
그 이후 원래 있던 클래스에서 값을 수정해준다.
```jsx
class Customer {
    constructor(name,discountRate){
      this._name = name;
      this._contract = new CustomerContract(dateToday())
      this._setDiscountRate(discountRate)
    }

    get discountRate(){return this._contract.discountRate_}
    _setDiscountRate(arg){this._contract._discountRate = arg}
    becomePreferred(){
      this.seDiscountRate(this._discountRate +=0.03);
    }
  }
```
## 8-3 문장을 함수로 옮기기
중복 제거는 코드를 건강하게 관리하는 가장 효과적인 방법 중 하나이다. 왜냐하면 이렇게 중복을 제거해주면 추후에 코드를 유지보수 할때 한군데서만 수정하면 되고 추후에 코드의 동작을 변형해야하는 순간이 오면 다시 리팩토링 할 수 있다.
<br>
아래와 같은 승모의 값을 얻어내는 두개의 함수가 있다.
```jsx
const fucntionA = ()=>{
    ...
    const getSeungmo = `키:${seungmo.height},몸무게:${seungmo.weight}`
  }

  const fucntionB = ()=>{
    ...
    const getSeungmo = `키:${seungmo.height},몸무게:${seungmo.weight}`
  }
```

이를 함수로 다시 만들어주고 기존 코드에 반영해준다.
```jsx
  const getSeungmo = (seungmo)=>{
return `키:${seungmo.height},몸무게:${seungmo.weight}`
}

const fucntionA = ()=>{
    ...
    const seungmo = getSeungmo(seungmo);
  }

  const fucntionB = ()=>{
    ...
    const seungmo = getSeungmo(seungmo);
  }
```

## 8-4 문장을 호출한 곳으로 옮기기
기존에 사용하던 기능이 일부 호출에서는 다르게 동작하도록 바뀌어야하면 우리는 다시 코드를 작성해야한다. 이떄 개발자는 달라진 동작을 함수에서 꺼내 해당 호출자로 옮긴다.
```jsx
 function renderPerson(outstream,person){
    ...
    emitPhotoData(outstream,person.phto)

    
  }

  function listRecentPhotos(outstream,photos){
    photos.filter().forEach(p=>{
      ...
      emitPhotoData(outstream,p)
    })
  }

   function emitPhotoData (outstream,photo)
{ 
  outstream.write(`${photo,title}`)
  outstream.write(`${photo,name}`)
  outstream.write(`${photo,date}`)
}  
```
위 코드에서 renderPerson에서만 outstream.write(`${photo,date}`)를 하지 않아야하는 상황이 생긴다.
```jsx
  function emitPhotoData (outstream,photo)
{ 
  zzstream(outstream,photo)
  outstream.write(`${photo,date}`)
}

fucntion zzstream(outstream,photo){
outstream.write(`${photo,title}`)
  outstream.write(`${photo,name}`)
}
```
위처럼 중복되는 부분만 따로 함수로 뺴준다.
```jsx
function renderPerson(outstream,person){
    ...
    zzstream(outstream,photo)
  }

function listRecentPhotos(outstream,photos){
    photos.filter().forEach(p=>{
      ...
      zzstream(outstream,photo)
      outstream.write(`${photo,date}`)
    })
  }
```
그 이후 기존 함수에서 원래 하려던 작업을 진행한다.

## 8-5 인라인 코드를 함수 호출로 바꾸기
함수의 이름이 코드의 동작 방식보다는 목적을 말해주기 때문에 함수를 활용하면 코드를 이해하기 쉬워진다. 그래서 동작을 똑같이 하는 인라인 코드가 있다면 라이브러리나 언어에서 제공하는 함수를 사용한다.
```tsx
let isBoolean = false
for(const s of state){
if(s==="MA") isBooelan = true
}
///////////
isBoolean = state.includes("MA)
```

## 8-6 문장 슬라이드하기 
관련된 코드들이 가까이 모여 있다면 이해하기가 더 쉽다. 예를들어 하나의 데이터 구조를 이용하는 문장들은 한데 모여 있어야 좋다. 가장 흔한 사례는 변수를 선언하고 사용할때이며 모든 변수선언을 함수 첫머리에 모아두는 사람도 있지만 이 책의 필자는 변수를 처음 사용할떄 선언하는 스타일을 선호한다고한다. 왜냐하면 관련 코드끼리 모으는 작업이 다른 리팩터링의 준비 단계로 자주 행해지기 떄문이다.
<br>
프론트엔드에서 흔히 컴포넌트 상단에 hook을 몰아넣고 그 뒤에 함수를 몰아넣는 방식의 코드 스타일이 꽤 많다.
```tsx
  const navigate = useNavigate()
  const [isOpenCalendar, setIsOpenCalendar] = useState(false)
  const [keyword, setKeyword] = useState('')
  const [isClickedInput, setIsClickedInput] = useState(false)
  const setSearchedKeyword = useSetRecoilState(searchedKeyword)

  useDebounce(
    () => {
      if (keyword.length > 0) {
        navigate('/myrecord/search')
        setSearchedKeyword({ keyword })
      }
    },
    500,
    [keyword]
  )

  함수들...

return (
html
)
```
물론 이런 방식이 마치 Class형태처럼 상태, 함수, 렌더링 부분으로 나눠서 읽기 쉽다고 생각할 수 있지만 많약에 상태값들이 많아지거나 사이에 함수들이 많아지거나 한다면 오히려 가독성을 해칠때가 있다.
그래서 나는 위에서 알려준 방식처럼 변수 혹은 상태, 함수로 문장슬라이드를 자주 한다.
```tsx
const [minutes, setMinutes] = useState(mm);
  const [seconds, setSeconds] = useState(ss);
  const [timer, setTimer] = useState('');
  //다른 상태들

  //다른 이펙트들
  useEffect(() => {
    const countdown = setInterval(() => {
      if (seconds > 0) {
        setSeconds(seconds - 1);
      }
      if (seconds === 0) {
        if (minutes === 0) {
          clearInterval(countdown);
        } else {
          setMinutes(minutes - 1);
          setSeconds(59);
        }
      }
    }, 1000);
    setTimer(`${minutes}:${seconds < 10 ? `0${seconds}` : seconds}`);
    return () => clearInterval(countdown);
  }, [minutes, seconds]);
//다른 함수들
```
원래는 위처럼 여러 상태와 함수가 혼용돼있어서 몰랐지만 용도가 맞는 코드들끼리 묶음으로써 데이터의 흐름대로 볼 수 있게됐고 마침 해당 로직을 다른 페이지에서도 필요했기에 useTimer라는 hook으로 리팩토링 할 수 있었다.
```tsx
import { useState, useEffect } from 'react';

interface TimerProps {
  mm: number;
  ss: number;
}

const useTimer = ({ mm, ss }: TimerProps) => {
  const [minutes, setMinutes] = useState(mm);
  const [seconds, setSeconds] = useState(ss);
  const [timer, setTimer] = useState('');

  useEffect(() => {
    const countdown = setInterval(() => {
      if (seconds > 0) {
        setSeconds(seconds - 1);
      }
      if (seconds === 0) {
        if (minutes === 0) {
          clearInterval(countdown);
        } else {
          setMinutes(minutes - 1);
          setSeconds(59);
        }
      }
    }, 1000);
    setTimer(`${minutes}:${seconds < 10 ? `0${seconds}` : seconds}`);
    return () => clearInterval(countdown);
  }, [minutes, seconds]);

  return timer;
};

export default useTimer;

```

## 8.7 반복문 쪼개기
종종 반복문 하나에서 두 가지 일을 수행하는 모습을 보게 된다.하지만 이렇게 하면 반복문을 수정해야 할 때마다 두 가지 일 모두를 잘 이해하고 진행해야 한다. 그래서 서로 연관없는 값이 한 반복문 내에서 다뤄지고 있다면 반복문을 분리해야 한다.
```tsx
  let youngest = people[0]?people[0].age:Infinity;
  let totalSalary = 0;
  for (const p of people){
    if(p.age<youngest) youngest = p.age
    totalSalary+=p.salary
  }

  return `최연소 :${youngest} 총 급여 :  ${totalSalary}`
```
위 반복무에서 totalSalary와 youngest는 서로 연관이 없는 변수기 떄문에 함수화 해준다. 그리고 함수로 바꿔줄 수 있는 부분을 바꿔준다.

```tsx
const youngest = getYoungest(people)
  const totalSalary = getTotalSalary(people)

  const getYoungest  = (people)=>{
    let youngest = people[0]?people[0].age:Infinity;
    for (const p of people){
      if(p.age<youngest) youngest = p.age
    }
    return youngest
  }

  const getTotalSalary  = (people)=>{
    for (const p of people){
      totalSalary+=p.salary
    }
    return youngest.reduce((total,p)=>total+p.salary,0)
  }
```

## 8.8 반복문을 파이프라인으로 바꾸기
반복문을 사용하는 코드들을 연산을 이용해서 컬렉션을 다시 연산을 사용한 코드로 작성하면 이해하기 더 쉽다.
```tsx
 function acquireData(input){
    const lines = input.split("\n")
    let firstLine =true;
    const result = [];
    for (const line of lines){
      if(firstLine){
        firstLine=false;
        continue
      }
      if(line.trim()==="")continue;
      const record = line.split(",")
      if(record[1].trim() ==="india"){
        result.push({city:fields[0].trim(),phone: fields[2].trim()})
      }
    }
  }

```
어떤 값을 string으로 바꿔서 인도에 자리한 사무실을 찾아 도시명과 전화번호를 반환하는 반복문이 있다.
  ```tsx
function acquireData(input){
    const lines = input.split("\n")
    return lines
            .slice(1)
            .filter(line => line.trim( !== ""))
            .map(line => line.split(","))
            .filter(fields => fields[1].trim()==="india")
            .map(fields => ({city:fields[0].trim(),phone: fields[2].trim()}))
  }
```

## 8.9 죽은 코드 제거하기
항상 쓰지 않는 코드는 제거해야된다. 왜냐하면 우리에게는 버전 관리 시스템인 git이 있기때문에 다시 해당 상태로 돌리면 되기 때문이다.<br>
그리고 한때 쓰지 않는 코드는 주석으로 처리하는게 유행했다고 한다.
### 쓰지 않는 코드는 그냥 지우기
위에서 설명한것처럼 쓰지 않는 코드를 주석으로 처리하는것이 좋지 않다고 생각한다.
<br>
왜냐하면 위에서 설명한것처럼 지웠다가 다시 git으로 되살릴 수 있기 때문이기도 하지만 사실 주석으로 처리하면 그 맥락을 주석처리한 개발자 밖에 알 수 없다.
<br>
회사에서도 일어난 일인데 어떤 코드들이 주석돼있었고 그 주석 위에 주석과 상당히 유사한 코드가 남아있었다. 나는 그 주석에 설명이 없었기 때문에 어떤 의도로 만든 주석인지 알 수 없었고 수정하기에 어려움을 느꼈다.
<br>
책에서 있는것처럼 주석조차도 안하는게 좋고 하더라도 그 주석을 단 이유라도 적어두는것이 협업을 위해 필수라고 생각한다.
