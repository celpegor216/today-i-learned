# 측정 단위

- DP: 밀도 독립형 픽셀
- SP: 확장 가능한 픽셀, 글꼴 크기 단위, 사용자가 기기 설정에서 선택한 크기에 따라 조절됨
- padding 값은 4dp 단위로 사용하는 것이 좋음

# 이미지 추가

1. Android Studio에서 Resource Manager 탭 클릭
2. `+` > `Import Drawables` 클릭
3. QUALIFIER TYPE에서 Density, VALUE에서 No Density 선택 → 픽셀 크기가 다른 다양한 화면에 지원 가능
4. drawable-nodpi 폴더에 배치 → 크기 조절 동작이 중지되어, Android 시스템에서 처리할 수 있는 것보다 큰 이미지의 크기를 조절할 때 발생하는 메모리 부족 오류 예방

# Bitmap Image vs Vector Drawable

- Bitmap Image
  - 각 픽셀의 색상 정보를 제외한 다른 정보를 잘 알지 못함
  - 사진과 같은 경우에 적합함
- Vector Drawable
  - 이미지를 그리는 방법을 알고 있음(색상 정보, 일련의 점, 선, 곡선 등)
  - 화질 저하 없이 모든 화면 밀도의 어떤 캔버스 크기로도 조정할 수 있음
  - xml로 정의, 모든 밀도 버킷에 제공할 필요 없이 한 번만 정의하면 됨 -> 앱의 크기가 줄어듦
  - 아이콘과 같이 단순한 모양에 적합함

---

# Resource

## 공식 문서

- [앱 리소스 개요](https://developer.android.com/guide/topics/resources/available-resources?hl=ko&_gl=1*1ytbaf*_up*MQ..*_ga*OTg1NDgwMzIxLjE3MTIyMTUyMzM.*_ga_6HH9YJMN9M*MTcxMjIxNTIzMy4xLjAuMTcxMjIxNTIzMy4wLjAuMA..)
- [리소스 유형 개요](https://developer.android.com/guide/topics/resources/available-resources?hl=ko&_gl=1*1ytbaf*_up*MQ..*_ga*OTg1NDgwMzIxLjE3MTIyMTUyMzM.*_ga_6HH9YJMN9M*MTcxMjIxNTIzMy4xLjAuMTcxMjIxNTIzMy4wLjAuMA..)

## Resource란?

- 코드에서 사용하는 추가 파일과 정적인 콘텐츠(비트맵, 사용자 인터페이스 문자열, 애니메이션 지침 등)
- 독립적인 유지가 가능하도록 코드로부터 분리해야 함
- Android에서 자동으로 생성되는 R 클래스에서 생성된 리소스 ID로 접근 가능, 대부분 ID는 파일 이름과 동일함

## @StringRes

- 함수의 매개변수가 문자열 리소스 참조임을 알려주는 주석
- 문자열 리소스를 사용하는 방법으로, 유형 안전성을 갖추고 있음
- 다른 개발자나 Android Studio의 린트 등 코드 검사 도구에 유용함
- 이외에도 @DrawableRes 등 다른 유형의 리소스를 위한 주석이 있음

## dimens.xml

- 크기 값을 저장하는 리소스 파일
- `dimensionResource(id = R.dimen.padding_small)`과 같이 접근해서 사용

```xml
<resources>
   <dimen name="padding_small">8dp</dimen>
   <dimen name="padding_medium">16dp</dimen>
   <dimen name="image_size">64dp</dimen>
</resources>
```

---

# 반응형 앱 디자인

- 에뮬레이터 생성 시 Resizable 기기를 선택하면 크기를 변경해서 테스트할 수 있음

## 중단점(Breakpoint)

- Material 디자인의 중단점 종류
  | | width | height |
  | - | - | - |
  | Compact(Phone) | ~ 599dp | ~ 479dp |
  | Medium(Foldable, Small tablet) | 600dp ~ 839dp | 480dp ~ 899dp |
  | Expanded(Large tablet) | 840dp ~ | 900dp ~ |

- Compose의 WindowSizeClass API를 사용한 중단점 구현

  1. 종속성 추가

  ```kotlin
  "androidx.compose.material3:material3-window-size-class:$material3_version"
  ```

  2. MainActivity에서 windowSize를 app 생성 시 전달

  ```kotlin
  // 버전 문제로 오류 발생 시 onCreate() 메서드에 주석 추가
  ReplyTheme {
      val windowSize = calculateWindowSizeClass(this)
      ReplyApp(windowSize = windowSize.widthSizeClass)
  }
  ```

  3. app에서 windowSize를 기준으로 화면 표시

  ```kotlin
  @Composable
  fun ReplyApp(
      windowSize: WindowWidthSizeClass,
      modifier: Modifier = Modifier
  ) {
      when (windowSize) {
          WindowWidthSizeClass.Compact -> {}
          WindowWidthSizeClass.Medium -> {}
          WindowWidthSizeClass.Expanded -> {}
          else -> {}
      }
  }
  ```

  4. preview의 경우 값 지정

  ```kotlin
  @Preview(showBackground = true, widthDp = 1000)
  @Composable
  fun ReplyAppExpandedPreview() {
      ReplyTheme {
          Surface {
              ReplyApp(windowSize = WindowWidthSizeClass.Expanded)
          }
      }
  }
  ```

## 중단점에 따른 Navigation Menu 구현

- Compact: [Bottom Navigation Bar](https://developer.android.com/static/codelabs/basic-android-kotlin-compose-adaptive-navigation-for-large-screens/img/8bc57664775332fe.png?hl=ko)
- Medium: [Navigation Rail](https://developer.android.com/static/codelabs/basic-android-kotlin-compose-adaptive-navigation-for-large-screens/img/fdb4e7b5bd686ba8.png?hl=ko)
- Expanded: [Persistent Navigation Drawer](https://developer.android.com/static/codelabs/basic-android-kotlin-compose-adaptive-navigation-for-large-screens/img/d5c215bae68b12f9.png?hl=ko)

1. Navigation Menu 종류를 나타내는 enum 클래스 생성

```kotlin
enum class ReplyNavigationType {
    BOTTOM_NAVIGATION, NAVIGATION_RAIL, PERMANENT_NAVIGATION_DRAWER
}

enum class ReplyContentType {
    LIST_ONLY, LIST_AND_DETAIL
}
```

2. app에 navigationType 변수를 만들고 windowSize를 기준으로 값 할당

```kotlin
val navigationType: ReplyNavigationType
val contentType: ReplyContentType

when (windowSize) {
    WindowWidthSizeClass.Compact -> {
        navigationType = ReplyNavigationType.BOTTOM_NAVIGATION
        contentType = ReplyContentType.LIST_ONLY
    }
    WindowWidthSizeClass.Medium -> {
        navigationType = ReplyNavigationType.NAVIGATION_RAIL
        contentType = ReplyContentType.LIST_ONLY
    }
    WindowWidthSizeClass.Expanded -> {
        navigationType = ReplyNavigationType.PERMANENT_NAVIGATION_DRAWER
        contentType = ReplyContentType.LIST_AND_DETAIL
    }
    else -> {
        navigationType = ReplyNavigationType.BOTTOM_NAVIGATION
        contentType = ReplyContentType.LIST_ONLY
    }
}
```

3. home screen에서 navigationType을 전달받아 PermanentNavigationDrawer 표시 여부 확인

```kotlin
// PermanentNavigationDrawer API를 사용하기 위해 필요한 주석
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ReplyHomeScreen(
    navigationType: ReplyNavigationType,
    contentType: ReplyContentType,
    replyUiState: ReplyUiState,
    onTabPressed: (MailboxType) -> Unit,
    onEmailCardPressed: (Email) -> Unit,
    onDetailScreenBackPressed: () -> Unit,
    modifier: Modifier = Modifier
) {
    if (navigationType == ReplyNavigationType.PERMANENT_NAVIGATION_DRAWER) {
        PermanentNavigationDrawer(
            drawerContent = {
                NavigationDrawerContent(
                    selectedDestination = replyUiState.currentMailbox,
                    onTabPressed = onTabPressed,
                    navigationItemContentList = navigationItemContentList
                )
            }
        ) {
            ReplyAppContent(
                navigationType: ReplyNavigationType,
                contentType: ReplyContentType,
                replyUiState = replyUiState,
                onTabPressed = onTabPressed,
                onEmailCardPressed = onEmailCardPressed,
                navigationItemContentList = navigationItemContentList,
                modifier = modifier
            )
        }
    } else {
        if (replyUiState.isShowingHomepage) {
            ReplyAppContent(
                navigationType: ReplyNavigationType,
                contentType: ReplyContentType,
                replyUiState = replyUiState,
                onTabPressed = onTabPressed,
                onEmailCardPressed = onEmailCardPressed,
                navigationItemContentList = navigationItemContentList,
                modifier = modifier
            )
        } else {
            ReplyDetailsScreen(
                replyUiState = replyUiState,
                onBackPressed = onDetailScreenBackPressed,
                modifier = modifier
            )
        }
    }
}
```

4. app content에서 navigationType을 전달받아 NavigationRail 표시 여부 확인

```kotlin
@Composable
private fun ReplyAppContent(
    navigationType: ReplyNavigationType,
    contentType: ReplyContentType,
    replyUiState: ReplyUiState,
    onTabPressed: ((MailboxType) -> Unit) = {},
    onEmailCardPressed: (Email) -> Unit = {},
    navigationItemContentList: List<NavigationItemContent>,
    modifier: Modifier = Modifier
) {
    Row(modifier = modifier.fillMaxSize()) {
        AnimatedVisibility(visible = navigationType == ReplyNavigationType.NAVIGATION_RAIL) {
            ReplyNavigationRail(
                currentTab = replyUiState.currentMailbox,
                onTabPressed = onTabPressed,
                navigationItemContentList = navigationItemContentList
            )
        }

        Column(
            modifier = Modifier
                .fillMaxSize()
                .background(MaterialTheme.colorScheme.inverseOnSurface)
        ) {
            if (contentType == ReplyContentType.LIST_AND_DETAIL) {
                ReplyListAndDetailContent(
                    replyUiState = replyUiState,
                    onEmailCardPressed = onEmailCardPressed,
                    modifier = Modifier.weight(1f)
                )
            } else {
                ReplyListOnlyContent(
                    replyUiState = replyUiState,
                    onEmailCardPressed = onEmailCardPressed,
                    modifier = Modifier.weight(1f)
                        .padding(
                            horizontal = dimensionResource(R.dimen.email_list_only_horizontal_padding)
                        )
                )
            }
            AnimatedVisibility(visible = navigationType == ReplyNavigationType.BOTTOM_NAVIGATION) {
                ReplyBottomNavigationBar(
                    currentTab = replyUiState.currentMailbox,
                    onTabPressed = onTabPressed,
                    navigationItemContentList = navigationItemContentList
                )
            }
        }
    }
}
```
