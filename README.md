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
  <a href="https://neurosense-ai-pd.onrender.com"><img src="https://img.shields.io/badge/Backend%20API-Live-brightgreen?style=for-the-badge&logo=render&logoColor=white" /></a>
  <a href="https://github.com/SuhasGowda24/ML-CNN-PD-Prediction"><img src="https://img.shields.io/badge/CNN%20Model-Repo-orange?style=for-the-badge&logo=github&logoColor=white" /></a>
  <a href="https://github.com/SuhasGowda24/ML-Classification-PD-Prediction"><img src="https://img.shields.io/badge/Classification%20Model-Repo-orange?style=for-the-badge&logo=github&logoColor=white" /></a>
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

### CNN Model WorkFlow
We have used pre-trained mode where this model can recognize  and extract the features.(like a human brain where it can differentiate among different shapes ,lines and colors).

To increase the efficiency of the model , data augmentation is done. During training, the Model never sees the exact same image twice. For every batch of 16 images, the system randomly:

- Rotates the drawing left or right.

- Zooms in and out.

- Flips the image horizontally.

By doing this, the model can’t just memorize a specific picture. It is forced to learn the actual patterns of Parkinson's like subtle line tremors (fuzziness), variations in line thickness, and irregular gaps between the spiral loops.

We set the model to train for 50 epochs (50 full rounds of learning) to ensure it extracts as much detail as possible. However, to prevent overfitting (where the model becomes too obsessed with the training data and fails in the real world), we use Early Stopping. This acts like a referee that stops the training the moment the model's performance on new data stops improving,  and ensuring we keep the "best version" of the Model.


Instead of just model gives a result without explanation. By integrating Grad-CAM Heatmaps, we provide Transparency.

The system generates a colored overlay on top of the original drawing:

- Red zones show exactly where the model found the most suspicious tremors or irregularities.

- This helps doctors and users see why the model made its decision.

- It often highlights micro-patterns in the handwriting that are nearly invisible to the naked human eye, building trust and confidence in the result.

### Classification Model WorkFlow

The second model looks at the process and the movement. It captures things a standard camera or the human eye could never see.

Using a digital tablet and stylus, we collect raw signals like the pen's position, pressure, and even the angle of the pen (azimuth/altitude). We then transform these into Kinematic Features:

- Velocity & Acceleration: Parkinson's often causes "bradykinesia" (slowness of movement). The model detects if the drawing speed is unusually slow or inconsistent.

- Jerk (The "Tremor" Metric): This is one of the most important features. "Jerk" measures how smooth the movement is. High "jerk" values indicate the micro-tremors and shakiness typical of PD.

- Pen Lifts & Air-Surface Ratio: This captures "hesitation." The model tracks how much time the patient spends with the pen in the air versus on the paper. Frequent lifting or long pauses can indicate motor planning difficulties.

A key feature here is Pressure. PD patients often struggle with "micrographia" (writing that gets smaller and smaller) or inconsistent pressure because of muscle stiffness. Our model analyzes the Mean, Max, and Standard Deviation of pressure to detect these subtle neurological "stiffness" signatures.

The model is trained with a specialized task of 8 different characters
Each task tests a different part of the brain's motor control:

- Spirals test continuous coordination.

- Sentences test cognitive-motor transitions (thinking while moving).

- Single letters test fine motor precision.

By having both models, your dashboard becomes a Double-Check System:

- The CNN (Vision): Identifies visual "fuzziness" and irregular gaps in the final image.

- The Kinematic Model (Motion): Identifies the "hidden" tremor (jerk), the slowness (velocity), and the struggle with pressure.

A prediction that isn't just based on a picture, but on the actual physical behavior of the patient. It’s like having a neurologist "watch" the patient draw in super-slow motion through the data.

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

**Frontend:** React 19, React Router, Tailwind CSS

**Backend:** Node.js, Express 5, MongoDB (Mongoose), Cloudinary

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
- [CNN Model — Image-based PD Prediction](https://github.com/SuhasGowda24/ML-CNN-PD-Prediction)
- [Classification Model — Signal-based PD Prediction](https://github.com/SuhasGowda24/ML-Classification-PD-Prediction)

---

## Disclaimer

NeuroSense AI is a research and educational project. It is **not** a certified medical device and must not be used as a substitute for professional medical diagnosis. Always consult a qualified healthcare provider.

## License

Licensed under the [MIT License](LICENSE).
