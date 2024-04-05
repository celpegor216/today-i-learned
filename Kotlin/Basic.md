# 변수

## var보다 val 사용을 권장하는 이유

- val을 사용하면 값이 항상 유지되어 예상하지 않는 경우 변수가 프로그램에서 업데이트되지 않음, 코드가 더 안전해짐
- var는 값이 변경될 것으로 예상되는 경우에만 사용

---

# 조건문

## when 문 변형

- 매개변수를 사용하지 않고, if-else 체인 대신 사용

  ```kotlin
  when {
      x.isOdd() -> print("x is odd")
      y.isEven() -> print("y is even")
      else -> print("x+y is odd")
  }
  ```

- `is` 키워드로 데이터 유형 확인

  ```kotlin
  fun Request.getBody() =
      when (val response = executeRequest()) {
          is Success -> response.body
          is HttpError -> throw HttpException(response.status)
      }
  ```

- 표현식으로 사용할 경우, 값을 반환해야 하기 때문에 else가 필요함

---

# 함수

## 매개변수와 인수의 차이

- 매개변수: 함수를 정의할 때, 함수 호출 시 전달할 변수(함수가 접근할 수 있는 변수)
- 인수: 함수를 호출할 때, 매개변수를 위한 값(전달하는 값)
- 다른 언어와 달리 Kotlin에서는 매개변수에 다른 값을 할당할 수 없음

## 함수 서명

- 입력(매개변수)이 있는 함수 이름
- 반환 유형 앞의 모든 내용으로 구성
- `fun birthdayGreeting(name: String, age: Int)`

## 어노테이션

- 코드에 추가 정보를 첨부하는 방법
- 컴파일러가 읽는 주석
- `@` 문자를 접두사로 추가하여 사용
- 매개변수를 받을 수 있음

## 함수 참조 연산자 `::`

- fun 키워드로 정의된 함수를 변수에 할당할 때 사용

  ```kotlin
  fun main() {
      val trickFunction = ::trick
  }

  fun trick() {
      println("No treats!")
  }
  ```

## repeat()

- 함수로 for 반복을 표현하는 간결한 방법, 고차 함수(함수를 반환하거나 함수를 인수로 취하는 함수)의 일종

  ```kotlin
  fun main() {
      repeat(4) {
        trick()
      }
  }

  fun trick() {
      println("No treats!")
  }
  ```

---

# 클래스

## 속성의 getter 함수와 setter 함수

- 컴파일러가 내부적으로 생성하는 기본 getter, setter

```kotlin
var speakerVolume = 2
    get() = field
    set(value) {
        field = value
    }
```

- val로 선언한 경우 setter가 없음
- 반환 값을 항상 대문자로 변환해서 반환해야 하는 등 값 반환 시 처리가 필요한 경우 getter 함수에서 처리
- 속성 값 변경 시 범위를 확인해야 하는 등 값 변경 시 처리가 필요한 경우 setter 함수에서 처리(공개 상태만 설정하고 작업이 없을 경우 `protected set`처럼 써도 됨)
- 기본적으로 속성에 내부적으로 정의된 클래스 변수인 지원 필드를 사용하여 메모리에 값을 보유함
- getter는 지원 필드의 값을 읽고, setter는 지원 필드의 값을 변경함(위 예제의 `field`)
- 위 예제에서 `field = value`가 아닌 `speakerVolume = value`로 지정할 경우, setter가 반복적으로 호출되어 무한 루프가 발생하므로 주의

## `open` 키워드

- 다른 클래스가 해당 클래스/함수를 확장할 수 있음

## 공개 상태 수정자

- `public`: 기본값, 모든 위치에서 접근 가능
- `internal`: 동일한 모듈에서 접근 가능
- `protected`: 하위 클래스에서 접근 가능
- `private`: 동일한 클래스에서 접근 가능
- getter, setter에도 설정할 수 있음(단, getter의 수정자와 속성의 수정자가 일치하지 않으면 컴파일 에러 발생)
- 클래스의 생성자에 설정할 경우, `class 클래스명 수정자 constructor (매개변수)`와 같이 작성
- 속성과 메서드의 공개 상태를 엄격하게 유지하는 것이 이상적이므로, 최대한 자주 private를 사용할 것

## 속성 위임

- `var 변수명 by 위임객체`
- 필드, getter, setter를 사용해 값을 관리하는 대신 위임을 통해 관리
- setter 함수에 범위 확인 코드를 재사용할 수 있음
- 위임 클래스

  ```kotlin
  class RangeRegulator(
      initialValue: Int,
      private val minValue: Int,
      private val maxValue: Int
  ) : ReadWriteProperty<Any?, Int> {

      var fieldData = initialValue

      override fun getValue(thisRef: Any?, property: KProperty<*>): Int {
          return fieldData
      }

      override fun setValue(thisRef: Any?, property: KProperty<*>, value: Int) {
          if (value in minValue..maxValue) {
              fieldData = value
          }
      }
  }


  class SmartTvDevice(deviceName: String, deviceCategory: String) :
    SmartDevice(name = deviceName, category = deviceCategory) {

    override val deviceType = "Smart TV"

    private var speakerVolume by RangeRegulator(initialValue = 2, minValue = 0, maxValue = 100)

    private var channelNumber by RangeRegulator(initialValue = 1, minValue = 0, maxValue = 200)
  }
  ```

  - var 유형의 경우 ReadWriteProperty 인터페이스를 구현하고, val 유형의 경우 ReadOnlyProperty 인터페이스를 구현
  - `KProperty`: 선언된 속성을 나타내는 인터페이스, 위임된 속성의 메타데이터에 접근 가능

---

# Kotlin Style Guide

코드의 가독성을 높이고, 공동 작업을 위해 코드 작성 방식을 통일

- 변수명은 카멜 표기법을 따르는, 보유한 데이터를 설명하는 이름
- 변수 선언 시 데이터 유형 지정하는 콜론 뒤 1칸 space
- 연산자 사용 시 앞 뒤로 1칸 씩 space
- 함수명은 카멜 표기법을 따르는 동사/동사구 (e.g. `displayErrorMessage`)
- 여는 중괄호 앞 1칸 space
- 함수 본문은 4칸 들여쓰기(Tab 아님 주의)
- 닫는 중괄호는 본문 마지막 코드 줄에서 한 줄 내려서 홀로 존재
- 가로 스크롤을 하지 않고도 코드를 쉽게 읽기 위해 영문 기준 줄 당 100자 제한 권장
- 구체적인 스타일 가이드는 [공식 문서](https://developer.android.com/kotlin/style-guide?hl=ko) 참고
