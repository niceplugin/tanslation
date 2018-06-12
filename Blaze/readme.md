# 목차

**개요**
* [개요](#개요)
  * [시작하기](#시작하기)
  * [세부사항](#세부사항)
  * [예제](#예제)
  * [신조](#신조)
  * [페키지](#페키지)
  * [우리의 계획](#우리의-계획)
  
**가이드**
* [소개](#소개)
* [Spacebars templates](#Spacebars-templates)
* [Blaze에서 사용가능한 컨포넌트](#Blaze에서-사용가능한-컨포넌트)
* [Blaze로 스마트 컴포넌트 작성하기](#Blaze로-스마트-컴포넌트-작성하기)
* [Blaze에서 코드 재사용하기](#Blaze에서-코드-재사용하기)
* [Blaze 이해하기](#Blaze-이해하기)
* [라우터](#라우터)

**API**
* [Templates](#Templates)
* [Blaze](#Blaze)
* [Spacebars](#Spacebars)

# 개요

Blaze는 반응형 HTML 템플릿으로 UI를 만드는 강력한 라이브러리 입니다.
전통적인 템플릿과 jQuery을 조합하여 사용하는 것과 비교하여, Blaze는 데이터의 변경사항을 감지하고 DOM을 조작할 "업데이트 로직"이 앱 내에 필요없습니다.
대신 [트레커](https://docs.meteor.com/api/tracker.html)의 "반응형"이나 [미니몽고](https://docs.meteor.com/api/collections.html)의 데이터베이스 커서를 이용하여 {{#if}}나 {{#each}}와 같은 익숙한 문법으로 DOM을 조작합니다.

## 시작하기

현재 Blaze는 Meteor 전용 페키지 입니다.
조만간 npm에서 Blaze를 만나볼 수 있으며, 당신의 작업물에 이것을 사용할 수 있습니다.

새로만든 각각의 Meteor 프로젝트에 Blaze가 설치되어 있습니다. (`blaze-html-templates`라는 페키지 명으로 되어 있음)

## 세부사항

원문:

Blaze has two major parts:

* A template compiler that compiles template files into JavaScript code that runs against the Blaze runtime library.
Moreover, Blaze provides a compiler toolchain (think LLVM) that can be used to support arbitrary template syntaxes.
The flagship template syntax is Spacebars, a variant of Handlebars, but a community alternative based on Jade is already in use by many apps.

* A reactive DOM engine that builds and manages the DOM at runtime, invoked via templates or directly from the app, which features reactively updating regions, lists, and attributes; event delegation; and many callbacks and hooks to aid the app developer.

Blaze is sometimes compared to frameworks like React, Angular, Ember, Polymer, Knockout, and others by virtue of its advanced templating system.
What sets Blaze apart is a relentless focus on the developer experience, using templating, transparent reactivity, and interoperability with existing libraries to create a gentle learning curve while enabling you to build world-class apps.

번역:

Blaze에는 두 개의 주요사항이 있습니다:

* Blaze는 이 라이브러리가 실행될 때 템플릿 파일을 자바스크립트 코드로 컴파일 하는 컴파일러 입니다.
또한 Blaze는 임의 템플릿 구문을 사용할 수 있도록 컴파일러 툴체인(LLVM이라고도 함)을 제공합니다.
기본적인 템플릿 구문은 Handlebars에서 파생된 Spacebars라는 구분을 사용하며, 이미 많은 앱들이 Jade를 기반으로 하는 템플릿 대용으로 많이 사용합니다.

* 반응형 DOM 엔진은 템플릿이나 엡에 의해 호출될 때 실행됩니다.
앱 개발자는 수많은 콜백과 후크를 이용하여 특정부분, 리스트, 속성이 대표적인 반응적인 업데이트 할 수 있습니다.

Blaze의 진보된 템플릿 시스템은 때때로 React, Angular, Ember, Polymer, Knockout과 같은 프레임 워크들과 비교됩니다.
Blaze는 템플릿 사용, 직관적인 반응형, 기존 사용중인 라이브러리와의 상호 운용성으로 개발자 경험을 끊임없이 집중하여 글로벌적인 수준의 앱 제작을 할 수 있게 합니다.

## 예제

"Leaderboard"라는 예제에 두개의 스페이스바 템플릿이 있습니다.
이 템플릿은 플레이어와 점수에 대한 리스트를 표시합니다.

```html
<template name="leaderboard">
  <ol class="leaderboard">
    {{#each players}}
      {{> player}}
    {{else}}
      {{> no_players}}
    {{/each}}
  </ol>
</template>

<template name="player">
  <li class="player {{selected}}">
    <span class="name">{{name}}</span>
    <span class="score">{{score}}</span>
  </li>
</template>

<template name="no_players">
  <li class="no-players">
    No players available
  </li>
</template>
```

템플릿 내에 `{{name}}`와 `{{score}}`는 데이터 컨텍스트(현재 플레이어)이며, `players`와 `selected`는 헬퍼 함수입니다.
헬퍼 함수와 이벤트 헨들러는 자바스크립트에서 정의됩니다:

```javascript
Template.leaderboard.helpers({
  players: function () {
    // 미니몽고의 Players라는 컬렉션에 쿼리문을 수행하여 반응적인 결과를 얻습니다.
    return Players.find({}, { sort: { score: -1, name: 1 } });
  }
});

Template.player.events({
  'click': function () {
    // 플레이어를 선택하여 클릭할 경우
    Session.set("selectedPlayer", this._id);
  }
});

Template.player.helpers({
  selected: function () {
    return Session.equals("selectedPlayer", this._id) ? "selected" : '';
  }
});
```

원문:

No additional UI code is necessary to ensure that the list of players stays up-to-date, or that the "selected" class is added and removed from the LI elements as appropriate when the user clicks on a player.

Thanks to a powerful template language, it doesn't take much ceremony to write a loop, include another template, or bind an attribute (or part of an attribute).
And thanks to Tracker's transparent reactivity, there's no ceremony around depending on reactive data sources like the database or Session; it just happens when you read the value, and when the value changes, the DOM will be updated in a fine-grained way.

번역:

플레이어 목록을 최신 상태로 유지하거나 사용자가 플레이어를 클릭 할 때 적절한 "LI"요소에서 "selected"클래스가 추가되고 제거되도록하기 위해 추가 UI 코드가 필요하지 않습니다.

고맙게도 강력한 템플릿 언어 덕분에 루프를 작성하거나, 다른 템플릿을 포함하거나 속성을 바인딩 하는 등의 구차한 작업이 필요하지 않습니다.
Tracker의 직관적인 반응형 덕분에 데이터베이스나 세션을 반응형으로 만들기위한 추가 작업이 필요없이 단순히 값을 읽어올때 변경된 값이 있으면 알아서 DOM을 업데이트 합니다.


## Principles

### 쉬운 학습

원문:

Many factors go into making Blaze easy to pick up and use, including the other principles below.
In general, we prefer APIs that lead to simple and obvious-looking application code, and we recognize that developers have limited time and energy to learn new and unfamiliar terms and syntaxes.

It may sound obvious to "keep it simple" and prioritize the developer experience when creating a system for reactive HTML, but it's also challenging, and we think it's not done often enough! We use feedback from the Meteor community to ensure that Blaze's features stay simple, understandable, and useful.

번역:

Blaze를 시작하기 위해 많은 개념이나 용어를 알 필요는 없습니다.
웹 개발자로서 우리는 이미 HTML, CSS, JavaScript 기술을 공부하는 학생입니다.
Blaze는 어떤 새로운 서적 없이도 기존의 지식을 적용할 수 있습니다.

일반적으로 우리는 단순명료하게 사용할 수 있는 API를 선호하며, 새로운 익숙하지 않은 용어와 구문을 익히기 위해 시간과 노력을 들이는것이 현실적으로 힘들다는 것을 알고 있습니다.

반응형 HTML 시스템을 만들때 "간단하게"유지할 수 있도록 개발자 경험을 우선적으로 고려하고 있습니다.
우리는 Meteor 커뮤니티의 피드백을 통해 더욱 노력할 것입니다.

### 직관적인 반응형

Blaze는 [Tracker](https://docs.meteor.com/api/tracker.html)라는 라이브러리를 사용하여 템플릿의 헬퍼가 재 계산 할 때 자동적으로 추적합니다.
예를들어 클라이언트 측 데이터베이스에서 헬퍼가 어떠한 값을 읽었다면, 값이 변경될 때 헬퍼는 자동적으로 재계산합니다.

이것이 의미하는 바는, DOM 업데이트를 할 시기를 명시적으로 선언할 필요가 없이 데이터 바인딩만 하면 된다는 것입니다.
반응형이 무엇인지 Tracker가 어떻게 동작하는지 몰라도 됩니다.
결론적으로 다른 접근방식 보다 고민할거리와 타이핑할것이 줄어든다는 것입니다.

### 깔끔한 템플릿

Blaze는 다른 프레임워크를 사용한 개발자도 깨끗하고 읽기 쉽도록 Handlebars 및 Jade와 같은 익숙하고 대중적인 템플릿 구문을 사용합니다.

좋은 템플릿 언어는 "템플릿 지시문"이 HTML로부터 명확하게 구분되어야 하며 HTML의 구조에 따라 제약성을 가져서는 안됩니다.
이러한 조건은 템플릿을 쉽게 익히고, 읽기 쉽고, CSS로 쉽게 스타일링 하고, DOM과 쉽게 연관시킬 수 있게 합니다.

반대로, 일부 최신 프레임 워크에서는 템플릿을 HTML로 다시 작성하거나(Angular, Polymer) JavaScript로만 대체하려고 합니다(React).

이러한 접근방식은 템플릿의 구조나 실제 DOM과 그렇지 않은 요소 또는 둘 다를 모호하게 만듭니다.
또한 템플릿은 일반적으로 최적화된 방법으로 컴파일되어 있으므로 원시템플릿 소스코드를 브라우저에서 분석 할 수 있는지 여부는 중요하지 않습니다.
그러나 템플릿을 읽고, 쓰고, 유지관리하는 개발자 경험은 대단히 중요합니다.

### 플러그인 상호 운용성

웹 개발자는 종종 HTML, JavaScript, CSS로 만든 작은 결과물을 서로 공유하거나 라이브러리, 위젯 또는 jQuery 플러그인으로 게시합니다.
또는 동영상,지도 및 기타 제 3자 콘텐츠를 퍼가려고합니다.

미번역 부분:

Blaze doesn't assume it owns the whole DOM, and it tries to make as few assumptions as possible about the DOM outside of its updates.
It hooks into jQuery's clean-up routines to prevent memory leaks, and it preserves classes, attributes, and styles added to elements by jQuery or any third-party library.

While it's certainly possible for Blaze and jQuery to step on each other's toes if you aren't careful, there are established patterns for keeping the peace, and Blaze developers rightfully expect to be able to use the various widgets and enhancements cooked up by the broader web community in their apps.

### 다른 라이브러리와의 비교

Blaze는 Backbone 또는 다른 라이브러리보다 훨씬 적은 재 렌더링을 수행하며, 한 페이지에서 여러개의 템플릿이 독립적으로 업데이트 될 수 없는 "nested view" 문제로 고심하지 않아도 됩니다.
또한 Blaze는 트래커를 사용하여 다시 렌더링 해야 할 시기를 자동으로 결정합니다.

원문:

Compared to Ember, Blaze offers finer-grained, automatic DOM updates.
Because Blaze uses Tracker's transparent reactivity, you don't have to perform explicit "data-binding" to get data into your template, or declare the data dependencies of each template helper.

번역:

Blaze는 Ember 보다 정교하고 자동적인 DOM 업데이트를 제공합니다.
Blaze는 트레커의 직관적인 반응성을 사용하기 때문에 "데이터 바인딩"을 사용하여 템플릿으로 데이터를 가져오거나 각 템플릿 헬퍼에 종속성을 선언 할 필요가 없습니다.

Blaze는 Angular 및 Polymer 보다 쉽게 학습할 수 있으며 개념이 단순하고 템플릿 문법과 HTML 구분이 깔끔하여 더 좋습니다.
또한 Blaze는 "미래지향적 브라우저"가 아닌 현시점의 브라우저를 대상으로 설계되었습니다.

원문:

Compared to React, Blaze emphasizes HTML templates rather than JavaScript component classes.
Templates are more approachable than JavaScript code and easier to read, write, and style with CSS.
Instead of using Tracker, React relies on a combination of explicit "setState" calls and data-model diffing in order to achieve efficient rendering.

번역:

JavaScript 코드로 구성요소를 짜내는 React와 비교하여 Blaze는 HTML템플릿을 강조합니다.
JavaScript 코드보다 HTML템플릿이 가독성, 수정, CSS수정 등이 더 쉽습니다.
React는 트레커를 사용하지 않으므로 대신 효율적인 렌더링을 구현하기 위해 "setState" 호출 및 다른 데이터 모델 조합에 의존합니다.

## 페키지

* blaze
* blaze-tools
* html-tools
* htmljs
* spacebars
* spacebars-compiler

*****

> 역주: 2017년 5월 이후로 프로젝트 팀이 해체되어 더이상 진행되지 않으므로 아래의 "우리의 계획"섹션은 번역하지 아니함.

## 우리의 계획

### Components (구성요소)

Blaze는 재사용 가능한 UI 컴포넌트를 만들기 위한 더 좋은 패턴을 얻을 것입니다.
템플릿은 이미 재사용 가능한 구성 요소로 제공됩니다.
앞으로의 개선사항은 다음에 중점을 둡니다:

* Argument handling (역주: 매개변수의 값 핸들링)
* Local reactive state (지역적인 반응형 상태)
* "Methods" that are callable from other components and have side effects, versus the current "helpers" which are called from the template language and are "pure"
* Scoping and the lookup chain
* Inheritance and configuration (상속과 구성)

### 양식

원본:

Most applications have a lot of forms, where input fields and other widgets are used to enter data, which must then be validated and turned into database changes.
Server-side frameworks like Rails and Django have well-honed patterns for this, but client-side frameworks are typically more lacking, perhaps because they are more estranged from the database.

Meteor developers have already found ways and built packages to deal with forms and validation, but we think there's a great opportunity to make this part of the core, out-of-the-box Meteor experience.

번역:

대부분의 앱에는 입력 필드 및 기타 위젯을 사용하여 데이터를 입력하는 양식이 많습니다.
그런 다음 유효성을 검사하고 데이터베이스에 있는 정보를 변경합니다.
Rails나 Django와 같은 서버 측 프레임워크는 이를 위해 잘 정돈 된 패턴을 가지고 있지만 클라이언트 측 프레임워크는 일반적으로 데이터베이스와 거리가 멀기때문에 잘 정돈 된 패턴이 부족합니다.

Meteor 개발자는 이미 양식과 유효성 검사를 처리 할 수 있는 방법과 패키지를 찾았지만(?), Meteor의 핵심 부분의 기능을 구현할 수있는 좋은 기회라 생각합니다.

### 모바일과 애니메이션

원문:

Blaze will cater to the needs of the mobile web, including enhanced performance and patterns for touch and other mobile interaction.

We'll also improve the ease with which developers can integrate animated transitions into their apps.

번역:

Blaze는 터치 및 기타 모바일 상호 작용을 위해 향상된 성능 및 패턴을 내포하여 모바일 웹의 요구를 충족시켜줍니다.

또한 개발자가 애니메이션 전환을 앱에서 할 수 있도록 편의성을 향상시킬 것 입니다.

### 템플릿 내에서 JavaScript 표현방법

We plan to support JavaScript expressions in templates.
This will make templates more expressive, and it will further shorten application code by eliminating the need for a certain class of one-line helpers.

The usual argument against allowing JavaScript expressions in a template language is one of "separation of concerns" -- separating business logic from presentation, so that the business logic may be better organized, maintained, and tested independently.
Meanwhile, even "logicless" template languages often include some concessions in the form of microsyntax for filtering, querying, and transforming data before using it.
This special syntax (and its extension mechanisms)
must then be learned.

While keeping business logic out of templates is indeed good policy, there is a large class of "presentation logic" that is not really separable from the concerns of templates and HTML, such as the code to calculate styles and classes to apply to HTML elements or to massage data records into a better form for templating purposes.
In many cases where this code is short, it may be more convenient or more readable to embed the code in the template, and it's certainly better than evolving the template syntax in a direction that diverges from JavaScript.

Because templates are already precompiled to JavaScript code, there is nothing fundamentally difficult or inelegant about allowing a large subset of JavaScript to be used within templates (see e.g.
the project
Ractive.js).

### Other Template Enhancements

Source maps for debugging templates.
Imagine seeing your template code in the browser's debugger! Pretty slick.

True lexical scoping.

Better support for pluggable template syntax (e.g.
Jade-like templates).
There is already a Jade package in use, but we should learn from it and clarify the abstraction boundary that authors of template syntaxes are programming against.

### Pluggable Backends (don't require jQuery)

While Blaze currently requires jQuery, it is architected to run against other "DOM backends" using a common adaptor interface.
You should be able to use Zepto, or some very small shim if browser compatibility is not a big deal for your application for some reason.
At the moment, no such adaptors besides the jQuery one have been written.

The Blaze team experimented with dropping jQuery and talking directly to "modern browsers," but it turns out there is about 5-10K of code at the heart of jQuery that you can't throw out even if you don't care about old browsers or supporting jQuery's app-facing API, which is required just to bring browsers up to the modest expectations of web developers.

### Better Stand-alone Support

Blaze will get better support for using it outside of Meteor, such as regular stand-alone builds.

## 리소스

* [Templates API](../api/templates.html)
* [Blaze API](../api/blaze.html)
* [Spacebars syntax](../api/spacebars.html)

> 역주: 원본 깃허브에서도 이 링크는 모두 끊겨있는 상태이다.

# 소개

Meteor의 프론트 엔드 렌더링 시스템 인 Blaze를 사용하여 사용 가능하고 관리 가능한 사용자 인터페이스를 구축하는 방법.

이 가이드를 읽으면 다음을 알 수 있습니다.

1. Spacebars 언어를 사용하여 Blaze 엔진으로 템플릿을 렌더링하는 방법.
2. Blaze에서 재사용 가능한 컴포넌트를 작성하는 모범 사례.
3. Blaze 렌더링 엔진이 내부적으로 작동하는 방식과 이를 사용하는 몇 가지 고급 기술.
4. Blaze 템플릿을 테스트하는 방법.

Blaze는 Meteor에 내장 된 반응형 렌더링 라이브러리입니다.
일반적으로 템플릿은 [Spacebars](#Spacebars-templates)로 작성되며, Meteor의 반응형 시스템인 [Tracker](https://github.com/meteor/meteor/tree/devel/packages/tracker)를 활용하도록 설계된 [Handlebars](http://handlebarsjs.com)의 변형입니다.
이러한 템플릿은 Blaze 라이브러리에서 렌더링되어 JavaScript UI 컴포넌트(구성요소)로 컴파일됩니다.

원문:

Blaze is not required to build applications in Meteor---you can also easily use [React](http://react-in-meteor.readthedocs.org/en/latest/) or [Angular](http://www.angular-meteor.com) to develop your UI. However, this particular article will take you through best practices in building an application in Blaze, which is used as the UI engine in all of the other articles.

번역:

Blaze는 Meteor에서 앱으로 빌드 할 필요가 없으며 [React](http://react-in-meteor.readthedocs.org/en/latest/)나 [Angular](http://www.angular-meteor.com)를 사용하여 UI를 쉽게 개발할 수 있습니다.
그러나 이 섹션에서는 Blaze에서 앱을 빌드하는 모범 사례를 안내합니다.
다른 모든 섹션에서 UI 엔진으로 사용합니다.

# Spacebars templates

Spacebars is a handlebars-like templating language, built on the concept of rendering a reactively changing *data context*. Spacebars templates look like simple HTML with special "mustache" tags delimited by curly braces: `{% raw %}{{ }}{% endraw %}`.

As an example, consider the `Todos_item` template from the Todos example app:

```html
<template name="Todos_item">
  <div class="list-item {{checkedClass todo}} {{editingClass editing}}">
    <label class="checkbox">
      <input type="checkbox" checked={{todo.checked}} name="checked">
      <span class="checkbox-custom"></span>
    </label>

    <input type="text" value="{{todo.text}}" placeholder="Task name">
    <a class="js-delete-item delete-item" href="#">
      <span class="icon-trash"></span>
    </a>
  </div>
</template>
```

This template expects to be rendered with an object with key `todo` as data context (we'll see [below](../guide/reusable-components.html#Validate-data-context) how to enforce that). We access the properties of the `todo` using the mustache tag, such as `{% raw %}{{todo.text}}{% endraw %}`. The default behavior is to render that property as a string; however for some attributes (such as `checked={% raw %}{{todo.checked}}{% endraw %}`) it can be resolved as a boolean value.

Note that simple string interpolations like this will always escape any HTML for you, so you don't need to perform safety checks for XSS.

Additionally we can see an example of a *template helper*---`{% raw %}{{checkedClass todo}}{% endraw %}` calls out to a `checkedClass` helper defined in a separate JavaScript file. The HTML template and JavaScript file together define the `Todos_item` component:

```js
Template.Todos_item.helpers({
  checkedClass(todo) {
    return todo.checked && 'checked';
  }
});
```

In the context of a Blaze helper, `this` is scoped to the current *data context* at the point the helper was used. This can be hard to reason about, so it's often a good idea to instead pass the required data into the helper as an argument (as we do here).

Apart from simple interpolation, mustache tags can be used for control flow in the template. For instance, in the `Lists_show` template, we render a list of todos like this:

```html
  {{#each todo in todos}}
    {{> Todos_item (todoArgs todo)}}
  {{else}}
    <div class="wrapper-message">
      <div class="title-message">No tasks here</div>
      <div class="subtitle-message">Add new tasks using the field above</div>
    </div>
  {{/each}}
```

This snippet illustrates a few things:

 - The `{% raw %}{{#each .. in}}{% endraw %}` block helper which repeats a block of HTML for each element in an array or cursor, or renders the contents of the `{% raw %}{{else}}{% endraw %}` block if no items exist.
 - The template inclusion tag, `{% raw %}{{> Todos_item (todoArgs todo)}}{% endraw %}` which renders the `Todos_item` component with the data context returned from the `todosArg` helper.

You can read about the full syntax [in the Spacebars](../api/spacebars.html). In this section we'll attempt to cover some of the important details beyond just the syntax.

## Data contexts and lookup

We've seen that `{% raw %}{{todo.title}}{% endraw %}` accesses the `title` property of the `todo` item on the current data context. Additionally, `..` accesses the parent data context (rarely a good idea), `list.todos.[0]` accesses the first element of the `todos` array on `list`.

Note that Spacebars is very forgiving of `null` values. It will not complain if you try to access a property on a `null` value (for instance `foo.bar` if `foo` is not defined), but instead simply treats it also as null. However there are exceptions to this---trying to call a `null` function, or doing the same *within* a helper will lead to exceptions.

## Calling helpers with arguments

You can provide arguments to a helper like `checkedClass` by simply placing the argument after the helper call, as in: `{% raw %}{{checkedClass todo true 'checked'}}{% endraw %}`. You can also provide a list of named keyword arguments to a helper with `{% raw %}{{checkedClass todo noClass=true classname='checked'}}{% endraw %}`. When you pass keyword arguments, you need to read them off of the `hash` property of the final argument. Here's how it would look for the example we just saw:

```js
Template.Todos_item.helpers({
  checkedClass(todo, options) {
    const classname = options.hash.classname || 'checked';
    if (todo.checked) {
      return classname;
    } else if (options.hash.noClass) {
      return `no-${classname}`;
    }
  }
});
```

Note that using keyword arguments to helpers is a little awkward, so in general it's usually easier to avoid them. This feature was included for historical reasons to match the way keyword arguments work in Handlebars.

You can also pass the output of a helper to a template inclusion or other helper. To do so, use parentheses to show precedence:

```html
{{> Todos_item (todoArgs todo)}}
```

Here the `todo` is passed as argument to the `todoArgs` helper, then the output is passed into the `Todos_item` template.

## Template inclusion

You "include" a sub-component with the `{% raw %}{{> }}{% endraw %}` syntax. By default, the sub-component will gain the data context of the caller, although it's usually a good idea to be explicit. You can provide a single object which will become the entire data context (as we did with the object returned by the `todoArgs` helper above), or provide a list of keyword arguments which will be put together into one object, like so:

```html
{{> subComponent arg1="value-of-arg1" arg2=helperThatReturnsValueOfArg2}}
```

In this case, the `subComponent` component can expect a data context of the form:

```js
{
  arg1: ...,
  arg2: ...
}
```

## Attribute Helpers

We saw above that using a helper (or data context lookup) in the form `checked={% raw %}{{todo.checked}}{% endraw %}` will add the checked property to the HTML tag if `todo.checked` evaluates to true. Also, you can directly include an object in the attribute list of an HTML element to set multiple attributes at once:

```html
<a {{attributes}}>My Link</a>
```

```js
Template.foo.helpers({
  attributes() {
    return {
      class: 'A class',
      style: {background: 'blue'}
    };
  }
});
```

## Rendering raw HTML

Although by default a mustache tag will escape HTML tags to avoid [XSS](https://en.wikipedia.org/wiki/Cross-site_scripting), you can render raw HTML with the triple-mustache: `{% raw %}{{{ }}}{% endraw %}`.

```html
{{{myHtml}}}
```

```js
Template.foo.helpers({
  myHtml() {
    return '<h1>This H1 will render</h1>';
  }
});
```

You should be extremely careful about doing this, and always ensure you aren't returning user-generated content (or escape it if you do!) from such a helper.

## Block Helpers

A block helper, called with `{% raw %}{{# }}{% endraw %}` is a helper that takes (and may render) a block of HTML. For instance, we saw the `{% raw %}{{#each .. in}}{% endraw %}` helper above which repeats a given block of HTML once per item in a list. You can also use a template as a block helper, rendering its content via the `Template.contentBlock` and `Template.elseBlock`. For instance, you could create your own `{% raw %}{{#if}}{% endraw %}` helper with:

```html
<template name="myIf">
  {{#if condition}}
    {{> Template.contentBlock}}
  {{else}}
    {{> Template.elseBlock}}
  {{/if}}
</template>

<template name="caller">
  {{#myIf condition=true}}
    <h1>I'll be rendered!</h1>
  {{else}}
    <h1>I won't be rendered</h1>
  {{/myIf}}
</template>
```

## Built-in Block Helpers

There are a few built-in block helpers that are worth knowing about:

### If / Unless

The `{% raw %}{{#if}}{% endraw %}` and `{% raw %}{{#unless}}{% endraw %}` helpers are fairly straightforward but invaluable for controlling the control flow of a template. Both operate by evaluating and checking their single argument for truthiness. In JS `null`, `undefined`, `0`, `''`, `NaN`, and `false` are considered "falsy", and all other values are "truthy".

```html
{{#if something}}
  <p>It's true</p>
{{else}}
  <p>It's false</p>
{{/if}}
```

### Each-in

The `{% raw %}{{#each .. in}}{% endraw %}` helper is a convenient way to step over a list while retaining the outer data context.

```html
{{#each todo in todos}}
  {{#each tag in todo.tags}}
    <!-- in here, both todo and tag are in scope -->
  {{/each}}
{{/each}}
```

### Let

The `{% raw %}{{#let}}{% endraw %}` helper is useful to capture the output of a helper or document subproperty within a template. Think of it just like defining a variable using JavaScript `let`.

```html
{{#let name=person.bio.firstName color=generateColor}}
  <div>{{name}} gets a {{color}} card!</div>
{{/let}}
```

Note that `name` and `color` (and `todo` above) are only added to scope in the template; they *are not* added to the data context. Specifically this means that inside helpers and event handlers, you cannot access them with `this.name` or `this.color`. If you need to access them inside a helper, you should pass them in as an argument (like we do with `(todoArgs todo)` above).

### Each and With

There are also two Spacebars built-in helpers, `{% raw %}{{#each}}{% endraw %}`, and `{% raw %}{{#with}}{% endraw %}`, which we do not recommend using (see [prefer using each-in](../guide/reusable-components.html#Prefer-lt-￼16-gt)). These block helpers change the data context within a template, which can be difficult to reason about.

Like `{% raw %}{{#each .. in}}{% endraw %}`, `{% raw %}{{#each}}{% endraw %}` iterates over an array or cursor, changing the data context within its content block to be the item in the current iteration. `{% raw %}{{#with}}{% endraw %}` simply changes the data context inside itself to the provided object. In most cases it's better to use `{% raw %}{{#each .. in}}{% endraw %}` and `{% raw %}{{#let}}{% endraw %}` instead, just like it's better to declare a variable than use the JavaScript `with` keyword.

## Chaining of Block Helpers
 
You can chain block helpers:

```html
{{#input isRadio}}
  <input type="radio" />
{{else input isCheckbox}}
  <input type="checkbox" />
{{else}}
  <input type="text" />
{{/foo}}
```

This is equivalent to:

```html
{{#input isRadio}}
  <input type="radio" />
{{else}}
  {{#input isCheckbox}}
    <input type="checkbox" />
  {{else}}
    <input type="text" />
  {{/input}}
{{/input}}
```

## Strictness

Spacebars has a very strict HTML parser. For instance, you can't self-close a `div` (`<div/>`) in Spacebars, and you need to close some tags that a browser might not require you to (such as a `<p>` tag). Thankfully, the parser will warn you when it can't understand your code with an exact line number for the error.

## Escaping

To insert literal curly braces: `{% raw %}{{ }}{% endraw %}` and the like, add a pipe character, `|`, to the opening braces:

```
<!-- will render as <h1>All about {{</h1> -->
<h1>All about {{|</h1>

<!-- will render as <h1>All about {{{</h1> -->
<h1>All about {{{|</h1>
```

# Blaze에서 사용가능한 컨포넌트

In the [UI/UX article](https://guide.meteor.com/ui-ux.html#smart-components) we discussed the merits of creating reusable components that interact with their environment in clear and minimal ways.

Although Blaze, which is a simple template-based rendering engine, doesn't enforce a lot of these principles (unlike other frameworks like React and Angular) you can enjoy most of the same benefits by following some conventions when writing your Blaze components. This section will outline some of these "best practices" for writing reusable Blaze components.

Examples below will reference the `Lists_show` component from the Todos example app.

## Validate data context

In order to ensure your component always gets the data you expect, you should validate the data context provided to it. This is just like validating the arguments to any Meteor Method or publication, and lets you write your validation code in one place and then assume that the data is correct.

You can do this in a Blaze component's `onCreated()` callback, like so:

```js
Template.Lists_show.onCreated(function() {
  this.autorun(() => {
    new SimpleSchema({
      list: {type: Function},
      todosReady: {type: Boolean},
      todos: {type: Mongo.Cursor}
    }).validate(Template.currentData());
  });
});
```

We use an `autorun()` here to ensure that the data context is re-validated whenever it changes.

## Name data contexts to template inclusions

It's tempting to just provide the object you're interested in as the entire data context of the template (like `{% raw %}{{> Todos_item todo}}{% endraw %}`). It's better to explicitly give it a name (`{% raw %}{{> Todos_item todo=todo}}{% endraw %}`). There are two primary reasons for this:

1. When using the data in the sub-component, it's a lot clearer what you are accessing; `{% raw %}{{todo.title}}{% endraw %}` is clearer than `{% raw %}{{title}}{% endraw %}`.
2. It's more flexible, in case you need to give the component more arguments in the future.

For instance, in the case of the `Todos_item` sub-component, we need to provide two extra arguments to control the editing state of the item, which would have been a hassle to add if the item was used with a single `todo` argument.

Additionally, for better clarity, always explicitly provide a data context to an inclusion rather than letting it inherit the context of the template where it was rendered:

```html
<!-- bad: inherits data context, who knows what is in there! -->
{{> myTemplate}}

<!-- explicitly passes empty data context -->
{{> myTemplate ""}}
```

## Prefer `{% raw %}{{#each .. in}}{% endraw %}`

For similar reasons to the above, it's better to use `{% raw %}{{#each todo in todos}}{% endraw %}` rather than the older `{% raw %}{{#each todos}}{% endraw %}`. The second sets the entire data context of its children to a single `todo` object, and makes it difficult to access any context from outside of the block.

The only reason not to use `{% raw %}{{#each .. in}}{% endraw %}` would be because it makes it difficult to access the `todo` symbol inside event handlers. Typically the solution to this is to use a sub-component to render the inside of the loop:

```html
{{#each todo in todos}}
  {{> Todos_item todo=todo}}
{{/each}}
```

Now you can access `this.todo` inside `Todos_item` event handlers and helpers.

## Pass data into helpers

Rather than accessing data in helpers via `this`, it's better to pass the arguments in directly from the template. So our `checkedClass` helper takes the `todo` as an argument and inspects it directly, rather than implicitly using `this.todo`. We do this for similar reasons to why we always pass arguments to template inclusions, and because "template variables" (such as the iteratee of the `{% raw %}{{#each .. in}}{% endraw %}` helper) are not available on `this`.

## Use the template instance

Although Blaze's simple API doesn't necessarily encourage a componentized approach, you can use the *template instance* as a convenient place to store internal functionality and state. The template instance can be accessed via `this` inside Blaze's lifecycle callbacks and as `Template.instance()` in event handlers and helpers. It's also passed as the second argument to event handlers.

We suggest a convention of naming it `instance` in these contexts and assigning it at the top of every relevant helper. For instance:

```js
Template.Lists_show.helpers({
  todoArgs(todo) {
    const instance = Template.instance();
    return {
      todo,
      editing: instance.state.equals('editingTodo', todo._id),
      onEditingChange(editing) {
        instance.state.set('editingTodo', editing ? todo._id : false);
      }
    };
  }
});

Template.Lists_show.events({
  'click .js-cancel'(event, instance) {
    instance.state.set('editingTodo', false);
  }
});
```

## Use a reactive dict for state

The [`reactive-dict`](https://atmospherejs.com/meteor/reactive-dict) package lets you define a simple reactive key-value dictionary. It's a convenient way to attach internal state to a component. We create the `state` dictionary in the `onCreated` callback, and attach it to the template instance:

```js
Template.Lists_show.onCreated(function() {
  this.state = new ReactiveDict();
  this.state.setDefault({
    editing: false,
    editingTodo: false
  });
});
```

Once the state dictionary has been created we can access it from helpers and modify it in event handlers (see the code snippet above).

## Attach functions to the instance

If you have common functionality for a template instance that needs to be abstracted or called from multiple event handlers, it's sensible to attach it as functions directly to the template instance in the `onCreated()` callback:

```js
import {
  updateName,
} from '../../api/lists/methods.js';

Template.Lists_show.onCreated(function() {
  this.saveList = () => {
    this.state.set('editing', false);

    updateName.call({
      listId: this.data.list._id,
      newName: this.$('[name=name]').val()
    }, (err) => {
      err && alert(err.error);
    });
  };
});
```

Then you can call that function from within an event handler:

```js
Template.Lists_show.events({
  'submit .js-edit-form'(event, instance) {
    event.preventDefault();
    instance.saveList();
  }
});
```

## Scope DOM lookups to the template instance

It's a bad idea to look up things directly in the DOM with jQuery's global `$()`. It's easy to select some element on the page that has nothing to do with the current component. Also, it limits your options on rendering *outside* of the main document (see testing section below).

Instead, Blaze gives you a way to scope a lookup to within the current template instance. Typically you use this either from a `onRendered()` callback to setup jQuery plugins (called via `Template.instance().$()` or `this.$()`), or from event handlers to call DOM functions directly (called via `Template.instance().$()` or using the event handler's second argument like `instance.$()`). For instance, when the user clicks the add todo button, we want to focus the `<input>` element:

```js
Template.Lists_show.events({
  'click .js-todo-add'(event, instance) {
    instance.$('.js-todo-new input').focus();
  }
});
```

## Use `.js-` selectors for event maps

When you are setting up event maps in your JS files, you need to 'select' the element in the template that the event attaches to. Rather than using the same CSS class names that are used to style the elements, it's better practice to use classnames that are specifically added for those event maps. A reasonable convention is a class starting with `js-` to indicate it is used by the JavaScript. For instance `.js-todo-add` above.

## Passing HTML content as a template argument

If you need to pass in content to a sub-component (for instance the content of a modal dialog), you can use the [custom block helper](../guide/spacebars.html#Block-Helpers) to provide a block of content. If you need more flexibility, typically just providing the component name as an argument is the way to go. The sub-component can then just render that component with:

```html
{{> Template.dynamic templateName dataContext}}
```

This is more or less the way that the [`kadira:blaze-layout`](https://atmospherejs.com/kadira/blaze-layout) package works.

## Pass callbacks

If you need to communicate *up* the component hierarchy, it's best to pass a *callback* for the sub-component to call.

For instance, only one todo item can be in the editing state at a time, so the `Lists_show` component manages the state of which is edited. When you focus on an item, that item needs to tell the list's component to make it the "edited" one. To do that, we pass a callback into the `Todos_item` component, and the child calls it whenever the state needs to be updated in the parent:

```html
{{> Todos_item (todoArgs todo)}}
```

```js
Template.Lists_show.helpers({
  todoArgs(todo) {
    const instance = Template.instance();
    return {
      todo,
      editing: instance.state.equals('editingTodo', todo._id),
      onEditingChange(editing) {
        instance.state.set('editingTodo', editing ? todo._id : false);
      }
    };
  }
});

Template.Todos_item.events({
  'focus input[type=text]'() {
    this.onEditingChange(true);
  }
});
```

## Use `onRendered()` for 3rd party libraries

As we mentioned above, the `onRendered()` callback is typically the right spot to call out to third party libraries that expect a pre-rendered DOM (such as jQuery plugins). The `onRendered()` callback is triggered *once* after the component has rendered and attached to the DOM for the first time.

Occasionally, you may need to wait for data to become ready before it's time to attach the plugin (although typically it's a better idea to use a sub-component in this use case). To do so, you can setup an `autorun` in the `onRendered()` callback. For instance, in the `Lists_show_page` component, we want to wait until the subscription for the list is ready (i.e. the todos have rendered) before we hide the launch screen:

```js
Template.Lists_show_page.onRendered(function() {
  this.autorun(() => {
    if (this.subscriptionsReady()) {
      // Handle for launch screen defined in app-body.js
      AppLaunchScreen.listRender.release();
    }
  });
});
```

# Blaze로 스마트 컴포넌트 작성하기

Some of your components will need to access state outside of their data context---for instance, data from the server via subscriptions or the contents of client-side store. As discussed in the [data loading](https://guide.meteor.com/data-loading.html#patterns) and [UI](https://guide.meteor.com/ui-ux.html#smart-components) articles, you should be careful and considered in how you use such smart components.

All of the suggestions about reusable components apply to smart components. In addition:

## Subscribe from `onCreated`

You should subscribe to publications from the server from an `onCreated` callback (within an `autorun` block if you have reactively changing arguments). In the Todos example app, in the `Lists_show_page` template we subscribe to the `todos.inList` publication based on the current `_id` FlowRouter param:

```js
Template.Lists_show_page.onCreated(function() {
  this.getListId = () => FlowRouter.getParam('_id');

  this.autorun(() => {
    this.subscribe('todos.inList', this.getListId());
  });
});
```

We use `this.subscribe()` as opposed to `Meteor.subscribe()` so that the component automatically keeps track of when the subscriptions are ready. We can use this information in our HTML template with the built-in `{% raw %}{{Template.subscriptionsReady}}{% endraw %}` helper or within helpers using `instance.subscriptionsReady()`.

Notice that in this component we are also accessing the global client-side state store `FlowRouter`, which we wrap in a instance method called `getListId()`. This instance method is called both from the `autorun` in `onCreated`, and from the `listIdArray` helper:

```js
Template.Lists_show_page.helpers({
  // We use #each on an array of one item so that the "list" template is
  // removed and a new copy is added when changing lists, which is
  // important for animation purposes.
  listIdArray() {
    const instance = Template.instance();
    const listId = instance.getListId();
    return Lists.findOne(listId) ? [listId] : [];
  },
});
```

## Fetch in helpers

As described in the [UI/UX article](https://guide.meteor.com/ui-ux.html#smart-components), you should fetch data in the same component where you subscribed to that data. In a Blaze smart component, it's usually simplest to fetch the data in a helper, which you can then use to pass data into a reusable child component. For example, in the `Lists_show_page`:

```html
{{> Lists_show_page (listArgs listId)}}
```

The `listArgs` helper fetches the data that we've subscribed to above:

```js
Template.Lists_show_page.helpers({
  listArgs(listId) {
    const instance = Template.instance();
    return {
      todosReady: instance.subscriptionsReady(),
      // We pass `list` (which contains the full list, with all fields, as a function
      // because we want to control reactivity. When you check a todo item, the
      // `list.incompleteCount` changes. If we didn't do this the entire list would
      // re-render whenever you checked an item. By isolating the reactiviy on the list
      // to the area that cares about it, we stop it from happening.
      list() {
        return Lists.findOne(listId);
      },
      // By finding the list with only the `_id` field set, we don't create a dependency on the
      // `list.incompleteCount`, and avoid re-rendering the todos when it changes
      todos: Lists.findOne(listId, {fields: {_id: true}}).todos()
    };
  }
});

```

# Blaze에서 코드 재사용하기

It's common to want to reuse code between two otherwise unrelated components. There are two main ways to do this in Blaze.

## Composition

If possible, it's usually best to try and abstract out the reusable part of the two components that need to share functionality into a new, smaller component. If you follow the patterns for [reusable components](../guide/reusable-components.html), it should be simple to reuse this sub-component everywhere you need this functionality.

For instance, suppose you have many places in your application where you need an input to blur itself when you click the "esc" key. If you were building an autocomplete widget that also wanted this functionality, you could compose a `blurringInput` inside your `autocompleteInput`:

```html
<template name="autocompleteInput">
  {{> blurringInput name=name value=currentValue onChange=onChange}}
</template>
```

```js
Template.autocompleteInput.helpers({
  currentValue() {
    // perform complex logic to determine the auto-complete's current text value
  },
  onChange() {
    // This is the `autocompleteInput`'s template instance
    const instance = Template.instance();
    // The second argument to this function is the template instance of the `blurringInput`.
    return (event) => {
      // read the current value out of the input, potentially change the value
    };
  }
});
```

By making the `blurringInput` flexible and reusable, we can avoid re-implementing functionality in the `autocompleteInput`.

## Libraries

It's usually best to keep your view layer as thin as possible and contain a component to whatever specific task it specifically needs to do. If there's heavy lifting involved (such as complicated data loading logic), it often makes sense to abstract it out into a library that simply deals with the logic alone and doesn't deal with the Blaze system at all.

For example, if a component requires a lot of complicated [D3](http://d3js.org) code for drawing graphs, it's likely that that code itself could live in a separate module that's called by the component. That makes it easier to abstract the code later and share it between various components that need to all draw graphs.

## Global Helpers

Another way to share commonly used view code is a global Spacebars helper. You can define these with the `Template.registerHelper()` function. Typically you register helpers to do simple things (like rendering dates in a given format) which don't justify a separate sub-component. For instance, you could do:

```js
Template.registerHelper('shortDate', (date) => {
  return moment(date).format("MMM do YY");
});
```

```html
<template name="myBike">
  <dl>
   <dt>Date registered</dt>
   <dd>{{shortDate bike.registeredAt}}</dd>
 </dl>
</template>
```

# Blaze 이해하기

Although Blaze is a very intuitive rendering system, it does have some quirks and complexities that are worth knowing about when you are trying to do complex things.

## Re-rendering

Blaze is intentionally opaque about re-rendering. Tracker and Blaze are designed as "eventual consistency" systems that end up fully reflecting any data change eventually, but may take a few re-runs or re-renders in getting there, depending on how they are used. This can be frustrating if you are trying to carefully control when your component is re-rendered.

The first thing to consider here is if you actually need to care about your component re-rendering. Blaze is optimized so that it typically doesn't matter if a component is re-rendered even if it strictly shouldn't. If you make sure that your helpers are cheap to run and consequently rendering is not expensive, then you probably don't need to worry about this.

The main thing to understand about how Blaze re-renders is that re-rendering happens at the level of helpers and template inclusions. Whenever the *data context* of a component changes, it necessarily must re-run *all* helpers and data accessors (as `this` within the helper is the data context and thus will have changed).

Additionally, a helper will re-run if any *reactive data source* accessed from within *that specific helper* changes.

You can often work out *why* a helper has re-run by tracing the source of the reactive invalidation:

```js
Template.myTemplate.helpers({
  helper() {
    // When this helper is scheduled to re-run, the `console.trace` will log a stack trace of where
    // the invalidation has come from (typically a `changed` message from some reactive variable).
    Tracker.onInvalidate(() => console.trace());
  }
});
```

## Controlling re-rendering

If your helper or sub-component is expensive to run, and often re-runs without any visible effect, you can short circuit unnecessary re-runs by using a more subtle reactive data source. The [`peerlibrary:computed-field`](https://atmospherejs.com/peerlibrary/computed-field) package helps achieve this pattern.

## Attribute helpers

Setting tag attributes via helpers (e.g. `<div {% raw %}{{attributes}}{% endraw %}>`) is a neat tool and has some precedence rules that make it more useful. Specifically, when you use it more than once on a given element, the attributes are composed (rather than the second set of attributes simply replacing the first). So you can use one helper to set one set of attributes and a second to set another. For instance:

```html
<template name="myTemplate">
  <div id="my-div" {{classes 'foo' 'bar'}} {{backgroundImageStyle 'my-image.jpg'}}>My div</div>
</template>
```


```js
Template.myTemplate.helpers({
  classes(names) {
    return {class: names.map(n => `my-template-${n}`)};
  },
  backgroundImageStyle(imageUrl) {
    return {
      style: {
        backgroundImage: `url(${imageUrl})`
      }
    };
  }
});
```

## Lookup order

Another complicated topic in Blaze is name lookups. In what order does Blaze look when you write `{% raw %}{{something}}{% endraw %}`? It runs in the following order:

1. Helper defined on the current component
2. Binding (eg. from `{% raw %}{{#let}}{% endraw %}` or `{% raw %}{{#each in}}{% endraw %}`) in current scope
3. Template name
4. Global helper
5. Field on the current data context

## Blaze and the build system

As mentioned in the [build system article](https://guide.meteor.com/build-tool.html#blaze), the [`blaze-html-templates`](https://atmospherejs.com/meteor/blaze-html-templates) package scans your source code for `.html` files, picks out `<template name="templateName">` tags, and compiles them into a JavaScript file that defines a function that implements the component in code, attached to the `Template.templateName` symbol.

This means when you render a Blaze template, you are simply running a function on the client that corresponds to the Spacebars content you defined in the `.html` file.

## What is a view?

One of the most core concepts in Blaze is the "view", which is a building block that represents a reactively rendering area of a template. The view is the machinery that works behind the scenes to track reactivity, do lookups, and re-render appropriately when data changes. The view is the unit of re-rendering in Blaze. You can even use the view tree to walk the rendered component hierarchy, but it's better to avoid this in favor of communicating between components using callbacks, template arguments, or global data stores.

You can read more about views in the [Blaze View](../api/blaze.html#Blaze-View).

# 라우터

List of routing packages which supports rendering of blaze templates.

## Iron Router
A client and server side router designed specifically for Meteor.

To add Iron Router to your app, install the `iron:router` package
```js
meteor add iron:router
```
For detailed information about all of the features Iron Router has to offer, refer to the [Iron Router Guide](https://iron-meteor.github.io/iron-router/)

## Flow Router
Carefully Designed Client Side Router for Meteor.

To add Flow Router to your app, install the `kadira:flow-router` package
```js
meteor add kadira:flow-router
```
For detailed information about all of the features Flow Router has to offer, refer to the [Kadira Meteor routing guide](https://kadira.io/academy/meteor-routing-guide).



## Flow Router Extra
Carefully extended flow-router with waitOn and template context


To add Flow Router Extra to your app, install the `ostrio:flow-router-extra` package
```js
meteor add ostrio:flow-router-extra
```

For detailed information about all of the features Flow Router Extra has to offer, refer to the [Flow Router Extra Documentation](https://github.com/VeliovGroup/flow-router#flowrouter-extra).

# Templates

Documentation of Meteor's template API.

When you write a template as `<template name="foo"> ... </template>` in an HTML file in your app, Meteor generates a
"template object" named `Template.foo`. Note that template name cannot
contain hyphens and other special characters.

The same template may occur many times on a page, and these
occurrences are called template instances.  Template instances have a
life cycle of being created, put into the document, and later taken
out of the document and destroyed.  Meteor manages these stages for
you, including determining when a template instance has been removed
or replaced and should be cleaned up.  You can associate data with a
template instance, and you can access its DOM nodes when it is in the
document.

Read more about templates and how to use them in the [Spacebars](../api/spacebars.html) and [Blaze](../guide/introduction.html) article in the Guide.

## Template Declarations

{% apibox "Template#events" instanceDelimiter:. %}

Declare event handlers for instances of this template. Multiple calls add
new event handlers in addition to the existing ones.

See [Event Maps](#Event-Maps) for a detailed description of the event
map format and how event handling works in Meteor.

{% apibox "Template#helpers" instanceDelimiter:. %}

Each template has a local dictionary of helpers that are made available to it,
and this call specifies helpers to add to the template's dictionary.

Example:

```js
Template.myTemplate.helpers({
  foo() {
    return Session.get("foo");
  }
});
```

Now you can invoke this helper with `{% raw %}{{foo}}{% endraw %}` in the template defined
with `<template name="myTemplate">`.

Helpers can accept positional and keyword arguments:

```js
Template.myTemplate.helpers({
  displayName(firstName, lastName, keyword) {
    var prefix = keyword.hash.title ? keyword.hash.title + " " : "";
    return prefix + firstName + " " + lastName;
  }
});
```

Then you can call this helper from template like this:

```
{{displayName "John" "Doe" title="President"}}
```

You can learn more about arguments to helpers in [Spacebars](../api/spacebars.html).

Under the hood, each helper starts a new
[`Tracker.autorun`](http://docs.meteor.com/api/tracker.html#Tracker-autorun).  When its reactive
dependencies change, the helper is rerun. Helpers depend on their data
context, passed arguments and other reactive data sources accessed during
execution.

To create a helper that can be used in any template, use
[`Template.registerHelper`](../api/templates.html#Template-registerHelper).


{% apibox "Template#onRendered" instanceDelimiter:. %}

Callbacks added with this method are called once when an instance of
Template.*myTemplate* is rendered into DOM nodes and put into the document for
the first time.

In the body of a callback, `this` is a [template instance](../api/templates.html#Template-instances)
object that is unique to this occurrence of the template and persists across
re-renderings. Use the `onCreated` and `onDestroyed` callbacks to perform
initialization or clean-up on the object.

Because your template has been rendered, you can use functions like
[`this.findAll`](../api/templates.html#Blaze-TemplateInstance-findAll) which look at its DOM nodes.

This can be a good place to apply any DOM manipulations you want, after the
template is rendered for the first time.

```html
<template name="myPictures">
  <div class="container">
    {{#each pictures}}
      <img class="item" src="/{{.}}"/>
    {{/each}}
  </div>
</template>
```

```js
Template.myPictures.onRendered(function () {
  // Use the Packery jQuery plugin
  this.$('.container').packery({
    itemSelector: '.item',
    gutter: 10
  });
});
```

{% apibox "Template#onCreated" instanceDelimiter:. %}

Callbacks added with this method are called before your template's logic is
evaluated for the first time. Inside a callback, `this` is the new [template
instance](#Template-instances) object. Properties you set on this object will be
visible from the callbacks added with `onRendered` and `onDestroyed` methods and
from event handlers.

These callbacks fire once and are the first group of callbacks to fire.
Handling the `created` event is a useful way to set up values on template
instance that are read from template helpers using `Template.instance()`.

```js
Template.myPictures.onCreated(function () {
  // set up local reactive variables
  this.highlightedPicture = new ReactiveVar(null);
  
  // register this template within some central store
  GalleryTemplates.push(this);
});
```

{% apibox "Template#onDestroyed" instanceDelimiter:. %}

These callbacks are called when an occurrence of a template is taken off
the page for any reason and not replaced with a re-rendering.  Inside
a callback, `this` is the [template instance](../api/templates.html#Template-instances) object
being destroyed.

This group of callbacks is most useful for cleaning up or undoing any external
effects of `created` or `rendered` groups. This group fires once and is the last
callback to fire.

```js
Template.myPictures.onDestroyed(function () {
  // deregister from some central store
  GalleryTemplates = _.without(GalleryTemplates, this);
});
```


## Template instances

A template instance object represents an occurrence of a template in
the document.  It can be used to access the DOM and it can be
assigned properties that persist as the template is reactively updated.

Template instance objects are found as the value of `this` in the
`onCreated`, `onRendered`, and `onDestroyed` template callbacks, and as an
argument to event handlers.  You can access the current template instance
from helpers using [`Template.instance()`](../api/templates.html#Template-instance).

In addition to the properties and functions described below, you can assign
additional properties of your choice to the object. Use the
[`onCreated`](../api/templates.html#Template-onCreated) and [`onDestroyed`](../api/templates.html#Template-onDestroyed)
methods to add callbacks performing initialization or clean-up on the object.

You can only access `findAll`, `find`, `firstNode`, and `lastNode` from the
`onRendered` callback and event handlers, not from `onCreated` and
`onDestroyed`, because they require the template instance to be in the DOM.

Template instance objects are `instanceof Blaze.TemplateInstance`.

{% apibox "Blaze.TemplateInstance#findAll" %}

`template.findAll` returns an array of DOM elements matching `selector`.

{% apibox "Blaze.TemplateInstance#$" %}

`template.$` returns a [jQuery object](http://api.jquery.com/Types/#jQuery) of
those same elements. jQuery objects are similar to arrays, with
additional methods defined by the jQuery library.

The template instance serves as the document root for the selector. Only
elements inside the template and its sub-templates can match parts of
the selector.

{% apibox "Blaze.TemplateInstance#find" %}

Returns one DOM element matching `selector`, or `null` if there are no
such elements.

The template instance serves as the document root for the selector. Only
elements inside the template and its sub-templates can match parts of
the selector.

{% apibox "Blaze.TemplateInstance#firstNode" %}

The two nodes `firstNode` and `lastNode` indicate the extent of the
rendered template in the DOM.  The rendered template includes these
nodes, their intervening siblings, and their descendents.  These two
nodes are siblings (they have the same parent), and `lastNode` comes
after `firstNode`, or else they are the same node.

{% apibox "Blaze.TemplateInstance#lastNode" %}

{% apibox "Blaze.TemplateInstance#data" %}

This property provides access to the data context at the top level of
the template.  It is updated each time the template is re-rendered.
Access is read-only and non-reactive.

{% apibox "Blaze.TemplateInstance#autorun" %}

You can use `this.autorun` from an [`onCreated`](../api/templates.html#Template-onCreated) or
[`onRendered`](../api/templates.html#Template-onRendered) callback to reactively update the DOM
or the template instance.  You can use `Template.currentData()` inside
of this callback to access reactive data context of the template instance.
The Computation is automatically stopped when the template is destroyed.

Alias for `template.view.autorun`.

{% apibox "Blaze.TemplateInstance#subscribe" %}

You can use `this.subscribe` from an [`onCreated`](../api/templates.html#Template-onCreated) callback
to specify which data publications this template depends on. The subscription is
automatically stopped when the template is destroyed.

There is a complementary function `Template.instance().subscriptionsReady()`
which returns true when all of the subscriptions called with `this.subscribe`
are ready.

Inside the template's HTML, you can use the built-in helper
`Template.subscriptionsReady`, which is an easy pattern for showing loading
indicators in your templates when they depend on data loaded from subscriptions.

Example:

```js
Template.notifications.onCreated(function () {
  // Use this.subscribe inside onCreated callback
  this.subscribe("notifications");
});
```

```html
<template name="notifications">
  {{#if Template.subscriptionsReady}}
    <!-- This is displayed when all data is ready. -->
    {{#each notifications}}
      {{> notification}}
    {{/each}}
  {{else}}
    Loading...
  {{/if}}
</template>
```

Another example where the subscription depends on the data context:

```js
Template.comments.onCreated(function () {
  // Use this.subscribe with the data context reactively
  this.autorun(() => {
    var dataContext = Template.currentData();
    this.subscribe("comments", dataContext.postId);
  });
});
```

```html
{{#with post}}
  {{> comments postId=_id}}
{{/with}}
```

Another example where you want to initialize a plugin when the subscription is
done:

```js
Template.listing.onRendered(function () {
  var template = this;
  
  template.subscribe('listOfThings', () => {
    // Wait for the data to load using the callback
    Tracker.afterFlush(() => {
      // Use Tracker.afterFlush to wait for the UI to re-render
      // then use highlight.js to highlight a code snippet
      highlightBlock(template.find('.code'));
    });
  });
});
```

{% apibox "Blaze.TemplateInstance#view" %}

{% apibox "Template.registerHelper" %}

{% apibox "Template.instance" %}

{% apibox "Template.currentData" %}

{% apibox "Template.parentData" %}

For example, `Template.parentData(0)` is equivalent to `Template.currentData()`.  `Template.parentData(2)`
is equivalent to `{% raw %}{{../..}}{% endraw %}` in a template.

{% apibox "Template.body" %}

You can define helpers and event maps on `Template.body` just like on
any `Template.myTemplate` object.

Helpers on `Template.body` are only available in the `<body>` tags of
your app.  To register a global helper, use
[Template.registerHelper](../api/templates.html#Template-registerHelper).
Event maps on `Template.body` don't apply to elements added to the
body via `Blaze.render`, jQuery, or the DOM API, or to the body element
itself.  To handle events on the body, window, or document, use jQuery
or the DOM API.

{% apibox "Template.dynamic" %}

`Template.dynamic` allows you to include a template by name, where the name
may be calculated by a helper and may change reactively.  The `data`
argument is optional, and if it is omitted, the current data context
is used. It's also possible, to use `Template.dynamic` as a block helper
(`{% raw %}{{#Template.dynamic}} ... {{/Template.dynamic}}{% endraw %}`)

For example, if there is a template named "foo", `{% raw %}{{> Template.dynamic
template="foo"}}{% endraw %}` is equivalent to `{% raw %}{{> foo}}{% endraw %}` and
`{% raw %}{{#Template.dynamic template="foo"}} ... {{/Template.dynamic}}{% endraw %}`
is equivalent to `{% raw %}{{#foo}} ... {{/foo}}{% endraw %}`.

## Event Maps

An event map is an object where
the properties specify a set of events to handle, and the values are
the handlers for those events. The property can be in one of several
forms:

<dl>
{% dtdd name:"<em>eventtype</em>" %}
Matches the type of events, such as `'click'`, separated by a forward 
slash, like so `'touchend/mouseup/keyup'`.
{% enddtdd %}

{% dtdd name:"<em>eventtype selector</em>" %}
Matches a particular type of event, but only when it appears on
an element that matches a certain CSS selector.
{% enddtdd %}

{% dtdd name:"<em>event1, event2</em>" %}
To handle more than one event / selector with the same function, use a
comma-separated list.
{% enddtdd %}
</dl>

The handler function receives two arguments: `event`, an object with
information about the event, and `template`, a [template
instance](#Template-instances) for the template where the handler is
defined.  The handler also receives some additional context data in
`this`, depending on the context of the current element handling the
event.  In a template, an element's context is the
data context where that element occurs, which is set by
block helpers such as `#with` and `#each`.

Example:

```js
{
  // Fires when any element is clicked
  'click'(event) { ... },
  
  // Fires when any element with the 'accept' class is clicked
  'click .accept'(event) { ... },
  
  // Fires when 'accept' is clicked or focused
  'click .accept, focus .accept'(event) { ... }
  'click/focus .accept'(event) { ... }
  
  // Fires when 'accept' is clicked or focused, or a key is pressed
  'click .accept, focus .accept, keypress'(event) { ... }
  'click/focus .accept, keypress'(event) { ... }
  
}
```

Most events bubble up the document tree from their originating
element.  For example, `'click p'` catches a click anywhere in a
paragraph, even if the click originated on a link, span, or some other
element inside the paragraph.  The originating element of the event
is available as the `target` property, while the element that matched
the selector and is currently handling it is called `currentTarget`.

```js
{
  'click p'(event) {
    var paragraph = event.currentTarget; // always a P
    var clickedElement = event.target; // could be the P or a child element
  }
}
```

If a selector matches multiple elements that an event bubbles to, it
will be called multiple times, for example in the case of `'click
div'` or `'click *'`.  If no selector is given, the handler
will only be called once, on the original target element.

The following properties and methods are available on the event object
passed to handlers:

<dl class="objdesc">
{% dtdd name:"type" type:"String" %}
The event's type, such as "click", "blur" or "keypress".
{% enddtdd %}

{% dtdd name:"target" type:"DOM Element" %}
The element that originated the event.
{% enddtdd %}

{% dtdd name:"currentTarget" type:"DOM Element" %}
The element currently handling the event.  This is the element that
matched the selector in the event map.  For events that bubble, it may
be `target` or an ancestor of `target`, and its value changes as the
event bubbles.
{% enddtdd %}

{% dtdd name:"which" type:"Number" %}
For mouse events, the number of the mouse button (1=left, 2=middle, 3=right).
For key events, a character or key code.
{% enddtdd %}

{% dtdd name:"stopPropagation()" %}
Prevent the event from propagating (bubbling) up to other elements.
Other event handlers matching the same element are still fired, in
this and other event maps.
{% enddtdd %}

{% dtdd name:"stopImmediatePropagation()" %}
Prevent all additional event handlers from being run on this event,
including other handlers in this event map, handlers reached by
bubbling, and handlers in other event maps.
{% enddtdd %}

{% dtdd name:"preventDefault()" %}
Prevents the action the browser would normally take in response to this
event, such as following a link or submitting a form.  Further handlers
are still called, but cannot reverse the effect.
{% enddtdd %}

{% dtdd name:"isPropagationStopped()" %}
Returns whether `stopPropagation()` has been called for this event.
{% enddtdd %}

{% dtdd name:"isImmediatePropagationStopped()" %}
Returns whether `stopImmediatePropagation()` has been called for this event.
{% enddtdd %}

{% dtdd name:"isDefaultPrevented()" %}
Returns whether `preventDefault()` has been called for this event.
{% enddtdd %}
</dl>

Returning `false` from a handler is the same as calling
both `stopImmediatePropagation` and `preventDefault` on the event.

Event types and their uses include:

<dl class="objdesc">
{% dtdd name:"<code>click</code>" %}
Mouse click on any element, including a link, button, form control, or div.
Use `preventDefault()` to prevent a clicked link from being followed.
Some ways of activating an element from the keyboard also fire `click`.
{% enddtdd %}

{% dtdd name:"<code>dblclick</code>" %}
Double-click.
{% enddtdd %}

{% dtdd name:"<code>focus, blur</code>" %}
A text input field or other form control gains or loses focus.  You
can make any element focusable by giving it a `tabindex` property.
Browsers differ on whether links, checkboxes, and radio buttons are
natively focusable.  These events do not bubble.
{% enddtdd %}

{% dtdd name:"<code>change</code>" %}
A checkbox or radio button changes state.  For text fields, use
`blur` or key events to respond to changes.
{% enddtdd %}

{% dtdd name:"<code>mouseenter, mouseleave</code>" %} The pointer enters or
leaves the bounds of an element.  These events do not bubble.
{% enddtdd %}

{% dtdd name:"<code>mousedown, mouseup</code>" %}
The mouse button is newly down or up.
{% enddtdd %}

{% dtdd name:"<code>keydown, keypress, keyup</code>" %}
The user presses a keyboard key.  `keypress` is most useful for
catching typing in text fields, while `keydown` and `keyup` can be
used for arrow keys or modifier keys.
{% enddtdd %}

</dl>

Other DOM events are available as well, but for the events above,
Meteor has taken some care to ensure that they work uniformly in all
browsers.

# Blaze

Documentation of how to use Blaze, Meteor's reactive rendering engine.

Blaze is the package that makes reactive templates possible.
You can use the Blaze API directly in order to render templates programmatically
and manipulate "Views," the building blocks of reactive templates.

{% apibox "Blaze.render" %}

When you render a template, the callbacks added with
[`onCreated`](../api/templates.html#Template-onCreated) are invoked immediately, before evaluating
the content of the template.  The callbacks added with
[`onRendered`](../api/templates.html#Template-onRendered) are invoked after the View is rendered and
inserted into the DOM.

The rendered template
will update reactively in response to data changes until the View is
removed using [`Blaze.remove`](#Blaze-remove) or the View's
parent element is removed by Meteor or jQuery.

{% pullquote warning %}
If the View is removed by some other mechanism
besides Meteor or jQuery (which Meteor integrates with by default),
the View may continue to update indefinitely.  Most users will not need to
manually render templates and insert them into the DOM, but if you do,
be mindful to always call [`Blaze.remove`](#Blaze-remove) when the View is
no longer needed.
{% endpullquote %}

{% apibox "Blaze.renderWithData" %}

`Blaze.renderWithData(Template.myTemplate, data)` is essentially the same as
`Blaze.render(Blaze.With(data, function () { return Template.myTemplate; }))`.

{% apibox "Blaze.remove" %}

Use `Blaze.remove` to remove a template or View previously inserted with
`Blaze.render`, in such a way that any behaviors attached to the DOM by
Meteor are cleaned up.  The rendered template or View is now considered
["destroyed"](../api/templates.html#Template-onDestroyed), along with all nested templates and
Views.  In addition, any data assigned via
jQuery to the DOM nodes is removed, as if the nodes were passed to
jQuery's `$(...).remove()`.

As mentioned in [`Blaze.render`](#Blaze-render), it is important to "remove"
all content rendered via `Blaze.render` using `Blaze.remove`, unless the
parent node of `renderedView` is removed by a Meteor reactive
update or with jQuery.

`Blaze.remove` can be used even if the DOM nodes in question have already
been removed from the document, to tell Blaze to stop tracking and
updating these nodes.

{% apibox "Blaze.getData" %}

{% apibox "Blaze.toHTML" %}

Rendering a template to HTML loses all fine-grained reactivity.  The
normal way to render a template is to either include it from another
template (`{% raw %}{{> myTemplate}}{% endraw %}`) or render and insert it
programmatically using `Blaze.render`.  Only occasionally
is generating HTML useful.

Because `Blaze.toHTML` returns a string, it is not able to update the DOM
in response to reactive data changes.  Instead, any reactive data
changes will invalidate the current Computation if there is one
(for example, an autorun that is the caller of `Blaze.toHTML`).

{% apibox "Blaze.toHTMLWithData" %}

{% apibox "Blaze.View" %}

Behind every template or part of a template &mdash; a template tag, say, like `{% raw %}{{foo}}{% endraw %}` or `{% raw %}{{#if}}{% endraw %}` &mdash; is
a View object, which is a reactively updating region of DOM.

Most applications do not need to be aware of these Views, but they offer a
way to understand and customize Meteor's rendering behavior for more
advanced applications and packages.

You can obtain a View object by calling [`Blaze.render`](#Blaze-render) on a
template, or by accessing [`template.view`](../api/templates.html#Blaze-TemplateInstance-view) on a template
instance.

At the heart of a View is an [autorun](https://docs.meteor.com/api/tracker.html#Tracker-autorun) that calls the View's
`renderFunction`, uses the result to create DOM nodes, and replaces the
contents of the View with these new DOM nodes.  A View's content may consist
of any number of consecutive DOM nodes (though if it is zero, a placeholder
node such as a comment or an empty text node is automatically supplied).  Any
reactive dependency established by `renderFunction` causes a full recalculation
of the View's contents when the dependency is invalidated.  Templates, however,
are compiled in such a way that they do not have top-level dependencies and so
will only ever render once, while their parts may re-render many times.

When a `Blaze.View` is constructed by calling the constructor, no hooks
are fired and no rendering is performed.  In particular, the View is
not yet considered to be "created."  Only when the View is actually
used, by a call to `Blaze.render` or `Blaze.toHTML` or by inclusion in
another View, is it "created," right before it is rendered for the
first time.  When a View is created, its `.parentView` is set if
appropriate, and then the `onViewCreated` hook is fired.  The term
"unrendered View" means a newly constructed View that has not been
"created" or rendered.

The "current View" is kept in [`Blaze.currentView`](#Blaze-currentView) and
is set during View rendering, callbacks, autoruns, and template event
handlers.  It affects calls such as [`Template.currentData()`](../api/templates.html#Template-currentData).

The following properties and methods are available on Blaze.View:

<dl class="objdesc">
{% dtdd name:"name" type:"String" id:"view_name" %}
  The name of this type of View.  View names may be used to identify
particular kinds of Views in code, but more often they simply aid in
debugging and comprehensibility of the View tree.  Views generated
by Meteor have names like "Template.foo" and "if".
{% enddtdd %}

{% dtdd name:"parentView" type:"View or null" id:"view_parentview" %}
  The enclosing View that caused this View to be rendered, if any.
{% enddtdd %}

{% dtdd name:"isCreated" type:"Boolean" id:"view_iscreated" %}
  True if this View has been called on to be rendered by `Blaze.render`
  or `Blaze.toHTML` or another View.  Once it becomes true, never
  becomes false again.  A "created" View's `.parentView` has been
  set to its final value.  `isCreated` is set to true before
  `onViewCreated` hooks are called.
{% enddtdd %}

{% dtdd name:"isRendered" type:"Boolean" id:"view_isrendered" %}
  True if this View has been rendered to DOM by `Blaze.render` or
  by the rendering of an enclosing View.  Conversion to HTML by
  `Blaze.toHTML` doesn't count.  Once true, never becomes false.
{% enddtdd %}

{% dtdd name:"isDestroyed" type:"Boolean" id:"view_isdestroyed" %}
  True if this View has been destroyed, such as by `Blaze.remove()` or
  by a reactive update that removes it.  A destroyed View's autoruns
  have been stopped, and its DOM nodes have generally been cleaned
  of all Meteor reactivity and possibly dismantled.
{% enddtdd %}

{% dtdd name:"renderCount" type:"Integer" id:"view_rendercount" %}
  The number of times the View has been rendered, including the
  current time if the View is in the process of being rendered
  or re-rendered.
{% enddtdd %}

{% dtdd name:"autorun(runFunc)" id:"view_autorun" %}
  Like [`Tracker.autorun`](https://docs.meteor.com/api/tracker.html#Tracker-autorun), except that the autorun is
  automatically stopped when the View is destroyed, and the
  [current View](#Blaze-currentView) is always set when running `runFunc`.
  There is no relationship to the View's internal autorun or render
  cycle.  In `runFunc`, the View is bound to `this`.
{% enddtdd %}

{% dtdd name:"onViewCreated(func)" id:"view_onviewcreated" %}
  If the View hasn't been created yet, calls `func` when the View
  is created.  In `func`, the View is bound to `this`.

  This hook is the basis for the [`created`](../api/templates.html#Template-onCreated)
  template callback.
{% enddtdd %}

{% dtdd name:"onViewReady(func)" id:"view_onviewready" %}
  Calls `func` when the View is rendered and inserted into the DOM,
  after waiting for the end of
  [flush time](https://docs.meteor.com/api/tracker.html#Tracker-afterFlush).  Does not fire if the View
  is destroyed at any point before it would fire.
  May fire multiple times (if the View re-renders).
  In `func`, the View is bound to `this`.

  This hook is the basis for the [`rendered`](../api/templates.html#Template-onRendered)
  template callback.
{% enddtdd %}

{% dtdd name:"onViewDestroyed(func)" id:"view_onviewdestroyed" %}
  If the View hasn't been destroyed yet, calls `func` when the
  View is destroyed.  A View may be destroyed without ever becoming
  "ready."  In `func`, the View is bound to `this`.

  This hook is the basis for the [`destroyed`](../api/templates.html#Template-onDestroyed)
  template callback.
{% enddtdd %}

{% dtdd name:"firstNode()" type:"DOM node" id:"view_firstnode" %}
The first node of the View's rendered content.  Note that this may
be a text node.  Requires that the View be rendered.
If the View rendered to zero DOM nodes, it may be a placeholder
node (comment or text node).  The DOM extent of a View consists
of the nodes between `view.firstNode()` and `view.lastNode()`,
inclusive.
{% enddtdd %}

{% dtdd name:"lastNode()" type:"DOM node" id:"view_lastnode" %}
The last node of the View's rendered content.

See [`firstNode()`](#view_firstnode).
{% enddtdd %}

{% dtdd name:"template" type:"Template" id:"view_template" %}
For Views created by invoking templates, the original Template
object.  For example, `Blaze.render(Template.foo).template === Template.foo`.
{% enddtdd %}

{% dtdd name:"templateInstance()" type:"Template instance" id:"view_templateinstance" %}
For Views created by invoking templates,
returns the [template instance](../api/templates.html#Template-instances) object for this
particular View.  For example, in a [`created`](../api/templates.html#Template-onCreated)
callback, `this.view.templateInstance() === this`.

Template instance objects have fields like `data`, `firstNode`, and
`lastNode` which are not reactive and which are also not automatically
kept up to date.  Calling `templateInstance()` causes these fields to
be updated.

{% enddtdd %}

</dl>

{% apibox "Blaze.currentView" nested:true %}

The "current view" is used by [`Template.currentData()`](../api/templates.html#Template-currentData) and
[`Template.instance()`](../api/templates.html#Template-instance) to determine
the contextually relevant data context and template instance.

{% apibox "Blaze.getView" nested:true %}

If you don't specify an `element`, there must be a current View or an
error will be thrown.  This is in contrast to
[`Blaze.currentView`](#Blaze-currentView).

{% apibox "Blaze.With" nested:true %}

Returns an unrendered View object you can pass to `Blaze.render`.

Unlike `{% raw %}{{#with}}{% endraw %}` (as used in templates), `Blaze.With` has no "else" case, and
a falsy value for the data context will not prevent the content from
rendering.

{% apibox "Blaze.If" nested:true %}

Returns an unrendered View object you can pass to `Blaze.render`.

Matches the behavior of `{% raw %}{{#if}}{% endraw %}` in templates.

{% apibox "Blaze.Unless" nested:true %}

Returns an unrendered View object you can pass to `Blaze.render`.

Matches the behavior of `{% raw %}{{#unless}}{% endraw %}` in templates.

{% apibox "Blaze.Each" nested:true %}

Returns an unrendered View object you can pass to `Blaze.render`.

Matches the behavior of `{% raw %}{{#each}}{% endraw %}` in templates.

{% apibox "Blaze.Template" %}

Templates defined by the template compiler, such as `Template.myTemplate`,
are objects of type `Blaze.Template` (aliased as `Template`).

In addition to methods like `events` and `helpers`, documented as part of
the [Template API](../api/templates.html), the following fields and methods are
present on template objects:

<dl class="objdesc">

{% dtdd name:"viewName" type:"String" id:"template_viewname" %}
  Same as the constructor argument.
{% enddtdd %}

{% dtdd name:"renderFunction" type:"Function" id:"template_renderfunction" %}
  Same as the constructor argument.
{% enddtdd %}

{% dtdd name:"constructView()" id:"template_constructview" %}
  Constructs and returns an unrendered View object.  This method is invoked
  by Meteor whenever the template is used, such as by `Blaze.render` or by
  `{% raw %}{{> foo}}{% endraw %}` where `foo` resolves to a Template object.

  `constructView()` constructs a View using `viewName` and `renderFunction`
  as constructor arguments, and then configures it as a template
  View, setting up `view.template`, `view.templateInstance()`, event maps, and so on.
{% enddtdd %}

</dl>

{% apibox "Blaze.isTemplate" %}

## Renderable Content

A value is *renderable content* if it is one of the following:

* A [template object](../api/templates.html) like `Template.myTemplate`
* An unrendered [View](../api/blaze.html#Blaze-View) object, like the return value of `Blaze.With`
* `null` or `undefined`

> Internally, renderable content includes objects representing HTML tags
as well, but these objects are not yet part of the officially-supported,
public API.

# Spacebars

Documentation of Meteor's `spacebars` package.

Spacebars is a Meteor template language inspired by
[Handlebars](http://handlebarsjs.com/).  It shares some of the spirit and syntax
of Handlebars, but it has been tailored to produce reactive Meteor templates
when compiled.

## Getting Started

A Spacebars template consists of HTML interspersed with template tags, which are
delimited by `{% raw %}{{{% endraw %}` and `}}` (two curly braces).

```html
<template name="myPage">
  <h1>{{pageTitle}}</h1>

  {{> nav}}

  {{#each posts}}
    <div class="post">
      <h3>{{title}}</h3>
      <div class="post-content">
        {{{content}}}
      </div>
    </div>
  {{/each}}
</template>
```

As illustrated by the above example, there are four major types of template
tags:

* `{% raw %}{{pageTitle}}{% endraw %}` - Double-braced template tags are used to insert a string of
  text.  The text is automatically made safe.  It may contain any characters
  (like `<`) and will never produce HTML tags.

* `{% raw %}{{> nav}}{% endraw %}` - Inclusion template tags are used to insert another template by
  name.

* `{% raw %}{{#each}}{% endraw %}` - Block template tags are notable for having a block of content.
  The block tags `#if`, `#each`, `#with`, and `#unless` are built in, and it is
  also possible define custom ones.  Some block tags, like `#each` and `#with`,
  establish a new data context for evaluating their contents.  In the above
  example, `{% raw %}{{title}}{% endraw %}` and `{% raw %}{{content}}{% endraw %}` most likely refer to properties of the
  current post (though they could also refer to template helpers).

* `{% raw %}{{{content}}}{% endraw %}` - Triple-braced template tags are used to insert raw HTML.  Be
  careful with these!  It's your job to make sure the HTML is safe, either by
  generating it yourself or sanitizing it if it came from a user input.

## Reactivity Model


Spacebars templates update reactively at a fine-grained level in response to
changing data.

Each template tag's DOM is updated automatically when it evaluates to a new
value, while avoiding unnecessary re-rendering as much as possible.  For
example, a double-braced tag replace its text node when its text value changes.
An `#if` re-renders its contents only when the condition changes from truthy to
falsy or vice versa.

## Identifiers and Paths


A Spacebars identifier is either a JavaScript identifier name or any string
enclosed in square brackets (`[` and `]`).  There are also the special
identifiers `this` (or equivalently, `.`) and `..`.  Brackets are required to
use one of the following as the first element of a path: `else`, `this`, `true`,
`false`, and `null`.  Brackets are not required around JavaScript keywords and
reserved words like `var` and `for`.

A Spacebars path is a series of one or more identifiers separated by either `.`
or `/`, such as `foo`, `foo.bar`, `this.name`, `../title`, or `foo.[0]` (numeric indices must be enclosed in brackets).

## Name Resolution

The first identifier in a path is resolved in one of two ways:

* Indexing the current data context.  The identifier `foo` refers to the `foo`
  property of the current data context object.

* As a template helper.  The identifier `foo` refers to a helper function (or
  constant value) that is accessible from the current template.

Template helpers take priority over properties of the data context.

If a path starts with `..`, then the *enclosing* data context is used instead of
the current one.  The enclosing data context might be the one outside the
current `#each`, `#with`, or template inclusion.

## Path Evaluation

When evaluating a path, identifiers after the first are used to index into the
object so far, like JavaScript's `.`.  However, an error is never thrown when
trying to index into a non-object or an undefined value.

In addition, Spacebars will call functions for you, so `{% raw %}{{foo.bar}}{% endraw %}` may be
taken to mean `foo().bar`, `foo.bar()`, or `foo().bar()` as appropriate.

## Helper Arguments

An argument to a helper can be any path or identifier, or a string, boolean, or
number literal, or null.

Double-braced and triple-braced template tags take any number of positional and
keyword arguments:

```html
{{frob a b c verily=true}}
```
calls:
```js
frob(a, b, c, Spacebars.kw({verily: true}))
```

`Spacebars.kw` constructs an object that is `instanceof Spacebars.kw` and whose
`.hash` property is equal to its argument.

The helper's implementation can access the current data context as `this`.

## Inclusion and Block Arguments

Inclusion tags (`{% raw %}{{> foo}}{% endraw %}`) and block tags (`{% raw %}{{#foo}}{% endraw %}`) take a single
data argument, or no argument.  Any other form of arguments will be interpreted
as an *object specification* or a *nested helper*:

* **Object specification**: If there are only keyword arguments, as in `{% raw %}{{#with
  x=1 y=2}}{% endraw %}` or `{% raw %}{{> prettyBox color=red}}{% endraw %}`, the keyword arguments will be
  assembled into a data object with properties named after the keywords.

* **Nested Helper**: If there is a positional argument followed by other
  (positional or keyword arguments), the first argument is called on the others
  using the normal helper argument calling convention.

## Template Tag Placement Limitations

Unlike purely string-based template systems, Spacebars is HTML-aware and
designed to update the DOM automatically.  As a result, you can't use a template
tag to insert strings of HTML that don't stand on their own, such as a lone HTML
start tag or end tag, or that can't be easily modified, such as the name of an
HTML element.

There are three main locations in the HTML where template tags are allowed:

* At element level (i.e. anywhere an HTML tag could go)
* In an attribute value
* In a start tag in place of an attribute name/value pair

The behavior of a template tag is affected by where it is located in the HTML,
and not all tags are allowed at all locations.

### Double-braced Tags

A double-braced tag at element level or in an attribute value typically evalutes
to a string.  If it evalutes to something else, the value will be cast to a
string, unless the value is `null`, `undefined`, or `false`, which results in
nothing being displayed.

Values returned from helpers must be pure text, not HTML.  (That is, strings
should have `<`, not `&lt;`.)  Spacebars will perform any necessary escaping if
a template is rendered to HTML.

### SafeString

If a double-braced tag at element level evalutes to an object created with
`Spacebars.SafeString("<span>Some HTML</span>")`, the HTML is inserted at the
current location.  The code that calls `SafeString` is asserting that this HTML
is safe to insert.

## In Attribute Values

A double-braced tag may be part of, or all of, an HTML attribute value:

```html
<input type="checkbox" class="checky {{moreClasses}}" checked={{isChecked}}>
```

An attribute value that consists entirely of template tags that return `null`,
`undefined`, or `false` is considered absent; otherwise, the attribute is
considered present, even if its value is empty.

### Dynamic Attributes

A double-braced tag can be used in an HTML start tag to specify an arbitrary set
of attributes:

```html
<div {{attrs}}>...</div>

<input type=checkbox {{isChecked}}>
```

The tag must evaluate to an object that serves as a dictionary of attribute name
and value strings.  For convenience, the value may also be a string or null.  An
empty string or null expands to `{}`.  A non-empty string must be an attribute
name, and expands to an attribute with an empty value; for example, `"checked"`
expands to `{checked: ""}` (which, as far as HTML is concerned, means the
checkbox is checked).

To summarize:

<table>
  <thead>
    <tr><th>Return Value</th><th>Equivalent HTML</th></tr>
  </thead>
  <tbody>
    <tr><td><code>""</code> or <code>null</code> or <code>{}</code></td></tr>
    <tr><td><code>"checked"</code> or <code>{checked: ""}</code></td><td><code>checked</code></td></tr>
    <tr><td><code>{checked: "", 'class': "foo"}</code></td><td><code>checked  class=foo</code></td></tr>
    <tr><td><code>{checked: false, 'class': "foo"}</code></td><td><code>class=foo</code></td></tr>
    <tr><td><code>"checked class=foo"</code></td><td>ERROR, string is not an attribute name</td></tr>
  </tbody>
</table>

You can combine multiple dynamic attributes tags with other attributes:

```html
<div id=foo class={{myClass}} {{attrs1}} {{attrs2}}>...</div>
```

Attributes from dynamic attribute tags are combined from left to right, after
normal attributes, with later attribute values overwriting previous ones.
Multiple values for the same attribute are not merged in any way, so if `attrs1`
specifies a value for the `class` attribute, it will overwrite `{% raw %}{{myClass}}{% endraw %}`.
As always, Spacebars takes care of recalculating the element's attributes if any
of `myClass`, `attrs1`, or `attrs2` changes reactively.


## Triple-braced Tags

Triple-braced tags are used to insert raw HTML into a template:

```html
<div class="snippet">
  {{{snippetBody}}}
</div>
```

The inserted HTML must consist of balanced HTML tags.  You can't, for example,
insert `"</div><div>"` to close an existing div and open a new one.

This template tag cannot be used in attributes or in an HTML start tag.

## Inclusion Tags

An inclusion tag takes the form `{% raw %}{{> templateName}}{% endraw %}` or `{% raw %}{{> templateName
dataObj}}{% endraw %}`.  Other argument forms are syntactic sugar for constructing a data
object (see Inclusion and Block Arguments).

An inclusion tag inserts an instantiation of the given template at the current
location.  If there is an argument, it becomes the data context, much as if the
following code were used:

```html
{{#with dataObj}}
  {{> templateName}}
{{/with}}
```

Instead of simply naming a template, an inclusion tag can also specify a path
that evalutes to a template object, or to a function that returns a template
object.

Note that the above two points interact in a way that can be surprising!
If `foo` is a template helper function that returns another template, then
`{% raw %}{{>foo bar}}{% endraw %}` will _first_ push `bar` onto the data context stack _then_ call
`foo()`, due to the way this line is expanded as shown above. You will need to
use `Template.parentData(1)` to access the original context. This differs
from regular helper calls like `{% raw %}{{foo bar}}{% endraw %}`, in which `bar` is passed as a
parameter rather than pushed onto the data context stack.

## Function Returning a Template

If an inclusion tag resolves to a function, the function must return a template
object or `null`.  The function is reactively re-run, and if its return value
changes, the template will be replaced.

## Block Tags

Block tags invoke built-in directives or custom block helpers, passing a block
of template content that may be instantiated once, more than once, or not at all
by the directive or helper.

```html
{{#block}}
  <p>Hello</p>
{{/block}}
```

Block tags may also specify "else" content, separated from the main content by
the special template tag `{% raw %}{{else}}{% endraw %}`.

A block tag's content must consist of HTML with balanced tags.

Block tags can be used inside attribute values:

```html
<div class="{{#if done}}done{{else}}notdone{{/if}}">
  ...
</div>
```

You can chain block tags:

```html
{{#foo}}
  <p>Foo</p>
{{else bar}}
  <p>Bar</p>
{{else}}
  <p></p>
{{/foo}}
```

This is equivalent to:

```html
{{#foo}}
  <p>Foo</p>
{{else}}
  {{#bar}}
    <p>Bar</p>
  {{else}}
    <p></p>
  {{/bar}}
{{/foo}}
```

## If/Unless

An `#if` template tag renders either its main content or its "else" content,
depending on the value of its data argument.  Any falsy JavaScript value
(including `null`, `undefined`, `0`, `""`, and `false`) is considered false, as
well as the empty array, while any other value is considered true.

```html
{{#if something}}
  <p>It's true</p>
{{else}}
  <p>It's false</p>
{{/if}}
```

`#unless` is just `#if` with the condition inverted.

## With

A `#with` template tag establishes a new data context object for its contents.
The properties of the data context object are where Spacebars looks when
resolving template tag names.

```html
{{#with employee}}
  <div>Name: {{name}}</div>
  <div>Age: {{age}}</div>
{{/with}}
```

We can take advantage of the object specification form of a block tag to define
an object with properties we name:

```html
{{#with x=1 y=2}}
  {{{getHTMLForPoint this}}}
{{/with}}
```

If the argument to `#with` is falsy (by the same rules as for `#if`), the
content is not rendered.  An "else" block may be provided, which will be
rendered instead.

If the argument to `#with` is a string or other non-object value, it may be
promoted to a JavaScript wrapper object (also known as a boxed value) when
passed to helpers, because JavaScript traditionally only allows an object for
`this`.  Use `String(this)` to get an unboxed string value or `Number(this)` to
get an unboxed number value.

## Each

An `#each` template tag takes a sequence argument and inserts its content for
each item in the sequence, setting the data context to the value of that item:

```html
<ul>
{{#each people}}
  <li>{{name}}</li>
{{/each}}
</ul>
```

The newer variant of `#each` doesn't change the data context but introduces a
new variable that can be used in the body to refer to the current item:

```html
<ul>
{{#each person in people}}
  <li>{{person.name}}</li>
{{/each}}
</ul>
```

The argument is typically a Meteor cursor (the result of `collection.find()`,
for example), but it may also be a plain JavaScript array, `null`, or
`undefined`.

An "else" section may be provided, which is used (with no new data
context) if there are zero items in the sequence at any time.

You can use a special variable `@index` in the body of `#each` to get the
0-based index of the currently rendered value in the sequence.

### Reactivity Model for Each

When the argument to `#each` changes, the DOM is always updated to reflect the
new sequence, but it's sometimes significant exactly how that is achieved.  When
the argument is a Meteor live cursor, the `#each` has access to fine-grained
updates to the sequence -- add, remove, move, and change callbacks -- and the
items are all documents identified by unique ids.  As long as the cursor itself
remains constant (i.e. the query doesn't change), it is very easy to reason
about how the DOM will be updated as the contents of the cursor change.  The
rendered content for each document persists as long as the document is in the
cursor, and when documents are re-ordered, the DOM is re-ordered.

Things are more complicated if the argument to the `#each` reactively changes
between different cursor objects, or between arrays of plain JavaScript objects
that may not be identified clearly.  The implementation of `#each` tries to be
intelligent without doing too much expensive work. Specifically, it tries to
identify items between the old and new array or cursor with the following
strategy:

1. For objects with an `_id` field, use that field as the identification key
2. For objects with no `_id` field, use the array index as the identification
   key. In this case, appends are fast but prepends are slower.
3. For numbers or strings, use their value as the identification key.

In case of duplicate identification keys, all duplicates after the first are
replaced with random ones. Using objects with unique `_id` fields is the way to
get full control over the identity of rendered elements.

## Let

The `#let` tag creates a new alias variable for a given expression. While it
doesn't change the data context, it allows to refer to an expression (helper,
data context, another variable) with a short-hand within the template:

```html
{{#let name=person.bio.firstName color=generateColor}}
  <div>{{name}} gets a {{color}} card!</div>
{{/let}}
```

Variables introduced this way take precedence over names of templates, global
helpers, fields of the current data context and previously introduced
variables with the same name.

## Custom Block Helpers

To define your own block helper, simply declare a template, and then invoke it
using `{% raw %}{{#someTemplate}}{% endraw %}` (block) instead of `{% raw %}{{> someTemplate}}{% endraw %}` (inclusion)
syntax.

When a template is invoked as a block helper, it can use `{% raw %}{{>
Template.contentBlock}}{% endraw %}` and `{% raw %}{{> Template.elseBlock}}{% endraw %}` to include the block
content it was passed.

Here is a simple block helper that wraps its content in a div:

```html
<template name="note">
  <div class="note">
    {{> Template.contentBlock}}
  </div>
</template>
```

You would invoke it as:

```html
{{#note}}
  Any content here
{{/note}}
```

Here is an example of implementing `#unless` in terms of `#if` (ignoring for the
moment that `unless` is a built-in directive):

```html
<template name="unless">
  {{#if this}}
    {{> Template.elseBlock}}
  {{else}}
    {{> Template.contentBlock}}
  {{/if}}
</template>
```

Note that the argument to `#unless` (the condition) becomes the data context in
the `unless` template and is accessed via `this`.  However, it would not work
very well if this data context was visible to `Template.contentBlock`, which is
supplied by the user of `unless`.

Therefore, when you include `{% raw %}{{> Template.contentBlock}}{% endraw %}`, Spacebars hides the
data context of the calling template, and any data contexts established in the
template by `#each` and `#with`.  They are not visible to the content block,
even via `..`.  Put another way, it's as if the `{% raw %}{{> Template.contentBlock}}{% endraw %}`
inclusion occurred at the location where `{% raw %}{{#unless}}{% endraw %}` was invoked, as far as
the data context stack is concerned.

You can pass an argument to `{% raw %}{{> Template.contentBlock}}{% endraw %}` or `{% raw %}{{>
Template.elseBlock}}{% endraw %}` to invoke it with a data context of your choice.  You can
also use `{% raw %}{{#if Template.contentBlock}}{% endraw %}` to see if the current template was
invoked as a block helper rather than an inclusion.

## Comment Tags

Comment template tags begin with `{% raw %}{{!{% endraw %}` and can contain any characters except for
`}}`.  Comments are removed upon compilation and never appear in the compiled
template code or the generated HTML.

```html
{{! Start of a section}}
<div class="section">
  ...
</div>
```

Comment tags also come in a "block comment" form.  Block comments may contain
`{% raw %}{{{% endraw %}` and `}}`:

```html
{{!-- This is a block comment.
We can write {{foo}} and it doesn't matter.
{{#with x}}This code is commented out.{{/with}}
--}}
```

Comment tags can be used wherever other template tags are allowed.

## Nested sub-expressions

Sometimes an argument to a helper call is best expressed as a return value of
some other expression. For this and other cases, one can use parentheses to
express the evaluation order of nested expressions.

```html
{{capitalize (getSummary post)}}
```

In this example, the result of the `getSummary` helper call will be passed to
the `capitalize` helper.

Sub-expressions can be used to calculate key-word arguments, too:

```html
{{> tmpl arg=(helper post)}}
```

## HTML Dialect

Spacebars templates are written in [standard
HTML](http://developers.whatwg.org/syntax.html) extended with
additional syntax (i.e. template tags).

Spacebars validates your HTML as it goes and will throw a compile-time
error if you violate basic HTML syntax in a way that prevents it from
determining the structure of your code.

Spacebars is not lenient about malformed markup the way a web browser
is.  While the latest HTML spec standardizes how browsers should
recover from parse errors, these cases are still not valid HTML.  For
example, a browser may recover from a bare `<` that does not begin a
well-formed HTML tag, while Spacebars will not.  However, gone are the
restrictions of the XHTML days; attribute values do not have to
quoted, and tags are not case-sensitive, for example.

You must close all HTML tags except the ones specified to have no end
tag, like BR, HR, IMG and INPUT.  You can write these tags as `<br>`
or equivalently `<br/>`.

The HTML spec allows omitting some additional end tags, such as P and
LI, but Spacebars doesn't currently support this.

## Top-level Elements in a `.html` file

Technically speaking, the `<template>` element is not part of the Spacebars
language. A `foo.html` template file in Meteor consists of one or more of the
following elements:

* `<template name="myName">` - The `<template>` element contains a Spacebars
  template (as defined in the rest of this file) which will be compiled to the
  `Template.myName` component.

* `<head>` - Static HTML that will be inserted into the `<head>` element of the
  default HTML boilerplate page. Cannot contain template tags. If `<head>` is
  used multiple times (perhaps in different files), the contents of all of the
  `<head>` elements are concatenated.

* `<body>` - A template that will be inserted into the `<body>` of the main
  page.  It will be compiled to the `Template.body` component. If `<body>` is
  used multiple times (perhaps in different files), the contents of all of the
  `<body>` elements are concatenated.

## Escaping Curly Braces

To insert a literal `{% raw %}{{{% endraw %}`, `{% raw %}{{{{% endraw %}`, or any number of curly braces, put a
vertical bar after it.  So `{% raw %}{{|{% endraw %}` will show up as `{% raw %}{{{% endraw %}`, `{% raw %}{{{|{% endraw %}` will
show up as `{% raw %}{{{{% endraw %}`, and so on.