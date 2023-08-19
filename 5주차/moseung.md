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
레코드를 통쨰로 넘기면 변화에 대응하기 쉽다 왜냐하면 어떤 함수가 어떤 레코드에 있는 더 많은 데이터를 쓰게 됐을때 코드 수정없이 사용이 가능하기 떄문이다.
<br>
