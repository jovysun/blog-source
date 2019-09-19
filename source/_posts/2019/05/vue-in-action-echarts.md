---
title: Vue实战@组件中使用ECharts
permalink: vue-in-action-echarts
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-21 20:03:17
updated: 2019-05-21 20:03:17
---

# 概述

ECharts 是一个流行的成熟的数据图表库，想在 vue 组件中使用，可以使用社区提供的封装好的 vue-eacharts 库，不过还是推荐自己分装，因为简单可靠。[配套测试源码](https://github.com/jovysun/Vue-my-pro)

<!-- more -->

# 详述

## 跑起来

根据[官方文档](https://echarts.baidu.com/tutorial.html#%E5%9C%A8%20webpack%20%E4%B8%AD%E4%BD%BF%E7%94%A8%20ECharts)，安装完 echarts 之后，只需简单的创建个组件文件 Chart.vue，然后在 mounted 的时候初始化 echarts 实例，然后在需要的页面注册使用即可使用。

### 创建 Chart 组件

```html
<template>
  <div ref="myChart"></div>
</template>
```

```js
import echarts from "echarts";
export default {
  mounted() {
    // 基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(this.$refs.myChart);
    // 绘制图表
    myChart.setOption({
      title: {
        text: "ECharts 入门示例"
      },
      tooltip: {},
      xAxis: {
        data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
      },
      yAxis: {},
      series: [
        {
          name: "销量",
          type: "bar",
          data: [5, 20, 36, 10, 10, 20]
        }
      ]
    });
  }
};
```

### 使用 Chart 组件

```html
<template>
  <div>
    <Chart style="height:400px" />
  </div>
</template>
```

```js
import Chart from "../../components/Chart";
export default {
  components: {
    Chart
  }
};
```

## 实际使用

需要解决三个问题：一是窗口大小改变的时候图表需要重新渲染；二是引用第三方库需要及时的销毁，访问内存溢出；三是数据应该由父级传入。

```js
import debounce from "lodash/debounce";
import echarts from "echarts";
import { addListener, removeListener } from "resize-detector";
export default {
  props: {
    option: {
      type: Object,
      default: () => {}
    }
  },
  watch: {
    option(val) {
      this.chart.setOption(val);
    }
    // 深度复制，可以监听到option对象内部属性的变化，但是比较耗性能
    // option: {
    //   handler(val) {
    //     this.chart.setOption(val);
    //   },
    //   deep: true
    // }
  },
  created() {
    this.resize = debounce(this.resize, 300);
  },
  mounted() {
    this.renderChart();
    // 监听dom变化，及时重新渲染eacharts实例
    addListener(this.$refs.myChart, this.resize);
  },
  beforeDestroy() {
    // 销毁eacharts实例
    removeListener(this.$refs.myChart, this.resize);
    this.chart.dispose();
    this.chart = null;
  },
  methods: {
    resize() {
      console.log("resize");
      // 重新渲染eacharts实例
      this.chart.resize();
    },
    renderChart() {
      // 基于准备好的dom，初始化echarts实例
      this.chart = echarts.init(this.$refs.myChart);
      // 绘制图表
      this.chart.setOption(this.option);
    }
  }
};
```

调用 Chart 组件

```html
<template>
  <div>
    <Chart :option="chartOption" style="height: 400px;" />
  </div>
</template>
```

```js
import random from "lodash/random";
import Chart from "../../components/Chart";
import { setInterval, clearInterval } from "timers";
export default {
  data() {
    return {
      chartOption: {
        title: {
          text: "ECharts 入门示例"
        },
        tooltip: {},
        xAxis: {
          data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
        },
        yAxis: {},
        series: [
          {
            name: "销量",
            type: "bar",
            data: [5, 20, 36, 10, 10, 20]
          }
        ]
      }
    };
  },
  mounted() {
    this.interval = setInterval(() => {
      this.chartOption.series[0].data = this.chartOption.series[0].data.map(
        () => random(100)
      );
      // 引用类型，重新创建对象才能保证Chart组件中监听option生效
      this.chartOption = { ...this.chartOption };
    }, 3000);
  },
  beforeDestroy() {
    clearInterval(this.interval);
  },
  components: {
    Chart
  }
};
```

# 参考

极客时间——Vue 开发实战
