---
id: thinking-in-react-ko-KR
title: 리액트로 생각해보기
prev: tutorial-ko-KR.html
next: videos-ko-KR.html
---

이 문서의 원본은 [공식 블로그](/react/blog)의 [포스팅](/react/blog/2013/11/05/thinking-in-react.html) 입니다.

제가 생각하기에, React는 Javascript로 크고 빠른 웹 어플리케이션을 만드는데 최고입니다. 페이스북과 인스타그램에서 우리에게 잘 맞도록 조정되어 왔습니다.

React의 많은 뛰어난 점들중 하나는 당신이 생각을 하면서 어플리케이션을 만들게 한다는 겁니다. 이 포스트에서 React를 이용해 검색이 가능한 상품자료 테이블을 만드는 생각 과정을 이해할 수 있게 차근차근 설명할겁니다. *만약 이 페이지에서 jsfiddle이 보이지 않는다면 https가 아니라 http페이지를 가리키고 있는지 확인해 보세요.*

## 모형으로 시작해보기

우리가 이미 JSON API와 디자이너로부터 넘겨받은 모형을 이미 가지고 있다고 생각해봅시다. 보시다시피 우리 디자이너는 별로 좋지 않습니다:

![Mockup](/react/img/blog/thinking-in-react-mock.png)

우리의 JSON API는 아래와 같은 데이터를 반환합니다:

```
[
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
```

## 1단계: UI를 계층 구조의 구성요소로 분쇄하세요.

당신이 하고싶은 첫번째는 모형에 있는 모든 구성요소 (그리고 자식요소) 주위에 상자를 그리고, 이름을 부여하는 것입니다. 만약 당신이 디자이너와 같이 작업중이라면, 그들은 이미 이 작업을 해놨을지도 모릅니다. 당장 가서 이야기해보세요. 그들의 포토샵 레이어 이름이 결국 당신의 React 구성요소들의 이름이 될것입니다.

그런데 무엇이 구성요소가 되어야 할까요? 당신이 새로운 함수나 객체를 만들어야만 한다면, 똑같이 적용하세요. 한가지 방법은 [단일 책임의 원칙](http://en.wikipedia.org/wiki/Single_responsibility_principle) 입니다. 즉 하나의 구성요소는 이상적으로 한가지 작업만 수행해야합니다. 구성요소가 결국 커진다면 작은 자식 구성요소로 쪼개져야 합니다.

때때로 사용자에게 JSON 데이터 모델을 보여준 이후, 자료 모델이 잘 설계 되었다면 UI(혹은 구성요소 구조)가 잘 맞아떨어진다는 것을 알게될겁니다. UI와 자료 모델은 같은 *정보 설계구조*로 들러붙으려는 경향이 있기 때문입니다. 즉, UI를 구성요소들로 쪼개려는 작업은 크게 어렵지 않습니다. 확실하게 각각 하나의 부분이 되도록 쪼개세요.

![Component diagram](/react/img/blog/thinking-in-react-components.png)

이 간단한 어플리케이션에는 다섯개의 구성요소가 있습니다. 각 컴포넌트들이 대표하는 자료를 이탤릭체로 표기했습니다.

  1. **`FilterableProductTable` (orange):** 예제 전부를 포함합니다. 
  2. **`SearchBar` (blue):** 모든 *사용자 입력* 을 받습니다.
  3. **`ProductTable` (green):** *자료 모음*을 *사용자 입력*에 맞게 거르고 보여줍니다.
  4. **`ProductCategoryRow` (turquoise):** 각 *카테고리*의 제목을 보여줍니다.
  5. **`ProductRow` (red):** 각 *프로덕트*를 보여줍니다.

`ProductTable`을 보면 테이블 제목("Name", "Price" 라벨들을 포함한)은 컴포넌트가 아닌것을 알 수 있습니다. 이건 기호의 문제이고, 어느쪽으로 만들던 논쟁거리입니다. 예를들어 저는 *자료 모음*을 그리는 `ProductTable`의 의무라고 생각했기 때문에 남겨 두었습니다. 하지만 이 제목이 복잡해진다면 (예를들어 정렬을 추가할 여유가 있다거나 하는), 이건 확실히 `ProductTableHeader` 구성요소로 만드는게 맞을겁니다.

이제 모형에 들어있는 구성요소들에 대해 알아보았으니, 계층 구조로 만들어 봅시다. 이건 쉽습니다. 다른 구성요소 속에 들어있는 구성요소를 그저 자식으로 나타내면 됩니다.

  * `FilterableProductTable`
    * `SearchBar`
    * `ProductTable`
      * `ProductCategoryRow`
      * `ProductRow`

## 2단계: 정적 버젼을 만드세요. 

<iframe width="100%" height="300" src="http://jsfiddle.net/reactjs/yun1vgqb/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

계층구조의 구성요소들을 가지고 있으니, 이젠 어플리케이션을 구현할 시간입니다. 가장 쉬운 방법은 상호작용을 하지 않는채로 자료 모델을 이용해 UI를 그리는 것입니다. 정적 버젼을 만드는데는 적은 생각과 많은 노동이 필요하고, 상호작용을 추가하는데는 많은 생각과 적은 노동이 필요하기때문에 둘을 분리하는게 쉽습니다. 왜그런지 봅시다.

자료 모델을 그리는 어플리케이션의 정적버전을 만들기 위해서, 다른 구성요소에 재사용할 컴포넌트를 만들고 자료 전달을 위해 *props*를 사용하고 싶을 것입니다. 만약 *상태*라는 개념에 익숙하다면, 정적 버젼을 만들때 **절대 상태를 사용하지 마세요**. 상태는 오직 상호작용, 즉 가변적인 자료를 위해서만 준비되어 있습니다. 정적 버젼을 만들때는 필요가 없다는 겁니다.

껍데기부터 혹은 속알맹이부터 만들 수 있습니다. 즉 계층구조상 위에서부터 (`FilterableProductTable` 부터) 혹은 아래에서부터 (`ProductRow`), 어느 방향에서든 시작해도 됩니다. 통상 큰 프로젝트에서는 계층구조상 위에서부터 시작하는게 쉽고, 테스트를 작성할때는 아래에서부터 시작하는게 쉽습니다.

이 단계의 결과, 자료 모델을 그리는 재활용 가능한 구성요소의 라이브러리를 갖게 되었습니다. 정적버젼 이후로 구성요소들은 오직 `render()` 메서드만 갖고 있습니다. 계층구조상 가장 위의 구성요소 (`FilterableProductTable`)은 자료 모델을 prop으로 취할 것입니다. 자료 모델이 변했을 때, `React.render()`를 다시 부르면 업데이트 됩니다. 어떻게 UI가 업데이트 되는지 참 알기 쉽습니다. 자료가 바뀌어도 처리해야 할 복잡한 일이 아무것도 없습니다. React의 **단일 방향 자료 흐름** (혹은 *딘일방향 바인딩*)이 모든것을 모듈식으로, 추론하기 쉽게, 그리고 빠르게 유지해줍니다.

이 단계를 진행하는데 도움이 필요하시다면, [React 문서](http://facebook.github.io/react/docs/)를 참조하세요.

### A brief interlude: props vs state

There are two types of "model" data in React: props and state. It's important to understand the distinction between the two; skim [the official React docs](http://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html) if you aren't sure what the difference is.

## Step 3: Identify the minimal (but complete) representation of UI state

To make your UI interactive you need to be able to trigger changes to your underlying data model. React makes this easy with **state**.

To build your app correctly you first need to think of the minimal set of mutable state that your app needs. The key here is DRY: *Don't Repeat Yourself*. Figure out what the absolute minimal representation of the state of your application needs to be and compute everything else you need on-demand. For example, if you're building a TODO list, just keep an array of the TODO items around; don't keep a separate state variable for the count. Instead, when you want to render the TODO count simply take the length of the TODO items array.

Think of all of the pieces of data in our example application. We have:

  * The original list of products
  * The search text the user has entered
  * The value of the checkbox
  * The filtered list of products

Let's go through each one and figure out which one is state. Simply ask three questions about each piece of data:

  1. Is it passed in from a parent via props? If so, it probably isn't state.
  2. Does it change over time? If not, it probably isn't state.
  3. Can you compute it based on any other state or props in your component? If so, it's not state.

The original list of products is passed in as props, so that's not state. The search text and the checkbox seem to be state since they change over time and can't be computed from anything. And finally, the filtered list of products isn't state because it can be computed by combining the original list of products with the search text and value of the checkbox.

So finally, our state is:

  * The search text the user has entered
  * The value of the checkbox

## Step 4: Identify where your state should live

<iframe width="100%" height="300" src="http://jsfiddle.net/reactjs/zafjbw1e/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

OK, so we've identified what the minimal set of app state is. Next we need to identify which component mutates, or *owns*, this state.

Remember: React is all about one-way data flow down the component hierarchy. It may not be immediately clear which component should own what state. **This is often the most challenging part for newcomers to understand,** so follow these steps to figure it out:

For each piece of state in your application:

  * Identify every component that renders something based on that state.
  * Find a common owner component (a single component above all the components that need the state in the hierarchy).
  * Either the common owner or another component higher up in the hierarchy should own the state.
  * If you can't find a component where it makes sense to own the state, create a new component simply for holding the state and add it somewhere in the hierarchy above the common owner component.

Let's run through this strategy for our application:

  * `ProductTable` needs to filter the product list based on state and `SearchBar` needs to display the search text and checked state.
  * The common owner component is `FilterableProductTable`.
  * It conceptually makes sense for the filter text and checked value to live in `FilterableProductTable`

Cool, so we've decided that our state lives in `FilterableProductTable`. First, add a `getInitialState()` method to `FilterableProductTable` that returns `{filterText: '', inStockOnly: false}` to reflect the initial state of your application. Then pass `filterText` and `inStockOnly` to `ProductTable` and `SearchBar` as a prop. Finally, use these props to filter the rows in `ProductTable` and set the values of the form fields in `SearchBar`.

You can start seeing how your application will behave: set `filterText` to `"ball"` and refresh your app. You'll see the data table is updated correctly.

## Step 5: Add inverse data flow

<iframe width="100%" height="300" src="http://jsfiddle.net/reactjs/n47gckhr/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

So far we've built an app that renders correctly as a function of props and state flowing down the hierarchy. Now it's time to support data flowing the other way: the form components deep in the hierarchy need to update the state in `FilterableProductTable`.

React makes this data flow explicit to make it easy to understand how your program works, but it does require a little more typing than traditional two-way data binding. React provides an add-on called `ReactLink` to make this pattern as convenient as two-way binding, but for the purpose of this post we'll keep everything explicit.

If you try to type or check the box in the current version of the example you'll see that React ignores your input. This is intentional, as we've set the `value` prop of the `input` to always be equal to the `state` passed in from `FilterableProductTable`.

Let's think about what we want to happen. We want to make sure that whenever the user changes the form we update the state to reflect the user input. Since components should only update their own state, `FilterableProductTable` will pass a callback to `SearchBar` that will fire whenever the state should be updated. We can use the `onChange` event on the inputs to be notified of it. And the callback passed by `FilterableProductTable` will call `setState()` and the app will be updated.

Though this sounds like a lot it's really just a few lines of code. And it's really explicit how your data is flowing throughout the app.

## And that's it

Hopefully this gives you an idea of how to think about building components and applications with React. While it may be a little more typing than you're used to, remember that code is read far more than it's written, and it's extremely easy to read this modular, explicit code. As you start to build large libraries of components you'll appreciate this explicitness and modularity, and with code reuse your lines of code will start to shrink :)
