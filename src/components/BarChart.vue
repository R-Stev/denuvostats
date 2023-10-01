<template>
  <div>
    <q-card :class="$q.dark.isActive?'bg-dark':''">
      <q-card-section class="text-h6">
        <q-btn icon="help" class="float-right" flat dense>
          <q-popup-proxy style="width: 75%">
            <q-banner>This bar chart displays the percentage of titles using Denuvo Anti-Tamber that have since removed it.  Tooltips for each publisher include the number of released titles, and exact percentage removed. </q-banner>
          </q-popup-proxy>
          <q-tooltip>Info</q-tooltip>
        </q-btn>
        <q-btn icon="download" class="float-right" @click="SaveImage" flat dense>
          <q-tooltip>Download SVG</q-tooltip>
        </q-btn>
      </q-card-section>
      <q-card-section>
        <ECharts ref="barchart"
          :option="options"
          class="q-mt-md"
          :resizable="true"
          autoresize :style="chartHeight"
        />
      </q-card-section>
    </q-card>
  </div>
</template>

<script>
import ECharts from 'vue-echarts'
import {useQuasar} from "quasar";

export default {
  name: "BarChart",
  props: {
    chartData: Array,
    chartHeight: String
  },
  components: {
    ECharts
  },
  data() {
    const $q = useQuasar()
    return {
      $q,
      model: false,
      options: {
        title: {
          // text: 'Bar Chart',
          // textStyle: {color: '#1d1d1d'}
        },
        tooltip: {
          trigger: 'item',
          formatter(params) {
            return `${params.marker}${params.name}<span style="float: right; margin-left: 20px">${params.data.removed + params.data.remain} titles, ${params.data.percent}% removed</span>`;
          }
        },
        dataset: {
          dimensions: ['publisher', 'percent'],
          source: this.chartData,
        },
        grid: {
          left: '10',
          right: '20',
          bottom: '10',
          top: '31',
          containLabel: true,
        },
        xAxis: {
          name: '%',
          nameLocation: 'center',
          type: 'value',
          // axisLabel: {
          //   color: '#00aa00'
          // }
        },
        yAxis: {
          type: 'category',
          inverse: true,
          // axisLabel: {
          //   color: '#00aa00'
          // }
        },
        series: [
          {
            type: 'bar',
            showBackground: true,
            barWidth: '100%',
            backgroundStyle: {
              color: 'rgba(220, 220, 220, 0.8)'
            }
          }
        ]
      },
      bar_chart: null
    }
  },
  // watch: {
  //   '$q.dark.isActive': function (val) {
  //     // For reasons unknown, changing axisLabel.color in the watcher causes a CSS error
  //     // in parsing value for 'fill' - causing the color change to be dropped.
  //     // this.options.xAxis.axisLabel.color = val ? '$aa0000' : '#00aa00';
  //     // this.options.yAxis.axisLabel.color = val ? '$aa0000' : '#00aa00';
  //     this.options.title.textStyle.color = val ? '#e2e2e2' : '#1d1d1d';
  //   }
  // },
  methods: {
    SaveImage() {
      const linkSource = this.$refs.barchart.getDataURL();
      const downloadLink = document.createElement('a');
      document.body.appendChild(downloadLink);

      downloadLink.href = linkSource;
      downloadLink.target = '_self';
      downloadLink.download = 'DenuvoRemovalPercentage_BarChart.png';
      downloadLink.click();
    }
  }
}
</script>

<style scoped>

</style>
