<template>
  <div class="calendar">
    <div class="header">
      <div class="day-label">Day</div>
      <div v-for="h in 24" :key="h" class="hour-label">
        {{ h - 1 }}:00
      </div>
    </div>

    <div
      v-for="(dayKey, idx) in days"
      :key="dayKey"
      class="day-row"
    >
      <div
        class="day-label all-day"
        @click="toggleAllDay(dayKey)"
      >
        {{ labels[idx] }}
      </div>

      <div
        v-for="hour in 24"
        :key="hour"
        class="hour-cell"
        :class="{ selected: isSelected(dayKey, hour - 1) }"
        @mousedown="startDrag(dayKey, hour - 1)"
        @mouseenter="dragOver(dayKey, hour - 1)"
        @mouseup="endDrag"
        @click="toggleHour(dayKey, hour - 1)"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, computed } from 'vue'

const props = defineProps({
  modelValue: { type: Object, required: true }
})
const emit = defineEmits(['update:model-value'])

const days = ['mo', 'tu', 'we', 'th', 'fr', 'sa', 'su']
const labels = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']

const schedule = ref(JSON.parse(JSON.stringify(props.modelValue)))

watch(
  () => props.modelValue,
  (val) => {
    schedule.value = JSON.parse(JSON.stringify(val))
  }
)

watch(schedule, (val) => {
  emit('update:model-value', JSON.parse(JSON.stringify(val)))
}, { deep: true })

function hourToMinutes(hour) {
  return hour * 60
}

function isSelected(day, hour) {
  const intervals = schedule.value[day] || []
  const start = hourToMinutes(hour)
  const end = start + 59
  return intervals.some(i => start <= i.et && end >= i.bt)
}

function toggleHour(day, hour) {
  const start = hourToMinutes(hour)
  const end = start + 59
  let intervals = schedule.value[day] || []

  if (isSelected(day, hour)) {
    intervals = intervals.flatMap(i => {
      if (i.bt <= start && i.et >= end) {
        const parts = []
        if (i.bt < start) parts.push({ bt: i.bt, et: start - 1 })
        if (i.et > end) parts.push({ bt: end + 1, et: i.et })
        return parts
      }
      return [i]
    })
  } else {
    intervals.push({ bt: start, et: end })
    intervals.sort((a, b) => a.bt - b.bt)
    intervals = merge(intervals)
  }
  schedule.value[day] = intervals
}

function merge(intervals) {
  const res = []
  for (const i of intervals) {
    if (!res.length || res[res.length - 1].et < i.bt - 1) {
      res.push({ ...i })
    } else {
      res[res.length - 1].et = Math.max(res[res.length - 1].et, i.et)
    }
  }
  return res
}

const dragging = ref(false)
const dragDay = ref(null)

function startDrag(day, hour) {
  dragging.value = true
  dragDay.value = day
  toggleHour(day, hour)
}

function dragOver(day, hour) {
  if (dragging.value && day === dragDay.value) {
    toggleHour(day, hour)
  }
}

function endDrag() {
  dragging.value = false
  dragDay.value = null
}

function toggleAllDay(day) {
  const full = [{ bt: 0, et: 1439 }]
  schedule.value[day] =
    JSON.stringify(schedule.value[day]) === JSON.stringify(full) ? [] : full
}
</script>

<style scoped>
.calendar {
  display: flex;
  flex-direction: column;
  user-select: none;
}
.header, .day-row {
  display: grid;
  grid-template-columns: 80px repeat(24, 1fr);
  border-bottom: 1px solid #ddd;
}
.day-label {
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  border-right: 1px solid #ddd;
}
.hour-label {
  text-align: center;
  font-size: 12px;
  border-right: 1px solid #eee;
}
.hour-cell {
  border-right: 1px solid #eee;
  height: 25px;
  background: #f0f0f0;
  cursor: pointer;
}
.hour-cell.selected {
  background: #444;
}
.all-day {
  cursor: pointer;
  background: #fafafa;
}
</style>