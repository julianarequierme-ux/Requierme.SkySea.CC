# SkySea

SkySea is a Vue 3 demo for exploring sample airplane and ship/ferry bookings around the Philippines. Search by travel mode, route, dates, and number of travelers.

The app includes CRUD controls for sample routes and demo reservations. Add, edit, and delete routes; create a reservation from a route card, then review, edit, or delete it in the management area. Data is stored in the browser's local storage.

Routes and fares are examples only. The app does not connect to live schedules, accept payments, or issue tickets. Reservations created here are not confirmed bookings.

## Project structure

- `src/App.vue` - page layout, sample routes, and search results
- `src/components/BookingSearch.vue` - reusable trip search form
- `src/components/BookingManager.vue` - review, edit, and delete demo reservations
- `src/components/RouteManager.vue` - create, edit, and delete sample routes
- `src/style.css` - global styles and responsive page layout
- `src/main.js` - Vue app entry point

## Run locally

```sh
npm install
npm run dev
```

Create a production build with `npm run build`.
