# 2. 변수의 스코프를 최소화하라

## 1. 변수의 스코프
- 추천
    - 프로퍼티보다는 지역 변수를 사용할 것
    - 최대한 좁은 스코프를 갖게 변수를 사용할 것
        - ex. 반복문 내부에서만 변수가 사용된다면 반복문 내부에 변수 작성
- 이유
    - 프로그램 추적 관리 용이
        - mutable 프로퍼티가 좁은 스코프에 걸쳐 있을수록 변경을 추적하는 것이 용이
    - 스코프가 너무 넓으면 협업 시 다른 개발자에 의해 변수 잘못된 사용 가능성 존재
- *예제: 변수 초기화*

```kotlin
// Bad Case
val user: User
if (hasValue) {
    user = getValue()
} else {
    user = User()
}

// Good Case
val user: User = if(hasValue) {
    getValue()
} else {
    User()
}
```

- *예제: 여러 프로퍼티를 한번에 설정*
```kotlin
// Bad Case
fun updateWeather(degrees: Int) {
    val description: String
    val color: Int
    if (degrees < 5) {
        description = "cold"
        color = Color.BLUE8
    } else if (degrees < 23) {
        description = "mild"
        color = Color.YELLOW
    } else {
        description = "hot"
        color = Color.RED
    }
    ...
}

// Good Case
fun updateWeather(degrees: Int) {
    val (description, color) = when {
        degrees < 5 -> "cold" to Color.BLUE
        degrees < 23 -> "mild" to Color.YELLOW
        else -> "hot" to Color.RED
    }
    ...
}
```

## 2. 캡처링
- 시퀀스 빌더를 사용해서 소수를 구하는 알고리즘(에라토스테네스의 체) 구현
    - 로직
        1. 2부터 시작하는 숫자 리스트(=(2..100)등) 생성
        2. 첫 번째 요소 선택, 이는 소수
        3. 남아 있는 숫자 중에서 2번에서 선택한 소수로 나눌 수 있는 모든 숫자 제거
    - 구현
        ```kotlin
        var numbers = (2..100).toList()
        val primes = mutableListOf<Int>()
        while (numbers.isNotEmpty()) {
            val prime = numbers.first()
            primes.add(prime)
            numbers = numbers.filter { it % prime != 0 }
        }
        print(primes)
        // [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97]
        ```
    - 구현(시퀀스 활용)
        ```kotlin
        val primes: Sequence<Int> = sequence {
            var numbers = generateSequence(2) { it + 1 }
            while (true) {
                val prime = numbers.first()
                yield(prime)
                numbers = numbers.drop(1)
                    .filter { it % prime != 0 }
            }
        }

        print(primes.take(10).toList())
        // [2, 3, 5, 7, 11, 13, 17, 19, 23, 29] 
        ```
    - 잘못된 구현
        ```kotlin
        val primes: Sequence<Int> = sequence {
                var numbers = generateSequence(2) { it + 1 }
                var prime: Int
                while (true) {
                    prime = numbers.first()
                    yield(prime)
                    numbers = numbers.drop(1)
                        .filter { it % prime != 0 }
                }
            }

            print(primes.take(10).toList())
            // [2, 3, 5, 6, 7, 8, 9, 10, 11, 12] 
        ```
        - prime 변수 캡쳐됨
        - 필터링 시 최종 캡쳐된 값으로만 필터링이 진행되어 버림
        