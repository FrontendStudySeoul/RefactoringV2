# Chapter02. 리팩터링 원칙

<br />

## 2.1 리팩터링 정의

> 명사: 소프트웨어의 겉보기 동작은 그대로 유지한 채, 코드를 이해하고 수정하기 쉽도록 내부 구조를 변경하는 기법

> 동사: 소프트웨어의 겉보기 동작은 그대로 유지한 채, 여러 가지 리팩터링 기법을 적용해서 소프트웨어를 재구성한다.


리팩터링은 특정한 방식에 따라 코드를 정리하는 것.
- 동작을 보존하는 작은 단계들을 거쳐 코드 수정
- 이러한 단계들을 순차적으로 연결해 큰 변화


리팩터링, 성능 최적화는 **코드를 변경하지만 프로그램의 전반적인 기능은 그대로 유지**한다는 점에서 비슷, **목적**은 다름

<br />

## 2.2 두 개의 모자

1. 기능 추가할 땐 기존 코드는 절대 건드리지 않고 새 기능을 추가
2. 리팩터링할 때 기능 추가는 절대 하지 않고 오로지 코드 재구성에만 전념

<br />

## 2.3 리팩터링하는 이유

<br />

### 리팩터링하면 소프트웨어 설계가 좋아진다.

같은 일을 하더라도 설계가 나쁘면 코드가 길어지는데 중복 코드 제거는 설계 개선 작업의 중요한 부분
코드량이 줄어든다고 시스템이 빨라지는건 아니지만, 코드량이 줄면 수정하는 데 드는 노력이 덜어짐. 

중복 코드를 제거하면 모든 코드가 언제나 고유한 일을 수행할 수 있도록 보장할 수 있고, 이는 바람직한 설계의 핵심

<br />

### 리팩터링하면 소프트웨어를 이해하기 쉬워진다.

프로그램을 동작시키는 데만 신경 쓰다 보면 나중에 유지보수가 어려워질 수 있음

잘 작동하지만 이상적인 구조가 아닌 코드를 발견했다면 잠시 시간을 내서 리팩터링 해보자.
코드의 목적이 잘 드러나게, 자신의 의도를 더 명확하게 전달할 수 있도록 리팩터링한다면 나를 포함한 다른 개발자를 배려하는 것

<br />

### 리팩터링하면 버그를 쉽게 찾을 수 있다.

`코드를 이해하기 쉽다.` = `버그를 찾기 쉽다.`

프로그램의 구조를 명확하게 다듬으면 버그를 지나치려야 지나칠 수 없을 정도까지 명확해짐

> 난 뛰어난 프로그래머가 아니에요. 단지 뛰어난 습관을 지닌 괜찮은 프로그래머일 뿐이에요. - 켄트 백

<br />

### 리팩터링하면 프로그래밍 속도를 높일 수 있다.

위의 장점들을 한 마디로 정리하면 **`리팩터링하면 코드 개발 속도를 높일 수 있다.`** 이다.


**지구력 가설**

- 내부 설계에 심혈을 기울이면 소프트웨어의 지구력이 높아짐
- 빠르게 개발할 수 있는 상태를 더 오래 지속할 수 있음 

<br />

## 2.4 언제 리팩터링해야 할까?

**3의 법칙**

1. 처음에는 그냥 한다.
2. 비슷한 일을 두 번째로 하게 되면, 일단 계속 진행한다.
3. 비슷한 일을 세 번째 하게 되면 리팩터링한다.

<br />

### `준비`를 위한 리팩터링: 기능을 쉽게 추가하게 만들기

리팩터링하기 가장 좋은 시점은 **코드베이스에 기능을 새로 추가하기 직전**

구조를 살짝 바꾸면 다른 작업을 하기 훨씬 쉬워질 만한 부분을 찾자.

버그를 잡을 때도 오류를 일으키는 곳을 여러곳에 두지 말고 한 곳으로 합치는 편이 훨씬 편하다.

<br />

### `이해`를 위한 리팩터링: 코드를 이해하기 쉽게 만들기

자신이 이해한 것을 코드에 반영하자. (사람의 기억력을 믿지 말자)
- 적절한 변수명으로 변경하기
- 긴 함수를 잘게 나누기

> a.k.a 밖을 잘 내다보기 위한 창문 닦기

<br />

### 쓰레기 줍기 리팩터링



쓸데없이 복잡하거나, 코드 중복을 발견할 때가 있다.
원래 하려던 작업과 관련없는 일에 시간을 뺏기고 싶지 않지만, 쓰레기가 나뒹굴게 방치하는 것도 좋지 않다.

**항상 처음 봤을 때보다 깔끔하게 정리하고 떠나자.**

<br />

### 계획된 리팩터링과 수시로 하는 리팩터링

> 보기 싫은 코드를 발견하면 리팩터링하자. 그런데 잘 작성된 코드 역시 수많은 리팩터링을 거쳐야 한다.

> 무언가 수정하려 할 때는 먼저 수정하기 쉽게 정돈하고 그런 다음 쉽게 수정하자.

앞에서 언급한 리팩터링들은 모두 **기회가 될 때만 진행**한다.

하지만 리팩터링은 우리가 프로그래밍할 때 if문 작성 시간을 따로 구분하지 않는 것처럼 
**프로그래밍과 구분되는 별개의 활동이 아니다.**

새 기능을 추가하기 쉽도록 정기적인 리팩터링으로 코드 베이스를 개선할 필요가 있다.
하지만 정기적으로 리팩터링을 하더라도 어떤 문제는 곪아갈 수도 있다.

따라서 리팩터링 작업은 **드러나지 않게, 기회가 될 때마다** 해야 한다.

<br />

### 오래 걸리는 리팩터링

간단한 리팩터링이 있다면 팀 전체가 달려들어도 몇 주 걸리는 대규모 리팩터링도 있다.

팀 전체가 리팩터링에 매달리는 것이 아니라 누구든지 리팩터링해야할 코드와 관련된 작업을 하게 될 때마다 원하는 방향으로 조금씩 개선하는 식이다.

리팩터링이 **코드를 깨트리지 않는다**는 장점을 활용하자.

<br />

### 코드 리뷰에 리팩터링 활용하기

`Pull Request`, `페어 프로그래밍` 등

나란히 앉아서 코드를 훑어가면서 리팩터링하는 것이 가장 좋은 방법이다. (필자 기준)

<br />

### 관리자에게는 뭐라고 말해야 할까?

기술을 모르는 상당수의 관리자와 고객은 코드베이스의 건강 상태가 생산성에 미치는 영향을 모른다.
이러한 경우라면 리팩터링한다고 말하지 마라.

개발자는 **프로**다. 프로 개발자의 역할을 효과적인 소프트웨어를 최대한 빠르게 만드는 것이다.
기능을 추가하든지, 버그를 수정하든지, 리팩터링부터 하는 편이 가장 빠르다.

프로 개발자에게 주어진 임무는 **새로운 기능을 빠르게 구현**하는 것이고, 가장 빠른 방법은 **리팩터링**이다.

<br />

### 리팩터링하지 말아야 할 때

무조건 리팩터링을 권장하는 것이 아니다.

외부 API 다루듯 호출해서 사용하는 코드라면 지저분하더라도 그냥 두자.
그리고 내부 동작을 이해해야 할 시점에 리팩터링을 해서 효과를 보자.

리팩터링할지 새로 작성할지를 잘 결정하려면 **뛰어난 판단력과 경험이 뒷받침**돼야 한다.
한 마디 조언으로 표현하기 어렵다.

<br />

## 2.5 리팩터링 시 고려할 문제

<br />

### 새 기능 개발 속도 저하

직접 경험하지 않고선 코드베이스가 건강할 때와 허약할 때의 생산성 차이를 체감하기 어렵다.

리팩터링의 본질은 코드 베이스를 예쁘게 꾸미는 데 있지 않다.
오로지 경제적인 이유로 하는 것이다.

리팩터링은 개발 기간을 단축하고자 하는 것이다. 리팩터링하도록 이끄는 동력은 어디까지나 경제적인 효과에 있다.

<br />

### 코드 소유권

코드 소유권이 나뉘어 있으면 리팩터링에 방해가 된다.

코드의 소유권을 팀에 두고 팀원이라면 다른 사람이 작성한 코드라고 하더라도 누구나 팀이 소유한 코드를 수정할 수 있도록 한다.

<br />

### 브랜치

**장점**
- 작업이 끝나지 않은 코드가 마스터에 섞이지 않고
- 기능이 추가될 때마다 버전을 명확히 나눌 수 있고
- 문제가 생기면 이전 상태로 쉽게 돌릴 수 있다.

**단점**
- 브랜치를 독립적으로 사용하는 기간이 길어질 수록 마스터로 통합하는 것이 어려워진다.

**대안**
- 지속적 통합(CI)으로 다른 브랜치들과의 차이가 크게 벌어지지 않도록 머지 복잡도 낮추기

<br />

### 테스팅

리팩터링의 특성은 **`프로그램의 동작은 똑같이 유지된다는 것`** 이다.

리팩터링은 단계별 변경 폭이 작아 오류의 원인이 되는 코드 범위가 넓지 않다.
핵심은 **오류를 빠르게 잡는 데** 있다.

테스트 코드는 리팩터링을 할 수 있게 해줄 뿐만 아니라, 안전하게 새 기능을 추가할 수 있도록 도와준다.
테스트가 실패한다면 가장 최근에 통과한 버전에서 무엇이 달라졌는지 살펴보자.

<br />

### 레거시 코드

대규모 레거시 시스템을 테스트 코드 없이 명료하게 리팩터링하기는 어렵다.

쉽게 해결할 방법은 없다.
프로그램에서 테스트를 추가할 틈새를 찾아서 시스템을 테스트 해야 한다.

서로 관련된 부분끼리 나눠서 하나씩 공략해 리팩터링하자.
시스템의 규모가 크다면 자주 보는 부분을 더 많이 리팩터링하자.
- 코드를 훑는 횟수가 많다면 그 부분을 이해하기 쉽게 개선했을 때 얻는 효과도 그만큼 크다는 뜻

<br />

### 데이터 베이스

데이터 베이스 리팩터링은 프로덕션 환경에서 여러 단계로 나눠서 릴리스 하는 것이 대체로 좋다.

예를 들어

- 첫 번째는 새로운 데이터 베이스 필드를 추가하기만 하고 사용하지 않는다.
- 그 다음 기존 필드와 새 필드를 동시에 업데이트 하도록 설정한다.
- 그 다음에는 데이터 베이스를 읽는 클라이언트들을 새 필드를 사용하는 버전으로 조금씩 교체한다.
- 이후 문제가 없다면 예전 필드를 삭제한다.

<br />

## 2.6 리팩터링, 아키텍쳐, 애그니(YAGNI)

개발을 시작하기 전에 소프트웨어 설계와 아키텍처를 어느정도, 심지어 거의 완료해야 한다고 배움
소프트웨어 요구사항을 사전에 모두 파악한다는것 실현할 수 없는 목표일 때가 많다.  

유연성 메커니즘을 소프트웨어에 심어두자.
다양한 예상 시나리오에 대응하기 위해 매개변수를 추가하는 경우
- 생각나는 대로 추가한다면 당장 쓰임에 비해 함수가 너무 복잡해지고,
- 깜빡 잊었다면 앞서 추가한 매개변수들 때문에 새로 추가하기가 더 어려워진다.

이렇게 유연성 매커니즘이 오히려 변화에 대응하는 능력을 떨어뜨릴 때가 대부분이다.  


리팩터링을 활용하면 **어느 부분에 유연성이 필요**하고, **어떻게 변화에 대응**해야 하는지 **추측하지 않고** 그저 현재까지 파악한 요구사항만을 해결하는 소프트웨어를 구축한다.  

**`YAGNI(you aren't going to need it)`**  

그렇다고 아키텍처에 완전히 소홀해도 된다는 뜻이 아니다. 다만 둘 사이의 균형이 크게 달라졌다는 것이 중요하다.

<br />

## 2.8 리팩터링과 성능

`직관적인 설계` vs `성능`은 중요한 주제다.

리팩터링하면 소프트웨어가 느려질 수 있다는 것은 사실이지만 동시에 성능을 튜닝하기는 더 쉬워진다.

잘 리팩터링해뒀을 때 장점
- 성능 튜닝에 투입할 시간을 벌 수 있다.
- 리팩터링이 잘 되어 있다면 성능을 더 세밀하게 분석할 수 있다.

단기적으로 보면 리팩터링 단계에서 성능이 느려질 수 있지만, 최적화 단계에서 코드를 튜닝하기 훨씬 쉬워지기 때문에 결국 더 빠는 소프트웨어를 얻게 된다.

