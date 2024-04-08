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

## enum class

- 가능한 값 집합이 제한되어 있는 유형을 생성
- enum 상수: 가능한 값, 이름은 모두 대문자로 표기 `.`으로 참조

```kotlin
enum class Difficulty {
    EASY, MEDIUM, HARD
}

class Question<T>(
    val questionText: String,
    val answer: T,
    val difficulty: Difficulty
)

val question = Question<String>("Quoth the raven ___", "nevermore", Difficulty.MEDIUM)
```

## data class

- 메서드 없이 데이터만 포함하는 클래스
- equals(), hashCode(), toString(), componentN(), copy() 등 메서드를 자동으로 구현해줌
- 생성자에 매개변수가 하나 이상 있어야 함
- abstrack, open, sealed, inner일 수 없음

```kotlin
data class Question<T>(
    val questionText: String,
    val answer: T,
    val difficulty: Difficulty
)
```

## singleton class

- 인스턴스를 하나만 가질 수 있는 클래스
- `object`로 생성
- 개발자가 인스턴스를 직접 만들 수 없기 때문에 생성자가 없음
- `companion object`: 클래스 내의 싱글톤 객체

---

# 인터페이스

- 여러 클래스에 동일하게 포함해야 하는 메서드 또는 속성을 지정한 일종의 계약
- 코드베이스가 커짐에 따라 슈퍼클래스에서 상속받지 않고도 동일한 인터페이스를 준수하는 클래스를 추가하여 코드를 재사용할 수 있음
- 인터페이스를 준수하는 클래스는 인터페이스를 확장/구현한다고 함
  - `클래스명: 인터페이스명` 형태로 확장
  - 인터페이스를 확장하는 클래스는 모든 속성과 메서드를 구현해야 함(override)
