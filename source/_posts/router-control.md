---
title: 路由控制权限控制
date: 2020-01-07 
---

&emsp;&emsp;工单的权限控制中前端的控制分为两部分，一部分是导航栏中的入口，另一部分是隐藏注册的路由（防止用户记住路由后直接输入路径去获取资源）。工单项目中路由的隐藏主要是通过路由表的配置  + 递归处理路由表 + ROUTER.addRoutes（处理完的路由数组）来实现的。
&emsp;&emsp;一、目前项目权限如下所示

| 专家类型       | 菜单权限                                               | 读写权限             | 备注                 | 
| ------------ | ------------------------------------------------------- | ------------------- | ---------------------- | 
| T1,T2,T3     | 除了xx管理，xx设置，其它全有                           | 有所有权限           | 可查看所有客户的信息，只能查看分配给自己的工单 | 
| T5           | 所有权限                                                 | 所有权限             | -                                         |
| 渠道          | 只有工单管理（不包括xx），客户管理（不包括xx）       | 工单只有查看权限      | 只能查看自己所属渠道的客户和所属渠道客户的工单| 

由于这个项目的隐藏需求都是对某个用户开放大的模块，然后再关闭里面小模块的权限，或者是关闭整个大模块的权限，所以我目前采取的配置是，在一级模块可以在meta中配置允许访问的roles，然后在子级模块中配置觉得一级模块中放通权限，但是在二级或者是三级模块中关闭某个用户的权限的即可以配置forbidenRoles。（这种配置方式比较适用于一级二级、三级路由有写成那种父子级的路由配置表）

&emsp;&emsp;二、首先用使用同步请求获取用户权限信息，然后用filter筛选出当前用户允许访问的一级路由，然后递归删除（此处的删除依旧是用的filter）子路由中不让当前用户访问的子路由。

&emsp;&emsp;三、递归删除的方法

```javascript
function deleteForbiddenRoutes (routerObj, role) {  //删除不允许被当前角色访问的路由
    if (get(routerObj, 'meta.forbiddenRoles', []).includes(role)) {
        return false;
    }

    if (Array.isArray(routerObj.children)) {
        const ROUTES_ARR = routerObj.children;
        let  filteredArr = ROUTES_ARR.filter(item => {
                return !get(item, 'meta.forbiddenRoles', []).includes(role);
            });

        filteredArr.forEach((item, index) => {
            if (Array.isArray(item.children)) {
                filteredArr[index].children = delChildForbiddenRoutes(item, role);
            }
        });
        routerObj.children = filteredArr;
    }

    return routerObj;
}

function delChildForbiddenRoutes (item, role) { //递归删除不允许被当前角色访问的路由
    let arr = [];

    arr = item.children.filter((childItem) => {
        return deleteForbiddenRoutes(childItem, role);
    });

    return arr;
}

```