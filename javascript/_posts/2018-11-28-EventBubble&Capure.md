---
layout: post
title: 이벤트 캡쳐링(capturing)과 버블링(bubbling)
---

최근에 vanilla js로 mvc pattern을 연습하는 중, 이벤트 캡쳐링과 버블링에 대해 습득한 지식을 정리하기위해 해당 포스트를 작성합니다.

# 이벤트 핸들링(Event handleing)

Event Target에 대하여 지정한 이벤트가 발생될 때마다, 호출할 함수를 등록하여 이벤트를 핸들링 합니다. 이 때 EventTarget은 일반적으로 Element, Document, Widnow 이지만 XMLHttpRequest 처럼 이벤트를 지원 하는 모든객체가 될수 있습니다.

DOM Level 0 에서 이벤트는 다음과 같이 등록하였습니다.
```javascript
window.onload = function() {
    // handler1
};
```

해당 방법은 Event Target에 Event당 하나의 handler만 등록가능하였기때문에, 둘이상의 handler를 호출하려면 다음과 같은 방식으로 코드를 작성해야 했습니다.

```javascript
function handler1() {
  // someting
}

function handler2() {
  // someting
}

window.onload = function() {
    handler1();
    handler2();
};
```

모던 브라우저(IE 9 이상, 크롬, 사파리,파이어폭스 등) 에서는 DOM Level 2 이상을 지원하기 때문에 addEventListener를 이용하여 이벤트 리스너를 등록하여 사용 가능 합니다.

```javascript

function windowOnLoadHandler1(){
  // something
}

function windowOnLoadHandler2(){
  // something
}

window.addEventListener("load", windowOnLoadHandler1)
window.addEventListener("load", windowOnLoadHandler2)
```

앞선 방식과 다르게 다중 handler를 등록함에 있어서 문제가 없습니다. 다만 동일 핸들러를 같은 이벤트의 핸들러로 등록 할 경우 중복된 항목이 버려지게 되어 한번만 동작 하게 되는데, 만약 다중 핸들러를 호출할 필요가 있다면 익명함수로 한번 랩핑하여 사용 하시면 됩니다.

앞서 설명드린 addEventListener에는 세번째 매개변수로 boolean 또는 객체를 사용할수 있습니다.

boolean 형식의 값을 넣을경우에는 useCapture로 caputuring 여부를 결정합니다. 기본 값은 false 입니다.
객체를 사용할 경우에는 event option 으로 사용 되는데 사용 가능 한 옵션은 다음과 같습니다. 
- capture : 이벤트 캡쳐링 여부를 나타내는 boolean.
- once : 이벤트를 한번만 사용할지 여부를 boolean. true로 설정할경우 리스너는 한번만 사용되고 삭제됩니다.
- passive : 리스너의 함수가 preventDefault()를 호출하지 않음을 나타내는 boolean입니다. 해당기능을 이용하여 성능을 향상할수 있습니다.



## 이벤트 캡쳐링(capturing)과 버블링(bubbling) 

이벤트 캡쳐링이랑 발생 된 이벤트의 전파 방향이 DOM tree 상단에서 하단(부모에서 자식)으로 이동하는 것을 뜻합니다. 버블링은 그 반대의 개념으로 DOM tree 하단에서 상단 즉 자식 엘리먼트 부터 부모로 이동하는것을 뜻한다. 캡쳐링과 버블링이 이벤트 전파 방향이 어떤지 직접 확인해보자. 

![이벤트의 전파 방향](/assets/img/caputre&bubble.png)




### capturing
{% raw %}
<iframe width="100%" height="350" src="//jsfiddle.net/7qa0hxdu/embedded/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>
{% endraw %}

### bubbling
{% raw %}
<iframe width="100%" height="350" src="//jsfiddle.net/Ledm1f3w/embedded/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>
{% endraw %}


## 이벤트 위임 (Event delegate)

vanilla js로 SPA를 컨트롤 할때, 고정된 요소가 아니라 동적으로 생성 & 삭제가 빈번하게 일어 나는 요소(주로 리스트)마다 이벤트를 bind 할 때 매번 이벤트를 등록하고 삭제시 해제하는 것은 바람직 하지 않아 보입니다. 이 때 이벤트 위임을 통하여 다음과 같은 문제를 해결 할 수 있습니다. 

{% raw %}
<iframe width="100%" height="300" src="//jsfiddle.net/38ojw9Lh/embedded/js,html,result/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>
{% endraw %}

### caputre parameter를 사용하는 이유
일부 이벤트는 bubble을 발생시키지 않습니다. 예를 들어 blur 같은 경우는 이벤트 위임 시 capture를 사용해야만 예상대로 동작합니다. (다만 blur의 경우에는 focusout 이벤트를 사용하면 굳이 capture를 사용하지 않아도 됩니다. 자세한 것은 MDN 문서참조). 만약 이벤트 위임이 정상적으로 이루어지지 않는다면 MDN에서 해당 이벤트에 대한 정보를 확인해 보는 것이 좋습니다.

# Refrence

- https://developer.mozilla.org/ko/docs/Web/API/EventTarget/addEventListener
- https://openhome.cc/eGossip/JavaScript/DOMLevel2EventModel.html


