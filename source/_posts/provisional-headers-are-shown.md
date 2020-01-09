---
title: provisional headers are shown 
date: 2020-01-09 
---

采坑记录：本地开发调用接口时显示请求头显示provisional headers are shown ；

原因是浏览器证书报错：NET::ERR_CERT_AUTHORITY_INVALID，导致请求被拦截；

解决办法：在chrome://flags/#allow-insecure-localhost中enabled #