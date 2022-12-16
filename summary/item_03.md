# 3. 최대한 플랫폼 타입을 사용하지 말라

## 1. 플랫폼 타입
- 자바와 같이 다른 프로그래밍 언어에서 넘어온 타입들을 특수하게 다루기 위한 타입
    - 코틀린은 타입이 nullable 인지 아닌지 구분 가능
    - 다른 프로그래밍 언어에서 전달되어서 nullable 인지 아닌지 알 수 없는 타입
- 타입 이름 뒤에 `!` 기호 붙여 표기, 이 노테이션이 직접적으로 코드에 나타나지 않음
- 도입 이유
    - 자바의 경우 `@Nullable` 어노테이션을 구분할 수 있으나 어노테이션이 없는 경우는 모든 타입이 nullable
    - 자바 제네릭 타입은 nullable 관련 자주 문제가 발생
        - `List<User>` 를 반환하는 함수, 어노테이션 없는 경우
            - 리스트, 리스트 내부 `User` 객체가 null 인지 확인 필요
        - `List<List<User>>` 를 반환하는 함수, 어노테이션 없는 경우
            - null 확인해야할 조건이 더 복잡해짐
    - *예제: 플랫폼 타입, nullable 타입, non-null 타입*
    ```kotlin
    // Java code
    public class UserRepo {
        public User getUser() {
            ...
        }
    }

    // Kotlin
    val repo = UserRepo()
    val user1 = repo.user           // Type: User!
    val user2: User = repo.user     // Type: User
    val user3: User? = repo.user    // Type: User?
    ```

## 2. 플랫폼 타입 사용 시 주의할 점
- `null` 이 아니라고 생각되는 것이 `null` 일 가능성 존재
- 특정 함수가 당장 `null` 을 반환하지 않아도 추후 변경될 수 있다는 것을 염두해 둬야 함
- 자바와 함께 사용할 때 조작 가능하다면 자바 API에 `@Nullable`, `@NotNull` 어노테이션 사용하여 플랫폼 타입을 쓰지 않도록 할 것
    - JetBrains
        - `org.jetbrains.annotations`의 `@Nullable`, `@NotNull`
    - Android
        - `androidx.annotation`, `com.android.annotations`, `android.support.annotations`의 `@Nullable`, `@NonNull`
    - JSR-305
        - `javax.annotation`의 `@Nullable`, `@CheckForNull`, `@Nonnull`, `@ParametersAreNonnullByDefault`
    - JavaX
        - `javax.annotation`의 `@Nullable`, `@CheckForNull`, `@Nonnull`
    - FindBugs
        - `edu.umd.cs.findbugs.annotations`의 `@Nullable`, `@CheckForNull`, `@PossiblyNull`, `@NonNull`
    - ReactiveX
        - `io.reactivex.annotations`의 `@Nullable`, `@NonNull`
    - Eclipse
        - `org.eclipse.jdt.annotation`의 `@Nullable`, `@NonNull`
    - Lombok
        - `lombok`의 `@NonNull`
- *예제: 플랫폼 타입 사용 권장하지 않는 이유*
    ```kotlin
    // Java code
    public class JavaClass {
        public String getValue() {
            return null;
        }
    }


    // Kotlin code
    fun statedType() {
        val value: String = JavaClass().getValue()
        ...
        println(value.length)
    }

    fun platformType() {
        val value = JavaClass().getValue()
        ...
        println(value.length)
    }
    ```
    - kotlin 코드의 2개 메소드 모두 Null Pointer Exception 발생
        - `statedtType` 메소드에서는 `value` 에 assign 하는 line 에서 발생
            - 예외 원인 파악이 쉬움(문제 지점과 동일 위치)
        - `platformType` 메소드에서는 `value` 를 사용하는  line 에서 발생
            - 예외 원인 파악 어려움(문제 지점과 거리가 있을 수 있음)