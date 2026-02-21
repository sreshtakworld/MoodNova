# 🌟 MoodNova – Backend Setup Guide

A full **Node.js + Express + SQLite** backend for the MoodNova mental health tracker.

---

## 📁 Project Structure

```
moodnova-backend/
├── server.js               ← Entry point – start here
├── .env                    ← Environment variables
├── package.json
├── db/
│   └── database.js         ← SQLite setup, schema, seed
├── middleware/
│   └── auth.js             ← JWT verify + admin guard
├── routes/
│   ├── auth.js             ← Signup, Login, Admin login, /me
│   ├── assessments.js      ← Submit & fetch assessments
│   ├── moodLogs.js         ← Daily mood quick-logs
│   └── admin.js            ← Admin-only dashboard routes
└── public/
    ├── moodnova.html        ← ← ← PUT YOUR FRONTEND FILE HERE
    └── api-connector.js    ← Frontend ↔ Backend wiring script
```

---

## ⚙️ Prerequisites

- **Node.js** v18 or higher → https://nodejs.org
- **npm** (comes with Node)

Check your versions:
```bash
node -v   # should print v18.x.x or higher
npm -v
```

---

## 🚀 Quick Start

### 1. Install dependencies
```bash
cd moodnova-backend
npm install
```

### 2. Configure environment
The `.env` file is already set up with defaults. You can edit it:
```env
PORT=5000
JWT_SECRET=moodnova_super_secret_key_change_me_in_production
JWT_EXPIRES_IN=7d
ADMIN_EMAIL=admin@moodnova.com
ADMIN_PASSWORD=admin123
DB_PATH=./db/moodnova.db
```

### 3. Add your frontend
Copy `moodnova.html` into the `public/` folder:
```
moodnova-backend/public/moodnova.html
```

Then open `moodnova.html` in a text editor and add this line just before `</body>`:
```html
<script src="/api-connector.js"></script>
```
> This wires all buttons (signup, login, submit assessment, etc.) to the real API.

### 4. Start the server
```bash
npm start
```
or for auto-reload during development:
```bash
npm run dev
```

### 5. Open in browser
```
http://localhost:5000
```
That's it! The frontend + backend are now fully connected. 🎉

---

## 🔑 Admin Login

| Field    | Value                   |
|----------|-------------------------|
| Email    | `admin@moodnova.com`    |
| Password | `admin123`              |

The admin account is auto-created on first launch.

---

## 📡 API Reference

All endpoints are prefixed with `/api`.

### Auth

| Method | Endpoint              | Body / Notes                          | Auth? |
|--------|-----------------------|---------------------------------------|-------|
| POST   | `/auth/signup`        | `{ first_name, last_name, email, password, age, gender, location, marital_status, is_working, ... }` | ❌ |
| POST   | `/auth/login`         | `{ email, password }`                 | ❌    |
| POST   | `/auth/admin-login`   | `{ email, password }`                 | ❌    |
| GET    | `/auth/me`            | Returns current user profile          | ✅    |

### Assessments

| Method | Endpoint                    | Description                        | Auth? |
|--------|-----------------------------|------------------------------------|-------|
| POST   | `/assessments`              | Submit 10 answers, get scored back | ✅    |
| GET    | `/assessments`              | Full assessment history            | ✅    |
| GET    | `/assessments/weekly`       | Last 7 entries for weekly chart    | ✅    |
| GET    | `/assessments/:id`          | Single assessment detail           | ✅    |

**Submit body:**
```json
{
  "answers": [0, 1, 2, 3, 4, 2, 1, 0, 3, 2]
}
```
Each value is `0` (option A) through `4` (option E).

**Response:**
```json
{
  "id": 1,
  "user_id": 3,
  "score": 27,
  "mood_label": "Mild Stress",
  "mood_emoji": "😐",
  "taken_at": "2025-02-21T10:30:00.000Z",
  "answers": [0, 1, 2, 3, 4, 2, 1, 0, 3, 2]
}
```

### Mood Logs

| Method | Endpoint              | Description                         | Auth? |
|--------|-----------------------|-------------------------------------|-------|
| POST   | `/mood-logs`          | Quick daily mood `{ mood, note }`   | ✅    |
| GET    | `/mood-logs`          | All logs for current user           | ✅    |
| GET    | `/mood-logs/heatmap`  | `{ "2025-01-15": "good", ... }`     | ✅    |

Valid mood values: `great`, `good`, `neutral`, `low`, `bad`

### Admin (requires admin JWT)

| Method | Endpoint                  | Description                      |
|--------|---------------------------|----------------------------------|
| GET    | `/admin/stats`            | KPIs: users, assessments, avg    |
| GET    | `/admin/users`            | Paginated user list              |
| GET    | `/admin/users/:id`        | User + their assessments         |
| DELETE | `/admin/users/:id`        | Delete a user                    |
| GET    | `/admin/assessments`      | All assessments (paginated)      |

---

## 🧪 Testing with curl

```bash
# Signup
curl -X POST http://localhost:5000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"first_name":"Jane","last_name":"Doe","email":"jane@test.com","password":"test1234","age":25,"gender":"Female","location":"Urban","marital_status":"Unmarried","is_working":true,"job_type":"Employee"}'

# Login
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"jane@test.com","password":"test1234"}'

# Submit assessment (use the token from login)
curl -X POST http://localhost:5000/api/assessments \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{"answers":[0,1,2,1,0,1,2,0,1,2]}'

# Admin stats
curl http://localhost:5000/api/admin/stats \
  -H "Authorization: Bearer YOUR_ADMIN_TOKEN"
```

---

## 🗄️ Database

SQLite file is saved at `./db/moodnova.db`. You can inspect it with:
- **DB Browser for SQLite** (free GUI): https://sqlitebrowser.org
- Or the `sqlite3` CLI: `sqlite3 db/moodnova.db ".tables"`

**Tables:**
- `users` – all registered accounts
- `assessments` – one row per test submission
- `mood_logs` – quick daily check-ins

---

## 🛡️ Security Notes (for production)

1. Change `JWT_SECRET` to a long random string
2. Restrict CORS `origin` to your frontend domain
3. Use HTTPS (add a reverse proxy like nginx)
4. Change the default admin password
5. Consider rate-limiting the auth routes (use `express-rate-limit`)

---

## 🔧 Troubleshooting

| Problem | Fix |
|---------|-----|
| `Cannot find module 'better-sqlite3'` | Run `npm install` again |
| Port 5000 in use | Change `PORT` in `.env` |
| CORS errors in browser | Ensure `API_BASE` in `api-connector.js` matches your server URL |
| Token expired | Log out and log back in |
