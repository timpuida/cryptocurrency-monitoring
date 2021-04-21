<template>
 <div class="chartdiv" ref="chartdiv">
 </div>
</template>

<script>
import * as am4core from "@amcharts/amcharts4/core";
import * as am4charts from "@amcharts/amcharts4/charts";
import am4themes_animated from "@amcharts/amcharts4/themes/animated";

am4core.useTheme(am4themes_animated);

export default {
  name: 'Chart',
  props: {
    chartProps:{
      type: Array,
      default: ()=>[]
    },
    socketRecieve: {
      type: Object,
    }
  },
  data(){
    return {
      chartData: [],
      chart: null,
      chartCurrentData:{}
    }
  },
  
 mounted() {
    this.init();
  },
  methods: {
    init(){
      let chart = am4core.create(this.$refs.chartdiv, am4charts.XYChart);

        chart.paddingRight = 30;

      this.chartData = this.chartProps.map(item =>{ 
        return {
          time: item.time*1000,
          val: item.close
        }
        });
        chart.data = this.chartData;

        let dateAxis = chart.xAxes.push(new am4charts.DateAxis());
        dateAxis.renderer.grid.template.location = 0;

        let valueAxis = chart.yAxes.push(new am4charts.ValueAxis());
        valueAxis.tooltip.disabled = true;
        valueAxis.renderer.minWidth = 35;

        let series = chart.series.push(new am4charts.LineSeries());
        series.dataFields.dateX = "time";
        series.dataFields.valueY = "val";

        series.tooltipText = "{valueY.value}";
        chart.cursor = new am4charts.XYCursor();

        let scrollbarX = new am4charts.XYChartScrollbar();
        scrollbarX.series.push(series);
        chart.scrollbarX = scrollbarX;

        this.chart = chart;
      }
  },
  watch:{
    socketRecieve() {
      if (this.chart){
        this.chartCurrentData.val = this.socketRecieve.P;
        this.chartCurrentData.time = this.socketRecieve.TS*1000;
        this.chart.addData(this.chartCurrentData)
      }
    },
    chartProps(){
      this.chart.dispose();
      this.init()
    }
      
  },
  beforeDestroy() {
    if (this.chart) {
     this.chart.dispose();
    }
  }
}
</script>

<style scoped>

</style>
