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
      <!-- <pie-chart></pie-chart> -->
      <div class="col-12">
        <pie-chart :chart-data="[this.dateBuckets]"
        :publisher-name="this.pubSelector"
        :publisher-index="this.pubIndex" />
      </div>
    </div>
    <div class="row">
      <div class="col-12">
        <!-- height should be some value of 48n + 104 -->
        <q-table class="my-sticky-header-table"
          title="Details" style="height: 488px"
          :rows="tableData" :columns="columns" row-key="index"
          virtual-scroll :rows-per-page-options="[0]"
        />
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from 'vue'
import PieChart from "components/PieChart.vue"
import gamelist from 'assets/gamelist.json'

export default defineComponent({
  name: 'PublisherPage',
  components: {PieChart},
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
  methods: {
    async loadData() {
      try {
        // const response = await fetch('src/assets/gamelist.json');
        // this.rawList = await response.json();
        this.rawList = gamelist.games;
        this.tableData = gamelist.games;

        this.publisherList = this.rawList.map(item => item.publisher).filter((value, index, self) => self.indexOf(value) === index).sort(Intl.Collator().compare);
        this.publisherList.unshift('All publishers');
        // console.log(this.publisherList);
        this.makeRemovalProportions()
        this.makeBuckets();
      } catch (err) {
        console.error('Failed to load game JSON data:', err);
      }
    },
    makeRemovalProportions() {
      let counts = [];
      for(let i in this.publisherList){
        counts.push({"publisher": this.publisherList[i], "removed": 0, "remain": 0, "percent": 0})
      }
      for(let i in this.rawList){
        let loc = this.publisherList.indexOf(this.rawList[i].publisher)
        if(this.rawList[i].removed){
          ++counts[0].removed;
          ++counts[loc].removed;
        } else {
          ++counts[0].remain;
          ++counts[loc].remain;
        }
      }
      for(let i in counts){
        counts[i].percent = Math.floor(100 * counts[i].removed / (counts[i].removed + counts[i].remain))
      }
      // console.log(counts);
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
      // console.log(this.dateBuckets);
    },
    // Prepares the data used by q-table
    makeTableData(){
      // console.log(this.pubSelector);
      if(this.pubSelector == 'All publishers' && this.statusFilter == 'all'){
        this.tableData = this.rawList;
      } else {
        this.tableData = [];
        let comparator;
        for(let i in this.rawList){
          if(this.rawList[i].removed){
            comparator = 'removed';
          } else {
            comparator = 'present';
          }
          if(this.pubSelector == 'All publishers' || this.pubSelector == this.rawList[i].publisher){
            if(this.statusFilter == 'all' || this.statusFilter == comparator){
              this.tableData.push(this.rawList[i]);
            }
          }
        }
      }
      // console.log(this.tableData);
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