---
title: 利用visibilitychange对项目进行优化
date: 2020-07-07
tags: 
     - 项目优化
---

&emsp;&emsp;我目前做的项目中由于后台架构设计的不合理（后台架构本来应该设计成类似node的事件通知类型的，但是由于项目一开始选型的问题，导致后面很多操作都得依赖前端轮询来实现。） 同时我们的项目有一些跳转操作是直接打开一个新的标签页（方便使用人员比对、查看数据），所以会导致一个浏览器里面打开了好些相同的标签页，然后标签页又有许多没必要进行的轮询。当遇到visibilitychange事件时，我知道，就是它了。

&emsp;&emsp;优化价值：减少CPU占用，减少不必要请求对带宽的占用，同时能比较大地减少服务器的压力。

&emsp;&emsp;这次我总共尝试了三种不同的方法，下面对这三种进行描述，最后再进行对比总结。

&emsp;&emsp; 第一种是使用自定义生命周期的方法，在main中创建vue之前，往Vue.config.optionMergeStrategie中加入pageVisible跟pageHidden这两个生命周期，然后在创建Vue之后，在window上绑定一个visibilitychange的监听事件，每次事件被触发之后就利用递归去执行每个组件中主次的这两个生命周期。在组件中的具体使用方法是：在pageVisible中初始化轮询方法，在pageHidden中对定时器进行销毁。

&emsp;&emsp; 在每个使用轮询的组件中加入这两个生命周期其实是挺繁琐的事情，所以我试图寻找一种全局控制的方法，所以就有了第二种尝试。这一次我试图去对window中存到的timer的信息进行删除跟初始化，进行了一波骚操作之后我发现这种方法是不可行的——这种对window下的timer进行删除跟重新创建定时器的方法，可能会导致组件中原来保存的timer其实是过期的，这时候组件在beforeDestroy中清除的定时器是已经被清除的了，而真正的定时器还是存在的。

&emsp;&emsp; 第三种方法，也是我最后使用的方法。分析了一波项目，发现里面用的定时器都是有经过封装的intervalTask的实例，我只需要遍历组件，然后找出每个组件中intervalTask实例化的属性，对属性轮询的状态进行判断之后再调用其中的intervalTask的start、stop的方法，基本就能解决大部分情景的问题了。

&emsp;&emsp; 由于第二种方法是失败的，所以主要对比第一种跟第三种方法，第一种方法的话比较繁琐，所以编码的人可能经常忘记这种处理，但是这种方法对性能的影响会比较小。而第三种方法虽然集中统一地处理了，但是在递归对组件的属性进行遍历的时候，应该还是比较耗费性能的。

最后附上第三种方法具体实现的代码

```bash

import IntervalTask from 'src/util/interval_task';

const PAGE_VISIBLE = 'pageVisible',
      PAGE_HIDDEN = 'pageHidden';

let goThroughAllAttr = (vm, callback) => {
    for (let key in vm) {
        if (vm.hasOwnProperty(key)) {

            if (Object.prototype.toString.call(vm[key]) === '[object Object]') {
                let attr = Object.getPrototypeOf(vm[key]);
                if (attr && attr.constructor === IntervalTask) {
                    callback && callback(vm[key], vm[key].state);
                }
            }
        }
    }
}; 

// 通知所有组件页面状态发生了变化
let notifyVisibilityChange = (pageState, vm) => {

    if (pageState === PAGE_VISIBLE) {
        goThroughAllAttr(vm, (intervalTask, state) => {
            if (state === IntervalTask.STATE.STOPPED_BY_ROOT) {
                intervalTask.startByRoot();
            }
        }); 
    }
    if (pageState === PAGE_HIDDEN) {
        goThroughAllAttr(vm, (intervalTask, state) => {
            if (state === IntervalTask.RUNNING || state === IntervalTask.STATE.WAITING) {
                intervalTask.stopByRoot();
            }
        }); 
    }

    // 遍历所有的子组件，然后依次递归执行
    if (vm.$children && vm.$children.length) {
        vm.$children.forEach(child => {
            notifyVisibilityChange(pageState, child);
        });
    }
};

/**
 * 将事件变化绑定到根节点上面
 * @param {*} rootVm
 */
export function bindVisibilityHook (rootVm) {
    window.addEventListener('visibilitychange', () => {
        if (document.visibilityState === 'hidden') {

            // 通知所有组件生命周期发生变化了
            notifyVisibilityChange(PAGE_HIDDEN, rootVm);
        } else if (document.visibilityState === 'visible') {
            notifyVisibilityChange(PAGE_VISIBLE, rootVm);
        }
        
    });
}


```