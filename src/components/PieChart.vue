<template>
  <div>
    <q-card class="no-box-shadow" :class="$q.dark.isActive?'bg-dark':''">
      <q-card-section class="text-h6">
        <!-- {{ publisherName }} -->
        <q-btn icon="help" class="float-right" flat dense>
          <q-popup-proxy style="width: 75%">
            <q-banner>This pie chart displays the amount (and percentage) of released titles that have used Denuvo Anti-Tamper for the selected publisher, grouped into various lifetime ranges.  The lifetime of Denuvo can be based on date of release or date of last update, and titles can be filtered to only those currently or formerly using Denuvo.</q-banner>
          </q-popup-proxy>
          <q-tooltip>Info</q-tooltip>
        </q-btn>
        <q-btn icon="download" class="float-right" @click="SaveImage" flat dense>
          <q-tooltip>Download SVG</q-tooltip>
        </q-btn>
      </q-card-section>
      <q-card-section>
        <ECharts ref="piechart"
          :option="options"
          class="q-mt-md pieChart"
          :resizable="true"
          autoresize
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
  @media only screen and (min-width: 701px) {
    .pieChart {
      height: 250px;
    }
  }
  @media only screen and (max-width: 700px) {
    .pieChart {
      height: 40vw;
      max-height: 250px;
    }
  }
</style>
