---
title: React手动重新渲染子组件
date: 2019-08-15 09:30:02
categories: 前端
tags:
    - React
    - JavaScript
---
昨天在开发APP项目时，需要实现一个上传图片和视频，然后上传后的图片和视频进行轮播的功能，轮播图使用的是第三方插件`react-native-snap-carousel`，在ios中使用时没有出现过兼容性问题，只是需要在轮播的数据更新时，需要手动调用一下`snapToItem`方法，展示其对应的下标。然而在android中发现该方法并不生效，并且组件的`firstItem`属性也不会生效，就导致了上传后数据更新的时候下标依然保留在上一次，但是需求方想要每次展示最新上传的图片或视频。后来发现该组件的`firstItem`属性会在组件第一次渲染的时候生效，所以想到了一个方法就是将该组件进行手动卸载和挂载使其重新渲染。自己捣鼓了半天没有好的方法，后来在老大的指点下搞定了，然后在这记录一下，如何手动重新渲染子组件。直接上代码：
```jsx
class Parent extends Component {
    constructor(props) {
        super(props);
        this.state={
            isShow: true
        }
    }
    componentDidUpdate(prevProps){
        // 判断是否需要更新
        if(this.props.data != prevProps.data){
            // 先将组件卸载
            this.setState({
                isShow: false
            })
        }
        // 利用setTimeout异步操作，再将组件重新渲染， 不过会有一下抖动
        setTimeout(() => {
            this.setState({
                isShow: true
            })
        }, 100)
    }
    render() {
        const {isShow} = this.state;
        return (
            <div>
                {
                    isShow? <Children/>:null
                }
            </div>
        )
    }
}
```