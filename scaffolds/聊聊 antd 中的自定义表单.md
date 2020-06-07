# antd 中自定义表单

![城市选择组件](http://oyy3cbpm3.bkt.clouddn.com/15379501603956.jpg)

项目组件当选择「公司项目」时，会显示额外输入框用以输入公司名称。

![项目填写组件](http://oyy3cbpm3.bkt.clouddn.com/15379502167528.jpg)
![项目填写组件](http://oyy3cbpm3.bkt.clouddn.com/15379501977105.jpg)


首先对「表单」进行定义，什么是表单？
能够获取用户输入内容的`UI`，比如`input`、`textarea`、`select`，`input`又可以设置`type`得到`radio`、`checkbox`、`file`、`date`、`password`、`number`、`email`等表单。

核心在于「获取用户输入」，虽然绝大部分场景上面的表单已经能够满足需求，但仍存在无法满足的情况，最典型的如`switch`，所以就出现了「自定义表单」。

自定义表单就是将已有表单组合、修改样式得到的表单，如`switch`本质上可以用`checkbox`实现，这也可能是没有提供原生`switch`的原因（移动端原生提供了）；`daterange`可以用两个`date`组合。

到如今组件化，大部分`UI`库都对已有的表单进行组件化，并且赋予更有语义化的名字，以`ant-design`为例，在`Data Entry`部分，有 18 个组件，除了熟悉的组件外，提供了`AutoComplete`、`Cascader`、`Mention`、`Rate`、`Transfer`等更加丰富的组件。

那么，为什么要写关于表单的博客呢？和其他组件相比，表单组件有什么特殊点呢？
如同上面所说，表单是用来获取用户输入的，所以相比其他展示型的组件，会和用户存在非常频繁的交互，并且，存在输入，就需要对输入进行「校验」，校验失败后也要给出对应的提示，帮助用户纠正错误。这在使用原生表单时（html 或者 antd 提供）都已经帮助我们处理好了，但自定义表单会存在一些问题，下面以`antd`表单为例说明。

## 取值与校验

就本质上来说，取值就是从表单获取到用户输入，并保存在变量中的这么一个过程。无论原生还是使用框架，都脱离不了这个本质。

### 原生 DOM

```html
<div>
    <input type="text" required="required" id="username" />
    <input type="password" required="required" id="password" />
    <button id="btn">登录</button>
</div>
```

```js
let value = undefiend;
const $username = document.getElementById('username');
const $password = document.getElementById('password');
const $btn = document.getElementById('btn');
$btn.onclick = function () {
    const username = $username.value;
    const password = $username.value;
    console.log('login', username, password);
}
```

原生取值，是在「需要使用」时，从表单中取出值再使用；而校验，原生就已经提供了。

### react

`react`的思想是声明式，以状态表达视图。在表单「每次发生改变」时，将值保存到`state`中，需要使用时再从`state`中获取。

```js
class Example extends React.PureComponent {
    state = {
        username: undefined,
        password: undefined,
    }
    handleUsernameChange = (e) => {
        this.setState({
            username: e.target.value,
        });
    }
    handlePasswordChange = (e) => {
        this.setState({
            password: e.target.value,
        });
    }
    handleClick = () => {
        const { username, password } = this.state;
        fetch('/api/login', { method: 'POST', body: { username, password } })
            .then(() => {
                // ...
            });
    }
    render() {
        const { username, password } = this.props;
        return (
            <div>
                <input value={username} onChange={this.handleUsernameChange} />
                <input value={password} onChange={this.handlePasswordChange} />
                <button onClick={this.handleClick}>登录</button>
            </div>
        );
    }
}
```

校验，需要额外添加文本与样式，并在提交时（或输入时）进行判断，显隐对应的样式（状态）。

### angular

`angular`有两种方式，既可以原生方式获取，也可以更简单的方式。

```html
<div>
    <input #username />
    <input #password />
    <button (click)="login(username, password)">
</div>
```

```js
// FormComponent
@Component({
    selector: 'login-form',
    // ... 省略
})
export class LoginForm {
    login(username: HTMLInputElement, password: HTMLInputElement) {
        console.log('login', username.value, password.value);
    }
}
```

双向绑定

```html
<div>
    <input [(ngModel)]="model.username" />
    <input [(ngModel)]="model.password" />
    <button (click)="login(username, password)">
</div>
```

```js
// FormComponent
@Component({
    selector: 'login-form',
    // ... 省略
})
export class LoginForm {
    login(username: HTMLInputElement, password: HTMLInputElement) {
        console.log('login', username.value, password.value);
    }
}
```

校验方式也是类似。

### vue

和`angular`非常类似，包括取值与校验。

```vue
<template>
    <input :value="username />
    <input type="password" :value="username />
    <button @click="login(username, password)">登录</button>
</template>
<script>
export default {
    data() {
        return {
            username: '',
            password: '',
        };
    },
    methods: {
        login(username, password) {
            console.log('login', username, password);
        }
    },
}
</script>
```

## 自定义表单用例

如下是`ant-design`中定义的一个价格输入组件，该组件由`Input`和`Select`两个组件组合而成。提交时可同时获取到「价格」与「货币单位」两个值。

![ant-design 自定义表单控件](http://oyy3cbpm3.bkt.clouddn.com/15359431976484.jpg)

由于并不是所有业务都需要这种「价格输入组件」，所以任何组件库都不会默认提供，而需要使用者组合成适合自己业务的。
虽然每个项目的需求都不同，但总的来说只有两种情况

### 二次封装

即在基础表单的基础上，加上特有的逻辑或者数据。
如城市选择组件，虽然感觉通用但没有组件库提供现成的，就需要自己借助`Select`或者`Cascader`封装城市数据。

特有逻辑，如两个`Input`，同时只能存在一个有输入值。

### 多个字段

基础表单，只能提供单个值，而我们的业务可能需要表单暴露出两个甚至更多的值，如同上面的价格输入组件，同时暴露了价格与货币单位两个值。
实际上这类场景占大部分，除了封装一个特殊的表单组件外，为了更好的维护，将一个复杂的表单拆分成多个子表单，也属于这类情况。

## 获取自定义表单的值

自定义组件写好后，为了和使用习惯一致，也可以通过事件暴露出需要的值比如使用`onChange`。
价格输入组件，在用户输入值、改变货币单位时，可以获取到当前的值，并通过`props.onChange`向父组件抛出事件并传值，如下是上面价格输入组件的代码。

```js
class PriceInput extends React.Component {
  constructor(props) {
    super(props);

    const value = props.value || {};
    // 初始值为 0、rmb
    this.state = {
      number: value.number || 0,
      currency: value.currency || 'rmb',
    };
  }
  render() {
    const { size } = this.props;
    const state = this.state;
    return (
      <span>
        <Input
          type="text"
          size={size}
          value={state.number}
          onChange={this.handleNumberChange}
          style={{ width: '65%', marginRight: '3%' }}
        />
        <Select
          value={state.currency}
          size={size}
          style={{ width: '32%' }}
          onChange={this.handleCurrencyChange}
        >
          <Option value="rmb">RMB</Option>
          <Option value="dollar">Dollar</Option>
        </Select>
      </span>
    );
  }
}
```

正如我们之前看到的，价格输入组件由`Input`和`Select`组合而成，并且对应的字段名为`number`与`currency`。重点在于`this.handleNumberChange`和`this.handleCurrencyChange`，从函数名可以看出，分别是用来处理`Input`改变与`Select`改变。
要怎么处理呢？

```js
  // 处理 Input 输入，保存输入的值，并调用 triggerChange，实际就是调用 props.onChange
  handleNumberChange = (e) => {
    const number = parseInt(e.target.value || 0, 10);
    if (isNaN(number)) {
      return;
    }
    if (!('value' in this.props)) {
      this.setState({ number });
    }
    this.triggerChange({ number });
  }
  // 处理下拉框改变，同样保存值后调用 props.onChange
  handleCurrencyChange = (currency) => {
    if (!('value' in this.props)) {
      this.setState({ currency });
    }
    this.triggerChange({ currency });
  }
  triggerChange = (changedValue) => {
    // Should provide an event to pass value to Form.
    const onChange = this.props.onChange;
    if (onChange) {
      onChange(Object.assign({}, this.state, changedValue));
    }
  }
```

无论`Input`改变还是`Select`改变，都调用`onChange`函数，并将**最新的完整的值**传入作为参数

```js
// 当前的值
state = {
    number: 100,
    currency: 'rmb',
};

// 如果改变了`number`，实际上是

const currentValues = Object.assign({}, { number: 100, currency: 'rmb' }, { number: 101 });
onChange(currentValues);
```

使用该价格输入组件的代码：

```js
class App extends React.Component {
    state = {
        value: undefined,
    }
    handleChange = (value) => {
        // value === { number: 101, currency: 'rmb' };
        this.setState({ value });
    }
    render() {
        return (
            <div>
                <PriceInput onChange={this.handleChange} />
            </div>
        );
    }
}
```

### Form 组件方便获取值

上面代码虽然看起来清晰，但如果表单非常多，就会存在非常多的模板代码，并且在原生`DOM`时，取值也没有这么麻烦，直接获取`DOM`上的`value`属性即可。
所以`antd`提供了`Form`组件来优化这一问题。简单来说，经过`Form.create()`包装的组件在`props`属性上会增加`form`属性，这其实是一个`store`，表单的值都由`form`维护。

上面价格输入组件，如果使用`Form`组件，可以改成这样：

```js
@Form.create({
    onValuesChange: (props, changedValues, allValues) => {
        props.onChange({
            ...allValues,
            ...changedValues,
        });
    },
})
class PriceInput extends React.Component {
  constructor(props) {
    super(props);

    const value = props.value || {};
    // 初始值为 0、rmb
    this.state = {
      number: value.number || 0,
      currency: value.currency || 'rmb',
    };
  }
  render() {
    const state = this.state;
    return (
      <span>
        {getFieldDecorator('number')(
            <Input
                style={{ width: '65%', marginRight: '3%' }}
            />
        )}
        {getFieldDecorator('currency')(
            <Select
                style={{ width: '32%' }}
            >
                <Option value="rmb">RMB</Option>
                <Option value="dollar">Dollar</Option>
            </Select>
        )}
      </span>
    );
  }
}
```

省去了我们使用`state`维护表单值的工作。

### 多个值的表单读取值时的问题

还是以价格输入组件为例，

```js
@Form.create()
class App extends React.Component {
    handleClick = () => {
        const { getFieldsValue } = this.props.form;
        const values = getFieldsValue();
        console.log(values);
    }
    render() {
        const { getFieldDecorator } = this.props.form;
        return (
            <div>
                {
                    getFieldDecorator('price')(
                        <PriceInput />
                    )
                }
                <Button onClick={this.handleClick}>提交</Button>
            </div>
        );
    }
}
```

使用`getFieldsValue`得到的是如下数据：

```js
{
    price: {
        number: 101,
        currency: 'rmb',
    },
}
```

但我们希望提交的是如下格式的数据，就需要额外的处理。

```js
handleClick = () => {
    const { getFieldsValue } = this.props.form;
    const values = getFieldsValue();
    console.log(values);
    // 这样提交的就是 { number: 101, currency: 'rmb' } 格式的数据
    this.submit({
        ...values.price,
    });
}
```

在使用价格组件的地方，都需要做这样的处理。

### 获取值之后值的转化

假设在我们项目中，有一个「用户搜索框」，基于`Select`封装了搜索员工的逻辑，每次选择员工后，向外暴露的是如下格式的数据：

```js
{
    userId: 10,
    ldap: 'xxx',
    name: 'xxx',
}
```

因为不同的接口可能需要不同的值，有些需要`userId`，有些需要`ldap`，所以无法只暴露`userId`或者`ldap`。这也导致了和上面类似的问题，每次都需要额外的逻辑处理值。

```js
handleClick = () => {
    const { getFieldsValue } = this.props.form;
    const values = getFieldsValue();
    console.log(values); // { staff: { userId: 10, ldap: 'xxx', name: 'xxx' } }
    this.submit({
        userId: values.staff.userId,
    });
}
```

但这个问题可以用`normalize`解决。

## 对自定义表单进行校验

前面提到，表单校验的本质，就是对值进行判断，并根据判断结果，展示提示信息及特定样式。在`antd`中这项工作是由`Form.Item`完成，它根据校验结果会改变内部组件的样式，以及在内部组件下方展示提示信息。

```js
@Form.create()
class App extends React.Component {
  handleSubmit = e => {
    e.preventDefault();
    this.props.form.validateFields((err, values) => {
      if (!err) {
        console.log('Received values of form: ', values);
      }
    });
  };

  checkPrice = (rule, value, callback) => {
    if (value.number > 0) {
      callback();
      return;
    }
    callback('Price must greater than zero!');
  };

  render() {
    const { getFieldDecorator } = this.props.form;
    return (
      <Form layout="inline" onSubmit={this.handleSubmit}>
        <FormItem label="Price">
          {getFieldDecorator('price', {
            initialValue: { number: 0, currency: 'rmb' },
            rules: [{ validator: this.checkPrice }]
          })(<PriceInput />)}
        </FormItem>
        <FormItem>
          <Button type="primary" htmlType="submit">
            Submit
          </Button>
        </FormItem>
      </Form>
    );
  }
}
```

可以看到配置了`rules`，校验规则为`this.checkPrice`，而在该校验规则内就可以拿到用户表单值进行判断，根据结果使用`callback`反馈输入值是否符合预期（校验通过）还是不符合预期（校验失败）。

### 校验规则无法封装到表单内

但可见的问题是，每次使用`PriceInput`组件时，都需要设置校验规则。

> 这在很多人看来很正常，因为使用方可能存在不同的校验要求，也许某些场景下允许`currency`为空呢？

但就实际情况来说，组件的规则就和「特有逻辑」一样，是和组件强相关的，在任何地方使用它，规则都是一样的。在这种场景下，如果能简化这部分配置工作还是很方便的。

以及前面提到的「表单拆分」场景，校验规则是已知并且配置好的，没有理由再在使用时再设置一次校验规则。

比如一个「简历填写表单」，总字段可能有数十个，所以为了更方便的维护，可以分为「个人信息」表单、「工作经历」表单，「其他」表单，

```js
class ResumeForm extends React.Component {
    render() {
        return (
            <PersonInfoForm />
            <ExperienceForm />
            <OtherForm />
        );
    }
}

class App extends React.Component {
    handleClick = () => {
        const { validateFields } = this.props.form;
        validateFields((err, values) => {
            if (err) return;
            console.log(values);
        });
    }
    render() {
        return (
            <div>
                {getFieldDecorator('resume')(
                    <Resume />
                )}
                <Button onClick={this.handleClick}>提交</Button>
            </div>
        );
    }
}
```

可见，校验规则是确定的，并不会因为`Resume`组件在不同地方使用而改变，所以将校验规则放在组件内是比较方便的，用户完全不需要了解应该怎么配置规则，只需要知道，如果校验不通过，就会展示错误提示；如果校验通过，就能拿到需要的值。

### Form.Item 内无法存在两个表单

更严格的说法是，`Form.Item`无法生成两个校验提示。还是以开始的「价格输入组件」为例，如果没有填写`number`或者没有选择`currency`，我们可以在组件下方给出对应的提示，但如果考虑更好的用户体验，是否能在`Input`下方显示「没有填写价格」提示，在`Select`下方显示「没有选择单位」提示呢？

可想而知是不行的，因为`Form.Item`就是用来生成单个提示。

### 解决方案

其实解决方案有多种，先来说最简单粗暴的方式。

#### form 属性

#### ref

## 默认值不生效

`antd`的表单组件，都提供了`defaultValue`属性，用以配置默认值：

```js
class App extends React.Component {
    state = { name: undefined }
    handleChange = (e) => {
        this.setState({
            name: e.target.value,
        });
    }
    render() {
        const { value } = this.state;
        return (
            <Input
                defaultValue="wuya"
                value={value}
                onChange={this.handleChange}
            />
        );
    }
}
```

渲染后可以看到内表单有`wuya`默认值。但如果默认值是从接口请求得到的，就无法达到我们的预期效果。

```js
class App extends React.Component {
    state = { 
        defaultValue: 
        name: undefined,
    }
    componentDidMount() {
        // 模拟请求接口
        setTimeout(() => {
            this.setState({
                defaultValue: 'wuya',
            });
        }, 2000);
    }
    handleChange = (e) => {
        this.setState({
            name: e.target.value,
        });
    }
    render() {
        const { defaultValue, value } = this.state;
        return (
            <Input
                defaultValue={defaultVlaue}
                value={value}
                onChange={this.handleChange}
            />
        );
    }
}
```

### 默认值如何才能生效

### 设置默认值的正确方式

https://www.youtube.com/watch?v=ey7H8h4ERHg&feature=youtu.be


## 参考

- [Form](https://en.wikipedia.org/wiki/Form_(HTML))
- [forms](https://angular.io/guide/forms)

