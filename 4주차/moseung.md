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
