<template>
  <q-page class="q-pa-md">
    <div class="row justify-around">
      <q-card class="col-md-8">
        <q-tabs
          v-model="tab"
          dense
          class="text-grey"
          active-color="primary"
          indicator-color="primary"
          align="justify"
          narrow-indicator
          >
          <q-tab name="add" icon="add" label="Agregar" />
          <q-tab name="history" icon="history" label="Histórico" />
          <q-tab name="statistics" icon="bar_chart" label="Estadísticas" />
        </q-tabs>

        <q-separator />

        <q-tab-panels v-model="tab" animated>
          <q-tab-panel name="add">
            <div class="text-h6 q-mb-lg">Agregar Registro</div>
            <AddForm />
          </q-tab-panel>

          <q-tab-panel name="history">
            <div class="text-h6 q-mb-lg">Histórico de Registros</div>
            <HistoryList />
          </q-tab-panel>

          <q-tab-panel name="statistics">
            <div class="text-h6 q-mb-lg">Estadísticas de los Registros</div>
            <StatisticsCharts />
          </q-tab-panel>
        </q-tab-panels>
      </q-card>
    </div>

  </q-page>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import AddForm from '../components/AddForm.vue'
import HistoryList from '../components/HistoryList.vue'
import StatisticsCharts from '../components/StatisticsCharts.vue'

const tab = ref('add')

</script>
