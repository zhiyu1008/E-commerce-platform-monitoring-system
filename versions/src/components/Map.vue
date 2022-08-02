<template>
  <div class="com-container" @dblclick="revertMap">
    <div class="com-chart" ref="map_ref"></div>
  </div>
</template>

<script>
import axios from "axios";
// import mapData from "../../public/static/map/china.json";
import { getProvinceMapInfo } from "@/utils/map_utils";
import { mapState } from "vuex";
export default {
  components: {},
  data() {
    return {
      chartInstance: null,
      allData: null,
      mapData: {}, // 所获取的省份的地图矢量数据
    };
  },
  created() {
    // 在组件创建完成之后 进行回调函数的注册
    this.$socket.registerCallBack("mapData", this.getData);
  },
  mounted() {
    this.initChart();
    // this.getData()
    this.$socket.send({
      action: 'getData',
      socketType: 'mapData',
      chartName: 'map',
      value: ''
    })
    window.addEventListener("resize", this.screenAdapter);
  },
  destroyed() {
    window.removeEventListener("resize", this.screenAdapter);
    this.$socket.unRegisterCallBack('mapData')
  },
  methods: {
    async initChart() {
      this.chartInstance = this.$echarts.init(this.$refs.map_ref, this.theme);
      // 获取中国地图的矢量数据
      // http://localhost:8999/static/map/china.json
      // 由于我们现在获取的地图矢量数据并不是位于KOA2的后台, 所以咱们不能使用this.$http
      const ret = await axios.get('http://localhost:9999/static/map/china.json')
      this.$echarts.registerMap('china', ret.data)
      // this.$echarts.registerMap("china", mapData);
      const initOption = {
        title: {
          text: "▎ 商家分布",
          left: 20,
          top: 20,
        },
        geo: {
          type: "map",
          map: "china",
          // 地图的位置和颜色
          top: "5%",
          bottom: "5%",
          itemStyle: {
            areaColor: "#2E72BF",
            borderColor: "#333",
          },
        },
        legend: {
          left: "5%",
          bottom: "5%",
          orient: "vertical", // 从上向下
        },
      };
      this.chartInstance.setOption(initOption);
      this.chartInstance.on("click", async (arg) => {
        // arg.name 得到所点击的省份, 这个省份他是中文
        const provinceInfo = getProvinceMapInfo(arg.name);
        console.log(provinceInfo);
        // 需要获取这个省份的地图矢量数据
        // 判断当前所点击的这个省份的地图矢量数据在mapData中是否存在,可以优化性能（相同点的ajax请求）
        if (!this.mapData[provinceInfo.key]) {
          const ret = await axios.get(
            "http://localhost:9999" + provinceInfo.path
          );
          this.mapData[provinceInfo.key] = ret.data;
          this.$echarts.registerMap(provinceInfo.key, ret.data);
        }
        const changeOption = {
          geo: {
            map: provinceInfo.key,
          },
        };
        this.chartInstance.setOption(changeOption);
      });
    },
    async getData() {
      const { data: ret } = await this.$http.get("map");
      this.allData = ret;
      // console.log(this.allData);
      this.updateChart();
    },
    updateChart() {
      // 处理图表需要的数据
      // 图例的数据
      const legendArr = this.allData.map((item) => {
        return item.name;
      });
      const seriesArr = this.allData.map((item) => {
        return {
          type: "effectScatter",
          // 涟漪效果
          rippleEffect: {
            scale: 5,
            brushType: "stroke",
          },
          name: item.name,
          data: item.children,
          coordinateSystem: "geo",
        };
      });
      const dataOption = {
        legend: {
          data: legendArr,
        },
        series: seriesArr,
      };
      this.chartInstance.setOption(dataOption);
    },
    screenAdapter() {
      const titleFontSize = (this.$refs.map_ref.offsetWidth / 100) * 3.6;
      const adapterOption = {
        // 标题大小
        title: {
          textStyle: {
            fontSize: titleFontSize,
          },
        },
        // 图例大小
        legend: {
          itemWidth: titleFontSize / 2,
          itemHeight: titleFontSize / 2,
          // 图例间隔
          itemGap: titleFontSize / 2,
          textStyle: {
            fontSize: titleFontSize / 2,
          },
        },
      };
      this.chartInstance.setOption(adapterOption);
      this.chartInstance.resize();
    },
    // 回到中国地图
    revertMap() {
      const revertOption = {
        geo: {
          map: "china",
        },
      };
      this.chartInstance.setOption(revertOption);
    },
  },
  computed: {
    ...mapState(["theme"]),
  },
  watch: {
    theme() {
      console.log("主题切换了");
      this.chartInstance.dispose(); // 销毁当前的图表
      this.initChart(); // 重新以最新的主题名称初始化图表对象
      this.screenAdapter(); // 完成屏幕的适配
      this.updateChart(); // 更新图表的展示
    },
  },
};
</script>
<style scoped lang="scss"></style>
