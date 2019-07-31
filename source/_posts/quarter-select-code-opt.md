---
title: 季度下拉框代码优化
---

结论：对比以上两种，最大的不同其实是思考逻辑。在思考这个内容的起初，我就给自己限定了思考的范围，就是我是从固定数量的季度数开始思考，然后就发现季度是要减去固定的数量才得出的，以致于写出冗长的代码。慢吃是一开始就认定这种是可以做成配置项的，所以思考的入口就很不一样，后面产出的东西就不一样。

在做这种通用性比较强的内容，一定要一开始就往可配置的方向去思考，否则可能就是做很多无用功，走很多的弯路。

优化之后的代码
``` bash
            getCurrentQuarter () {                                  //计算当前在哪个季度 
                const MONTH = new Date().getMonth() + 1,            //用getMonth获取的月份比实际月份少一个月
                        CURRENTQUARTER = QUARTERS_OBJ[MONTH];
                        
                return CURRENTQUARTER;
            
            },

             /**
             * 将某个年份的指定季度推进指定数组里，如果order是true，则是取的指定年份的靠前的年份，反之是取的是靠后的年份
             * @param {String} current //年份跟季度的组合 形如2019-02 
             * @param {Number} beforeNum  //需要显示的季度数，不包括当前季度
             * @return {Array} 
             */
            getBeforeCurrentQuarter (current, beforeNum) {
                let season = ['1', '2', '3', '4'],
                    len = season.length,
                    [year, quarter] = current.split('-'),
                    index = season.findIndex((item) => item === quarter),
                    arr = [],
                    loopIndex = index - 1;

                for (let i = 0; i < beforeNum; i++) {

                    if (loopIndex < 0) {
                        year--;
                        loopIndex = len - 1;
                    }

                    arr.push(this.handleYearAndQuarter(year, season[loopIndex]));
                    loopIndex--;
                }

                return arr;
            }
```