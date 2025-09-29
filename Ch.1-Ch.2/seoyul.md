# 함수형 코딩

- 함수형 코딩이란 부수효과 없이 순수 함수만 사용하는 프로그래밍 스타일
  - 같은 인자를 넣으면 항상 같은 결과를 돌려준다.
- but, 부수효과는 소프트웨어를 실행하는 이유로 필요하다.
- 있어야한다.
- 부를 때 조심해야하는 코드를 구분한다.(액션)
- 실행하는 코드와 그렇지 않은 코드를 구분한다.(실행하는 코드-> 계산, 그렇지 않은 코드->데이터)
- 함수형 프로그래밍에서 중요한 두가지 개념은 액션과 계산, 데이터를 구분해서 생각하는 것과, 일급 추상이라는 개념

## 액션과 계산, 데이터

- 액션은 호출하는 시점과 횟수에 의존하기 떄문에 호출시 조심해야한다.
- 계산이나 데이터는 둘다 부르는 시점이나 횟수가 중요하지 않다.
  - 계산은 실행 가능하나, 데이터는 그렇지 않다.
  - 데이터는 정적이지만, 계산은 실행하지 전까지 어떻게 동작할지 알 수 없다.

### 액션

- 실행 시점이나 횟수에 의존
- 함수형 프로그래밍 도구
  - 시간이 지남에 따라 안전하게 상태를 부꿀 수 있는 방법
  - 순서를 보장하는 방법
  - 액션이 정확히 한번만 실행되게 보장하는 방법

### 계산

- 입력값으로 출력값을 만드는것으로 같은 입력값을 갖고 계산하면 항상 같은 결괏값이 나온다.
- 함수형 프로그래밍 도구
  - 정확성을 위한 정적 분석
  - 소프트웨어에서 쓸 수 있는 수학적 지석
  - 테스트 전략

### 데이터

- 이벤트에 대해 기록한 사실로, 실행하지 않아도 그 자체로 의미가 있다.
- 같은 데이터를 여러 헝태로 해석할 수 있다. (ex. 영수중 데이터: 인기있는 메뉴, 외식비 지출 내역)
- 함수형 프로그래밍 도구
  - 효율적으로 접근하기 위해 데이터를 구성하는 방법
  - 데이터를 보관하기 위한 기술
  - 데이터를 이용해 중요한 것을 발견하는 원칙

### 장점

- 분산 시스템에 잘 어울림.
  - 메세지 순서가 바뀔 수 있다.
  - 메세지는 한번 이상 도착할수도 있고, 도착하지 않을수도 있다.
  - 응답을 받지 못하면 무슨 일이 생겼는지 알 수 없다.
- 시간에 따라 바뀌는 값을 모델링할 때 실행시점이나 횟수에 의존한느 코드를 없애면 코드를 더 쉽게 이해하고, 심각한 버그를 막을 수 있다.
  - 데이터와 계산은 실행 히점이나 횟수에 의존하지 않아 유리하다.
- 액션은 코드 전체에 영향을 주지 않도록 격리한다.
- 코드의 많은 부분을 액션에서 계산으로 옮기면 액션도 다루기 쉬워진다.

# 함수형 사고

## 액션, 계산, 데이터

### 변경 가능성에 따라 코드 나누기

- 계층형 설계는 일반적으로 비즈니스 규칙, 도메인 규칙, 기술 스택 계층으로 나뉜다.
  - 아래있는 것을 기반으로 구현하며, 가장 위에 있는 코드는 의존성이 거의 없기 떄문에 쉽게 바꿀 수 있고, 어래있는 코드는 위에 있는 코드보다 의조성이 많아 바꾸기 어렵지만 자주 바뀌지 않는다.
  - 계층형 설계는 테스트, 재사용, 유지보수에 유리하다.

## 일급 추상

- 타임라인 다이어그램은 한가지 목적을 위한 액션들을 보여준다.
  - 액션이 시간 순서에 따라 어떻게 실행되는지 볼 수 있다.
  - 액션은 실행 시점에 의존하기 때문에 실행순서가 중요함

### 분산시스템

- 여러대의 로봇이 함께 일을 하는것이 분산 시스템

  - 분산 시스템에서 독립된 액션의 실행 순서는 어떻게 될지 모름, 각각의 타임라인은 다른 순서로 실행됨
  - 타임라인을 서로 맞추지 않은 분산시스템은 예측 불가능한 순서로 실행됨
  - 올바른 순서로 동작하는 프로그램을 만들려면 시간에 의존적인 액션에 집중할 필요가 있다.
  - 각각의 타임라인은 다른 타임라인의 순서와 관계없이 만들어야한다.
  - 타임라인은 항상 옳바른 결과를 보장해야한다.

- 여러 타임라인이 동시에 진행될 떄 서로 순서를 맞추기 위해 고차동작(고차 함수로 만든 동작)으로 구현한다.
  - 각 타임란인은 독립적으로 동작하고 작업이 완료되면 달느 타임라인 끝나기를 기다리기 때문에 어떤 타임라인이 먼저 끝나도 괜찮다.
  - 고차동작을 통해 동 시에 할 수 있는 것과 순서대로 해야하는것을 분리

# 피자 만들기

## 주요 포인트

1.  액션, 계산, 데이터 구분

데이터: pizzaOrder, ingredients - 변하지 않는 사실
계산: calculateDoughAmount 등 - 같은 입력이면 항상 같은 출력
액션: receiveOrder, makeDough 등 - 실행 시점과 횟수에 의존

2. 분산 시스템 (병렬 타임라인)

로봇 1, 2, 3이 동시에 반죽, 소스, 치즈를 준비
각각 다른 시간이 걸리지만 순서는 상관없음
waitForAll 고차 함수로 모든 작업 완료를 보장

3. 순차 처리 (순서가 중요한 타임라인)

반죽 펴기 → 소스 뿌리기 → 치즈 뿌리기 → 오븐
runInSequence 고차 함수로 올바른 순서 보장

4. 고차 함수의 역할

여러 타임라인을 조정하여 예측 가능한 결과 보장
동시에 할 수 있는 것과 순서대로 해야 하는 것을 분리

```js
// ============================================
// 데이터: 정적인 사실, 실행되지 않음
// ============================================
const pizzaOrder = {
  orderId: "P001",
  size: "large",
  toppings: ["pepperoni", "mushroom"],
  timestamp: new Date(),
};

const ingredients = {
  dough: { flour: 500, water: 300, yeast: 10 },
  sauce: { tomato: 200, garlic: 5, basil: 3 },
  cheese: { mozzarella: 250 },
};

// ============================================
// 계산: 순수 함수, 같은 입력 -> 같은 출력
// ============================================
function calculateDoughAmount(size) {
  const sizeMultiplier = {
    small: 0.7,
    medium: 1.0,
    large: 1.3,
  };
  return {
    flour: ingredients.dough.flour * sizeMultiplier[size],
    water: ingredients.dough.water * sizeMultiplier[size],
    yeast: ingredients.dough.yeast * sizeMultiplier[size],
  };
}

function calculateSauceAmount(size) {
  const sizeMultiplier = {
    small: 0.7,
    medium: 1.0,
    large: 1.3,
  };
  return {
    tomato: ingredients.sauce.tomato * sizeMultiplier[size],
    garlic: ingredients.sauce.garlic * sizeMultiplier[size],
    basil: ingredients.sauce.basil * sizeMultiplier[size],
  };
}

function calculateCheeseAmount(size) {
  const sizeMultiplier = {
    small: 0.7,
    medium: 1.0,
    large: 1.3,
  };
  return ingredients.cheese.mozzarella * sizeMultiplier[size];
}

// ============================================
// 액션: 실행 시점과 횟수에 의존
// ============================================
function receiveOrder(order) {
  console.log(`📝 [액션] 주문 접수: ${order.orderId} (${order.size})`);
  return order;
}

// 병렬로 실행될 수 있는 액션들 (로봇 1, 2, 3)
function makeDough(order) {
  return new Promise((resolve) => {
    const amount = calculateDoughAmount(order.size);
    console.log(`🤖1 [액션] 반죽 만들기 시작...`);
    setTimeout(() => {
      console.log(`✅ [액션] 반죽 완료! (밀가루: ${amount.flour}g)`);
      resolve({ type: "dough", data: amount, completedAt: Date.now() });
    }, 2000);
  });
}

function makeSauce(order) {
  return new Promise((resolve) => {
    const amount = calculateSauceAmount(order.size);
    console.log(`🤖2 [액션] 소스 만들기 시작...`);
    setTimeout(() => {
      console.log(`✅ [액션] 소스 완료! (토마토: ${amount.tomato}g)`);
      resolve({ type: "sauce", data: amount, completedAt: Date.now() });
    }, 1500);
  });
}

function grateCheese(order) {
  return new Promise((resolve) => {
    const amount = calculateCheeseAmount(order.size);
    console.log(`🤖3 [액션] 치즈 갈기 시작...`);
    setTimeout(() => {
      console.log(`✅ [액션] 치즈 완료! (모짜렐라: ${amount}g)`);
      resolve({ type: "cheese", data: amount, completedAt: Date.now() });
    }, 1000);
  });
}

// 순서대로 실행되어야 하는 액션들
function spreadDough(ingredients) {
  return new Promise((resolve) => {
    console.log(`\n👨‍🍳 [액션] 반죽 펴기...`);
    setTimeout(() => {
      console.log(`✅ [액션] 반죽 펴기 완료!`);
      resolve({ ...ingredients, doughSpread: true });
    }, 500);
  });
}

function applySauce(pizza) {
  return new Promise((resolve) => {
    console.log(`👨‍🍳 [액션] 소스 뿌리기...`);
    setTimeout(() => {
      console.log(`✅ [액션] 소스 뿌리기 완료!`);
      resolve({ ...pizza, sauceApplied: true });
    }, 500);
  });
}

function addCheese(pizza) {
  return new Promise((resolve) => {
    console.log(`👨‍🍳 [액션] 치즈 뿌리기...`);
    setTimeout(() => {
      console.log(`✅ [액션] 치즈 뿌리기 완료!`);
      resolve({ ...pizza, cheeseAdded: true });
    }, 500);
  });
}

function putInOven(pizza) {
  return new Promise((resolve) => {
    console.log(`\n🔥 [액션] 오븐에 넣기...`);
    setTimeout(() => {
      console.log(`✅ [액션] 오븐에 넣기 완료!`);
      resolve(pizza);
    }, 300);
  });
}

function waitForBaking() {
  return new Promise((resolve) => {
    console.log(`⏰ [액션] 10분 기다리는 중...`);
    setTimeout(() => {
      console.log(`✅ [액션] 굽기 완료!`);
      resolve({ baked: true });
    }, 3000); // 실제로는 10분이지만 예시를 위해 3초로
  });
}

function serve(pizza) {
  console.log(`\n🍕 [액션] 서빙! 맛있는 피자가 완성되었습니다!`);
  console.log(`📊 최종 결과:`, pizza);
  return pizza;
}

// ============================================
// 고차 함수: 타임라인 조정
// ============================================
// 여러 비동기 작업이 모두 완료될 때까지 기다리는 고차 함수
function waitForAll(tasks) {
  return Promise.all(tasks);
}

// 순차적으로 실행하는 고차 함수
async function runInSequence(initialValue, ...functions) {
  let result = initialValue;
  for (const fn of functions) {
    result = await fn(result);
  }
  return result;
}

// ============================================
// 피자 만들기 프로세스 실행
// ============================================
async function makePizza(order) {
  console.log("🍕 ===============================");
  console.log("🍕   피자 만들기 시작!");
  console.log("🍕 ===============================\n");

  // 1단계: 주문 접수 (액션)
  const receivedOrder = receiveOrder(order);

  console.log("\n⏳ 병렬 작업 시작 (로봇들이 동시에 작업)...\n");

  // 2단계: 병렬 처리 - 3개의 독립적인 타임라인
  // 타임라인을 조정하여 모두 완료될 때까지 대기
  const [dough, sauce, cheese] = await waitForAll([
    makeDough(receivedOrder),
    makeSauce(receivedOrder),
    grateCheese(receivedOrder),
  ]);

  console.log("\n✨ 모든 재료 준비 완료! 이제 순차 작업을 시작합니다.\n");

  // 3단계: 순차 처리 - 순서가 중요한 작업들
  const preparedIngredients = { dough, sauce, cheese, order: receivedOrder };

  const finishedPizza = await runInSequence(
    preparedIngredients,
    spreadDough,
    applySauce,
    addCheese,
    putInOven
  );

  // 4단계: 굽기 대기
  await waitForBaking();

  // 5단계: 서빙
  return serve({ ...finishedPizza, baked: true });
}

// ============================================
// 실행
// ============================================
console.log("💡 함수형 프로그래밍 개념 설명:");
console.log("- 데이터: pizzaOrder, ingredients (정적)");
console.log("- 계산: calculateDoughAmount 등 (순수 함수)");
console.log("- 액션: receiveOrder, makeDough 등 (시점/횟수 의존)");
console.log("- 고차 함수: waitForAll, runInSequence (타임라인 조정)\n");

makePizza(pizzaOrder)
  .then(() => console.log("\n✅ 모든 작업 완료!"))
  .catch((err) => console.error("❌ 에러 발생:", err));
```

## 느낀점

- 실무에서 적용하기 힘든 부수효과 없이 순수 함수만 사용하는 프로그래밍에 대한 내용이 아닌 순수하지 않은 함수를 잘 다룰 수 있는 방법을 같이 제시해 유용할 것 같다.
- 계층형 설계를 보며 현재 프론트엔드에서 사용하고 있는 FSD(Feature Sliced Design)가 떠올랐고, 해당 아키텍쳐를 고민하는데도 도움이 되지 않을까
