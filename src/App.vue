<script setup>
import { computed, ref } from 'vue'
import BookingSearch from './components/BookingSearch.vue'
import BookingManager from './components/BookingManager.vue'
import RouteManager from './components/RouteManager.vue'

const activeMode = ref('flight')
const search = ref(null)
const searchMessage = ref('')
const bookingMessage = ref('')
const storageMessage = ref('')

const starterRoutes = [
  {
    id: 'flight-mnl-ceb',
    mode: 'flight',
    from: 'Manila',
    to: 'Cebu',
    code: 'MNL → CEB',
    time: '1 hr 25 min',
    schedule: 'Daily departures',
    price: 2480,
    symbol: '✈',
    color: 'sky'
  },
  {
    id: 'flight-mnl-boracay',
    mode: 'flight',
    from: 'Manila',
    to: 'Boracay',
    code: 'MNL → MPH',
    time: '1 hr 10 min',
    schedule: 'Daily departures',
    price: 3190,
    symbol: '☀',
    color: 'sunset'
  },
  {
    id: 'ship-ceb-bohol',
    mode: 'ship',
    from: 'Cebu',
    to: 'Bohol',
    code: 'CEB → TAG',
    time: '2 hr',
    schedule: 'Morning & afternoon',
    price: 850,
    symbol: '⚓',
    color: 'sea'
  },
  {
    id: 'ship-batangas-puerto-galera',
    mode: 'ship',
    from: 'Batangas',
    to: 'Puerto Galera',
    code: 'BTG → PGL',
    time: '1 hr 15 min',
    schedule: 'Daily departures',
    price: 720,
    symbol: '⛴',
    color: 'island'
  }
]

function isValidRoute(route) {
  return route
    && typeof route.id === 'string'
    && ['flight', 'ship'].includes(route.mode)
    && typeof route.from === 'string'
    && typeof route.to === 'string'
    && typeof route.code === 'string'
    && typeof route.time === 'string'
    && typeof route.schedule === 'string'
    && Number.isFinite(Number(route.price))
    && typeof route.symbol === 'string'
    && ['sky', 'sunset', 'sea', 'island'].includes(route.color)
}

function isValidBooking(booking) {
  return booking
    && typeof booking.id === 'string'
    && typeof booking.from === 'string'
    && typeof booking.to === 'string'
    && ['flight', 'ship'].includes(booking.mode)
    && typeof booking.travelDate === 'string'
    && Number.isInteger(Number(booking.passengers))
    && Number(booking.passengers) >= 1
    && Number.isFinite(Number(booking.price))
}

function loadRecords(key, fallback, isValidRecord) {
  try {
    const saved = localStorage.getItem(key)
    if (saved === null) {
      return fallback
    }

    const records = JSON.parse(saved)
    if (!Array.isArray(records) || !records.every(isValidRecord)) {
      throw new Error('The saved data has an unexpected format.')
    }

    return records
  } catch (error) {
    storageMessage.value = `Could not load saved ${key.replace('skysea-', '')}. ${error.message}`
    return fallback
  }
}

const trips = ref(loadRecords('skysea-routes', starterRoutes, isValidRoute))
const bookings = ref(loadRecords('skysea-bookings', [], isValidBooking))

function saveRecords(key, records) {
  try {
    localStorage.setItem(key, JSON.stringify(records))
    storageMessage.value = ''
  } catch (error) {
    storageMessage.value = `Changes are visible for this session but could not be saved in this browser. ${error.message}`
  }
}

const visibleTrips = computed(() => {
  if (!search.value) {
    return trips.value.filter(trip => trip.mode === activeMode.value)
  }

  return trips.value.filter(trip => {
    const matchesMode = trip.mode === search.value.mode
    const matchesFrom = trip.from.toLowerCase() === search.value.from.toLowerCase()
    const matchesTo = trip.to.toLowerCase() === search.value.to.toLowerCase()

    return matchesMode && matchesFrom && matchesTo
  })
})

function changeMode(mode) {
  activeMode.value = mode
  search.value = null
  searchMessage.value = ''
  bookingMessage.value = ''
}

function searchTrips(criteria) {
  search.value = { ...criteria, mode: activeMode.value }
  bookingMessage.value = ''
  searchMessage.value = visibleTrips.value.length
    ? 'Sample routes are shown below. Fares are indicative and may change.'
    : 'No sample route matches this search yet. Try another route.'
}

function createBooking(trip) {
  const departDate = search.value?.departDate || dateAfterToday(14)
  const returnDate = search.value?.tripType === 'one-way'
    ? ''
    : search.value?.returnDate || dateAfterToday(21)

  bookings.value.unshift({
    id: `SKY-${Date.now().toString(36).toUpperCase()}`,
    tripId: trip.id,
    mode: trip.mode,
    from: trip.from,
    to: trip.to,
    routeCode: trip.code,
    time: trip.time,
    price: Number(trip.price),
    travelDate: departDate,
    returnDate,
    passengers: search.value?.passengers || 1
  })
  saveRecords('skysea-bookings', bookings.value)
  bookingMessage.value = `Demo reservation saved for ${trip.from} to ${trip.to}. Review it in Manage trips.`
}

function dateAfterToday(days) {
  const date = new Date()
  date.setDate(date.getDate() + days)
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

function createRoute(route) {
  trips.value.push({
    ...route,
    id: `route-${Date.now().toString(36)}`
  })
  saveRecords('skysea-routes', trips.value)
}

function updateRoute(route) {
  trips.value = trips.value.map(existing => existing.id === route.id ? route : existing)
  saveRecords('skysea-routes', trips.value)
}

function deleteRoute(id) {
  trips.value = trips.value.filter(route => route.id !== id)
  saveRecords('skysea-routes', trips.value)
}

function updateBooking(booking) {
  bookings.value = bookings.value.map(existing => existing.id === booking.id ? booking : existing)
  saveRecords('skysea-bookings', bookings.value)
}

function deleteBooking(id) {
  bookings.value = bookings.value.filter(booking => booking.id !== id)
  saveRecords('skysea-bookings', bookings.value)
}

function formatPrice(price) {
  return new Intl.NumberFormat('en-PH').format(price)
}
</script>

<template>
  <div class="site-shell">
    <header class="site-header">
      <a class="brand" href="#" aria-label="SkySea home">
        <span class="brand-mark" aria-hidden="true">
          <svg viewBox="0 0 40 40" fill="none">
            <path d="M7 24.5 20 5l13 19.5-13-5.2-13 5.2Z" fill="currentColor" />
            <path d="M5 28.5c4.2-2.6 8.4-2.6 12.6 0 4.2 2.6 8.4 2.6 12.6 0 1.7-1 3.3-1.6 4.8-1.8" stroke="currentColor" stroke-width="3" stroke-linecap="round" />
            <path d="M9 34c3.4-1.9 6.8-1.9 10.2 0 3.4 1.9 6.8 1.9 10.2 0" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" />
          </svg>
        </span>
        <span>sky<span class="brand-accent">sea</span></span>
      </a>

      <nav class="main-nav" aria-label="Main navigation">
        <a class="nav-link active" href="#book">Book a trip</a>
        <a class="nav-link" href="#popular">Popular routes</a>
        <a class="nav-link" href="#manage">Manage trips</a>
        <a class="nav-link" href="#about">Why SkySea</a>
      </nav>

      <a class="header-help" href="#about">
        <span class="help-icon" aria-hidden="true">?</span>
        <span>Need help?</span>
      </a>
    </header>

    <main>
      <section class="hero-section" id="book">
        <div class="hero-content">
          <div class="eyebrow"><span class="eyebrow-dot"></span> YOUR NEXT ADVENTURE STARTS HERE</div>
          <h1>Go further.<br /><span>Feel closer.</span></h1>
          <p class="hero-copy">
            Island-hop by air or take the scenic route by sea.<br class="desktop-break" />
            Your Philippines adventure is just a booking away.
          </p>
          <div class="hero-highlights" aria-label="SkySea benefits">
            <span><span aria-hidden="true">✓</span> Easy booking</span>
            <span><span aria-hidden="true">✓</span> Island-wide routes</span>
            <span><span aria-hidden="true">✓</span> One happy journey</span>
          </div>
        </div>
        <div class="hero-art" aria-hidden="true">
          <div class="sun-disc"></div>
          <div class="island island-back"></div>
          <div class="island island-front"></div>
          <div class="cloud cloud-one"></div>
          <div class="cloud cloud-two"></div>
          <div class="wave wave-one"></div>
          <div class="wave wave-two"></div>
          <div class="art-label"><span>✦</span> 7,641 islands to explore</div>
        </div>
        <div class="hero-decoration decoration-one"></div>
        <div class="hero-decoration decoration-two"></div>
      </section>

      <section class="booking-section" aria-label="Search flights and ferries">
        <div class="booking-card">
          <div class="booking-heading">
            <div>
              <p class="section-kicker">PLAN YOUR JOURNEY</p>
              <h2>Where would you like to go?</h2>
            </div>
            <span class="secure-note"><span aria-hidden="true">✦</span> Simple, seamless travel</span>
          </div>
          <div class="mode-switch" role="tablist" aria-label="Choose how to travel">
            <button
              class="mode-button"
              :class="{ selected: activeMode === 'flight' }"
              type="button"
              role="tab"
              :aria-selected="activeMode === 'flight'"
              @click="changeMode('flight')"
            >
              <span aria-hidden="true">✈</span> Flights
            </button>
            <button
              class="mode-button"
              :class="{ selected: activeMode === 'ship' }"
              type="button"
              role="tab"
              :aria-selected="activeMode === 'ship'"
              @click="changeMode('ship')"
            >
              <span aria-hidden="true">⚓</span> Ships & ferries
            </button>
          </div>
          <BookingSearch :mode="activeMode" :routes="trips" @search="searchTrips" />
        </div>
      </section>

      <section class="popular-section" id="popular">
        <div class="section-heading">
          <div>
            <p class="section-kicker">{{ search ? 'YOUR SEARCH' : 'A LITTLE TRAVEL INSPIRATION' }}</p>
            <h2>{{ search ? 'Routes for your journey' : 'Popular journeys' }}</h2>
            <p v-if="search" class="search-summary">
              {{ search.from }} to {{ search.to }} · {{ search.mode === 'flight' ? 'Flight' : 'Ship & ferry' }}
              · {{ search.departDate }}
            </p>
          </div>
          <a v-if="!search" class="text-link" href="#book">Plan a trip <span aria-hidden="true">→</span></a>
        </div>

        <p v-if="searchMessage" class="search-message" role="status">{{ searchMessage }}</p>
        <p v-if="bookingMessage" class="selection-message" role="status">{{ bookingMessage }}</p>

        <div v-if="visibleTrips.length" class="trip-grid">
          <article v-for="trip in visibleTrips" :key="trip.id" class="trip-card">
            <div class="trip-art" :class="`trip-art-${trip.color}`">
              <span class="trip-symbol" aria-hidden="true">{{ trip.symbol }}</span>
              <span class="trip-badge">{{ trip.mode === 'flight' ? '✈ FLIGHT' : '⚓ FERRY' }}</span>
              <span class="trip-island-shape"></span>
            </div>
            <div class="trip-details">
              <div class="trip-route">
                <span>{{ trip.from }}</span>
                <span class="route-arrow" aria-hidden="true">→</span>
                <span>{{ trip.to }}</span>
              </div>
              <p class="trip-code">{{ trip.code }} <span>·</span> {{ trip.time }}</p>
              <div class="trip-footer">
                <div>
                  <span class="price-caption">Sample fare from</span>
                  <strong>₱{{ formatPrice(trip.price) }}<small> / person</small></strong>
                </div>
                <button class="trip-select" type="button" @click="createBooking(trip)">
                  Book trip
                </button>
              </div>
              <p class="trip-schedule">{{ trip.schedule }}</p>
            </div>
          </article>
        </div>
        <div v-else-if="search" class="empty-state">
          <span aria-hidden="true">🧭</span>
          <div>
            <strong>We couldn't find a sample route for that trip.</strong>
            <p>Try one of the popular routes, or change your travel mode above.</p>
          </div>
        </div>
      </section>

      <section class="management-section" id="manage">
        <p v-if="storageMessage" class="storage-message" role="alert">{{ storageMessage }}</p>
        <BookingManager
          :bookings="bookings"
          @update="updateBooking"
          @delete="deleteBooking"
        />
        <RouteManager
          :routes="trips"
          @create="createRoute"
          @update="updateRoute"
          @delete="deleteRoute"
        />
      </section>

      <section class="about-section" id="about">
        <div class="about-copy">
          <p class="section-kicker">THE SKYSEA WAY</p>
          <h2>More than getting there.<br /><span>It's how you get there.</span></h2>
          <p>From a quick flight to a slow sail, make every island connection feel effortless.</p>
        </div>
        <div class="about-features">
          <article class="feature-item">
            <span class="feature-icon icon-blue" aria-hidden="true">↗</span>
            <div><h3>One easy search</h3><p>Explore air and sea journeys in one friendly place.</p></div>
          </article>
          <article class="feature-item">
            <span class="feature-icon icon-green" aria-hidden="true">◎</span>
            <div><h3>Made for island life</h3><p>Find routes that bring the Philippines closer together.</p></div>
          </article>
        </div>
      </section>
    </main>

    <footer class="site-footer">
      <a class="brand footer-brand" href="#" aria-label="SkySea home">
        <span class="brand-mark" aria-hidden="true">
          <svg viewBox="0 0 40 40" fill="none">
            <path d="M7 24.5 20 5l13 19.5-13-5.2-13 5.2Z" fill="currentColor" />
            <path d="M5 28.5c4.2-2.6 8.4-2.6 12.6 0 4.2 2.6 8.4 2.6 12.6 0 1.7-1 3.3-1.6 4.8-1.8" stroke="currentColor" stroke-width="3" stroke-linecap="round" />
            <path d="M9 34c3.4-1.9 6.8-1.9 10.2 0 3.4 1.9 6.8 1.9 10.2 0" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" />
          </svg>
        </span>
        <span>sky<span class="brand-accent">sea</span></span>
      </a>
      <p>Every island, one happy journey.</p>
      <span class="footer-note">A travel-booking demo · Fares shown are samples</span>
    </footer>
  </div>
</template>
