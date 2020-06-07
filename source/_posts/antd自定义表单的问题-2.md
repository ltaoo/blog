---
title: antd 自定义表单的问题 - 2
categories: React
tags:
- react
- antd
- form
date: 2018/10/23
---

{% post_link antd自定义表单的问题-1 %}

<hr >

前面提到，「基本信息」包含姓名、出生年月、性别、城市以及邮箱共 5 个输入项。而其中性别、省市与邮箱封装为了单独的组件，因为这类组件**包含特有的数据或者逻辑**。

- 性别选择，因为有性别对应的 value。当然也可以将性别数据作为常量引入。
- 省市选择，包含了省市信息。
- 邮箱，有补全邮箱后缀等逻辑。

`antd`官网中「自定义表单组件」就是一个有「特有逻辑」的组件，它同时包含两个字段。

<!--more-->

## 一、自定义表单标准代码分析

以省市选择为例来说明如何封装一个自定义表单组件。

```js
/**
 * @file 中国省市选择，代码来源 https://gist.github.com/afc163/7582f35654fd03d5be7009444345ea17
 */
import React from 'react';
import { Cascader } from 'antd';
import provinces from 'china-division/dist/provinces.json';
import cities from 'china-division/dist/cities.json';
import areas from 'china-division/dist/areas.json';

areas.forEach(area => {
  const matchCity = cities.filter(city => city.code === area.cityCode)[0];
  if (matchCity) {
    matchCity.children = matchCity.children || [];
    matchCity.children.push({
      label: area.name,
      value: area.code
    });
  }
});

cities.forEach(city => {
  const matchProvince = provinces.filter(
    province => province.code === city.provinceCode
  )[0];
  if (matchProvince) {
    matchProvince.children = matchProvince.children || [];
    matchProvince.children.push({
      label: city.name,
      value: city.code,
      children: city.children
    });
  }
});

const options = provinces.map(province => ({
  label: province.name,
  value: province.code,
  children: province.children
}));

export default class CitySelect extends React.Component {
  constructor(props) {
    super(props);

    const { value } = props;
    this.state = {
      value,
    };
  }
  componentWillReceiveProps(nextProps) {
    if ('value' in nextProps) {
      console.log(nextProps.value);
      this.setState({
        value: nextProps.value,
      });
    }
  }
  handleChange = (value) => {
    const { onChange } = this.props;
    this.setState({
      value,
    });
    if (onChange) {
      onChange(value);
    }
  }
  render() {
    const { value } = this.state;
    return (
      <Cascader
        value={value}
        options={options}
        showSearch
        placeholder="请选择地址"
        onChange={this.handleChange}
      />
    );
  }
}
```

### 1、代码说明

这段代码属于标准的「antd 自定义表单」，既可作为普通表单使用，也可配合`antd`中的`Form`组件使用。
可以看到有`constructor`、`componentWillReceiveProps`和`handleChange`，这三个方法都有各自的作用。
首先，`handleChange`方法响应表单值的改变，并调用`props.onChange`方法，实现了向父组件通信，将数据传递给父组件。

`constructor`是为了配合`initialValue`，当配置了`initialValue`时，在`constructor`中可以从`props.value`上获取到对应值。

而`componentWillReceiveProps`是为了配合`resetFields`以及`setFields`方法，能够从父组件直接控制表单的值，以及`initialValue`如果会发生改变，比如从接口中获取值，也是通过这里实现赋值的。

### 2、通过组合得到的自定义表单组件

性别选择和邮箱输入组件同理，所以我们的`BasicInfoForm`代码应该如下：

```js
/**
 * @file 基本信息表单
 */
import React from 'react';
import {
  Form,
  Input,
  DatePicker,
} from 'antd';

import SexSelect from '../SexSelect';
import CitySelect from '../CitySelect';
import EmailInput from '../EmailInput';

export default class BasicInfoForm extends React.PureComponent {
  render() {
    return (
      <div className="basic__content">
        <Form.Item label="姓名">
          <Input placeholder="请输入姓名" />
        </Form.Item>
        <Form.Item label="出生年月">
          <DatePicker placeholder="请选择出生年月" />
        </Form.Item>
        <Form.Item label="性别">
          <SexSelect style={{ width: '100%' }} placeholder="请选择性别" />
        </Form.Item>
        <Form.Item label="所在城市">
          <CitySelect />
        </Form.Item>
        <Form.Item label="邮箱">
          <EmailInput placeholder="请输入邮箱" />
        </Form.Item>
      </div>
    );
  }
}
```

虽然实现了 UI，但这并不是一个「表单组件」，如果希望该组件是一个「自定义表单组件」，应该和上面的省市选择一样，实现`constructor`、`componentWillReceiveProps`和`handleChange`方法，前两个好说，问题就在于`handleChange`方法，由于存在 5 个表单，所以需要每个表单发生改变时，都调用`props.onChange`，那就需要有

- handleNameChange
- handleBirthdayChange
- handleSexChange
- handleCityChange
- handleEmailChange

### 3、onValueChange 简化获取多个表单值

幸好借助`antd`的`Form`组件可以简化这部分代码，

```js
@Form.create({
  // 当表单值发生改变时都会调用该方法
  onValuesChange(props, changed, values) {
    const { onChange } = props;
    if (onChange) {
      onChange({
        ...values,
        ...changed,
      });
    }
  },
})
export default class BasicInfoForm extends React.PureComponent {
  render() {
    const { getFieldDecorator } = this.props.form;
    return (
      <div className="basic__content">
        <Form.Item label="姓名">
          {getFieldDecorator('name')(<Input placeholder="请输入姓名" />)}
        </Form.Item>
        <Form.Item label="出生年月">
          {getFieldDecorator('birthday')(
            <DatePicker placeholder="请选择出生年月" />
          )}
        </Form.Item>
        <Form.Item label="性别">
          {getFieldDecorator('sex')(
            <SexSelect style={{ width: '100%' }} placeholder="请选择性别" />
          )}
        </Form.Item>
        <Form.Item label="所在城市">
          {getFieldDecorator('city')(
            <CitySelect />
          )}
        </Form.Item>
        <Form.Item label="邮箱">
          {getFieldDecorator('email')(
            <EmailInput placeholder="请输入邮箱" />
          )}
        </Form.Item>
      </div>
    );
  }
}
```

将组件替换掉我们页面组件中「基本信息」相关的代码，这是线上示例「 [封装基本信息表单](https://codesandbox.io/s/m9o5lznjlp)」。

这里是我们实际输入后能够获取到的数据

```js
{
  "basic": {
    "name": "ltaoo",
    "birthday": "2018-10-01T03:33:44.541Z",
    "sex": 0,
    "city": [
      "33",
      "3301",
      "330105"
    ],
    "email": "litaowork@aliyun.com"
  },
  "work": [
    "深圳联友科技",
    "杭州群核科技"
  ],
  "projects": [
    {
      "title": "BD System",
      "type": 1,
      "company": "杭州群核科技"
    }
  ]
} 
```

### 4、组合组件后带来的问题

OK，能满足我们**获取值**的需求，但存在两个问题

- 1、丢失了校验规则
- 2、获取到的是`basic`字段，我们需要的是`basic`字段的**值**。


## 二、恢复丢失的校验规则

如果有实际试用过该代码的人可能会有疑问，输入邮箱时会对输入内容进行校验啊，为什么说「丢失了校验规则」呢？
实际上**即使邮箱格式不正确并且有错误提示**，但点击「保存」后还是可以**获取表单值**，而开始的例子是不能的，并且会将页面滚动到邮箱输入处。

最直观的感受是什么都不填，直接点击「保存」按钮，[最开始的实现](https://codesandbox.io/s/xl8v6olw0o) 是可以正确校验的，而 [拆分为自定义表单组件](https://codesandbox.io/s/m9o5lznjlp) 后，点击按钮会通过校验，直接打印出当前的表单值。

### 1、自定义校验规则

参考`antd`中的自定义表单，如果需要对自定义表单进行校验，需要通过自定义`validator`实现，代码如下：

```js
import React from 'react';
import { Icon, Form, Card, Button, Select, Input } from 'antd';

import BasicInfoForm from '../components/BasicInfoForm';

function checkBasicInfo(rule, values, callback) {
  console.log(rule, values);
  if (!values) {
    callback('请输入基本信息');
    return;
  }
  const emailRegexp = /[\w!#$%&'*+/=?^_`{|}~-]+(?:\.[\w!#$%&'*+/=?^_`{|}~-]+)*@(?:[\w](?:[\w-]*[\w])?\.)+[\w](?:[\w-]*[\w])?/;
  if (emailRegexp.exec(values.email)) {
    callback();
    return;
  }
  callback('请检查邮箱格式');
}

@Form.create()
export default class App extends React.Component {
  save = () => {
    const { validateFieldsAndScroll } = this.props.form;
    validateFieldsAndScroll((err, values) => {
      if (err) {
        return;
      }
      const body = JSON.stringify(values, null, 2);
      console.log(body);
    });
  };
  
  render() {
    const { getFieldDecorator, getFieldValue } = this.props.form;
    // 省略部分代码
    return (
      <div className="resume">
        <Card
          title="基本信息"
          style={{ marginBottom: 10 }}
          className="resume__basic"
        >
          <div className="basic__content">
            {getFieldDecorator('basic', {
              rules: [
                {
                  validator: checkBasicInfo
                }
              ]
            })(<BasicInfoForm />)}
          </div>
        </Card>
        {/* 省略部分代码 */}
        <Button type="primary" onClick={this.save}>
          保存
        </Button>
      </div>
    );
  }
}
```

直接点击「保存」按钮后，发现虽然没有直接打印表单值，但页面上也没有显示错误信息，只有控制台显示`async-validator: ["请输入基本信息"]`，这说明校验规则的确生效了。
这是因为**错误提示是由`Form.Item`显示的**，必须将`BasicInfoForm`放在`Form.Item`组件内才会显示我们在`callback`传入的错误信息。

![Form.Item 组件](http://oyy3cbpm3.bkt.clouddn.com/15379691339723.jpg)

但是给`BasicInfoForm`包裹`Form.Item`后，虽然错误信息显示，但只会出现在最下方，无法实现在实际错误的表单下方显示，并且明显校验规则还需要我们再实现一次。[线上实例](https://codesandbox.io/s/88vvl45r92)

> 这也是一个`Form.Item`组件内无法同时存在两个及以上`getFieldDecorator`的原因。

### 2、更友好的错误展示

这两个缺点都是非常不友好的，如果希望使用`Form.Item`提供的错误展示机制，正确地在表单下方展示，要怎么做呢？

想到最开始的实现代码，虽然不怎么优雅，但校验却实实在在有用，能否直接复用呢？所以问题就是，为什么这样封装一层，原先的校验规则就失效了呢？

```js
@Form.create({
    // 省略...
})
export default class BasicInfoForm extends React.PureComponent {
  render() {
    const { getFieldDecorator } = this.props.form;
    return (
      <div className="basic__content">
        <Form.Item label="姓名">
          {getFieldDecorator('name', {
            rules: [
              {
                required: true,
                message: '请输入姓名',
              },
            ],
          })(<Input placeholder="请输入姓名" />)}
        </Form.Item>
        <Form.Item label="出生年月">
          {getFieldDecorator('birthday', {
            rules: [
              {
                required: true,
                message: '请选择出生年月',
              },
            ],
          })(
            <DatePicker placeholder="请选择出生年月" />
          )}
        </Form.Item>
        <Form.Item label="性别">
          {getFieldDecorator('sex', {
            rules: [
              {
                required: true,
                message: '请选择性别',
              },
            ],
          })(
            <SexSelect style={{ width: '100%' }} placeholder="请选择性别" />
          )}
        </Form.Item>
        <Form.Item label="所在城市">
          {getFieldDecorator('city', {
            rules: [
              {
                required: true,
                message: '请选择所在城市',
              },
            ],
          })(
            <CitySelect />
          )}
        </Form.Item>
        <Form.Item label="邮箱">
          {getFieldDecorator('email', {
            rules: [
              {
                required: true,
                message: '请输入邮箱',
              },
              {
                type: 'email',
                message: '邮箱格式不正确',
              },
            ],
          })(
            <EmailInput placeholder="请输入邮箱" />
          )}
        </Form.Item>
      </div>
    );
  }
}
```

### 3、props.form 存储表单值

这是因为`props.form`的问题。
`props.form`简单来说就是一个`store`，存储着所有经过`props.form.getFieldDecorator`包装后的表单组件的值与校验规则。通过调用`props.form.validateFieldsAndScroll`就可以对值进行校验了。

而我们的代码中，实际上存在多个`props.form`，`App`组件有一个，`BasicInfoForm`组件也有一个，各自为政，互不干扰。

所有如果想校验`BasicInfoForm`组件的表单，就必须用该组件内的`form.validateFieldsAndScroll`。

![props.form](http://oyy3cbpm3.bkt.clouddn.com/15379701681393.jpg)

第一反应是使用`ref`，但由于`BasicInfoForm`是被`getFieldDecorator`装饰后的组件，`props`上并没有我们期望的`form`属性。这时应该使用官方提供的`wrappedComponentRef`替代。

```
this.basicInfoForm.props.form.validateFieldsAndScroll((err, values) => {
    if (err) {
        return;
    }
    // ...
});
```

又因为还有`workExpForm`和`projectExpForm`，所以就要再获取这两个表单的值，再组合起来。

### 4、自定义表单组件带来更多问题？

看到这，就会有疑问了，拆分后带来了一大堆的问题。难道不应该对组件进行拆分吗？
如果只将一些简单的组件作为自定义表单组件，比如`CitySelect`，其他的保持原样是不是更简单些？。
这也不失为一种方法，所以是否应该拆分，就是仁者见仁智者见智了。

但是就上面的问题而言，有一种解决办法，就是**只有一个`props.form`**，即只在`App`组件使用`Form.create`包装，其他组件都通过`props`传递`form`。所以代码会变成这样：

```js
  render() {
    const { getFieldDecorator } = this.props.form;
    return (
      <div className="resume">
        <ResumeForm form={this.props.form} />
        <Button type="primary" onClick={this.save}>保存</Button>
        <Button onClick={this.reset}>重置</Button>
      </div>
    );
  }
```

这样做，就仅仅是「将代码拆分」，而不是「封装自定义表单组件」了。但这种做法带来的好处也是明显的，上面提到的第二个问题也同时解决了。

## 三、多余的字段

再来详细谈谈第二个问题。

```js
{
  "basic": {
    "name": "ltaoo",
    "birthday": "2018-10-01T03:33:44.541Z",
    "sex": 0,
    "city": [
      "33",
      "3301",
      "330105"
    ],
    "email": "litaowork@aliyun.com"
  },
  "work": [
    "深圳联友科技",
    "杭州群核科技"
  ],
  "projects": [
    {
      "title": "BD System",
      "type": 1,
      "company": "杭州群核科技"
    }
  ]
} 
```

封装组件后，获取到的是这样的数据，而我们实际需要的是

```js
{
  "name": "ltaoo",
  "birthday": "2018-10-01T03:33:44.541Z",
  "sex": 0,
  "city": [
    "33",
    "3301",
    "330105"
  ],
  "email": "litaowork@aliyun.com"
  "work": [
    "深圳联友科技",
    "杭州群核科技"
  ],
  "projects": [
    {
      "title": "BD System",
      "type": 1,
      "company": "杭州群核科技"
    }
  ]
} 
```

最后提交前处理一下就好了嘛，就这样：

```js
  save = () => {
    console.log(this.basicInfoForm.props);
    this.basicInfoForm.props.form.validateFieldsAndScroll((err, values) => {
      if (err) {
        return;
      }
      const body = JSON.stringify(values, null, 2);
      console.log({
        ...body.basic,
        work: body.work,
        projects: body.projects
      });
    });
  };
```

虽然解决了这个问题，但我们需要**在所有用到`ResumeForm`组件的地方处理数据**，这很明显不够优雅。能否做到获取到的`values`就是我们期望的**最终数据**呢？

从我们的使用经验来说，获取到的数据是和`getFieldDecorator`强相关的，`key`是参数，`value`是表单的值。所以应该从`getFieldDecorator`入手。

### 1、表单值转换

还有一个类似的问题，当组件使用到「日期输入」时，往往需要将表单的值转换为时间戳，这也是重复工作，能否表单暴露的值就是时间戳呢？

如果上面的`city`数据，我们只需要最后一位，这个问题似乎是一样的。但这个问题可以使用`normalize`解决，该方法是用来「转换默认的 value」给控件。

就是可以将表单的值做处理，但要求处理后的值也是控件能够接受的。默认我们选择城市后得到`["33", "3301", "330105"]`，可以将其转换为`["330105]`，但不能变成`"330105"`，所以无法处理`moment`变成时间戳。

```js
        <Form.Item label="所在城市">
          {getFieldDecorator('city', {
            rules: [
              {
                required: true,
                message: '请选择所在城市'
              }
            ],
            normalize: function(value) {
              console.log(value);
              return value ? [value[value.length - 1]] : value;
            }
          })(<CitySelect />)}
        </Form.Item>
```

## 四、默认值

默认值也是表单组件一个非常重要的功能，无论是初始化默认值，减少用户填写成本；还是进入编辑状态时赋值，都要用到该功能。

### 1、默认值通过接口得到不会生效

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

而如果改成`initialValue`就能够生效。

```js
@Form.create()
export default class App extends React.Component {
  state = {
    defaultValue: undefined
  };
  componentDidMount() {
    setTimeout(() => {
      this.setState({
        defaultValue: 'ltaoo'
      });
    }, 3000);
  }
  save = () => {
    const { validateFieldsAndScroll } = this.props.form;
    validateFieldsAndScroll((err, values) => {
      if (err) {
        return;
      }
      const body = JSON.stringify(values, null, 2);
      console.log(body);
    });
  };
  render() {
    const { getFieldDecorator, getFieldValue } = this.props.form;
    const { defaultValue } = this.state;
    return (
      <div className="resume">
        {getFieldDecorator('name', {
          initialValue: defaultValue
        })(<Input />)}
        <Button type="primary" onClick={this.save}>
          保存
        </Button>
      </div>
    );
  }
}
```

### 2、自定义表单实现 initialValue 默认值

前面我们自己实现了`CitySelect`，它支持默认值吗？

```js
@Form.create()
export default class App extends React.Component {
  state = {
    defaultValue: undefined
  };
  componentDidMount() {
    setTimeout(() => {
      this.setState({
        defaultValue: ['33', '3301', '330105']
      });
    }, 3000);
  }
  save = () => {
    const { validateFieldsAndScroll } = this.props.form;
    validateFieldsAndScroll((err, values) => {
      if (err) {
        return;
      }
      const body = JSON.stringify(values, null, 2);
      console.log(body);
    });
  };
  render() {
    const { getFieldDecorator, getFieldValue } = this.props.form;
    const { defaultValue } = this.state;
    return (
      <div className="resume">
        {getFieldDecorator('city', {
          initialValue: defaultValue
        })(<CitySelect />)}
        <Button type="primary" onClick={this.save}>
          保存
        </Button>
      </div>
    );
  }
}
```

幸运的是支持。因为当`initialValue`发生改变时，会调用`CitySelect`的`componentWillReceiveProps`，并将`initialValue`作为`props.value`传入，实现了默认值的效果。

### 3、支持 defaultValue 默认值

那`CitySelect`支持`defaultValue`默认值吗？很明显不支持对吧，因为回头看我们的`CitySelect`组件代码，完全没有出现过`defaultValue`。

```js
@Form.create()
export default class App extends React.Component {
  state = {
    defaultValue: ['33', '3301', '330105']
  };
  save = () => {
    const { validateFieldsAndScroll } = this.props.form;
    validateFieldsAndScroll((err, values) => {
      if (err) {
        return;
      }
      const body = JSON.stringify(values, null, 2);
      console.log(body);
    });
  };
  render() {
    const { getFieldDecorator, getFieldValue } = this.props.form;
    const { defaultValue } = this.state;
    return (
      <div className="resume">
        <CitySelect defaultValue={defaultValue} />
        <Button type="primary" onClick={this.save}>
          保存
        </Button>
      </div>
    );
  }
}
```

即使是直接给初始值也不行，更别说通过接口获取默认值了。那么接下来在不影响原有功能的基础上，添加`defaultValue`的支持。

```js
// CitySelect.js
  constructor(props) {
    super(props);

    const { defaultValue, value = defaultValue } = props;
    this.state = {
      value
    };
  }
```


