<template>
  <q-page class="q-pa-md q-col-gutter-md">
    <div class="row justify-between">
        <q-select v-model="pubSelector" :options="publisherList" label="Publisher"
        style="width: 300px" />
        <div class="self-center">
          Denuvo status:
          <q-btn-toggle
            v-model="statusFilter"
            toggle-color="primary"
            :options="[
              {label: 'All', value: 'all'},
              {label: 'Present', value: 'present'},
              {label: 'Removed', value: 'removed'}
            ]"
          />
        </div>
    </div>
    <div class="row">
      <div class="col-12">
        <pie-chart :chart-data="[this.dateBuckets]"
        :publisher-name="this.pubSelector"
        :denuvo-status="this.statusFilter" />
      </div>
      <div class="col-12">
        <!-- height should be some value of 48n + 104 -->
        <q-table class="my-sticky-header-table"
          title="Details" style="height: 488px"
          :rows="tableData" :columns="columns" row-key="index"
          virtual-scroll :rows-per-page-options="[0]"
        />
      </div>
    </div>
    <div class="row">
      <div class="col-12">
        <bar-chart :chart-data="this.percentCounts"
        :chart-height="this.barChartHeight"/>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from 'vue'
import BarChart from "components/BarChart.vue"
import PieChart from "components/PieChart.vue"
import gamelist from 'assets/gamelist.json'

export default defineComponent({
  name: 'PublisherPage',
  components: {
    BarChart,
    PieChart
  },
  data() {
    return {
      // pubSelector: ref('All publishers'),
      // statusFilter: ref('all'),
      pubSelector: 'All publishers',
      statusFilter: 'all',
      rawList: [],
      dateBuckets: {},
      publisherList: [],
      tableData: [],
      percentCounts: [],
      barChartHeight: 'height: 200px;',
      columns: [
        // { name: 'title', required: true, label: 'Title', align: 'left', field: rawList => rawList.title, format: val => `${val}` },
        { name: 'title', required: true, label: 'Title', align: 'left', field: 'title' },
        // { name: 'developer', label: 'Developer', field: 'developer', sortable: true },
        { name: 'publisher', label: 'Publisher', field: 'publisher', sortable: true },
        { name: 'released', label: 'Release Date', field: 'released', sortable: true },
        { name: 'removed', label: 'Removal Date', field: 'removed', sortable: true },
        // { name: 'lastupdate', label: 'Last Update', field: 'lastupdate', sortable: true },
        // { name: 'notes', label: 'Notes', field: 'notes'}
        // { name: 'iron', label: 'Iron (%)', field: 'iron', sortable: true, sort: (a, b) => parseInt(a, 10) - parseInt(b, 10) }
      ],
    }
  },
  mounted() {
    this.loadData();
  },
  computed: {
    pubIndex: function () {return this.publisherList.indexOf(this.pubSelector)}
  },
  watch: {
    pubSelector: function(){
      this.makeBuckets();
      this.makeTableData();
    },
    statusFilter: function(){
      this.makeBuckets();
      this.makeTableData();
    }
  },
  
  /* TODO:
   add the option to choose between released and last_update for the start of date ranges
   display age/lifetime values in the table data
   move the data loading/processing from PublisherPage to on initial load
    move the bar chart location from PublisherPage to ?
   possibly adjust the BarChart/PieChart imports - https://vue-echarts.dev/#codegen
   Fix ./publishers only loading via the sidebar and not directly?

   "It can be difficult to determine the date of game updates.  For uniformity, 'last update' is based on the original release date of the last paid DLC that is not a soundtrack or artbook (and before the removal of Denovo, where relevant)"

   Unknown removal date so unlisted:
   Halo Wars 2, Rock Band VR
   */

  methods: {
    async loadData() {
      try {
        // Filters out any titles that are not yet released
        this.rawList = gamelist.games.filter((item) => item.released <= gamelist.lastupdate);
        this.tableData = this.rawList;

        this.publisherList = this.rawList.map(item => item.publisher).filter((value, index, self) => self.indexOf(value) === index).sort(Intl.Collator().compare);
        this.publisherList.unshift('All publishers');
        this.makeRemovalProportions()
        this.makeBuckets();
      } catch (err) {
        console.error('Failed to load game JSON data:', err);
      }
    },
    // Prepares the data used by the bar chart
    makeRemovalProportions() {
      for(let i in this.publisherList){
        this.percentCounts.push({"publisher": this.publisherList[i], "removed": 0, "remain": 0, "percent": 0})
      }
      for(let i in this.rawList){
        let loc = this.publisherList.indexOf(this.rawList[i].publisher)
        if(this.rawList[i].removed){
          ++this.percentCounts[0].removed;
          ++this.percentCounts[loc].removed;
        } else {
          ++this.percentCounts[0].remain;
          ++this.percentCounts[loc].remain;
        }
      }
      for(let i in this.percentCounts){
        this.percentCounts[i].percent = Math.floor(100 * this.percentCounts[i].removed / (this.percentCounts[i].removed + this.percentCounts[i].remain));
      }
      this.barChartHeight = ('height: ').concat((15 + this.percentCounts.length * 17).toString(), 'px;');
    },
    // Prepares the data used by the pie chart
    makeBuckets() {
      this.dateBuckets = {
        source: [
          ['bucket', 'number'],
          ['0-3', 0],
          ['3-6', 0],
          ['6-12', 0],
          ['12-24', 0],
          ['24+', 0],
        ]
      };

      for(let i in this.rawList){
        let end;
        let comparator;
        if(this.rawList[i].removed){
          end = new Date(this.rawList[i].removed);
          comparator = 'removed';
        } else {
          end = new Date(gamelist.lastupdate);
          comparator = 'present';
        }
        if(this.pubSelector == 'All publishers' || this.pubSelector == this.rawList[i].publisher){
          if(this.statusFilter == 'all' || this.statusFilter == comparator){
            let start = new Date(this.rawList[i].released);
            let monthNum = Math.ceil((end.getTime() - start.getTime()) / 86400000 / 30);

            if(monthNum <= 3){
              ++this.dateBuckets.source[1][1];
            }
            else if(monthNum <= 6){
              ++this.dateBuckets.source[2][1];
            }
            else if(monthNum <= 12){
              ++this.dateBuckets.source[3][1];
            }
            else if(monthNum <= 24){
              ++this.dateBuckets.source[4][1];
            }
            else if(monthNum > 24){
              ++this.dateBuckets.source[5][1];
            }
          }
        }
      };
      for(let n = 1; n <= 5; ++n){
        if(this.dateBuckets.source[n][1] == 0){
          this.dateBuckets.source[n][1] = null;
        }
      }

      // console.log(this.dateBuckets);
    },
    // Prepares the data used by q-table
    makeTableData(){
      if(this.pubSelector == 'All publishers' && this.statusFilter == 'all'){
        this.tableData = this.rawList;
      } else {
        this.tableData = [];
        let comparator;
        for(let i in this.rawList){
          comparator = this.rawList[i].removed ? 'removed' : 'present'
          if(this.pubSelector == 'All publishers' || this.pubSelector == this.rawList[i].publisher){
            if(this.statusFilter == 'all' || this.statusFilter == comparator){
              this.tableData.push(this.rawList[i]);
            }
          }
        }
      }
    }
  }
})
</script>

<style lang="sass">
.my-sticky-header-table
  /* height or max-height is important */
  height: 310px

  .q-table__top,
  .q-table__bottom,
  thead tr:first-child th
    /* bg color is important for th; just specify one */
    background-color: #00b4ff

  thead tr th
    position: sticky
    z-index: 1
  thead tr:first-child th
    top: 0

  /* this is when the loading indicator appears */
  &.q-table--loading thead tr:last-child th
    /* height of all previous header rows */
    top: 48px

  /* prevent scrolling behind sticky top row on focus */
  tbody
    /* height of all previous header rows */
    scroll-margin-top: 48px
</style>