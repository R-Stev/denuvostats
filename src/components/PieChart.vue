<template>
  <div>
    <q-card :class="$q.dark.isActive?'bg-dark':''">
      <q-card-section class="text-h6">
        {{ publisherName }}
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
    publisherName: String
  },
  watch: {
    chartData: function(newOne){
      this.options.dataset = newOne;
    }
  },
  components: {
    ECharts
  },
  data() {
    const $q = useQuasar()

    return {
    $q,
      options: {
        // tooltip: {
        //   trigger: 'item',
        //   formatter: '{a} <br/>{b}: {c} ({d}%)'
        // },
        // legend: {
        //   top: 'bottom',
        //   bottom: '5%',
        //   left: 'center'
        // },
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
              formatter: '{b} months: {@number} ({d}%)'
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
