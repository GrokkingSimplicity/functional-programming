# Grokking Simplicity 원칙을 적용한 결제수단 사용 여부 확인 기능 리팩토링

## 들어가며

결제수단 사용 여부 확인 기능을 구현하면서 "Grokking Simplicity"의 핵심 원칙인 **액션(Actions)**, **계산(Calculations)**, **데이터(Data)** 의 구분을 적용한 리팩토링 사례를 공유

이번 리팩토링의 핵심 목표는 다음과 같습니다:
- MapStruct + Kotlin 환경에서 null 안전성을 보장하면서도 유연한 매핑 구현
- 비즈니스 로직을 순수 함수(계산)로 분리하여 테스트 용이성 향상
- 서비스 레이어에서 액션과 계산의 명확한 분리

## Chapter 3: 액션과 계산, 데이터의 차이를 알기

### 세 가지 코드의 분류

Grokking Simplicity 3장에서는 모든 코드를 세 가지 범주로 구분합니다:

#### 1. 액션 (Actions)
- **정의**: 실행 시점과 횟수에 의존하는 코드
- **특징**:
  - 부수 효과(side effects)가 있음
  - 같은 입력에 대해 다른 결과를 낼 수 있음
  - 외부 시스템과의 상호작용 (DB, API 호출 등)
- **예시**: 데이터베이스 조회, API 호출, 이메일 전송, 현재 시간 조회

#### 2. 계산 (Calculations)
- **정의**: 입력으로부터 출력을 계산하는 순수 함수
- **특징**:
  - 같은 입력에 대해 항상 같은 출력 반환
  - 부수 효과가 없음
  - 테스트하기 쉬움
  - 재사용하기 쉬움
- **예시**: 수학 연산, 데이터 변환, 필터링, 정렬

#### 3. 데이터 (Data)
- **정의**: 이벤트나 사실에 대한 기록
- **특징**:
  - 불변(immutable)
  - 직렬화 가능
  - 데이터 자체로는 아무 일도 일어나지 않음
- **예시**: DTO, Entity, 설정 값

### 왜 구분이 중요한가?

1. **테스트 용이성**: 계산은 외부 의존성 없이 단위 테스트 가능
2. **재사용성**: 계산은 다양한 컨텍스트에서 재사용 가능
3. **이해 가능성**: 코드의 복잡도를 명확히 이해할 수 있음
4. **유지보수성**: 액션을 최소화하고 계산을 최대화하면 변경에 강한 코드가 됨

## Chapter 4: 액션에서 계산 빼내기

### 계산 추출의 원칙

1. **서브루틴 추출하기**
   - 액션 속의 계산 로직을 별도 함수로 분리
   - 함수명으로 의도를 명확히 표현

2. **암묵적 입력과 출력을 명시적으로 만들기**
   - 전역 변수 읽기 → 함수 파라미터로 전달
   - 전역 변수 쓰기 → 함수 반환값으로 처리

3. **계산을 위한 레이어 만들기**
   - 비즈니스 로직 계산을 별도 레이어로 분리
   - 도메인 로직을 재사용 가능한 형태로 구성

### 계산 추출의 이점

1. **테스트 가능성**: 외부 의존성 없이 순수 함수 테스트 가능
2. **재사용성**: 다양한 액션에서 같은 계산 로직 활용
3. **가독성**: 액션의 책임이 줄어들어 코드 이해가 쉬워짐
4. **병렬화**: 계산은 안전하게 병렬 실행 가능

## 리팩토링 분석


### 1. 액션과 계산의 명확한 분리

#### Before: 액션과 계산이 혼재된 코드
```kotlin
@DriverTransactional
fun checkPaymentCardUsage(cardId: UUID): PaymentCardUsageResponseDTO {
    // 액션: DB 조회 + 계산: 가장 오래된 구독 선택
    // 나쁜 코드라고 볼수는 없으나 테스트 용이성이나 재사용성이좋다고 말할수는 없는 코드 
    val activeSubscription = subscriptionRepository
        .findActiveSubscriptionsByPaymentMethodId(cardId.toString())
        .minByOrNull { it.createdAt!! }  // 
    
    
    // 계산: 데이터 변환
    // 계산영역이나 함수화 되지 않아 테스트 용이성, 재사용성 떨어짐
    return if (activeSubscription != null) {
        val serviceType = activeSubscription.subscriptionType?.serviceType
        PaymentCardUsageResponseDTO(
            inUse = true,
            serviceType = serviceType?.let { SubscriptionServiceTypeDTO.forValue(it.value) },
            subscriptionId = UUID.fromString(activeSubscription.id)
        )
    } else {
        PaymentCardUsageResponseDTO(
            inUse = false,
            serviceType = null,
            subscriptionId = null
        )
    }
}
```

**문제점**:
- DB 조회(액션)와 비즈니스 로직(계산)이 한 곳에 섞여 있음
- 데이터 변환 로직이 서비스 레이어에 직접 구현되어 재사용 불가
- "가장 오래된 구독 선택" 로직의 비즈니스 의도가 명확하지 않음
- Null 처리와 데이터 변환 로직이 복잡하게 얽혀 있음

#### After: 액션과 계산을 명확히 분리
```kotlin
/**
 * 결제수단 사용 여부 확인
 *
 * - 액션(Action): DB 조회
 * - 계산(Calculation): 확장 함수 (비즈니스 로직) + MapStruct Mapper (데이터 변환)
 * 
 * 서비스 로직이므로 가독성에 집중 액션 자체에 
 */
@DriverTransactional
fun checkPaymentCardUsage(cardId: UUID): PaymentCardUsageResponseDTO {
    val activeSubscriptions = subscriptionRepository.findActiveSubscriptionsByPaymentMethodId(cardId.toString())
    val oldestSubscription = activeSubscriptions.selectOldest()
    return paymentMethodMapper.toPaymentCardUsageResponse(oldestSubscription)
}
```

**개선점**:
- **액션**: DB 조회 (`subscriptionRepository.findActiveSubscriptionsByPaymentMethodId`)
- **계산 1**: 비즈니스 로직 (`selectOldest()` 확장 함수)
- **계산 2**: 데이터 변환 (`paymentMethodMapper.toPaymentCardUsageResponse`)
- 각 계산은 독립적으로 테스트 가능
- 서비스 레이어는 액션과 계산의 조합만 담당

### 2. 비즈니스 로직을 순수 함수로 추출

#### SubscriptionExtensions.kt: 순수 계산 함수
```kotlin
package chabot.api.domain.model.entity.driver.subscription

/**
 * 구독 리스트에서 가장 오래된 구독을 선택
 *
 * 비즈니스 규칙:
 * - 여러 구독이 있을 경우 가장 먼저 생성된 구독을 우선
 * - 결제수단 사용 여부 확인 시 가장 오래된 구독 정보를 제공
 *
 * @return 가장 오래된 구독 (없으면 null)
 */
fun List<Subscription>.selectOldest(): Subscription? {
    return this.minByOrNull { it.createdAt!! }
}
```

- **순수 함수 (계산)**: 같은 입력에 대해 항상 같은 출력
- **부수 효과 없음**: 외부 상태를 변경하지 않음
- **테스트 용이**: 구독 리스트를 통산 계산에 집중한 테스트 작성 가능
- **재사용 가능**: 다른 컨텍스트에서도 사용 가능
- **명확한 의도**: 함수명에 비즈니스 규칙 명시

**확장 함수 선택 이유**:
- 도메인 객체(`List<Subscription>`)와 비즈니스 로직을 자연스럽게 결합
- Kotlin의 관용적 표현으로 가독성 향상
- 도메인 로직에 자연스럽게 녹여내는것 고려 중

### 3. 데이터 변환을 MapStruct Mapper로 분리

데이터 변환 로직을 별도 계층으로 분리

#### PaymentMethodMapper.kt: 데이터 변환 계산
```kotlin
@Mapper(
    componentModel = "spring",
    uses = [SubscriptionServiceTypeMapper::class]
)
abstract class PaymentMethodMapper {
    /**
     * Null 처리를 포함한 결제수단 사용 여부 응답 생성
     *
     * Public API로 null 입력을 안전하게 처리하고,
     * Non-null 케이스는 MapStruct가 생성한 내부 구현에 위임
     */
    fun toPaymentCardUsageResponse(subscription: Subscription?): PaymentCardUsageResponseDTO {
        return if (subscription == null) {
            PaymentCardUsageResponseDTO(inUse = false, serviceType = null, subscriptionId = null)
        } else {
            toPaymentCardUsageResponseInternal(subscription)
        }
    }

    /**
     * MapStruct가 구현을 생성하는 내부 매핑 메서드
     *
     * Non-null Subscription을 PaymentCardUsageResponseDTO로 변환
     */
    @Mapping(target = "inUse", expression = "java(true)")
    @Mapping(target = "serviceType", source = "subscriptionType.serviceType")
    @Mapping(target = "subscriptionId", source = "id", qualifiedByName = ["stringToUUID"])
    protected abstract fun toPaymentCardUsageResponseInternal(subscription: Subscription): PaymentCardUsageResponseDTO

    companion object {
        @JvmStatic
        @Named("stringToUUID")
        fun stringToUUID(id: String?): UUID? {
            return id?.let { UUID.fromString(it) }
        }
    }
}
```

**Two-Method 패턴의 이유**:
- **Public 메서드** (`toPaymentCardUsageResponse`): Null 처리를 명시적으로 구현
- **Protected 메서드** (`toPaymentCardUsageResponseInternal`): MapStruct가 코드 생성
- Kotlin의 non-null 타입 시스템과 MapStruct의 코드 생성을 조화롭게 결합

#### SubscriptionServiceTypeMapper.kt: 재사용 가능한 계산
```kotlin
package chabot.api.application.mapper

import chabot.api.domain.model.entity.driver.subscription.SubscriptionServiceType
import chabot.api.server.web.v1.model.SubscriptionServiceTypeDTO
import org.mapstruct.Mapper

@Mapper(componentModel = "spring")
abstract class SubscriptionServiceTypeMapper {
    fun toDTO(serviceType: SubscriptionServiceType?): SubscriptionServiceTypeDTO? {
        return serviceType?.let { SubscriptionServiceTypeDTO.forValue(it.value) }
    }

    fun toDomain(serviceTypeDTO: SubscriptionServiceTypeDTO?): SubscriptionServiceType? {
        return serviceTypeDTO?.let { SubscriptionServiceType.valueOf(it.value) }
    }
}
```

**Mapper 조합 패턴**:
```kotlin
@Mapper(
    componentModel = "spring",
    uses = [SubscriptionServiceTypeMapper::class]  // 다른 Mapper 재사용
)
abstract class PaymentMethodMapper {
    // SubscriptionServiceTypeMapper의 toDTO 메서드가 자동으로 사용됨
}
```

### 4. 전체 흐름: 액션과 계산의 조합

리팩토링 후 전체 데이터 흐름은 다음과 같이 명확하게 구분됩니다:

```
PaymentMethodService.checkPaymentCardUsage()
    │
    ├─ [액션] subscriptionRepository.findActiveSubscriptionsByPaymentMethodId()
    │   └─ DB에서 활성 구독 목록 조회
    │
    ├─ [계산] activeSubscriptions.selectOldest()
    │   └─ 순수 함수로 구현된 비즈니스 로직
    │   └─ 가장 오래된 구독 선택
    │
    └─ [계산] paymentMethodMapper.toPaymentCardUsageResponse()
        ├─ Null 체크 및 기본값 반환
        └─ Non-null 케이스: MapStruct 코드 생성
            └─ subscriptionServiceTypeMapper.toDTO()
                └─ Enum 변환 (재사용 가능한 계산)
```


## 인사이트

### 1. 테스트 용이성 향상

**Before**: 서비스 메서드를 테스트하려면 DB Mock 필요
```kotlin
// 복잡한 통합 테스트만 가능
@Test
fun checkPaymentCardUsage() {
    // DB Mock 설정
    // 서비스 호출
    // 결과 검증 (비즈니스 로직 + 데이터 변환 + Null 처리 모두 포함)
}
```

**After**: 각 계산을 독립적으로 단위 테스트 가능
```kotlin
// 확장 함수 테스트 (비즈니스 로직만)
@Test
fun `가장 오래된 구독 선택`() {
    val subscriptions = listOf(/* ... */)
    val oldest = subscriptions.selectOldest()
    // 검증
}

// Mapper 테스트 (데이터 변환만)
@Test
fun `구독 정보를 응답 DTO로 변환`() {
    val subscription = createSubscription()
    val response = mapper.toPaymentCardUsageResponse(subscription)
    // 검증
}
```

### 2. 재사용성 증가

**Before**: 비즈니스 로직이 서비스 메서드에 묶여 있음
``` text
  다른 곳에서 "가장 오래된 구독" 로직이 필요하면?
    → 코드 중복 발생
```

**After**: 계산은 어디서든 재사용 가능
```kotlin
// 확장 함수는 List<Subscription>이 있는 모든 곳에서 사용 가능
val oldest = subscriptions.selectOldest()

// Mapper도 다른 API에서 재사용 가능
val response = paymentMethodMapper.toPaymentCardUsageResponse(subscription)
```

### 3. 가독성 및 유지보수성 향상

**Before**: 서비스 메서드가 너무 많은 책임을 가짐
- DB 조회
- 비즈니스 로직 (가장 오래된 구독 선택)
- Null 처리
- 데이터 변환 (Enum 변환 포함)

**After**: 각 레이어가 명확한 단일 책임을 가짐
- Service: 액션과 계산의 조합만
- Extension: 비즈니스 로직만
- Mapper: 데이터 변환만


## 결론

이번 리팩토링을 통해 Grokking Simplicity의 핵심 원칙들이 실제 프로덕션 코드에서 어떻게 적용되고, 어떤 가치를 제공하는지 확인할 수 있었습니다.

### 얻은 것들

1. **명확한 코드 구조**
   - 액션, 계산, 데이터의 구분으로 각 코드의 역할이 명확해짐
   - 가독성 증가

2. **강력한 테스트 전략**
   - 계산은 단위 테스트로 빠르게 검증
   - 액션은 필요한 경우에만 통합 테스트
   - 테스트 실행 속도와 안정성 모두 향상

3. **높은 재사용성**
   - `selectOldest()` 확장 함수는 다른 구독 관련 기능에서 재사용
   - `SubscriptionServiceTypeMapper`는 다른 Mapper에서 조합 가능
   - 비즈니스 로직의 중복 제거

### 마치며
- 액션의 전염에 대한부분은 일정 부분 타협할수밖에 없었다. 
- 계산 로직을 분리해내며 테스트가 용이하고 재사용 가능한 코드를 작성했다는 점에 의미
- 현재 아키텍쳐의 한계상 Entity 자체가 불변데이터가 아님이 조금 아쉬움
---


