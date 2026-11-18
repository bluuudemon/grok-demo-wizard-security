# Fly-Lo Air booking app bundle

Self-contained source for the deliberately insecure Fly-Lo Air booking demo, so it can be recreated without access to the original Cursor Origin repo.

- **Repo:** `flylo-air/security-checks` (Cursor Origin, internal)
- **Primary ref:** tag `demo-baseline` → commit `126585dec8e5d59d60453415c89097517636943d` (tree `9b4ccbf4b2a581b2634e38c970917132fad5f14a`)
- **Commit message:** Add Fly-Lo flight booking demo app (#1)
- **Commit time:** 2026-11-12 09:36:59 America/Toronto (14:36:59 UTC)
- **Included files:** 25 (23 blobs from the tag, plus 2 `demo/` files from `demo/assets` / current `main`)
- **Origin repo:** not modified

## How to run

Requires Node.js >= 22.13 (uses the built-in `node:sqlite` module).

```bash
npm install
npm test      # API tests against an in-memory database (8 tests)
npm start     # http://localhost:3000
npm run dev   # same as start, with --watch
```

To regenerate the excluded Snyk and CloudWatch fixtures after unpacking:

```bash
node demo/tools/generate-fixtures.js
```

## npm scripts

| Script | Command |
| --- | --- |
| `start` | `node --no-warnings src/server.js` |
| `dev` | `node --no-warnings --watch src/server.js` |
| `test` | `node --no-warnings --test "test/**/*.test.js"` |

## Notes for unpacking

- **`.env` is a replacement**, not a copy of the original. Key names match `.env.example` and `src/config.js`. Values are obviously fake (`DEMO` / `FAKE`). The original `.env` was never read. The file is still committed so the “secret in the repo” finding stays intact for the demo.
- **`demo/` is not in the `demo-baseline` tag.** App blobs on `demo/assets` and current `main` (tree `a8d9294ab15a27632a9ea0e421efd12963629cf6`) are identical to the tag. This bundle adds `demo/RUNBOOK.md` and `demo/tools/generate-fixtures.js` from that tree.
- **Excluded large fixtures** (regenerate with the script above):
  - `demo/scans/snyk-report.json` — 11,122 bytes on Origin
  - `demo/logs/cloudwatch-flylo-booking-api.json` — 72,127 bytes on Origin
- **Not in the tree:** `node_modules/`, binaries, or lockfiles over 50 KB. `package-lock.json` is 36,074 bytes and is included.
- Seed accounts (also listed in the README): `admin@flylo-air.com` / `admin123`, `alice@example.com` / `password1`, `bob@example.com` / `qwerty`. These are documented demo seeds, not copied from the original `.env`.

## Files

### .env

Same keys as the committed .env; values are obviously fake. Real .env was never read.

```dotenv
PORT=3000
NODE_ENV=development
JWT_SECRET=demo_fake_jwt_secret_DO_NOT_USE
DB_PATH=data/flylo.db
PAYMENT_GATEWAY_KEY=sk_test_DEMO_FAKE_000000
ADMIN_EMAIL=admin-demo@example.invalid
ADMIN_PASSWORD=DEMO_FAKE_admin_password
```

### .env.example

```dotenv
PORT=3000
NODE_ENV=development
JWT_SECRET=
DB_PATH=data/flylo.db
PAYMENT_GATEWAY_KEY=
ADMIN_EMAIL=
ADMIN_PASSWORD=
```

### .gitignore

```gitignore
node_modules/
data/*.db
data/*.db-*
boarding-passes/*.txt
npm-debug.log*
.DS_Store
```

### Dockerfile

```
FROM node:22

WORKDIR /app
COPY . .
RUN npm install

EXPOSE 3000
CMD ["npm", "start"]
```

### README.md

````markdown
# Fly-Lo Air booking

A small flight booking web app for Fly-Lo Air. Customers search flights, sign in, book with a card, view their trips, download a boarding pass and cancel bookings. Admins can list users and bookings and change fares.

> This repository is a demo test bed for automated security review. It intentionally contains insecure code. Do not deploy it with real data.

## Stack

- Node.js 22 (uses the built-in `node:sqlite` module, so there are no native dependencies)
- Express 4 with JWT auth
- Vanilla HTML/CSS/JS frontend in `public/`

## Running locally

```bash
npm install
npm start          # http://localhost:3000
npm test           # API tests against an in-memory database
```

The SQLite database is created at `data/flylo.db` and seeded on first start. Delete the file to reset it.

### Seed accounts

| Email | Password | Role |
| --- | --- | --- |
| admin@flylo-air.com | admin123 | admin |
| alice@example.com | password1 | customer |
| bob@example.com | qwerty | customer |

### Docker

```bash
docker build -t flylo-booking .
docker run -p 3000:3000 flylo-booking
```

## API

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| GET | `/api/health` | – | Health check |
| POST | `/api/auth/register` | – | Create an account |
| POST | `/api/auth/login` | – | Sign in, returns a JWT |
| GET | `/api/flights/airports` | – | Airport codes served |
| GET | `/api/flights/search?from=&to=&date=` | – | Search flights |
| GET | `/api/flights/:id` | – | Flight details |
| POST | `/api/bookings` | user | Book a flight |
| GET | `/api/bookings` | user | My bookings |
| GET | `/api/bookings/:reference` | user | Booking details |
| GET | `/api/bookings/boarding-pass?file=` | user | Boarding pass text |
| POST | `/api/bookings/:reference/cancel` | user | Cancel a booking |
| GET | `/api/users/me` | user | My profile |
| PUT | `/api/users/me` | user | Update name / phone |
| GET | `/api/admin/users` | admin | All users |
| GET | `/api/admin/bookings` | admin | All bookings |
| PUT | `/api/admin/flights/:id/price` | admin | Change a fare |

## Project layout

```
src/
  app.js          Express app and middleware
  server.js       Entry point
  config.js       Loads .env
  db.js           Schema and seed data
  auth.js         JWT helpers and middleware
  routes/         auth, flights, bookings, users, admin
public/           Frontend
test/             node:test API tests
```
````

### boarding-passes/.gitkeep

```
```

### data/.gitkeep

```
```

### package.json

```json
{
  "name": "flylo-booking",
  "version": "0.1.0",
  "description": "Fly-Lo flight booking web app",
  "main": "src/server.js",
  "private": true,
  "scripts": {
    "start": "node --no-warnings src/server.js",
    "dev": "node --no-warnings --watch src/server.js",
    "test": "node --no-warnings --test \"test/**/*.test.js\""
  },
  "engines": {
    "node": ">=22.13"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^4.21.2",
    "jsonwebtoken": "^9.0.2",
    "lodash": "4.17.15"
  }
}
```

### public/app.js

```javascript
const state = {
  token: localStorage.getItem('flylo_token'),
  user: JSON.parse(localStorage.getItem('flylo_user') || 'null'),
  registering: false,
};

const $ = (sel) => document.querySelector(sel);

async function api(path, options = {}) {
  const headers = { 'Content-Type': 'application/json', ...(options.headers || {}) };
  if (state.token) headers.Authorization = `Bearer ${state.token}`;
  const res = await fetch(path, { ...options, headers });
  const isJson = res.headers.get('content-type')?.includes('application/json');
  const body = isJson ? await res.json() : await res.text();
  if (!res.ok) throw new Error(body.error || body || res.statusText);
  return body;
}

function toast(message) {
  const el = $('#toast');
  el.textContent = message;
  el.classList.remove('hidden');
  clearTimeout(toast.timer);
  toast.timer = setTimeout(() => el.classList.add('hidden'), 3000);
}

function fmtTime(iso) {
  return new Date(iso).toLocaleString(undefined, { dateStyle: 'medium', timeStyle: 'short' });
}

function show(view) {
  document.querySelectorAll('.view').forEach((v) => v.classList.add('hidden'));
  $(`#view-${view}`).classList.remove('hidden');
  if (view === 'trips') loadTrips();
}

function renderAuth() {
  const badge = $('#user-badge');
  if (state.user) {
    badge.innerHTML = `Hi, ${state.user.name} · ${state.user.loyalty_points ?? 0} pts`;
    badge.classList.remove('hidden');
    $('#auth-btn').textContent = 'Sign out';
  } else {
    badge.classList.add('hidden');
    $('#auth-btn').textContent = 'Sign in';
  }
}

function setSession(token, user) {
  state.token = token;
  state.user = user;
  if (token) {
    localStorage.setItem('flylo_token', token);
    localStorage.setItem('flylo_user', JSON.stringify(user));
  } else {
    localStorage.removeItem('flylo_token');
    localStorage.removeItem('flylo_user');
  }
  renderAuth();
}

async function loadAirports() {
  const { airports } = await api('/api/flights/airports');
  const options = airports.map((code) => `<option value="${code}">${code}</option>`).join('');
  $('#from').innerHTML = options;
  $('#to').innerHTML = options;
  $('#from').value = 'YYZ';
  $('#to').value = 'YVR';
  const tomorrow = new Date(Date.now() + 86400000);
  $('#date').value = tomorrow.toISOString().slice(0, 10);
}

async function search(event) {
  event?.preventDefault();
  const params = new URLSearchParams(new FormData($('#search-form')));
  const { flights } = await api(`/api/flights/search?${params}`);
  const results = $('#results');
  if (!flights.length) {
    results.innerHTML = '<div class="card empty">No flights found for that search.</div>';
    return;
  }
  results.innerHTML = flights.map((f) => `
    <div class="flight">
      <div><div class="num">${f.flight_number}</div><div class="time">${f.seats_available} seats left</div></div>
      <div><div class="route">${f.origin} → ${f.destination}</div>
        <div class="time">${fmtTime(f.depart_time)} – ${fmtTime(f.arrive_time)}</div></div>
      <div class="price">$${f.price}</div>
      <button class="btn primary" data-book='${JSON.stringify(f)}'>Select</button>
    </div>`).join('');
}

function startBooking(flight) {
  if (!state.token) {
    openAuth();
    return;
  }
  const form = $('#book-form');
  form.reset();
  form.flightId.value = flight.id;
  form.totalPrice.value = flight.price;
  form.passengerName.value = state.user?.name || '';
  $('#book-summary').innerHTML = `<strong>${flight.flight_number}</strong> · ${flight.origin} → ${flight.destination}
    · ${fmtTime(flight.depart_time)} · <strong>$${flight.price}</strong>`;
  show('book');
}

async function submitBooking(event) {
  event.preventDefault();
  const data = Object.fromEntries(new FormData(event.target));
  data.flightId = Number(data.flightId);
  data.totalPrice = Number(data.totalPrice);
  try {
    const { booking } = await api('/api/bookings', { method: 'POST', body: JSON.stringify(data) });
    toast(`Booked! Reference ${booking.reference}`);
    await refreshUser();
    showDetail(booking.reference);
  } catch (err) {
    toast(err.message);
  }
}

async function loadTrips() {
  const el = $('#trips');
  if (!state.token) {
    el.innerHTML = '<div class="empty">Sign in to see your trips.</div>';
    return;
  }
  const { bookings } = await api('/api/bookings');
  if (!bookings.length) {
    el.innerHTML = '<div class="empty">No trips yet. Time to book one!</div>';
    return;
  }
  el.innerHTML = bookings.map((b) => `
    <div class="trip">
      <div>
        <strong>${b.reference}</strong> · ${b.flight_number} · ${b.origin} → ${b.destination}<br/>
        <span class="muted">${fmtTime(b.depart_time)} · ${b.passenger_name}</span>
      </div>
      <div>
        <span class="status ${b.status}">${b.status}</span>
        <button class="btn ghost" data-detail="${b.reference}">View</button>
      </div>
    </div>`).join('');
}

async function showDetail(reference) {
  const { booking: b } = await api(`/api/bookings/${reference}`);
  $('#detail').innerHTML = `
    <h2>Booking ${b.reference} <span class="status ${b.status}">${b.status}</span></h2>
    <div class="summary">
      <strong>${b.flight_number}</strong> · ${b.origin} → ${b.destination} · ${fmtTime(b.depart_time)}
    </div>
    <p><strong>Passenger:</strong> ${b.passenger_name}</p>
    <p><strong>Paid:</strong> $${b.total_price} with card ending ${b.card_number.slice(-4)}</p>
    <p><strong>Notes:</strong> ${b.notes || '<span class="muted">None</span>'}</p>
    <div class="actions">
      <button class="btn ghost" data-view="trips">Back to trips</button>
      <span>
        <button class="btn ghost" data-pass="${b.reference}.txt">Boarding pass</button>
        ${b.status === 'confirmed' ? `<button class="btn primary" data-cancel="${b.reference}">Cancel booking</button>` : ''}
      </span>
    </div>`;
  show('detail');
}

async function downloadPass(file) {
  const text = await api(`/api/bookings/boarding-pass?file=${file}`);
  const win = window.open('', '_blank');
  win.document.write(`<pre>${text}</pre>`);
}

async function cancelBooking(reference) {
  if (!confirm(`Cancel booking ${reference}?`)) return;
  await api(`/api/bookings/${reference}/cancel`, { method: 'POST' });
  toast('Booking cancelled');
  showDetail(reference);
}

async function refreshUser() {
  const { user } = await api('/api/users/me');
  setSession(state.token, user);
}

function openAuth() {
  state.registering = false;
  syncAuthMode();
  $('#auth-error').textContent = '';
  $('#auth-dialog').showModal();
}

function syncAuthMode() {
  $('#auth-title').textContent = state.registering ? 'Create your Fly-Lo account' : 'Sign in';
  $('#auth-submit').textContent = state.registering ? 'Create account' : 'Sign in';
  $('#auth-toggle').textContent = state.registering ? 'I already have an account' : 'Create an account';
  document.querySelectorAll('.register-only').forEach((el) => el.classList.toggle('hidden', !state.registering));
}

async function submitAuth(event) {
  event.preventDefault();
  const data = Object.fromEntries(new FormData(event.target));
  const path = state.registering ? '/api/auth/register' : '/api/auth/login';
  try {
    const { token, user } = await api(path, { method: 'POST', body: JSON.stringify(data) });
    setSession(token, user);
    $('#auth-dialog').close();
    toast(`Welcome, ${user.name}`);
  } catch (err) {
    $('#auth-error').textContent = err.message;
  }
}

document.addEventListener('click', (event) => {
  const t = event.target.closest('[data-view],[data-book],[data-detail],[data-cancel],[data-pass]');
  if (!t) return;
  if (t.dataset.view) {
    event.preventDefault();
    show(t.dataset.view);
  } else if (t.dataset.book) {
    startBooking(JSON.parse(t.dataset.book));
  } else if (t.dataset.detail) {
    showDetail(t.dataset.detail);
  } else if (t.dataset.cancel) {
    cancelBooking(t.dataset.cancel);
  } else if (t.dataset.pass) {
    downloadPass(t.dataset.pass);
  }
});

$('#auth-btn').addEventListener('click', () => (state.user ? setSession(null, null) : openAuth()));
$('#auth-toggle').addEventListener('click', () => {
  state.registering = !state.registering;
  syncAuthMode();
});
$('#auth-form').addEventListener('submit', submitAuth);
$('#search-form').addEventListener('submit', search);
$('#book-form').addEventListener('submit', submitBooking);

renderAuth();
loadAirports().then(search);
if (location.hash === '#trips') show('trips');
```

### public/index.html

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Fly-Lo Air | Book low-fare flights</title>
  <link rel="stylesheet" href="/styles.css" />
</head>
<body>
  <header class="topbar">
    <div class="brand"><span class="logo">✈</span> Fly-Lo <span class="muted">Air</span></div>
    <nav>
      <a href="#search" data-view="search">Book</a>
      <a href="#trips" data-view="trips">My trips</a>
      <span id="user-badge" class="badge hidden"></span>
      <button id="auth-btn" class="btn ghost">Sign in</button>
    </nav>
  </header>

  <main>
    <section id="view-search" class="view">
      <div class="hero">
        <h1>Low fares. Big skies.</h1>
        <p>Search Fly-Lo flights across Canada and beyond.</p>
        <form id="search-form" class="search">
          <label>From<select name="from" id="from"></select></label>
          <label>To<select name="to" id="to"></select></label>
          <label>Date<input type="date" name="date" id="date" /></label>
          <button class="btn primary" type="submit">Search flights</button>
        </form>
      </div>
      <div id="results" class="results"></div>
    </section>

    <section id="view-book" class="view hidden">
      <div class="card">
        <h2>Passenger &amp; payment</h2>
        <div id="book-summary" class="summary"></div>
        <form id="book-form" class="grid">
          <label>Passenger name<input name="passengerName" required /></label>
          <label>Passport number<input name="passportNumber" required /></label>
          <label>Card number<input name="cardNumber" inputmode="numeric" required /></label>
          <label>Expiry (MM/YY)<input name="cardExpiry" placeholder="08/28" required /></label>
          <label>CVV<input name="cardCvv" inputmode="numeric" required /></label>
          <label class="wide">Notes for the crew<textarea name="notes" rows="2" placeholder="Seat preferences, special assistance…"></textarea></label>
          <input type="hidden" name="flightId" />
          <input type="hidden" name="totalPrice" />
          <div class="wide actions">
            <button type="button" class="btn ghost" data-view="search">Back</button>
            <button class="btn primary" type="submit">Pay &amp; book</button>
          </div>
        </form>
      </div>
    </section>

    <section id="view-trips" class="view hidden">
      <div class="card">
        <h2>My trips</h2>
        <div id="trips"></div>
      </div>
    </section>

    <section id="view-detail" class="view hidden">
      <div class="card" id="detail"></div>
    </section>
  </main>

  <dialog id="auth-dialog">
    <form id="auth-form" method="dialog">
      <h2 id="auth-title">Sign in</h2>
      <label class="register-only hidden">Full name<input name="name" /></label>
      <label>Email<input name="email" type="email" required /></label>
      <label>Password<input name="password" type="password" required /></label>
      <p id="auth-error" class="error"></p>
      <div class="actions">
        <button type="button" id="auth-toggle" class="btn link">Create an account</button>
        <button type="submit" class="btn primary" id="auth-submit">Sign in</button>
      </div>
    </form>
  </dialog>

  <div id="toast" class="toast hidden"></div>
  <script src="/app.js"></script>
</body>
</html>
```

### public/styles.css

```css
:root {
  --brand: #ff5a1f;
  --brand-dark: #d9450f;
  --ink: #1d2433;
  --muted: #6b7280;
  --bg: #f5f6fa;
  --card: #ffffff;
  --line: #e5e7eb;
  --radius: 12px;
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
  color: var(--ink);
}

* { box-sizing: border-box; }
body { margin: 0; background: var(--bg); }
.hidden { display: none !important; }
.muted { color: var(--muted); font-weight: 400; }

.topbar {
  display: flex; justify-content: space-between; align-items: center;
  padding: 14px 32px; background: var(--card); border-bottom: 1px solid var(--line);
  position: sticky; top: 0; z-index: 5;
}
.brand { font-size: 22px; font-weight: 800; letter-spacing: -0.5px; }
.logo { color: var(--brand); }
nav { display: flex; gap: 18px; align-items: center; }
nav a { color: var(--ink); text-decoration: none; font-weight: 600; }
nav a:hover { color: var(--brand); }
.badge { background: #fff1ea; color: var(--brand-dark); padding: 4px 10px; border-radius: 999px; font-size: 13px; }

main { max-width: 980px; margin: 0 auto; padding: 24px 16px 64px; }

.hero {
  background: linear-gradient(135deg, var(--brand), #ff9a3c);
  color: white; border-radius: var(--radius); padding: 36px 32px;
}
.hero h1 { margin: 0 0 6px; font-size: 34px; }
.hero p { margin: 0 0 22px; opacity: 0.9; }

.search { display: grid; grid-template-columns: 1fr 1fr 1fr auto; gap: 12px; align-items: end; }
.search label { color: white; font-size: 13px; font-weight: 600; }
label { display: flex; flex-direction: column; gap: 6px; font-size: 14px; font-weight: 600; }
input, select, textarea {
  font: inherit; padding: 10px 12px; border: 1px solid var(--line);
  border-radius: 8px; background: white; color: var(--ink);
}

.btn { font: inherit; font-weight: 700; border: 0; border-radius: 8px; padding: 10px 18px; cursor: pointer; }
.btn.primary { background: var(--ink); color: white; }
.btn.primary:hover { background: #000; }
.btn.ghost { background: transparent; border: 1px solid var(--line); color: var(--ink); }
.btn.link { background: none; color: var(--brand-dark); padding: 0; }

.results { margin-top: 20px; display: flex; flex-direction: column; gap: 10px; }
.flight {
  display: grid; grid-template-columns: 1fr 2fr 1fr auto; gap: 16px; align-items: center;
  background: var(--card); padding: 16px 20px; border-radius: var(--radius); border: 1px solid var(--line);
}
.flight .num { font-weight: 800; }
.flight .route { font-size: 18px; font-weight: 700; }
.flight .time { color: var(--muted); font-size: 14px; }
.flight .price { font-size: 22px; font-weight: 800; color: var(--brand-dark); }

.card { background: var(--card); border-radius: var(--radius); border: 1px solid var(--line); padding: 24px 28px; }
.card h2 { margin-top: 0; }
.summary { background: var(--bg); padding: 12px 16px; border-radius: 8px; margin-bottom: 18px; }
.grid { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; }
.wide { grid-column: 1 / -1; }
.actions { display: flex; justify-content: space-between; gap: 12px; align-items: center; }

.trip { display: flex; justify-content: space-between; align-items: center; padding: 14px 0; border-bottom: 1px solid var(--line); }
.trip:last-child { border-bottom: 0; }
.status { text-transform: uppercase; font-size: 12px; font-weight: 800; letter-spacing: 0.5px; }
.status.confirmed { color: #15803d; }
.status.cancelled { color: #b91c1c; }
.empty { color: var(--muted); padding: 24px 0; text-align: center; }

dialog { border: 0; border-radius: var(--radius); padding: 28px; width: min(420px, 92vw); }
dialog::backdrop { background: rgba(15, 23, 42, 0.5); }
dialog form { display: flex; flex-direction: column; gap: 14px; }
dialog h2 { margin: 0; }
.error { color: #b91c1c; margin: 0; min-height: 1em; font-size: 14px; }

.toast {
  position: fixed; bottom: 24px; left: 50%; transform: translateX(-50%);
  background: var(--ink); color: white; padding: 12px 20px; border-radius: 8px;
}

@media (max-width: 720px) {
  .search, .grid, .flight { grid-template-columns: 1fr; }
}
```

### src/app.js

```javascript
const express = require('express');
const cors = require('cors');
const config = require('./config');

const app = express();

app.use(cors({ origin: true, credentials: true }));
app.use(express.json());
app.use(express.static(config.publicDir));

app.get('/api/health', (req, res) => res.json({ status: 'ok', env: config.env }));
app.use('/api/auth', require('./routes/auth'));
app.use('/api/flights', require('./routes/flights'));
app.use('/api/bookings', require('./routes/bookings'));
app.use('/api/users', require('./routes/users'));
app.use('/api/admin', require('./routes/admin'));

app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: err.message, stack: err.stack });
});

module.exports = app;
```

### src/auth.js

```javascript
const jwt = require('jsonwebtoken');
const config = require('./config');

function signToken(user) {
  return jwt.sign(
    { sub: user.id, email: user.email, name: user.name, role: user.role },
    config.jwtSecret,
    { expiresIn: '30d' }
  );
}

function bearer(req) {
  const header = req.headers.authorization || '';
  return header.startsWith('Bearer ') ? header.slice(7) : null;
}

function requireAuth(req, res, next) {
  const token = bearer(req);
  if (!token) return res.status(401).json({ error: 'Authentication required' });
  try {
    req.user = jwt.verify(token, config.jwtSecret);
    next();
  } catch (err) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

function requireAdmin(req, res, next) {
  const token = bearer(req);
  if (!token) return res.status(401).json({ error: 'Authentication required' });
  const payload = jwt.decode(token);
  if (!payload || payload.role !== 'admin') {
    return res.status(403).json({ error: 'Admin access required' });
  }
  req.user = payload;
  next();
}

module.exports = { signToken, requireAuth, requireAdmin };
```

### src/config.js

```javascript
const path = require('path');
const fs = require('fs');

const envFile = path.join(__dirname, '..', '.env');
if (fs.existsSync(envFile)) {
  process.loadEnvFile(envFile);
}

const root = path.join(__dirname, '..');

module.exports = {
  port: Number(process.env.PORT) || 3000,
  env: process.env.NODE_ENV || 'development',
  jwtSecret: process.env.JWT_SECRET || 'flylo-secret',
  dbPath: process.env.DB_PATH === ':memory:'
    ? ':memory:'
    : path.resolve(root, process.env.DB_PATH || 'data/flylo.db'),
  paymentGatewayKey: process.env.PAYMENT_GATEWAY_KEY,
  adminEmail: process.env.ADMIN_EMAIL || 'admin@flylo-air.com',
  adminPassword: process.env.ADMIN_PASSWORD || 'admin123',
  boardingPassDir: path.join(root, 'boarding-passes'),
  publicDir: path.join(root, 'public'),
};
```

### src/db.js

```javascript
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');
const { DatabaseSync } = require('node:sqlite');
const config = require('./config');

function hashPassword(password) {
  return crypto.createHash('md5').update(password).digest('hex');
}

function open() {
  if (config.dbPath !== ':memory:') {
    fs.mkdirSync(path.dirname(config.dbPath), { recursive: true });
  }
  const db = new DatabaseSync(config.dbPath);
  db.exec(`
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      email TEXT UNIQUE NOT NULL,
      password TEXT NOT NULL,
      name TEXT NOT NULL,
      phone TEXT,
      role TEXT NOT NULL DEFAULT 'customer',
      loyalty_points INTEGER NOT NULL DEFAULT 0,
      created_at TEXT NOT NULL DEFAULT (datetime('now'))
    );

    CREATE TABLE IF NOT EXISTS flights (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      flight_number TEXT NOT NULL,
      origin TEXT NOT NULL,
      destination TEXT NOT NULL,
      depart_time TEXT NOT NULL,
      arrive_time TEXT NOT NULL,
      price REAL NOT NULL,
      seats_available INTEGER NOT NULL
    );

    CREATE TABLE IF NOT EXISTS bookings (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      reference TEXT UNIQUE NOT NULL,
      user_id INTEGER NOT NULL REFERENCES users(id),
      flight_id INTEGER NOT NULL REFERENCES flights(id),
      passenger_name TEXT NOT NULL,
      passport_number TEXT NOT NULL,
      card_number TEXT NOT NULL,
      card_expiry TEXT NOT NULL,
      card_cvv TEXT NOT NULL,
      total_price REAL NOT NULL,
      notes TEXT,
      status TEXT NOT NULL DEFAULT 'confirmed',
      created_at TEXT NOT NULL DEFAULT (datetime('now'))
    );
  `);
  seed(db);
  return db;
}

const ROUTES = [
  ['YYZ', 'YVR', 189, 300, 290],
  ['YVR', 'YYZ', 179, 300, 260],
  ['YYZ', 'YUL', 89, 70, 75],
  ['YUL', 'YYZ', 85, 70, 75],
  ['YYZ', 'JFK', 129, 90, 95],
  ['JFK', 'YYZ', 135, 90, 95],
  ['YUL', 'CDG', 449, 420, 435],
  ['YYZ', 'LHR', 499, 420, 445],
  ['YVR', 'LAX', 159, 160, 170],
  ['YYC', 'YYZ', 149, 220, 230],
];

function seed(db) {
  const { count } = db.prepare('SELECT COUNT(*) AS count FROM users').get();
  if (count > 0) return;

  const insertUser = db.prepare(
    'INSERT INTO users (email, password, name, phone, role, loyalty_points) VALUES (?, ?, ?, ?, ?, ?)'
  );
  insertUser.run(config.adminEmail, hashPassword(config.adminPassword), 'Fly-Lo Admin', '+1-416-555-0100', 'admin', 0);
  insertUser.run('alice@example.com', hashPassword('password1'), 'Alice Martin', '+1-416-555-0142', 'customer', 12400);
  insertUser.run('bob@example.com', hashPassword('qwerty'), 'Bob Singh', '+1-604-555-0199', 'customer', 3100);

  const insertFlight = db.prepare(
    'INSERT INTO flights (flight_number, origin, destination, depart_time, arrive_time, price, seats_available) VALUES (?, ?, ?, ?, ?, ?, ?)'
  );
  const start = new Date();
  start.setUTCHours(0, 0, 0, 0);
  let n = 100;
  for (let day = 1; day <= 30; day++) {
    for (const [origin, destination, price, minutes] of ROUTES) {
      for (const hour of [7, 13, 19]) {
        const depart = new Date(start.getTime() + day * 86400000 + hour * 3600000);
        const arrive = new Date(depart.getTime() + minutes * 60000);
        const fare = Math.round(price * (1 + ((day * 7 + hour) % 11) / 20));
        insertFlight.run(
          `FL${n++}`,
          origin,
          destination,
          depart.toISOString(),
          arrive.toISOString(),
          fare,
          40 + ((day * hour) % 120)
        );
        if (n > 999) n = 100;
      }
    }
  }

  const insertBooking = db.prepare(`
    INSERT INTO bookings (reference, user_id, flight_id, passenger_name, passport_number,
      card_number, card_expiry, card_cvv, total_price, notes)
    VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
  `);
  insertBooking.run('FL1001', 2, 1, 'Alice Martin', 'GA123456', '4111111111111111', '08/28', '123', 189, 'Window seat please');
  insertBooking.run('FL1002', 3, 15, 'Bob Singh', 'HB987654', '5555555555554444', '11/27', '456', 129, 'Travelling with a guitar');
  insertBooking.run('FL1003', 2, 22, 'Alice Martin', 'GA123456', '4111111111111111', '08/28', '123', 449, null);
}

const db = open();

module.exports = { db, hashPassword };
```

### src/routes/admin.js

```javascript
const express = require('express');
const { db } = require('../db');
const { requireAdmin } = require('../auth');
const config = require('../config');

const router = express.Router();

router.get('/users', requireAdmin, (req, res) => {
  res.json({ users: db.prepare('SELECT * FROM users ORDER BY id').all() });
});

router.get('/bookings', requireAdmin, (req, res) => {
  const bookings = db.prepare(`
    SELECT b.*, u.email, f.flight_number, f.origin, f.destination, f.depart_time
    FROM bookings b
    JOIN users u ON u.id = b.user_id
    JOIN flights f ON f.id = b.flight_id
    ORDER BY b.id DESC
  `).all();
  res.json({ bookings });
});

router.put('/flights/:id/price', requireAdmin, (req, res) => {
  const { price } = req.body;
  db.prepare('UPDATE flights SET price = ? WHERE id = ?').run(price, req.params.id);
  res.json({ flight: db.prepare('SELECT * FROM flights WHERE id = ?').get(req.params.id) });
});

router.get('/debug/config', (req, res) => {
  res.json({ config, env: process.env });
});

module.exports = router;
```

### src/routes/auth.js

```javascript
const express = require('express');
const { db, hashPassword } = require('../db');
const { signToken } = require('../auth');

const router = express.Router();

router.post('/register', (req, res) => {
  const { email, password, name, phone, role } = req.body;
  if (!email || !password || !name) {
    return res.status(400).json({ error: 'email, password and name are required' });
  }

  const existing = db.prepare('SELECT id FROM users WHERE email = ?').get(email);
  if (existing) return res.status(409).json({ error: 'An account with that email already exists' });

  const result = db
    .prepare('INSERT INTO users (email, password, name, phone, role) VALUES (?, ?, ?, ?, ?)')
    .run(email, hashPassword(password), name, phone || null, role || 'customer');

  const user = db.prepare('SELECT * FROM users WHERE id = ?').get(result.lastInsertRowid);
  res.status(201).json({ token: signToken(user), user });
});

router.post('/login', (req, res) => {
  const { email, password } = req.body;
  const user = db.prepare('SELECT * FROM users WHERE email = ?').get(email);
  if (!user) return res.status(401).json({ error: 'No account found for that email' });
  if (user.password !== hashPassword(password || '')) {
    return res.status(401).json({ error: 'Incorrect password' });
  }
  console.log(`login ok email=${email} password=${password}`);
  res.json({ token: signToken(user), user });
});

module.exports = router;
```

### src/routes/bookings.js

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');
const { db } = require('../db');
const { requireAuth } = require('../auth');
const config = require('../config');

const router = express.Router();

function nextReference() {
  const { maxId } = db.prepare('SELECT COALESCE(MAX(id), 0) AS maxId FROM bookings').get();
  return `FL${1001 + maxId}`;
}

function writeBoardingPass(booking, flight) {
  fs.mkdirSync(config.boardingPassDir, { recursive: true });
  const body = [
    'FLY-LO AIR  BOARDING PASS',
    '-------------------------',
    `Reference : ${booking.reference}`,
    `Passenger : ${booking.passenger_name}`,
    `Flight    : ${flight.flight_number}`,
    `Route     : ${flight.origin} -> ${flight.destination}`,
    `Departs   : ${flight.depart_time}`,
    '',
  ].join('\n');
  fs.writeFileSync(path.join(config.boardingPassDir, `${booking.reference}.txt`), body);
}

router.post('/', requireAuth, (req, res) => {
  const { flightId, passengerName, passportNumber, cardNumber, cardExpiry, cardCvv, totalPrice, notes } = req.body;
  if (!flightId || !passengerName || !passportNumber || !cardNumber || !cardExpiry || !cardCvv) {
    return res.status(400).json({ error: 'Missing required booking fields' });
  }

  const flight = db.prepare('SELECT * FROM flights WHERE id = ?').get(flightId);
  if (!flight) return res.status(404).json({ error: 'Flight not found' });
  if (flight.seats_available < 1) return res.status(409).json({ error: 'Flight is full' });

  const reference = nextReference();
  db.prepare(`
    INSERT INTO bookings (reference, user_id, flight_id, passenger_name, passport_number,
      card_number, card_expiry, card_cvv, total_price, notes)
    VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
  `).run(
    reference,
    req.user.sub,
    flight.id,
    passengerName,
    passportNumber,
    cardNumber,
    cardExpiry,
    cardCvv,
    totalPrice ?? flight.price,
    notes || null
  );
  db.prepare('UPDATE flights SET seats_available = seats_available - 1 WHERE id = ?').run(flight.id);
  db.prepare('UPDATE users SET loyalty_points = loyalty_points + ? WHERE id = ?')
    .run(Math.round(totalPrice ?? flight.price), req.user.sub);

  const booking = db.prepare('SELECT * FROM bookings WHERE reference = ?').get(reference);
  writeBoardingPass(booking, flight);
  res.status(201).json({ booking });
});

router.get('/', requireAuth, (req, res) => {
  const bookings = db.prepare(`
    SELECT b.*, f.flight_number, f.origin, f.destination, f.depart_time, f.arrive_time
    FROM bookings b JOIN flights f ON f.id = b.flight_id
    WHERE b.user_id = ?
    ORDER BY b.created_at DESC
  `).all(req.user.sub);
  res.json({ bookings });
});

router.get('/boarding-pass', requireAuth, (req, res) => {
  const file = path.join(config.boardingPassDir, req.query.file || '');
  fs.readFile(file, 'utf8', (err, contents) => {
    if (err) return res.status(404).json({ error: `Could not read ${file}` });
    res.type('text/plain').send(contents);
  });
});

router.get('/:reference', requireAuth, (req, res) => {
  const booking = db.prepare(`
    SELECT b.*, f.flight_number, f.origin, f.destination, f.depart_time, f.arrive_time
    FROM bookings b JOIN flights f ON f.id = b.flight_id
    WHERE b.reference = ?
  `).get(req.params.reference);
  if (!booking) return res.status(404).json({ error: 'Booking not found' });
  res.json({ booking });
});

router.post('/:reference/cancel', requireAuth, (req, res) => {
  const booking = db.prepare('SELECT * FROM bookings WHERE reference = ? AND user_id = ?')
    .get(req.params.reference, req.user.sub);
  if (!booking) return res.status(404).json({ error: 'Booking not found' });
  if (booking.status === 'cancelled') return res.status(409).json({ error: 'Booking already cancelled' });

  db.prepare("UPDATE bookings SET status = 'cancelled' WHERE id = ?").run(booking.id);
  db.prepare('UPDATE flights SET seats_available = seats_available + 1 WHERE id = ?').run(booking.flight_id);
  res.json({ booking: { ...booking, status: 'cancelled' } });
});

module.exports = router;
```

### src/routes/flights.js

```javascript
const express = require('express');
const { db } = require('../db');

const router = express.Router();

router.get('/search', (req, res) => {
  const { from, to, date } = req.query;
  let sql = 'SELECT * FROM flights WHERE seats_available > 0';
  if (from) sql += ` AND origin = '${from}'`;
  if (to) sql += ` AND destination = '${to}'`;
  if (date) sql += ` AND depart_time LIKE '${date}%'`;
  sql += ' ORDER BY depart_time LIMIT 50';

  const flights = db.prepare(sql).all();
  res.json({ flights });
});

router.get('/airports', (req, res) => {
  const rows = db
    .prepare('SELECT origin AS code FROM flights UNION SELECT destination FROM flights ORDER BY code')
    .all();
  res.json({ airports: rows.map((r) => r.code) });
});

router.get('/:id', (req, res) => {
  const flight = db.prepare('SELECT * FROM flights WHERE id = ?').get(req.params.id);
  if (!flight) return res.status(404).json({ error: 'Flight not found' });
  res.json({ flight });
});

module.exports = router;
```

### src/routes/users.js

```javascript
const express = require('express');
const _ = require('lodash');
const { db } = require('../db');
const { requireAuth } = require('../auth');

const router = express.Router();

router.get('/me', requireAuth, (req, res) => {
  const user = db.prepare('SELECT * FROM users WHERE id = ?').get(req.user.sub);
  if (!user) return res.status(404).json({ error: 'User not found' });
  res.json({ user });
});

router.put('/me', requireAuth, (req, res) => {
  const user = db.prepare('SELECT * FROM users WHERE id = ?').get(req.user.sub);
  if (!user) return res.status(404).json({ error: 'User not found' });

  const updated = _.merge({}, user, req.body);
  db.prepare('UPDATE users SET name = ?, phone = ? WHERE id = ?').run(updated.name, updated.phone, user.id);
  res.json({ user: db.prepare('SELECT * FROM users WHERE id = ?').get(user.id) });
});

module.exports = router;
```

### src/server.js

```javascript
const app = require('./app');
const config = require('./config');

app.listen(config.port, () => {
  console.log(`Fly-Lo booking running on http://localhost:${config.port} (${config.env})`);
});
```

### test/api.test.js

```javascript
process.env.DB_PATH = ':memory:';

const { test, before, after } = require('node:test');
const assert = require('node:assert/strict');
const app = require('../src/app');

let server;
let base;

before(async () => {
  server = app.listen(0);
  await new Promise((resolve) => server.once('listening', resolve));
  base = `http://127.0.0.1:${server.address().port}`;
});

after(() => server.close());

async function call(path, { token, ...options } = {}) {
  const headers = { 'Content-Type': 'application/json' };
  if (token) headers.Authorization = `Bearer ${token}`;
  const res = await fetch(base + path, { ...options, headers });
  const body = res.headers.get('content-type')?.includes('json') ? await res.json() : await res.text();
  return { status: res.status, body };
}

async function login(email, password) {
  const { body } = await call('/api/auth/login', {
    method: 'POST',
    body: JSON.stringify({ email, password }),
  });
  return body.token;
}

test('health check responds', async () => {
  const { status, body } = await call('/api/health');
  assert.equal(status, 200);
  assert.equal(body.status, 'ok');
});

test('searches flights by route', async () => {
  const { status, body } = await call('/api/flights/search?from=YYZ&to=YVR');
  assert.equal(status, 200);
  assert.ok(body.flights.length > 0);
  for (const f of body.flights) {
    assert.equal(f.origin, 'YYZ');
    assert.equal(f.destination, 'YVR');
  }
});

test('registers and logs in a customer', async () => {
  const reg = await call('/api/auth/register', {
    method: 'POST',
    body: JSON.stringify({ email: 'carol@example.com', password: 'S3cure!pass', name: 'Carol Chen' }),
  });
  assert.equal(reg.status, 201);
  assert.ok(reg.body.token);

  const token = await login('carol@example.com', 'S3cure!pass');
  assert.ok(token);
});

test('rejects bad credentials', async () => {
  const { status } = await call('/api/auth/login', {
    method: 'POST',
    body: JSON.stringify({ email: 'alice@example.com', password: 'wrong' }),
  });
  assert.equal(status, 401);
});

test('books, lists and cancels a flight', async () => {
  const token = await login('alice@example.com', 'password1');
  const { body: search } = await call('/api/flights/search?from=YYZ&to=YUL');
  const flight = search.flights[0];

  const created = await call('/api/bookings', {
    method: 'POST',
    token,
    body: JSON.stringify({
      flightId: flight.id,
      passengerName: 'Alice Martin',
      passportNumber: 'GA123456',
      cardNumber: '4111111111111111',
      cardExpiry: '08/28',
      cardCvv: '123',
    }),
  });
  assert.equal(created.status, 201);
  const ref = created.body.booking.reference;

  const list = await call('/api/bookings', { token });
  assert.ok(list.body.bookings.some((b) => b.reference === ref));

  const detail = await call(`/api/bookings/${ref}`, { token });
  assert.equal(detail.status, 200);
  assert.equal(detail.body.booking.flight_number, flight.flight_number);

  const pass = await call(`/api/bookings/boarding-pass?file=${ref}.txt`, { token });
  assert.equal(pass.status, 200);
  assert.match(pass.body, /BOARDING PASS/);

  const cancelled = await call(`/api/bookings/${ref}/cancel`, { method: 'POST', token });
  assert.equal(cancelled.status, 200);
  assert.equal(cancelled.body.booking.status, 'cancelled');
});

test('booking requires authentication', async () => {
  const { status } = await call('/api/bookings', { method: 'POST', body: '{}' });
  assert.equal(status, 401);
});

test('admin can list users', async () => {
  const token = await login('admin@flylo-air.com', 'admin123');
  const { status, body } = await call('/api/admin/users', { token });
  assert.equal(status, 200);
  assert.ok(body.users.length >= 3);
});

test('customers cannot use admin endpoints', async () => {
  const token = await login('bob@example.com', 'qwerty');
  const { status } = await call('/api/admin/users', { token });
  assert.equal(status, 403);
});
```

### demo/RUNBOOK.md

Not present on the demo-baseline tag.

````markdown
# Fly-Lo security walkthrough — demo fixtures

> **Demo fixtures only.** The files under `demo/scans/` and `demo/logs/` stand in for **Snyk** and **AWS CloudWatch Logs Insights**. They are not connected to Snyk or AWS; no scanner or live log export was used for this walkthrough.

## Assets

| Asset | Path | Purpose |
| --- | --- | --- |
| Snyk-shaped export | `demo/scans/snyk-report.json` | Prioritized code, dependency, and secret findings |
| CloudWatch-shaped export | `demo/logs/cloudwatch-flylo-booking-api.json` | Time-windowed API request / error narrative |
| Generator | `demo/tools/generate-fixtures.js` | Regenerates both exports deterministically |
| Vulnerable app baseline | Git tag `demo-baseline` | Known-good snapshot of intentionally vulnerable `main` |

## Hero vulnerability (SQL injection)

| Field | Detail |
| --- | --- |
| Finding | `SNYK-JS-FLYLO-SQLI-0001` |
| Route | `GET /api/flights/search` |
| Code | `src/routes/flights.js` lines **9–14** (source: `req.query` line 7; sink: `db.prepare(sql)` line 14) |
| Fix | Build the query with bound `?` parameters for `from`, `to`, and `date` instead of string concatenation. |

## Log clusters (`demoClusters`)

| Cluster | Count | What it shows | Code / route refs |
| --- | ---: | --- | --- |
| `normal` | 140 | Benign traffic | Mixed `/api/*` |
| `sqli-probe` | 26 | sqlmap-style probes → 500 syntax error | `src/routes/flights.js:14` |
| `sqli-tautology` | 10 | `' OR 1=1--` → 200, 50 rows | `GET /api/flights/search` |
| `unauth-admin-endpoint` | 12 | Unauthenticated config dump | `src/routes/admin.js:29-31` |
| `forged-jwt` | 9 | `alg:none` admin token (7 reads + 2 fare PUTs to $1) | `src/auth.js:31` |
| `path-traversal` | 6 | `?file=../.env`, `/etc/passwd`, etc. | `src/routes/bookings.js:77-78` |
| `credential-in-logs` | 4 | Plaintext passwords in login log lines | `src/routes/auth.js:31` |
| `performance` | 6 | Slow `/api/admin/bookings` (2.9–5.0s) | `src/routes/admin.js:13` |
| `reliability` | 4 | `SQLITE_BUSY` on booking create | `src/routes/bookings.js:45` |

**Total events:** 217 (see `eventCount` in the CloudWatch fixture).

## Regenerate fixtures

```bash
node demo/tools/generate-fixtures.js
```

## Reset between walkthroughs

1. **Close open PRs** whose head branch name starts with `sec/` **without merging**.
2. **Delete** remote and local `sec/*` branches.
3. **Restore app code** to `demo-baseline` if a fix was merged (keeps `demo/` as on `main`):

   ```bash
   git fetch origin --tags && git checkout main && git pull && \
     git restore --source=demo-baseline --staged --worktree -- . ':!demo' && \
     git commit -m "Reset demo to demo-baseline" && git push origin main
   ```

   (No force-push.)

4. In Linear team **Solutions Demo (SOL2)**, cancel open issues whose title contains `[daxko-demo]`.
5. Confirm a clean app diff vs baseline (excluding demo assets):

   ```bash
   git diff demo-baseline main -- . ':!demo'
   ```

   Output should be empty.
````

### demo/tools/generate-fixtures.js

Not present on the demo-baseline tag. Regenerates demo/scans and demo/logs.

```javascript
#!/usr/bin/env node
'use strict';

const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

const ROOT = path.join(__dirname, '..', '..');
const OUT_SCAN = path.join(ROOT, 'demo', 'scans', 'snyk-report.json');
const OUT_LOGS = path.join(ROOT, 'demo', 'logs', 'cloudwatch-flylo-booking-api.json');

/** @param {number} seed */
function mulberry32(seed) {
  let s = seed >>> 0;
  return function next() {
    s += 0x6d2b79f5;
    let t = s;
    t = Math.imul(t ^ (t >>> 15), t | 1);
    t ^= t + Math.imul(t ^ (t >>> 7), t | 61);
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  };
}

const rand = mulberry32(0xf17c9a2b);

function pickIp(clusterSalt) {
  const r = mulberry32(0x9e3779b9 ^ clusterSalt);
  const use203 = r() < 0.5;
  const third = Math.floor(r() * 254) + 1;
  const fourth = Math.floor(r() * 254) + 1;
  if (use203) return `203.0.113.${third}`;
  return `198.51.100.${fourth}`;
}

function isoAddMs(baseIso, ms) {
  const d = new Date(baseIso);
  return new Date(d.getTime() + ms).toISOString().replace(/\.\d{3}Z$/, 'Z');
}

function snykFindings() {
  return [
    {
      id: 'SNYK-JS-FLYLO-SQLI-0001',
      title: 'SQL injection via flight search query parameters',
      severity: 'critical',
      cwe: ['CWE-89'],
      type: 'code',
      file: 'src/routes/flights.js',
      line: 9,
      endLine: 14,
      message:
        'User-controlled query parameters from, to, and date are concatenated into SQL without parameterization.',
      fix: 'Use bound parameters (? placeholders) for all filters instead of string concatenation.',
      cvssScore: 9.8,
      route: 'GET /api/flights/search',
    },
    {
      id: 'SNYK-JS-FLYLO-JWT-0002',
      title: 'JWT accepted without signature verification in admin middleware',
      severity: 'critical',
      cwe: ['CWE-347'],
      type: 'code',
      file: 'src/auth.js',
      line: 31,
      endLine: 31,
      message: 'requireAdmin uses jwt.decode instead of jwt.verify, allowing forged tokens.',
      fix: 'Verify the JWT with jwt.verify and the configured secret before trusting claims.',
      cvssScore: 9.1,
      route: 'GET /api/admin/*',
    },
    {
      id: 'SNYK-JS-FLYLO-AUTHZ-0003',
      title: 'Unauthenticated admin debug endpoint exposes configuration',
      severity: 'critical',
      cwe: ['CWE-306', 'CWE-200'],
      type: 'code',
      file: 'src/routes/admin.js',
      line: 29,
      endLine: 31,
      message: 'GET /api/admin/debug/config returns application config and process.env without authentication.',
      fix: 'Remove the debug route in production or protect it with strong admin authentication.',
      cvssScore: 9.3,
      route: 'GET /api/admin/debug/config',
    },
    {
      id: 'SNYK-JS-FLYLO-SECRET-0004',
      title: 'Secrets committed in .env (JWT_SECRET, PAYMENT_GATEWAY_KEY, ADMIN_PASSWORD)',
      severity: 'critical',
      cwe: ['CWE-798', 'CWE-312'],
      type: 'secret',
      file: '.env',
      line: 3,
      endLine: 7,
      message:
        'Repository contains a .env file with JWT_SECRET (line 3), PAYMENT_GATEWAY_KEY (line 5), and ADMIN_PASSWORD (line 7). Values are omitted in this report.',
      fix: 'Remove .env from version control, rotate all exposed secrets, and use a secret manager.',
      cvssScore: 9.0,
    },
    {
      id: 'SNYK-JS-FLYLO-MASS-0005',
      title: 'Mass assignment allows self-granting admin role at registration',
      severity: 'high',
      cwe: ['CWE-915'],
      type: 'code',
      file: 'src/routes/auth.js',
      line: 8,
      endLine: 18,
      message: 'The role field from req.body is persisted when creating a new user.',
      fix: 'Ignore client-supplied role and always assign customer unless provisioned by an admin.',
      cvssScore: 8.2,
      route: 'POST /api/auth/register',
    },
    {
      id: 'SNYK-JS-FLYLO-IDOR-0006',
      title: 'Insecure direct object reference on booking lookup',
      severity: 'high',
      cwe: ['CWE-639'],
      type: 'code',
      file: 'src/routes/bookings.js',
      line: 84,
      endLine: 91,
      message: 'Booking is returned by reference without verifying it belongs to the authenticated user.',
      fix: 'Add AND b.user_id = ? using req.user.sub in the query.',
      cvssScore: 7.5,
      route: 'GET /api/bookings/:reference',
    },
    {
      id: 'SNYK-JS-FLYLO-PATH-0007',
      title: 'Path traversal in boarding pass download',
      severity: 'high',
      cwe: ['CWE-22'],
      type: 'code',
      file: 'src/routes/bookings.js',
      line: 77,
      endLine: 78,
      message: 'req.query.file is joined to the boarding pass directory without sanitization.',
      fix: 'Resolve paths and ensure the result stays within boardingPassDir; use an allowlist of references.',
      cvssScore: 7.7,
      route: 'GET /api/bookings/boarding-pass',
    },
    {
      id: 'SNYK-JS-LODASH-567746',
      title: 'Prototype pollution in lodash (CVE-2020-8203, CVE-2021-23337, CVE-2020-28500)',
      severity: 'high',
      cwe: ['CWE-1321'],
      type: 'open-source',
      file: 'package.json',
      line: 19,
      endLine: 19,
      message: 'lodash 4.17.15 is vulnerable to prototype pollution; fixed in 4.17.21.',
      fix: 'Upgrade lodash to 4.17.21 or later.',
      cvssScore: 7.4,
    },
    {
      id: 'SNYK-JS-FLYLO-MERGE-0008',
      title: 'Unsafe _.merge of user profile with attacker-controlled body',
      severity: 'high',
      cwe: ['CWE-915'],
      type: 'code',
      file: 'src/routes/users.js',
      line: 18,
      endLine: 18,
      message: 'lodash merge combines database user row with req.body, enabling prototype pollution via lodash.',
      fix: 'Pick allowed fields explicitly instead of merging req.body; upgrade lodash.',
      cvssScore: 7.1,
      route: 'PUT /api/users/me',
    },
    {
      id: 'SNYK-JS-FLYLO-MD5-0009',
      title: 'Weak unsalted MD5 password hashing',
      severity: 'high',
      cwe: ['CWE-328', 'CWE-916'],
      type: 'code',
      file: 'src/db.js',
      line: 8,
      endLine: 8,
      message: 'Passwords are hashed with MD5 without salt.',
      fix: 'Use bcrypt, scrypt, or Argon2 with a per-user salt.',
      cvssScore: 7.4,
    },
    {
      id: 'SNYK-JS-FLYLO-LOG-0010',
      title: 'Plaintext password logged on successful login',
      severity: 'high',
      cwe: ['CWE-532'],
      type: 'code',
      file: 'src/routes/auth.js',
      line: 31,
      endLine: 31,
      message: 'console.log writes the user password to application logs.',
      fix: 'Remove password from log statements; log only non-sensitive identifiers.',
      cvssScore: 7.2,
      route: 'POST /api/auth/login',
    },
    {
      id: 'SNYK-JS-FLYLO-PCI-0011',
      title: 'Primary account number and CVV stored in database',
      severity: 'high',
      cwe: ['CWE-311', 'CWE-359'],
      type: 'code',
      file: 'src/routes/bookings.js',
      line: 41,
      endLine: 48,
      message: 'Booking flow persists card_number, card_expiry, and card_cvv to SQLite.',
      fix: 'Tokenize card data with a PCI-compliant payment provider; never store CVV.',
      cvssScore: 7.8,
      route: 'POST /api/bookings',
    },
    {
      id: 'SNYK-JS-FLYLO-JWTDEF-0012',
      title: 'Hard-coded fallback JWT secret',
      severity: 'high',
      cwe: ['CWE-798'],
      type: 'code',
      file: 'src/config.js',
      line: 14,
      endLine: 14,
      message: 'jwtSecret falls back to a static default when JWT_SECRET is unset.',
      fix: 'Require JWT_SECRET from environment and fail fast if missing.',
      cvssScore: 7.0,
    },
    {
      id: 'SNYK-JS-FLYLO-PAYCFG-0013',
      title: 'Payment gateway key loaded into application config object',
      severity: 'high',
      cwe: ['CWE-200'],
      type: 'code',
      file: 'src/config.js',
      line: 18,
      endLine: 18,
      message: 'PAYMENT_GATEWAY_KEY is exposed on the config object and may leak via debug endpoints.',
      fix: 'Keep payment keys out of shared config objects returned to clients or logs.',
      cvssScore: 7.1,
    },
    {
      id: 'SNYK-JS-FLYLO-CORS-0015',
      title: 'Permissive CORS (reflect any origin with credentials)',
      severity: 'medium',
      cwe: ['CWE-942'],
      type: 'code',
      file: 'src/app.js',
      line: 7,
      endLine: 7,
      message: 'cors({ origin: true, credentials: true }) allows any Origin header.',
      fix: 'Restrict allowed origins to trusted front-end hosts.',
      cvssScore: 5.4,
    },
    {
      id: 'SNYK-JS-FLYLO-RATE-0016',
      title: 'No rate limiting on authentication or search endpoints',
      severity: 'medium',
      cwe: ['CWE-307'],
      type: 'code',
      file: 'src/app.js',
      line: 8,
      endLine: 16,
      message: 'Express app mounts API routes without throttling or brute-force protection.',
      fix: 'Add rate limiting middleware (e.g. express-rate-limit) on /api/auth and sensitive routes.',
      cvssScore: 5.3,
    },
    {
      id: 'SNYK-JS-FLYLO-ERR-0017',
      title: 'Verbose error responses include stack traces',
      severity: 'medium',
      cwe: ['CWE-209'],
      type: 'code',
      file: 'src/app.js',
      line: 18,
      endLine: 20,
      message: 'Global error handler returns err.stack to API clients.',
      fix: 'Return generic error messages in production; log stacks server-side only.',
      cvssScore: 5.0,
    },
    {
      id: 'SNYK-JS-FLYLO-JWTEXP-0018',
      title: 'Excessive JWT lifetime (30 days)',
      severity: 'medium',
      cwe: ['CWE-613'],
      type: 'code',
      file: 'src/auth.js',
      line: 8,
      endLine: 8,
      message: 'Access tokens are issued with expiresIn 30d.',
      fix: 'Use shorter-lived access tokens with refresh token rotation.',
      cvssScore: 4.8,
    },
    {
      id: 'SNYK-JS-FLYLO-HEALTH-0019',
      title: 'Health endpoint leaks NODE_ENV',
      severity: 'medium',
      cwe: ['CWE-200'],
      type: 'code',
      file: 'src/app.js',
      line: 11,
      endLine: 11,
      message: '/api/health includes env in the JSON response.',
      fix: 'Return only { status: "ok" } from health checks.',
      cvssScore: 4.3,
      route: 'GET /api/health',
    },
    {
      id: 'SNYK-JS-FLYLO-ADMDEF-0020',
      title: 'Default admin credentials in configuration',
      severity: 'low',
      cwe: ['CWE-1392'],
      type: 'code',
      file: 'src/config.js',
      line: 20,
      endLine: 20,
      message: 'adminPassword defaults to a known value when ADMIN_PASSWORD is unset.',
      fix: 'Require ADMIN_PASSWORD in production and remove hard-coded defaults.',
      cvssScore: 3.7,
    },
    {
      id: 'SNYK-JS-FLYLO-DISC-0021',
      title: 'Filesystem path echoed in boarding pass error',
      severity: 'low',
      cwe: ['CWE-200'],
      type: 'code',
      file: 'src/routes/bookings.js',
      line: 79,
      endLine: 79,
      message: 'Error responses include the resolved absolute file path.',
      fix: 'Return a generic not-found message without filesystem details.',
      cvssScore: 3.1,
      route: 'GET /api/bookings/boarding-pass',
    },
  ];
}

function buildSnykReport() {
  const findings = snykFindings();
  const bySeverity = { critical: 0, high: 0, medium: 0, low: 0 };
  for (const f of findings) bySeverity[f.severity]++;

  return {
    note: 'Demo fixture standing in for a Snyk export. No Snyk scanner was run against this repository.',
    org: 'flylo-air',
    project: 'security-checks',
    scannedAt: '2026-11-12T15:05:00Z',
    summary: {
      total: findings.length,
      ...bySeverity,
    },
    findings,
  };
}

function logEvent(ts, stream, message) {
  return {
    '@timestamp': ts,
    '@logStream': stream,
    '@message': JSON.stringify(message),
  };
}

function buildCloudWatchLogs() {
  const windowStart = '2026-11-11T15:30:00Z';
  const windowEnd = '2026-11-12T15:30:00Z';
  const events = [];
  let tick = 0;

  const clusters = {
    normal: 140,
    'sqli-probe': 26,
    'sqli-tautology': 10,
    'unauth-admin-endpoint': 12,
    'forged-jwt': 9,
    'path-traversal': 6,
    'credential-in-logs': 4,
    performance: 6,
    reliability: 4,
  };

  const normalPaths = [
    ['GET', '/api/health', 200],
    ['GET', '/api/flights/airports', 200],
    ['GET', '/api/flights/search', 200],
    ['POST', '/api/auth/login', 200],
    ['GET', '/api/bookings', 200],
    ['POST', '/api/bookings', 201],
    ['GET', '/api/flights/1', 200],
  ];

  for (let i = 0; i < clusters.normal; i++) {
    const [method, p, status] = normalPaths[i % normalPaths.length];
    const ts = isoAddMs(windowStart, tick * 375_000);
    tick++;
    const ip = pickIp(1000 + i);
    const latency = 12 + Math.floor(rand() * 180);
    events.push(
      logEvent(ts, `ecs/flylo-api/${i % 3}`, {
        level: status < 400 ? 'info' : 'warn',
        method,
        path: p,
        status,
        latencyMs: latency,
        ip,
        msg: 'request completed',
        tags: ['normal'],
      })
    );
  }

  const sqliPayloads = [
    "1' AND 1=CONVERT(int,(SELECT @@version))--",
    "YYZ' UNION SELECT sql,sql,sql,sql,sql,sql,sql,sql FROM sqlite_master--",
    "' OR ''='",
    "1'; WAITFOR DELAY '0:0:5'--",
    '-1 OR 1=1',
    "admin'--",
    "1' AND (SELECT COUNT(*) FROM users)>0--",
    "'; DROP TABLE flights;--",
  ];

  for (let i = 0; i < clusters['sqli-probe']; i++) {
    const payload = sqliPayloads[i % sqliPayloads.length];
    const ts = isoAddMs(windowStart, tick * 375_000);
    tick++;
    const ip = pickIp(2000 + i);
    const pathWithQuery = `/api/flights/search?from=${encodeURIComponent(payload)}&to=YVR`;
    events.push(
      logEvent(ts, 'ecs/flylo-api/0', {
        level: 'error',
        method: 'GET',
        path: pathWithQuery,
        status: 500,
        latencyMs: 45 + i,
        ip,
        msg: 'SqliteError: near "admin": syntax error',
        stack: 'SqliteError: near "admin": syntax error\n    at GET /search (src/routes/flights.js:14:35)',
        tags: ['sqli-probe'],
      })
    );
  }

  for (let i = 0; i < clusters['sqli-tautology']; i++) {
    const ts = isoAddMs(windowStart, tick * 375_000);
    tick++;
    const ip = pickIp(3000 + i);
    events.push(
      logEvent(ts, 'ecs/flylo-api/1', {
        level: 'info',
        method: 'GET',
        path: "/api/flights/search?from=' OR 1=1--&to=YVR",
        status: 200,
        latencyMs: 22 + i,
        ip,
        msg: 'request completed rows=50',
        tags: ['sqli-tautology'],
      })
    );
  }

  for (let i = 0; i < clusters['unauth-admin-endpoint']; i++) {
    const ts = isoAddMs(windowStart, tick * 375_000);
    tick++;
    const ip = pickIp(4000 + i);
    events.push(
      logEvent(ts, 'ecs/flylo-api/2', {
        level: 'warn',
        method: 'GET',
        path: '/api/admin/debug/config',
        status: 200,
        latencyMs: 8 + i,
        ip,
        msg: 'unauthenticated config dump',
        tags: ['unauth-admin-endpoint'],
      })
    );
  }

  const forgedPaths = [
    ['/api/admin/users', 'GET', 200],
    ['/api/admin/bookings', 'GET', 200],
    ['/api/admin/users', 'GET', 200],
    ['/api/admin/bookings', 'GET', 200],
    ['/api/admin/users', 'GET', 200],
    ['/api/admin/bookings', 'GET', 200],
    ['/api/admin/users', 'GET', 200],
    ['/api/admin/flights/42/price', 'PUT', 200],
    ['/api/admin/flights/15/price', 'PUT', 200],
  ];

  for (let i = 0; i < clusters['forged-jwt']; i++) {
    const [p, method, status] = forgedPaths[i];
    const ts = isoAddMs(windowStart, tick * 375_000);
    tick++;
    const ip = pickIp(5000 + i);
    const bodyMsg =
      method === 'PUT'
        ? 'fare updated price=1 alg=none token'
        : 'admin data read with alg=none token';
    events.push(
      logEvent(ts, 'ecs/flylo-api/0', {
        level: method === 'PUT' ? 'warn' : 'info',
        method,
        path: p,
        status,
        latencyMs: 14 + i,
        ip,
        msg: bodyMsg,
        tags: ['forged-jwt'],
      })
    );
  }

  const travFiles = ['../.env', '../../.env', '../package.json', '/etc/passwd', '../../../.env', '..\\..\\data\\flylo.db'];
  for (let i = 0; i < clusters['path-traversal']; i++) {
    const file = travFiles[i];
    const ts = isoAddMs(windowStart, tick * 375_000);
    tick++;
    const ip = pickIp(6000 + i);
    const status = i < 2 ? 200 : 404;
    events.push(
      logEvent(ts, 'ecs/flylo-api/1', {
        level: status === 200 ? 'warn' : 'info',
        method: 'GET',
        path: `/api/bookings/boarding-pass?file=${encodeURIComponent(file)}`,
        status,
        latencyMs: 11 + i,
        ip,
        msg: status === 200 ? 'boarding pass served outside intended dir' : 'Could not read file',
        tags: ['path-traversal'],
      })
    );
  }

  const creds = [
    { email: 'alice@example.com', password: 'password1-demo-not-real' },
    { email: 'bob@example.com', password: 'qwerty-demo-not-real' },
    { email: 'admin@flylo-air.com', password: 'admin123-demo-not-real' },
    { email: 'alice@example.com', password: 'password1-demo-not-real' },
  ];

  for (let i = 0; i < clusters['credential-in-logs']; i++) {
    const { email, password } = creds[i];
    const ts = isoAddMs(windowStart, tick * 375_000);
    tick++;
    const ip = pickIp(7000 + i);
    events.push(
      logEvent(ts, 'ecs/flylo-api/2', {
        level: 'info',
        method: 'POST',
        path: '/api/auth/login',
        status: 200,
        latencyMs: 31 + i,
        ip,
        msg: `login ok email=${email} password=${password}`,
        tags: ['credential-in-logs'],
      })
    );
  }

  const perfLatency = [2900, 3100, 3400, 4100, 4700, 5000];
  for (let i = 0; i < clusters.performance; i++) {
    const ts = isoAddMs(windowStart, tick * 375_000);
    tick++;
    const ip = pickIp(8000 + i);
    events.push(
      logEvent(ts, 'ecs/flylo-api/0', {
        level: 'warn',
        method: 'GET',
        path: '/api/admin/bookings',
        status: 200,
        latencyMs: perfLatency[i],
        ip,
        msg: 'slow admin bookings query',
        tags: ['performance'],
        source: 'src/routes/admin.js:13',
      })
    );
  }

  for (let i = 0; i < clusters.reliability; i++) {
    const ts = isoAddMs(windowStart, tick * 375_000);
    tick++;
    const ip = pickIp(9000 + i);
    events.push(
      logEvent(ts, 'ecs/flylo-api/1', {
        level: 'error',
        method: 'POST',
        path: '/api/bookings',
        status: 500,
        latencyMs: 120 + i * 5,
        ip,
        msg: 'SQLITE_BUSY: database is locked',
        stack: 'SQLITE_BUSY: database is locked\n    at POST / (src/routes/bookings.js:45:6)',
        tags: ['reliability'],
      })
    );
  }

  events.sort((a, b) => (a['@timestamp'] < b['@timestamp'] ? -1 : a['@timestamp'] > b['@timestamp'] ? 1 : 0));

  return {
    note: 'Demo fixture standing in for AWS CloudWatch Logs Insights export. No AWS account or live log stream was queried.',
    logGroup: '/ecs/flylo-booking-api',
    window: { start: windowStart, end: windowEnd },
    demoClusters: clusters,
    eventCount: events.length,
    events,
  };
}

function writeOutputs() {
  fs.mkdirSync(path.dirname(OUT_SCAN), { recursive: true });
  fs.mkdirSync(path.dirname(OUT_LOGS), { recursive: true });

  const snyk = buildSnykReport();
  const logs = buildCloudWatchLogs();

  const snykText = `${JSON.stringify(snyk, null, 2)}\n`;
  const logsText = `${JSON.stringify(logs, null, 2)}\n`;

  fs.writeFileSync(OUT_SCAN, snykText, 'utf8');
  fs.writeFileSync(OUT_LOGS, logsText, 'utf8');

  return { snyk, logs, snykText, logsText };
}

function main() {
  const { snyk, logs, snykText, logsText } = writeOutputs();

  const snykMd5 = crypto.createHash('md5').update(snykText).digest('hex');
  const logsMd5 = crypto.createHash('md5').update(logsText).digest('hex');

  console.log('Generated demo fixtures');
  console.log(`  ${path.relative(ROOT, OUT_SCAN)}`);
  console.log(`  ${path.relative(ROOT, OUT_LOGS)}`);
  console.log(
    `Snyk: ${snyk.summary.total} findings (critical=${snyk.summary.critical} high=${snyk.summary.high} medium=${snyk.summary.medium} low=${snyk.summary.low})`
  );
  console.log(`CloudWatch: ${logs.eventCount} events`);
  console.log('demoClusters:', JSON.stringify(logs.demoClusters));
  console.log(`MD5 snyk-report.json: ${snykMd5}`);
  console.log(`MD5 cloudwatch-flylo-booking-api.json: ${logsMd5}`);
}

main();
```

