<template>
  <div class="app">
    <h1>Weekly Calendar</h1>
    <WeekCalendar
      :model-value="schedule"
      @update:model-value="schedule = $event"
    />

    <h2>Output JSON:</h2>
    <pre>{{ JSON.stringify(schedule, null, 2) }}</pre>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import WeekCalendar from './components/WeekCalendar.vue'

const schedule = ref({
  mo: [{ bt: 240, et: 779 }],
  tu: [],
  we: [],
  th: [
    { bt: 240, et: 779 },
    { bt: 1140, et: 1319 }
  ],
  fr: [{ bt: 660, et: 1019 }],
  sa: [{ bt: 0, et: 1439 }],
  su: []
})
</script>

<style>
.app {
  font-family: sans-serif;
  padding: 20px;
}
</style>