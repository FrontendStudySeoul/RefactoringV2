
## 1. Introduce

코드를 수정할 일이 있고 그 코드가 수백줄이 될거라면 코드를 여러 함수와 프로그램으로 재구성한다. 그리고 만약 기존 코드가 있고 그 코드에 새로운 기능을 추가하기 어려운 구조라면 기존 기능을 리팩토링하고 새로 기능을 추가한다.
<br>
그리고 코드는 잘 작동하고 추후 변경이 없을거라면 그대로 둬도 상관없지만 그것이 아닐때 코드를 리팩토링하거나 새로 만들때 고려해서 구조를 잡고 코딩해야한다.

## 1 . 3 리팩터링의 첫 단계

코드 예시<br>
아래는 극단이 공연할 연극 정보 JSON<br>
```tsx
// play.json

{
  "hamlet": { "name": "Hamlet", "type": "tragedy" },
  "as-like": { "name": "As You Like It", "type": "comedy" },
  "othello": { "name": "Othello", "type": "tragedy" }
}
```
```tsx
// invoices.json

[
  {
    "customer": "BigCo",
    "performances": [
      {
        "playID": "hamlet", //공연 ID
        "audience": 55 //관객수
      },
      {
        "playID": "as-like",
        "audience": 35
      },
      {
        "playID": "othello",
        "audience": 40
      }
    ]
  }
]
```
공연료 청구서를 출력하는 코드
```tsx
const statement = (invoice, plays) => {
  let totalAmount = 0 //총액
  let volumeCredits = 0 //포인트
  let result = `청구 내역 (고객명: ${invoice.customer})\n`

  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2,
  }).format

  for (let perf of invoice.performances) {
    const play = plays[perf.playID]
    let thisAmount = 0 //이 공연 금액

    switch (play.type) {
      case 'tragedy': {
        thisAmount = 40000
        if (perf.audience > 30) thisAmount += 1000 * (perf.audience - 30)
        break
      }
      case 'comedy': {
        thisAmount = 30000
        if (perf.audience > 20) thisAmount += 10000 + 500 * (perf.audience - 20)
        thisAmount += 300 * perf.audience
        break
      }
      default:
        throw new Error(`알 수 없는 장르: ${play.type}`)
    }

    volumeCredits += Math.max(perf.audience - 30, 0)
    if (play.type === 'comedy') volumeCredits += Math.floor(perf.audience / 5)
    result += `  ${play.name}: ${format(thisAmount / 100)} (${perf.audience}석)\n`
    totalAmount += thisAmount
  }
  result += `총액: ${format(totalAmount / 100)}\n`
  result += `적립 포인트: ${volumeCredits}점\n`
  return result
}

export default statement
```

리팩터링의 첫 단계는 테스트코드를 작성하는것이다. 위의코드에서는 예상 정답들에 대한 반환값을 만들어두고 해당 반환값을 테스트라이브러리에 넣은 후 리팩터링 할떄 수시로 테스트 해본다.<br>
즉 리팩터링할떄는 제대로된 테스트부터 만들고 테스트는 반드시 자가진단하도록 만든다.

## 1 . 4 statement() 함수 쪼개기

switch 문부터 쪼갠다. switch문은 결국 thisAmount를 구하기때문에 묶어서 가져간다. 함수로 분리하기 위해 적절한 이름인 amountFor이라는 함수로 분리한다.
```tsx
const amountFor = (perf,play) =>{
 let thisAmount = 0 //이 공연 금액

    switch (play.type) {
      case 'tragedy': {
        thisAmount = 40000
        if (perf.audience > 30) thisAmount += 1000 * (perf.audience - 30)
        break
      }
      case 'comedy': {
        thisAmount = 30000
        if (perf.audience > 20) thisAmount += 10000 + 500 * (perf.audience - 20)
        thisAmount += 300 * perf.audience
        break
      }
      default:
        throw new Error(`알 수 없는 장르: ${play.type}`)
return thisAmout
    }
```

이제 위 함
### 궁금한점
위 amountFor이라는 함수에서 play에서는 type만 쓰이고 pefr에서는 audience만 쓰이는데 이러면 나는 좀 더 쪼개서 함수를 만들것 같다. 그리고 추후에 perf나 play의 다른 property들로 값을 더해야한다면 함수를 하나 더 만든다. 왜 그렇게 생각했냐면 좀 더 함수의 인자를 쪼개야 해당 함수가 뭘 하는지 좀 더 직관적일거같다. <br>
그리고 나는 항삼 함수를 추상화할때 ``const thisAmount = getThisAmount()``처럼 해당 변수의 값을 get하는 형식으로 이름을 만들었는데 해당 코드내에서만 쓰이면 괜찮겠지만 이를 유틸함수나 커스텀훅으로 분리할때는 어떻게 이름을 가져야할지?

### 커밋하기
위의 과정으로 하나의 기능을 수정했으면 테스트까지 진행후 커밋한다.



**리팩토링은 프로그램을 작은 단계로 나눠서 진행한다. 너무 큰 단위로 가져가게되면 디버깅하는데 시간이 오래 걸릴 수 있기때문에 최대한 작은 단계로 쪼개서 수시로 테스트를 확인하면서 진행한다.**
