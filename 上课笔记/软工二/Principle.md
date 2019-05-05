## Principles

#### 1.Global Variable Consider Harmful

```
全局变量会被共享到其他的模块
```

- 一个模块的错误与更改会影响到其他每一个模块
- 重用降低
- 系统复杂度提高

#### 2.To be Explicit

```
显式的属性，调用，事件，子类
```

#### 3.Don't repeat

#### 4.Programming to Interface

#### 5.The Law of Demeter

- 每一个单元对于其他的单元只能拥有有限的知识
- 每个单元只能和朋友单元交谈
- 只和自己直接的朋友交谈

不可以出现 `A.b.method()`

#### 6.Interface Segregation Priciple

```
ISP,接口分离原则
即将一个统一的接口分离为多个独立接口
```

#### 7.Liskov Substitution Principle

```
里氏替换原则
```

#### 8.Favor Composition Over Inheritance

- 组合来替代继承

#### 9.Single Responsibility Principle

```
信息与行为除了要集中，还要联合起来表达一个内聚的概念，而不是单纯的堆砌
```

