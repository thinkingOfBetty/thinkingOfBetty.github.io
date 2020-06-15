---
title: 封装load-btn
date: 2019-11-28
tags: 
     - 组件封装
---

 &emsp;&emsp;之前由于公司组件库使用的load-btn是一直有load这个icon的，只是当loading的值为true时这个图标才会转动。但是产品线是要求该icon只在load的时候才出现。于是每次使用这个按钮，都需要使用v-if去写两次。所以需要自己重新对该load按钮进行封装。

 &emsp;&emsp;接下来阐述下封装这个按钮的几个过程。

version 1.0 （接收参数：text、loading）
```html
    <load-sfv-btn v-if="loading">{{text}}</load-sfv-btn>
    <sfv-primary-btn v-else>{{text}}</sfv-primary-btn>
```
这种看着感觉没啥问题，但是等到我用另外种按钮的时候就需要额外拓展。

version 1.2 （接收参数：text、loading、type（按钮的类型有primary、differ、primary、 danger、 blank、secondary、 circle））
```html
<span v-if="isPrimaryBtn">
    <sfv-button-primary @click="handleClick" 
                        v-if="!showLoading">{{text}}</sfv-button-primary>
    <sfv-button-load v-else 
                        loading 
                        class="sfv-btn-primary">{{text}}</sfv-button-load>
</span>
<span v-if="isNormalBtn">
    <sfv-button @click="handleClick" 
                v-if="!showLoading">{{text}}</sfv-button>
    <sfv-button-load v-else 
                        loading>{{text}}</sfv-button-load>
</span>
```

此时代码有点相似而冗余了，此时代码审核人mc建议这种根据不同状态展示相似组件的可以考虑用render去实现

version 1.3 
```javascript
render (createElement, context)  {
    let {text, loading, type} = context.props, 
        domName,
        btnClass;

        if (type === 'primary') {
            domName = 'sfv-button-primary';
            btnClass = 'sfv-btn-primary';
        } else {
            domName = 'sfv-button';
        }
    

    if (loading) {
        domName = 'sfv-button-load';

        return createElement(
            domName, text, {
                props:{
                    loading:true
                },
                class: btnClass
            }
        );
        
    } 

    return createElement(
        domName, {
            on:{
                click:context.data.on['on-click']
            },
            domProps: {
                innerText: text
            }
        }
    );
}
```
此时是可以看到感觉两个return应该是可以写成一个的，但是不知道为啥那个load-btn中text用domProps的形式传进去总是会有问题。此时mc觉得这种写法既不可以传slot、又不可以传其他类型的btn，感觉整个组件的灵活性比较差。此时重新看了组件源码，发现组件里面其实是通过不同的类名去区分组件的。因此有了版本1.4。

version1.4
```html
<sfv-button @click="handleClick" 
            v-if="!showLoading"
            :class="typeCls">
            {{text}}
            <slot name="content"></slot>
</sfv-button>
<sfv-button-load v-else 
                loading
                :class="typeCls">
                {{text}}
                <slot name="loadContent"></slot>
</sfv-button-load>
```
此时感觉有点兜兜转转倒回去的感觉，只是加多了多种按钮类型的拓展、同时添加了slot。但是乍一看才知道这种写法传slot的话还得传两个，既不够简洁、反而使用起来还麻烦了。
version1.5
```html
<sfv-button @click="handleClick"
                :class="typeCls">
                <i slot="icon" :class="iconCls"></i>
                <span v-if="text">{{text}}</span>
                <slot v-else
                      name="content"></slot>
</sfv-button>
```
这次是通过动态class（公司组件中icon的展示用的是类名），感觉比上次简洁了点，但是此时还需考虑不直接使用封装好的load按钮，而是直接使用的图标控制，会不会造成原来组件功能不能使用。此时发现了一个bug，就是调用这个封装的组件时，配置disable属性是不生效的，此时有了version1.6
version1.6
```html
<sfv-button @click="handleClick"
            :class="typeCls"
            :default-disable="disabled">
            <i slot="icon" :class="iconCls"></i>
            <slot name="content">{{text}}</slot>
</sfv-button>
```
此时问题又来了，我发现这个btn组件除了disable，应该还有其他属性的，所以有了目前的终极版本，希望以后还能继续改进。
```html
<sfv-button @click="handleClick"
            :class="typeCls"
            v-bind="$attrs"
            v-on="$listeners">
            <i slot="icon" :class="iconCls"></i>
            <slot name="content">{{text}}</slot>
</sfv-button>
```
一天花了不少时间在优化这个组件，从写重复的html，到写render函数，到写回html（加动态类型、动态icon、slot、$attr），到考虑组件库升级之后可能会发生的问题（用类型去添加icon，如果新版本把类名改了就容易出问题），感觉自己造轮子的能力还是有很大的提升空间，应该多去学习一些优秀源码的写法、设计思想。







             