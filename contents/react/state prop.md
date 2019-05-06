#### Prop

```
常常用于组件间的简单数据传输(父组件->子组件)
```

代码如下

```react
export default class index extends Component {
  state = {
    type: 'normal',
    count: 0,
  };

  render() {
    const Tick = (props) => {
      return (
        <div>
          <h1>Hello World!</h1>
          <h2>It is {props.date.toLocaleTimeString()}</h2>
        </div>
      );
    };

    return (
      <div className={styles.body}>
        <h1 className={styles.demo}>Demo</h1>
        <Tick date={new Date()}/>
      </div>
    );
  }
}
```

- 我们把 `date` 这一个 `prop`传递给了子组件
- `Tick`作为一个函数，接收 `prop`

#### State

```
状态作为一个私有的数据，完全受控于当前组件；
常常用于类内部
```

例如

```react
class Tick extends Component {
  state = {
    date: new Date(),
  };

  render() {
    return (
      <div>
        <h1>Hello world</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}</h2>
      </div>
    );
  }
}
```

还可以添加 **constructor** 来进行初始化

```react
  constructor(props) {
    super(props);
    this.state = { date: new Date() };
  }
```

#### 事件绑定

##### 1.简单事件

```react
  handleClick = (e) => {
    this.setState({ date: new Date() });
  };

render() {
    return (
      <div>
        <h1>Hello world</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}</h2>
        <Button onClick={this.handleClick}>Update</Button>
      </div>
    );
  }
```

或如下

```react

  render() {
    return (
      <div>
        <h1>Hello world</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}</h2>
        <Button onClick={(e) => {
          this.setState({ date: new Date() });
        }}>Update</Button>
      </div>
    );
  }
```

##### 2.事件传参

```react
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

