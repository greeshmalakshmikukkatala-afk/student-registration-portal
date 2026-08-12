# student-registration-portal

A full-stack MERN (MongoDB, Express, React, Node.js) application for managing
student registrations — built to the "Full Stack Development" minor project
brief. Staff/admin users log in with JWT-based authentication and can
register, view, search, update, and delete student records.

---

## 1. Functional Requirements

- **Authentication**: staff/admin registration and login, JWT-protected API.
- **Registration**: create new student records with validated fields.
- **Profile management**: view, edit, and delete existing student records.
- **Dashboard**: searchable, paginated list of all registered students.

## 2. System Architecture

```
┌─────────────────┐        HTTPS/JSON        ┌──────────────────┐        ┌───────────┐
│  React Frontend  │  ───────────────────▶   │  Express REST API │ ─────▶ │  MongoDB  │
│  (port 3000)     │  ◀───────────────────   │  (port 5000)       │        │           │
└─────────────────┘        JWT in header      └──────────────────┘        └───────────┘
```

- **Frontend** (`/frontend`): React + React Router. `AuthContext` holds the
  logged-in user and JWT (persisted in `localStorage`). Axios instance in
  `src/api/api.js` attaches the token to every request automatically.
- **Backend** (`/backend`): Express REST API. Routes are split into
  `routes/auth.js` (register/login/me) and `routes/students.js` (CRUD).
  `middleware/auth.js` verifies the JWT on protected routes;
  `middleware/validate.js` centralizes `express-validator` error handling.
- **Database**: MongoDB via Mongoose. Two collections: `users` (staff/admin
  accounts, hashed passwords) and `students` (registration records).

### Database schema

**User**
| Field | Type | Notes |
|---|---|---|
| name | String | required |
| email | String | required, unique |
| password | String | required, bcrypt-hashed |
| role | String | `admin` \| `staff` |

**Student**
| Field | Type | Notes |
|---|---|---|
| firstName, lastName | String | required |
| email | String | required, unique |
| phone | String | optional, validated format |
| dateOfBirth | Date | required |
| course | String | required |
| address | String | optional |
| enrollmentDate | Date | defaults to creation date |
| status | String | `active` \| `inactive` \| `graduated` |
| createdBy | ObjectId → User | audit trail |

## 3. Tools & Platforms Used

- **Frontend**: HTML, CSS, JavaScript, React (React Router, Axios)
- **Backend**: Node.js, Express
- **Database**: MongoDB (Mongoose ODM) — MySQL can be substituted; see note below
- **Authentication**: JSON Web Tokens (JWT) + bcrypt password hashing
- **Testing**: Jest, Supertest, mongodb-memory-server

> **Note on MongoDB vs MySQL**: the brief lists both as options. This build
> uses MongoDB/Mongoose because the data (flat student/user records) maps
> naturally onto documents and needs no relational joins. To use MySQL
> instead, swap the Mongoose models for Sequelize models with an equivalent
> schema — the route/controller logic stays the same.

## 4. Project Structure

```
student-registration-portal/
├── backend/
│   ├── config/db.js          # MongoDB connection
│   ├── models/                # User.js, Student.js (Mongoose schemas)
│   ├── middleware/             # auth.js (JWT), validate.js
│   ├── routes/                 # auth.js, students.js
│   ├── tests/students.test.js  # Jest/Supertest integration tests
│   ├── seed.js                  # Seeds sample/synthetic data
│   ├── server.js                 # App entry point
│   └── .env.example
└── frontend/
    ├── src/
    │   ├── api/api.js            # Axios client + JWT interceptor
    │   ├── context/AuthContext.js
    │   ├── components/            # StudentForm, StudentList, ProtectedRoute
    │   ├── pages/                  # Login, Register, Dashboard
    │   ├── App.js, index.js, styles.css
    └── .env.example
```

## 5. Setup & Running Locally

### Prerequisites
- Node.js 18+ and npm
- A MongoDB instance — either local (`mongodb://localhost:27017`) or a free
  [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) cluster

### Backend

```bash
cd backend
npm install
cp .env.example .env      # then edit .env with your MONGO_URI and JWT_SECRET
npm run seed               # optional: creates a demo admin + sample students
npm run dev                 # starts on http://localhost:5000
```

Demo login after seeding: **admin@example.com / Admin@123**

### Frontend

```bash
cd frontend
npm install
cp .env.example .env       # points to the backend API URL
npm start                    # starts on http://localhost:3000
```

### Running tests

```bash
cd backend
npm install
npm test
```

## 6. API Reference

Base URL: `http://localhost:5000/api`

### Auth

| Method | Endpoint | Access | Body | Description |
|---|---|---|---|---|
| POST | `/auth/register` | Public | `{ name, email, password }` | Create a staff account, returns JWT |
| POST | `/auth/login` | Public | `{ email, password }` | Authenticate, returns JWT |
| GET | `/auth/me` | Private | — | Get the logged-in user's profile |

### Students

All student routes require header: `Authorization: Bearer <token>`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/students?search=&page=&limit=` | List/search students (paginated) |
| GET | `/students/:id` | Get one student |
| POST | `/students` | Create (register) a student |
| PUT | `/students/:id` | Update a student |
| DELETE | `/students/:id` | Delete a student |

**Example — register a student**
```bash
curl -X POST http://localhost:5000/api/students \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "Aarav",
    "lastName": "Sharma",
    "email": "aarav@example.com",
    "dateOfBirth": "2002-05-14",
    "course": "Computer Science"
  }'
```

All validation errors return `400` with a body like:
```json
{ "message": "Validation failed", "errors": [{ "field": "email", "message": "A valid email is required" }] }
```

## 7. Dataset

A synthetic sample dataset (3 student records + 1 admin user) is provided in
`backend/seed.js` and can be loaded with `npm run seed`. It mirrors the shape
of the brief's referenced `json-server`-style sample data
(https://github.com/typicode/json-server) but is stored in MongoDB instead of
a static JSON file, since the app needs real create/update/delete support.

## 8. Deployment

**Backend** (e.g. [Render](https://render.com) or [Railway](https://railway.app)):
1. Push this repo to GitHub.
2. Create a new Web Service pointing at `/backend`, build command `npm install`, start command `npm start`.
3. Add environment variables from `.env.example` (use your MongoDB Atlas URI and a strong `JWT_SECRET`).

**Frontend** (e.g. [Vercel](https://vercel.com) or [Netlify](https://netlify.com)):
1. Point the project at `/frontend`, build command `npm run build`, publish directory `build`.
2. Set `REACT_APP_API_URL` to your deployed backend's URL (e.g. `https://your-api.onrender.com/api`).
3. Update the backend's `CLIENT_ORIGIN` env var to your deployed frontend URL so CORS allows it.

**Database**: use [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) free tier for a cloud-hosted database reachable from your deployed backend.

## 9. Reference Projects

The brief's reference links (Brad Traversy's MERN auth project, Academind's
MERN course, and the Express.js repo) informed the auth flow (JWT +
bcrypt) and REST conventions used here, though all code in this repo was
written fresh for this project's schema and requirements.
