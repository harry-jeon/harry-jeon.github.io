---
date: 2021-06-29
title: "Vue 중급강좌 Refactoring챕터에서 배운 점"
categories: Javascript
tags: Vue
toc: true  
classes: wide
---

## Vue Anti Pattern

요구사항은 할일 관리 리스트에서 체크 표시를 클릭하여 그 아이템의 completed 속성을 toggle해주는 기능이다.
이 때 상위 컨테이너 컴포넌트인 App.vue에서 List 컴포넌트로 props로 todoItems 내려준다.
그럼 List에서는 토글버튼이 선택되는 아이템을 담아서 이벤트를 emit하는데,

여기서 emit 이벤트 버스로 받은 아이템의 속성을 직접 수정하게 되는데

```javascript
toggleComplete: function(item, index) {
    item.completed = !item.completed
}
```
이러면 부모에서 내려준 데이터를 자식이 또 올리고 부모가 그걸 수정하게 되는 현상이 발생한다.
이걸 강사님은 **안티패턴** 이라고 하셨다. 그래서 좀 조사를 해볼건데... 어라 하나가 아니라 여러개네 ㅠㅠ

안티 패턴이라는 용어가 이 패턴을 가리키는 게 아니라 그냥 피해야할 패턴들을 말할 떄 안티 패턴이라고 하나보다... 칼을 뽑은 김에 무를 뽑아보자는 심정으로 하나만 알아봤다.

## Side effects inside computed properties

Computed 속성은 뷰에서 제공하는 편리한 기능 중 하나이지만, 이 속성은 다른 state에 의존해서 어떤 값을 **보여주는** 곳에만 사용되어야한다. 다른 메소드를 부르거나, 무슨 properties를 assign하는 일을 computed 안에서 하면 뭔가 잘못 돌아갈 가능성이 많다.


``` javascript
export default {
    data() {
        return {
            array: [1,2,3]
        };
    },
    computed: {
        reversedArray() {
            return this.array.reverse(); // SIDE EFFECT - mutates a data property
        }
    }
}
```
=> reverse 함수가 원래 오리지널 array를 건드리고 있기 떄문에 무한 업데이트가 되고 있는 걸 볼 수 있다.


``` javascript
export default {
  props: {
    order: {
      type: Object,
      default: () => ({})
    }
  },
  computed:{
    grandTotal() {
      let total = (this.order.total + this.order.tax) * (1 - this.order.discount);
      this.$emit('total-change', total)
      return total.toFixed(2);
    }
  }
}
```
=> taxes 랑 discount를 포함한 토탈 prices를 보여주기 위해 computed 하나를 만들었다. 여기서 total을 계산해서 total-change 이벤트를 부모에 emit한다. 그리고 우리는 스페셜 고객에게 10 퍼센트의 추가 디스카운트를 하고 싶어서 아래 코드를 넣었다.

```html
<price-details :order="order"
               @total-change="totalChange">
</price-details>
```
```javascript
export default {
  // other properties which are not relevant in this example
  methods: {
    totalChange(grandTotal) {
      if (this.isSpecialCustomer) {
        this.order = {
          ...this.order,
          discount: this.order.discount + 0.1
        };
      }
    }
  }
};
```

=> 이런 식으로 부모에서 order를 넘기고 그 order에 따라 grandTotal을 계산하고(computed) 그 안에서 이벤트를 emit한다. 근데 부모에서 emit받은 핸들러에서 다시 order를 고치는 현상 때문에 무한 루프가 일어날 수 있다. 더 중요한 건, 이 경우에는 평소에는 디버깅에 안걸리지만 스페셜 고객(매우 드문 경우)에만 버그로 잡히기 때문에 개발자가 쉽게 알아차리지 못할 가능성이 높다는 것!(그래서 어떻게 해결하란거지 ..)

## 그래서 computed를 어떻게 하라는 건데?

vue style guide에서 computed에 대한 내용은 최대한 **simple** 하게 개발되어야 된다고 나와있다.(strongly recommended)

```javascript
// bad case
computed: {
  price: function () {
    var basePrice = this.manufactureCost / (1 - this.profitMargin)
    return (
      basePrice -
      basePrice * (this.discountPercent || 0)
    )
  }
}

//good case
computed: {
  basePrice: function () {
    return this.manufactureCost / (1 - this.profitMargin)
  },
  discount: function () {
    return this.basePrice * (this.discountPercent || 0)
  },
  finalPrice: function () {
    return this.basePrice - this.discount
  }
}
```


## 포스팅 요약

1. 부모 컴포넌트에서 넘겨준 자식이 다시 올려준 데이터를 직접 수정하지 말고 부모가 가진 데이터로 수정하자. (this.todoItems[index]를 고치자, 자식에서 넘어온 todoItem을 고치지 말고!)
2. computed 속성에서는 진짜 다른 state를 이용해서 뭔갈 보여주기만 해야지, 다른 짓(메소드호출, 값 assign)을 하면 안된다.

