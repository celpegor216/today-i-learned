# User Interface

- 화면에 표시되는 텍스트, 이미지, 버튼, 기타 여러 유형의 요소 및 화면에 표시되는 방식
- 앱이 콘텐츠를 사용자에게 표시하는 방식이자 사용자가 앱과 상호작용하는 방식

# Jetpack Compose

- Android UI를 빌드하기 위해 사용하는 최신 툴킷
- 적은 양의 코드, 강력한 도구 및 직관적인 Kotlin 기능으로 Android에서 UI 개발을 간소화하고 가속
- 데이터를 받아서 UI 요소를 설명하는 Composable 함수 집합을 정의하여 UI를 빌드

---

# Composable 함수

- Compose에서 UI의 기본 빌드 블록
- UI의 구성 과정에 집중하기보다는 앱의 모양을 설명하여 앱의 UI를 프로그래매틱 방식으로 정의할 수 있음
- 입력은 받을 수 있고, 반환은 없음
- 함수명은 파스칼 표기법에 따름(첫 글자 대문자 필수), 명사 혹은 형용사 + 명사 형태([공식 문서](https://github.com/androidx/androidx/blob/androidx-main/compose/docs/compose-api-guidelines.md#naming-unit-composable-functions-as-entities))
- 함수 앞에 `@Composable` 주석 추가 필수

## Modifier

- 기본값을 지정해두고서도 인수를 전달해야 하는 이유

  ```kotlin
  @Composable
  fun DiceWithButtonAndImage(modifier: Modifier = Modifier) {}

  DiceWithButtonAndImage(modifier = Modifier)
  ```

  - 컴포저블이 재구성될 수 있는데, 기본값에 의존하면 재구성될 때 Modifier가 다시 만들어져서 비효율적이기 때문
  - 모든 컴포저블에 Modifier 매개변수를 선택적 매개변수로 추가하여 재사용성을 높이는 것이 권장됨

- scroll 추가하는 방법
  - `.verticalScroll(rememberScrollState())`: 세로로 스크롤 가능, rememberScrollState()가 스크롤 상태를 만들어 자동으로 기억함

## Image

- `painterResource()`로 드로어블 이미지 리소스를 로드해서 painter의 인수로 전달
  ```
  Image(
    painter = painterResource(R.drawable.androidparty)
  )
  ```
- `contentDescription`: 앱 접근성을 개선하기 위한 콘텐츠의 설명, 장식 목적 등 설명이 필요하지 않는 경우 null로 설정
- `contentScale`: 배율 유형을 설정해서 이미지 크기 조절

## LazyCloumn

- Column
  - 한 번에 모두 로드
  - 사전 정의된 또는 고정된 개수의 컴포저블만 보유할 수 있음
  - 표시할 항목이 적은 경우에 적합함
- LazyColum
  - 주문형 콘텐츠를 추가할 수 있어, 긴 목록이나 목록의 길이를 알 수 없을 때 유용함
  - 기본적으로 스크롤도 제공함
  - `items()`로 항목을 추가

---

# Compose의 상태

- 컴포지션은 초기 컴포지션을 통해 생성되고, 이후에는 상태를 추적해서 상태가 변경될 때에만 리컴포지션을 통해 업데이트 됨
- `State`: Compose가 추적 가능한 상태, 읽기만 가능
- `MutableState`: Compose가 추적 가능한 상태, 읽기 및 쓰기 가능
- `mutableStateOf`: 추적 가능한 MutableState 생성, 초깃값을 State에 래핑된 매개변수로 수신하고 value의 값을 관찰 가능한 상태로 만들어 리컴포지션을 발생시킴
- `remember`: 계산된 값을 초기 컴포지션 중에 컴포지션에 저장하고 리컴포지션 중에 반환함
- by로 remember에 속성을 위임하여 getter, setter 자동 적용

## 상태 호이스팅

- 호이스팅이 필요한 상황: 상태를 여러 컴포저블이 공유하는 경우, 재사용할 수 있는 Stateless 컴포저블을 만드는 경우
- 컴포저블이 가능한 한 적게 상태를 소유하고, 컴포저블의 API에 상태를 노출하여 상태를 호이스팅할 수 있도록 컴포저블을 디자인해야 함
- 컴포저블에 value, onValueChange를 매개변수로 추가
