# 1. 가변성을 제한하라

## 1. 상태
- var 를 사용하거나 mutable 객체를 사용하여 중간 상태값을 저장할 수 있다.
- 요소가 상태를 갖는 경우 사용 방법, 히스토리에 의존하게 됨
    - 디버깅, 테스팅 어려움
    - 코드 실행의 추론이 어려움
    - 멀티스레드 프로그램에서 동기화 필요
    - 상태 변경이 발생할 때 의존하는 다른 부분에 전달시켜야 함
- 결국 다음과 같은 사항이 고려되어야 함
    - 일관성(Consistency)
    - 복잡성(Complexity)
- *예제: 동기화가 누락된 멀티스레드 예제*
    - 마지막 출력에서 매번 실행 시 결과가 동일하지 않음
    - 코루틴으로 변경한다 하더라도 결국 문제가 해결되진 않음, 동기화 필요함
    ```kotlin
    var num = 0
    for (i in 1..1000) {
        thread {
            Thread.sleep(10)
            num += 4
        }
    }
    Thread.sleep(3000)
    print(num)
    ```
- *예제: 동기화 코드가 존재하는 멀티스레드 예제*
    ```kotlin
    var lock = Any()
    var num = 0
    for (i in 1..1000) {
        thread {
            Thread.sleep(10)
            synchronized(lock) {
                num += 4
            }
        }
    }
    Thread.sleep(3000)
    print(num)
    ```
- pure functional language
    - 가변성 완벽히 제한
    - ex. Haskell

## 2. Kotlin 에서 가변성 제한
### `val`(앍가 전용 프로퍼티) 사용
- 해당 프로퍼티가 100% 변경 불가능한 것은 아님
- 읽기 전용 프로퍼티가 mutable 객체를 가진다면 변경 가능
    ```kotlin
    val list = mutableListOf(1, 2, 3, 4)
    list.add(5)
    ```
- `var` 프로퍼티를 사용하는 `val` 프로퍼티는 가변성 존재함
    - 다른 프로퍼티를 활용하는 사용자 정의 getter 로 정의 가능
        ```kotlin
        var name: String = "INIT"
        var surname: String = "SUR_INIT"
        val fullName
            get() = "$name $surname"
        fun main() {
            println(fullName)   // INIT SUR_INIT
            name = "Yongsu"
            println(fullName)   // Yongsu SUR_INIT
        }
        ```
- `val` 은 프로퍼티 레퍼런스 자체를 변경할 수는 없으므로 동기화 문제를 줄이기는 가능함
- 완전히 변경할 피룡가 없다면 `final` 붙일 것
- 스마트 캐스트 활용 가능  
    ※ 스마트 캐스트: `null` 이 아님이 체크된 후 컴파일러가 자동으로 nullable 을 없애는 것과 같이 로직상 타입 확인이 되면 자동으로 인식시켜 주는 것
### 가변 컬렉션과 읽기 전용 컬렉션 구분
- 읽기 전용 컬렉션 인터페이스
    - `Iterable`
    - `Collection`
    - `Set`
    - `List`
- RW 컬렉션 인터페이스
    - `MutableIterable`
    - `MutableCollection`
    - `MutableSet`
    - `MutableList`
- 읽기 전용 컬렉션이 내부의 값을 변경할 수 없다는 의미는 아님
    - 대부분의 경우 변경 가능함
    - 예로 `Iterable<T>.map`, `Iterable<T>.filter` 는 `ArrayList` 반환
- 불변이 아닌 읽기 전용으로의 컬렉션 설계는 더 많은 자유를 얻을 수 있음
    - 내부적으로 인터페이스를 사용하므로 실제 컬렉션을 반환할 수 있음
    - 플랫폼 고유 컬렉션 사용 가능
- 이는 코틀린이 내부적으로 immutable 하지 않은 컬렉션으로 외부적으로 immutable 로 보이게 만들어서 얻어지는 안정성
- __문제__: *다운캐스팅을 수행해서 writable 하게 변경하는 경우*
    - 컬렉션 다운캐스팅을 규약 위반이며 추상화를 무시하는 행위
        - 안전하지 않고 예측하지 못한 결과를 초래함
        - *예제: 다운캐스팅을 시도한 예제*
            - 아래 예제에서 `listOf` 는 자바의 `List` 인터페이스를 구현한 `Arrays.ArrayList` 인스턴스를 반환
            - 자바의 `List` 인터페이스는 코틀린의 `MutableList` 로 변경가능
            - 그러나 `Arrays.ArrayList` 는 `add`, `set` 연산을 구현하지 않음
        ```kotlin
        val list = listOf(1, 2, 3)

        if(list is MutableList) { // 다운캐스팅 금지!! 이렇게 하면 안됨!!!
            list.add(4)
        }
        ```
        ```shell
        실행 결과
        ...
        Exception in thread "main"java.lang.UnsupportedOperationExceptionat java.util.AbstractList.add(AbstractList.java:148)at java.util.AbstractList.add(AbstractList.java:108)
        ...
        ```
    - mutable 로 변경해야 한다면 새로운 mutable 컬렉션 생성할 것
    ```kotlin
    val list = listOf(1, 2, 3)
    val mutableList = list.toMutableList()
    mutableList.add(4)
    ```

### 데이터 클래스의 `copy`
- immutable 객체 사용의 장점
    - 상태 유지 -> 코드 이해의 쉬움
    - 불변 객체 공유로도 충돌 발생 없음, 안전한 병렬 처리 가능
    - 불변 객체 참조가 변경되지 않으므로 쉽게 캐싱 가능
    - 방어적인 복사본(defensive copy), deep copy 만들 필요 없음
    - 다른 객체 생성 시 활용 좋음, 쉽게 예측 가능
    - set 혹은 map 의 키로 활용 가능
- immutable 객체는 수정이 불가하므로 새로운 객체를 만들어내는 메서드 필요함
    - 모든 프로퍼티를 대상으로 이런 메서드를 만들어내는 건 굉장히 귀찮은 일
- `copy` 메서드를 활용
    ```kotlin
    data class User(
        val name: String,
        val surname: String
    )

    var user = User("Yongsu", "Lee")
    user = user.copy(surname = "Kim")
    print(user) // User(name=Yongsu, surname=Kim)
    ```
## 3. 다른 종류의 변경 가능 지점
- mutable 컬렉션 수정(Case_1) VS immutable 컬렉션을 통한 새로운 컬렉션 생성(Case_2)
    - Case_1 은 구현 내부에서 변경이 수행됨
    - Case_2 는 프로퍼티 자체 변경 수행됨
    - Case_1 대비 Case_2 가 멀티스레드 처리 안정성이 더 좋음
        - Case_1 은 내부적으로 동기화 처리가 되어 있는지 확실하게 알 수 없음
- mutable 리스트 대신 mutable 프로퍼티를 사용하는 형태는 사용자 정의 setter를 활용해서 변경 추적 가능
    - `Delegates.observable` 사용 시 리스트 변경이 존재할 때 로그 출력 가능
    ```kotlin
    var names by Delegates.observable(listOf<String>()) { _, old, new ->
        println("Names changed from $old to $new")
    }

    names += "Fabio"
    // Names changed from [] to [Fabio]
    names += "Bill"
    // Names changed from [Fabio] to [Fabio, Bill]
    ```
- 프로퍼티와 컬렉션 모두 변경 가능한 것으로 만들지 말 것!!
    - 두 지점에 대한 동기화 모두 구현해야 함
    - 모호성 발생으로 `+=` 사용 불가

## 4. 변경 가능 지점 노출하지 말기
- 상태를 저장하는 프로퍼티를 `private` 화 하더라도 get 하는 메서드가 있다면 수정 가능
```kotlin
data class User(val name: String)
class UserRepository{
    private val storedUsers: MutableMap<int, String> =
        mutableMapOf()

    fun loadAll(): MutableMap<Int, String> {
        return storedUsers
    }
}
val userRepository = UserRepository()
storedUsers[4] = "YongSu"
...
```
- 해결 방법
    - 방어적 복제(defensive copying) 를 사용할 것
    ```kotlin
    class UserHolder {
        private val user: MutableUser()

        fun get(): MutableUser {
            return user.copy()
        }
        ...
    }
    ```
    - 읽기 전용 슈퍼 타입으로 업캐스팅하여 가변성 제한할 것
    ```kotlin
    data class User(val name: String)
    class UserRepository{
        private val storedUsers: MutableMap<int, String> =
            mutableMapOf()

        fun loadAll(): Map<Int, String> {
            return storedUsers
        }
        ...
    }
    ```
