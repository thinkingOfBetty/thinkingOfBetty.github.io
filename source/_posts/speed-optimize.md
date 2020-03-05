---
title: 页面加载过多DOM节点导致卡顿的优化实践
date: 2020-03-05 
---

&emsp;&emsp;安全运营中心新工单首页的工单详情——误判确认环节在开发时，确认的该节点展示的ioc条数最多为20条，但是实际上线后却发现展示的条数最多为1000条（这1000条数据展开来还有表格、图表等节点），所以当某条工单拥有1000条ioc节点时，不仅页面空白时间长，而且当数据加载完之后会直接导致网页崩溃。。

&emsp;&emsp;之前优化的方案：1000条数据不一次性加载，而是10条10条加载，这样子加载可以减少空白的时间，更快地渲染数据，但是这样优化的缺点就是在数据加载的过程中，滚动条会一直缩小，导致数据加载过程中滚动条几乎滚动不了，等到数据加载完成后，页面依旧会卡顿。

&emsp;&emsp;目前的优化方案主要有两方面，交互层面控制ioc展示的数量（最多只展示100条），前端代码层面的优化主要有如下两点：

一、一次性加载所有的数据，但是不提前渲染没打开的ioc内部Dom节点，只渲染需要提交给后台的节点。

二、用对象映射+render渲染的方式取代在template中用大量的v-else-if判断渲染哪个组件。


```javascript

map对象写法：
[ORDER_STAGE_TYPE.notReady]: {
    name: 'not-ready',
    ref: REF_NAME,
    props: {
        orderId: this.orderId,
        stageName: this.stageName
    },
    on: {
        'is-ready': function (stageName, data)  {
            vm.dataIsReady(stageName, data, index);
        }
    }
}


render函数写法：

renderTask (renderCallback) {
    let vm = this;
    return vm.tasks.map(function (item, index) {
        if (vm.shouldShowTask(item)) {
            const MAP_ITEM = vm.getHandledMapItem(item, index);

            return renderCallback(MAP_ITEM.name, {
                class: MAP_ITEM.class,
                props: MAP_ITEM.props,
                ref: MAP_ITEM.ref,
                refInFor: true,
                on: MAP_ITEM.on 
            });
        }
    });
}
```

其中rener中的refInFor:true是很重要的一个属性，如果没有配置该属性，则用$refs获取的是单个对象，如果配置了则是一个数组，对于遍历dom节点获取提交给后台数据的，获取的是数组这点可是至关重要。