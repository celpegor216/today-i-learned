# User Interface

- 화면에 표시되는 텍스트, 이미지, 버튼, 기타 여러 유형의 요소 및 화면에 표시되는 방식
- 앱이 콘텐츠를 사용자에게 표시하는 방식이자 사용자가 앱과 상호작용하는 방식

# Jetpack Compose

- Android UI를 빌드하기 위해 사용하는 최신 툴킷
- 적은 양의 코드, 강력한 도구 및 직관적인 Kotlin 기능으로 Android에서 UI 개발을 간소화하고 가속
- 데이터를 받아서 UI 요소를 설명하는 Composable 함수 집합을 정의하여 UI를 빌드
- 종속성 추가
  - BOM(재료명세서): Compose 라이브러리 버전을 관리하는 데 권장되는 방법, BOM 버전만 지정하면 다른 Compose 라이브러리의 버전은 최신 안정화된 버전으로 자동으로 지정됨
  ```kotlin
  implementation(platform("androidx.compose:compose-bom:2023.06.01"))
  implementation("androidx.compose.ui:ui")
  ```

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

## Scaffold & TopAppBar

- 다양한 구성요소와 TopAppBar를 포함한 화면 요소의 슬롯을 제공하는 레이아웃
  - TopAppBar: 앱에 브랜딩과 개성을 적용하는 용도 등 다양하게 사용 가능, center, small, medium, large 4가지 유형이 있음
- `contentWindowInsets`: 콘텐츠의 inset을 지정하는 데 도움이 되는 PaddingValues 데이터 유형의 매개변수, 화면에서 앱이 시스템 UI와 교차할 수 있는 부분이므로 내부 콘텐츠에 padding을 추가하는 게 좋음

```kotlin
@Composable
fun WoofApp() {
    Scaffold(
        topBar = {
            WoofTopAppBar()
        }
    ) { it ->
        LazyColumn(contentPadding = it) {
            items(dogs) {
                DogItem(
                    dog = it,
                    modifier = Modifier.padding(dimensionResource(id = R.dimen.padding_small))
                )
            }
        }
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun WoofTopAppBar(modifier: Modifier = Modifier) {
    CenterAlignedTopAppBar(
        title = {
            Row(verticalAlignment = Alignment.CenterVertically) {
                Image(
                    modifier = Modifier
                        .size(dimensionResource(id = R.dimen.image_size))
                        .padding(dimensionResource(id = R.dimen.padding_small)),
                    painter = painterResource(R.drawable.ic_woof_logo),
                    contentDescription = null
                )

                Text(
                    text = stringResource(R.string.app_name),
                    style = MaterialTheme.typography.displayLarge
                )
            }
        },
        modifier = modifier
    )
}
```

## 애니메이션

- 앱에 일어나고 있는 일을 사용자에게 알려주는 시각적 단서를 추가할 수 있음
- `Modifier.animateContentSize()`로 크기 변경 시 애니메이션 추가

  - animationSpec 값으로 spring 애니메이션 추가
  - spring 애니메이션
    - 스프링력에 기반한 물리학 기반 애니메이션
    - dampingRatio: 감쇠비, 스프링의 탄성 정도
    - stiffness: 강도 수준, 스프링이 끝까지 이동하는 속도

  ```kotlin
  Column(
      modifier = Modifier.animateContentSize(
          animationSpec = spring(
              dampingRatio = Spring.DampingRatioNoBouncy,
              stiffness = Spring.StiffnessMedium
          )
      )
  ) {}
  ```

- `animate*AsState()`

  - 단일 값을 애니메이션 처리하는 API
  - 최종 값을 지정하면 API가 현재 값에서 지정 값으로 애니메이션을 시작함
  - Float, Color, Dp, Size, Offset, Int 등 함수를 제공하고 animateValueAsState로 다른 데이터 유형도 지원함

  ```kotlin
  val color by animateColorAsState(
      targetValue = if (expanded) MaterialTheme.colorScheme.tertiaryContainer
      else MaterialTheme.colorScheme.primaryContainer,
      label = ""
  )

  Column(
      modifier = Modifier.animateContentSize(
          animationSpec = spring(
              dampingRatio = Spring.DampingRatioNoBouncy,
              stiffness = Spring.StiffnessMedium
          )
      )
          .background(color = color)
  ) {}
  ```

---

# Compose의 상태

- 컴포지션은 초기 컴포지션을 통해 생성되고, 이후에는 상태를 추적해서 상태가 변경될 때에만 리컴포지션을 통해 업데이트 됨
- `State`: Compose가 추적 가능한 상태, 읽기만 가능
- `MutableState`: Compose가 추적 가능한 상태, 읽기 및 쓰기 가능
- `mutableStateOf`: 추적 가능한 MutableState 생성, 초깃값을 State에 래핑된 매개변수로 수신하고 value의 값을 관찰 가능한 상태로 만들어 리컴포지션을 발생시킴
- `remember`: 계산된 값을 초기 컴포지션 중에 컴포지션에 저장하고 리컴포지션 중에 반환함
  - by로 remember에 속성을 위임하여 getter, setter 자동 적용
- `rememberSaveable`: 구성 변경 중에 상태를 유지

## 상태 호이스팅

- 호이스팅이 필요한 상황: 상태를 여러 컴포저블이 공유하는 경우, 재사용할 수 있는 Stateless 컴포저블을 만드는 경우
- 컴포저블이 가능한 한 적게 상태를 소유하고, 컴포저블의 API에 상태를 노출하여 상태를 호이스팅할 수 있도록 컴포저블을 디자인해야 함
- 컴포저블에 value, onValueChange를 매개변수로 추가

---

# Navigation

## Navigation 경로

- URL과 유사하게, 대상(단일 Composable 혹은 Composable 그룹)에 매핑되어 고유한 식별자 역할을 하는 문자열
- 앱의 화면 수가 한정되어 있는 것처럼 경로도 한정되어 있어 enum 클래스를 사용해 경로를 정의할 수 있음

  ```kotlin
  enum class CupcakeScreen(@StringRes val title: Int) {
    Start(title = R.string.app_name),
    Flavor(title = R.string.choose_flavor),
    Pickup(title = R.string.choose_pickup_date),
    Summary(title = R.string.order_summary)
  }
  ```

## NavHost

- NavGraph의 현재 대상을 표시하는 컨테이너 역할을 하는 Composable

  ```kotlin
  private fun cancelOrderAndNavigateToStart(
      viewModel: OrderViewModel,
      navController: NavHostController
  ) {
      viewModel.resetOrder()
      navController.popBackStack(CupcakeScreen.Start.name, inclusive = false)
  }

  NavHost(
      navController = navController,
      startDestination = CupcakeScreen.Start.name,
      modifier = Modifier.padding(innerPadding)
  ) {
      composable(route = CupcakeScreen.Start.name) {
          StartOrderScreen(
              quantityOptions = DataSource.quantityOptions,
              onNextButtonClicked = {
                  viewModel.setQuantity(it)
                  navController.navigate(CupcakeScreen.Flavor.name)
              },
              modifier = Modifier
                  .fillMaxSize()
                  .padding(dimensionResource(R.dimen.padding_medium))
          )
      }
      composable(route = CupcakeScreen.Flavor.name) {
          val context = LocalContext.current
          SelectOptionScreen(
              subtotal = uiState.price,
              onNextButtonClicked = { navController.navigate(CupcakeScreen.Pickup.name) },
              onCancelButtonClicked = {
                  cancelOrderAndNavigateToStart(viewModel, navController)
              },
              options = DataSource.flavors.map { id -> context.resources.getString(id) },
              onSelectionChanged = { viewModel.setFlavor(it) },
              modifier = Modifier.fillMaxHeight()
          )
      }
      composable(route = CupcakeScreen.Pickup.name) {
          SelectOptionScreen(
              subtotal = uiState.price,
              onNextButtonClicked = { navController.navigate(CupcakeScreen.Summary.name) },
              onCancelButtonClicked = {
                  cancelOrderAndNavigateToStart(viewModel, navController)
              },
              options = uiState.pickupOptions,
              onSelectionChanged = { viewModel.setDate(it) },
              modifier = Modifier.fillMaxHeight()
          )
      }
      composable(route = CupcakeScreen.Summary.name) {
          OrderSummaryScreen(
              orderUiState = uiState,
              onCancelButtonClicked = {
                  cancelOrderAndNavigateToStart(viewModel, navController)
              },
              onSendButtonClicked = { subject: String, summary: String ->
                  val intent = Intent(Intent.ACTION_SEND).apply {
                      type = "text/plain"
                      putExtra(Intent.EXTRA_SUBJECT, subject)
                      putExtra(Intent.EXTRA_TEXT, summary)
                  }
                  context.startActivity(
                      Intent.createChooser(
                          intent,
                          context.getString(R.string.new_cupcake_order)
                      )
                  )
              },
              modifier = Modifier.fillMaxHeight()
          )
      }
  }
  ```

- navController: NavHostController 클래스의 인스턴스, Composable 함수에서 `rememberNavController()`를 호출하여 가져올 수 있음
  - `navigate(route)`: 경로에 해당하는 화면으로 이동
  - `popBackStack(route, inclusive)`: 경로에 해당하는 화면으로 돌아감, inclusive가 false인 경우 시작 대상 위의 모든 대상을 삭제함
- startDestination: NavHost를 처음 표시할 때 기본적으로 표시되는 대상을 정의

## NavController

- 앱 화면 간 이동 담당
- BackStack

  - startDestination부터 최상단 화면까지의 화면 기록

  ```kotlin
  val backStackEntry by navController.currentBackStackEntryAsState()

  val currentScreen = CupcakeScreen.valueOf(
      backStackEntry?.destination?.route ?: CupcakeScreen.Start.name
  )

  CupcakeAppBar(
      currentScreen = currentScreen,
      canNavigateBack = navController.previousBackStackEntry != null,
      navigateUp = { navController.navigateUp() }
  )
  ```

## NavGraph

- 이동할 Composable 대상 매핑
