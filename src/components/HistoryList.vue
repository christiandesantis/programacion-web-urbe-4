<template>
  <q-spinner
    v-if="items.length === 0"
    class="block q-ml-auto q-mr-auto q-mb-xl q-mt-xl"
    color="primary"
    size="3em"
    :thickness="8"
    />
  <q-list v-else bordered separator>
    <q-item clickable v-ripple v-for="(item, i) in paginator.data" :key="i">
      <q-item-section>
        <q-item-label>{{ formatCityName(item.shaft) }} -  <b>{{ item.psi }} PSI</b></q-item-label>
        <q-item-label caption>{{ new Date(item.datetime) }}</q-item-label>
      </q-item-section>
    </q-item>
  </q-list>

  <div v-if="items.length > 0" class="q-pa-lg flex flex-center">
    <q-pagination
      v-model="currentPage"
      :max="paginator.totalPages > 5 ? 5 : paginator.totalPages"
      direction-links
      gutter="sm"
    />
  </div>
</template>

<script setup lang="ts">
import { collection, getDocs } from 'firebase/firestore'
import { db } from 'src/composables/firebase'
import { computed, onMounted, ref } from 'vue'

const collectionsReference = [
  'puerto-ordaz',
  'barcelona',
  'maracaibo',
  'cabimas'
]

const items = ref<{ shaft: string, psi: string, datetime: string }[]>([])

const itemsSorted = computed(() => {
  if (items.value.length > 0) {
    // eslint-disable-next-line vue/no-side-effects-in-computed-properties
    return items.value.sort((a, b) => Number(new Date(b.datetime)) - Number(new Date(a.datetime)))
  } else {
    return items.value
  }
})

async function fetchItems () {
  items.value = []
  collectionsReference.forEach(async (element) => {
    const collectionsRef = collection(db, `${element}`)
    const docsSnap = await getDocs(collectionsRef)
    if (docsSnap) {
      docsSnap.forEach(doc => {
        const data = doc.data()
        items.value.push({ shaft: element, psi: data?.psi, datetime: data?.datetime })
        // console.log(doc.data())
      })
    }
  })
}

function formatCityName (name: string): string {
  let res = ''
  switch (name) {
    case 'puerto-ordaz':
      res = 'Puerto Ordaz'
      break
    default:
      res = name.charAt(0).toUpperCase() + name.slice(1)
      break
  }
  return res
}

const currentPage = ref(1)
const itemsPerPage = ref(6)

const paginator = computed(() => {
  const page = currentPage.value || 1
  const perPage = itemsPerPage.value || 10
  const offset = (page - 1) * perPage

  const paginatedItems = itemsSorted.value.slice(offset).slice(0, perPage)
  const totalPages = Math.ceil(itemsSorted.value.length / perPage)
  return {
    page,
    perPage,
    pre_page: page - 1 ? page - 1 : null,
    next_page: (totalPages > page) ? page + 1 : null,
    total: itemsSorted.value.length,
    totalPages,
    data: paginatedItems
  }
})

onMounted(async () => {
  await fetchItems()
})
</script>
