# Activity Lifecycle

- Activity의 생성부터 소멸(운영체제가 메모리를 회수)까지 거쳐가는 여러 상태
- Activity 간 이동하거나, 앱 안팎으로 이동할 때 상태가 변경됨
- 상태가 변경될 때, onCreate와 같은 수명 주기 콜백 메서드를 구현해 작업을 수행함
- 수명 주기가 변경될 때 올바르게 처리하지 않을 경우 버그가 발생하거나 리소스를 너무 많이 사용할 수 있음

## Activity Lifecycle 콜백 메서드

![이미지](https://developer.android.com/static/codelabs/basic-android-kotlin-compose-activity-lifecycle/img/468988518c270b38_960.png?hl=ko)

- onCreate(): Activity가 초기화된 직후, 운영체제가 메모리에 앱을 생성할 때 한 번만 호출됨
- onStart(): onCreate() 직후, 앱이 화면에 표시될 때 호출됨, 사용자와 아직 상호작용할 수 없음
- onResume(): onStart() 직후, 앱을 foreground로 가져와 포커스를 받을 때 호출됨, 사용자와 상호작용할 수 있음
- onPause(): 앱이 포커스를 잃었을 때 호출됨, 화면에는 표시되므로 UI를 계속 업데이트하여 앱이 멈춘 것처럼 보이지 않게 하는 것이 좋음(ex. 공유 버튼을 눌러서 관련 창이 화면의 일부에 표시되는 경우)
- onStop(): 앱이 background로 이동해 화면에 표시되지 않을 때 호출됨
- onDestroy(): `finish()`가 호출되거나, Compose 변경이 발생해서 Activity가 완전히 종료되고 다시 빌드되거나, 앱이 종료되거나 닫힐 때 한 번만 호출됨, 메모리와 같은 메소스를 회수함(사용하고 있는 객체를 무효화하거나 닫거나 소멸)
- 콜백 메스드를 재정의할 때 Activity에서 슈퍼클래스 구현을 즉시 호출해야 함(ex. `super.onCreate()`)

## Composable Lifecycle

- Composition: Composable을 실행하여 빌드함
- Recomposition: 앱의 상태가 변경되면 예약되어, 상태가 변경된 Composable을 다시 실행하여 UI를 업데이트함
- [State](Jetpack%20Compose.md/#Compose의-상태): Recomposition을 추적하고 트리거할 때 사용되는 상태
