<template>
  <div>
    <q-card :class="$q.dark.isActive?'bg-dark':''">
      <q-card-section class="text-h6">
        <!-- {{ publisherName }} -->
        <q-btn icon="download" class="float-right" @click="SaveImage" flat dense>
          <q-tooltip>Download SVG</q-tooltip>
        </q-btn>
      </q-card-section>
      <q-card-section>
        <ECharts ref="piechart"
          :option="options"
          class="q-mt-md"
          :resizable="true"
          autoresize style="height: 250px;"
        />
      </q-card-section>
    </q-card>
  </div>
</template>

<script>
import ECharts from 'vue-echarts'
import * as echarts from 'echarts/core';
import { BarChart, PieChart } from 'echarts/charts';
import {
  TitleComponent,
  TooltipComponent,
  GridComponent,
  DatasetComponent,
  TransformComponent
} from 'echarts/components';
import { LabelLayout, UniversalTransition } from 'echarts/features';
import { SVGRenderer } from 'echarts/renderers';

echarts.use([
  BarChart,
  PieChart,
  TitleComponent,
  TooltipComponent,
  GridComponent,
  DatasetComponent,
  TransformComponent,
  LabelLayout,
  UniversalTransition,
  SVGRenderer
]);

import {useQuasar} from "quasar"

export default {
  name: "PieChart",
  props: {
    chartData: Array,
    publisherName: String,
    denuvoStatus: String
  },
  watch: {
    chartData: function(newOne){
      this.options.dataset = newOne;
    },
    publisherName: function(newOne){
      this.options.title.text = newOne;
    },
    // '$q.dark.isActive': function (val) {
    //   this.options.series[0].label.color = val ? '#e2e2e2' : '#1d1d1d'
    //   this.options.title.textStyle.color = val ? '#e2e2e2' : '#1d1d1d'
    // }
},
  components: {
    ECharts
  },
  methods: {
    SaveImage() {
      const linkSource = this.$refs.piechart.getDataURL();
      const downloadLink = document.createElement('a');
      document.body.appendChild(downloadLink);

      downloadLink.href = linkSource;
      downloadLink.target = '_self';
      downloadLink.download = this.publisherName.concat(' ', this.denuvoStatus, ' PieChart.png');
      downloadLink.click();
    }
  },
  data() {
    const $q = useQuasar()

    return {
    $q,
      options: {
        title: {
          text: this.publisherName,
          // textStyle: {
          //   color: $q.dark.isActive ? '#e2e2e2' : '#1d1d1d'
          // }
        },
        grid: {
          left: '0',
          right: '0',
          bottom: '10',
          top: '30',
          containLabel: true
        },
        dataset: this.chartData,
        series: [
          {
            name: 'Access source',
            type: 'pie',
            radius: ['0%', '70%'],
            center: ['50%', '50%'],
            avoidLabelOverlap: true,
            itemStyle: {
              borderRadius: 0,
              borderColor: '#fff',
              borderWidth: 2
            },
            label: {
              formatter: '{b} months:\n{@number} ({d}%)',
              // color: $q.dark.isActive ? '#e2e2e2' : '#1d1d1d'
              color: '#6e7079'
            },
            stillShowZeroSum: false,
            datasetIndex: 0
          }
        ]
      }
    }
  }
}
</script>

<style scoped>

</style>
