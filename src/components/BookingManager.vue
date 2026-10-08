<script setup>
import { reactive, ref } from 'vue'

const props = defineProps({
  bookings: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['update', 'delete'])
const editingId = ref(null)
const editError = ref('')
const draft = reactive({
  travelDate: '',
  returnDate: '',
  passengers: 1
})

function startEditing(booking) {
  editingId.value = booking.id
  editError.value = ''
  draft.travelDate = booking.travelDate
  draft.returnDate = booking.returnDate || ''
  draft.passengers = booking.passengers
}

function cancelEditing() {
  editingId.value = null
  editError.value = ''
}

function keepReturnDateValid() {
  if (draft.returnDate && draft.returnDate < draft.travelDate) {
    draft.returnDate = draft.travelDate
  }
}

function saveBooking(booking) {
  if (draft.returnDate && draft.returnDate < draft.travelDate) {
    editError.value = 'Return date cannot be before the departure date.'
    return
  }

  emit('update', {
    ...booking,
    travelDate: draft.travelDate,
    returnDate: draft.returnDate,
    passengers: Number(draft.passengers)
  })
  cancelEditing()
}

function formatPrice(price) {
  return new Intl.NumberFormat('en-PH').format(price * 1)
}
</script>

<template>
  <section class="manager-section booking-manager" aria-labelledby="booking-manager-title">
    <div class="manager-heading">
      <div>
        <p class="section-kicker">YOUR TRIPS</p>
        <h2 id="booking-manager-title">Manage bookings</h2>
        <p class="manager-description">Review, update, or delete reservations saved in this browser.</p>
      </div>
      <span class="booking-count">{{ props.bookings.length }} {{ props.bookings.length === 1 ? 'booking' : 'bookings' }}</span>
    </div>

    <div v-if="props.bookings.length" class="booking-list">
      <article v-for="booking in props.bookings" :key="booking.id" class="booking-record">
        <div class="booking-record-top">
          <div>
            <span class="booking-reference">{{ booking.id }}</span>
            <h3>{{ booking.from }} <span>→</span> {{ booking.to }}</h3>
            <p>{{ booking.mode === 'flight' ? 'Flight' : 'Ship / ferry' }} · {{ booking.routeCode }} · {{ booking.time }}</p>
          </div>
          <span class="demo-status">Saved demo</span>
        </div>

        <form v-if="editingId === booking.id" class="booking-edit-form" @submit.prevent="saveBooking(booking)">
          <label>
            Departure date
            <input v-model="draft.travelDate" type="date" required @change="keepReturnDateValid" />
          </label>
          <label v-if="booking.returnDate">
            Return date
            <input v-model="draft.returnDate" type="date" :min="draft.travelDate" required />
          </label>
          <label>
            Travelers
            <select v-model.number="draft.passengers">
              <option v-for="count in 9" :key="count" :value="count">{{ count }}</option>
            </select>
          </label>
          <p v-if="editError" class="manager-error" role="alert">{{ editError }}</p>
          <div class="booking-actions">
            <button class="manager-primary" type="submit">Save changes</button>
            <button class="manager-secondary" type="button" @click="cancelEditing">Cancel</button>
          </div>
        </form>
        <div v-else class="booking-record-bottom">
          <div class="booking-facts">
            <span><small>DEPARTURE</small>{{ booking.travelDate }}</span>
            <span v-if="booking.returnDate"><small>RETURN</small>{{ booking.returnDate }}</span>
            <span><small>TRAVELERS</small>{{ booking.passengers }}</span>
            <span><small>DEMO TOTAL</small>₱{{ formatPrice(booking.price * booking.passengers) }}</span>
          </div>
          <div class="booking-actions">
            <button class="manager-secondary" type="button" @click="startEditing(booking)">Edit</button>
            <button class="manager-danger" type="button" @click="emit('delete', booking.id)">Delete</button>
          </div>
        </div>
      </article>
    </div>
    <p v-else class="manager-empty">No bookings yet. Search a route and choose “Book trip” to add one.</p>
    <p class="manager-footnote">Demo reservations are not tickets or confirmed travel. No payment is collected.</p>
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

.manager-heading,
.booking-record-top,
.booking-record-bottom {
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

.booking-count,
.demo-status {
  padding: 6px 9px;
  border-radius: 20px;
  background: #edf7f8;
  color: #387c88;
  font-size: 9px;
  font-weight: 700;
  white-space: nowrap;
}

.demo-status {
  background: #fff8e9;
  color: #9b7737;
}

.booking-list {
  display: grid;
  gap: 11px;
  margin-top: 17px;
}

.booking-record {
  padding: 15px;
  border: 1px solid #e7edf0;
  border-radius: 7px;
  background: #fcfefe;
}

.booking-reference {
  color: #8295a0;
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 0.6px;
}

.booking-record h3 {
  margin: 4px 0;
  color: #1a3d54;
  font-family: var(--heading-font);
  font-size: 14px;
  font-weight: 800;
}

.booking-record h3 span {
  color: #27a4a7;
}

.booking-record-top p {
  margin: 0;
  color: #83939d;
  font-size: 10px;
}

.booking-record-bottom {
  margin-top: 13px;
}

.booking-facts {
  display: flex;
  flex-wrap: wrap;
  gap: 13px 23px;
}

.booking-facts > span {
  display: grid;
  gap: 4px;
  color: #34566b;
  font-size: 10px;
  font-weight: 600;
}

.booking-facts small {
  color: #8998a1;
  font-size: 8px;
  font-weight: 700;
  letter-spacing: 0.4px;
}

.booking-actions {
  display: flex;
  flex: 0 0 auto;
  gap: 6px;
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

.booking-edit-form {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-end;
  gap: 10px;
  margin-top: 14px;
  padding-top: 13px;
  border-top: 1px solid #e7edf0;
}

.booking-edit-form label {
  display: grid;
  gap: 5px;
  color: #657d8c;
  font-size: 9px;
  font-weight: 600;
}

.booking-edit-form input,
.booking-edit-form select {
  min-height: 34px;
  padding: 6px 8px;
  border: 1px solid #dce6eb;
  border-radius: 5px;
  background: #fff;
  color: #24465e;
  font-size: 10px;
}

.booking-edit-form .manager-error {
  flex-basis: 100%;
  margin: 0;
  color: #b23b43;
  font-size: 10px;
}

.manager-footnote {
  margin-top: 13px;
  font-size: 9px;
}

@media (max-width: 650px) {
  .manager-section {
    padding: 17px 13px;
  }

  .booking-record {
    padding: 12px;
  }

  .booking-record-bottom {
    align-items: flex-start;
    flex-direction: column;
  }
}

@media (max-width: 390px) {
  .booking-record-top {
    align-items: flex-start;
    flex-direction: column-reverse;
    gap: 8px;
  }
}
</style>
