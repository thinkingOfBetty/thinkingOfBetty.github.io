---
title: 字段值可能为undefined的处理
date: 2019-09-09
---

 &emsp;&emsp;前几天在写代码，后端传一个code值给前端，前端根据常量表决定展示的文案以及字体颜色，当code为-1时，不展示内容。因为后台有做不展示的处理，所以我误以为该字段是必定会返回的。后面出现bug才知道该字段是在某种特定类型的工单才会进行返回。于是我在页面渲染的时候用v-if="data.code && data.code !== -1"去进行判断，此时问题又来了。此前约定的code里面包含了0，而0也是要展示的，如果用这种方式进行判断，则code为0时这里的v-if的结果为false。所以之后开发应当明确两点，最大可能地避免bug。
 &emsp;&emsp;一、明确字段是否返回，尽可能地处理字段为undifined的情况
 &emsp;&emsp;二、处理字段为undifined时，分情况，如果该字段是一个object类型的，用loadash中的get进行处理（此处要注意loadash的按需引入方法）；如果该字段是某个对象的属性，可以用obj.hasOwnProperty进行判断，而不是用data.code && xxx 去进行判断，因为这里的code可能为0或者是空字符串。