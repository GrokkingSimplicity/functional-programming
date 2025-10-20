# 불변성 다루기: val의 함정

## 불변성이란?

불변성(Immutability)은 데이터가 생성된 후 변경되지 않는 특성을 의미

- **예측 가능한 코드**: 데이터가 변하지 않으므로 프로그램의 동작을 예측하기 쉽습니다
- **스레드 안전성**: 여러 스레드에서 동시에 접근해도 안전합니다
- **디버깅 용이성**: 데이터 변경 추적이 필요 없어 버그를 찾기 쉽습니다
- **시간 여행 디버깅**: 이전 상태를 보관하고 되돌릴 수 있습니다

## Copy-on-Write 규율
Copy-on-Write의 3단계:

- 복사본 만들기
- 복사본 수정하기
- 복사본 반환하기

## 불변성을 다루는 방법들

### 1. 방어적 복사 (Defensive Copy)

객체를 전달하거나 반환할 때 복사본을 만드는 방법입니다.

```kotlin
class UserService {
    private val users = mutableListOf<User>()
    
    // 방어적 복사를 통한 불변성 보장
    fun getUsers(): List<User> {
        return users.toList() // 새로운 리스트 반환
    }
}
```

### 2. 불변 데이터 클래스 사용

```kotlin
data class User(
    val id: Long,
    val name: String,
    val email: String
)

// 변경이 필요한 경우 copy() 사용
val user = User(1, "Alice", "alice@example.com")
val updatedUser = user.copy(name = "Alice Smith")
```


### 3. 불변 컬렉션 사용

```kotlin
// 읽기 전용 인터페이스 사용
val immutableList: List<String> = mutableListOf("A", "B", "C")
// immutableList.add("D") -> 불가능 
(immutableList as MutableList).add("E") // -> 

// 실제 불변 컬렉션 (kotlinx.collections.immutable)
val persistentList = persistentListOf("A", "B", "C")
```

## val의 함정: 참조 불변성 vs 객체 불변성

많은 개발자들이 `val` 키워드를 사용하면 데이터가 완전히 불변이라고 착각할 수 있음

### val이 보장하는 것

`val`은 참조의 불변성(Reference Immutability)만 보장
즉, 변수가 가리키는 객체를 다른 객체로 재할당할 수 없다는 의미

```kotlin
val list = mutableListOf(1, 2, 3)
list = mutableListOf(4, 5, 6) //  컴파일 에러: 재할당 불가능
```

### val이 보장하지 않는 것

`val`은 객체 내부 상태의 불변성(Object Immutability)을 보장하지 않는다

```kotlin
val list = mutableListOf(1, 2, 3)

list.add(4)           // 리스트에 요소 추가
list.remove(1)        // 리스트에서 요소 제거
list[0] = 100         // 리스트 요소 변경
list.clear()          // 리스트 전체 삭제

println(list) // []
```

### 예제1 

```kotlin
class ShoppingCart {
    private val items = mutableListOf<Item>()
    
    fun addItem(item: Item) {
        items.add(item)
    }
    
    // 위험한 코드: 내부 mutable 컬렉션 노출
    fun getItems(): List<Item> {
        return items
    }
}

// 사용 예시
val cart = ShoppingCart()
cart.addItem(Item("Book", 10000))
cart.addItem(Item("Pen", 1000))

val items = cart.getItems()
println(items.size) // 2

// critical section
// 동시 작업으로 여러 섹션에서 처리하는 로직이라고 가정해봐도 좋음
if (items is MutableList) {
    items.clear() // 장바구니가 비워짐!
}

println(cart.getItems().size) // 0
```

### 왜 이런 일이 발생할까?

`getItems()`가 `List<Item>`을 반환하지만, 실제로는 내부의 `MutableList`에 대한 참조를 반환 Kotlin의 `List`는 단지 **읽기 전용 인터페이스**일 뿐, 실제 객체가 불변임을 보장하지 않는다.

```kotlin
val items: List<Item> = cart.getItems()
// items는 List 타입이지만, 실제로는 MutableList를 가리킴
// 스마트 캐스트나 타입 체크를 통해 변경 가능
```

## copy()의 함정: 얕은 복사의 위험

`data class`의 `copy()` 메서드도 안전하지 않음
`copy()`는 **얕은 복사(Shallow Copy)**만 수행하기 때문에, 컬렉션이나 중첩 객체의 참조만 복사

### copy()가 안전한 경우

기본 타입(Int, String 등)만 포함된 경우는 안전

```kotlin
data class User(val id: Long, val name: String)

val user1 = User(1, "Alice")
val user2 = user1.copy(name = "Bob")

println(user1.name) // Alice
println(user2.name) // Bob
// ✅ 독립적으로 동작
```

### copy()가 위험한 경우 1: 컬렉션 포함

```kotlin
data class Team(
    val name: String,
    val members: MutableList<String>
)

val team1 = Team("Dev Team", mutableListOf("Alice", "Bob"))
val team2 = team1.copy()

// Issue!
team2.members.add("Charlie")

println(team1.members) // [Alice, Bob, Charlie]
println(team2.members) // [Alice, Bob, Charlie]
println(team1.members === team2.members) // true
```

-> `copy()`는 새로운 `Team` 객체를 만들지만, `members` 리스트는 같은 객체를 참조

### copy()가 위험한 경우 2: 중첩 객체

```kotlin
data class Address(var city: String, var street: String)
data class Person(val name: String, val address: Address)

val person1 = Person("Alice", Address("Seoul", "Gangnam"))
val person2 = person1.copy()

// Issue!
person2.address.city = "Busan"

println(person1.address.city) // Busan - 예상치 못한 변경!
println(person2.address.city) // Busan
println(person1.address === person2.address) // true - 같은 Address 객체!
```

### 해결 방법: 깊은 복사 (Deep Copy)

#### 컬렉션의 경우

```kotlin
// ❌ 위험한 방법
val team2 = team1.copy()

// ✅ 안전한 방법 1: 컬렉션도 복사
val team2 = team1.copy(
    members = team1.members.toMutableList()
)

// ✅ 안전한 방법 2: 불변 컬렉션 + 연산자
data class Team(val name: String, val members: List<String>)
val team2 = team1.copy(
    members = team1.members + "Charlie" // 새 리스트 생성
)
```

#### 중첩 객체의 경우

```kotlin
// ❌ 위험한 방법
val person2 = person1.copy()

// ✅ 안전한 방법: 중첩 객체도 copy()
val person2 = person1.copy(
    address = person1.address.copy()
)

// 이제 독립적
person2.address.city = "Busan"
println(person1.address.city) // Seoul ✅
```

#### 복잡한 중첩 구조의 경우

```kotlin
data class Department(val name: String)
data class Address(val city: String)
data class Employee(
    val name: String,
    val department: Department,
    val address: Address,
    val skills: List<String>
)

// 모든 중첩 객체를 수동으로 복사해야 함
val employee2 = employee1.copy(
    department = employee1.department.copy(),
    address = employee1.address.copy(),
    skills = employee1.skills.toList()
)
```

이런 경우 deep copy 함수를 별도로 만들거나, 직렬화 라이브러리를 사용하는 것 권장

### copy()를 안전하게 사용하는 원칙

1. **불변 컬렉션 사용**: `List`, `Set`, `Map` (읽기 전용 인터페이스)
2. **기본 타입만 포함**: String, Int, Boolean 등
3. **중첩 객체도 copy()**: 모든 참조 타입에 대해 명시적으로 복사
4. **깊은 복사가 필요하면 명시적으로**: 자동으로 처리되지 않음을 인지

```kotlin
// ✅ 안전한 설계
data class SafeTeam(
    val name: String,
    val members: List<String>  // 불변 컬렉션
)

val team2 = team1.copy(
    members = team1.members + "Charlie"  // 새 리스트 생성
)
```

## 해결 방법

### 방법 1: 방어적 복사

```kotlin
class ShoppingCart {
    private val items = mutableListOf<Item>()
    
    fun addItem(item: Item) {
        items.add(item)
    }
    
    // ✅ 안전: 새로운 리스트 반환
    fun getItems(): List<Item> {
        return items.toList()
    }
}
```

### 방법 2: 불변 컬렉션 사용

```kotlin
import kotlinx.collections.immutable.*

class ShoppingCart {
    private var items: ImmutableList<Item> = persistentListOf()
    
    fun addItem(item: Item) {
        // 새로운 불변 리스트 생성
        items = items.add(item)
    }
    
    // ✅ 안전: 진짜 불변 리스트 반환
    fun getItems(): ImmutableList<Item> {
        return items
    }
}
```

### 방법 3: 읽기 전용 뷰 + 내부 복사

```kotlin
class ShoppingCart {
    private val _items = mutableListOf<Item>()
    val items: List<Item>
        get() = _items.toList()
    
    fun addItem(item: Item) {
        _items.add(item)
    }
}
```

## 체크리스트: 진짜 불변성을 달성했는가?

불변성을 제대로 구현했는지 확인하려면 다음을 점검하세요:

- [ ] `val`을 사용했는가?
- [ ] 컬렉션이 `List`, `Set`, `Map` 같은 읽기 전용 인터페이스인가?
- [ ] 내부 mutable 컬렉션을 직접 노출하지 않는가?
- [ ] 반환 시 방어적 복사를 수행하는가?
- [ ] 중첩된 객체나 컬렉션도 불변인가?
- [ ] 데이터 클래스의 모든 프로퍼티가 `val`인가?
- [ ] 진짜 불변 컬렉션 라이브러리를 고려했는가?

## 결론

진정한 불변성을 달성하려면:

1. **참조 불변성**과 **객체 불변성**을 구분
2. 컬렉션을 다룰 때는 항상 주의
3. 중첩된 객체/배열 수정시 모든 레벨에서 copy-on-write 적용

