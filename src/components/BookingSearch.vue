<script setup>
import { computed, reactive, watch } from 'vue'

const props = defineProps({
  mode: {
    type: String,
    required: true
  },
  routes: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['search'])

function toDateInput(date) {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')

  return `${year}-${month}-${day}`
}

function addDays(date, days) {
  const result = new Date(date)
  result.setDate(result.getDate() + days)
  return toDateInput(result)
}

const today = toDateInput(new Date())
const form = reactive({
  tripType: 'round-trip',
  from: props.mode === 'flight' ? 'Manila' : 'Cebu',
  to: props.mode === 'flight' ? 'Cebu' : 'Bohol',
  departDate: addDays(new Date(), 14),
  returnDate: addDays(new Date(), 21),
  passengers: 1
})

const locations = computed(() => [
  ...new Set(
    props.routes
      .filter(route => route.mode === props.mode)
      .flatMap(route => [route.from, route.to])
  )
])

watch(
  [() => props.mode, locations],
  () => {
    if (!locations.value.includes(form.from)) {
      form.from = locations.value[0] || ''
    }

    if (!locations.value.includes(form.to) || form.to === form.from) {
      form.to = locations.value.find(location => location !== form.from) || ''
    }
  },
  { immediate: true }
)

function swapLocations() {
  const origin = form.from
  form.from = form.to
  form.to = origin
}

function keepReturnDateValid() {
  if (form.returnDate < form.departDate) {
    form.returnDate = form.departDate
  }
}

function submitSearch() {
  if (!locations.value.length || !form.from || !form.to || form.from === form.to) {
    return
  }

  emit('search', {
    ...form,
    passengers: Number(form.passengers)
  })
}
</script>

<template>
  <form class="search-form" @submit.prevent="submitSearch">
    <fieldset class="trip-type-control">
      <legend class="visually-hidden">Trip type</legend>
      <label class="radio-option">
        <input v-model="form.tripType" type="radio" value="round-trip" />
        <span>Round trip</span>
      </label>
      <label class="radio-option">
        <input v-model="form.tripType" type="radio" value="one-way" />
        <span>One way</span>
      </label>
    </fieldset>

    <div class="search-fields">
      <label class="field-group location-field">
        <span class="field-label">{{ mode === 'flight' ? 'FROM' : 'DEPARTING FROM' }}</span>
        <select v-model="form.from" required>
          <option v-for="location in locations" :key="`from-${location}`" :value="location">
            {{ location }}
          </option>
        </select>
      </label>

      <button class="swap-button" type="button" aria-label="Swap departure and destination" @click="swapLocations">
        <span aria-hidden="true">⇄</span>
      </button>

      <label class="field-group location-field">
        <span class="field-label">{{ mode === 'flight' ? 'TO' : 'ARRIVING AT' }}</span>
        <select v-model="form.to" required>
          <option v-for="location in locations" :key="`to-${location}`" :value="location">
            {{ location }}
          </option>
        </select>
      </label>

      <label class="field-group date-field field-depart">
        <span class="field-label">DEPARTURE</span>
        <input v-model="form.departDate" type="date" :min="today" required @change="keepReturnDateValid" />
      </label>

      <label v-if="form.tripType === 'round-trip'" class="field-group date-field field-return">
        <span class="field-label">RETURN</span>
        <input v-model="form.returnDate" type="date" :min="form.departDate" required />
      </label>

      <label class="field-group passenger-field field-passengers">
        <span class="field-label">TRAVELERS</span>
        <select v-model.number="form.passengers" required>
          <option v-for="count in 9" :key="count" :value="count">
            {{ count }} {{ count === 1 ? 'traveler' : 'travelers' }}
          </option>
        </select>
      </label>

      <button class="search-button" type="submit" :disabled="!locations.length">
        <svg viewBox="0 0 20 20" aria-hidden="true">
          <circle cx="8.8" cy="8.8" r="5.8" fill="none" stroke="currentColor" stroke-width="1.8" />
          <path d="m13.2 13.2 4 4" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" />
        </svg>
        Search {{ mode === 'flight' ? 'flights' : 'ferries' }}
      </button>
    </div>
    <p v-if="!locations.length" class="form-hint form-error" role="status">
      No sample routes for this travel type yet. Add one in Manage trips.
    </p>
    <p v-else-if="form.from === form.to" class="form-hint form-error" role="alert">
      Choose a different destination from your departure point.
    </p>
    <p v-else class="form-hint">
      <span aria-hidden="true">✦</span> Sample routes only — no live availability or ticketing.
    </p>
  </form>
</template>
