---
title: IE下get请求报错排查
date: 2020-06-04
---

 &emsp;&emsp;报错原因：在ie下pathInfo的编码方式为UTF-8，而queryString的编码方式是GB2312编码（谷歌、火狐都是UTF-8），所以后台解析不了queryString中的内容，也就报错了。
 &emsp;&emsp;解决方法：使用encodeURI对url进行包裹，或者是将get请求改成post。
