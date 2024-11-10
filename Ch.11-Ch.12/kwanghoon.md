# 일급 함수 II: 함수를 반환하는 함수

## 고차 함수 (Higher-Order Functions)

고차 함수는 다른 함수를 인자로 받거나 함수를 반환하는 함수

## 서버측 코드들에서 활용

### 1. 트랜잭션 처리 래퍼

```kotlin
fun <T> withTransaction(block: () -> T): T {
    return transactionManager.execute {
        block()
    }
}

fun <T> createTransactional(f: () -> T): () -> T {
    return { withTransaction(f) }
}

// 사용
val saveOrderTransactional = createTransactional {
    orderRepository.save(order)
}

// 실제 스프링에서

@Transactional

```

### 2. 캐싱 래퍼

```kotlin
fun <K, V> cached(
    cache: MutableMap<K, V>,
    f: (K) -> V
): (K) -> V {
    return { key: K ->
        cache.getOrPut(key) { f(key) }
    }
}

// 사용
val getUserById = cached(userCache) { id ->
    userRepository.findById(id)
}
```

### 3. 재시도 로직

```kotlin
fun <T> withRetry(
    maxAttempts: Int = 3,
    f: () -> T
): () -> T {
    return {
        var lastException: Exception? = null
        repeat(maxAttempts) { attempt ->
            try {
                return@withRetry f()
            } catch (e: Exception) {
                lastException = e
                logger.warn("Attempt ${attempt + 1} failed: ${e.message}")
            }
        }
        throw lastException!!
    }
}
```

---


## 인사이트

### 1. 가독성과 성능의 균형

함수형 체이닝은 가독성을 높이지만, 중간 컬렉션 생성으로 인한 오버헤드가 있을 수 있다.

```kotlin
// 가독성 우선
users
    .filter { it.isActive }
    .map { it.email }
    .filter { it.endsWith("@company.com") }

// 성능 최적화 (Sequence 사용)
users.asSequence()
    .filter { it.isActive }
    .map { it.email }
    .filter { it.endsWith("@company.com") }
    .toList()
```

**대량 데이터 처리시 `Sequence` 사용 고려**

### 2. Stream API의 한계 인식

```kotlin
// 부수효과가 있는 map (안티패턴)
orders.map {
    orderRepository.save(it)  // 각 요소마다 DB 저장
}

// forEach 사용 (의도 명확)
orders.forEach {
    orderRepository.save(it)
}

// 또는 배치 처리
orderRepository.saveAll(orders)
```

**Map은 변환용, 부수효과는 forEach나 별도 함수에서 처리**

### 3. Null 안전성과 함수형 도구

Kotlin의 Null 안전성과 함수형 도구를 결합하면 강력하다:

```kotlin
val validEmails = users
    .mapNotNull { it.email }  // null 제거 + 타입 변환
    .filter { it.isNotBlank() }
    .distinct()
```

## 정리

Ch.11 함수를 반환하는 고차 함수를 통해 횡단 관심사를 우아하게 처리하는 방법 고안

Ch.12의 내용은 \ Map/Filter/Reduce는 단순히 반복문을 대체하는 것을 넘어,
**선언적 프로그래밍**의 핵심이다. "어떻게"가 아닌 "무엇을" 표현하는 코드는
가독성과 유지보수성을 크게 향상시킨다.

서버측에서 이 패턴들은:
- **DTO 변환**: Entity ↔ DTO 매핑에 필수
- **데이터 집계**: 통계, 리포트 생성
- **유효성 검증**: 복잡한 비즈니스 규칙 체이닝
- **횡단 관심사**: 로깅, 캐싱, 트랜잭션 처리

등에서 매일 사용하는 핵심 도구가 될 수 있다.
