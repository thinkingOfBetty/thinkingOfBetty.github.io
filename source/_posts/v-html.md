---
title: 用DOMPurify处理传入给v-html内容
date: 2019-09-04
tags: 
     - 安全
     - html
---

 &emsp;&emsp;昨天提交代码的时候，同事审核到代码里面用到v-html，要求需对后台传入内容进行转义，否则可能会有安全性问题，比如后台现在传给你的内容是依赖用户输入的，假设后台直接把那段文本传到前端，那么就容易发生xss攻击。假如用户传的是一段scrpit代码，就用可能直接导致网页瘫痪。当时我直接调用了转义函数，结果发现标签被当成文本输出了，所以我知道了所谓转义应该是指的过滤，把非法的输入给过滤。此时DOMPurify就派上用场了。
 &emsp;&emsp;DOMPurify使用：一、引入这个库 二、可以直接定义一个全局配置常量 三、调用DOMPurify.sanitize函数对要传入给v-html的内容进行处理。e.g v-html="DOMPurify.sanitize(itemData.value, DOMPURIFY_CONFIG)"

 全局配置参考如下：
 
DOMPURIFY_CONFIG = {
 &emsp;&emsp; USE_PROFILES: {html: true},  //允许所有安全的HTML元素，但不允许SVG或MathML
 &emsp;&emsp; ALLOWED_ATTR: ['style'],     //允许html元素上的style属性
 &emsp;&emsp; WHOLE_DOCUMENT: true         //返回整个文档，包括<html>标签，为false则只返回body内（不包括body）的内容，设置为true是t为了拿到
 &emsp;&emsp; style标签
};
