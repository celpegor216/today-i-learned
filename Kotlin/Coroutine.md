# Coroutine

- co(기본 이벤트 루프를 공유해 작업들이 함께 동작함, 즉 한 작업이 정지해도 다른 작업이 실행될 수 있음) + routine()
- 서버에서 데이터를 가져오는 동시에 사용자의 입력에 응답하고 UI를 업데이트하는 등, 여러 작업을 동시에 작업하기 위해 사용하는 기술
- 한 작업이 완료되지 않아도 다음 작업을 시작하는 비동기 코드를 쉽게 작성할 수 있음
- 구조화된 동시 실행(Structured concurrency)
  - 코드가 기본적으로 순차적이고, 동시 실행을 명시적으로 요청(`launch()` 사용 등)하지 않는 한 기본 이벤트 루프와 협력함
  - 함수를 호출하면, 작업이 성공적으로 완료되었는지 여부와 상관없이 제어 흐름이 함수에서 반환될 때 작업이 종료됨

## Coroutine 개념

- CoroutineScope
  - Coroutine이 실행되는 범위
  - Coroutine을 관리하여 손실이나 리소스 낭비를 방지함
  - 수명 주기와 연결되어 범위 내의 Coroutine이 유지되는 기간에 경계를 설정함
  - 범위가 취소되거나 실패하면 범위 내의 Job과 하위 Job도 취소됨
  - `runBlocking()`, `coroutineScope {}`로 생성하거나 Android에 특정 항목의 수명 주기에 따르도록 미리 정의된 `lifecycleScope`(Activity), `viewModelScope`(ViewModel)을 사용
  - `launch()`, `async()`: CoroutineScope의 확장 함수로, 범위 내에 CoroutineContext를 상속받는 새 Coroutine을 실행시킴
- CoroutineContext
  - Coroutine이 실행되는 Context에 대한 정보, Map 유형
  - name, job, dispatcher, handler(예외 처리) 등 필드 포함
  - 각 필드는 `+` 연산자로 추가할 수 있음
    ```kotlin
    Job() + Dispatchers.Main + exceptionHandler
    ```
  - Coroutine 내에서 새 Coroutine을 실행하는 경우 상위 Coroutine의 CoroutineContext를 상속받는데 이를 재정의할 수 있음
    ```kotlin
    scope.launch(Dispatchers.Default) {}
    ```
  - `withContext()`: CoroutineContext를 변경할 수 있는 suspend 함수
    ```kotlin
    runBlocking {
        launch {
            withContext(Dispatchers.Default) {
                delay(1000)
                println("10 results found.")
            }
        }
        println("Loading...")
    }
    ```
- Job
  - launch()로 Coroutine을 실행했을 때 반환되는 값
  - 계층 구조를 이루고 있으며, 취소와 달리 예외는 상향 전파됨
  - Coroutine에 대한 Handle 또는 참조를 보유하여 수명 주기를 관리할 수 있음
    ```kotlin
    val job = launch { }
    job.cancle()
    ```
- Dispatcher
  - Coroutine이 실행될 스레드를 결정함
  - 종류
    - Main: 앱이 실행될 때 프로세스와 함께 생성되는 단일 실행 스레드, UI 업데이트 및 사용자 상호작용을 처리함
    - IO: 디스크 또는 네트워크 I/O 작업을 처리함
    - Default: CoroutineContext에 Dispatcher가 지정되지 않은 상태에서 launch() 또는 async()를 호출할 때 사용되는 기본값, 비트맵 이미지 파일 처리같이 계산이 많은 작업을 처리함
  - Main 스레드는 UI를 담당하므로 빠른 실행을 위해 비차단 상태로 유지해야 함, 따라서 네트워크 작업과 같이 장기 실행 작업은 다른 스레드를 생성해서 처리하는 것이 좋음
    - 차단: 일반 함수는 작업이 완료될 때까지 호출 스레드를 차단해서 다른 작업을 실행할 수 없음
    - 비차단: 특정 조건이 충족될 때까지 호출 스레드를 생성해서 다른 작업을 실행할 수 있음

## 동기 vs 비동기

- 동기

  ```kotlin
  fun main() {
      // 다시 시작할 준비가 되면 각 작업을 중단된 지점에서부터 진행하여 여러 작업을 한 번에 처리할 수 있는 이벤트 루프 실행
      // Android에서는 이벤트 루프를 제공하기 때문에 학습, 테스트 용도로 사용함
      runBlocking {
          println("Weather forecast")
          printForecast()
      }
  }

  suspend fun printForecast() {
      // 정지 함수, 정지되었다가 나중에 다시 시작할 수 있음
      // Coroutine이나 suspend 함수에서만 호출될 수 있음
      delay(1000)
      println("Sunny")
  }
  ```

  - 한 번에 하나의 작업만 진행되고, 이전 작업이 완료되어야 다음 작업이 시작됨(순차 선형 경로)

- 비동기

  ```kotlin
  fun main() {
      runBlocking {
          println("Weather forecast")
          launch {
              printForecast()
          }
          println(getWeatherReport())
          println("Have a good day!")
      }
  }

  suspend fun printForecast() {
      delay(1000)
      println("Sunny")
  }

  suspend fun getWeatherReport() = coroutineScope {
      val forecast = async { getForecast() }
      val temperature = async { getTemperature() }
      "${forecast.await()} ${temperature.await()}"
  }

  suspend fun getForecast(): String {
      delay(1000)
      return "Sunny"
  }

  suspend fun getTemperature(): String {
      delay(1000)
      return "30\u00b0C"
  }
  ```

  - 동시에 작업을 실행하기 위해 여러 Coroutine이 실행될 수 있도록 새 Coroutine 실행
  - 작업이 완료되기 전에 호출을 반환해서 다음 Coroutine을 시작할 수 있음
  - `async()`
    - Coroutine이 완료되는 시점이나 반환 값이 필요한 경우 사용
    - `launch()`와 달리 Deferred 객체를 반환함, await()를 사용해서 반환 값에 접근할 수 있음
  - `coroutineScope{}`
    - 작업의 로컬 범위를 만들고, 이 범위 내에서 실행된 Coroutine은 그룹화됨
    - 내부적으로는 동시에 작업하고 있지만, 내부의 모든 작업이 완료되기 전까지는 couroutineScope가 반환되지 않아 동기 작업처럼 보임

## 예외 처리

- 하위 Coroutine에서 예외가 발생한 경우, 상위 Coroutine으로 예외가 전파되어 상위 Coroutine도 취소되고 그 Coroutine의 모든 하위 Coroutine도 취소됨
- try-catch 문을 사용해 처리할 수 있음

  - launch()의 경우 예외가 즉시 발생하므로 예외가 발생할 것으로 예상되는 코드를 try-catch로 처리
  - async()의 경우, await()는 소비자이므로 생산자인 async()에서 예외를 처리

    ```kotlin
    suspend fun getWeatherReport() = coroutineScope {
        val forecast = async { getForecast() }
        val temperature = async {
            try {
                getTemperature()
            } catch (e: AssertionError) {
                println("Caught exception $e")
                "{ No temperature found }"
            }
        }

        "${forecast.await()} ${temperature.await()}"
    }
    ```

## 취소

- 하위 Coroutine을 취소하더라도 상위 Coroutine은 취소되지 않음

  ```kotlin
  suspend fun getWeatherReport() = coroutineScope {
      val forecast = async { getForecast() }
      val temperature = async { getTemperature() }

      delay(200)
      temperature.cancel()

      "${forecast.await()}"
  }
  ```
