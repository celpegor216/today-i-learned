# Coroutine

- `LaunchedEffect()`
  - Composable에서 suspend 함수 호출을 위해 Coroutine 생성
  - 컴포지션이 유지되는 동안 함수를 실행함
  - key 인수의 인스턴스가 교체되면 기본 Coroutine이 취소되고 다시 실행됨
    ```kotlin
    LaunchedEffect(playerOne, playerTwo) {
        coroutineScope {
          launch { playerOne.run() }
          launch { playerTwo.run() }
          raceInProgress = false
        }
    }
    ```
