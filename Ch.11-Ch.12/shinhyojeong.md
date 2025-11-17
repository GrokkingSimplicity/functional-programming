# 12. 함수형 반복

## reduce로 할 수 있는 것들 - 실행 취소/실행 복귀(p.313) 
```
const actions = [
  { type: 'ADD_TEXT', payload: 'Hello' },
  { type: 'ADD_TEXT', payload: 'World' },
  { type: 'DELETE_LAST_WORD' },
  { type: 'ADD_TEXT', payload: '!' }
];

function textReducer(state, action) {
  switch(action.type) {
    case 'ADD_TEXT':
      return `${state} ${action.payload}`;
    case 'DELETE_LAST_WORD':
      return state.split(' ').slice(0, -1).join(' ');
    default:
      return state;
  }
}

// 특정 시점까지의 상태 계산
function getStateAtIndex(actions, index) {
  return actions.slice(0, index + 1).reduce(textReducer, '');
}

// 사용 예시
let currentIndex = actions.length - 1;

console.log('현재 상태:', getStateAtIndex(actions, currentIndex));
// "Hello !"

// Undo
currentIndex--;
console.log('Undo:', getStateAtIndex(actions, currentIndex));
// "Hello"

// Redo
currentIndex++;
console.log('Redo:', getStateAtIndex(actions, currentIndex));
// "Hello !"
```

