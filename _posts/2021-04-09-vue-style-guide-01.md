---
date: 2021-04-09
title: "Priority A Rules: Esstential"
categories: Javascript
tags: Vue
toc: true  
classes: wide
---

## 컴포넌트 이름은 두 가지 이상의 의미 있는 단어로 네이밍

컴포넌트 이름은 App 컴포넌트(root), Vue에서 제공하는 컴포넌트 빼고는 전부 합성어로 이루어진 네이밍으로 하는 것이 좋다.  
이런 식으로 하면 기존의 **HTML elements와의 충돌**을 피할 수 있다.(HTML elements는 전부 single word로 되어있기 때문)


```javascript
// Bad Case
Vue.component('todo', {
  // ...
})

// Bad Case
export default {
  name: 'Todo',
  // ...
}
```

```javascript
// Good Case
Vue.component('todo-item', {
  // ...
})

// Good Case
export default {
  name: 'TodoItem',
  // ...
}
```

## 컴포넌트의 data 속성은 무조건 함수형이어야 된다.

컴포넌트에 data 프로퍼티를 쓸때, 프로퍼티의는 object를 반환하는 함수여야한다. 만약 data 프로퍼티가 함수가 아니라 그냥 object였을 때는 그 컴포넌트가 재사용 되었을 때 같은 데이터 object를 참조하기 때문에 독립된 data를 가질 수 없다.

```javascript
//Bad Case
Vue.component('some-comp', {
  data: {
    foo: 'bar'
  }
})
```

```javascript
//Good Case
Vue.component('some-comp', {
  data: function () {
    return {
      foo: 'bar'
    }
  }
})
```

## Props 정의는 최대한 디테일하게 정의되어야 한다.

Props를 정의할 때 최소한 타입을 정의할 수 있어야한다.

```javascript
// Bad Case! This is only OK when prototyping
props: ['status']
```
  
Props를 자세하게 정의하였을 때 두가지 이점이 있다고 스타일 가이드에 나와있다.

1. Props 코드 자체가 문서가 되어서 사용 방법을 알려준다.
2. 개발 시에 Vue는 잘못된 format의 props가 들어왔을 때 **경고** 해줄 것이고, 잠재적인 에러를 잡도록 도와줄 것이다.

```javascript
//Good Case
props: {
  status: String
}

//Good Case
props: {
  status: {
    type: String,
    required: true,
    validator: function (value) {
      return [
        'syncing',
        'synced',
        'version-conflict',
        'error'
      ].indexOf(value) !== -1
    }
  }
}
```