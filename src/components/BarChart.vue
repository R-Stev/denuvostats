<template>
  <div>
    <q-card :class="$q.dark.isActive?'bg-dark':''">
      <q-card-section class="text-h6">
        <q-btn icon="download" class="float-right" @click="SaveImage" flat dense>
          <q-tooltip>Download SVG</q-tooltip>
        </q-btn>
      </q-card-section>
      <q-card-section>
        <ECharts ref="barchart"
          :option="options"
          class="q-mt-md"
          :resizable="true"
          autoresize style="height: 800px;"
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
    chartData: Array
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
          text: 'Bar Chart',
        },
        // tooltip: {
        //   trigger: 'item',
        //   formatter: '{a} <br/>{b}: {c} ({d}%)'
        // },
        dataset: {
          dimensions: ['publisher', 'percent'],
          source: this.chartData,
        },
        grid: {
          left: '10',
          right: '20',
          bottom: '10',
          top: '30',
          containLabel: true,
        },
        xAxis: {
          type: 'value',
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
  // watch: {
  //   '$q.dark.isActive': function () {
  //     this.init();
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
