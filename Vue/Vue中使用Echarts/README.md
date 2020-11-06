# Vue中使用Echarts
1.准备工作
```
npm install echarts --save
```
main.js
```
import Echarts from 'echarts'
Vue.prototype.$echarts = Echarts
```

# 地图
## 1.1 百度地图

`public/index.html`的`head`的标签中 加入
```html
<script src="http://api.map.baidu.com/api?v=2.0&ak=53oVIOgmSIejwV7EfphPgTynOZbIiVYu"></script>
```

从 `https://github.com/apache/incubator-echarts/tree/master/extension/bmap` 目录下将bmap.js下载到项目js中
mapChart.vue
```
<template>
  <div class="com-page">
    <div class="com-container" id="map"></div>
  </div>
</template>
<script>
import '../js/bmap.js'

  mounted() {
   this.$nextTick(() => {
       this.mapChart();
    })
  }
</script>
```


```html
  let myChart = this.$echarts.init(document.getElementById("map"));
      myChart.setOption({
        bmap: {
          center: [120.13066322374, 30.240018034923],
          zoom: 14,
          roam: true,
          mapStyle: {
            styleJson: [
              {
                featureType: "land", //调整土地颜色
                elementType: "geometry",
                stylers: {
                  color: "#081734",
                },
              },
              {
                featureType: "building", //调整建筑物颜色
                elementType: "geometry",
                stylers: {
                  color: "#04406F",
                },
              },
              {
                featureType: "building", //调整建筑物标签是否可视
                elementType: "labels",
                stylers: {
                  visibility: "off",
                },
              },
              {
                featureType: "highway", //调整高速道路颜色
                elementType: "geometry",
                stylers: {
                  color: "#015B99",
                },
              },
              {
                featureType: "highway", //调整高速名字是否可视
                elementType: "labels",
                stylers: {
                  visibility: "off",
                },
              },
              {
                featureType: "arterial", //调整一些干道颜色
                elementType: "geometry",
                stylers: {
                  color: "#003051",
                },
              },
              {
                featureType: "arterial",
                elementType: "labels",
                stylers: {
                  visibility: "off",
                },
              },
              {
                featureType: "green",
                elementType: "geometry",
                stylers: {
                  visibility: "off",
                },
              },
              {
                featureType: "water",
                elementType: "geometry",
                stylers: {
                  color: "#044161",
                },
              },
              {
                featureType: "subway", //调整地铁颜色
                elementType: "geometry.stroke",
                stylers: {
                  color: "#003051",
                },
              },
              {
                featureType: "subway",
                elementType: "labels",
                stylers: {
                  visibility: "off",
                },
              },
              {
                featureType: "railway",
                elementType: "geometry",
                stylers: {
                  visibility: "off",
                },
              },
              {
                featureType: "railway",
                elementType: "labels",
                stylers: {
                  visibility: "off",
                },
              },
              {
                featureType: "all", //调整所有的标签的边缘颜色
                elementType: "labels.text.stroke",
                stylers: {
                  color: "#313131",
                },
              },
              {
                featureType: "all", //调整所有标签的填充颜色
                elementType: "labels.text.fill",
                stylers: {
                  color: "#FFFFFF",
                },
              },
              {
                featureType: "manmade",
                elementType: "geometry",
                stylers: {
                  visibility: "off",
                },
              },
              {
                featureType: "manmade",
                elementType: "labels",
                stylers: {
                  visibility: "off",
                },
              },
              {
                featureType: "local",
                elementType: "geometry",
                stylers: {
                  visibility: "off",
                },
              },
              {
                featureType: "local",
                elementType: "labels",
                stylers: {
                  visibility: "off",
                },
              },
              {
                featureType: "subway",
                elementType: "geometry",
                stylers: {
                  lightness: -65,
                },
              },
              {
                featureType: "railway",
                elementType: "all",
                stylers: {
                  lightness: -40,
                },
              },
              {
                featureType: "boundary",
                elementType: "geometry",
                stylers: {
                  color: "#8b8787",
                  weight: "1",
                  lightness: -29,
                },
              },
            ],
          },
        },
        series: [
          {
            type: "scatter",
            coordinateSystem: "bmap",
            data: [[120, 30, 1]],
          },
        ],
      });
    myChart.on("click", function (params) {
        console.log(params);
        if (params.componentSubType === "scatter") {
          console.log(123);
        }
      });
      var bmap = myChart.getModel().getComponent('bmap').getBMap();
      bmap.addControl(new BMap.MapTypeControl());
```






## 1.2 高德地图
`public/index.html`的`head`的标签中 加入
```html
<script src="//webapi.amap.com/maps?v=1.4.15&key=8f128538b68147617c0c326272c52276&plugin=AMap.CustomLayer"></script>
```


```
npm install echarts-extension-amap --save
```
```
require("echarts-extension-amap");
  mounted() {
   this.$nextTick(() => {
       this.mapChart();
    })
  }
```



```html
 var data = [
        { name: "海门", value: 9 },
        { name: "鄂尔多斯", value: 12 },
        { name: "招远", value: 12 },
        { name: "舟山", value: 12 },
        { name: "齐齐哈尔", value: 14 },
        { name: "盐城", value: 15 },
        { name: "赤峰", value: 16 },
        { name: "青岛", value: 18 },
        { name: "乳山", value: 18 },
        { name: "金昌", value: 19 },
      ];

      var geoCoordMap = {
        海门: [121.15, 31.89],
        鄂尔多斯: [109.781327, 39.608266],
        招远: [120.38, 37.35],
        舟山: [122.207216, 29.985295],
        齐齐哈尔: [123.97, 47.33],
        盐城: [120.13, 33.38],
        赤峰: [118.87, 42.28],
        青岛: [120.33, 36.07],
        乳山: [121.52, 36.89],
        金昌: [102.188043, 38.520089],
      };

      var convertData = function (data) {
        var res = [];
        for (var i = 0; i < data.length; i++) {
          var geoCoord = geoCoordMap[data[i].name];
          if (geoCoord) {
            res.push({
              name: data[i].name,
              value: geoCoord.concat(data[i].value),
            });
          }
        }
        return res;
      };

      var iconGD =
        "image://data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAAyCAMAAAA3KEgwAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNi1jMDY3IDc5LjE1Nzc0NywgMjAxNS8wMy8zMC0yMzo0MDo0MiAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENDIDIwMTUgKFdpbmRvd3MpIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjc3NDEzNTQ1NjJDMzExRUE5OUREQ0U1NUREMjAzNzk3IiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjc3NDEzNTQ2NjJDMzExRUE5OUREQ0U1NUREMjAzNzk3Ij4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6Nzc0MTM1NDM2MkMzMTFFQTk5RERDRTU1REQyMDM3OTciIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6Nzc0MTM1NDQ2MkMzMTFFQTk5RERDRTU1REQyMDM3OTciLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7BVZrKAAADAFBMVEXl6erGvrLKuq2goKBjaW6doaWqqaklKSoOEBTJys3m28lSVFS1tbW7vsBUXGZTWVuqnJF6gYLq4c5ESlLZy7xLTU24ubk8QEJ5dGs0OUmBfHNjbHF9fX5FSUqtra1sa2RlYFu8raCkm4+JhXvVybl7enDs5NCenp5eYWGSnZ2zpJlrcXXy7Ne7saT09faNjY1OUVJ3fYQ5PD3h1sW5qZ1zdn1OVmQzNDSkpKSqrrPk2MeCgHre0cHEt6mwsLBdZmqmpqZoaWlOVFmJiYnCuauEjIzLwLGssrR4eHhFTVuVlZVbWlezuLm3sKOFhYU2OTk7QUqBhYZaXF3R09RQVlvZ2tuCgoKYmZpjXFWVeXZgYmLMxrqblIliZGVCR0qsopXRxbbVw7ZBREXAsaRqbnAvMjJ6fH2+vr5cXl+MmpqKhYCQkJB9iYmAgIDGycqGjpXVw7WVk45sbm5jZ2rItqrg1MMqLTFAQkKNkpRvamK+wcNxdHZeaXGcnJxrbG+fo6qprK5fZW6RjIJ1eHm0rqCRlZbAtabDwsKemYywoZZFRkfo3syVkIRvenp1fH1ITlJ2dnY9RVCmq67///3PvrGam55udXpyc3JqZl+zsrJmZ2esq6tYWlnc0L+gpKVeXlmanZ5qbGvDs6aSkpRbXVmmoJN1gYGKi4tnZmJyenrDx8hvcHGOiX52enxaZG9XYGWwpplzc3QyNjibn6BbZGkfIiQ/Q0ZxcnFvcG/g08LYxri0s7NycnFfaG2xsbG8t6yuq6YZHB5wdHJ0c3J1dXHg3t+0t72bm5ucm5zo49qCg4SCh4iDg4NHSEiio6SgoaOhoqKhoaHXxbd8goqjo6ODf3aIiIdGTmRubmeLjo/289+zsqzb3uCTlZNlb3F/f39mc3mvr6+dkYaff33FtKdaYWhWV1ZWYGxdZG5IUFxEQj2np6diY2BYW1yysrHw8vT28+/TwLNxeHWtqZ/b19Db1sydnZ1FRUODkZDKzs9weX6xtLWEhITh1MSioqIAAACuYItKAAABAHRSTlP///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////8AU/cHJQAABnNJREFUeNqU1ntYUmcYAHDiNEAwMfCAilKmgkkFoSljpdJtxlLMTBuFmQmSSqtNITOzza7QRMul81LzUnZZNwu1TDfWtlppjabNdsm1GmXU2trWVoNzdgRR3NMF3//O85zf837ve77vPR8KHkmYrdFhf0Q5hTgPfJPFCva1DrM50UemYI8E80xdwhPJVMmsBpkskQ20Urqcx4YuXhS91SAQ+SogqH1rOobtk+gkhmxhqIyYlcpNWpgOXJP5IIU7hSF7MHlbPyQ09gp9fKxdcwbbKcA0ubq6UAhKCTvWWTxoATqFQomoNBFFb7OdxPZ6mbxWiiulHWDSMQRuIbvDGTyYFkNBbAQAQIb2C6m+5llO4CgTz9opqJ3iarPIIloTG8Vsm30BzszMFAImHgCYXIcsBPHocu4RdsdLcGJvb6+5DuAhnXKwkOGmkEpsiH0xtrY0URbbnu6K2Jt2C0GX2wXcPLYTuEORaYigDLOQgS6kcmAncG+DISmJvrWdOWQhqKuywugETozNB/LzHwsNPAcLfR0FxBY9A1ssjrZDITScZ/LMQQrMMN0VteCgbOz/sMUaQ7iXbchn8mIFLRXXKrscMW9BUdHB4dhicdDWRT9mWm1Zi0BYaRjqmMk0Nu3gsGVbhiIHhqddNHckCgFmF2Kbm8tqmiHT4BnB/LqAyGKpUgexZViE5KyZ5qNgAwDPapubz1aYK+2WXlnTlCwG1VTVACZpLDmOqfkhmxRTzxvstrm5pSYJY1tzBM8YrlKrOqnht25Z8Sj/LSTN6WEaH/DHB0mxgsNnbbhMoLDh1vOgSJ3aSZATx7/Zn9nl0x4vbWAKA+/ILXy8x5pjggGLpG7JN0EGnmtmqkit5sSLJXJQjeCbMFy/fv087cenGfzhnD85esOgFjRgDFGtZq5IDVYFScRqrE4KozA7YbiPFkk7Oo+wjc8IGSo9hI/no8s22HGNIgrTXizigka5WGKU6rGNXBjF/BL272uLjIyk3SHXH9eQbJ3LseD502vLvTM2nDtnS14TTY+oCFcREBpP1eMIG2UIvvwq3BNG69eRNLeScVsYmhyE8zXTS905YXLvEy0nivv1WUEsJj4cjA+SyFU6HC750JRDIIwCxsCEPu1oWr9uo4WRPVOQ0vH80PG5uW67yzNGk8lp/bqsuKOq0RgkloBKnR6n2xMhW8dBGvbKku6SEi0x0pq8rb9z0zUenqDb/dx3WAeMd56SS1jR0dHFh0EpRyyOD9crfZt01CeFR3Y29n+qGYwJRK+EG6do1uy0o8p9+MXzWLvH5dLCymvijk1NThAYjUaVVCQXP8DipHuepOk4C588CbJuTw8LqbpbPnr9zJk32iLbiEffSu8mT0tjjSsne4Z+dieOtl4LclNFUmynQCLC6ZMW/XASy92zsFBpxX+iq0mMUeNK+rTaY21urILv/UBWWt7dN7ZPuLqaczRBu7vvfYJSKoXheDEWV7Uq/uRHepGvr9R2qn5LmcHXkPDH788jE0+dSv8mMDghL68NdF9+ptTP2PdLctaN/Ql66xGQ6nHXXQTz02Csyj5JFn03d28MA8/gL+W4sZ5Sph3/2a3wLhh6tXZFcDBYMvXYP1mFV2xnD4fTXUeh1oqGrhX3lN3T98X5I7kZHv6pYf9e8vMe/S4Y6rFi+zI/9KOfJt4ocimue936sl6vfLDnOhcewq8dxnG3ecbtjcGTGIyYzwt+XF7+LSH7TLB7cOnV20Urd445Ubd5f9ZAaqWqUQo74KovXLqXlXeqyQT/uRqGhl4wynt8Ldo9OPv327WBTekL0+c8ypp4EXzWhEVqVhV8MmHTjr/BwPq4vXNDlsRMvo0OdV9xJtv7flVk1vx1a/evrJMR4OdgWJfb47V0y2LCvtk70CkpASvcg/3QVycEJWi54fI5KydemtX/TZ+H4b0B27xmT9pxz3sSqdrjr9mh6OzVS2cmqJVYKdc4dqNP+PN+DAMDcFJAQEDPe93lxxlfPZzrl13qV0XkYg8fQSakSv38H/jg6MWf7vE6sHS895RdpX7ZqzOInfqxD1GSF99YHIY+aTKhOgPXWVlbi/bIIEoTVl0/JHMaI4uf3MRT4zyrl7sbqcorG5vmB40Aw8uUrrvyFgfXkxtx0vmHxpyUjgQje7/hApfqBepw0s0oSio8MgzDyIBE9qFeRKigwiPGA4Glhr/0TvufAAMAxMxYn/ZXll8AAAAASUVORK5CYII=";

      let myChart = this.$echarts.init(document.getElementById("map"));
      var option = {
        // 加载 amap 组件
        amap: {
          // 3D模式，无论你使用的是1.x版本还是2.x版本，都建议开启此项以获得更好的渲染体验
          viewMode: "3D",
          // 高德地图支持的初始化地图配置
          // 高德地图初始中心经纬度
          center: [108.39, 39.9],
          // 高德地图初始缩放级别
          zoom: 4,
          // 是否开启resize
          resizeEnable: true,
          // 自定义地图样式主题
          mapStyle: "amap://styles/dark",
          // 移动过程中实时渲染 默认为true 如数据量较大 建议置为false
          renderOnMoving: true,
          // 高德地图自定义EchartsLayer的zIndex，默认2000
          echartsLayerZIndex: 100,
          // 说明：如果想要添加卫星、路网等图层
          // 暂时先不要使用layers配置，因为存在Bug
          // 建议使用amap.add的方式，使用方式参见最下方代码
        },
        tooltip: {
          show: true,
          formatter: function (params, ticket, callback) {
            // return e.data.displayname;
            console.log(params, ticket, callback);
            return "hello";
          },
          // position: [500, 10],
          triggerOn: "click",
        },
        series: [
          {
            name: "PM2.5",
            type: "scatter",
            // 使用高德地图坐标系
            coordinateSystem: "amap",
            data: convertData(data),
            symbolSize: 40,
            // symbolSize: function (val) {
            //     return val[2] / 10;
            // },
            // encode: {
            //  // 编码使用数组中第三个元素作为value维度
            //   value: 2
            // },
            symbol: iconGD,
            // symbol:'image:/http://datav.oss-cn-hangzhou.aliyuncs.com/uploads/images/666a56ba532990f9c7185781f6df6edb.png',
            // label: {
            //     normal: {
            //         formatter: function(arg){
            //           return arg.value[2]
            //         },
            //         position: 'right',
            //         show: true
            //     },
            //     emphasis: {
            //         show: true
            //     }
            // },
            // itemStyle: {
            //     normal: {
            //         color: '#00c1de'
            //     }
            // }
          },
          {
            name: "Top 5",
            type: "effectScatter",
            coordinateSystem: "amap",
            data: convertData(
              data
                .sort(function (a, b) {
                  return b.value - a.value;
                })
                .slice(0, 6)
            ),
            symbolSize: function (val) {
              return 20;
            },
            encode: {
              value: 2,
            },
            showEffectOn: "render",
            rippleEffect: {
              brushType: "stroke",
            },
            hoverAnimation: true,
            label: {
              normal: {
                formatter: "{b}",
                position: "right",
                show: true,
              },
            },
            itemStyle: {
              normal: {
                color: "#fff",
                shadowBlur: 10,
                shadowColor: "#333",
              },
            },
            zlevel: 1,
          },
        ],

        // [
        //   {
        //     type: "scatter",
        //     // 使用高德地图坐标系
        //     coordinateSystem: "amap",
        //     // 数据格式跟在 geo 坐标系上一样，每一项都是 [经度，纬度，数值大小，其它维度...]
        //     data: [
        //       [120, 30, 8],
        //       [120.1, 30.2, 20],
        //     ],
        //     encode: {
        //       value: 2,
        //     },
        //     lable:{
        //       formatter:"{b}",
        //       positon:'right',
        //       show:true
        //     }
        //   },
        // ],
      };

      myChart.setOption(option);
      myChart.on("click", function (params) {
        console.log(params);
        if (params.componentSubType === "scatter") {
          console.log(123);
        }
      });
      // 获取高德地图实例，使用高德地图自带的控件(需要在高德地图js API script标签手动引入)
      var amap = myChart.getModel().getComponent("amap").getAMap();
      console.log('amap ',amap)
      // 添加控件
      // amap.addControl(new AMap.Scale());
      // amap.addControl(new AMap.ToolBar());
      // // 添加图层
      var satelliteLayer = new AMap.TileLayer.Satellite();
      var roadNetLayer = new AMap.TileLayer.RoadNet();
      amap.add([satelliteLayer, roadNetLayer]);
```
