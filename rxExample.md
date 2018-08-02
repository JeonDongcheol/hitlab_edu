## The introduction to Reactive Programming you've been missing
(by [@andrestaltz](https://twitter.com/andrestaltz))

- - -

## Thinking in RP, with examples

RP에서 생각하는 방법에 대한 가이드가 있는 실제 예시. JavaScript로 예제 구현(RxJS)

## Implementing a "Who to follow" suggestions box

Twitter의 다른 계정을 보여주는 부분을 따라한 예제

![Twitter Who to follow suggestions box](http://i.imgur.com/eAlNb0j.png)

다음과 같은 기능을 구현

1. Github의 user API에서 3개의 계정 데이터 로드
2. 'Refresh' 버튼 클릭 시 다른 3개의 계정 로드
3. 계정 별로 있는 'x' 버튼을 누르면 해당 자리에 기존 계정이 없어지고 다른 계정이 다시 로드
4. 각 계정들에 대해서는 아바타(사진)와 해당 계정의 Github 링크를 표시하고 클릭하면 해당 페이지로 이동

[Github API for getting users](https://developer.github.com/v3/users/#get-all-users).

해당 예제에 대한 전체 소스 : http://jsfiddle.net/staltz/8jFJH/48/

## 요청 및 응답

**시작시 API에서 3개의 계정 데이터 로드**

1. 요청하기(Doing a Request)
2. 응답받기(Getting a Response)
3. 응답하기(Rendering a Response)

1개의 데이터 스트림으로 모델링하면, 한 개의 출력값이 포함된 스트림이 된다.

```
--a------|->

여기에서 a는 문자열 'https://api.github.com/users'
```

요청 이벤트 발생 시 이벤트가 발생한 때(When)와 요청된 값(What : 여기서는 URL 문자열)이 포함된다.

하나의 값으로 스트림을 생성하는 것은 Rx*에서는 매우 간단하다. 스트림에 대한 용어는 Observable이다.(여기서는 stream으로 부른다.)

```javascript
var requestStream = Rx.Observable.just('https://api.github.com/users');
```

위에 코드는 문자열의 흐름에 불과하며 다른 동작 및 연산을 수행하지 않기 때문에 값을 내보낼 때 어떤 이벤트가 발생해야한다. 그렇게 하기 위해선 아래와 같이 subscribe 를 사용한다.

```javascript
requestStream.subscribe(function(requestUrl) {
  // execute the request
  jQuery.getJSON(requestUrl, function(responseData) {
    // ...
  });
}
```

Rx는 비동기 데이터 스트림을 처리하기 위한 것이고, 그 방법은 다음과 같다.

```javascript
requestStream.subscribe(function(requestUrl) {
  // execute the request
  var responseStream = Rx.Observable.create(function (observer) {
    jQuery.getJSON(requestUrl)
    .done(function(response) { observer.onNext(response); })
    .fail(function(jqXHR, status, error) { observer.onError(error); })
    .always(function() { observer.onCompleted(); });
  });
  
  responseStream.subscribe(function(response) {
    // do something with the response
  });
}
```

[`Rx.Observable.create()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservablecreatesubscribe) 는 명시적으로 데이터 이벤트 혹은 에러에 대해서 Observer 혹은 Subscriber에게 알려줌으로써 커스텀 스트림을 생성 가능하게 한다.

Observable은 Promise++이다. Rx는 Promise를 Observable로 `var strea = Rx.Observable.fromPromise(promise)`를 사용해서 쉽게 변환하게 해준다. Observable은 [Promises/A+](http://promises-aplus.github.io/promises-spec/) 와 호환이 되는 것은 아니지만, 개념적으로는 충돌이 일어나지는 않는다. Promise는 단순히 한 개의 출력 값을 가지고 있는 Observable이다. Rx 스트림은 많은 반환 값을 허용함으로써 Promise가 가지고 있는 한계를 뛰어 넘는다. 그렇기 때문에 Observable은 최소한 Promises 만큼 좋다. 이러한 점에서 Observable은 최소한 Promises 만큼 효율이 좋다.

하나의 `subcribe()` 콜을 다른 `subscribe()` 안에 가지고 있으면 많은 콜백을 사용하는 것이고, `responseStream` 의 생성은 `requestStream` 와 관련이 되어 있다. Rx에는 외부에서 새로운 스트림을 변형하고 생성하는 간단한 매커니즘이 있고, 그것을 사용해야 한다.

[`map(f)`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemapselector-thisarg) 은 스트림 A에서 각 값들을 가져오고, f에 적용한다. 그리고 스트림 B에 값을 생성한다. 요청 및 응답 스트림에서 map(f)를 수행하면 요청 URL을 응답 Promise에 매핑이 가능하다.

```javascript
var responseMetastream = requestStream
  .map(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });
```

메타 스트림(metastream)은 각 출력 값이 또 다른 스트림인 스트림으로 쉽게 [pointers](https://en.wikipedia.org/wiki/Pointer_(computer_programming)) 로 생각할 수 있다. 즉, 출력 값은 다른 스트림에 대한 포인터이다. 이 예에서는 각 요청 URL은 해당 응답이 포함된 Promise 스트림에 대한 포인터에 매핑이 된다.

![Response metastream](http://i.imgur.com/HHnmlac.png)

[Flatmap](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypeflatmapselector-resultselector) 은 map() 버전 중 하나로 'branch' 스트림에서 출력되는 모든 것을 'trunk' 스트림에서 방출하여 메타 스트림을 단순하게 해준다. Rx에서 비동기 응답을 처리하기 위한 도구이다. 

```javascript
var responseStream = requestStream
  .flatMap(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });
```

![Response stream](http://i.imgur.com/Hi3zNzJ.png)

Response 스트림은 Request 스트림에 의해 정의되기 때문에, 만약에 추후에 Request 스트림에서 이벤트가 발생한다고 하면, 이에 상응하는 응답 이벤트를 Response 스트림에서 발생시켜야 한다.

```
requestStream:  --a-----b--c------------|->
responseStream: -----A--------B-----C---|->

(lowercase is a request, uppercase is its response)
```

응답 스트림을 받고 받은 데이터 렌더링하기.

```javascript
responseStream.subscribe(function(response) {
  // render `response` to the DOM however you wish
});
```

위의 코드의 종합

```javascript
var requestStream = Rx.Observable.just('https://api.github.com/users');

var responseStream = requestStream
  .flatMap(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });

responseStream.subscribe(function(response) {
  // render `response` to the DOM however you wish
});
```

## The refresh button

API를 사용하면 페이지 크기가 아닌 페이지 오프셋을 지정할 수 있기 때문에 객체 3개만 사용한다. 하지만 응답에는 100명의 사용자 목록이 들어간다.(100명의 사용자 목록을 가져온 JSON 응답 중에서 3명만 사용)

Refresh 버튼이 클릭될 때마다 Request 스트림은 새로운 URL을 발생시켜야하고, 그렇게 해야지 새로운 Response를 가져올 수 있다. 필요한 두 가지
1. Refresh 버튼에서 클릭 이벤트들의 스트림
2. Request 스트림을 Refresh 클릭 스트림에 따라서 바꿔야 함.

RxJS에는 event listener들로부터 Observable을 만드는 툴이 있다. 그 예시는 다음과 같다.

```javascript
var refreshButton = document.querySelector('.refresh');
var refreshClickStream = Rx.Observable.fromEvent(refreshButton, 'click');
```

Refresh 클릭 이벤트만으로는 API URL을 전달하지 않기 때문에 실제 URL을 매핑해야한다. 임의의 오프셋 매개 변수로 API 끝에 매핑된 새로 고침 클릭 스트림으로 요청 스트림을 만들어준다.

```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

위와 같은 방법을 적용하면 시작할 때는 요청이 발생하지 않고 Refresh를 클릭할 때만 요청이 발생하기 때문에, 시작할 때와 Refresh를 할 때의 두 가지 동작에 대해서 요청을 발생시켜야한다.

```javascript
var requestOnRefreshStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
  
var startupRequestStream = Rx.Observable.just('https://api.github.com/users');
```

[`merge()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemergemaxconcurrent--other) 를 통해서 두 기능을 통합할 수 있다.

```
stream A: ---a--------e-----o----->
stream B: -----B---C-----D-------->
          vvvvvvvvv merge vvvvvvvvv
          ---a-B---C--e--D--o----->
```

코드는 다음과 같다.

```javascript
var requestOnRefreshStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
  
var startupRequestStream = Rx.Observable.just('https://api.github.com/users');

var requestStream = Rx.Observable.merge(
  requestOnRefreshStream, startupRequestStream
);
```

더 깔끔하게 중간 스트림 없이 코드를 구성한 결과.

```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  })
  .merge(Rx.Observable.just('https://api.github.com/users'));
```

여기서 더 짧고 읽기 쉽게 다음과 같이 가능하다.

```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  })
  .startWith('https://api.github.com/users');
```

[`startWith()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypestartwithscheduler-args) 입력 스트림이 어떤 형태를 나타내더라고 startWith(x)의 결과로 나오는 출력 스트림은 처음부터 `x` 가 된다. `refreshClickStream` 다음에 `startWith()` 를 사용하고 안에 'startup click' 요소를 추가함으로써 시작할 때 바로 Refresh 클릭 이벤트가 동작하도록 해서 두 가지 동작을 모두 수행할 수 있다.

```javascript
var requestStream = refreshClickStream.startWith('startup click')
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

결국 최종적으로 추가한 요소는 `startWith()` 를 기존 코드에서 추가한 것이다.

## Modelling the 3 suggestions with streams

지금까지는 Response 스트림에서 발생하는 렌더링 단계에서 Suggestion UI 요소만 `subscribe()` 를 통해서 다루었다. 하지만 Refresh 버튼을 클릭하면 현재 3개의 계정에 대해서 지워지지 않는다. 새로운 스트림이 왔을 때 기존의 계정들을 제거해 주어야한다. 즉, Refresh 버튼을 클릭했을 때 기존의 계정들을 제거하고 새로 가져와야 한다.

```javascript
refreshClickStream.subscribe(function() {
  // clear the 3 suggestion DOM elements 
});
```

위의 방법은 DOM 요소에 영향을 미치는 두 개의 subscriber가 있기 때문에 Seperation of concern이 아니다.

![Mantra](http://i.imgur.com/AIimQ8C.jpg)

따라서 Suggestion을 스트림으로 따로 모델을 해야한다. 여기서 출력된 값은 suggestion 데이터를 포함하는 JSON 객체이다. 3개의 suggestion을 각각 작업 해준다.

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // get one random user from the list
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  });
```

responseStream의 subscribe()에서 렌더링을 수행하는 대신 아래와 같이 작업을 해준다.

```javascript
suggestion1Stream.subscribe(function(suggestion) {
  // render the 1st suggestion to the DOM
});
```

refresh 버튼 클릭을 null suggestion 데이터와 매핑하고, 이것을 suggestion1Stream에 다음과 같이 포함시킨다.

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // get one random user from the list
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  })
  .merge(
    refreshClickStream.map(function(){ return null; })
  );
```

그리고 렌더링할 때, `null` 은 'No Data'로 해석해 UI 요소를 숨긴다.

```javascript
suggestion1Stream.subscribe(function(suggestion) {
  if (suggestion === null) {
    // hide the first suggestion DOM element
  }
  else {
    // show the first suggestion DOM element
    // and render the data
  }
});
```

그림으로 나타내면 다음과 같다.

```
refreshClickStream: ----------o--------o---->
     requestStream: -r--------r--------r---->
    responseStream: ----R---------R------R-->   
 suggestion1Stream: ----s-----N---s----N-s-->
 suggestion2Stream: ----q-----N---q----N-q-->
 suggestion3Stream: ----t-----N---t----N-t-->
```

여기서 `N` 은 `null` 이다.

추가로 시작할 때 '비어있는' suggestion을 `startWith(null)`을 suggestion 스트림에 추가함으로써 렌더링할 수 있다.

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // get one random user from the list
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  })
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

```
refreshClickStream: ----------o---------o---->
     requestStream: -r--------r---------r---->
    responseStream: ----R----------R------R-->   
 suggestion1Stream: -N--s-----N----s----N-s-->
 suggestion2Stream: -N--q-----N----q----N-q-->
 suggestion3Stream: -N--t-----N----t----N-t-->
```

## Closing a suggestion and using cached responses

각각의 Suggestion에는 현재 계정을 닫고 그 자리에 다른 것을 로드하기 위한 'x'버튼이 있어야한다.

```javascript
var close1Button = document.querySelector('.close1');
var close1ClickStream = Rx.Observable.fromEvent(close1Button, 'click');
// and the same for close2Button and close3Button

var requestStream = refreshClickStream.startWith('startup click')
  .merge(close1ClickStream) // we added this
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

That does not work. It will close and reload _all_ suggestions, rather than just only the one we clicked on. There are a couple of different ways of solving this, and to keep it interesting, we will solve it by reusing previous responses. The API's response page size is 100 users while we were using just 3 of those, so there is plenty of fresh data available. No need to request more.

Again, let's think in streams. When a 'close1' click event happens, we want to use the _most recently emitted_ response on `responseStream` to get one random user from the list in the response. As such:

```
    requestStream: --r--------------->
   responseStream: ------R----------->
close1ClickStream: ------------c----->
suggestion1Stream: ------s-----s----->
```

In Rx* there is a combinator function called [`combineLatest`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypecombinelatestargs-resultselector) that seems to do what we need. It takes two streams A and B as inputs, and whenever either stream emits a value, `combineLatest` joins the two most recently emitted values `a` and `b` from both streams and outputs a value `c = f(x,y)`, where `f` is a function you define. It is better explained with a diagram:

```
stream A: --a-----------e--------i-------->
stream B: -----b----c--------d-------q---->
          vvvvvvvv combineLatest(f) vvvvvvv
          ----AB---AC--EC---ED--ID--IQ---->

where f is the uppercase function
```

We can apply combineLatest() on `close1ClickStream` and `responseStream`, so that whenever the close 1 button is clicked, we get the latest response emitted and produce a new value on `suggestion1Stream`. On the other hand, combineLatest() is symmetric: whenever a new response is emitted on `responseStream`, it will combine with the latest 'close 1' click to produce a new suggestion. That is interesting, because it allows us to simplify our previous code for `suggestion1Stream`, like this:

```javascript
var suggestion1Stream = close1ClickStream
  .combineLatest(responseStream,             
    function(click, listUsers) {
      return listUsers[Math.floor(Math.random()*listUsers.length)];
    }
  )
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

One piece is still missing in the puzzle. The combineLatest() uses the most recent of the two sources, but if one of those sources hasn't emitted anything yet, combineLatest() cannot produce a data event on the output stream. If you look at the ASCII diagram above, you will see that the output has nothing when the first stream emitted value `a`. Only when the second stream emitted value `b` could it produce an output value.

There are different ways of solving this, and we will stay with the simplest one, which is simulating a click to the 'close 1' button on startup:

```javascript
var suggestion1Stream = close1ClickStream.startWith('startup click') // we added this
  .combineLatest(responseStream,             
    function(click, listUsers) {l
      return listUsers[Math.floor(Math.random()*listUsers.length)];
    }
  )
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

## Wrapping up

전체 코드는 다음과 같이 구성한다.

```javascript
var refreshButton = document.querySelector('.refresh');
var refreshClickStream = Rx.Observable.fromEvent(refreshButton, 'click');

var closeButton1 = document.querySelector('.close1');
var close1ClickStream = Rx.Observable.fromEvent(closeButton1, 'click');
// and the same logic for close2 and close3

var requestStream = refreshClickStream.startWith('startup click')
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });

var responseStream = requestStream
  .flatMap(function (requestUrl) {
    return Rx.Observable.fromPromise($.ajax({url: requestUrl}));
  });

var suggestion1Stream = close1ClickStream.startWith('startup click')
  .combineLatest(responseStream,             
    function(click, listUsers) {
      return listUsers[Math.floor(Math.random()*listUsers.length)];
    }
  )
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
// and the same logic for suggestion2Stream and suggestion3Stream

suggestion1Stream.subscribe(function(suggestion) {
  if (suggestion === null) {
    // hide the first suggestion DOM element
  }
  else {
    // show the first suggestion DOM element
    // and render the data
  }
});
```
