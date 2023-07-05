<template>
  <div class="chart-container" :style="containerVars">
    <div class="months-row">
      <div class="empty-cell"></div>
      <div class="month-names">
        <div class="month" v-for="(month, index) in availableMonths" :key="index">
          {{ month }}
        </div>
      </div>
    </div>
    <div class="days-row">
      <div class="day-names">
        <div class="weekday" v-for="weekday in availableDays" :key="weekday">
          {{ weekday }}
        </div>
      </div>
      <div class="days">
        <div class="day" v-for="day in dataDays" :key="day" :title="day.title" :class="day.class">
        </div>
      </div>
    </div>
    <div class="legend-row">
      <span class="label">{{ props.minLabel }}</span>
      <div class="swatch bg-1"></div>
      <div class="swatch bg-2"></div>
      <div class="swatch bg-3"></div>
      <div class="swatch bg-4"></div>
      <div class="swatch bg-5"></div>
      <div class="swatch bg-6"></div>
      <div class="swatch bg-7"></div>
      <div class="swatch bg-8"></div>
      <div class="swatch bg-9"></div>
      <div class="swatch bg-10"></div>
      <span class="label"> {{ props.maxLabel }}</span>
    </div>
  </div>
</template>

<script setup>
import { watchEffect } from 'vue';
import { reactive } from 'vue';
import { computed } from 'vue'

const props = defineProps({
  startDate: {
    type: String,
    required: false,
    default() {
      const currentDate = new Date()
      const currentYear = currentDate.getFullYear()
      const lastYear = currentYear - 1
      const lastYearToday = new Date(lastYear, currentDate.getMonth(), currentDate.getDate())
      const lastYearTodayMonday = new Date(lastYearToday.setDate(lastYearToday.getDate() - ((lastYearToday.getDay() + 6) % 7)))

      return `${lastYearTodayMonday.getFullYear()}-${(lastYearTodayMonday.getMonth() + 1).toString().padStart(2, '0')}-${lastYearTodayMonday.getDate().toString().padStart(2, '0')}`
    }
  },
  endDate: {
    type: String,
    required: false,
    default() {
      const date = new Date()
      return `${date.getFullYear()}-${(date.getMonth() + 1).toString().padStart(2, '0')}-${date.getDate().toString().padStart(2, '0')}`
    }
  },
  locale: {
    type: String,
    required: false,
    default: 'en-US'
  },
  noOfWeekDays: {
    type: Number,
    required: false,
    default: 7,
    validator: (value) => {
      return value >= 1 && value <= 7
    }
  },
  cssOptions: {
    type: Object,
    required: false
  },
  weekends: {
    type: Boolean,
    required: false,
    default: true
  },
  weekDays: {
    type: Array,
    required: false,
    default: () => {
      return []
    }
  },
  data: {
    type: Array,
    required: false,
    default: () => {
      return []
    }
  },
  tooltip: {
    type: Function,
    required: false,
    default: (date, locale, work) => {
      return `${date.toLocaleDateString(locale)} - ${work}`
    }
  },
  colors: {
    type: Array,
    required: false,
    default() {
      return [
        '#e3f2fd',
        '#bbdefb',
        '#90caf9',
        '#64b5f6',
        '#42a5f5',
        '#2196f3',
        '#1e88e5',
        '#1976d2',
        '#1565c0',
        '#0d47a1'
      ]
    }
  },
  minLabel: {
    type: String,
    required: false,
    default: 'Less'
  },
  maxLabel: {
    type: String,
    required: false,
    default: 'More'
  }
})

const sDate = computed(() => {
  const startDate = new Date(new Date(props.startDate).setHours(0, 0, 0, 0))

  const startDateMonday = startDate.toLocaleDateString(props.locale, { weekday: 'short' }) === 'Mon'
    ? startDate
    : startDate.setDate(startDate.getDate() - ((startDate.getDay() + 6) % 7))

  return new Date(startDateMonday)
})
const eDate = computed(() => {
  const end = new Date(new Date(props.endDate).setHours(23, 59, 59, 999))
  const nextSunday = end.toLocaleDateString(props.locale, { weekday: 'short' }) === 'Sun'
    ? end
    : end.setDate(end.getDate() + (7 - end.getDay()))

  return new Date(nextSunday)
})

const noOfDays = computed(() => {
  const diffTime = Math.abs(eDate.value - sDate.value)
  const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))
  return 7 * Math.ceil(diffDays / 7)
})
const availableDays = computed(() => {
  if (props.weekDays.length) return props.weekDays
  const days = []

  const today = new Date()
  const day = today.getDay()
  const firstDayOfWeek = new Date(today.setDate(today.getDate() - day + 1))

  const daysInWeek = props.weekends ? 7 : 5

  for (let i = 0; i < daysInWeek; i += Math.ceil(daysInWeek / props.noOfWeekDays)) {
    const date = new Date(firstDayOfWeek)
    date.setDate(date.getDate() + i)
    days.push(date.toLocaleDateString(props.locale, { weekday: 'short' }))
  }

  return days
})
const dataDaysTemplate = computed(() => {
  const days = []
  let max
  let startIndex
  let endIndex
  let intensityValues
  let pace

  if (props.data.length) {
    max = props.data.reduce((acc, curr) => {
      return Math.max(acc, curr.value)
    }, -Infinity)

    startIndex = Math.ceil(((new Date(new Date(props.startDate).setHours(0, 0, 0, 0))) - sDate.value) / (1000 * 60 * 60 * 24))
    endIndex = noOfDays.value - Math.floor((eDate.value - (new Date(new Date(props.endDate).setHours(23, 59, 59, 999)))) / (1000 * 60 * 60 * 24))

    intensityValues = 10
    pace = max / intensityValues
  }

  for (let i = 0; i < noOfDays.value; i++) {
    const date = new Date(sDate.value)
    date.setDate(date.getDate() + i)

    if (props.data.length) {
      if (i >= startIndex && i < endIndex) {
        const work = props.data[i - startIndex]?.value ?? 0
        const intensityInterval = Math.ceil(work / pace)

        days.push({
          title: props.tooltip(date, props.locale, work),
          date: date.toLocaleDateString(props.locale),
          work,
          class: '',
          className: `bg-${intensityInterval}`,
          inData: true
        })
      } else {
        days.push({
          title: props.tooltip(date, props.locale, 0),
          date: date.toLocaleDateString(props.locale),
          work: 0,
          class: '',
          className: 'filler',
          inData: false
        })
      }
    } else {
      days.push({
        title: props.tooltip(date, props.locale, 0),
        date: date.toLocaleDateString(props.locale),
        work: 0,
        class: '',
        className: 'no-data',
        inData: false
      })
    }
  }
  return days
})

const dataDays = reactive(dataDaysTemplate.value)
watchEffect(() => {
  dataDays.forEach(d => {
    setTimeout(() => {
      d.class = d.className
    }, Math.random() * 1000)
    return d
  })
})

const availableMonths = computed(() => {
  const startYear = sDate.value.getFullYear()
  const startDate = new Date(sDate.value)
  const endDate = startDate.setDate(startDate.getDate() + noOfDays.value)
  const endYear = new Date(endDate).getFullYear()

  const months = []
  for (let i = 0; i < noOfDays.value; i++) {
    const date = new Date(sDate.value)
    date.setDate(date.getDate() + i)
    let month
    if (startYear === endYear) {
      month = date.toLocaleString(props.locale, { month: 'short' })
    } else {
      month = date.toLocaleString(props.locale, { year: 'numeric', month: 'short' })
    }

    if (!months.includes(month)) {
      months.push(month)
    }
  }

  if ((new Date(props.startDate)).getMonth() !== sDate.value.getMonth()) {
    months.shift()
  }
  if ((new Date(props.endDate)).getMonth() !== eDate.value.getMonth()) {
    months.pop()
  }

  return months.map(m => m.split(' ')[0])
})
const noOfMonths = computed(() => availableMonths.value.length)

const noOfWeeks = computed(() => {
  return Math.ceil(noOfDays.value / 7)
})

const containerVars = computed(() => {
  return {
    '--number-of-months': noOfMonths.value,
    '--number-of-week-days': availableDays.value.length,
    '--number-of-weeks': noOfWeeks.value,
    '--font-size': getDefaultOrOveride('--font-size'),
    '--cell-size': getDefaultOrOveride('--cell-size'),
    '--weekdays-width': getDefaultOrOveride('--weekdays-width'),
    '--padding': getDefaultOrOveride('--padding'),
    '--gap': getDefaultOrOveride('--gap'),
    '--border-radius': getDefaultOrOveride('--border-radius'),
    '--bg-color': getDefaultOrOveride('--bg-color'),
    '--border-color': getDefaultOrOveride('--border-color'),
    '--border-width': getDefaultOrOveride('--border-width'),
    '--border-style': getDefaultOrOveride('--border-style'),
    '--bg-1': props.colors[0],
    '--bg-2': props.colors[1],
    '--bg-3': props.colors[2],
    '--bg-4': props.colors[3],
    '--bg-5': props.colors[4],
    '--bg-6': props.colors[5],
    '--bg-7': props.colors[6],
    '--bg-8': props.colors[7],
    '--bg-9': props.colors[8],
    '--bg-10': props.colors[9],
  }
})

function getDefaultOrOveride(value) {

  const defaults = {
    '--font-size': 'min(12px, 2cqmin)',
    '--cell-size': '1cqmin',
    '--weekdays-width': 'min(calc(1cqmin * 6), 40px)',
    '--padding': 'max(1cqmin, 2px)',
    '--gap': 'min(1cqmin, 2px)',
    '--border-radius': '2px',
    '--bg-color': 'transparent',
    '--border-color': 'transparent',
    '--border-width': '0',
    '--border-style': 'none',
  }
  return props.cssOptions?.[value] ?? defaults[value]
}

</script>

<style lang="stylus">
.chart-container
  border-radius: var(--border-radius)
  padding: var(--padding)
  font-size: var(--font-size)
  // max-width: 100%
  max-width: fit-content;
  display: grid
  grid-template-columns: 1fr
  grid-template-rows: auto 1fr auto
  gap: var(--gap)
  overflow: auto
  background-color: var(--bg-color)
  border: var(--border-width) var(--border-style) var(--border-color)

  .empty-cell
    width: var(--weekdays-width)

  .months-row, .days-row
    display: grid
    grid-template-columns: auto 1fr
    grid-template-rows: 1fr
    gap: var(--gap)

  .legend-row
    display: flex
    align-items: center
    justify-content: flex-end
    gap: var(--gap)

  .label
    margin: 0 0.5rem

  .swatch
    width: calc(var(--cell-size) / 2)
    aspect-ratio: 1
    border-radius: var(--border-radius)
    opacity: .5

  .month-names
    display: grid
    grid-template-columns: repeat(var(--number-of-months), 1fr)
    grid-template-rows: 1fr
    place-items: center
    gap: var(--gap)
    // padding: var(--padding) 0

  .month
    text-align: center

  .day-names
    display: grid
    grid-template-columns: var(--weekdays-width)
    grid-template-rows: repeat(var(--number-of-week-days), 1fr)
    place-items: center
    gap: var(--gap)
    // padding: 0 var(--padding)

  .days
    display: grid
    grid-template-columns: repeat(var(--number-of-weeks), 1fr)
    grid-template-rows: repeat(7, 1fr)
    grid-auto-flow: column
    place-items: center
    gap: var(--gap)

  .day
    // align-self: stretch
    // justify-self: stretch
    width: var(--cell-size)
    aspect-ratio: 1
    border-radius: var(--border-radius)
    background-color: transparent
    cursor: pointer
    transition: all 0.1s ease-in-out
    border: var(--border-width) var(--border-style) var(--border-color)
    opacity: .5
    z-index: 1

    &:hover
      scale: 2
      opacity: .95
      z-index: 10

    &.filler, &.no-data
      background-color: transparent
      border: none

  .bg-0
    background-color: transparent
    transition: background-color .0s ease-in-out
  .bg-1
    background-color: var(--bg-1)
    transition: background-color .1s ease-in-out
  .bg-2
    background-color: var(--bg-2)
    transition: background-color .2s ease-in-out
  .bg-3
    background-color: var(--bg-3)
    transition: background-color .3s ease-in-out
  .bg-4
    background-color: var(--bg-4)
    transition: background-color .4s ease-in-out
  .bg-5
    background-color: var(--bg-5)
    transition: background-color .5s ease-in-out
  .bg-6
    background-color: var(--bg-6)
    transition: background-color .6s ease-in-out
  .bg-7
    background-color: var(--bg-7)
    transition: background-color .7s ease-in-out
  .bg-8
    background-color: var(--bg-8)
    transition: background-color .8s ease-in-out
  .bg-9
    background-color: var(--bg-9)
    transition: background-color .9s ease-in-out
  .bg-10
    background-color: var(--bg-10)
    transition: background-color .10s ease-in-out
</style>
