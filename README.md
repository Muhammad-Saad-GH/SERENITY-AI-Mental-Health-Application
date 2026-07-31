# Serenity

This Mental Health AI System uses emotion detection and intelligent text analysis to help users understand and manage stress. The system uses a webcam and text input to identify emotional states, while users can log upcoming events to receive predicted stress levels based on event type. All emotional data and event logs are securely stored in a MySQL database to track trends over time. The system prioritizes privacy and security, ensuring no unnecessary data is collected and all personal information remains protected.

Devpost Link: [Serenity](https://devpost.com/software/serenity-ludmaf)

**Built at HackHive 2025 in 24 hours · React · TypeScript · FastAPI · MySQL · Hugging Face Transformers**

<h2>What it does</h2>

Stress tends to be recognised late. People notice they are struggling after a bad week, not during it, and the events that caused it are often ones they knew about in advance.

Serenity approaches that from three directions at once. It reads emotional state from a webcam snapshot using a facial expression model. It reads sentiment from whatever the user writes. And it lets the user log events that have not happened yet, assigning each an expected stress level so the pressure ahead is visible before it arrives. Everything is timestamped into a database so patterns emerge across days rather than being judged one moment at a time.

The result is a check-in page where a user can capture how they feel, write about it, log what is coming, and see all three combined.

> This is a hackathon prototype built in 24 hours. It is not a diagnostic tool and it does not provide clinical advice.

<h2>Features</h2>

**Facial emotion detection** — capture a webcam snapshot from the browser, classify the expression, and store the result with its confidence score.

**Text sentiment analysis** — submit free-text and receive a sentiment label with a confidence score, so what a user writes is assessed alongside what their face shows.

**Event logging with stress prediction** — log an upcoming event and receive two independent stress readings: one from the event category, one from current emotional state.

**Journaling** — attach written entries to logged events.

**Accounts** — registration and login with bcrypt-hashed passwords.

**History** — emotions, events, and journal entries are all timestamped in MySQL so trends can be tracked over time.

<h2>Architecture</h2>

```
React + TypeScript (Vite)
   │  WebcamCapture · TextAnalyzer · EventLogger
   │
   ▼
Express proxy (server.js, port 8001)
   │  receives multipart uploads, forwards to the Python service
   ▼
FastAPI (port 8000)
   │
   ├── /detect_emotion/  ──► image-classification model ──┐
   ├── /analyze_text/    ──► sentiment-analysis model ────┤
   ├── /log_event/       ──► rule-based stress classifier ┤
   ├── /log_journal/     ─────────────────────────────────┤
   └── /users/register, /users/login (bcrypt) ────────────┤
                                                          ▼
                                      SQLAlchemy ORM ──► MySQL
                                      users · events · emotions · journals
```

<h2>The models</h2>

| Purpose | Model | What it returns |
| --- | --- | --- |
| Facial emotion | `dima806/facial_emotions_image_detection` (Hugging Face) | An emotion label plus a confidence score, from a single webcam frame |
| Text sentiment | Hugging Face default `sentiment-analysis` pipeline | A sentiment label plus a confidence score for free-text input |
| Event stress | Rule-based keyword classifier | An event type and an expected stress level from 1 to 10 |

The event classifier is deliberately not a model. Event names map to stress levels by keyword: exams and tests score 9, midterms 8, quizzes 6, assignments 5, anything unrecognised 3. Emotional state maps separately, with angry at 9, sad at 8, neutral at 5, and happy at 2. Both numbers are stored, so an event carries a prediction from what it is and a reading from how the user felt when logging it.

<h2>Key terms</h2>

| Term | Meaning |
| --- | --- |
| **Image classification** | Assigning a label to a picture. Here the labels are emotions and the picture is a single webcam frame. |
| **Sentiment analysis** | Scoring text as positive or negative. A confidence score accompanies each label, so weak signals can be told apart from strong ones. |
| **Confidence score** | How certain the model is, from 0 to 1. Stored alongside every prediction, so a low-confidence reading is not treated as fact. |
| **Pipeline (Hugging Face)** | A one-line wrapper that loads a pretrained model and handles the preprocessing around it, rather than training a model from scratch. |
| **ORM** (object-relational mapping) | Writing database operations as Python objects instead of raw SQL. SQLAlchemy generates the SQL and the table definitions live in `models.py`. |
| **bcrypt** | A password hashing algorithm designed to be slow, which makes brute-force guessing expensive. Passwords are never stored as written. |
| **FastAPI** | A Python web framework that validates request bodies against declared schemas and generates interactive API documentation automatically. |

<h2>Tech stack</h2>

| Layer | Technology |
| --- | --- |
| Frontend | React 19, TypeScript, Vite, React Router, react-webcam, Framer Motion, CSS Modules |
| Proxy | Node.js, Express, Multer, Axios |
| API | Python, FastAPI, Uvicorn, Pydantic schemas |
| ML | Hugging Face Transformers, Pillow |
| Database | MySQL, SQLAlchemy ORM, PyMySQL |
| Auth | passlib, bcrypt |

<h2>Running locally</h2>

Requires Python 3.10+, Node.js 18+, and a running MySQL server with a database named `serenity`.

**API**

```bash
cd backend
pip install fastapi uvicorn sqlalchemy pymysql transformers pillow passlib bcrypt python-multipart
uvicorn main:app --reload --port 8000
```

Interactive API docs are then available at `http://localhost:8000/docs`.

**Proxy**

```bash
cd backend
npm install
node server.js
```

**Frontend**

```bash
cd frontend
npm install
npm run dev
```

The first request to an ML endpoint downloads the model weights from Hugging Face, so expect a delay on the initial call.

Connection settings live in `backend/models/database.py`. Tables are created automatically on first startup.

## Team

| GitHub Username | Name |
| --- | --- |
| Muhammad-Saad-GH | Muhammad Saad |
| Ajay0-0Ram | Ajay Ramsaran |
| SameerKarodia | Sameer Karodia |
| tabishghouri | Tabish Ghouri |
