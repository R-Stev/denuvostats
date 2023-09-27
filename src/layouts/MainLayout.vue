<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-toolbar-title>
          Quasar App
        </q-toolbar-title>
        <q-btn flat round @click="$q.dark.toggle()" :icon="$q.dark.isActive ? 'nights_stay' : 'wb_sunny'"/>

        <div>Quasar v{{ $q.version }}</div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <publisher-page />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent } from 'vue'
import PublisherPage from 'pages/PublisherPage.vue'

export default defineComponent({
  name: 'MainLayout',
  components: {
    PublisherPage
  },
  setup () {
    return {}
  }
})
</script>
