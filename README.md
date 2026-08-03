<p align="center">
  <img src="frontend/public/assets/Logo.png" alt="NeuroSense AI Logo" width="120"/>
</p>

<h1 align="center">NeuroSense AI — Parkinson's Disease Screening & Care Platform</h1>

<p align="center">
  A full-stack (MERN) platform that screens for Parkinson's Disease from handwriting patterns and images,
  and gives patients a complete companion app for tracking symptoms, medication, exercise, and progress.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Stack-MERN-4EA94B?style=for-the-badge&logo=mongodb&logoColor=white" />
  <img src="https://img.shields.io/badge/Frontend-React%2019-61DAFB?style=for-the-badge&logo=react&logoColor=black" />
  <img src="https://img.shields.io/badge/Backend-Express%205-000000?style=for-the-badge&logo=express" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" />
</p>

<p align="center">
  <a href="https://neurosense-ai-pd.onrender.com"><img src="https://img.shields.io/badge/🚀%20Backend%20API-Live-brightgreen?style=for-the-badge" /></a>
  <a href="https://github.com/SuhasGowda24/ML-CNN-PD-Prediction"><img src="https://img.shields.io/badge/🧠%20CNN%20Model-Repo-orange?style=for-the-badge" /></a>
  <a href="https://github.com/SuhasGowda24/ML-Classification-PD-Prediction"><img src="https://img.shields.io/badge/📈%20Classification%20Model-Repo-orange?style=for-the-badge" /></a>
</p>

---

## Overview

NeuroSense AI is the patient- and admin-facing application built on top of two independently
trained and deployed machine learning models. It handles authentication, patient records, and
the day-to-day patient experience, and calls out to the models below whenever a screening is run.

| Model | What it does | Repo |
|---|---|---|
| **CNN (Image-based)** | Classifies uploaded/captured handwriting images (e.g. spirals) as Healthy or PD using EfficientNetB0, with Grad-CAM heatmap explainability | [ML-CNN-PD-Prediction](https://github.com/SuhasGowda24/ML-CNN-PD-Prediction) |
| **Classification (Signal-based)** | Classifies live canvas drawing signals (x, y, time, pressure, pen state) into 12 kinematic features, then predicts PD status | [ML-Classification-PD-Prediction](https://github.com/SuhasGowda24/ML-Classification-PD-Prediction) |

Both models are deployed as Hugging Face Spaces and are called directly from this app's backend/frontend over REST.

---

## How a screening works

```
 User draws on canvas (DrawingPad) or uploads/captures a handwriting image (ImageCNN)
                            │
        ┌───────────────────┴───────────────────┐
        ▼                                        ▼
 Signal-based flow                        Image-based flow
 (stroke points: x, y,                    (PNG/JPG image)
  time, pressure, button)
        │                                        │
        ▼                                        ▼
 Backend (Express) stores               Frontend sends image
 image → Cloudinary,                    directly to CNN model
 metadata → MongoDB                     on Hugging Face
        │                                        │
        ▼                                        ▼
 Backend calls Classification            CNN Space runs EfficientNetB0,
 model Space with stroke data            returns prediction + Grad-CAM
        │                                heatmap/overlay
        ▼                                        │
 Prediction + confidence saved                   ▼
 back to the same MongoDB record         Result shown in Patient
        │                                Dashboard (AIAssessment)
        ▼
 Result returned to frontend
 and shown in Patient Dashboard
```

---

## Features

### Patient
- Secure signup/login with role-based JWT auth (patient / admin)
- **Drawing Pad** — spiral and other handwriting tasks used for signal-based PD screening
- **AI Assessment** — upload or capture a handwriting image for CNN-based screening (webcam supported)
- Symptom tracking, medication schedule & reminders, exercise recommendations
- Health journey timeline, progress charts (Recharts)
- PDF report generation for sharing with a doctor
- Caregiver view, community section, live chat, and a global map of care resources

### Admin
- Analytics dashboard
- User & dataset management
- Feedback management and system logs
- Per-patient reports

---

## Tech Stack

**Frontend:** React 19, React Router, Tailwind CSS, Framer Motion, Recharts, Leaflet, Socket.IO Client, Axios, TanStack Query

**Backend:** Node.js, Express 5, MongoDB (Mongoose), JWT auth, Cloudinary, Multer, Socket.IO, Puppeteer

**ML:** EfficientNetB0 CNN + a feature-based classifier, both served as Hugging Face Spaces

**Deployment:** Frontend → Vercel · Backend → Render · ML → Hugging Face Spaces · Database → MongoDB Atlas

---

## Project Structure

```
NeuroSense-AI-PD/
├── backend/
│   ├── controllers/     # auth, dashboard, journey, prediction, profile
│   ├── middlewares/      # JWT auth, role-based access
│   ├── models/            # Mongoose schemas (User, Drawing, Prediction, Medication, ...)
│   ├── routes/             # REST endpoints per feature
│   └── server.js           # Express app + Socket.IO entrypoint
├── frontend/
│   └── src/
│       ├── Pages/            # route-level pages (Home, Dashboard, Auth, ...)
│       ├── Pages/Dashboard/  # Patient & Admin dashboards
│       ├── components/       # PDashboard widgets, AdminDashboard widgets, shared UI
│       └── layouts/          # Patient / Admin / base layouts
└── package.json           # root scripts (runs frontend + backend together)
```

---

## Getting Started

### Prerequisites
- Node.js (LTS) and npm
- A MongoDB Atlas connection string
- A Cloudinary account (for image storage)

### 1. Clone the repo
```bash
git clone https://github.com/SuhasGowda24/NeuroSense-AI-PD.git
cd NeuroSense-AI-PD
```

### 2. Install dependencies
```bash
npm install                    # root (concurrently + shared deps)
cd backend && npm install      # backend deps
cd ../frontend && npm install  # frontend deps
```

### 3. Configure environment variables

Create a `.env` file in `backend/`:
```env
PORT=5000
MONGO_URI=your_mongodb_atlas_connection_string
JWT_SECRET=your_jwt_secret
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
```

Create a `.env` file in `frontend/`:
```env
REACT_APP_API_URL=http://localhost:5000
```

### 4. Run it

From the project root, run frontend and backend together:
```bash
npm run dev
```

Or run them separately:
```bash
cd backend && npm run dev     # http://localhost:5000
cd frontend && npm start      # http://localhost:3000
```

---

## Deployment

| Layer | Platform | Status |
|---|---|---|
| Frontend | Vercel | Deployed |
| Backend | Render | Live — `https://neurosense-ai-pd.onrender.com` |
| ML Models | Hugging Face Spaces | Live |
| Database | MongoDB Atlas | Live |

---

## Related Repositories
- 🧠 [CNN Model — Image-based PD Prediction](https://github.com/SuhasGowda24/ML-CNN-PD-Prediction)
- 📈 [Classification Model — Signal-based PD Prediction](https://github.com/SuhasGowda24/ML-Classification-PD-Prediction)

---

## Disclaimer

NeuroSense AI is a research and educational project. It is **not** a certified medical device and must not be used as a substitute for professional medical diagnosis. Always consult a qualified healthcare provider.

## License

Licensed under the [MIT License](LICENSE).
