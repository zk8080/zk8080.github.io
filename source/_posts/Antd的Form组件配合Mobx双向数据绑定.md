---
title: Antd的Form组件配合Mobx双向数据绑定
date: 2019-08-09 10:52:47
categories: 前端
tags:
    - React
    - Antd
    - Mobx
    - Form
---
最近忙于赶项目，一直没有时间去更新博客，今天得空，正好把最近在项目中使用Mobx和Antd中的Form组件进行双向数据绑定的方法记录一下。
# 为什么需要双向数据绑定？
在后台管理系统中，表单是及其重要的，因为我们的增、改、查三个操作都需要用到表单。增加和查询一般是手动填入的，而修改则是需要将已有的数据先反显到表单中，再进行修改。这个反显的过程就需要对表单进行双向数据绑定了。
# 怎么进行双向数据绑定？
在Antd的Form组件中，使用Form.create()包装组件后，该组件的props属性就会多一个form对象，该对象提供了一些api，具体的可以查看Antd的官方文档进行查看，其中`getFieldDecorator`是该组件提供的双向数据绑定的api。直接上代码来看用法：
```
    <Form>
        <Form.Item>
            {getFieldDecorator('username', {
                rules: [{ required: true, message: 'Please input your username!' }],
            })(
                <Input
                    placeholder="Username"
                />
            )}
        </Form.Item>
    </Form>
```
用法很简单，将控件对应的key值和配置项传入到`getFieldDecorator`函数中，然后再使用`getFieldDecorator`包装控件。这个配置项具体有哪些参数可以去查看官方文档，目前我在项目中用的比较多的就是校验功能，也就是rules配置，使用该参数可以配置校验规则和提示。  
但是使用`getFieldDecorator`函数也有antd定制的规则：
+ 经过`getFieldDecorator`包装的控件，表单控件会自动添加`value`和`onChange`，数据同步将被Form接管。
- 不再需要也不应该用`onChange`来做同步，但还是可以继续监听`onChange`等事件。
* 不能用控件的`value`、`defaultValue`等属性来设置表单域的值，默认值可以用`getFieldDecorator`里的`initialValue`。
+ 不应该用`setState`，可以使用`this.props.form.setFieldsValue`来动态改变表单值。
有了这些规则后，说明我们不能去使用value和onChange配合使用进行双向数据绑定了，而是要使用`this.props.form.setFieldsValue`来设置值，但是这样通过onChange又是及其的繁琐。

# 使用Mobx数据双向数据绑定
在我的项目中，用的状态管理库为Mobx，如何将Mobx中观察的数据传入到Form组件中，我们走了很多弯路，开始我们发现`getFieldDecorator`函数的配置项中提供了一个`initialValue`的属性，使用该属性可以将Mobx的数据反显到表单控件中，但是该属性仅是表单控件的初始值，类似于`defaultValue`的功能，所以只要我们在页面中手动触发了控件的`value`改变，`initialValue`就不会再生效了，也就是说反显是ok的，但是如果要操作就会出问题。后面仔细阅读文档后，发现Form.create()函数中提供了`mapPropsToFields`和`onFieldsChange`两个api，这两个api配合使用可以将Mobx观察的数据传入到Form组件中，而且组件修改时也能实时传入到Mobx中。具体用法如下：
```
//  将mobx中观察的数据 转换为mapPropsToFields所需要的结构
const objToForm = (obj = {}) => {
    const target = {};
    for(const [key,value] of Object.entries(obj)){
        if(typeof value == 'object' && !Array.isArray(value)){
            target[key] = Form.createFormField(value);
        }else{
            target[key] = Form.createFormField({value});
        }
    }
    return target;
}

const onFieldsChange = (props, changedFields) => {
    props.setQueryData({...props.queryData, ...changedFields});
};

const mapPropsToFields = (props) => {
    return objToForm(props.queryData);
};

@Form.create({
    mapPropsToFields,
    onFieldsChange
})
class Index extends Component {
    constructor(props) {
        super(props);
        this.state={};
    }

    render() {
        const {getFieldDecorator} = this.props.form;
        return (
            <Form>
                <Form.Item>
                    {getFieldDecorator('username', {
                        rules: [{ required: true, message: 'Please input your username!' }],
                    })(
                        <Input
                            placeholder="Username"
                        />
                    )}
                </Form.Item>
            </Form>
        );
    }
}
```
主要是要将`mapPropsToFields`方法中接受的Mobx数据转换为Form组件可接受的数据结构，并且使用`Form.createFormField`包装数据，而`onFieldsChange`方法则是将当前改变的控件数据再提供给Mobx去使用，这样就形成了一个双向数据绑定的效果。
但是需要注意的是，`onFieldsChange`函数提供的数据并不是简单的key: value这种直接可以使用的键值对了，如果需要拿到表单数据进行接口交互的话，需要再将Mobx中的数据进行转换一下，或者直接使用`this.props.form.validateFields`方法拿到校验通过的所有表单数据来进行交互。