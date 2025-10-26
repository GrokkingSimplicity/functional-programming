# 신뢰할 수 없는 코드를 쓰면서 불변성 지키기

## 방어적 복사의 실무 사례

```kotlin
@Entity
class ShoppingCart {
    // DB에 의해 계속 변경될 수 있는 값, 신뢰 할 수 없는 영역
    @Column(name = "items")
    private val _items = mutableListOf<Item>()

    // 다른 레이어, 모듈에서 해당 값을 참조할때 방어적 복사 진행 
    val items: List<Item>
        get() = _items.toList()
    
    // 변경을 유연하게 제공
    internal fun addItem(item: Item) {
        _items.add(item)
    }

    internal fun deleteItem(item: Item) { _items.remove(item) }

    // 다른 모듈의 비즈니스 로직 접근 제어 통제
    private fun updateItem(item: Item) {}
}
```
-> _items는 JPA의 영속성 컨텍스트에 의해 관리되어 변경가능성 존재
-> items를 getter의 경우 `방어적 복사`를 통한 신뢰성 획득
-> kotlin의 경우 internal이라는 키워드를 통해 각 레이어, 모듈간 제어권 제어 (비즈니스 정책만 관리하는 모듈에서만 사용가능하도록 제어)


# Stratified Design (계층형 설계)
소프트웨어를 계층으로 구성하는 기술로, 계층에 있는 함수는 바로 아래 계층에 있는 함수를 이용해 정의한다.
책에서 '미감'이라는 표현을 사용하여 정의

### 계층형 설계 패턴

- **직접 구현**
- 추상화 벽
- 작은 인터페이스
- 편리한 계층

## 직접 구현
-> 직접 구현에서 중요한 건 함수 시그니쳐(이름)에서 나타내는 문제를 적절한 추상화 수준에서 구현해야 한다는 것

-> 너무 구체화 되어 있으면 코드 스멜

그에 대한 예시로 freeTieClip로직에서 반복문, 배열 참조등 저수준 로직이 들어가 있는걸 지적하며 체크함

과도한 추상화는 구현의 비효율성을 높이거나, 오히려 복잡도를 높일수 있음
개인마다 가지고 있는 추상화의 정도나 개념이 다를수 있기에 중용의 자세가 필요할지도

### 줌 레벨

너무 많은 정보로 다이어 그램에서 문제를 찾기 어려울때,
마치 카메라 렌즈처럼 관점을 다르게 하여 코드의 문제를 분석하는 관점

세가지 다른 영역
- 계층 사이의 상호 관계
- 특정 계층의 구현
- 특정 함수의 구현


``` kotlin
fun processOrder(order: Order) {
    // Hign Level
    validateOrder(order)
    
    // Low Level
    val tax = order.items.sumOf { it.price * 0.1 }  
    
    // Hign Level
    applyDiscount(order)
    
    // Low Level
    order.status = "PROCESSING"
    
    // Hign Level
    sendNotification(order)
}
```
-> 같은 구체화 수준이 아님

``` kotlin
fun processOrder(order: Order) {
    // 모두 같은 추상화 수준
    validateOrder(order)
    calculateTax(order)
    applyDiscount(order)
    updateOrderStatus(order, OrderStatus.PROCESSING)
    sendNotification(order)
}
```

--- 

# 정리
신뢰성 있는 데이터를 얻기 위한 방법들중에 하나로써 알고있었던 내용을 방어적 복사라는 키워드를 통해 개념을 정리했다는 점에서 의의가 있었다.

Stratified Design 파트에서 타이틀을 보았을 땐 용어상 어떤 아키텍쳐의 개념을 제안하는것처럼 느껴졌지만,
단순히 하나의 Service 로직을 구현할 때에도 해당 관점을 통해 설계하고 구현하는데 사용해도 충분하다는 느낌을 받음

항상 실무에서의 생산성과 효율성, 코드 퀄리티에 대한 기회비용을 고려하고 타협하는 부분도 있지만,
다음에 진행할 프로젝트에선 시간이 걸리더라도 한번 타협하지 않는 코드를 작성해보고 싶은 마음

-> 코드 작성 단계에서 들였던 비용들이 시간이 지나면 지날수록 부채를 늘리지 않고, 프로젝트의 복잡성이 더해졌을 때 변경의 비용에 대한 차이를 느껴보고 싶음