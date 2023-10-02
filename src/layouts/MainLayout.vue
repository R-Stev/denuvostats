<template>
  <q-layout view="lHh Lpr lFf">
    <q-header>
      <q-toolbar>
        <q-toolbar-title>
          Quasar App
        </q-toolbar-title>
        <q-btn-toggle
        v-model="model"
        flat stretch
        toggle-color="yellow"
        :options="options" />
        <q-btn flat round @click="$q.dark.toggle()" :icon="$q.dark.isActive ? 'nights_stay' : 'wb_sunny'"/>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <publisher-page :displayed-section="this.model" />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from 'vue'
import PublisherPage from 'pages/PublisherPage.vue'

export default defineComponent({
  name: 'MainLayout',
  components: {
    PublisherPage
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
