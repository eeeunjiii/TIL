### 컴포넌트 만들기 (1)

- 컴포넌트를 만들 때, 컴포넌트는 반드시 하나의 최상위 태그로 시작해야 한다.
- 하나의 최상위 태그만 사용해야 한다.

```java
class App extends Component {
  render() {
    return (
        <div className='App'>
          <Subject title="WEB" sub="world wide web"></Subject>
          {/* <Subject title="React" sub="For UI"></Subject> */}
          <TOC></TOC>
          <Content title="HTML" desc="HTML is HyperText Markup Language."></Content>
          {/* <Content title="React" desc="React content."></Content> */}
        </div>
    )
  }
}
```

- 위의 코드에서 최상위 태그는 `<div>`임을 알 수 있다.

---

### Component 파일로 분리하기

```jsx
import React, {Component} from 'react'

class Content extends Component{
    render(){
      return (
        <article>
              <h2>{this.props.title}</h2>
              {this.props.desc}
          </article>
      )
    }
  }
  
  export default Content;
```

- 작성된 내용을 보여주는 부분인 `Content` 컴포넌트를 다른 파일로 분리했다.
- 항상 React 파일을 작성할 때는 `import React`를 추가해야 한다.
- 자바와 동일하게 리액트에도 `this`를 사용한다. 따라서 객체 자신을 가리킨다.
- `export` 파일을 밖으로 내보낼 때 사용한다. 즉, 다른 페이지의 함수나 클래스를 외부에서 사용할 수 있게 해준다.
    - `export default`는 객체의 이름을 임의로 설정할 수 있게 해준다. 리액트는 컴포넌트 단위로 파일을 관리하기 때문에 하나의 클래스나 메소드를 가져오면 되기 때문에 보통 `default`를 붙여서 많이 사용한다.

```jsx
class App extends Component {
  render() {
    return (
        <div className='App'>
          <Subject title="WEB" sub="world wide web"></Subject>
          {/* <Subject title="React" sub="For UI"></Subject> */}
          <TOC></TOC>
          <Content title="HTML" desc="HTML is HyperText Markup Language."></Content>
          {/* <Content title="React" desc="React content."></Content> */}
        </div>
    )
  }
}
```

- `App` 클래스 코드에서 각 컴포넌트에 속성(`props`)을 부여하며 값을 변경할 수 있다.
- 위의 코드에서 `this.props.{속성값}`코드를 통해 값을 받아온다.

---

### State 소개

- `props`: 사용자가 컴포넌트를 사용하는 입장에서 중요한 것
- `state`: `props` 값에 따라서 내부의 구현에 필요한 데이터들

```jsx
class App extends Component {
constructor(props){
  super(props);
  this.state = {
    Subject:{title:'WEB', sub:'world wide web'} 
  };
}

/* render 메소드
* <Subject title={this.state.Subject.title} sub={this.state.Subject.sub}></Subject>
*/

```

- 계속 `<Subject title="WEB" sub="world wide web"></Subject>` 이러한 방식으로 코드를 주는 것보다 기본적인 값이라면 미리 설정을 해두는 것이 더 좋다.
- 그때 사용하는 것이 **생성자**이다. `render` 메소드보다 먼저 실행된다.
- `this.state`를 이용해서 설정을 한다. 위의 코드에서는 `Subject`라는 컴포넌트의 `title`과 `sub`라는 props를 state로 설정한 것이다. (굳이 클래스 이름과 춰줄 필요는 없다.)
- `render` 메소드에서 사용할 때는 그대로 그 경로를 따라가서 작성해주면 된다.

---

### Key

```jsx
/*App.js*/
constructor(props){
    super(props);
    this.state = {
    subject:{title:'WEB', sub:'world wide web'},
    contents:[
      {id:1, title:'HTML', desc:'HTMl is for information'},
      {id:2, title:'CSS', desc:'CSS is for design'},
      {id:3, title:'JavaScript', desc:'JavaScript is for interactive'}
    ]
}

<TOC data={this.state.contents}></TOC>

/*TOC.js*/
class TOC extends Component{
    render() {
        var lists=[];
        var data=this.props.data;
        var i=0;
        
        while(i<data.length){
            lists.push(<li key={data[i].id}><a href = {"/content/"+data[i].id}>{data[i].title}</a></li>);
            i=i+1;
        }

      return(
        <nav>
              <ul>
                  {lists}
              </ul>
          </nav>
      )
    }
}

```

- `<li>`에 `key`가 없으면 에러가 발생한다. 각각의 요소들이 `key` 값을 가져야 하는 것이다.
    - 위의 코드에서는 `data.id`가 그 역할을 한다.