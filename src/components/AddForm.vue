<template>
  <q-form
    @submit.prevent="onSubmit"
    @reset="onReset"
    class="q-gutter-md q-ml-auto q-mr-auto"
    :class="{'q-pa-md': $q.screen.lt.md, 'q-mt-md': $q.screen.gt.sm}"
    >

    <q-select
      filled
      v-model="data.shaft"
      :options="shaftOptions"
      label="Pozo"
      hint="Seleccionar pozo"
      />

    <q-input
      filled
      v-model="data.psi"
      type="number"
      label="PSI"
      hint="PSI regisrados en el pozo"
      lazy-rules
      :rules="[ ((val => val && val > 0) || formSubmitted) || 'El valor en PSI debe ser mayor a 0']"
    />

    <div class="row justify-around">
      <div class="col-md-6">
        <div class="text-h6 text-center">Fecha</div>
        <q-date
          v-model="data.date"
          class="block q-ml-auto q-mr-auto"
        />
      </div>
      <div class="col-md-6">
        <div class="text-h6 text-center">Hora</div>
        <q-time v-model="data.time" class="block q-ml-auto q-mr-auto" />
      </div>
    </div>

    <div class="q-mt-lg q-mb-md q-ml-none">
      <q-btn label="Registrar" type="submit" color="primary" class="q-ml-auto q-mr-auto block q-pt-sm"/>
      <!-- <q-btn label="Reset" type="reset" color="primary" flat class="q-ml-sm" /> -->
    </div>
  </q-form>
</template>

<script setup lang="ts">
import { onMounted, reactive, ref } from 'vue'
import { collection, doc, setDoc } from 'firebase/firestore'
import { db } from 'src/composables/firebase'
import { useQuasar } from 'quasar'

const $q = useQuasar()

const currentDateTime = new Date()
const year = currentDateTime.getFullYear()
const month = currentDateTime.getMonth() + 1
const day = currentDateTime.getDate()
const time = `${currentDateTime.getHours()}:${currentDateTime.getMinutes()}`

const formSubmitted = ref(false)

const data = reactive({
  shaft: '',
  psi: 0,
  date: null as unknown as string,
  time: null as unknown as string
})

const shaftOptions = ref([
  'Puerto Ordaz',
  'Barcelona',
  'Maracaibo',
  'Cabimas'
])

onMounted(() => {
  data.shaft = shaftOptions.value[0]
  setDefaultValues()
})

async function onSubmit () {
  // console.log('onSubmit')
  await createRegistry()
}

function onReset () {
  console.log('onReset')
}

async function createRegistry () {
  const collectionsRef = collection(db, `${data.shaft.toLowerCase().replace(' ', '-')}`)
  await setDoc(doc(collectionsRef), {
    psi: data.psi,
    datetime: `${data.date} ${data.time}`
  })
  $q.notify({
    color: 'green-4',
    textColor: 'white',
    icon: 'cloud_done',
    message: 'Registro agregado!'
  })

  setDefaultValues()

  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  })
}

async function setDefaultValues () {
  formSubmitted.value = true
  data.psi = 0
  data.date = `${year}/${month < 10 ? `0${month}` : month}/${day < 10 ? `0${day}` : day}`
  data.time = `${Number(time.split(':')[0]) < 10 ? `0${time.split(':')[0]}` : time.split(':')[0]}:${Number(time.split(':')[1]) < 10 ? `0${time.split(':')[1]}` : time.split(':')[1]}`
  formSubmitted.value = false
}

</script>
