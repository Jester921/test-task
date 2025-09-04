<template>
  <div class="calendar">
    <div class="header">
      <div class="day-label">Day</div>
      <div v-for="h in 24" :key="h" class="hour-label">
        {{ (h - 1).toString().padStart(2, '0') }}:00
      </div>
    </div>

    <div v-for="(dayKey, idx) in days" :key="dayKey" class="day-row">
      <button class="day-label all-day" @mousedown.prevent="toggleAllDay(dayKey)">
        {{ labels[idx] }}
      </button>

      <div
        v-for="hour in 24"
        :key="hour"
        class="hour-cell"
        :class="{ selected: isSelected(dayKey, hour - 1) }"
        @mousedown.prevent="startDrag(dayKey, hour - 1)"
        @mouseenter="dragOver(dayKey, hour - 1)"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const days = ['mo', 'tu', 'we', 'th', 'fr', 'sa', 'su']
const labels = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']

const props = defineProps({
  modelValue: { type: Object, required: true }
})
const emit = defineEmits(['update:model-value'])

const schedule = ref(deepClone(props.modelValue))

watch(
  () => props.modelValue,
  (val) => {
    schedule.value = deepClone(val)
  },
  { deep: true }
)

watch(
  schedule,
  (val) => {
    emit('update:model-value', deepClone(val))
  },
  { deep: true }
)

function deepClone(v) {
  return JSON.parse(JSON.stringify(v))
}
function hourToMinutes(hour) {
  return hour * 60
}
function isSelected(day, hour) {
  const intervals = schedule.value[day] || []
  const start = hourToMinutes(hour)
  const end = start + 59
  return intervals.some((i) => start <= i.et && end >= i.bt)
}

function addHour(day, hour) {
  const start = hour * 60
  const end = start + 59
  let intervals = schedule.value[day] || []
  intervals.push({ bt: start, et: end })
  intervals.sort((a, b) => a.bt - b.bt)
  schedule.value[day] = merge(intervals)
}

function removeHour(day, hour) {
  const start = hour * 60
  const end = start + 59
  let intervals = schedule.value[day] || []
  intervals = intervals.flatMap((i) => {
    if (i.bt <= start && i.et >= end) {
      const parts = []
      if (i.bt < start) parts.push({ bt: i.bt, et: start - 1 })
      if (i.et > end) parts.push({ bt: end + 1, et: i.et })
      return parts
    }
    return [i]
  })
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
const dragMode = ref(null)

function startDrag(day, hour) {
  dragging.value = true
  dragDay.value = day
  dragMode.value = isSelected(day, hour) ? 'remove' : 'add'
  applyDrag(day, hour)
  window.addEventListener('mouseup', endDragOnce)
}

function dragOver(day, hour) {
  if (!dragging.value || day !== dragDay.value) return
  applyDrag(day, hour)
}

function applyDrag(day, hour) {
  if (dragMode.value === 'add' && !isSelected(day, hour)) {
    addHour(day, hour)
  }
  if (dragMode.value === 'remove' && isSelected(day, hour)) {
    removeHour(day, hour)
  }
}

function endDrag() {
  dragging.value = false
  dragDay.value = null
  dragMode.value = null
  window.removeEventListener('mouseup', endDragOnce)
}
function endDragOnce() {
  endDrag()
}

function toggleAllDay(day) {
  const full = [{ bt: 0, et: 1439 }]
  const current = schedule.value[day] || []
  const isFull =
    current.length === 1 && current[0].bt === 0 && current[0].et === 1439
  schedule.value[day] = isFull ? [] : full
}
</script>

<style scoped>
.calendar {
  display: flex;
  flex-direction: column;
  user-select: none;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  overflow: hidden;
}
.header,
.day-row {
  display: grid;
  grid-template-columns: 90px repeat(24, 1fr);
}
.header {
  background: #fafafa;
  border-bottom: 1px solid #e5e7eb;
}
.day-row {
  border-top: 1px solid #f2f2f2;
}
.day-label {
  display: flex;
  color: red;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  border-right: 1px solid #eee;
  padding: 8px;
}
.all-day {
  cursor: pointer;
  background: #fff;
  border: none;
  font: inherit;
}
.hour-label {
  color: red;
  text-align: center;
  font-size: 12px;
  border-right: 1px solid #f0f0f0;
  padding: 6px 0;
}
.hour-cell {
  border-right: 1px solid #f7f7f7;
  height: 28px;
  background: #f3f4f6;
  cursor: pointer;
  transition: background 80ms ease-out;
}
.hour-cell:hover {
  background: #e5e7eb;
}
.hour-cell.selected {
  background: #111827;
}
</style>
