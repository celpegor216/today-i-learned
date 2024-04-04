# User Interface

- 화면에 표시되는 텍스트, 이미지, 버튼, 기타 여러 유형의 요소 및 화면에 표시되는 방식
- 앱이 콘텐츠를 사용자에게 표시하는 방식이자 사용자가 앱과 상호작용하는 방식

# Jetpack Compose

- Android UI를 빌드하기 위해 사용하는 최신 툴킷
- 적은 양의 코드, 강력한 도구 및 직관적인 Kotlin 기능으로 Android에서 UI 개발을 간소화하고 가속
- 데이터를 받아서 UI 요소를 설명하는 Composable 함수 집합을 정의하여 UI를 빌드

# Composable 함수

- Compose에서 UI의 기본 빌드 블록
- UI의 구성 과정에 집중하기보다는 앱의 모양을 설명하여 앱의 UI를 프로그래매틱 방식으로 정의할 수 있음
- 입력은 받을 수 있고, 반환은 없음
- 함수명은 파스칼 표기법에 따름(첫 글자 대문자 필수), 명사 혹은 형용사 + 명사 형태([공식 문서](https://github.com/androidx/androidx/blob/androidx-main/compose/docs/compose-api-guidelines.md#naming-unit-composable-functions-as-entities))
- 함수 앞에 `@Composable` 주석 추가 필수

# Image

- `painterResource()`로 드로어블 이미지 리소스를 로드해서 painter의 인수로 전달
  ```
  Image(
    painter = painterResource(R.drawable.androidparty)
  )
  ```
- `contentDescription`: 앱 접근성을 개선하기 위한 콘텐츠의 설명, 장식 목적 등 설명이 필요하지 않는 경우 null로 설정
- `contentScale`: 배율 유형을 설정해서 이미지 크기 조절
