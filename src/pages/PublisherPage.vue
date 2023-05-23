<template>
  <q-page class="">
      <div >
        <pie-chart></pie-chart>
      </div>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'
import PieChart from "components/PieChart.vue"
// import BarChart from "components/BarChart.vue"


export default defineComponent({
  name: 'PublisherPage',
  components: {
    PieChart,
    // BarChart
  }
})
</script>
