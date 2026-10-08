<script setup>
import { reactive, ref } from 'vue'

const props = defineProps({
  routes: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['create', 'update', 'delete'])

const colors = ['sky', 'sunset', 'sea', 'island']
const editingId = ref(null)
const isAdding = ref(false)
const formError = ref('')
const form = reactive(createEmptyRoute())

function createEmptyRoute() {
  return {
    mode: 'flight',
    from: '',
    to: '',
    code: '',
    time: '',
    schedule: '',
    price: 0,
    symbol: '✈',
    color: 'sky'
  }
}

function startAdding() {
  Object.assign(form, createEmptyRoute())
  form.mode = 'flight'
  isAdding.value = true
  editingId.value = null
  formError.value = ''
}

function startEditing(route) {
  Object.assign(form, route)
  editingId.value = route.id
  isAdding.value = false
  formError.value = ''
}

function cancelEditing() {
  isAdding.value = false
  editingId.value = null
  formError.value = ''
}

function saveRoute() {
  if (form.from.trim().toLowerCase() === form.to.trim().toLowerCase()) {
    formError.value = 'Departure and destination must be different.'
    return
  }

  const route = {
    ...form,
    from: form.from.trim(),
    to: form.to.trim(),
    code: form.code.trim(),
    time: form.time.trim(),
    schedule: form.schedule.trim(),
    price: Number(form.price)
  }

  if (editingId.value) {
    emit('update', { ...route, id: editingId.value })
  } else {
    emit('create', route)
  }

  cancelEditing()
}
</script>

<template>
  <section class="manager-section route-manager" aria-labelledby="route-manager-title">
    <div class="manager-heading">
      <div>
        <p class="section-kicker">ROUTE CATALOG</p>
        <h2 id="route-manager-title">Manage routes</h2>
        <p class="manager-description">Add, edit, or remove sample flight and ferry routes.</p>
      </div>
      <button class="manager-primary" type="button" @click="startAdding">+ Add route</button>
    </div>

    <form v-if="isAdding || editingId" class="manager-form" @submit.prevent="saveRoute">
      <h3>{{ editingId ? 'Edit route' : 'Add a route' }}</h3>
      <div class="manager-form-grid">
        <label>
          Travel type
          <select v-model="form.mode">
            <option value="flight">Flight</option>
            <option value="ship">Ship / ferry</option>
          </select>
        </label>
        <label>
          Departure
          <input v-model="form.from" type="text" maxlength="40" required />
        </label>
        <label>
          Destination
          <input v-model="form.to" type="text" maxlength="40" required />
        </label>
        <label>
          Route code
          <input v-model="form.code" type="text" maxlength="24" placeholder="MNL → CEB" required />
        </label>
        <label>
          Travel time
          <input v-model="form.time" type="text" maxlength="30" placeholder="1 hr 25 min" required />
        </label>
        <label>
          Schedule
          <input v-model="form.schedule" type="text" maxlength="50" placeholder="Daily departures" required />
        </label>
        <label>
          Sample fare (₱)
          <input v-model.number="form.price" type="number" min="0" step="1" required />
        </label>
        <label>
          Card color
          <select v-model="form.color">
            <option v-for="color in colors" :key="color" :value="color">{{ color }}</option>
          </select>
        </label>
        <label>
          Card symbol
          <input v-model="form.symbol" type="text" maxlength="3" required />
        </label>
      </div>
      <p v-if="formError" class="manager-error" role="alert">{{ formError }}</p>
      <div class="manager-form-actions">
        <button class="manager-primary" type="submit">{{ editingId ? 'Save changes' : 'Create route' }}</button>
        <button class="manager-secondary" type="button" @click="cancelEditing">Cancel</button>
      </div>
    </form>

    <div v-if="props.routes.length" class="manager-table-wrap">
      <table class="manager-table">
        <thead>
          <tr>
            <th>Type</th>
            <th>Route</th>
            <th>Schedule</th>
            <th>Sample fare</th>
            <th scope="col">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="route in props.routes" :key="route.id">
            <td><span class="type-pill">{{ route.mode === 'flight' ? 'Flight' : 'Ship / ferry' }}</span></td>
            <td><strong>{{ route.from }} → {{ route.to }}</strong><small>{{ route.code }} · {{ route.time }}</small></td>
            <td>{{ route.schedule }}</td>
            <td>₱{{ new Intl.NumberFormat('en-PH').format(route.price) }}</td>
            <td class="row-actions">
              <button class="manager-secondary" type="button" @click="startEditing(route)">Edit</button>
              <button class="manager-danger" type="button" @click="emit('delete', route.id)">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <p v-else class="manager-empty">No routes yet. Add a route to show it in search results.</p>
    <p class="manager-footnote">Changes are saved in this browser only. Sample route data is not a live carrier schedule.</p>
  </section>
</template>

<style scoped>
.manager-section {
  width: 100%;
  min-width: 0;
  padding: 24px;
  border: 1px solid #e5edf0;
  border-radius: 10px;
  background: #fff;
  box-shadow: 0 4px 18px rgba(31, 65, 84, 0.035);
}

.manager-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}

.manager-heading h2 {
  margin: 0;
  color: #193a52;
  font-family: var(--heading-font);
  font-size: 21px;
  letter-spacing: -0.7px;
}

.manager-description,
.manager-footnote,
.manager-empty {
  margin: 6px 0 0;
  color: #7d909c;
  font-size: 11px;
}

.manager-primary,
.manager-secondary,
.manager-danger {
  min-height: 34px;
  padding: 7px 11px;
  border: 1px solid #cde5e8;
  border-radius: 5px;
  background: #f0faf9;
  color: #127f8b;
  cursor: pointer;
  font-size: 10px;
  font-weight: 700;
}

.manager-primary {
  border-color: #117db9;
  background: #117db9;
  color: #fff;
  white-space: nowrap;
}

.manager-primary:hover {
  background: #086da7;
}

.manager-secondary {
  background: #fff;
  color: #557083;
}

.manager-secondary:hover {
  background: #f3f8fa;
}

.manager-danger {
  border-color: #f0d6d8;
  background: #fff8f8;
  color: #a84149;
}

.manager-danger:hover {
  background: #fff0f0;
}

.manager-form {
  margin: 19px 0;
  padding: 17px;
  border: 1px solid #e1ebef;
  border-radius: 7px;
  background: #f8fbfc;
}

.manager-form h3 {
  margin: 0 0 13px;
  color: #274a60;
  font-family: var(--heading-font);
  font-size: 14px;
}

.manager-form-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
}

.manager-form-grid label {
  display: grid;
  gap: 5px;
  color: #657d8c;
  font-size: 10px;
  font-weight: 600;
}

.manager-form-grid input,
.manager-form-grid select {
  width: 100%;
  min-height: 36px;
  padding: 7px 9px;
  border: 1px solid #dce6eb;
  border-radius: 5px;
  background: #fff;
  color: #24465e;
  font-size: 11px;
}

.manager-form-actions {
  display: flex;
  gap: 8px;
  margin-top: 13px;
}

.manager-error {
  margin: 10px 0 0;
  color: #b23b43;
  font-size: 11px;
}

.manager-table-wrap {
  min-width: 0;
  margin-top: 18px;
  overflow-x: auto;
}

.manager-table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
}

.manager-table th {
  padding: 9px 10px;
  border-bottom: 1px solid #e7edf0;
  color: #80919b;
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 0.5px;
  text-transform: uppercase;
}

.manager-table td {
  padding: 11px 10px;
  border-bottom: 1px solid #eef2f4;
  color: #617987;
  font-size: 10px;
  white-space: nowrap;
}

.manager-table tbody tr:last-child td {
  border-bottom: 0;
}

.manager-table td strong,
.manager-table td small {
  display: block;
}

.manager-table td strong {
  color: #274a60;
  font-size: 11px;
}

.manager-table td small {
  margin-top: 3px;
  color: #8999a3;
  font-size: 9px;
}

.type-pill {
  padding: 5px 7px;
  border-radius: 4px;
  background: #edf7f8;
  color: #387c88;
  font-size: 9px;
  font-weight: 700;
}

.row-actions {
  display: flex;
  gap: 6px;
}

.manager-footnote {
  margin-top: 13px;
  font-size: 9px;
}

.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  clip-path: inset(50%);
}

@media (max-width: 650px) {
  .manager-section {
    padding: 17px 13px;
  }

  .manager-form-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (max-width: 390px) {
  .manager-form-grid {
    grid-template-columns: 1fr;
  }
}
</style>
