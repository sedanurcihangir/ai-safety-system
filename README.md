<div align="center">

# 🦺 AI-Powered Real-Time Occupational Safety Violation Detection System

### Real-time AI monitoring for workplace safety

An end-to-end occupational health and safety monitoring platform that analyzes
camera frames with computer vision, detects safety violations, generates
event-based evidence clips, and delivers real-time alerts to web and mobile clients.

<br>

![Java](https://img.shields.io/badge/Java-21-orange?style=for-the-badge&logo=openjdk)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-Backend-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![Python](https://img.shields.io/badge/Python-AI_&_Gateway-3776AB?style=for-the-badge&logo=python&logoColor=white)
![React](https://img.shields.io/badge/React-Web-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![Flutter](https://img.shields.io/badge/Flutter-Mobile-02569B?style=for-the-badge&logo=flutter)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-4169E1?style=for-the-badge&logo=postgresql)
![MinIO](https://img.shields.io/badge/MinIO-Object_Storage-C72E49?style=for-the-badge&logo=minio)
![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?style=for-the-badge&logo=docker&logoColor=white)

</div>

---

## 📌 Overview

This project is an **AI-powered real-time occupational safety violation detection system**
designed primarily for industrial welding environments.

Instead of continuously storing or streaming camera footage, the system analyzes
selected frames in real time and creates evidence only when a verified safety
violation occurs.

A mobile device can act as the camera source during the MVP. Frames are sent to a
dedicated ingestion gateway, sampled for AI inference, evaluated by a temporal
violation engine, and converted into short event-based video clips when necessary.

The system provides:

- 🤖 AI-based PPE and activity detection
- ⏱️ Temporal violation verification
- 📱 Real-device camera streaming
- 🚨 Real-time violation alerts
- 🎬 Automatic event-based evidence clips
- 🖥️ Web operations dashboard
- 📲 Mobile operations application
- 👥 Role-based user and department management
- 📊 Safety analytics and violation history
- 🗺️ Restricted-zone monitoring
- 🔐 Secure access to violation evidence

---

## 🎯 Why This Project?

Industrial safety monitoring traditionally depends heavily on continuous human
observation.

In environments such as welding areas, this creates several challenges:

- Safety violations may be noticed too late.
- Continuous manual camera monitoring is difficult.
- Evidence related to past incidents may not be systematically stored.
- Continuous video storage creates unnecessary bandwidth and storage costs.
- Historical safety data can be difficult to analyze.

This project approaches the problem differently:

> **Analyze continuously, verify intelligently, record only the event.**

Rather than treating every AI detection as a violation, detections are evaluated
over time before a safety event is created.

---

# 🏗️ System Architecture

```text
┌──────────────────────┐
│   Flutter Mobile     │
│   Camera Source      │
└──────────┬───────────┘
           │ JPEG Frames
           ▼
┌──────────────────────┐
│ Camera Ingestion     │
│ Gateway              │
│                      │
│ • Session Management │
│ • Ring Buffer        │
│ • Frame Sampling     │
│ • Backpressure       │
└──────────┬───────────┘
           │ ~3 FPS
           ▼
┌──────────────────────┐
│      AI Worker       │
│                      │
│ • Object Detection   │
│ • PPE Detection      │
│ • Class Mapping      │
└──────────┬───────────┘
           │ Detection Events
           ▼
┌────────────────────────────┐
│ Spring Boot Backend        │
│                            │
│ • Authentication           │
│ • Users / Cameras          │
│ • Violation Engine         │
│ • Temporal Verification    │
│ • Notifications            │
│ • Reporting                │
└───────┬────────────┬───────┘
        │            │
        │            │ Recording Command
        │            ▼
        │    ┌──────────────────┐
        │    │ Event Recorder   │
        │    │                  │
        │    │ Ring Buffer      │
        │    │       ↓          │
        │    │ FFmpeg → MP4     │
        │    └────────┬─────────┘
        │             │
        │             ▼
        │       ┌──────────┐
        │       │  MinIO   │
        │       │ MP4/JPG  │
        │       └──────────┘
        │
        │ WebSocket / REST
        ▼
┌────────────────────────────┐
│        User Clients        │
│                            │
│  React Web   Flutter App   │
└────────────────────────────┘
```

---

# 🔄 How It Works

### 1. Camera Streaming

The Flutter application acts as a camera source and sends JPEG frames to the
**Camera Ingestion Gateway**.

The Gateway manages:

- camera sessions
- frame queues
- backpressure
- connection lifecycle
- stale-session cleanup
- in-memory ring buffers

---

### 2. AI Frame Sampling

Sending every incoming frame to the AI model would unnecessarily increase
inference load.

The Gateway therefore samples the incoming stream at approximately **3 FPS**
before forwarding frames to the AI Worker.

```text
Mobile Camera
     │
     │ ~15 FPS
     ▼
   Gateway
     │
     │ ~3 FPS sampled
     ▼
 AI Worker
```

---

### 3. AI Detection

The AI Worker analyzes frames and produces normalized detection results.

The canonical detection contract contains **9 classes**:

| Class | Description |
|---|---|
| `person` | Person detection |
| `gloves` | Protective gloves |
| `welding_jacket` | Welding jacket |
| `welding_apron` | Welding apron |
| `welding_mask` | Welding mask |
| `welding` | Welding activity/context |
| `non_welding_mask` | Missing / unsuitable welding mask |
| `non_welding_jacket` | Missing / unsuitable welding jacket |
| `non_gloves` | Missing gloves |

AI detections themselves are **not directly considered violations**.

---

# 🧠 Temporal Violation Detection

One of the core design decisions of the system is avoiding single-frame
violation decisions.

```text
Frame 1     Detection
Frame 2     Detection
Frame 3     Detection
Frame 4     Detection
   │
   ▼
Temporal Evaluation
   │
   ▼
Confirmed Violation
```

The backend evaluates detections using:

- temporal confirmation
- person/PPE association
- welding context
- restricted-zone information
- violation lifecycle
- cooldown rules

This helps reduce false positives caused by temporary or inaccurate detections.

---

# 🎬 Event-Based Recording

The platform does **not** continuously archive the full camera stream.

Instead, the Gateway maintains a short in-memory **ring buffer** containing the
most recent frames.

When a violation is confirmed:

```text
       Ring Buffer
           │
           ▼
┌─────────────────────────────┐
│ Pre-event footage           │
│        +                    │
│ Violation duration          │
│        +                    │
│ Post-roll footage           │
└──────────────┬──────────────┘
               │
               ▼
            FFmpeg
               │
               ▼
             MP4
               │
               ▼
             MinIO
```

The resulting evidence contains the context immediately before the violation,
the violation itself, and a short post-event period.

This reduces unnecessary storage while preserving useful incident context.

---

# 🚨 Real-Time Notifications

Once a violation is confirmed, authorized clients receive realtime updates.

The notification lifecycle can include:

```text
Detection
   ↓
Candidate Violation
   ↓
Confirmed Violation
   ↓
Realtime Alert
   ↓
Recording Processing
   ↓
Clip Ready
   ↓
Client Update
```

Realtime events are delivered to both the web and mobile applications, while
REST endpoints remain the source of truth for recovery after reconnection.

---

# 🖥️ Web Application

The React web application provides the primary desktop operations and
administration interface.

### Main Features

- Dashboard
- Safety analytics
- Violation history
- Violation filtering
- Violation detail
- Evidence clip playback
- Violation review
- Camera management
- User management
- Realtime notifications
- Role-based authorization
- Restricted-zone configuration

### Restricted Zones

Administrators can define restricted areas using polygons drawn over a camera
reference image.

Coordinates are normalized between `0..1`, allowing them to remain independent
of the displayed image resolution.

---

# 📱 Mobile Application

The Flutter application has two responsibilities.

### Camera Source

The phone can act as the physical camera source used by the system.

It manages:

- camera preview
- Gateway connection
- session lifecycle
- frame transmission
- heartbeat
- reconnect
- connection state

### Operations Client

The application also provides mobile access to:

- Dashboard
- Cameras
- Violations
- Violation details
- Evidence clips
- Users
- Notifications
- Authentication

Restricted-zone editing intentionally remains **web-only**.

---

# 🔐 Authentication & Authorization

The backend uses JWT-based authentication and role-aware authorization.

Example roles include:

```text
ADMIN
OHS_SPECIALIST
SHIFT_SUPERVISOR
```

Authorization is enforced by the backend rather than relying only on hidden
frontend components.

Users can access resources according to their assigned role and department.

---

# 📊 Violation Lifecycle

Violations and recordings have independent lifecycle states.

### Violation

```text
ACTIVE
   ↓
RECORDING_PREPARING
   ↓
COMPLETED
```

Error conditions may transition the violation to:

```text
ERROR
```

### Review

```text
UNREVIEWED
REVIEWED
CONFIRMED
FALSE_ALARM
```

### Recording

```text
REQUESTED
   ↓
RECORDING
   ↓
PROCESSING
   ↓
READY
```

Keeping these states separate makes it possible to distinguish the safety event,
human review state, and media-processing state.

---

# 🧰 Technology Stack

| Layer | Technology |
|---|---|
| Web Frontend | React / TypeScript |
| Mobile | Flutter / Dart |
| Backend | Java 21 / Spring Boot |
| AI Worker | Python |
| Camera Gateway | Python |
| Database | PostgreSQL |
| Object Storage | MinIO |
| Video Processing | FFmpeg |
| Realtime | WebSocket / STOMP |
| Authentication | JWT |
| Containerization | Docker |

---

# 📂 Project Structure

```text
ai-safety-system/
│
├── backend/
│   └── Spring Boot backend
│
├── ai-service/
│   └── AI inference worker
│
├── gateway/
│   └── Camera ingestion & event recording
│
├── frontend/
│   └── React web application
│
├── mobile/
│   └── Flutter mobile application
│
├── docker-compose.yml
│
└── README.md
```

> Directory names may differ slightly depending on the current development branch.

---

# ⚙️ Runtime Architecture

Default local service ports:

| Service | Port |
|---|---:|
| Spring Boot Backend | `8080` |
| Camera Ingestion Gateway | `8000` |
| AI Worker | `8001` |
| PostgreSQL | `5432` |
| MinIO API | `9000` |

Default MinIO bucket:

```text
violation-media
```

Configuration should be provided through environment variables rather than
hard-coded runtime values.

Secrets and environment-specific credentials should **never be committed to the repository**.

---

# 🚀 Running the Project

## Prerequisites

Make sure the following are installed:

- Java 21+
- Python
- Flutter SDK
- Node.js
- Docker
- FFmpeg

---

## 1. Clone the Repository

```bash
git clone <repository-url>
cd ai-safety-system
```

---

## 2. Configure Environment

Create the required environment files using the provided examples.

```bash
cp .env.example .env
```

Update host addresses and credentials according to your local environment.

> When using a physical Android device, `localhost` refers to the phone itself.
> The Gateway must therefore be reachable through the computer's network address
> or another reachable network interface.

---

## 3. Start Infrastructure

```bash
docker compose up -d
```

Verify that PostgreSQL and MinIO are running before starting application services.

---

## 4. Start Backend

```bash
cd backend
./mvnw spring-boot:run
```

Windows:

```powershell
.\mvnw.cmd spring-boot:run
```

---

## 5. Start AI Worker

```bash
cd ai-service

python -m venv .venv
```

Activate the environment and install dependencies:

```bash
pip install -r requirements.txt
```

Then start the AI service using the project's configured entrypoint.

---

## 6. Start Camera Gateway

```bash
cd gateway

python -m venv .venv
pip install -r requirements.txt
```

Start the Gateway using the configured application entrypoint.

---

## 7. Start Web Application

```bash
cd frontend
npm install
npm run dev
```

---

## 8. Start Mobile Application

```bash
cd mobile
flutter pub get
flutter run
```

For a physical Android device:

```bash
flutter devices
flutter run -d <device-id>
```

---

# 🧪 End-to-End Flow

The complete MVP pipeline is:

```text
Physical Android Device
        │
        ▼
Camera Ingestion Gateway
        │
        ▼
AI Worker
        │
        ▼
Spring Boot Backend
        │
        ▼
Temporal Violation Engine
        │
        ├──────────────► WebSocket Alert
        │
        ▼
Event Recorder
        │
        ▼
FFmpeg
        │
        ▼
MinIO
        │
        ▼
Recording READY
        │
        ▼
Web / Mobile Evidence Playback
```

A successful end-to-end test demonstrates that the system can move from a real
camera frame to an AI detection, verified violation, realtime notification and
playable evidence clip.

---

# 🛡️ Design Principles

### Event-Based Instead of Continuous Recording

Only safety events are stored, reducing storage and bandwidth requirements.

### AI Detection ≠ Violation

The AI model provides observations. Business rules and temporal verification
determine whether those observations become actual violations.

### Backend-Enforced Authorization

Frontend visibility rules improve UX, but security decisions remain on the
backend.

### REST as Source of Truth

Realtime events improve responsiveness, while REST recovery prevents state loss
after reconnects.

### Modular Architecture

The core backend remains a modular monolith while compute-heavy media ingestion
and AI inference run as independent components.

---

# 🔮 Future Improvements

Potential post-MVP improvements include:

- RTSP / CCTV camera integration
- Horizontal Gateway scaling
- Distributed AI workers
- Production-grade camera authentication
- Push notifications
- Advanced analytics
- Multi-site deployment
- Model optimization
- Automated retention policies
- Improved observability and monitoring

---

# ⚠️ Project Scope

This project is designed to **assist occupational safety monitoring**.

It is not intended to replace:

- occupational safety professionals
- physical safety barriers
- industrial interlocks
- organizational safety procedures

The system does not perform employee identity tracking or facial recognition.

---

# 👥 Contributors

Developed collaboratively as an AI-assisted occupational safety monitoring platform.

| Area | Responsibility |
|---|---|
| Backend | Authentication, cameras, violations, recording, reporting |
| AI | Detection and model integration |
| Gateway | Camera ingestion, buffering and event recording |
| Web | Operations and administration dashboard |
| Mobile | Camera streaming and mobile operations |

---

<div align="center">

### 🦺 Safer workplaces through intelligent monitoring.

**AI • Computer Vision • Real-Time Systems • Occupational Safety**

</div>