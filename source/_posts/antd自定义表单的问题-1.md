---
title: antd 自定义表单的问题 - 1
categories: React
tags:
- react
- antd
- form
date: 2018/10/22
---

在使用 antd 的表单过程中，发现存在许多的问题。该博客会对我曾经遇到过的问题做一个总结，由于内容太多，所以预计会分成 3 篇。

这是该主题的第一篇，主要介绍「什么是自定义表单」。全文会以「简历表单」作为示例来进行说明。

<!--more-->

## 一、示例说明

现在需要一个「简历」表单，支持填写个人信息、工作经历和项目经历。

![简历表单截图](http://oyy3cbpm3.bkt.clouddn.com/xl8v6olw0o.codesandbox.io_.png)

### 1、字段

基本信息包含 7 个字段，需要提交的数据格式如下：

```json
{
  "name": "ltaoo",
  "birthday": 1538323200000,
  "sex": 0,
  "city": [
    "33",
    "3301",
    "330105"
  ],
  "email": "litaowork@aliyun.com",
  "works": [
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

### 2、校验规则

对于字段会有一些校验要求

- 姓名必填
- 性别必填
- 城市必填
- 邮箱必填，并且符合邮箱格式
- 工作经历选填
- 项目经历选填，但如果增加了项目经历，项目名、项目类型必填

## 二、最简单的实现代码

完成这么一个表单是非常简单的,300 行左右代码即可完成，

```js
import React from 'react';
import {
  Icon,
  Form,
  Card,
  Button,
  Cascader,
  Select,
  Input,
  DatePicker
} from 'antd';

import provinces from 'china-division/dist/provinces.json';
import cities from 'china-division/dist/cities.json';
import areas from 'china-division/dist/areas.json';

const { Option } = Select;

const formItemLayout = {
  labelCol: {
    xs: { span: 24 },
    sm: { span: 4 }
  },
  wrapperCol: {
    xs: { span: 24 },
    sm: { span: 20 }
  }
};
const formItemLayoutWithOutLabel = {
  wrapperCol: {
    xs: { span: 24, offset: 0 },
    sm: { span: 20, offset: 4 }
  }
};

// 格式化城市数据
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

// 工作经历与项目经历
let workUid = 0;
let projectUid = 0;

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
  reset = () => {
    const { resetFields } = this.props.form;
    resetFields();
  };
  removeWorkExp = k => {
    const { form } = this.props;
    const keys = form.getFieldValue('workKeys');
    form.setFieldsValue({
      workKeys: keys.filter(key => key !== k)
    });
  };
  addWorkExp = () => {
    const { form } = this.props;
    const keys = form.getFieldValue('workKeys');
    const nextKeys = keys.concat(workUid);
    workUid++;
    form.setFieldsValue({
      workKeys: nextKeys
    });
  };
  removeProjectExp = k => {
    const { form } = this.props;
    const keys = form.getFieldValue('projectKeys');
    form.setFieldsValue({
      projectKeys: keys.filter(key => key !== k)
    });
  };
  addProjectExp = () => {
    const { form } = this.props;
    const keys = form.getFieldValue('projectKeys');
    const nextKeys = keys.concat(projectUid);
    projectUid++;
    form.setFieldsValue({
      projectKeys: nextKeys
    });
  };
  render() {
    const { getFieldDecorator, getFieldValue } = this.props.form;
    getFieldDecorator('workKeys', { initialValue: [] });
    getFieldDecorator('projectKeys', { initialValue: [] });

    const workKeys = getFieldValue('workKeys');
    const projectKeys = getFieldValue('projectKeys');
    const workItems = workKeys.map((k, index) => {
      return (
        <Form.Item
          {...(index === 0 ? formItemLayout : formItemLayoutWithOutLabel)}
          label={index === 0 ? '公司名' : ''}
          required={false}
          key={k}
        >
          {getFieldDecorator(`work[${k}]`, {
            validateTrigger: ['onChange', 'onBlur'],
            rules: [
              {
                required: true,
                whitespace: true,
                message: '请输入公司名'
              }
            ]
          })(
            <Input
              placeholder="公司名"
              style={{ width: '60%', marginRight: 8 }}
            />
          )}
          {workKeys.length > 1 ? (
            <Icon
              className="dynamic-delete-button"
              type="minus-circle-o"
              disabled={workKeys.length === 1}
              onClick={() => this.removeWorkExp(k)}
            />
          ) : null}
        </Form.Item>
      );
    });
    const projectItems = projectKeys.map((k, index) => {
      return (
        <Card
          key={k}
          extra={<a onClick={() => this.removeProjectExp(k)}>删除</a>}
        >
          <Form.Item {...formItemLayout} label="项目名称">
            {getFieldDecorator(`projects[${k}].title`, {
              validateTrigger: ['onChange', 'onBlur'],
              rules: [
                {
                  required: true,
                  whitespace: true,
                  message: '请输入项目名称'
                }
              ]
            })(
              <Input
                placeholder="项目名称"
                style={{ width: '60%', marginRight: 8 }}
              />
            )}
          </Form.Item>
          <Form.Item {...formItemLayout} label="项目类型">
            {getFieldDecorator(`projects[${k}].type`, {
              validateTrigger: ['onChange', 'onBlur'],
              rules: [
                {
                  required: true,
                  message: '请输入项目类型'
                }
              ]
            })(
              <Select
                style={{ width: '100%' }}
                placeholder="请选择项目类型（公司项目需要填写公司）"
              >
                <Option value={0}>个人项目</Option>
                <Option value={1}>公司项目</Option>
              </Select>
            )}
            {getFieldValue(`projects[${k}].type`) === 1 &&
              getFieldDecorator(`projects[${k}].company`, {
                validateTrigger: ['onChange', 'onBlur'],
                rules: [
                  {
                    required: true,
                    message: '请输入公司名称'
                  }
                ]
              })(<Input placeholder="请输入公司名称" />)}
          </Form.Item>
        </Card>
      );
    });
    return (
      <div className="resume">
        <Card
          title="基本信息"
          style={{ marginBottom: 10 }}
          className="resume__basic"
        >
          <div className="basic__content">
            <Form.Item label="姓名">
              {getFieldDecorator('name', {
                rules: [
                  {
                    required: true,
                    message: '请输入姓名'
                  },
                  {
                    max: 10,
                    message: '长度不能超过 10 个字符'
                  }
                ]
              })(<Input placeholder="请输入姓名" />)}
            </Form.Item>
            <Form.Item label="出生年月">
              {getFieldDecorator('birthday', {
                rules: [
                  {
                    required: true,
                    message: '请选择出生年月'
                  }
                ]
              })(<DatePicker placeholder="请选择出生年月" />)}
            </Form.Item>
            <Form.Item label="性别">
              {getFieldDecorator('sex', {
                rules: [
                  {
                    required: true,
                    message: '请选择性别'
                  }
                ]
              })(
                <Select style={{ width: '100%' }} placeholder="请选择性别">
                  <Option value={0}>男</Option>
                  <Option value={1}>女</Option>
                </Select>
              )}
            </Form.Item>
            <Form.Item label="所在城市">
              {getFieldDecorator('city', {
                rules: [
                  {
                    required: true,
                    message: '请选择所在城市'
                  }
                ]
              })(
                <Cascader
                  options={options}
                  showSearch
                  placeholder="请选择地址（支持搜索）"
                  style={{ width: 400 }}
                />
              )}
            </Form.Item>
            <Form.Item label="邮箱">
              {getFieldDecorator('email', {
                rules: [
                  {
                    required: true,
                    message: '请输入邮箱'
                  },
                  {
                    type: 'email',
                    message: '邮箱格式不正确'
                  }
                ]
              })(<Input placeholder="请输入邮箱" />)}
            </Form.Item>
          </div>
        </Card>
        <Card
          title="工作经历"
          style={{ marginBottom: 10 }}
          className="resume__work"
        >
          {workItems}
          <Form.Item {...formItemLayoutWithOutLabel}>
            <Button
              type="dashed"
              onClick={this.addWorkExp}
              style={{ width: '60%' }}
            >
              <Icon type="plus" /> 添加
            </Button>
          </Form.Item>
        </Card>
        <Card
          title="项目经历"
          style={{ marginBottom: 10 }}
          className="resume__education"
        >
          {projectItems}
          <Form.Item {...formItemLayoutWithOutLabel}>
            <Button
              type="dashed"
              onClick={this.addProjectExp}
              style={{ width: '60%' }}
            >
              <Icon type="plus" /> 添加
            </Button>
          </Form.Item>
        </Card>
        <Button type="primary" onClick={this.save}>
          保存
        </Button>
        <Button onClick={this.reset}>重置</Button>
      </div>
    );
  }
}
```

也可访问线上示例「[示例基础实现代码](https://codesandbox.io/s/xl8v6olw0o)」。虽然代码行数不多，但仍有许多优化空间，这也是本文的主题，拆分为多个自定义表单。

## 三、预期的简历表单组件

既然是封装为表单组件，那么在使用上应该和现有的表单组件相同，支持直接使用，也支持配合`Form`组件使用。

```js
@Form.create()
export default class HomePage extends React.Component {
    save = () => {
        const { validateFieldsAndScroll } = this.props.form;
        validateFieldsAndScroll((err, values) => {
          if (err) {
            return;
          }
          // 拿到需要的数据
          alert(JSON.stringify(values, null, 2));
        });
      };
    render() {
        return (
          <div className="resume">
            {getFieldDecorator('data')(
                <ResumeForm />
            )}
            <Button type="primary" onClick={this.save}>保存</Button>
          </div>
        );
    }
}
```

而`ResumeForm`也不是简单地将原先代码拷贝过来，而是同样做一些拆分，代码如下：

```js
export default class ResumeForm extends React.Component {
    render() {
        return (
          <div className="resume">
            <Card title="基本信息" className="resume__basic">
                {getFieldDecorator('basic')(<BasicInfoForm />)}
            </Card>
            <Card title="工作经历" className="resume__work">
                {getFieldDecorator('works')(<WorkExpForm />)}
            </Card>
            <Card title="项目经历" className="resume__education">
                {getFieldDecorator('projects')(<ProjectExpForm />)}
            </Card>
          </div>
        );
    }
}
```

### 1、拆分组件为细粒度的表单有什么好处

- 1、`ResumeForm`组件可复用，只需引入即可，而不是复制粘贴代码。
- 2、代码整洁，原先的 300 余行代码现在变成了 20 行。
- 3、**语义、职责更加清晰**，首页由一个「表单」和一个「按钮」组成，表单负责提供数据，页面组件不再像之前一样掺杂了许多无关的代码。

第二、三点从`ResumeForm`代码可以很直观看到，从原先的一个大整体，变成了三个子组件，每个子组件只负责一部分数据。

而再深入到`BasicInfoForm`组件中

```js
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
                message: '请输入姓名'
              }
            ]
          })(<Input placeholder="请输入姓名" />)}
        </Form.Item>
        <Form.Item label="出生年月">
          {getFieldDecorator('birthday', {
            rules: [
              {
                required: true,
                message: '请选择出生年月'
              }
            ]
          })(<DatePicker placeholder="请选择出生年月" />)}
        </Form.Item>
        <Form.Item label="性别">
          {getFieldDecorator('sex', {
            rules: [
              {
                required: true,
                message: '请选择性别'
              }
            ]
          })(<SexSelect style={{ width: '100%' }} placeholder="请选择性别" />)}
        </Form.Item>
        <Form.Item label="所在城市">
          {getFieldDecorator('city', {
            rules: [
              {
                required: true,
                message: '请选择所在城市'
              }
            ]
          })(<CitySelect />)}
        </Form.Item>
        <Form.Item label="邮箱">
          {getFieldDecorator('email', {
            rules: [
              {
                required: true,
                message: '请输入邮箱'
              },
              {
                type: 'email',
                message: '邮箱格式不正确'
              }
            ]
          })(<EmailInput placeholder="请输入邮箱" />)}
        </Form.Item>
      </div>
    );
  }
}```

除了使用`antd`提供的基础表单组件外，还自己声明了`SexSelect`、`CitySelect`和`EmailInput`组件，这类组件只是简单对原有基础表单组件做了一层封装，但还是可以配合`getFieldDecorator`使用。

### 2、自定义表单组件的说明

上面提到的`ResumeForm`、`BasicInfoForm`以及这里的`SexSelect`，都是对原有组件进行「组合」或「封装」得到的，能够直接使用，也可以配合`Form`组件使用的组件，就是我想要描述的「自定义表单组件」。

## 四、下期预告

接下来会先对「基本信息表单」做详细的讲解，主要会涉及到

- 怎么衡量是否要作为自定义表单组件
- 自定义表单的写法
- 取值的问题
- 校验的问题

<hr >


{% post_link antd自定义表单的问题-2 %}


