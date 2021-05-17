---
layout: post
title: Javascript Lecture 복습 - 01 
author: "Paycis"
tags: [javascript]
---

자바스크립트 강의를 복습차원에서 작성하는 포스트입니다.<br>
잘못된 부분이 있거나, 문제가 되는 부분 알려주시면 수정하거나 포스팅 삭제하겠습니다.

# 프로그래밍/생각할것/학습방법

1. 코드 또한 글의 한 유형. 글쓴이(개발자)가 어떤 생각을 갖고 글을 작성했는지 이해하는 것이 중요
2. 반복, 반복, 반복
    * 기본예제를 응용해보고 발생하는 에러를 해결하며 실력을 쌓는 과정 필요 (지름길 없음)
    * 구글링을 통해 찾은 코드를 붙여넣어 에러를 해결할 경우 나중에 다른 문제가 발생하면 해결하기 어려워 진다
3. 시대에 따라 프로그래밍 언어의 매커니즘(문법), 사상, 제약조건(환경, 규모 등)이 변하기 때문에 항상 염두해야 한다

<hr>

# Web FrontEnd Programming

가장 빠르게 변화하는 분야이며 웹표준이 존재하지만 정의가 명확하지 않은 부분들이 있어 React JS, Vue JS와 같은 도구들이 관심을 받고 많은 사람들이 사용하고 있다. 각각의 도구들을 만든 개발자들은 특정문제(ex. 도구의 기능)를 해결하기 위해 각자의 철학을 갖고 접근한다. 그래서 React, Vue 와 같은 도구들은 같은문제를 해결하는 방법이 다를 수 있다.
 
### HTML의 변화
1. Document : 최초 형태
2. Dynamic Document : 서비스,컨텐츠 형태 제공 (시장증가에 따라 탄생)
3. Web Service : 2000년대 초 등장
4. Web Application : 2007년쯤 등장
5. Web Component : 웹표준, 정의가 미비한 부분 존재
6. Cross Platform Application : 웹이 아닌 다른곳에서도 JS로 개발이 가능해짐 (ex. React-Native)

### 도구들
배포를 도와주는 Bundler (ex. WebPack) 나 Babel과 같이 브라우저 호환성을 고려하여 최신 버전의 스크립트를 하위 버전으로 변환해주는 다양한 도구들이 존재한다

* Babel : 최신스펙의 자바스크립트 코드를 브라우저 호환성을 고려하여 이전스펙의 자바스크립트 코드로 변환해 주는 도구 
* CoffeeScript(Transpiler) : 다른언어로 작성한 코드를 자바스크립트 코드로 변환해주는 도구
* Webpack(Bundler) : 배포 도구

<hr>

# Javascript 데이터, 상태
프로그래밍 언어는 크게 2가지로 구분할 수 있는데, `선언형 언어`와 `명령형 언어`로 구분할 수 있다. `선언형 언어`의 경우 한번 코드를 작성하면 항상 정해진대로 동작하게 되는데 HTML/CSS가 선언형 언어에 속한다. 반면 `명령형 언어`는 상태 (값의 변화)에 따라 동작과 결과가 달라질 수 있는 언어이며 javascrpit도 명령형 언어입니다.

### 값의 종류
자바스크립트에서 값으로 인식하는 종류는 아래와 같이 존재한다.

1. 숫자(Number) : 정수 / 실수 / 무한대(Infinity) / NaN(Not a Number)
    {% highlight js  linenos %}
    10; 10.321; Infinity; NaN;
    //Not a Number는 해당값이 숫자가 아님을 나타냄
    {% endhighlight %}

2. 문자열(String) : 문자열을 나타내는 여러 방법 존재
    {% highlight js  linenos %}
    '안녕하세요'; //요즘에는 ""보단 ''를 많이 사용한다
    "안녕하세요";
    `안녕하세요`; //ES6, 문자열 내부에 변수(값)을 포함해서 작성할 수 있다.
    {% endhighlight %}

3. 논리값(Boolean) : true, false

4. 정의되지 않은 값(undefined) : 값을 담을 바구니(?)를 준비해놓고 값을 담기 전 상태. (값이 담겨있지않아 값의 종류를 알 수 없어)
    {% highlight js  linenos %}
    let name; //undefined 상태
    name = 'paycis'; //string으로 값의 종류가 정해짐
    {% endhighlight %}

5. 값이 없는(null) : 변수(바구니)를 만들고 `null`이라는 빈 값을 할당 한 경우
    * null은 객체형 데이터(값)의 빈값을 의미
    * `undefined`는 <ins>값이 정해지지않은 상태</ins> 이고 `null`은 속이 비어있는 객체형 데이터
    
6. 심볼(Symbol)
7. 객체(Object) : 여러 값들을 하나의 관계로 묶어둔 형태

### 값의 저장
위에서 **바구니**라는 표현을 쓰고 `변수`라고 말했는데 자바스크립트의 모든 값은 `변수`라는 공간에 저장할 수 있다. 특정한 값을 계속 사용하기 위해선 `변수`라는 공간에 담아둬야 자바스크립트가 해당 값이 어디있는지 알고 가져다 쓸 수 있다.

* var : ES5.0 하위버전부터 사용되던 형태, 설계상 오류가 존재하며, 이로인해 다양한 오류를 야기함

* let : ES6.0에서 생긴 개념으로 var의 문제를 개선하였으며 작동 매커니즘이 var와 다름

자바스크립트는 값들의 계산(ex. 덧셈, 뺄셈)을 할 때 값들의 종류를 보고 어떤 값의 유형으로 맞출지 판단하여 결과를 만들어 낸다.

{% highlight js  linenos %}
let score=99;
let message='당신의 점수 : ';

console.log(score+1); // 100
console.log(message + score ); //당신의 점수 : 99
// number + string = string
{% endhighlight %}

### 값 그룹 짓기
앞에서 객체를 <ins>여러 값들을 하나의 관계로 묶어둔 형태</ins>라고 했다. 예를 들어 **사람**이라는 객체를 만든다고 하면 아래와 같이 만들 수 있다.
{% highlight js  linenos %}
let person = {
    name: '김할배',
    age: 99,
    hobby: 'Coding',
    address: 'Korea'
}

//name, age, hobby, address 는 객체의 요소(key) 라고 부른다
//그리고 각 key들은 각각의 값(value)를 갖는다
{% endhighlight %}

앞에서 변수는 자바스크립트에서 말하는 모든 값을 담을 수 있다고 했고, 값의 종류에는 객체가 있다고 했습니다. 그렇기 때문에 객체 또한 변수에 담을 수 있어 person이라는 변수에 객체를 담을 수 있습니다.<br>
조금 더 생각해보면 객체의 요소(key)도 변수이기 때문에 객체를 담을 수 있습니다.

