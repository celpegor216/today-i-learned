# 소프트웨어 테스트

- 소프트웨어가 예상대로 작동하는지 확인하는 구조화된 방법
- 앱을 공개적으로 출시하기 전에 앱의 정확성, 기능 동작 및 사용성을 확인할 수 있음
- 변경사항이 도입될 때 테스트를 통해 기존 코드를 지속적으로 확인할 수 있음

## 자동 테스트

- 개발자가 작성한 코드의 또 다른 부분이 올바르게 작동하는지 확인하는 코드 <-> 사람이 직접 테스트하는 수동 테스트
- 수동 테스트보다 적은 시간을 보시하며 대부분의 사용 시나리오를 테스트할 수 있음
- 새 기능을 만들 때마다 테스트를 만들면 추후 앱이 성장할 때의 작업량이 줄어듦

### 로컬 테스트

- 함수, 클래스, 속성 등 소수의 코드를 직접 테스트하여 제대로 작동하는지 확인
- 워크스테이션(기기나 에뮬레이터가 아닌 개발 환경, Android Studio)에서 실행됨
- 컴퓨터 리소스의 오버헤드가 낮아 제한된 리소스에서도 빠르게 실행 가능
- 항목이 구현되는 방식에 집중하는 일반 메서드와 달리, 지정된 입력의 예상 출력을 엄격하게 확인함
- 작성 방법
  1. 테스트할 함수, 클래스, 속성 등을 `internal` 키워드로 선언하고 `@VisibleForTesting` 어노테이션 추가해서 테스트 목적으로만 공개됨을 표시
  2. `app/src/test [unitTest]/java/패키지명`(Android 뷰에서는 `패키지명 (test)`)에 클래스를 생성해서 메서드 형태로 작성
     - 프로젝트 생성하면 `ExampleUnitTest`라는 이름으로 샘플이 생성되어 있음
     - 메서드 이름에 테스트의 내용과 예상 결과를 명확히 기재
  3. `@Test` 어노테이션 추가해서 컴파일러에게 테스트 메서드임을 알림
  4. 메서드 마지막에 이름에 `assert`가 있는 함수를 호출해서 코드의 결과와 예상 결과가 같은지 확인
  5. 메서드 이름 왼쪽의 초록색 화살표를 눌러 실행

```kotlin
@Test
fun calculateTip_20PercentNoRoundUp() {
    val amount = 10.00
    val tipPercent = 20.00

    val expectedTip = NumberFormat.getCurrencyInstance().format(2)
    val actualTip = calculateTip(
        amount = amount,
        tipPercent = tipPercent,
        roundUp = false
    )

    assertEquals(expectedTip, actualTip)
}
```

### 계측 테스트

- Android 개발에서는 UI 테스트를 의미
- Android API와 플랫폼 API 및 서비스에 종속된 앱 일부를 테스트
- 앱과 UI의 실제 인스턴스를 테스트하기 때문에 onCreate에서 콘텐츠를 설정하는 방식과 유사하게 UI 콘텐츠 설정
- 테스트 APK를 생성해서 앱이나 앱의 일부를 실행하고 사용자 상호작용을 시뮬레이션하여 앱이 적절하게 반응했는지 확인 -> 기기나 에뮬레이터에서 실행됨

- 작성 방법
  1. `app/src/androidTest/java/패키지명`(Android 뷰에서는 `패키지명 (androidTest)`)에 클래스를 생성해서 메서드 형태로 작성
     - 프로젝트 생성하면 `ExampleInstrumentedTest`라는 이름으로 샘플이 생성되어 있음
  2. `createComposeRule`로 composeTestRule 생성 및 `@get:Rule` 어노테이션 추가
  3. `@Test` 어노테이션 추가해서 컴파일러에게 테스트 메서드임을 알림
     - 컴파일러는 디렉터리에 따라 로컬 테스트인지 계측 테스트인지 구분함
  4. `composeTestRule.setContent()`을 호출해서 composeTestRule에 UI 콘텐츠 설정
  5. `composeTestRule.onNodeWithText()`로 특정 텍스트가 포함된 컴포저블에 접근 후 `.performTextInput()`으로 입력 값을 전달
  6. 메서드 마지막에 이름에 `assert`가 있는 함수를 호출해서 코드의 결과와 예상 결과가 같은지 확인
  7. 메서드 이름 왼쪽의 초록색 화살표를 눌러 실행

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
