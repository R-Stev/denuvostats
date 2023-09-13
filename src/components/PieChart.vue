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
    }
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
        },
        // tooltip: {
        //   trigger: 'item',
        //   formatter: '{a} <br/>{b}: {c} ({d}%)'
        // },
        // legend: {
        //   top: 'bottom',
        //   bottom: '5%',
        //   left: 'center'
        // },
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
            avoidLabelOverlap: false,
            itemStyle: {
              borderRadius: 0,
              borderColor: '#fff',
              borderWidth: 2
            },
            label: {
            //   show: false,
            //   position: 'center',
              formatter: '{b} months: {@number} ({d}%)',
              // color: $q.dark.isActive ? '#ffffff' : '#1d1d1d'
              color: '#6e7079'
            },
            // labelLine: {
            //   show: false
            // },
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
