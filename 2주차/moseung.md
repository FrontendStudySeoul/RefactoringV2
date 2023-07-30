## 1. 6 계산 단계와 포맷팅 단계 분리하기

1.5장까지 리팩토링한 함수를 토대로 HTML 만드는 함수를 만들어야한다.<br>
하지만 해당 함수로 HTML 버전으로 그대로 복사해서 만들어도 되지만 필자는 두 버전 똑같은 함수를 사용해서 용도만 다르게 리팩토링하려고 했다.

### 단계 쪼개기
위와 같은 함수를 만들기 위해서 단계를 쪼갠다.<br>
우선 statement의 로직을 데이터를 처리하는 부분과 결과를 텍스트와 HTML로 표현하도록 분리한다. 첫번째 단계는 두번째 단계를 위한 데이터 생성의 과정이다.

```tsx
  function statement(invoice,plays){
    return renderPlainText(invoice,plays)
  }
  function renderPlainText(invoice,plays){
    ... data를 처리해서 Text나 html로 만들 로직(기존에는 여기에 데이터를 계산하는 로직도 들어가 있음)
  }
```

<br>
<div></div>

```tsx
  function statement(invoice,plays){
    //... 기존에 renderPlainText에 들어가있던 모든 계산로직을 statement함수에서 모두 계산후 데이터로 갖고있기.
  const statementData = {};
  statementData.~~~ = ~~~
    return renderPlainText(data,plays)
  }
  function renderPlainText(data,plays){
    ... statement에서 계산된 로직으로 오직 텍스트를 생성하거나 HTML을 그려주는 로직만 정리
  
  }
```
<div></div>
<br>
<div></div>
이제 statement에서 가지고 있던 데이터를 들고있는 store를 createStatmentData함수로 분리하고 내가 원하는 형식으로 기능을 추가한다.(책에서 말하는 text or HTML)
<br>
```tsx
function statement(invoice,plays){
    return renderPlainHTML(createStatementData(invoice,plays))
  }
function createStatementData(invoice,plays){
return data
// 이떄 data는 invoice와 plays를 토대로 내가 원하는 데이터로 가공한값
}
function renderPlainHTML(data){
//HTML을 render하는 로직들
let dom;
dom+= `<tag>고객명 : ${data.customer}</h1>`
...
}
```
위처럼 statement를 분리하면 text로 만들때도 render하는 함수만 바꿔주면 된다.
```tsx
function statement(invoice,plays){
    return renderPlainText(createStatementData(invoice,plays))
  }
```

### 반복문을 파이프라인으로 바꾸기
기존에 for문을 사용하는걸 좀 더 선언적으로 [자바스크립트 고차함수](https://inpa.tistory.com/entry/JS-%F0%9F%93%9A-%EB%B0%B0%EC%97%B4-%EA%B3%A0%EC%B0%A8%ED%95%A8%EC%88%98-%EC%B4%9D%EC%A0%95%EB%A6%AC-%F0%9F%92%AF-mapfilterfindreducesortsomeevery)를 이용해서 사용한다.

```tsx
// 이 방법으로
const sum = array.reduce((prev,current)=>return prev+current)

// 이 방법은 X
  function getSum(array){
    let sum = 0;
    for(let i =0;i++;i<array.length-1){
      sum+= array[i]
    }
    return sum
  }
```
<br>
<img width="902" alt="image" src="https://github.com/FrontendStudySeoul/RefactoringV2/assets/103626175/b0aac2ad-e6be-4050-a8d3-5fc329adfc80">

<br>
<img width="904" alt="image" src="https://github.com/FrontendStudySeoul/RefactoringV2/assets/103626175/14366bd6-1c4a-4e80-9eb6-f4e78f16085b">

