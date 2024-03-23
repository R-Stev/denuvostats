<template>
  <q-page class="q-my-md" id="mainPage">
    <div v-if="displayedSection == 'publishers'">
      <header><h1>Publisher Details</h1></header>
      <q-card class="q-px-md q-pb-sm">
        <div class="row q-py-xs" id="publisherControls">
          <div>
            <q-select v-model="pubSelector" style="width: 245px"
              id="pubSelector"
              :options="publisherList"
              label="Publisher" />
          </div>
          <q-space />
          <div class="mobileButtons">
            <div style="width: 125px" v-show="releaseOrUpdate == 'release'">
              <q-btn class="full-width" label="Release" color="primary" outline
              @click="releaseOrUpdate = 'lastupdate'" />
            </div>
            <div style="width: 125px" v-show="releaseOrUpdate == 'lastupdate'">
              <q-btn class="full-width" label="Last update" color="primary" outline
              @click="releaseOrUpdate = 'release'" />
            </div>
            <div style="width: 125px" v-show="statusFilter == 'all'">
              <q-btn class="full-width" label="All" color="primary" outline 
              @click="statusFilter = 'present'"/>
            </div>
            <div style="width: 125px" v-show="statusFilter == 'present'">
              <q-btn class="full-width" label="Present" color="primary" outline 
              @click="statusFilter = 'removed'"/>
            </div>
            <div style="width: 125px" v-show="statusFilter == 'removed'">
              <q-btn class="full-width" label="Removed" color="primary" outline 
              @click="statusFilter = 'all'"/>
            </div>
          </div>
          <div class="desktopButtons">
            <div class="row">
              <div style="width: 100px; display: grid; align-items: center">Age since:</div>
              <div style="width: 250px">
                <q-btn-toggle
                  spread
                  v-model="releaseOrUpdate"
                  outline
                  toggle-color="primary"
                  :options="[
                    {label: 'Release', value: 'release'},
                    {label: 'Last update', value: 'lastupdate'}
                  ]"
                />
              </div>
            </div>
            <div class="row">
              <div style="width: 100px; display: grid; align-items: center">Denuvo status:</div>
              <div style="width: 250px">
                <q-btn-toggle
                  spread
                  v-model="statusFilter"
                  outline
                  toggle-color="primary"
                  :options="[
                    {label: 'All', value: 'all'},
                    {label: 'Present', value: 'present'},
                    {label: 'Removed', value: 'removed'}
                  ]"
                />
              </div>
            </div>
          </div>
        </div>
        <div>Last update: <b>{{ updateDate }}</b></div>
        <div class="row q-mb-sm">
          <div style="width: 100%; max-width: 600px; min-height: 270px; margin: auto">
            <pie-chart :chart-data="[this.dateBuckets]"
            :publisher-name="this.pubSelector"
            :denuvo-status="this.statusFilter" 
            :dark-mode="this.darkMode"/>
          </div>
          <div class="col-12">
            <!-- height should be some value of 48n + 104 -->
            <q-table class="my-sticky-header-table no-box-shadow"
              title="Details"
              style="height: 488px"
              :rows="tableData"
              :columns="columns"
              row-key="index"
              :visible-columns="visibleColumns"
              virtual-scroll
              :rows-per-page-options="[0]"
            />
          </div>
        </div>
        <q-separator />
        <div>
          <h2>Notes</h2>
          <p>It can be difficult to determine the date of game updates.  For uniformity, 'last update' is based on the original release date of the last paid DLC that is not a soundtrack or artbook (and before the removal of Denovo, where relevant).</p>
          <p>The following games have allegedly had Denuvo Anti-Tamper removed, but the removal date is unknown.  As a result they are not included above.</p>
          <ul>
            <li>Halo Wars 2, published by Microsoft Studios and THQ Nordic</li>
            <li>Rock Band VR, published by Oculus Studios</li>
          </ul>
        </div>
      </q-card>
    </div>

    <div v-if="displayedSection == 'percentages'">
      <header><h1>Removal Percentages</h1></header>
      <div style="width: 100%; max-width: 850px; margin: auto">
        <bar-chart :chart-data="this.percentCounts"
        :chart-height="this.barChartHeight"
        :dark-mode="this.darkMode"/>
      </div>
    </div>

    <div v-if="displayedSection == 'description'">
      <header><h1>What is Denuvo?</h1></header>
      <q-card>
        <q-card-section horizontal>
          <q-card-section class="q-pt-xs">
            <q-card-section class="flex flex-center mobileButtons">
              <q-img
                class="rounded-borders"
                width="50%" :ratio="8/5"
                src="~assets/Denuvo_Logo_2021.svg"
              />
            </q-card-section>
            <p>Denuvo Anti-Tamper is an anti-tamper technology and digital rights management (DRM) system developed by Austrian software company Denuvo Software Solutions GmbH, a subsidiary of Irdeto; itself a subsidiary of the MultiChoice group. It is generally what any mention of Denuvo is referring to, although there also exists Denuvo Anti-Cheat and Denuvo SecureDLC products.</p>
            <p>Games protected by Denuvo Anti-Tamper require periodic online activation which, while not the same as needing a persistent online connection and having an overall negligible impact, can and has prevented play due to server outages.  There is also the potential of performance impacts due to the obfuscation techniques used to deter piracy.</p>
            <q-separator />
            <h2>See also</h2>
            <ul>
              <li><a href="https://irdeto.com/denuvo/">Official website</a></li>
              <li><a href="https://www.pcgamingwiki.com/wiki/Denuvo">PCGamingWiki</a></li>
              <li><a href="https://en.wikipedia.org/wiki/Denuvo">Wikipedia</a></li>
            </ul>
          </q-card-section>
          <q-card-section class="col-5 flex flex-center desktopButtons">
            <q-img
              class="rounded-borders"
              width="100%" :ratio="8/5"
              src="~assets/Denuvo_Logo_2021.svg"
            />
          </q-card-section>
        </q-card-section>
      </q-card>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, defineAsyncComponent } from 'vue'
// import BarChart from "components/BarChart.vue"
// import PieChart from "components/PieChart.vue"
import gamelist from 'assets/gamelist.json'

export default defineComponent({
  name: 'PublisherPage',
  components: {
    BarChart: defineAsyncComponent(() => import('components/BarChart.vue')),
    PieChart: defineAsyncComponent(() => import('components/PieChart.vue'))
  },
  props: {
    displayedSection: String,
    darkMode: Boolean
  },
  data() {
    return {
      pubSelector: 'All publishers',
      releaseOrUpdate: 'release',
      statusFilter: 'all',
      updateDate: '',
      rawList: [],
      dateBuckets: {},
      publisherList: [],
      tableData: [],
      percentCounts: [],
      barChartHeight: 'height: 200px;',
      columns: [
        { name: 'title', required: true, label: 'Title', align: 'left', field: 'title' },
        { name: 'developer', label: 'Developer', field: 'developer', sortable: true },
        { name: 'publisher', label: 'Publisher', field: 'publisher', sortable: true },
        { name: 'released', label: 'Release Date', field: 'released', sortable: true, format: (val, row) => this.makeLocaleString(val) },
        { name: 'last_update', label: 'Last Update', field: 'last_update', sortable: true, format: (val, row) => this.makeLocaleString(val) },
        { name: 'removed', label: 'Removal Date', field: 'removed', sortable: true, format: (val, row) => this.makeLocaleString(val) },
        { name: 'release_age', label: 'Months', field: 'release_age', sortable: true },
        { name: 'update_age', label: 'Months', field: 'update_age', sortable: true },
        { name: 'notes', label: 'Notes', field: 'notes'}
      ],
      visibleColumns: ['title', 'publisher', 'released', 'removed', 'release_age']
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
    releaseOrUpdate: function(val){
      this.makeBuckets();
      this.visibleColumns = val == 'release' ? ['title', 'publisher', 'released', 'removed', 'release_age']
                                             : ['title', 'publisher', 'released', 'last_update', 'removed', 'update_age']
    },
    statusFilter: function(){
      this.makeBuckets();
      this.makeTableData();
    }
  },
  
  /* TODO:
   possibly adjust the BarChart/PieChart imports - https://vue-echarts.dev/#codegen
   */

  methods: {
    async loadData() {
      try {
        this.updateDate = new Date(gamelist.lastupdate).toLocaleDateString();
        // Filters out any titles that are not yet released
        this.rawList = gamelist.games.filter((item) => item.released <= gamelist.lastupdate);
        for(let i in this.rawList){
          this.rawList[i].released = this.makeEpoch(this.rawList[i].released);
          this.rawList[i].removed = this.makeEpoch(this.rawList[i].removed);
          this.rawList[i].last_update = this.makeEpoch(this.rawList[i].last_update);
        }
        this.tableData = this.rawList;

        this.publisherList = this.rawList.map(item => item.publisher).filter((value, index, self) => self.indexOf(value) === index).sort(Intl.Collator().compare);
        this.publisherList.unshift('All publishers');
        this.makeRemovalProportions()
        this.makeBuckets();
      } catch (err) {
        console.error('Failed to load game JSON data:', err);
      }
    },
    makeLocaleString(epoch){
      return epoch ? new Date(epoch).toLocaleDateString() : null;
    },
    makeEpoch(dateStr){
      return dateStr ?  new Date(dateStr).getTime() : null;
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
          ['0-2', 0],
          ['3-5', 0],
          ['6-11', 0],
          ['12-23', 0],
          ['24+', 0],
        ]
      };

      for(let i in this.rawList){
        let end;
        let comparator = this.rawList[i].removed ? 'removed' : 'present';
        if(this.pubSelector == 'All publishers' || this.pubSelector == this.rawList[i].publisher){
          if(this.statusFilter == 'all' || this.statusFilter == comparator){
            // let start;
            // if(this.releaseOrUpdate == 'lastupdate' && this.rawList[i].last_update){
            //   start = this.rawList[i].last_update;
            // } else {
            //   start = this.rawList[i].released;
            // }
            // console.log(`start: ${start}, end: ${end}, ${typeof start}`)
            // let monthNum = Math.ceil((end - start) / 86400000 / 30);
            let monthNum = (this.releaseOrUpdate == 'lastupdate') ? this.rawList[i].update_age : this.rawList[i].release_age;

            if(monthNum < 3){
              ++this.dateBuckets.source[1][1];
            }
            else if(monthNum < 6){
              ++this.dateBuckets.source[2][1];
            }
            else if(monthNum < 12){
              ++this.dateBuckets.source[3][1];
            }
            else if(monthNum < 24){
              ++this.dateBuckets.source[4][1];
            }
            else if(monthNum >= 24){
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
header h1
  font-size: 2rem
  margin: 0

h2
  font-size: 1.25rem
  line-height: 1.25rem
  margin: 12px 0 0 0

p
  margin: 6px 0

#publisherControls
  border-bottom: 1px solid #dddddd

.my-sticky-header-table
  /* height or max-height is important */
  height: 310px

  .q-table__top,
  .q-table__bottom,
  thead tr:first-child th
    /* bg color is important for th; just specify one */
    background-color: $primary

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

<style>
  @media only screen and (min-width: 721px) {
    #mainPage {
      margin-left: min(calc(10vw - 48px), 48px);
      margin-right: min(calc(10vw - 48px), 48px);
    }
    .mobileButtons {
      display: none;
    }
  }
  @media only screen and (max-width: 720px) {
    #mainPage {
      margin-left: 0;
      margin-right: 0;
    }
    .desktopButtons {
      display: none;
    }
  }

</style>