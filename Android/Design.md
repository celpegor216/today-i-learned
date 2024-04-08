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
