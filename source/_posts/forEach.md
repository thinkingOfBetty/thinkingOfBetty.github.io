---
title: forEach不能采用break或者returen false终止循环
date: 2019-09-21
---

 &emsp;&emsp;前几天写PPT项目代码的时候，想要找到符合条件的一项，就跳出循环，不要继续循环，结果我用了break发现没用，查了一下原来forEach要终止循环需要通过try catch。。。于是发现some是提前终止循环的。但是some也有个坑要提防，那就是some去遍历一个空数组，会返回false；