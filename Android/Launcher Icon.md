# 앱 아이콘

- 홈 화면, 모든 앱 화면, 설정 등 여러 위치에서 표시되는 아이콘
- `app/src/main/res/mipmap` 폴더에 런처 아이콘 에셋이 위치함
- Android Studio에서 Image Asset Studio를 사용하면 다양한 버전의 런처 아이콘을 생성할 수 있음
- 밀도(dpi)에 따라 다른 리소스를 표시함
  - [밀도 한정자 목록](https://developer.android.com/training/multiscreen/screendensities?hl=ko&_gl=1*164qba7*_up*MQ..*_ga*MjQ4MDcxODE5LjE3MTI1NTQ1MDU.*_ga_6HH9YJMN9M*MTcxMjU1NDUwNS4xLjAuMTcxMjU1NDUwNS4wLjAuMA..#TaskProvideAltBmp)
- Android는 기기의 화면 밀도를 기준으로 가장 가까운 큰 밀도 버킷의 리소스를 선택한 후 축소해서 사용함
- 흐릿한 이미지를 방지하려면 각 밀도마다 이미지를 지원해야 함

## 적응형 아이콘

- 기기 제조업체에서 다양한 모양으로 표시하기 때문에, 일관된 사용자 환경을 제공하고자 API 26부터 적응형 아이콘을 지원함
- `mipmap-anydpi-v26`과 같이 `-v26` 리소스 한정자가 있는 디렉터리에 선언해야 함
- 유연성과 시각적 효과를 더할 수 있음
- 앱 아이콘이 포어그라운드 레이어와 백그라운드 레이어로 구성됨

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
      <background android:drawable="@drawable/ic_launcher_background"/>
      <foreground android:drawable="@drawable/ic_launcher_foreground"/>
      <monochrome android:drawable="@drawable/ic_launcher_foreground" />
  </adaptive-icon>
  ```

  - 크기가 108dpi \* 108dpi여야 하는 등 특정 요구사항이 있음
  - 기기 제조업체의 마스크 모양에 따라 아이콘의 가장자리가 잘릴 수 있으므로, 주요 정보는 안전 영역(포어그라운드 레이어 중심에 있는 직경 66dpi의 원)에 배치해야 함

- 변경 방법
  1. 기존의 아이콘 xml 및 mipmap 폴더 삭제
  2. Android Studio에서 Resource Manager 탭 클릭
  3. `+` > `Image Asset` 클릭
  4. Foreground Layer 탭 선택 후 Source Asset의 Path에 foreground로 사용할 xml 파일 지정
  5. Background도 마찬가지로 지정
  6. 콘텐츠 영역 크기는 Scaling의 Resize 슬라이더바를 조정해서 변경
  7. 확인을 누르면 xml과 webp 파일이 생성됨(Android 8.0 미만의 기기에서 지원하기 위해 비트맵 이미지도 생성됨)
