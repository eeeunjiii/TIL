### 이벤트 state props 그리고 render 함수

> `HTML` 의 링크를 누르면 그에 맞는 내용이 출력되고, `CSS`의 링크를 누르면 CSS에 대한 설명이 출력되도록, 동적으로 작동하도록 하는 것이 목표이다.
> 
- 리액트에서 state의 값이 변경되면 그 state를 가지고 있는 컴포넌트의 `render` 함수가 다시 호출된다.
- 그 `render` 함수가 다시 호출됨에 따라서 그 `render`함수 하위에 있는 컴포넌트 각자가 가지고 있는 `render`함수들도 호출된다. → 즉 화면이 다시 구성된다.
    - `render` 함수의 역할: 어떤 HTML을 그릴 것인가 결정

```jsx
constructor(props){
    super(props);
    this.state = {
    mode:'welcome', // <-
    subject:{title:'WEB', sub:'world wide web'},
    welcome:{title: 'Welcome', desc:'Hello React'}, // <-
    contents:[
      {id:1, title:'HTML', desc:'HTMl is for information'},
      {id:2, title:'CSS', desc:'CSS is for design'},
      {id:3, title:'JavaScript', desc:'JavaScript is for interactive'}
    ]
    }
  }
```

- `mode` 부분을 추가해서 어떤 모드인지에 따라 출력하 값을 다르게 설정할 것이다.

<aside>
📌 자바스크립트에서 `==`는 서로 다른 유형의 두 변수의 값만 비교한다면 `===`는 더 엄격하게 **값과 자료형**을 비교하는 것이다.

</aside>

```jsx
render() {
    var _title, _desc = null;
    if(this.state.mode === 'welcome'){
      _title = this.state.welcome.title;
      _desc = this.state.welcome.desc;
    } else if(this.state.mode === 'read'){
      _title = this.state.contents[0].title;
      _desc = this.state.contents[0].desc;
    }
    
    return (
        <div className='App'>
          <Subject title={this.state.subject.title} 
          sub={this.state.subject.sub}></Subject>
          <TOC data={this.state.contents}></TOC>
          <Content title={_title} desc={_desc}></Content>
        </div>
    )
  }
```

- `render` 함수 내에 `mode`에 따른 출력 내용을 다르게 하기 위해 조건문을 추가했다. 위에서 `mode`를 `welcome`으로 지정했기 때문에 그에 해당하는 제목과 내용이 출력된다.
- 이제 자바스크립트를 이용해서 각 링크를 눌렀을 때 거기에 맞는 `mode`가 설정되도록 코드를 작성하면 된다.

---

### 이벤트 설치

```jsx
return (
    <div className='App'>
      {/* <Subject title={this.state.subject.title} 
        sub={this.state.subject.sub}></Subject> */}
        <header>
            <h1><a href="/">{this.state.subject.title}</a></h1>
            {this.state.subject.sub}
        </header>
        <TOC data={this.state.contents}></TOC>
        <Content title={_title} desc={_desc}></Content>
    </div>
)

```

- 우선 `render` 함수 내에 원래 `Subject` 컴포넌트를 활용했는데 이벤트의 동작을 이해하기 위해서 잠깐 `Subject` 컴포넌트를 풀어서 `App` 자바스크립트에 옮겼다.

```jsx
<header>
	<h1><a href="/" onClick={function() {
  alert("hi");
  }}>{this.state.subject.title}</a></h1>
  {this.state.subject.sub}
</header>
```

- `onClick` 이벤트를 생성하여 해당 이벤트를 실행했을 때 `function`에 있는 동작이 실행되도록 한다.
- 기본적으로 리액트는 이벤트가 실행되면 reload를 하는데 이러한 기본 동작을 못하도록 막고 싶은 것이 당장의 목표이다. 이벤트가 실행되어서 `function`이 실행되면 `function`의 첫번째 인자로 이벤트의 객체가 생성된다.
- 그 인자를 `e`로 설정해놓고, 그 내부의 속성들 중 `e.preventDefault`는 위의 기본 동작들을 못하도록 막아준다.
    - 이렇게 해주면 페이지가 전환되지 않는다.

---

### 이벤트에서 state 변경하기

> 이제 이전에 설정한 `state`와 이벤트만 연결해주면 된다.
> 

→ 처음에는 단순히 `WEB`을 눌렀을 때 `welcome`모드에 해당하는 제목과 내용이 출력되어야 하니까 `function` 안에 `this.state.mode='welcome'`으로 코드를 작성해주면 되는 줄 알았다. 이 코드는 **두 가지의 문제점**을 갖고 있다.

1. 이벤트가 호출되었을 때 실행되는 함수 안에서는 `this`의 값이 아무것도 설정되어 있지 않다. 즉 `this`가 가리키고 있는 것이 없다. 따라서 `state`가 정의되어 있지 않다는 에러가 발생하게 된다.

```jsx
function(e) {
	e.preventDefault();
  // this.state.mode='welcome'
}.bind(this)
```

- 위와 같이 `.bind(this)`를 `function`의 마지막에 추가해줘야 한다. 하지만 에러는 발생하지 않지만 값이 변경되지 않는다.
1. 위와 같이 코드를 작성하면 리액트는 `state`가 변경되었다는 것을 알지 못한다. 이를 해결하기 위해서는 아래와 같이 리액트에게 `state`가 변경되었다는 것을 알려주는 코드를 추가해야 한다.

```jsx
function(e) {
	e.preventDefault();
  this.setState({
	  mode:'welcome'
  });
}.bind(this)
```

---

### 이벤트 `bind` 함수 이해하기

- 원래는 컴포넌트 자기 자신을 가리켜야 하는데, `function` 함수에 `.bind(this)`를 붙이지 않으면 `this`는 아무것도 가리키지 않는다.
- 즉, `App` 컴포넌트를 가리키는 `this`를 `function`에 직접 주입해주는 과정이 필요하다. 무조건 사용하면 된다.

---

### 이벤트 `setState` 함수 이해하기

- `state`에 대한 설정이 끝난 상태에서 이를 동적으로 변경하려고 하면 안된다. 리액트 입장에서는 값이 변경된 것을 모르기 때문에 rendering을 하지 않는다. 이럴 때 사용하는 것이 `setState`라는 함수이다.