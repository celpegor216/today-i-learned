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

## 범위 함수

- 개발자가 객체의 이름을 참조하지 않고도 객체의 속성 및 메서드에 접근할 수 있는 고차 함수
- 전달된 함수의 본문이 범위 함수가 호출되는 객체의 범위를 가져오기 때문에 범위 함수라고 함
- 객체 이름을 생략할 수 있어 코드를 더 쉽게 읽을 수 있음
- 종류
  - `.let()`: 식별자 `it`을 사용하여 람다 표현식의 객체를 참조, 길고 구체적인 객체 이름을 반복적으로 사용하지 않아도 됨
    ```kotlin
    question1.let {
      println(it.questionText)
      println(it.answer)
      println(it.difficulty)
    }
    ```
  - `.apply()`: 객체가 변수에 할당되기도 전에 객체의 메서드를 호출, 변수에 저장될 수 있도록 해당 객체의 참조를 반환함
    ```kotlin
    val quiz = Quiz().apply {
    printQuiz()
    }
    ```

## 컬렉션을 사용한 고차 함수

- `groupBy(람다 표현식)`

  - 람다 표현식의 결과를 key로 삼아 리스트를 맵으로 변환

  ```kotlin
  val groupedMenu = cookies.groupBy { it.softBaked }

  val softBakedMenu = groupedMenu[true] ?: listOf()
  val crunchyMenu = groupedMenu[false] ?: listOf()
  ```

  - 반환 값의 유형을 변환할 수도 있음
  - 두 개로 분할하기만 해야 하면 `partition()`을 사용해도 됨

- `fold(초깃값, 람다 표현식)`

  - 컬렉션에서 단일 값을 생성
  - 주로 총합을 계산하거나 평균을 구하는 등 작업에 사용
  - 람다 표현식에 매개변수로 누산기(누계, 초깃값과 데이터 유형 동일), 컬렉션 요소 두 개가 추가됨

  ```kotlin
  val totalPrice = cookies.fold(0.0) {total, cookie ->
    total + cookie.price
  }
  ```

  - reduce()와 동일하게 작동, 단 reduce()는 컬렉션의 첫 번째 요소가 초깃값
  - 숫자 유형의 sum(), sumOf()도 있음

- `sortedBy()`

  - 정렬 기준으로 사용하려는 속성을 반환하는 람다 표현식을 작성하면 자연스러운 정렬 순서에 따라 정렬됨(문자열은 알파벳순, 숫자값은 오름차순)

  ```kotlin
  val alphabeticalMenu = cookies.sortedBy {
    it.name
  }
  ```

---

# Generic

- 클래스와 같은 데이터 유형이 속성 및 메서드와 함께 사용할 수 있는 알 수 없는 자리표시자 데이터 유형
- `클래스 이름<Generic>(매개 변수)`와 같이 작성
- 일반적으로 type의 약자인 T나 다른 대문자를 사용하지만 명확한 규칙은 없음

```kotlin
class Question<T>(
    val questionText: String,
    val answer: T,
    val difficulty: String
)
```

---

# 확장

- 기존 데이터 유형을 확장하여, 해당 데이터 유형의 일부인 것처럼 점 문법으로 접근할 수 있는 속성과 메서드를 추가하는 것
- 다른 개발자에게 코드를 쉽게 보여줄 수 있음
- 속성 확장: 데이터를 저장할 수 없어 get만 가능
  ```kotlin
  val Quiz.StudentProgress.progressText: String
    get() = "${answered} of ${total} answered"
  ```
- 함수 확장
  ```kotlin
  fun Quiz.StudentProgress.printProgressBar() {
    repeat(Quiz.answered) { print("O") }
    repeat(Quiz.total - Quiz.answered) { print("X") }
    println()
    println(Quiz.progressText)
  }
  ```

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
