# 타임라인 조율하기

## 좋은 타임라인의 원칙

1. 타임라인은 적을수록 이해하기 쉽다
2. 타임라인은 짧을수록 이해하기 쉽다.
3. 공유하는 자원이 적을수록 이해하기 쉽다.
4. 자원을 공유한다면 서로 조율해야한다.
5. 시간을 일급으로 다룬다.

- 스레드나 비동기 호츨, 서버 요청을 줄여 타임라인 수를 줄이고, 타임라인에 있는 액션을 줄이고 액션을 계산으로 바꿔 암묵적인 입/출력을 업애 타임라인 길이를 줄이고, 가능한 단일 스레드에서 공유 자원에 접근하도록 하고 공유 자우너을 없애고, 큐나 락을 사용해 안전하게 동시성 기본형으로 자원을 공유하며, promise나 컷과 같은것으로 액션에 순서나 반복을 제한해 동시성 기본형으로 조율한다.

# 반응형 아키텍쳐와 어니언 아키텍쳐

## 반응형 아키텍쳐

- 어플리케이션을 구조화하는 방법으로 핵심 원칙은 이벤트에 대한 반응으로 일어날 일일 지정하는것
- 코드에 나타난 순차적 액션의 순서를 뒤집는다.

1. 원인과 효과가 결합한것을 분리한다.
2. 여러단계를 파이프 라인으로 처리한다.
3. 순서를 표한하는 방법을 뒤집으면 타임라인이 유연해진다.

```js
//`useReactiveState`가 반응형의 핵심. 상태가 바뀌면 자동으로 로깅/API 호출이 실행된다.

import { useState, useEffect } from "react";

// 상태 변경을 스트림처럼 다루는 커스텀 훅
function useReactiveState(initialValue) {
  const [state, setState] = useState(initialValue);

  // 상태 변경 시 자동으로 구독된 효과 실행
  useEffect(() => {
    // 효과1: 로깅
    console.log("State changed:", state);

    // 효과2: API 호출 (상태가 5 이상일 때)
    if (state.count >= 5) {
      fetchUserData(state.count);
    }
  }, [state]);

  return [state, setState];

  //순수 함수
  // 1. DATA (불변 상태)
  const [cart, setCart] = useState([]);
  const [total, setTotal] = useState(0);

  // 2. CALCULATION (순수 함수 - React가 자동 실행)
  const calculatedTotal = useMemo(
    () => cart.reduce((sum, item) => sum + item.price, 0),
    [cart]
  );

  const calculatedTax = useMemo(() => calculatedTotal * 0.1, [calculatedTotal]);

  // 3. ACTION (핸들러 - 유일한 사이드 이펙트)
  const addToCart = (item) => {
    setCart((prev) => [...prev, item]); // ✅ 이것만!
  };

  // 파이프라인은 React가 자동 처리
  function ShoppingCart() {
    return (
      <div>
        <button onClick={() => addToCart({ name: "Pizza", price: 10000 })}>
          ➕ Add Pizza
        </button>

        {/* React가 자동 계산/DOM 업데이트 */}
        <div>합계: {calculatedTotal.toLocaleString()}원</div>
        <div>세금: {calculatedTax.toLocaleString()}원</div>
      </div>
    );
  }
}

function Counter() {
  const [count, setCount] = useReactiveState({ count: 0 });

  return (
    <div>
      <p>Count: {count.count}</p>
      <button onClick={() => setCount({ ...count, count: count.count + 1 })}>
        Increment
      </button>
    </div>
  );
}
```

```js
//입력 + 타이머 이벤트를 스트림으로 만들어 병합하고, `debounceTime`으로 성능 최적화. 입력만 바뀌어도 자동으로 검색 결과가 업데이트된다.

import React, { useEffect, useState } from "react";
import { View, Text, TextInput, FlatList } from "react-native";
import {
  fromEvent,
  merge,
  debounceTime,
  distinctUntilChanged,
  switchMap,
} from "rxjs";
import { fromRef } from "rxjs/internal/observable/fromRef"; // React Native용

function SearchScreen() {
  const [results, setResults] = useState([]);

  useEffect(() => {
    // 입력 스트림 + 타이머 스트림 병합
    const inputRef = React.createRef();
    const searchInput$ = fromEvent(inputRef.current, "onChangeText").pipe(
      debounceTime(300),
      distinctUntilChanged()
    );

    const timer$ = interval(5000).pipe(
      // 5초마다 새로고침
      map(() => "refresh")
    );

    // 스트림들 병합 → API 호출
    const subscription = merge(searchInput$, timer$)
      .pipe(
        switchMap((value) => fetchUsers(value)), // 검색어 또는 'refresh'
        catchError(() => of([]))
      )
      .subscribe(setResults);

    return () => subscription.unsubscribe();
  }, []);

  return (
    <View>
      <TextInput ref={inputRef} placeholder="Search users..." />
      <FlatList
        data={results}
        renderItem={({ item }) => <Text>{item.name}</Text>}
        keyExtractor={(item) => item.id}
      />
    </View>
  );
}
```

- react 에서 state 관리시 자동으로 업데이트 되는 방식이나, useEffect, context API, 글로벌 state 관리 라이브러리, Rxjs 와 같은 툴로 적용하는 방식이 될 수 있음

## 어니언 아키텍쳐

- 반응형 아키텍쳐가 코드에 나타난 순차적 액션의 순서를 뒤집어 효과와 그 효과에 대한 원인을 분석에 코드에 복잡하게 꼬인 부분을 푼다면, 어니언 아키텍쳐는 웹서비스나 온도조절 장치같은 현실 세계와 상호작용 하기 위한 서비스 구조를 만든다.

1. 인터렉션 계층

- 바깥 세상에 영항을 주거나 받는 액션

2. 도메인 계층

- 비즈니스 규칙을 정의하는 계산
  리애

3. 언어 계층

- 언어 유틸리티와 라이브러리

### 리액트에서의 어니언 아키텍쳐

- 순수 Onion보단 FSD + Domain Layer 조합이 더 많다. Domain 폴더만 만들어서 핵심 비즈니스 로직 분리
- 특히 RN의 경우 네이티브 모듈 + API 연동이 많아서 Infrastructure 계층 분리가 더 유용

```js
src/
├── domain/           # 2. Domain (가장 내부)
│   ├── entities/
│   │   └── User.ts
│   └── services/
│       └── UserDomainService.ts
├── application/      # 3. Application
│   └── usecases/
│       └── GetUserListUseCase.ts
├── infrastructure/   # 1. Infrastructure (가장 외부)
│   └── api/
│       └── userApi.ts
└── presentation/     # 4. Presentation
    └── components/
        └── UserList.tsx

```

1. Domain (순수 - 의존성 0개)

```ts
// domain/entities/User.ts
export class User {
  constructor(public id: string, public name: string) {}

  // 도메인 규칙 (순수)
  updateName(newName: string) {
    if (newName.length < 2) throw new Error("Too short");
    return new User(this.id, newName);
  }
}

// domain/services/UserDomainService.ts
export class UserDomainService {
  static calculateTotalValue(users: User[]): number {
    return users.reduce((sum, u) => sum + u.value, 0);
  }
}
```

2.  Application (UseCase - Domain만 의존)

```ts
// application/usecases/GetUserListUseCase.ts
import { UserRepository } from "../../domain/repositories/UserRepository";
import { User } from "../../domain/entities/User";

export class GetUserListUseCase {
  constructor(private userRepo: UserRepository) {}

  async execute(): Promise<User[]> {
    const users = await this.userRepo.getAll();
    return users.map((u) => new User(u.id, u.name));
  }
}
```

3. Infrastructure (Repository 구현체 - 외부 기술)

```ts
// infrastructure/api/userApi.ts
export class UserRepositoryImpl implements UserRepository {
  async getAll(): Promise<UserDto[]> {
    const res = await fetch("/api/users");
    return res.json();
  }
}
```

4. Presentation (React 컴포넌트 - UseCase만 호출)

```ts
// presentation/components/UserList.tsx
function UserList() {
  const [users, setUsers] = useState<User[]>([]);

  const getUsers = useCallback(async () => {
    // ✅ Presentation → Application → Domain → Infrastructure
    const usecase = new GetUserListUseCase(new UserRepositoryImpl());
    const result = await usecase.execute();
    setUsers(result);
  }, []);

  return (
    <div>
      <button onClick={getUsers}>Load Users</button>
      {users.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

- 다음은 **“순수 FSD”**에 가깝지만, `entities/`와 `model/`이 Onion Domain 개념을 섞은 형태

```js

src/
├── app/                 # ✅ FSD: 앱 설정 (라우팅, Provider)
├── pages/               # ✅ FSD: 페이지 슬라이스
├── features/
│   └── user/
│       ├── ui/          # ✅ FSD: presentation (컴포넌트)
│       ├── model/       # ✅ FSD model + Onion Domain
│       └── api/         # ✅ FSD api + Onion Infrastructure
├── entities/            # ✅ Onion Domain (공통 Entity)
└── shared/              # ✅ FSD: 공통 UI/유틸


```
