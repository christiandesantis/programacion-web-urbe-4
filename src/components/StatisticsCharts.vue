<template>
  <div>
    <div class="text-h6 text-center q-mb-md">Seleccionar pozo</div>
    <q-select
      class="mw-400 q-ml-auto q-mr-auto"
      filled
      v-model="shaft"
      :options="shaftOptions"
      label="Pozo"
      hint="Seleccionar pozo"
      />
  </div>

  <div class="q-mt-xl">
    <div class="text-h6 text-center q-mb-md">Seleccionar Rango de fecha</div>
    <q-date v-model="dateRange" class="block q-ml-auto q-mr-auto" minimal range />
    <div class="q-mt-md q-ml-none">
      <q-btn label="Generar Gráfica" @click="fetchItems()" color="primary" class="q-ml-auto q-mr-auto block q-pt-sm"/>
    </div>
  </div>

  <div v-if="dynamicOptions.xaxis.categories.length > 0 && dynamicSeries[0].data.length > 0" class="q-mt-xl bg-grey-2 q-pa-md" style="border-radius: 10px;">
    <div class="text-h6 text-center">Gráfica</div>
    <apexchart width="600" type="bar" :options="dynamicOptions" :series="dynamicSeries"></apexchart>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, reactive, ref } from 'vue'
import { collection, getDocs } from 'firebase/firestore'
import { db } from 'src/composables/firebase'

const currentDateTime = new Date()
const year = currentDateTime.getFullYear()
const month = currentDateTime.getMonth() + 1
const day = currentDateTime.getDate()

const dateRange = ref({ from: `${year}/${month < 10 ? `0${month}` : month}/01`, to: `${year}/${month < 10 ? `0${month}` : month}/${day < 10 ? `0${day}` : day}` })

const options = reactive({
  chart: {
    id: 'vuechart-statistics'
  },
  xaxis: {
    categories: ['Sep 2022', 'Oct 2022', 'Nov 2022', 'Dic 2022', 'Ene 2023', 'Feb 2023']
  }
})

const series = ref([{
  name: 'Promedio',
  data: [45, 50, 49, 60, 70, 91]
}])

const shaft = ref('')

const shaftOptions = ref([
  'Puerto Ordaz',
  'Barcelona',
  'Maracaibo',
  'Cabimas'
])

const collectionsReference = [
  'puerto-ordaz',
  'barcelona',
  'maracaibo',
  'cabimas'
]

const items = ref<{ psi: number, datetime: string }[]>([])

const itemsSorted = computed(() => {
  if (items.value.length > 0) {
    // eslint-disable-next-line vue/no-side-effects-in-computed-properties
    return items.value.sort((a, b) => Number(new Date(a.datetime)) - Number(new Date(b.datetime)))
  } else {
    return []
  }
})

const dynamicOptions = computed(() => {
  const newCategories: string[] = []
  if (items.value.length > 0) {
    itemsSorted.value.forEach(item => {
      const cat = generateCategory(item.datetime)
      if (
        newCategories.find(el => el === cat) === null ||
        newCategories.find(el => el === cat) === undefined
      ) {
        console.log(cat)
        if (new Date(item.datetime) >= new Date(dateRange.value.from) && new Date(item.datetime) <= new Date(dateRange.value.to)) {
          newCategories.push(cat)
        }
      }
    })

    return {
      chart: {
        id: 'vuechart-statistics'
      },
      xaxis: {
        categories: newCategories
      }
    }
  } else {
    return {
      chart: {
        id: 'vuechart-statistics'
      },
      xaxis: {
        categories: []
      }
    }
  }
})

const dynamicSeries = computed(() => {
  if (dynamicOptions.value.xaxis.categories.length > 0) {
    const data: number[] = []
    dynamicOptions.value.xaxis.categories.forEach(element => {
      data.push(getMonthAverage(element))
    })
    return [{
      name: 'Promedio',
      data
    }]
  } else {
    return [{
      name: 'Promedio',
      data: []
    }]
  }
})

async function fetchItems () {
  items.value = []
  const collectionsRef = collection(db, `${shaft.value.toLowerCase().replace(' ', '-')}`)
  const docsSnap = await getDocs(collectionsRef)
  if (docsSnap) {
    docsSnap.forEach(doc => {
      const data = doc.data()
      items.value.push({ psi: data?.psi, datetime: data?.datetime })
      // console.log(doc.data())
    })
  }
}

function generateCategory (datetime: string) {
  const date = new Date(datetime)
  const year = date.getFullYear()
  const month = date.getMonth() + 1
  let monthString = ''

  switch (month) {
    case 1:
      monthString = 'ENE'
      break
    case 2:
      monthString = 'FEB'
      break
    case 3:
      monthString = 'MAR'
      break
    case 4:
      monthString = 'ABR'
      break
    case 5:
      monthString = 'MAY'
      break
    case 6:
      monthString = 'JUN'
      break
    case 7:
      monthString = 'JUL'
      break
    case 8:
      monthString = 'AGO'
      break
    case 9:
      monthString = 'SEP'
      break
    case 10:
      monthString = 'OCT'
      break
    case 11:
      monthString = 'NOV'
      break
    case 12:
      monthString = 'DIC'
      break

    default:
      monthString = month < 10 ? `0${month} / ` : `${month} / `
      break
  }
  return `${monthString} ${year}`
}

function getMonthAverage (monthString: string): number {
  const yearValue = Number(monthString.split(' ')[1])
  let monthValue: number = null as unknown as number
  switch (monthString.split(' ')[0]) {
    case 'ENE':
      monthValue = 1
      break
    case 'FEB':
      monthValue = 2
      break
    case 'MAR':
      monthValue = 3
      break
    case 'ABR':
      monthValue = 4
      break
    case 'MAY':
      monthValue = 5
      break
    case 'JUN':
      monthValue = 6
      break
    case 'JUL':
      monthValue = 7
      break
    case 'AGO':
      monthValue = 8
      break
    case 'SEP':
      monthValue = 9
      break
    case 'OCT':
      monthValue = 10
      break
    case 'NOV':
      monthValue = 11
      break
    case 'DIC':
      monthValue = 12
      break
  }
  let sum = 0
  let count = 0
  itemsSorted.value.forEach(element => {
    if (
      (new Date(element.datetime).getMonth() + 1) === monthValue &&
      (new Date(element.datetime).getFullYear()) === yearValue
    ) {
      sum += Number(element.psi)
      console.log('sum', sum)
      count += 1
      console.log('count', count)
    }
  })
  return (sum / count)
}

function orderCategoriesByYear (categories: string[]): string[] {
  return categories.sort((a, b) => Number(b.split(' ')[1]) - Number(a.split(' ')[1]))
}

onMounted(async () => {
  shaft.value = shaftOptions.value[0]
  // await fetchItems()
})

</script>

<style>
  .apexcharts-canvas {
    display: block!important;
    margin-left: auto!important;
    margin-right: auto!important;
  }
  .mw-400 {
    max-width: 400px;
  }
</style>
