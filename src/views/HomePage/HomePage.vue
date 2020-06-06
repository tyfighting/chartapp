<template>
  <div class="homepage">
    <p class="select">数据源配置</p>
    <div class="top">
      <div class="left">
        <p class="sub-title">今日实时数据</p>
      </div>
      <div class="center">
        <p class="sub-title">引擎处理进度比例</p>
        <div ref="bar"></div>
      </div>
      <div class="right">
        <p class="sub-title">消息通知</p>
      </div>
    </div>
    <div class="bottom">
      <div class="left">
        <p class="sub-title">七日数据量</p>
        <div ref="line"></div>
      </div>
      <div class="right">
        <p class="sub-title">七日数据处理情况</p>
        <div ref="radar"></div>
      </div>
    </div>
  </div>
</template>

<script>
import echarts from "echarts";
export default {
  props: {},
  data() {
    return {
      bar: "", //柱状图
      barOption: "",
      line: "", //折线图
      lineOption: "",
      radar: "", //雷达图
      radarOption: ""
    };
  },
  mounted() {
    this.bar = echarts.init(this.$refs.bar);
    this.line = echarts.init(this.$refs.line);
    this.radar = echarts.init(this.$refs.radar);
    this.barOption = {
      grid: {
        left: "0%",
        right: "0%",
        bottom: "0%",
        containLabel: true
      },
      xAxis: {
        type: "category",
        data: [
          "分词",
          "命名实体识别",
          "文法句法",
          "关键词抽取",
          "摘要抽取",
          "关系抽取",
          "事件抽取"
        ],
        axisLine: {
          lineStyle: {
            color: "#b8d6f3"
          }
        },
        axisTick: { show: false },
        axisLabel: {
          interval: 0,
          rotate: 40
        }
      },
      yAxis: {
        type: "value",
        splitLine: {
          show: false
        },
        axisLine: {
          show: false,
          lineStyle: {
            color: "#b8d6f3"
          }
        },
        axisTick: { show: false },
        axisLabel: {
          formatter: "{value}%"
        }
      },
      series: [
        {
          name: "处理比例",
          data: [92, 97, 68, 59, 76, 89, 79],
          type: "bar",
          barWidth: 15,
          itemStyle: {
            normal: {
              //
              color: new echarts.graphic.LinearGradient(
                0,
                0,
                0,
                1, //4个参数用于配置渐变色的起止位置, 这4个参数依次对应右/下/左/上四个方位. 而0 0 0 1则代表渐变色从正上方开始
                [
                  { offset: 0, color: "#ffc107" },
                  { offset: 0.5, color: "#ff9800" },
                  { offset: 1, color: "#ff5722" }
                ]
              ),
              label: {
                show: true,
                position: "top",
                formatter: "{c}%", //这是关键，在需要的地方加上就行了
                color: "#fff"
              }
            }
          }
        }
      ],
      legend: {
        //设置图例
        right: 10,
        data: [
          {
            name: "处理比例"
          }
        ],
        icon: "rect",
        itemWidth: 10, // 设置宽度
        itemHeight: 10, // 设置高度
        textStyle: {
          color: "#fff"
        }
      }
    };
    this.lineOption = {
      grid: {
        left: "1%",
        right: "3%",
        bottom: "0%",
        containLabel: true
      },
      xAxis: {
        type: "category",
        // boundaryGap: false,
        data: [
          "2020-04-01",
          "2020-04-02",
          "2020-04-03",
          "2020-04-04",
          "2020-04-05",
          "2020-04-06",
          "2020-04-07"
        ],
        axisLine: {
          lineStyle: {
            color: "#b8d6f3",
            fontSize: 14
          }
        },
        axisTick: {
          show: false
        }
      },
      yAxis: {
        name: "单位：万（条）",
        nameTextStyle: {
          // align: "center"
          padding: [3, 4, 5, 20]
        },
        type: "value",
        axisLine: {
          show: false,
          lineStyle: {
            color: "#b8d6f3"
          }
        },
        max: 600,
        axisTick: { show: false },
        splitLine: {
          lineStyle: {
            type: "dotted",
            color: "#6d84bd"
          }
        }
      },
      series: [
        {
          name: "数据量",
          type: "line",
          data: [404, 497, 410, 496, 400, 198, 396],
          symbolSize: 4,
          areaStyle: {
            normal: {
              color: new echarts.graphic.LinearGradient(
                0,
                0,
                0,
                1, //4个参数用于配置渐变色的起止位置, 这4个参数依次对应右/下/左/上四个方位. 而0 0 0 1则代表渐变色从正上方开始
                [
                  { offset: 0, color: "#14dbad" },
                  { offset: 0.5, color: "#426fac" },
                  { offset: 1, color: "#0c3291" }
                ]
              )
            }
          },
          itemStyle: {
            normal: {
              color: "#9effec", //改变折线点的颜色
              lineStyle: {
                color: "#14dbad" //改变折线颜色
              }
            }
          },
          label: {
            show: true,
            position: "top",
            formatter: "{c}", //这是关键，在需要的地方加上就行了
            color: "#fff"
          }
        }
      ],
      legend: {
        right: 10,
        data: [
          {
            name: "数据量"
          }
        ],
        textStyle: {
          color: "#fff"
        },
        itemWidth: 25, // 设置宽度
        itemHeight: 4 // 设置高度
      }
    };
    this.radarOption = {
      radar: {
        center: ["50%", "50%"], // 图的位置
        name: {
          textStyle: {
            color: "#b8d6f3",
            borderRadius: 3,
            padding: [3, 5]
          }
        },
        nameGap: 3,
        indicator: [
          { name: "4月1日", max: 6500 },
          { name: "4月2日", max: 16000 },
          { name: "4月3日", max: 30000 },
          { name: "4月4日", max: 38000 },
          { name: "4月5日", max: 52000 },
          { name: "4月6日", max: 25000 },
          { name: "4月7日", max: 30000 }
        ],
        radius: 50,
        splitArea: {
          show: true,
          areaStyle: {
            color: []
            // 图表背景网格的颜色
          }
        }
      },
      series: [
        {
          name: "预算 vs 开销（Budget vs spending）",
          type: "radar",
          data: [
            {
              value: [4300, 10000, 28000, 25000, 40000, 19000, 28000],
              name: "预算分配（Allocated Budget）",
              symbolSize: 2,
              areaStyle: {
                normal: {
                  type: "default",
                  opacity: 0.5, // 图表中各个图区域的透明度
                  color: "#894453" // 图表中各个图区域的颜色
                }
              },
              lineStyle: {
                color: "#894453" // 图表中各个图区域的边框线颜色
              }
            },
            {
              value: [2000, 4000, 8000, 9000, 10000, 6000, 8000],
              name: "实际开销（Actual Spending）",
              symbolSize: 0,
              areaStyle: {
                normal: {
                  type: "default",
                  opacity: 0.9, // 图表中各个图区域的透明度
                  color: "#efceca" // 图表中各个图区域的颜色
                }
              },
              lineStyle: {
                color: "#efceca" // 图表中各个图区域的边框线颜色
              }
            }
          ]
        }
      ]
    };
    this.bar.setOption(this.barOption);
    this.line.setOption(this.lineOption);
    this.radar.setOption(this.radarOption);

    let _this = this;
    window.onresize = () => {
      _this.pageResize();
    };
  },
  methods: {
    pageResize() {
      this.$nextTick(() => {
        this.bar.resize();
        this.line.resize();
        this.radar.resize();
      });
    }
  }
};
</script>

<style lang="scss" scoped>
.homepage {
  color: #fff;
  padding: 20px 10px 20px 20px;
  .select {
    text-align: left;
  }
  .sub-title {
    border-left: 3px solid #0255a4;
    text-align: left;
    font-size: 14px;
    padding: 0px 3px;
  }
  .top {
    display: flex;
    justify-content: space-around;
    & > div {
      box-shadow: 0 0 3px 3px #0255a4;
      margin: 10px;
      padding: 5px;
    }
    .left {
      flex: 1;
    }
    .center {
      flex: 2;
      div {
        width: 100%;
        height: 230px;
      }
    }
    .right {
      width: 200px;
    }
  }
  .bottom {
    display: flex;
    & > div {
      box-shadow: 0 0 3px 3px #0255a4;
      margin: 10px;
      padding: 5px;
    }
    .left {
      flex: 3;
      div {
        width: 100%;
        height: 230px;
      }
    }
    .right {
      width: 200px;
      div {
        width: 100%;
        height: 230px;
      }
    }
  }
}
</style>
