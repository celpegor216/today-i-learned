# 소프트웨어 테스트

- 소프트웨어가 예상대로 작동하는지 확인하는 구조화된 방법
- 앱을 공개적으로 출시하기 전에 앱의 정확성, 기능 동작 및 사용성을 확인할 수 있음
- 변경사항이 도입될 때 테스트를 통해 기존 코드를 지속적으로 확인할 수 있음

## 자동 테스트

- 개발자가 작성한 코드의 또 다른 부분이 올바르게 작동하는지 확인하는 코드 <-> 사람이 직접 테스트하는 수동 테스트
- 수동 테스트보다 적은 시간을 보시하며 대부분의 사용 시나리오를 테스트할 수 있음
- 새 기능을 만들 때마다 테스트를 만들면 추후 앱이 성장할 때의 작업량이 줄어듦

### 로컬 테스트(단위 테스트)

- 함수, 클래스, 속성 등 소수의 코드를 직접 테스트하여 제대로 작동하는지 확인
- 워크스테이션(기기나 에뮬레이터가 아닌 개발 환경, Android Studio)에서 실행됨
- 컴퓨터 리소스의 오버헤드가 낮아 제한된 리소스에서도 빠르게 실행 가능
- 항목이 구현되는 방식에 집중하는 일반 메서드와 달리, 지정된 입력의 예상 출력을 엄격하게 확인함

#### 작성 방법

1. 테스트할 함수, 클래스, 속성 등을 `internal` 키워드로 선언하고 `@VisibleForTesting` 어노테이션 추가해서 테스트 목적으로만 공개됨을 표시
2. `app/src/test [unitTest]/java/패키지명`(Android 뷰에서는 `패키지명 (test)`)에 클래스를 생성해서 메서드 형태로 작성
   - 프로젝트 생성하면 `ExampleUnitTest`라는 이름으로 샘플이 생성되어 있음
   - 메서드 이름에 테스트의 내용과 예상 결과를 명확히 기재
3. `@Test` 어노테이션 추가해서 컴파일러에게 테스트 메서드임을 알림
4. 메서드 마지막에 이름에 `assert`가 있는 함수를 호출해서 코드의 결과와 예상 결과가 같은지 확인
5. 메서드 이름 왼쪽의 초록색 화살표를 눌러 실행

#### 종속성 추가

- `testImplementation`: 로컬 테스트 소스 코드에는 적용되지만 애플리케이션 코드에는 적용되지 않아 APK에 포함되지 않음

  ```kotlin
  testImplementation("junit:junit:4.13.2")

  // Compose 테스트 라이브러리
  androidTestImplementation(platform("androidx.compose:compose-bom:2023.06.01"))
  androidTestImplementation("androidx.compose.ui:ui-test-junit4")
  ```

#### 테스트 전략

- 성공 경로: 예외나 오류 조건이 없는 흐름, 앱의 의도된 동작에 초점을 맞춤

  ```kotlin
  class GameViewModelTest {
      private val viewModel = GameViewModel()

      // 함수 이름 구성: 테스트하려는 것_조건_결과
      @Test
      fun gameViewModel_CorrectWordGuessed_ScoreUpdatedAndErrorFlagUnset() {
          var currentGameUiState = viewModel.uiState.value
          val correctPlayerWord = getUnscrambledWord(currentGameUiState.currentScrambledWord)
          viewModel.updateUserGuess(correctPlayerWord)
          viewModel.checkUserGuess()

          currentGameUiState = viewModel.uiState.value

          assertFalse(currentGameUiState.isGuessedWordWrong)
          assertEquals(SCORE_AFTER_FIRST_CORRECT_ANSWER, currentGameUiState.score)
      }

      @Test
      fun gameViewModel_WordSkipped_ScoreUnchangedAndWordCountIncreased() {
          var currentGameUiState = viewModel.uiState.value
          val correctPlayerWord = getUnscrambledWord(currentGameUiState.currentScrambledWord)
          viewModel.updateUserGuess(correctPlayerWord)
          viewModel.checkUserGuess()

          currentGameUiState = viewModel.uiState.value
          val lastWordCount = currentGameUiState.currentWordCount
          viewModel.skipWord()
          currentGameUiState = viewModel.uiState.value
          // Assert that score remains unchanged after word is skipped.
          assertEquals(SCORE_AFTER_FIRST_CORRECT_ANSWER, currentGameUiState.score)
          // Assert that word count is increased by 1 after word is skipped.
          assertEquals(lastWordCount + 1, currentGameUiState.currentWordCount)
      }

      companion object {
          private const val SCORE_AFTER_FIRST_CORRECT_ANSWER = SCORE_INCREASE
      }
  }
  ```

- 오류 경로: 오류 조건 또는 잘못된 입력에 어떻게 응답하는지 확인

  ```kotlin
  class GameViewModelTest {
      private val viewModel = GameViewModel()

      @Test
      fun gameViewModel_IncorrectGuess_ErrorFlagSet() {
          val incorrectPlayerWord = "and"    // 단어 목록에 없는 단어
          viewModel.updateUserGuess(incorrectPlayerWord)
          viewModel.checkUserGuess()

          val currentGameUiState = viewModel.uiState.value

          assertTrue(currentGameUiState.isGuessedWordWrong)
          assertEquals(0, currentGameUiState.score)
      }
  }
  ```

- 경계 사례: 앱의 경계 조건을 테스트

  ```kotlin
  class GameViewModelTest {
      private val viewModel = GameViewModel()

      @Test
      fun gameViewModel_Initialization_FirstWordLoaded() {
          val gameUiState = viewModel.uiState.value
          val unScrambledWord = getUnscrambledWord(gameUiState.currentScrambledWord)

          assertNotEquals(unScrambledWord, gameUiState.currentScrambledWord)
          assertTrue(gameUiState.currentWordCount == 1)
          assertTrue(gameUiState.score == 0)
          assertFalse(gameUiState.isGuessedWordWrong)
          assertFalse(gameUiState.isGameOver)
      }

      @Test
      fun gameViewModel_AllWordsGuessed_UiStateUpdatedCorrectly() {
          var expectedScore = 0
          var currentGameUiState = viewModel.uiState.value
          var correctPlayerWord = getUnscrambledWord(currentGameUiState.currentScrambledWord)

          repeat(MAX_NO_OF_WORDS) {
              expectedScore += SCORE_INCREASE
              viewModel.updateUserGuess(correctPlayerWord)
              viewModel.checkUserGuess()
              currentGameUiState = viewModel.uiState.value
              correctPlayerWord = getUnscrambledWord(currentGameUiState.currentScrambledWord)
              assertEquals(expectedScore, currentGameUiState.score)
          }

          assertEquals(MAX_NO_OF_WORDS, currentGameUiState.currentWordCount)
          assertTrue(currentGameUiState.isGameOver)
      }
  }
  ```

#### 테스트 적용 범위 확인

- Test파일명 우클릭 > `Run 'Test파일명' with Coverage` 클릭 > 테스트 실행 종료 후 Flatten Packages 옵션 클릭

  ![이미지](https://developer.android.com/static/codelabs/basic-android-kotlin-compose-test-viewmodel/img/14cf6ca3ffb557c4_960.png?hl=ko)

- 파일 별로 작성된 전체 메소드와 코드 줄 중 몇 %가 테스트에 사용되었는지 확인 가능
- 테스트에 사용되지 않은 코드를 사용하는 테스트를 작성해서 적용 범위를 개선할 수 있음
- 적용 범위가 넓은 것도 중요하지만, 동작을 확인하는 어설션이 있는지 확인해야 함

---

### 계측 테스트(UI 테스트)

- Android 개발에서는 UI 테스트를 의미
- Android API와 플랫폼 API 및 서비스에 종속된 앱 일부를 테스트
- 앱과 UI의 실제 인스턴스를 테스트하기 때문에 onCreate에서 콘텐츠를 설정하는 방식과 유사하게 UI 콘텐츠 설정
- 테스트 APK를 생성해서 앱이나 앱의 일부를 실행하고 사용자 상호작용을 시뮬레이션하여 앱이 적절하게 반응했는지 확인 -> 기기나 에뮬레이터에서 실행됨

#### 작성 방법

1. `app/src/androidTest/java/패키지명`(Android 뷰에서는 `패키지명 (androidTest)`)에 클래스를 생성해서 메서드 형태로 작성
   - 프로젝트 생성하면 `ExampleInstrumentedTest`라는 이름으로 샘플이 생성되어 있음
2. `createComposeRule`로 composeTestRule 생성 및 `@get:Rule` 어노테이션 추가
3. `@Test` 어노테이션 추가해서 컴파일러에게 테스트 메서드임을 알림
   - 컴파일러는 디렉터리에 따라 로컬 테스트인지 계측 테스트인지 구분함
4. `composeTestRule.setContent()`을 호출해서 composeTestRule에 UI 콘텐츠 설정
5. `composeTestRule.onNodeWithText()`로 특정 텍스트가 포함된 컴포저블에 접근 후 `.performTextInput()`으로 입력 값을 전달
6. 메서드 마지막에 이름에 `assert`가 있는 함수를 호출해서 코드의 결과와 예상 결과가 같은지 확인
7. 메서드 이름 왼쪽의 초록색 화살표를 눌러 실행

#### 종속성 추가

- UI 테스트는 자체 소스 세트 androidTest에 포함되어 있고, 앱 코드가 포함된 다른 모듈을 위해 컴파일할 필요가 없음
- `androidTestImplementation`: UI 테스트에서만 사용되는 종속 항목을 추가할 때 사용하는 키워드

  ```kotlin
  androidTestImplementation(platform("androidx.compose:compose-bom:2023.05.01"))
  androidTestImplementation("androidx.compose.ui:ui-test-junit4")
  androidTestImplementation("androidx.navigation:navigation-testing:2.6.0")
  androidTestImplementation("androidx.test.espresso:espresso-intents:3.5.1")
  androidTestImplementation("androidx.test.ext:junit:1.1.5")
  ```

#### 예제

- 기본 Compose

  ```kotlin
  @get:Rule
  val composeTestRule = createComposeRule()

  @Test
  fun calculate_20_percent_tip() {
      composeTestRule.setContent {
          MyApplicationTheme {
              TipTimeLayout()
          }
      }

      composeTestRule.onNodeWithText("Bill Amount")
          .performTextInput("10")
      composeTestRule.onNodeWithText("Tip Percentage").performTextInput("20")

      val expectedTip = NumberFormat.getCurrencyInstance().format(2)
      composeTestRule.onNodeWithText("Tip Amount: $expectedTip").assertExists(
          "No node with this text was found."
      )
  }
  ```

- Navigation

  ```kotlin
  // 테스트 규칙 생성
  @get:Rule
  val composeTestRule = createAndroidComposeRule<ComponentActivity>()

  // navHost의 경로 확인을 위해 필요
  private lateinit var navController: TestNavHostController

  // @Test보다 먼저 실행
  @Before
  fun setupCupcakeNavHost() {
      composeTestRule.setContent {
          navController = TestNavHostController(LocalContext.current).apply {
              navigatorProvider.addNavigator(ComposeNavigator())
          }
          CupcakeApp(navController = navController)
      }
  }

  // 경로 확인을 위한 확장함수
  fun NavController.assertCurrentRouteName(expectedRouteName: String) {
      assertEquals(expectedRouteName, currentBackStackEntry?.destination?.route)
  }

  @Test
  fun cupcakeNavHost_verifyStartDestination() {
      navController.assertCurrentRouteName(CupcakeScreen.Start.name)
  }

  // UI 구성요소에 접근하기 위한 확장함수
  fun <A : ComponentActivity> AndroidComposeTestRule<ActivityScenarioRule<A>, A>.onNodeWithStringId(
      @StringRes id: Int
  ): SemanticsNodeInteraction = onNodeWithText(activity.getString(id))

  @Test
  fun cupcakeNavHost_verifyBackNavigationNotShownOnStartOrderScreen() {
      val backText = composeTestRule.activity.getString(R.string.back_button)
      composeTestRule.onNodeWithContentDescription(backText).assertDoesNotExist()
  }

  @Test
  fun cupcakeNavHost_clickOneCupcake_navigatesToSelectFlavorScreen() {
      composeTestRule.onNodeWithStringId(R.string.one_cupcake)
          .performClick()
      navController.assertCurrentRouteName(CupcakeScreen.Flavor.name)
  }

  ```

---

# 접근성 테스트

- 장애가 있는 사용자 및 다른 접근성 기능이 필요한 사용자를 고려하여 사용자 환경을 개선하기 위해 접근성 테스트 필요

## Android 접근성 도구 모음을 활용한 접근성 테스트

1. 테스트를 진행할 기기에 Google Play 스토어에서 [Android 접근성 도구 모음](https://play.google.com/store/apps/details?id=com.google.android.marvin.talkback&hl=ko) 설치하고 테스트할 앱 설치
2. 설정에서 TalkBack을 활성화하고 앱을 테스트
   - TalkBack: 화면을 보지 않고도 기기를 탐색할 수 있도록 음성 피드백을 제공하는 구글 스크린 리더, 시각 장애인에게 유용
3. 설정에서 스위치 제어를 활성화하고 앱을 테스트
   - 스위치 제어: 터치스크린 대신 하나 이상의 스위치를 사용하여 기기와 상호작용 하는 기능, 수동기민성이 제한된 사용자에게 유용

## UI 접근성 개선 방법

- contentDescription
  - 스크린 리더 등 접근성 서비스를 이용할 때 인터페이스 요소의 의미를 이해하기 위한 정보
  - 장식적인 용도만을 위한 경우 null로 설정해도 됨
- 터치 영역 크기
  - 안정적으로 상호작용 하기 위한 최소 터치 영역 크기는 48dp \* 48dp
- 색상 대비
  - 충분한 고대비는 시각 장애인에게 도움이 될 뿐 아니라, 직사광선에 노출되었거나 어두운 곳에 있는 등 극단적인 채광 조건에서 기기를 볼 때 도움이 됨
  - [Android 접근성 도움말 문서](https://support.google.com/accessibility/android/answer/7158390?hl=ko)
  - [WebAIM](https://webaim.org/resources/contrastchecker/)에서 배경색과 전경색의 대비율을 테스트할 수 있음
  - 작은 텍스트의 권장 비율은 4.5:1, 큰 텍스트의 경우 권장 비율은 3.0:1
