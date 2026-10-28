# LockedIN

**IS216 — Web Application Development II · Group Project**
Section **G10** · Team **T3**

A web application that helps students discover suitable study locations across Singapore — libraries, cafés, university study areas and other public study spaces.

---

## Links

| | |
|---|---|
| **Source repository** | https://github.com/KhinBhonePyae/WAD2_Project |
| **Deployed application** | ⚠️ **TODO — add the Vercel URL after deployment** (see [Deployment](#deployment)) |
| **API server** | ⚠️ **TODO — add the Render URL after deployment** |

> Planned hosting: the website on **Vercel**, the API on **Render**, data in **MongoDB Atlas**. Fill in both links once the site is live, before the Week 12 deadline (Fri 6 Nov, 9:00am).

---

## Overview

### The problem

Students often waste time travelling to study locations only to discover that the environment is unsuitable due to overcrowding, excessive noise, a lack of power sockets or insufficient seating. Information about study locations is also scattered across platforms such as Google Maps, university websites and social media, making it difficult for students to compare locations based on their individual study requirements.

### The solution

LockedIN provides a centralized platform where users can browse, search and filter study locations based on facilities, study environment, opening hours and location. Study spaces are imported from the NLB and Google Places APIs into our own database, and the app combines them with live weather and a suitability score that rates each place against the user's preferences and the current conditions.

Users can also save places, write reviews, report how busy a place is right now, say when they plan to go, chat with other students and ask an AI study assistant for recommendations.

---

## Features

1. **Accounts and profiles:** sign-up with study preferences, log in and out, edit profile, change password, delete account
2. **Study space catalogue:** browse places, detail pages with a photo carousel, facilities, opening hours and a live "Open now" status in Singapore time
3. **Search, filters and suitability scoring:** filter by facilities, area, crowd level, rating, open now, free entry and indoor or outdoor; each place gets a score and plain-language reasons based on your preferences and the weather
4. **Map and directions:** OneMap map, location search, and walking, driving and public-transport routes
5. **Ratings, reviews and crowd reporting:** star ratings and reviews from real users only; a "How busy is it?" card with one-tap crowd check-ins (change or undo within the hour)
6. **"Planning to go":** say which time slot you'll visit, and see how busy a slot is likely to be
7. **Saved places**
8. **Weather integration:** current Singapore weather feeds into the suitability score
9. **AI study assistant ("Ask Locker"):** recommendations and study questions, powered by Google Gemini
10. **Global chat:** a shared chat room for logged-in users
11. **Report incorrect information** about a place, and **admin pages** to manage study spaces, users and reports
12. **Installable app:** can be added to a phone's home screen

---

## Tech stack

| Layer | Technology |
|---|---|
| Markup & styling | HTML, CSS, **Bootstrap** |
| Front-end framework | **Vue.js 3** with Vue Router (Vite) |
| Map | **Leaflet** with OneMap tiles |
| Animation | GSAP, Lenis smooth scrolling |
| Back-end | **Node.js + Express.js** |
| Database | **MongoDB Atlas + Mongoose** |
| HTTP client | **Axios** (async requests from the client to our API and to external APIs) |
| Version control | **GitHub** |
| Deployment | **Vercel** (website) + **Render** (API) |
| Testing | **Vitest** + Vue Test Utils (unit and component), **Node test runner** (server), **Playwright** (end-to-end) |

---

## External APIs

| API | Use | When it is called | Docs |
|---|---|---|---|
| **NLB Open Web API** | Public library branches (names, addresses, hours) | Once, by `server/import-libraries.js`; results are stored in MongoDB | [NLB Open Web](https://openweb.nlb.gov.sg/) |
| **Google Places API (New)** | Study spaces near SMU: name, address, coordinates, opening hours, photos and Google ratings | Once, by `server/import-google-places.js`; results are stored in MongoDB | [Overview](https://developers.google.com/maps/documentation/places/web-service/op-overview) |
| **OneMap API** | Map tiles, location search and directions | While the app runs (directions go through our server, which logs in to OneMap and refreshes its token automatically) | [OneMap](https://www.onemap.gov.sg/apidocs/) |
| **Open-Meteo Weather API** | Current weather in central Singapore, used in the suitability score (outdoor spots score lower when it rains) | While the app runs, from the browser; no key needed | [Docs](https://open-meteo.com/en/docs) |
| **Google Gemini API** | The AI study assistant (`gemini-3.5-flash-lite`) | While the app runs, from our server only | [Gemini API](https://ai.google.dev/gemini-api/docs) |

### API cost controls

The application must keep running until Week 18, so:

- **Google Places and NLB are only called by the import scripts**, never by the running app. Every page reads from our own database, so normal use costs nothing and can't run out of quota.
- **Google Places needs a billing account.** Restrict the key to "Places API (New)" and set a quota cap in Google Cloud Console. The import script prints its quota use per run (`pnpm import:google:preview` shows what it would do without calling the API).
- **Gemini:** set a daily request limit on the key. If the key fails, is rate-limited or runs out, the assistant shows a clear message and the rest of the app keeps working.
- **OneMap, NLB and Open-Meteo are free.** OneMap tokens expire every 3 days; the server renews them by itself from `ONEMAP_EMAIL` and `ONEMAP_PASSWORD`.
- **One nominated owner per account:**

| Service | Account owner | Spend cap |
|---|---|---|
| Google Cloud (Places API) | *TODO* | *TODO* |
| Google Gemini API | *TODO* | *TODO* |
| OneMap | *TODO* | Free |
| MongoDB Atlas | *TODO* | Free tier |
| Render (API hosting) | *TODO* | *TODO* |
| Vercel (website hosting) | *TODO* | Free tier |

---

## Data stores

MongoDB collections:

| Collection | Contents |
|---|---|
| `users` | Names, email addresses, hashed passwords, roles (user or admin) and study preferences |
| `places` | Study spaces: names, addresses, coordinates, opening hours, photos, facilities and descriptions |
| `reviews` | Ratings, review text, a crowd level, the place and the author |
| `checkins` | One-tap crowd reports: the place, the user, the crowd level and the time |
| `savedplaces` | Each user's saved places |
| `intents` | "Planning to go" entries for a place and time slot (deleted automatically once the slot is over) |
| `reports` | Incorrect-information reports from users, and whether an admin has resolved them |
| `chatMessages` | Global chat messages |

---

## Repository structure

> Mirrors the `client/` + `server/` layout used in the IS216 course scaffold (see the Week 5 and Lab Test projects).

```
WAD2_Project/
├─ client/                 # Vite + Vue 3 single-page application
│  ├─ e2e/                 # Playwright end-to-end specs
│  ├─ public/              # images, icons, app manifest
│  ├─ src/
│  ├─ playwright.config.js
│  ├─ vitest.config.js
│  ├─ vercel.json          # lets Vercel serve every page route
│  └─ package.json
├─ server/                 # Express + Mongoose REST API
│  ├─ controllers/
│  ├─ data/places.json     # sample places for mock mode and seeding
│  ├─ middleware/
│  ├─ models/
│  ├─ routes/
│  ├─ tests/               # server tests (node --test)
│  ├─ utils/
│  ├─ import-google-places.js
│  ├─ import-libraries.js
│  ├─ seed.js
│  ├─ server.js
│  └─ package.json
└─ README.md
```

---

## Prerequisites

- **Node.js** — `^22.18.0 || >=24.12.0` (as required by the course Vue scaffold)
- **pnpm** (or npm)
- A **MongoDB** connection string (local `mongod` or a hosted cluster), or none at all for mock mode
- API keys — see [Environment variables](#3-environment-variables)

---

## Setup

### 1. Clone

```bash
git clone https://github.com/KhinBhonePyae/WAD2_Project.git
cd WAD2_Project
```

### 2. Install dependencies

The team uses **pnpm** (install it once with `npm install -g pnpm`). npm still works too: swap `pnpm` for `npm` / `pnpm <script>` for `npm run <script>` in any command below.

```bash
cd client && pnpm install
cd ../server && pnpm install
```

### 3. Environment variables

Copy `server/config.env.example` to `server/config.env` and fill it in. The server only ever reads `server/config.env` (found relative to the server folder, so it works whichever folder you start it from); the `.example` file is just the template.

```
DB=<your MongoDB Atlas connection string>
JWT_SECRET=<any long random string>
GEMINI_API_KEY=<Google Gemini key>
ONEMAP_EMAIL=<OneMap login email>
ONEMAP_PASSWORD=<OneMap login password>
```

| Key | Required? | What happens without it |
|---|---|---|
| `JWT_SECRET` | **Yes** for a real deployment | Locally the server falls back to an insecure development secret and prints a warning. Always set your own when deploying. |
| `DB` | No | The server starts in **mock mode**: places load from `server/data/places.json`, and users, reviews and check-ins are kept in memory (lost on restart). `GET /health` reports which mode is active. |
| `GEMINI_API_KEY` | No | The AI study assistant replies that it isn't switched on. |
| `ONEMAP_EMAIL` and `ONEMAP_PASSWORD` | No | The map still shows each place, but directions are unavailable. The server uses these credentials to fetch and refresh OneMap tokens automatically. |
| `PORT` | No | The server listens on port 8000. Hosts such as Render set `PORT` themselves. |
| `GOOGLE_PLACES_API_KEY` | Only for `pnpm import:google` | Not needed to run the app. |
| `NLB_API_KEY` and `NLB_API_CODE` | Only for `import-libraries.js` | Not needed to run the app. |

`ONEMAP_TOKEN` may still be used as a legacy, manually managed fallback if login credentials are not configured.

If `DB` is set but `/health` still reports `mock`, check that your IP is on the Atlas **Network Access** allowlist.

**Client:** no env file is needed locally; the client talks to `http://localhost:8000`. For a deployed build, set `VITE_API_BASE_URL` to the API's address (see [Deployment](#deployment)).

> `config.env` is git-ignored and must **never** be committed. The repo is public at submission, so a committed password is a public password. Keys are shared within the team privately.

### 4. Data (optional)

The shared Atlas database already holds every study space. For a fresh database:

```bash
cd server
pnpm seed                    # copies data/places.json into MongoDB (safe to re-run)
pnpm import:google:preview   # shows what the Google Places import would add, without calling the API
pnpm import:google           # imports study spaces near SMU (needs GOOGLE_PLACES_API_KEY)
node import-libraries.js     # imports NLB library branches (needs NLB_API_KEY and NLB_API_CODE)
```

---

## Running the application

Two terminals:

```bash
# Terminal 1 — API
cd server
pnpm dev             # http://localhost:8000

# Terminal 2 — client
cd client
pnpm dev             # http://localhost:5173
```

Open **http://localhost:5173** — this is the first page of the application.

> `pnpm dev` restarts the API by itself whenever server files change, including after a `git pull`. If a pull changes a `package.json`, run `pnpm install` in that folder first. (Starting the API with `node server.js` instead does not restart; stop it with Ctrl+C and start it again.)

Production preview:

```bash
cd client
pnpm build
pnpm preview         # http://localhost:4173
```

---

## Testing

There are three layers of automated tests: client unit and component tests (Vitest), server tests (Node's built-in test runner) and end-to-end tests (Playwright).

| Layer | Where | Tests | Needs the API / database? |
|---|---|---|---|
| Unit tests | `client/src/utils/__tests__/` | 33 | No |
| Component tests | `client/src/components/__tests__/`, `client/src/views/__tests__/` | 27 | No (API calls are faked) |
| Server tests | `server/tests/` | 47 | No (runs in mock mode) |
| End-to-end tests | `client/e2e/` | 33 | Yes |

### Unit and component tests (Vitest)

```bash
cd client
pnpm test:unit            # watch mode; add -- --run to run once
```

Unit tests check pure functions with fixed inputs, e.g. the suitability score (`suitability.spec.js`), the Browse filters (`searchFilters.spec.js`) and the crowd "time ago" wording (`crowd.spec.js`).

Component tests mount a single Vue component in a simulated browser (jsdom) with `@vue/test-utils` and check what it renders and emits:

- `StarRating.spec.js`: stars, average and count, and "No reviews yet" for a place without reviews
- `CrowdHistogram.spec.js`: the "How busy is it?" card, including the empty state, "right now" vs "last reported", the report split, "planning to go" counts, hiding check-ins while closed, the login redirect, checking in, and undo
- `ReportIssue.spec.js`: the report form's login redirect, validation and success message
- `AdminReportsView.spec.js`: the admin reports page, including filters, resolving, deleting and the non-admin redirect

### Server tests (Node test runner)

```bash
cd server
pnpm test
```

These run in mock mode (`data/places.json`, in-memory data), so they never touch MongoDB Atlas and leave no data behind. No extra packages are needed.

- `crowd.test.js`: crowd reports and check-ins, including validation (bad level, unknown place, or a place that is closed right now), one report per user per hour, undo, and the place's crowd level following the newest report
- `place-summary.test.js`: "Open now" in Singapore time, including Google weekly hours, past-midnight hours, 24/7 places and admin closures
- `auth-middleware.test.js`: the login guard rejects missing, malformed and forged tokens and deleted accounts
- `report-model.test.js` and `reports-integration.test.js`: incorrect-information reports, from submitting to an admin resolving, reopening and deleting them, through the real routes

### End-to-end tests (Playwright)

```bash
cd client
pnpm exec playwright install chromium   # first run only, downloads the test browser
pnpm test:e2e
```

Playwright starts the client dev server by itself and runs the specs in `client/e2e/` against Chromium. Configuration is in `client/playwright.config.js`.

> **Before running:** start the API (`cd server && pnpm dev`) with a reachable database. Playwright starts the client but not the API.
>
> **The admin tests need an admin login.** Set it in the same terminal before `pnpm test:e2e`, using the demo admin from [Test accounts](#test-accounts):
>
> ```powershell
> # Windows PowerShell
> $env:E2E_ADMIN_EMAIL = "demo.admin@lockedin.test"; $env:E2E_ADMIN_PASSWORD = "LockedIN-Admin-2026"
> ```
>
> ```bash
> # macOS / Linux / Git Bash
> export E2E_ADMIN_EMAIL=demo.admin@lockedin.test E2E_ADMIN_PASSWORD=LockedIN-Admin-2026
> ```
>
> Tests that need a normal login create a throwaway account through the API and delete it afterwards, together with anything it posted.

Useful variations:

```bash
pnpm test:e2e --debug                         # step through interactively
pnpm exec playwright show-report              # open the HTML report
```

### Core user journeys covered

| # | Journey | End-to-end test |
|---|---|---|
| 1 | Register, log in, log out | `accounts.spec.js` (sign-up with preferences, password checks, duplicate email, wrong password, log in, log out) |
| 2 | Browse the study space catalogue and open a place detail page | `reviews.spec.js` (home page lists places), `search.spec.js` (opens a place from a result), `carousel.spec.js` (photo carousel) |
| 3 | Search and apply advanced filters | `search.spec.js`, `reviews.spec.js` (catalogue filters) |
| 4 | Submit, edit and delete a review | `reviews.spec.js` |
| 5 | Submit a crowd-level report | `crowd.spec.js` (check in, change it, undo it) |
| 6 | Save and remove a favourite place | `saved.spec.js` (log-in prompt for visitors, save from Browse, Saved page, remove) |
| 7 | Report incorrect information, and an admin handles it | `reports.spec.js` |
| 8 | Admin creates, edits and deletes a study space | `admin-places.spec.js` |
| 9 | Admin promotes, demotes and deletes users | `admin-users.spec.js` |

> Each team member writes the tests for the feature they own. Use stable `data-test` attributes as selectors rather than CSS classes, so tests do not break when styling changes.

---

## Test accounts

These work on both the deployed site and a local copy connected to the shared database. Please don't change their passwords or delete them.

| Role | Email | Password | Can do |
|---|---|---|---|
| Standard user | `Rubert@gmail.com` | `12345678` | Everything a student can: reviews, crowd check-ins, saved places, "Planning to go", chat |
| Admin | `demo.admin@lockedin.test` | `LockedIN-Admin-2026` | Everything above, plus the Manage, Users and Reports pages |

Or sign up for a new account on the Sign up page.

---

## Deployment

The website (`client/`) is deployed on **Vercel** and the API (`server/`) on **Render**. Data lives in **MongoDB Atlas**. Both hosts install dependencies with **pnpm** from the committed `pnpm-lock.yaml` files, so the live site runs exactly the library versions that were tested.

### 1. MongoDB Atlas

1. In **Network Access**, add `0.0.0.0/0` (allow access from anywhere). Render's free plan has no fixed IP address, so the API can't be allowlisted by IP.
2. Use a database user with a strong password. Its connection string becomes `DB` below.

### 2. API on Render

1. **New → Web Service → Public Git Repository**, and paste `https://github.com/KhinBhonePyae/WAD2_Project`.
2. Settings:

| Setting | Value |
|---|---|
| Language | Node |
| Branch | `main` |
| Region | Singapore |
| Root Directory | `server` |
| Build Command | `npx --yes pnpm@12.3.4 install --frozen-lockfile` |
| Start Command | `npm start` |
| Health Check Path | `/health` (under Advanced) |

3. Under **Environment Variables**, add `DB`, `JWT_SECRET` (a new long random string, not the one used locally), `GEMINI_API_KEY`, `ONEMAP_EMAIL` and `ONEMAP_PASSWORD`. Do not set `PORT`; Render sets it. The Google Places and NLB keys are not needed.
4. Deploy, then open `https://<your-service>.onrender.com/health`. It should show `{"ok":true,"mode":"mongodb"}`. If it says `mock`, Atlas is refusing the connection (step 1).

> Render's free plan sleeps after 15 minutes without visits, and the first request after that takes 30–60 seconds. The paid Starter plan stays awake.

### 3. Website on Vercel

Deployed from a laptop with the Vercel CLI (no GitHub connection needed):

```bash
cd client
npx vercel login                 # first time only
npx vercel link                  # first time only: create the project, keep the detected Vite settings
npx vercel env add VITE_API_BASE_URL production    # paste https://<your-service>.onrender.com (no trailing slash)
npx vercel --prod                # build and publish
```

`client/vercel.json` sets the install command (pnpm, as on Render) and makes refreshing any page (e.g. `/places/12`) work instead of showing a 404. `VITE_API_BASE_URL` is built into the site, so run `npx vercel --prod` again after changing it.

> You can also import the repository in the Vercel dashboard (**Add New → Project**, Root Directory `client`) and add `VITE_API_BASE_URL` there; `vercel.json` applies either way.

### 4. Check the live site

On a phone and a laptop:

1. Open the Vercel URL, then refresh a place page.
2. Log in with the test accounts above.
3. Post and delete a review, report the crowd, and save a place.
4. Get directions and ask the assistant a question.
5. Log in as the admin and open Manage, Users and Reports.

Then put both URLs in [Links](#links).

### Updating the live site

Deploys are manual, so nothing reaches the live site by accident:

- **API:** in Render, **Manual Deploy → Deploy latest commit** (after pushing to `main`).
- **Website:** `cd client` then `npx vercel --prod`.

Do not redeploy after submission.

### Before submission

- [ ] Both live URLs in [Links](#links), checked by someone who didn't deploy
- [ ] Atlas password and `JWT_SECRET` changed from the ones shared during development
- [ ] Account owners and spend caps filled in under [API cost controls](#api-cost-controls)
- [ ] AI declaration below completed by every member
- [ ] Copy this file to `README.txt` (the course asks for that file name)

---

## Use of AI / LLM tools

Per IS216 Group Project FAQ 20.

**Permitted** — information search; generating website concepts, layouts or themes; exploring UI/UX design inspiration; explaining coding errors and debugging hints; boilerplate and starter code snippets; generating unit test cases, sample inputs or mock data.

**Not permitted** — core implementation (main business logic, backend endpoints, critical frontend interactivity); problem solving, where students must demonstrate their own capability rather than relying on automated AI fixes.

### Declaration

Parts of this project written with AI/LLM tools. Each member completes their own rows.

| Member | Area / files | How AI was used |
|---|---|---|
| Jeffrey Wen | Crowd check-ins and the "How busy is it?" card (`server/models/checkin-model.js`, `server/controllers/crowd-controller.js`, `server/routes/crowd-routes.js`, `client/src/components/CrowdHistogram.vue`, `client/src/utils/crowd.js`) | Written with an AI coding assistant (Claude Code), then reviewed and tested by Jeffrey |
| Jeffrey Wen | Bug fixes from the team's code audit (20–21 Oct and 25 Oct), including `server/utils/place-summary.js` and fixes in reviews, places, users, search and the place page | Bugs found and fixed with Claude Code, then reviewed and tested by Jeffrey |
| Jeffrey Wen | Tests: client unit and component tests (`suitability`, `searchFilters`, `crowd`, `StarRating`, `CrowdHistogram`), server tests (`crowd`, `place-summary`, `auth-middleware`) and end-to-end tests (`accounts`, `reviews`, `search`, `crowd`, `saved`) | Generated with Claude Code, then run and checked by Jeffrey |
| Jeffrey Wen | Chat buttons closing on an outside click, their icons and labels (`AssistantBubble.vue`, `WorldChat.vue`); pnpm setup; README | Written with Claude Code |
| Leong Yew Kai | *TODO* | *TODO* |
| Khin Bhone Pyae | *TODO* | *TODO* |
| Thet Htar Nwe | *TODO* | *TODO* |
| Gerena Lim | *TODO* | *TODO* |
| Liew Tian Le | *TODO* | *TODO* |

> Note: the AI study assistant is a **product feature** that calls an LLM API at runtime. That is separate from this declaration, which covers AI assistance used while writing the code.

---

## Team and task allocation

| # | Name | Email ID | Responsibility |
|---|---|---|---|
| 1 | Leong Yew Kai | yk.leong.2025 | Location search, including location-based search, and Google Places API integration |
| 2 | Khin Bhone Pyae | bp.khin.2025 | User authentication and profile management, including registration, login, logout, profile updates, account deletion and saved user preferences |
| 3 | Thet Htar Nwe | hn.thet.2025 | Study space catalogue and management, including listing and detail pages, facility information, image carousel, CRUD functionality and responsive Vue/CSS interface development |
| 4 | Gerena Lim | gerena.lim.2025 | Search, advanced filters and suitability scoring, including filtering study spaces by facilities, study preferences and location suitability |
| 5 | Jeffrey Wen | jeffrey.wen.2025 | Ratings, reviews and crowd reporting, including creating, viewing, editing and deleting reviews, calculating ratings and displaying user-reported crowd levels |
| 6 | Liew Tian Le | tianle.liew.2024 | AI-powered study assistant development; OneMap API integration for real-time weather suitability data and interactive route mapping with estimated travel times |

---

## Design

Colour palette: Sky Blue, Light Green, Light Purple, Light Pink.

Responsive design is tested with Chrome DevTools from iPhone-class widths up to the Bootstrap XL breakpoint.
