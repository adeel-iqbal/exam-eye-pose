# 👁️ ExamEye — AI Exam Monitoring System

Upload an exam surveillance video and ExamEye automatically detects every student, tracks them across frames, and flags suspicious behavior like head turns, body rotation, and sudden movement.

[![Python](https://img.shields.io/badge/Python-3.12%2B-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115-009688.svg)](https://fastapi.tiangolo.com/)
[![YOLO](https://img.shields.io/badge/YOLO-11n--pose-00FFFF.svg)](https://github.com/ultralytics/ultralytics)
[![Next.js](https://img.shields.io/badge/Next.js-15-black.svg)](https://nextjs.org/)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.x-5C3EE8.svg)](https://opencv.org/)

---

## 🎯 Overview

ExamEye is a three-step pipeline: upload a video, run the analysis, get a full monitoring report. YOLO11n-pose detects every person in each frame and returns 17 body keypoints per person. ByteTrack assigns each student a persistent ID so they are tracked reliably across frames. The analyzer then checks each person's keypoints every frame using geometric rules — no confidence guessing, pure position-based logic — and flags behaviors that indicate potential cheating.

The web interface streams live annotated frames to the browser as processing runs. Students are outlined in green when clean and red when flagged. A real-time sidebar shows per-student flag counts and the latest detected action. When processing finishes, a full session report is generated with a downloadable annotated output video.

---

## ✨ Features

- 🧍 **Pose Detection** — YOLO11n-pose detects every student and returns 17 COCO body keypoints per person per frame
- 🔁 **Persistent Tracking** — ByteTrack maintains stable student IDs across frames so the same student is never double-counted
- 🔴 **Live Color Coding** — Skeleton overlay turns red the moment a flag is detected and back to green when behavior is clean
- 📡 **Live Streaming** — Annotated frames stream to the browser in real time over WebSocket during processing
- 📋 **Per-Student Sidebar** — Live flag count and last detected action for each student during analysis
- 🎬 **Annotated Output Video** — Full annotated MP4 saved automatically, downloadable from the report screen
- 📊 **Session Report** — Total students, total flags, most flagged student, per-student breakdown, and full timestamped event log
- 🗂️ **Video Management** — Upload, browse, and delete source videos from the UI

---

## 🚨 What Gets Flagged

| Behavior | How It Is Detected |
|---|---|
| **Head Turn** | Three independent geometric methods — ear confidence asymmetry, nose offset from ear midpoint, and nose-to-eye distance ratio |
| **Body Turn** | One shoulder significantly higher than the other (student twisting to talk to a neighbour) |
| **Looking Up** | Nose rises more than 30 pixels above the ear level line |
| **Sudden Movement** | Shoulder midpoint rises more than 45 pixels above the student's rolling baseline (standing up or large shift) |

Each flag has a 2-second cooldown per student — one continuous head turn logs one event, not hundreds.

---

## 🔄 How It Works

| Step | What Happens |
|---|---|
| **1. Upload** | Upload a surveillance video or pick one already on the server |
| **2. Analyze** | YOLO11n-pose + ByteTrack process every frame; flags stream live to the browser |
| **3. Report** | Final stats, annotated output video, and full event log |

### Detection Pipeline

- YOLO11n-pose runs on every frame returning bounding boxes and 17 keypoints (x, y, confidence) per person
- ByteTrack assigns each person a persistent track ID for the full video
- `analyzer.py` runs on each person's keypoints every frame using geometric rules that work purely on position, not on confidence scores alone
- `StudentTracker` converts per-frame flags into logged events with timestamps, applying a 2-second cooldown per student per action type
- Student count uses the mode over a 90-frame rolling window to ignore brief spurious extra detections from the tracker
- Skeleton color is set per-frame: red if any flag is active this frame, green if clean

---

## 🖼️ Screenshots

<table align="center">
  <tr>
    <td align="center" width="100%">
      <img src="assets/UI.png" alt="ExamEye UI" width="100%"/>
      <br/><sub><b>ExamEye interface — upload, live monitoring, and session report screens</b></sub>
    </td>
  </tr>
</table>

---

## 🎬 Sample Output Videos

GitHub does not support video playback. Download the sample annotated output videos from the links below to see the skeleton overlays, color coding, and overlay stats in action.

- [Output.mp4](assets/Output.mp4) — Sample processed video with skeleton overlays and flag annotations
- [Output2.mp4](assets/Output2.mp4) — Second sample showing multi-student tracking and body turn detection

---

## 🛠️ Tech Stack

### Detection & Tracking
- **YOLO11n-pose** — Pose estimation, 17 COCO keypoints per person per frame
- **ByteTrack** — Multi-object tracking with persistent IDs (via Ultralytics)
- **OpenCV** — Frame processing and annotation drawing

### Backend
- **FastAPI** — REST API and WebSocket server
- **imageio / libx264** — Annotated output video encoding
- **Python 3.12+**

### Frontend
- **Next.js 15** — Three-screen UI flow (upload, processing, report)
- **Tailwind CSS** — Dark clinical theme
- **WebSocket** — Live frame and event streaming from backend to browser

---

## 🚀 Getting Started

### Prerequisites

- Python 3.12+
- Node.js 18+
- An exam surveillance video (MP4, AVI, MOV, or MKV)

### Backend Setup

1. **Clone the repository**
```bash
git clone https://github.com/adeel-iqbal/exam-eye.git
cd exam-eye
```

2. **Create a virtual environment**
```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
```

3. **Install dependencies**
```bash
pip install --prefer-binary -r requirements.txt
```

> Note: Use `--prefer-binary` to avoid building OpenCV from source. Tested on Python 3.12 with numpy 1.26.4.

4. **Model weights**

`yolo11n-pose.pt` downloads automatically from Ultralytics on first run. No manual download needed.

5. **Run the backend**
```bash
uvicorn backend.main:app --host 0.0.0.0 --port 8002
```

The model takes 15-20 seconds to load on first startup. The API is ready when you see `Pose model loaded`.

### Frontend Setup

1. **Navigate to the frontend directory**
```bash
cd frontend
```

2. **Install dependencies**
```bash
npm install
```

3. **Start the dev server**
```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## 💻 Usage

1. Open the app and upload an exam surveillance video, or select one already on the server
2. Click **Start Analysis**
3. Watch the live annotated feed — green skeletons are clean, red are flagged
4. Monitor the sidebar for per-student flag counts and the live alert feed
5. When processing finishes, review the session report
6. Watch or download the annotated output video
7. Click **New Session** to run again with a different video

---

## 📓 Development Notebook

`exameye_dev.ipynb` is a Colab-compatible notebook that walks through the full pipeline cell by cell:

- Load YOLO and inspect raw keypoint output
- Visualize skeleton overlays on individual frames
- Test and print detection logic values (ear confidence, nose ratios, eye asymmetry, shoulder angles)
- Process a full video and download the annotated output
- Threshold tuning playground — change a value and see how detection count changes across sampled frames

---

## 📁 Project Structure

```
exam-eye/
│
├── backend/
│   ├── main.py                # FastAPI app, WebSocket processing loop
│   ├── detector.py            # YOLO11n-pose inference, ByteTrack tracking
│   ├── analyzer.py            # Pose analysis rules and StudentTracker class
│   ├── drawer.py              # Skeleton and overlay drawing
│   └── bytetrack_exam.yaml    # ByteTrack tuning (thresholds, buffer size)
│
├── frontend/
│   ├── app/
│   │   ├── page.tsx           # Upload, processing, and report screens
│   │   └── globals.css        # Dark clinical theme
│   └── package.json
│
├── models/
│   └── yolo11n-pose.pt        # Auto-downloaded on first run
│
├── videos/                    # Uploaded source videos
├── outputs/                   # Annotated output videos
├── assets/                    # Screenshots
├── exameye_dev.ipynb          # Development and prototyping notebook
├── requirements.txt
└── README.md
```

---

## ⚠️ Disclaimer

Detection accuracy depends on video quality, camera angle, lighting, and how many students are in frame. The system flags behavior geometrically and will miss subtle cheating that does not involve head or body movement. Do not use flags as definitive evidence of cheating without human review.

---

## 📧 Contact

**Adeel Iqbal**

- 📧 Email: [ai@rankmeglobal.com](mailto:ai@rankmeglobal.com)
- 💼 LinkedIn: [linkedin.com/in/adeeliqbalmemon](https://linkedin.com/in/adeeliqbalmemon)
- 🐙 GitHub: [@adeel-iqbal](https://github.com/adeel-iqbal)

---

<div align="center">
  <p>Made with ❤️ by Adeel Iqbal</p>
  <p>⭐ Star this repo if you find it useful!</p>
</div>
