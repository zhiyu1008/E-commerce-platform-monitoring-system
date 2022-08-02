<template>
  <div class="com-container">
    <div class="com-chart" ref="map_ref"></div>
  </div>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      chartInstance: null,
      allData: null,
    };
  },
  mounted() {
    this.initChart();
    this.getData();
    window.addEventListener("resize", this.screenAdapter);
  },
  destroyed() {
    window.removeEventListener("resize", this.screenAdapter);
  },
  methods: {
    initChart() {
      this.chartInstance = this.$echarts.init(this.$refs.map_ref);
      const initOption = {};
      this.chartInstance.setOption(initOption);
    },
    async getData() {
      this.updataChart();
    },
    updataChart() {
      // 处理图表需要的数据
      const dataOption = {};
      this.chartInstance.setOption(dataOption);
    },
    screenAdapter() {
      const adapterOption = {};
      this.chartInstance.setOption(adapterOption);
      this.chartInstance.resize();
    },
  },
};
</script>
<style scoped lang="scss"></style>
