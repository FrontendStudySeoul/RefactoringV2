## Chapter 01 - 리팩터링: 첫 번째 예시

> 프로그램이 새로운 기능을 추가하기에 편한 구조가 아니라면,  
> 먼저 기능을 추가하기 쉬운 형태로 리팩터링하고 나서 원하는 기능을 추가한다.


**`테스트 코드`**

- 실제 작업은 사람이 하기 때문에 언제든 실수할 수 있다.


<br />

특정 코드가 어떤 역할을 하는지 코드 분석을 통해 알아야 하는 경우 잊어버리기 쉽다. 
잊기 전에 코드에 반영해야 한다.

<br />

**`함수 추출하기`**


- 더 명확하게 표현할 수 있는 방법이 있을 지 고민하기
- 변수명
  - `thisAmount` -> `result`
  - `a`/`an` 관사

**`임시 변수를 질의 함수로 바꾸기`**

- `play`는 이미 매개변수로 전달된 `aPerformance`에서 얻는 정보
  - 애초에 매개변수로 전달할 필요가 없다.
- 대입문의 우변을 함수 추출
  ```ts
  for(let perf of invoice.performances) {
    const play = plays[perf.playID];
    let thisAmount = amountFor(perf, play);
  
  ...
  
  for(let perf of invoice.performances) {
    let thisAmount = amountFor(perf, play);
  ```
- 성능 문제 
  - `before`: 루프를 한 번 돌때마다 공연 조회
  - `after`: 세 번 조회
  - 👍 제대로 리팽터링된 코드베이스는 그렇지 않은 코드보다 성능을 개선하기가 훨씬 수월하다.

**`함수 선언 바꾸기`**

- 긴 함수를 작게 쪼개는 리팩터링은 이름을 잘 지어야만 효과가 있다.



 
