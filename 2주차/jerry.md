## 1.7 중간점검: 두 파일(과 두 단계)로 분리됨

- 두 파일로 분리되어 코드량이 늘었지만 전체적인 로직을 구성하는 '계산'부분과 '출력'부분으로 분리되어 코드 파악이 쉽다.
- 간결함 보다는 명료함이 중요하다.

### 모듈화의 장점

> 모듈화의 주된 목적 중 하나는 코드가 향후에 어떻게 변경되거나 재구성될지 정확히 알지 못한 상태에서 변경과 재구성이 용이한 코드를 작성하는 것이다.

- 재사용성
- 테스트에 용이

<br />

## 📚 좋은 코드, 나쁜 코드

### 깊이 중첩된 코드를 피해라

> 중첩은 하나의 함수에서 너무 많은 일을 처리한 결과이기도 하다.

- 가장 좋은 건 더 작은 함수로 분리해 추상화하는 것이 좋다.
- `르블랑의 법칙`: "한번 작성한 쓰레기 코드를 나중에 수정하는 일은 결코 없다"

<br />

### 함수 인자의 개수

- 함수의 인자 개수는 최대 2개로 유지하고 2개 이상인 경우 객체 자체를 넘기자
- 컴포넌트에서도 동일하게 작용

<br />

### 관련 있는 데이터는 캡슐화하라

컴포넌트에 특정 데이터의 필드들을 나눠 prop으로 전달하지 않고 데이터 자체를 prop으로 넘기는 방식을 떠올릴 수 있겠다.

```tsx
<Product name={product.name} price={product.price} stock={product.stock} />
<Product product={product} /> // 👍
<Product {...product} /> // 👍
```

이렇게 객체 자체를 넘기면 스펙이 변경되어도 안정적이며, 나중에 유지보수에도 용이하다.
컴포넌트에 prop이 확장될 경우가 있다면 객체 자체를 넘겨 컴포넌트를 사용하는 관점에서 해당 컴포넌트의 구현 세부 사항을 알지 않아도 되는 이점이 있다.

<br />

### 익명 함수 적절히 사용하기

간단하고 자명한 곳에 익명 함수를 적절히 사용하면 코드의 가독성을 높여주지만, 복잡하거나 자명하지 않은 혹은 재사용해야 하는 것에 사용하면 문제가 될 수 있다.

```ts
const getUser = (userId) => {
  return userList.filter(({ id }) => userId === id);
};
```

아래 filter에 작성된 콜백 함수는 너무나 간단하고 쉽게 읽히기에 익명 함수를 작성하기에 적절하지만 만약 복잡한 조건을 표현해야 한다면 익명 함수로 작성하기 보다 명명 함수로 작성하는 것이 좋다. 익명 함수는 길면 문제가 되기 때문이다.

<br />

### 매직값 반환하지 않기

> 예를 들어 메서드 indexOf가 특정 값을 찾지 못하면 -1을 반환하는 경우를 말한다.

함수에서 매직값을 반환할 때의 문제점은 호출하는 쪽에서 함수 계약의 세부 조항을 알아야 한다는 점이다.

해결책으로는 널, 옵셔널 또는 차라리 오류를 반환하는 것이 좋다.

하지만 널을 반환하는 경우의 함수는 호출하는 곳에서 널을 반환하는 경우를 처리해야 하는 번거로움이 생긴다. 그렇지만 어떤 값이 존재하지 않을 수 있고 이 사실을 분명히 하지 않는다면 더 많은 버그를 발생시킬 수 있고, 버그를 처리하고 고치는 비용이 널 반환값을 올바르게 처리하기 위해 코드를 추가하는 비용보다 훨씬 높을 수 있다.

<br />

### 하드 코드화된 의존성은 문제가 될 수 있다

```ts
/* 
` useOnlineMap: 지도의 최신 버전을 가져올지의 여부
  includeSeasonalRoads: 특정 기간에만 개통되는 도로를 지도에 포함할지의 여부
*/
const getSuwonRoadMap = (
  useOnlineMap: boolean,
  includeSeasonalRoads: boolean
) => {};

const searchRoute = () => {
  const roadMap = getSuwonRoadMap(false, true);
  // ...
};
```

위 코드에서 searchRoute는 getSuwonRoadMap에 의존되어 항상 수원 지도를 사용할 때 최신 지도를 사용하지 않을 것이며 계절 도로는 제외되지 않을 것이므로 다용도로 사용할 수 없다는 단점이 있다.

이렇게 하드 코드화된 종속성은 바람직하지 않다.

```ts
interface RoadMap {
  startPoint: LatLong;
  endPoint: LatLong;
}

const getSuwonRoadMap = (
  useOnlineMap: boolean,
  includeSeasonalRoads: boolean
): RoadMap => {};
const searchRoute = (roadMap: RoadMap) => {};

searchRoute(getSuwonRoadMap(false, true));
```

차라리 의존성을 주입해 하드 코드화된 종속성을 없애고 어떠한 로드맵을 구할 수 있다.

<br />

### 미래를 대비한 열거형 처리

> 열거형을 처리해야 하는 경우 나중에 열거형에 더 많은 값이 추가될 수 있다는 점을 기억하는 것이 중요하다.

```ts
const OUTCOME = {
  GO_BUST: "GO_BUST",
  MAKE_A_PROFIT: "MAKE_A_PROFIT",
} as const;

const isOutcomeSafe = (prediction: keyof typeof OUTCOME) => {
  if (prediction == OUTCOME.GO_BUST) {
    return false;
  }

  return true;
};
```

위 코드는 앞으로 나중에 열거형 값이 더 추가될 수도 있음을 간과했다. 그 결과 불안정한 결과값을 반환하는 함수가 되었다.

```ts
const isOutcomeSafe = (prediction: keyof typeof OUTCOME) => {
  switch (prediction) {
    case OUTCOME.GO_BUST:
      return false;
    case OUTCOME.MAKE_A_PROFIT:
      return true;
    default:
      throw new Error("에러");
  }
};
```

암시적인 방식으로 처리하지 말고 명시적으로 작성해 처리되지 않은 새로운 열거값이 추가되는 경우 코드 컴파일이 실패하거나 테스트가 실패하도록 하는 것이 좋다.

일반적인 방법으로는 모든 경우를 다 처리하는 스위치 문을 사용하는 것이다.
