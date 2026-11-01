# Geo Attend

> A geofenced employee attendance system: employees check in and out from the browser using their location, backed by an Express + MongoDB API, with an experimental face-recognition attendance service.

![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![MUI](https://img.shields.io/badge/MUI-007FFF?style=for-the-badge&logo=mui&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-000000?style=for-the-badge&logo=flask&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white)

## About

Geo Attend marks attendance only when an employee is physically inside a configured area. The React frontend reads the browser's geolocation and sends it to the Node.js API, which uses `geolib` to check the point against a geofence before recording a check-in. Despite the repository name, it contains all three parts of the project: the React frontend, the Express server and a Python face-recognition prototype.

## Features

- **Geofenced check-in**: `/mark-attendance` accepts a check-in only if the coordinates fall within the geofence radius around a fixed centre point (configured in `server/routes/checkRoute.js`).
- **Check-out**: records the check-out time against the day's entry; duplicate check-ins and check-outs are detected.
- **Daily attendance records**: one document per day with each user's status, check-in and check-out; fetch a day's records with `/attendance/:date`.
- **Registration with profile photo**: multer upload (JPEG/PNG/GIF, up to 5 MB) stored in `server/uploads/` and served statically.
- **Auth**: bcrypt-hashed passwords, JWT in an HTTP-only cookie, and `Admin` / `Employee` roles with a role-protected admin route on the frontend.
- **Face-recognition prototype** (`faceApi/`): a Flask app that encodes faces from a folder of training images, matches faces in a webcam stream with `face_recognition` and OpenCV, and appends matches to a daily CSV.

### Current state

The employee dashboard (geolocation check-in) and the sidebars are implemented, but the employee route is not wired into `App.jsx` yet and the admin dashboard is an empty placeholder. The face-recognition service is a standalone prototype and is not called by the server or the frontend.

## Tech Stack

| Part | Tools |
|---|---|
| Frontend | React 18 (Create React App), React Router 6, Axios, MUI 6, styled-components, Framer Motion, react-hot-toast |
| Server | Node.js, Express 4, Mongoose 8, geolib, multer, bcrypt, jsonwebtoken, cookie-parser |
| Face API | Python 3.9, Flask, face_recognition, OpenCV, NumPy |
| Database | MongoDB |
| Deployment config | `vercel.json` for the server (`@vercel/node`) and the face API (`@vercel/python`) |

## Project Structure

```
Geo-Attend-Frontend/
├── frontend/               # React app
│   └── src/
│       ├── App.jsx         # routes
│       ├── AuthContext.jsx # login / logout / session check
│       ├── auth/           # ProtectedRoute
│       ├── components/     # Login
│       └── pages/          # Home, admin/, employee/
├── server/                 # Express API
│   ├── index.js
│   ├── routes/             # auth, userRoutes, checkRoute, getAttendance
│   ├── controllers/        # login, logout
│   ├── models/             # User, Attendance
│   ├── middlewares/        # JWT auth check
│   └── vercel.json
└── faceApi/                # Flask face-recognition prototype
    ├── api/app.py
    ├── requirements.txt
    └── vercel.json
```

## Getting Started

### Prerequisites

- Node.js 18+
- MongoDB
- Python 3.9 (only for the face API)

### Server

```bash
cd server
npm install
npx nodemon index.js      # http://localhost:3000
```

The MongoDB connection string and the JWT secret are currently set in `server/index.js`, `server/controllers/auth.js` and `server/middlewares/authCheck.js`. Replace them with your own values, ideally loaded from environment variables such as `MONGO_URI` and `JWT_SECRET`. `NODE_ENV=production` makes the auth cookie `secure`.

### Frontend

```bash
cd frontend
npm install
PORT=3001 npm start       # the server's CORS config allows http://localhost:3001
```

The frontend calls the API at `http://localhost:3000`. Other scripts: `npm run build`, `npm test`.

### Face API (optional)

```bash
cd faceApi
pip install -r requirements.txt
```

`api/app.py` expects a `Training images/` folder with one photo per person (the file name is used as the person's name) and a `templates/index.html` page, and it reads from the local webcam.

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/register` | Register a user (multipart: `username`, `email`, `role`, `password`, `image`) |
| `POST` | `/login` | Log in; sets the JWT cookie |
| `POST` | `/logout` | Clear the session cookie |
| `GET` | `/` | Current user from the JWT (auth required) |
| `POST` | `/mark-attendance` | Check in. Body: `userId`, `latitude`, `longitude`, `checkinTime`. Rejected with 403 outside the geofence |
| `POST` | `/mark-checkout` | Check out. Body: `userId`, `checkoutTime` |
| `GET` | `/attendance/:date` | Attendance record for a given date |
| `GET` | `/uploads/:imageName` | Serve an uploaded profile image |

Face API (Flask): `GET /` (page) and `GET /video_feed` (MJPEG stream with recognised faces).

## Author

**Aryan Bhandari**: [GitHub](https://github.com/aryanbha123)
