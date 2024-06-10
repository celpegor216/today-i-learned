# App Architecture

- 앱의 디자인 규칙 집합
- 코드를 강력하고, 유연하며, 확장 가능하고, 테스트 가능하도록 만들어 유지보수가 용이한 구조
- 가장 일반적인 원칙
  - **관심사 분리**: 각각 별개의 책임이 있는 여러 함수 클래스로 앱을 나눠야 함
  - **model에서 UI 만들기**: 앱의 UI 요소 및 구성요소와 독립적으로 앱의 데이터 처리를 담당하는 model에서 UI를 만들어야 앱의 수명주기 및 관련 문제의 영향을 받지 않음

## 일반적인 원칙에 따른 Layer 구조

### UI(Presentation) Layer

- 화면에 앱 데이터를 표시
- **UI 요소**: 화면에 데이터를 렌더링, Jetpack Compose를 사용해서 빌드
- **상태 홀더**: 데이터(상태)를 보유하고 UI에 노출하며 앱 로직을 처리
- **ViewModel**
  - 상태 홀더의 일종
  - rememberSaveable을 사용하거나 인스턴스 상태를 저장하는 등 방법으로 앱 데이터를 저장할 수 있지만, 이 경우 Composable 함수 내부 혹은 근처에 로직을 유지해야 함
  - 앱의 규모가 커진다면 로직을 Composable 함수와 분리해야 함
  - ViewModel은 Activity 인스턴스와 달리 소멸되지 않아, Activity가 소멸되고 다시 생성될 때 소멸되지 않아야 하는 데이터를 저장함
  - 종속성 추가 필요
    ```kotlin
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.6.1")
    ```
  - `ViewModel` 클래스를 확장한 클래스 생성
- **StateFlow**
  ```kotlin
  private val _uiState = MutableStateFlow(GameUiState())
  val uiState: StateFlow<GameUiState> = _uiState.asStateFlow()
  ```
  - 현재 상태와 새로운 상태 변화를 내보내는 관찰 가능한 상태 홀더의 흐름
  - `value`: 현재 상태 값, 상태를 변경하려면 이 속성에 새로운 값을 할당하면 됨
  - `asStateFlow()`: 읽기 전용 상태 흐름
  - `collectAsState()`: StateFlow에서 값을 수집하고 State를 통해 최신 값 표시, value가 초깃값으로 사용되고 StateFlow에 새 값이 할당될 때마다 State.value가 사용된 모든 곳에서 Recomposition이 발생함
- **Backing property**
  - 실제 객체가 아닌 getter를 통해 값을 반환
  - 외부 클래스에서 값의 조회는 가능하되 수정은 불가능하도록 설정
  - private 키워드와 함께 선언하고 이름 앞에 `_`를 붙임
- 여러 소스가 한 순간에 앱의 상태를 변경하지 않도록 보장하기 위해 UI 상태 정의는 변경할 수 없어야 함

#### 단방향 데이터 흐름(UDF)

![이미지](https://developer.android.com/static/codelabs/basic-android-kotlin-compose-viewmodel-and-state/img/61eb7bcdcff42227_960.png?hl=ko)

- 상태는 아래로 이동하고 이벤트는 위로 이동하는 디자인 패턴
- UI 업데이트 과정
  1. 이벤트 발생: UI 일부가 이벤트를 생성하거나(ex. 버튼 클릭), 앱의 다른 레이어에서 이벤트 전달(ex. 사용자 세션 종료)
  2. 상태 업데이트: 이벤트 핸들러(ViewModel)가 상태 변경
  3. 상태 표시: 상태 홀더가 상태를 아래로 전달, UI 요소가 상태 표시
- 구현
  1. Screen Composable의 인자로 ViewModel 인스턴스 전달
  ```kotlin
  @Composable
  fun GameScreen(
    gameViewModel: GameViewModel = viewModel()
  ) {}
  ```
  2. `collectAsState()`를 통해 ViewModel 인스턴스를 사용하여 uiState에 접근
  ```kotlin
  val gameUiState by gameViewModel.uiState.collectAsState()
  ```
  3.

### Domian Layer

- UI와 Data 레이어 간 상호작용을 간소화하고 재사용

### Data Layer

- 앱 데이터를 저장하고, 가져오고, 노출함

- model

- 데이터 클래스로 표시되는 데이터 모델 포함
