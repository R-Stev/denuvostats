<template>
  <div>
    <q-card>
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
          :theme="this.darkMode?'customdark':'customlight'"
          autoresize :style="chartHeight"
        />
      </q-card-section>
    </q-card>
  </div>
</template>

<script>
import {useQuasar} from "quasar";
import ECharts from 'vue-echarts'
import { use, registerTheme } from 'echarts/core';
import { BarChart } from 'echarts/charts'
import {
  TooltipComponent,
  DatasetComponent,
  GridComponent
} from 'echarts/components'
import { SVGRenderer } from 'echarts/renderers'

import customdark from 'src/assets/customdark';
import customlight from 'src/assets/customlight';

use([
  TooltipComponent,
  DatasetComponent,
  GridComponent,
  BarChart,
  SVGRenderer
])


export default {
  name: "BarChart",
  props: {
    chartData: Array,
    chartHeight: String,
    darkMode: Boolean
  },
  components: {
    ECharts
  },
  beforeMount() {
    registerTheme('customdark', customdark);
    registerTheme('customlight', customlight);
  },
  data() {
    return {
      model: false,
      options: {
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
          left: '0',
          right: '10',
          bottom: '10',
          top: '0',
          containLabel: true
        },
        xAxis: {
          name: '%',
          nameLocation: 'center',
          type: 'value'
        },
        yAxis: {
          type: 'category',
          inverse: true
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
