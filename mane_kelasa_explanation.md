# 📖 Mane-Kelasa — Complete Project Explanation

## 🏠 What Does This App Do?

**Mane-Kelasa (ಮನೆ-ಕೆಲ್ಸ)** means **"House Work"** in Kannada.

It is a **digital notice board** for small towns where:
- 🧹 **Workers** (maids, cooks, gardeners, plumbers) register themselves
- Every morning, workers tap **"I am available today"**
- 🏡 **Residents** open the app and see who is available **right now** near them
- Residents can **call the worker directly** with one tap

> **Real-world problem it solves:** In small towns, workers lose income when their employer is away. Residents can't find workers quickly. There is no local job board.

---

## 🗂️ Project Structure

```
mane_kelasa/
│
├── backend/          ← The BRAIN (Python)
│   ├── main.py       ← Starts the server
│   ├── database.py   ← Creates & manages the database
│   ├── models.py     ← Defines data shapes
│   └── routes/
│       ├── workers.py   ← Register / Login / Toggle availability
│       └── ratings.py   ← Thumbs up rating
│
└── frontend/         ← The FACE (what users see)
    ├── index.html           ← Home page — shows worker list
    ├── worker-register.html ← Registration page
    ├── worker-dashboard.html← Worker's dashboard (toggle ON/OFF)
    ├── style.css            ← All colors and design
    ├── app.js               ← Language switching + shared logic
    └── manifest.json        ← Makes it installable on phone (PWA)
```

---

## 🖥️ THE FRONTEND (What Users See)

**Technology: HTML + CSS + JavaScript**

### 3 Pages:

#### 1. `index.html` — Home Page (Residents use this)
- Shows a **list of all workers** in cards
- Each card shows: Name, Skill, Area, Daily Rate, Availability status
- Filter buttons: 🧹 Cleaning | 🌱 Gardening | 🍳 Cooking | 🛡️ Security | 🔧 Plumbing
- **📞 Call Now** button — directly calls the worker's phone
- **👍 Thumbs Up** button — gives the worker a rating

#### 2. `worker-register.html` — Registration Page (New workers)
Worker fills in:
- Full Name
- Mobile Number (10 digits)
- Skill type
- Area / Street name
- Daily Rate in ₹
- A 4-digit PIN (their password)

#### 3. `worker-dashboard.html` — Worker Dashboard
- Worker logs in with mobile number + PIN
- Sees a **big ON/OFF toggle switch**
- Toggle ON = "I am available today" ✅ → everyone sees this instantly
- Toggle OFF = "Not available today" ❌

### `app.js` — Shared Logic
- Handles **language switching** between English and ಕನ್ನಡ
- Saves language preference in browser memory
- All text labels are stored in two versions (English + Kannada)

### `style.css` — Design
- All colors, fonts, animations, card layouts
- Mobile-first responsive design

### `manifest.json` — PWA (Progressive Web App)
- Tells the phone browser: **"This website can be installed as an app"**
- Contains: App name, icons, theme color, start URL
- When installed, it opens fullscreen like a real native app

---

## ⚙️ THE BACKEND (The Brain)

**Technology: Python + FastAPI + SQLite**

### `database.py` — The Database
Creates a file called `mane_kelsa.db` (SQLite database).

It has **one table** called `workers` with these columns:

| Column | What it stores |
|---|---|
| `id` | Auto number (1, 2, 3...) |
| `name` | Worker's full name |
| `phone` | Mobile number (must be unique) |
| `skill` | cleaning / cooking / gardening etc. |
| `area` | Street / area name |
| `daily_rate` | Rate in ₹ per day |
| `pin` | 4-digit login password |
| `is_available` | 0 = No, 1 = Yes (available today) |
| `thumbs_up` | Count of 👍 ratings received |
| `created_at` | When they registered |

### `models.py` — Data Shapes
Defines what data is expected for each action:
- **WorkerRegister** — name, phone, skill, area, daily_rate, pin
- **AvailabilityToggle** — pin + true/false

### `routes/workers.py` — Worker APIs
Handles all worker actions:

| API | What it does |
|---|---|
| `GET /api/workers` | Get all workers (for the home page list) |
| `POST /api/workers/register` | Register a new worker |
| `POST /api/workers/{phone}/login` | Login with phone + PIN |
| `POST /api/workers/{phone}/toggle` | Toggle availability ON/OFF |

### `routes/ratings.py` — Rating API
| API | What it does |
|---|---|
| `POST /api/workers/{phone}/rate` | Add a thumbs up 👍 |

### `main.py` — The Server
- **Starts the FastAPI server** on port 8000
- Sets up **CORS** (allows the frontend to talk to the backend)
- Sets up **WebSocket** for real-time updates
- **Also serves the frontend files** — so when you open `http://localhost:8000`, it shows `index.html`

---

## 🔄 HOW FRONTEND AND BACKEND TALK TO EACH OTHER

```
┌─────────────────────────────────────────────────────┐
│                    USER'S PHONE/BROWSER              │
│                                                      │
│   Frontend (HTML/CSS/JS)                             │
│   ┌──────────────────────────────────────────────┐  │
│   │  index.html shows worker cards               │  │
│   │  JavaScript fetches data from backend via    │  │
│   │  HTTP API calls (fetch/axios)                │  │
│   └──────────────────────────────────────────────┘  │
│                        │  ↑                          │
│              API Request│  │API Response             │
│                        ↓  │                          │
└────────────────────────────────────────────────────-─┘
                         │  ↑
               http://localhost:8000/api/...
                         │  ↑
┌────────────────────────────────────────────────────-─┐
│                    COMPUTER (Server)                  │
│                                                      │
│   Backend (Python FastAPI)                           │
│   ┌──────────────────────────────────────────────┐  │
│   │  main.py receives request                    │  │
│   │  routes/workers.py handles it                │  │
│   │  database.py reads/writes mane_kelsa.db      │  │
│   │  Sends back JSON data                        │  │
│   └──────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## ⚡ REAL-TIME UPDATES — WebSocket

This is the **special feature** of the app.

**Normal websites:** You ask → server replies → done. You have to refresh to see new data.

**This app uses WebSocket:** The server can **push updates automatically** to ALL open browsers.

**How it works:**
1. Worker A opens their dashboard and toggles to "Available" ✅
2. Backend receives the toggle → updates database → **broadcasts to all connected browsers**
3. Resident B's home page **automatically refreshes** the worker list — without pressing F5!

```
Worker toggles ON
      ↓
Backend updates database
      ↓
WebSocket broadcasts to everyone
      ↓
All open home pages update instantly ⚡
```

---

## 📱 COMPLETE USER JOURNEY

### Journey 1 — Worker registering for the first time:
```
1. Worker opens app on phone
2. Taps "Register" (+ನೋಂದಣಿ)
3. Fills form → submits
4. Frontend sends POST /api/workers/register to backend
5. Backend saves to database
6. Success message shown ✅
```

### Journey 2 — Worker marking availability every morning:
```
1. Worker opens "My Account" (ನನ್ನ ಖಾತೆ)
2. Enters phone + PIN → Login
3. Frontend sends POST /api/workers/{phone}/login
4. Backend checks PIN in database → returns worker data
5. Worker sees their dashboard with the toggle
6. Worker turns toggle ON
7. Frontend sends POST /api/workers/{phone}/toggle
8. Backend updates is_available = 1 in database
9. WebSocket broadcasts to all browsers → everyone's home page updates
```

### Journey 3 — Resident finding a worker:
```
1. Resident opens home page
2. Frontend sends GET /api/workers
3. Backend fetches all workers from database → returns JSON
4. JavaScript creates a card for each worker on the page
5. Resident filters by skill (cleaning, cooking, etc.)
6. Resident taps "📞 Call Now" → phone app opens and dials the worker
```

---

## 🛠️ TECHNOLOGIES USED

| Technology | Category | What it does in this project |
|---|---|---|
| **HTML** | Frontend | Structure of all 3 pages |
| **CSS** | Frontend | Colors, fonts, card design, animations |
| **JavaScript** | Frontend | API calls, language toggle, real-time updates |
| **Python** | Backend | Programming language for the server |
| **FastAPI** | Backend | Web framework — handles all API requests |
| **SQLite** | Database | Stores all worker data in a single .db file |
| **WebSocket** | Backend | Pushes real-time updates to all browsers |
| **Uvicorn** | Server | Runs the FastAPI application |
| **Pydantic** | Backend | Validates incoming data shapes |
| **PWA** | Frontend | Makes the website installable as a phone app |

---

## 💡 Key Points to Remember

1. **Backend and frontend are separate** — backend handles data, frontend handles display
2. **SQLite database** is a single file (`mane_kelsa.db`) — easy to backup, no extra software needed
3. **WebSocket** makes availability updates instant for all users
4. **PIN system** — simple 4-digit password for workers (no complex login/signup)
5. **Bilingual** — full support for English and ಕನ್ನಡ, switchable with one button
6. **PWA** — users can install this on their Android phone from Chrome (Add to Home Screen)
7. **One-tap calling** — the app uses `tel:` links so calling is instant
