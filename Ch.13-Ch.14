# 13. 함수형 도구 체이닝
> <i>인자로 받은 값을 그대로 리턴하는 함수를 **항등 함수**(identity function)라고 합니다. 아무 일도 하지 않지만 아무것도 하지 않아야 할 때 유용하게 쓸 수 있습니다. (p.323)</i>

용어가 생소해서 좀 더 알아봤다. 조건부 처리할 때 명시적으로 코드를 작성하기 위해 많이 사용하고 있는 방식이었다.

```
const processors = {
  'uppercase': str => str.toUpperCase(),
  'lowercase': str => str.toLowerCase(),
  'none': str => str
}
```

<br/>


> <i>map()과 filter(), reduce() 체인을 최적화하는 것을 **스트림 결합(stream fusion)**이라고 합니다.(p. 311)</i>

자주 사용하던 방식이나 용어가 생소해서 정리해봤다. 

<br/>

> <i>다시 말하지만 지금 하는 일은 최적화입니다. 병목이 생겼을 때만 쓰는 것이 좋고 대부분의 경우에는 여러 단계를 사용하는 것이 더 명확하고 읽기 쉽습니다. (p.331)</i>

개인적으로 한번에 처리할 수 있는 일들은 reduce 하나로 해결하는 편이었는데, 최적화를 위한 것이 아니면 명시적으로 분리된 단계로 작성해야겠다는 생각이 들었다.

<br/>

> <i>함수형 도구는 배열 전체를 다룰 때 잘 동작합니다. 배열 일부에 대해 동작하는 반복문이 있다면 배열 일부를 새로운 배열로 나눌 수 있습니다. (p.339)</i>

일부분을 다루는 함수는 splice, slice 등을 통해 새로운 배열을 만들어 함수형으로 다룰 것 ! 


<br/>

> <i>frequenciesBy()와 groupBy(), 개수를 세거나 그룹화하는 일은 종종 쓸모가 있습니다. 이 함수는 객체 도는 맵을 리턴합니다. (p.341) </i>

``` 
// frequenciesBy - 특정 기준으로 개수 세기
function frequenciesBy(array, keyFn) {
  return array.reduce((acc, item) => {
    const key = keyFn(item);
    acc[key] = (acc[key] || 0) + 1;
    return acc;
  }, {});
}

// groupBy - 특정 기준으로 그룹화
function groupBy(array, keyFn) {
  return array.reduce((acc, item) => {
    const key = keyFn(item);
    if (!acc[key]) {
      acc[key] = [];
    }
    acc[key].push(item);
    return acc;
  }, {});
}

```

위 함수는 데이터 구조에 영향을 많이 받는 함수라 공통 함수로 빼서 사용할 생각은 못했던 것 같다. 공통 함수로 관리하고 추상화 단계를 둬서 사용하면 좋을 것 같다. 




<br/>

# 14.중첩된 데이터에 함수형 도구 사용하기

실무에서 자주 중첩된 구조를 다루는데 보통 단번에 중첩된 객체로 접근하기 때문에 update 함수처럼 공통화를 생각하지 못했다.
기본적인 동작 과정에서도 함수형 도구를 사용해서 공통화하고, 재귀를 통해 반복 작업들을 수행하는 방법을 알게됐다.
고민이 되는 점은 실제로 가독성 측면에서 재귀를 통해 작성하는 방법이 나은지 의문이 들었다 🤔
