<template>
  <v-container>
    <v-row>
      <v-col cols="12" class="d-flex justify-center">
        <ActivityChart :no-of-week-days="3" locale="ro-RO" :data="data1" :css-options="options1" :colors="colors1" />
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" class="d-flex justify-center">
        <ActivityChart start-date="2022-06-01" end-date="2023-05-31" :week-days="['L', 'M', 'V']" locale="ro-RO"
          :css-options="options2" :data="data2" :colors="colors2" />
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" class="d-flex justify-center">
        <ActivityChart start-date="2023-06-01" end-date="2023-06-30" :no-of-week-days="5" :weekends="false" locale="ro-RO"
          :css-options="options3" :data="data3" :colors="colors3" />
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" class="d-flex justify-center">
        <ActivityChart start-date="2023-02-01" end-date="2023-06-30" :no-of-week-days="5" :weekends="false" locale="ja-JA"
          :css-options="options4" :data="data4" :colors="colors4" />
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { useTheme } from 'vuetify'
import colors from 'vuetify/lib/util/colors'
import { ref, computed } from 'vue'
import ActivityChart from '@/components/ActivityChart.vue'

const theme = useTheme()

const data1 = ref([])
const data2 = ref([])
const data3 = ref([])
const data4 = ref([])

const options1 = computed(() => ({
  '--border-radius': '0',
  '--border-color': theme.global.current.value.dark ? colors.grey.darken2 : colors.grey.lighten2,
  '--border-width': '1px',
  '--border-style': 'solid'
}))
const options2 = computed(() => ({
  '--cell-size': '12px',
  '--font-size': '10px',
  '--bg-color': theme.global.current.value.dark ? colors.blueGrey.darken4 : colors.blueGrey.lighte5,
  '--border-color': theme.global.current.value.dark ? colors.blueGrey.darken1 : colors.blueGrey.lighten3,
  '--border-width': '3px',
  '--border-radius': '20px',
  '--border-style': 'solid'
}))
const options3 = computed(() => ({
  '--weekdays-width': '40px',
  '--font-size': '16px',
  '--cell-size': '24px',
  '--padding': '10px',
  '--gap': '14px 6px',
  '--border-radius': '20px',
  '--bg-color': theme.global.current.value.dark ? colors.brown.darken4 : colors.brown.lighten5,
  '--border-color': theme.global.current.value.dark ? colors.blueGrey.darken2 : colors.blueGrey.lighten2,
  '--border-width': '1px',
  '--border-style': 'solid'
}))
const options4 = computed(() => ({
  '--font-size': 'max(12px, max(1cqw, 1cqh))',
  '--cell-size': 'max(1cqw, 1cqh)',
}))

const colors1 = [
  colors.green.lighten5 + 'ff',
  colors.green.lighten4 + 'ff',
  colors.green.lighten3 + 'ff',
  colors.green.lighten2 + 'ff',
  colors.green.lighten1 + 'ff',
  colors.green.base + 'ff',
  colors.green.darken1 + 'ff',
  colors.green.darken2 + 'ff',
  colors.green.darken3 + 'ff',
  colors.green.darken4 + 'ff'
]
const colors2 = [
  colors.red.lighten5 + 'ff',
  colors.red.lighten4 + 'ff',
  colors.red.lighten3 + 'ff',
  colors.red.lighten2 + 'ff',
  colors.red.lighten1 + 'ff',
  colors.red.base + 'ff',
  colors.red.darken1 + 'ff',
  colors.red.darken2 + 'ff',
  colors.red.darken3 + 'ff',
  colors.red.darken4 + 'ff'
]
const colors3 = [
  colors.pink.lighten5 + 'ff',
  colors.pink.lighten4 + 'ff',
  colors.pink.lighten3 + 'ff',
  colors.pink.lighten2 + 'ff',
  colors.pink.lighten1 + 'ff',
  colors.pink.base + 'ff',
  colors.pink.darken1 + 'ff',
  colors.pink.darken2 + 'ff',
  colors.pink.darken3 + 'ff',
  colors.pink.darken4 + 'ff'
]
const colors4 = [
  colors.deepOrange.lighten5,
  colors.deepOrange.lighten4,
  colors.deepOrange.lighten3,
  colors.deepOrange.lighten2,
  colors.deepOrange.lighten1,
  colors.deepOrange.base,
  colors.deepOrange.darken1,
  colors.deepOrange.darken2,
  colors.deepOrange.darken3,
  colors.deepOrange.darken4
]

const randomData = (startDate, endDate) => {
  const data = []
  const date = new Date(startDate)
  const endDateObj = new Date(endDate)
  while (date <= endDateObj) {
    data.push({
      date: new Date(date),
      value: Math.floor(Math.random() * 100)
    })
    date.setDate(date.getDate() + 1)
  }
  return data
}

data1.value = randomData('2022-07-04T00:00:00.000', '2023-07-04T23:59:59.999')
data2.value = randomData('2022-06-01T00:00:00.000', '2023-05-31T23:59:59.999')
data3.value = randomData('2023-06-01T00:00:00.000', '2023-06-30T23:59:59.999')
data4.value = randomData('2023-01-01T00:00:00.000', '2023-06-30T23:59:59.999')
</script>
