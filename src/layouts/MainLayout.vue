<template>
  <q-layout view="lHh Lpr lFf">
    <q-header>
      <q-toolbar>
        <q-toolbar-title>
          Denuvo Stats
        </q-toolbar-title>
        <q-btn-toggle
        v-model="model"
        flat stretch
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
        { label: 'Publisher Details', value: 'publishers' },
        { label: 'Removal Percentages', value: 'percentages' },
        { label: 'What is Denuvo?', value: 'description' }
      ]
    }
  }
})
</script>
