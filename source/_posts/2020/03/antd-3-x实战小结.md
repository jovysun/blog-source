---
title: antd 3.x实战小结
permalink: antd-3-1
tags:
  - React
categories:
  - 框架与库
date: 2020-03-28 10:46:36
updated: 2020-03-28 10:46:36
---

# 概述
antd(v3.26.3)实战中的一些注意点做个小结。
<!-- more -->

# 详述
## Input
Input的onChange绑定函数参数是e，想获取值用`e.target.value`。
## Select
Select的onChange绑定函数参数是value，想获取值直接用value，想获取其他值例如显示的文本等，可以通过第二个参数option获取。

一个典型的场景就是Select最终提交的值是选项的id，回显需要的是option的文本内容，这时就可以：
```js
  handleChangeProvince(provinceId, option) {
    ...

    this.props.form.setFieldsValue({
      provinceName: option.props.children
    });

    ...
  }
```
![断点option值](1.jpg)
## defaultValue与value的区别

defaultValue与value的区别是都可以实现初始值，但是加value属性后变成完全受控组件，defaultValue不是。

### Modal弹框表单提交

想通过Modal弹框的Confirm按钮触发提交需要通过ref来获取form实例：
```js
// Hook创建一个ref实例
const addressFormRef = useRef(null);
// 弹框确认处理函数
const handleOkAddressForm = () => {
  // validateFieldsAndScroll是Form.creat包装的form组件示例方法
  addressFormRef.current.validateFieldsAndScroll((err, values) => {
    if (!err) {
      // console.log('Received values of form: ', values)
    }
  });
};
// 设置ref属性
<Modal
  destroyOnClose={true}
  title={"Add Address"}
  okText="Confirm"
  visible={addressVisible}
  width={MODALWIDTH}
  confirmLoading={confirmLoading}
  onCancel={() => setAddressVisible(false)}
  onOk={handleOkAddressForm}
>
  <WrappedAddressForm ref={addressFormRef} />
</Modal>
```

## Form.create包装的组件校验
综合示例：
```js
<Form.Item label="Zip Code" required={zipCodeRequied}>
  {getFieldDecorator("zip", {
    initialValue: address.zip,
    rules: [
      {
        required: true,
        message: "Please enter zip code."
      },
      {
        validator: (rule, value, callback)=>{
          ...
          callback()
        }
      },
      {
        isUS: encryptedSelectedCountryId === ENCRYPTREGIONID.US,
        validator: method.zipCode()
      },
      {             
        validator: (rule, value, callback) => {
          // 异步校验
          let formData = this.props.form.getFieldsValue(['countryId','provinceId','zip'])
          shoppingCartApi.zipValid(formData).then(response => {
            if (response.status === "1") {
              callback();
            } else {
              callback(response.message || "Please enter a valid zip code.");
            }
          });

        }
      }
    ]
  })(
    <Input maxLength={20} onBlur={this.handleBlur.bind(this, "zip")} />
  )}
</Form.Item>
```
### 自定义校验方法
校验规则除了关键字如`required`，还可以通过`validator`自定义校验（注意，callback 必须被调用）。在实际生产中，通常会把校验方法放到一个地方集中维护，这时候就涉及把表单页面中的参数传递到校验方法中。官网没有明确给出示例，网上很多文章也没有，其实思路也很清楚，肯定在第一个参数`rule`上，但是看了官网介绍也没明白rule到底咋用。经过一番断点测试才理顺，原来这个rule就是rules里的一个对象`{}`，因此在这里定义的任何键值对都可以通过rule取到。
```js
  zipCode: msg => {
    return (rule, value, callback) => {
      if (
        value &&
        rule.isUS &&
        !/^(\d{5}(?:\-?\d{4})?)$/.test(value)
      ) {
        callback(
          msg ||
            'Must be a 5 or 9 digit number, e.g.00000、000000000、00000-0000.'
        );
      } else {
        callback();
      }
    };
  }
```
![断点rule](2.jpg)

### 手动触发某个表单项校验
这个也常见，实际场景就是表单项联动，例如国家、省（州）、邮编，重新选择不同的国家、省（州）需要重新校验邮编。
```js
handleChangeProvince(provinceId, option) {
  ...
  this.props.form.validateFields(['zip']);
  ...
}
```
### 异步校验
这个就是自定义校验的一种形式，见上面的综合示例。

更多高级用法可研究 [async-validator](https://github.com/yiminghe/async-validator)。

## Form表单的自定义校验
官网有直接示例，主要是通过`validateStatus`、`help`、`hasFeedback` 等属性，你可以不需要使用 Form.create 和 getFieldDecorator，自己定义校验的时机和内容。

- validateStatus: 校验状态，可选 'success', 'warning', 'error', 'validating'。
- hasFeedback：用于给输入框添加反馈图标。
- help：设置校验文案。
   
```js
<Form.Item
  label="Fail"
  validateStatus="error"
  help="Should be combination of numbers & alphabets"
>
  <Input placeholder="unavailable choice" id="error" />
</Form.Item>
```
# 参考
https://3x.ant.design/components/form-cn/#components-form-demo-global-state