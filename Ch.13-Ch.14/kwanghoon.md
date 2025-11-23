# 함수형 도구 연쇄 (Chaining Functional Tools)

## 체이닝의 필요성

실무에서는 단순한 map, filter 하나로 끝나지 않는다.
복잡한 비즈니스 로직은 여러 단계의 변환과 필터링을 거쳐야 한다.

## 체이닝을 명확하게 만드는 방법

1. 단계별 이름 붙이기 (Step Naming)

2. 콜백 함수에 이름 붙이기 (Named Callbacks)

3. 균형 잡기

## 체이닝 개선 과정 

### 1. 체이닝 없는 클래식한 구현체 
``` kotlin
private fun findUpcomingTransactionInfo(
    transactions: List<Transaction>,
    allSchedules: List<RepaymentSchedule>,
    today: LocalDate,
    twoDaysFromNow: LocalDate,
): UpcomingTransactionInfoDTO? {
    val transactionIds = transactions.mapNotNull { it.id }
    
    // 1단계: 첫 번째 필터링 - 중간 리스트 생성
    val filteredByTransactionId = mutableListOf<RepaymentSchedule>()
    for (schedule in allSchedules) {
        if (transactionIds.contains(schedule.transactionId)) {
            filteredByTransactionId.add(schedule)
        }
    }
    
    // 2단계: 두 번째 필터링 - 또 다른 중간 리스트 생성
    val filteredByStatus = mutableListOf<RepaymentSchedule>()
    for (schedule in filteredByTransactionId) {
        if (schedule.status == RepaymentStatus.IN_PROGRESS) {
            filteredByStatus.add(schedule)
        } else if (schedule.scheduledDate >= today && schedule.scheduledDate <= twoDaysFromNow) {
            filteredByStatus.add(schedule)
        }
    }
    
    // 3단계: 빈 리스트 체크
    if (filteredByStatus.isEmpty()) {
        return null
    }
    
    // 4단계: 최소값 찾기 - 순회
    var earliestSchedule: RepaymentSchedule = filteredByStatus[0]
    for (i in 1 until filteredByStatus.size) {
        val currentSchedule = filteredByStatus[i]
        if (currentSchedule.scheduledDate < earliestSchedule.scheduledDate) {
            earliestSchedule = currentSchedule
        }
    }
    
    // 5단계: DTO 생성
    return UpcomingTransactionInfoDTO(
        scheduleId = earliestSchedule.id!!,
        dueDate = earliestSchedule.scheduledDate,
        amount = earliestSchedule.scheduledAmount.toLong(),
    )
}
```
-> 주석과 함께 각 중간단계를 형성하며 단계별로 진행은 이해할수 있으나, 코드의 가독성이 좋다고 말할 수는 없음
-> 각 단계에서 정확히 무엇을 하는지를 인지할 수는 없고 코드를 읽고 이해해야함
-> 불필요한 중간 컬렉션 생성

-> 데이터 전체 순회 (이 코드에선 예외)
-> Early exit 불가 (이 코드에선 예외)

### 2. 체이닝 반영
``` kotlin
private fun findUpcomingTransactionInfo(
    transactions: List<Transaction>,
    allSchedules: List<RepaymentSchedule>,
    today: LocalDate,
    twoDaysFromNow: LocalDate,
): UpcomingTransactionInfoDTO? {
    val transactionIds = transactions.mapNotNull { it.id }

    return allSchedules
        .filter { it.transactionId in transactionIds }
        .filter { 
            it.status == RepaymentStatus.IN_PROGRESS ||
                (it.scheduledDate in today..twoDaysFromNow)
        }
        .minByOrNull { it.scheduledDate }
        ?.let { earliestSchedule: RepaymentSchedule ->
            UpcomingTransactionInfoDTO(
                scheduleId = earliestSchedule.id!!,
                dueDate = earliestSchedule.scheduledDate,
                amount = earliestSchedule.scheduledAmount.toLong(),
            )
        }
}

```
-> `filter`, `minByOrNull`등의 함수형 메소드 사용으로 명확한 동작 인지 (가독성)
-> 체이닝 과정에서 이름을 붙여 중간 단계를 표현하면 조금 더 가독성이 좋아질 여지가 있음
-> 불필요한 컬렉션 생성중

3. 체이닝 성능 개선 (`asSequence`)
``` kotlin
private fun findUpcomingTransactionInfo(
    transactions: List<Transaction>,
    allSchedules: List<RepaymentSchedule>,
    today: LocalDate,
    twoDaysFromNow: LocalDate,
): UpcomingTransactionInfoDTO? {
    val transactionIds = transactions.mapNotNull { it.id }

    return allSchedules
        .asSequence()
        .filter { it.transactionId in transactionIds }
        .filter { 
            it.status == RepaymentStatus.IN_PROGRESS ||
                (it.scheduledDate in today..twoDaysFromNow)
        }
        .minByOrNull { it.scheduledDate }   // 실제로 평가 되는 시점
        ?.let { earliestSchedule: RepaymentSchedule ->
            UpcomingTransactionInfoDTO(
                scheduleId = earliestSchedule.id!!,
                dueDate = earliestSchedule.scheduledDate,
                amount = earliestSchedule.scheduledAmount.toLong(),
            )
        }
}

```
-> 지연 평가 사용하여 성능정 이점 획득 (단일 순회)
-> 불필요한 컬렉션 생성하지 않음


## Early Exit
``` kotlin
fun main() {
    val data = (1..1_000_000).toList()
    
    // List 방식 (Eager)
    val result1 = data
        .filter { it % 2 == 0 }      // 중간 리스트: 500,000개 요소
        .filter { it % 3 == 0 }      // 중간 리스트: 166,666개 요소
        .filter { it % 5 == 0 }      // 중간 리스트: 33,333개 요소
        .firstOrNull()               // 6 찾음
    
    // Sequence 방식 (Lazy)
    val result2 = data
        .asSequence()
        .filter { it % 2 == 0 }      // 중간 리스트 없음
        .filter { it % 3 == 0 }      // 중간 리스트 없음
        .filter { it % 5 == 0 }      // 중간 리스트 없음
        .firstOrNull()               // 30 찾으면 즉시 종료 (30개 요소만 확인)
}
```

