# Material Design

- 개발자가 Android 및 기타 모바일과 웹 플랫폼을 위한 고품질 디지털 환경을 빌드할 수 있도록 Google 디자이너와 개발자들이 빌드하고 지원하는 디자인 시스템
- 읽기 쉽고 매력적이며 일관된 방식으로 앱 UI를 빌드하는 방법에 관한 가이드라인을 제시

---

# Theme.kt

## 색 구성표

- primary: UI에서 주요 구성 요소에 사용
- secondary: UI에서 눈에 덜 띄는 구성 요소에 사용
- tertiary: 기본 색상과 보조 색상의 균형을 맞추거나, 입력란과 같은 특정 요소로 관심을 유도할 때 사용하는 대비 강조로 사용
- on~ : 팔레트의 색상 위에 사용(주로 텍스트, 아이콘, 획)
- [Material 테마 빌더](https://m3.material.io/theme-builder#/custom)에서 색 구성표를 만들어서 `Color.kt`와 `Theme.kt` 파일로 다운로드 받을 수 있음

## 어두운 테마

- 더 어둡고 부드러운 색상을 사용
- 전력 사용을 줄일 수 있음
- 시력이 낮거나 밝은 빛에 민감산 사용자의 가시성을 개선함
- 조명이 어두운 환경에서 편하게 기기를 사용할 수 있음
- 앱에서 어두운 테마를 강제 적용하면 시스템이 구현하는데, 개발자가 직접 구현해서 앱 테마를 원하는 대로 관리하면 더 나은 사용자 환경을 제공할 수 있음
- [접근성 대비 표준](https://webaim.org/resources/contrastchecker/)을 충족해서 색상을 선정해야 함
- 어두운 테마를 미리보기에 적용하려면 `~Theme(darkTheme = true)`와 같이 작성

## 동적 색상

- 사용자가 기기에서 지정한 색상이 있다면 해당 색상을 반영하는 것
- Android 12부터 적용됨
- 동적 테마 설정을 지원하려면 Theme.kt의 ~Theme() 컴포저블에 `dynamicColor` 매개변수를 true로 설정

---

# Shape.kt

- Compose 구성요소의 도형을 정의
- 구성요소는 small, medium, large 세 가지 유형이 있음

## 이미지를 원형으로 만들기

1. Shape.kt에 small 지정

   ```kotlin
   val Shapes = Shapes(
     small = RoundedCornerShape(50.dp),
   )
   ```

2. Image의 modifier에 clip 속성을 추가하고, 인자로 MaterialTheme.shapes.small 전달

   ```kotlin
   Image(
       modifier = modifier
           .size(dimensionResource(id = R.dimen.image_size))
           .padding(dimensionResource(id = R.dimen.padding_small))
           .clip(MaterialTheme.shapes.small),
   )
   ```

3. 이미지가 도형에 맞게 잘리도록 contentScale 값을 ContentScale.Crop으로 지정

---

# Type.kt

## 서체 스케일

- 유연하면서도 일관된 스타일을 위해 앱 전반에서 사용할 수 있는 글꼴 스타일 모음
- Material Design의 서체 스케일 종류
  - display: 가장 큰 텍스트, 짧고 중요한 텍스트나 숫자, 대형 화면에 적합
  - headline: 텍스트의 주요 구절이나 콘텐츠의 중요한 부분 등 짧은 강조 문구에 적합, 소형 화면에 적합
  - title: 상대적으로 짧게 유지되는 중간 강조 텍스트에 적합
  - body: 긴 텍스트 문구에 적합
  - label: 구성요소 내부 혹은 캡션과 같은 콘텐츠 본문의 작은 텍스트에 적합

## 폰트 추가 및 적용

1. `res/font` 디렉토리 생성
2. ttf 형식의 폰트 다운로드 후 파일명에 대문자가 포함되지 않도록 변경한 후 font 디렉토리에 추가
3. Type.kt의 Typography 상단에 폰트 패밀리 추가

   ```kotlin
   val Montserrat = FontFamily(
      Font(R.font.montserrat_regular),
      Font(R.font.montserrat_bold, FontWeight.Bold)
   )
   ```

4. Typography 작성

   ```kotlin
   val Typography = Typography(
       displayMedium = TextStyle(
           fontFamily = Montserrat,
           fontWeight = FontWeight.Bold,
           fontSize = 20.sp
       ),
       labelSmall = TextStyle(
           fontFamily = Montserrat,
           fontWeight = FontWeight.Bold,
           fontSize = 14.sp
       ),
       bodyLarge = TextStyle(
           fontFamily = Montserrat,
           fontWeight = FontWeight.Normal,
           fontSize = 14.sp
       )
   )
   ```

5. 텍스트 인스턴스에 적용

   ```kotlin
   Text(
       text = stringResource(dogName),
       style = MaterialTheme.typography.displayMedium,
       modifier = Modifier.padding(top = dimensionResource(id = R.dimen.padding_small))
   )
   ```

---

# Icon

- 아래의 라이브러리 종속성을 추가하면 Icons 클래스를 통해 아이콘 이미지 다운로드 없이 사용 가능

  ```kotlin
  implementation("androidx.compose.material:material-icons-extended")
  ```

  ```kotlin
  IconButton(
      onClick = onClick,
      modifier = modifier
  ) {
      Icon(
          imageVector = Icons.Filled.ExpandMore,
          contentDescription = stringResource(R.string.expand_button_content_description),
          tint = MaterialTheme.colorScheme.secondary
      )
  }
  ```
