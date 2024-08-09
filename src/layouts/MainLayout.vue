<template>
  <q-layout view="lHh Lpr lFf">
    <q-header>
      <q-toolbar>
        <q-toolbar-title class="headerTitle"><span>Denuvo Stats</span></q-toolbar-title>
        <q-btn-toggle
        v-model="model" class="headerBtns"
        flat
        toggle-color="yellow"
        :options="options" />
        <q-btn flat round @click="this.darkModeToggle()" :icon="$q.dark.isActive ? 'nights_stay' : 'wb_sunny'"/>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <publisher-page :displayed-section="this.model"
      :dark-mode="$q.dark.isActive" />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from 'vue'
import {useQuasar} from "quasar"
import PublisherPage from 'pages/PublisherPage.vue'

export default defineComponent({
  name: 'MainLayout',
  components: {
    PublisherPage
  },
  beforeMount () {
    const $q = useQuasar()
    if($q.localStorage.has('darkMode')){
      $q.dark.set(true);
    }
  },
  methods: {
    darkModeToggle() {
      this.$q.dark.toggle()
      if(this.$q.localStorage.has('darkMode')){
        this.$q.localStorage.remove('darkMode')
      } else {
        this.$q.localStorage.set('darkMode', true)
      }
    }
  },
  setup () {
    return {
      model: ref('publishers'),
      options: [
        { label: 'Details', value: 'publishers' },
        { label: 'Removal %', value: 'percentages' },
        { label: 'What is Denuvo?', value: 'description' }
      ]
    }
  }
})
</script>

<style>
@media only screen and (max-width: 720px) {
  header h1 {
    line-height: 4rem;
  }
  .headerTitle span {
    display: none;
  }
  .headerTitle::after{
    content: "DS";
  }
  .headerBtns button {
    padding-left: 4px;
    padding-right: 4px;
  }
}
</style>