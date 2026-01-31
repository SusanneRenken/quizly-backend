# Quizly – Backend

Quizly automatically converts YouTube videos into quizzes with 10 AI‑generated questions.  
This backend provides the core API for authentication, quiz creation (stub or production AI pipeline), and quiz management.

---

## Table of Contents
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Quickstart](#quickstart-development)
- [Environment Variables](#environment-variables)
- [API Reference](#api-reference)
  - [Authentication](#authentication)
  - [createQuiz – AI / Stub Pipeline](#createquiz--ai--stub-pipeline)
  - [Quiz Endpoints](#quiz-endpoints)
- [Error Handling](#error-handling)
- [Tests & Coverage](#tests--coverage)
- [Project Structure](#project-structure)

---

## Features

- User registration & login (JWT via HttpOnly cookies)
- Quiz generation from YouTube URLs (stub mode or Whisper + Gemini)
- Manage own quizzes (List, Detail, Update, Delete)
- Clear error handling and clean API structure
- Full automated test suite with high coverage

---

## Tech Stack

- Python 3.12  
- Django 5 + DRF  
- ffmpeg (must be installed globally)
- yt_dlp  
- Whisper  
- Gemini Flash (via google-genai)

---

## Installation / Quickstart

### FFmpeg Note

FFmpeg must be installed globally for Whisper to work.

Check if FFmpeg is installed:
```bash
ffmpeg -version
```

### Clone the repository
```bash
git clone <repo-url>
cd Quizly-backend
```

### Rename .env.template and set keys
- Rename `.env.template` to `.env`.
- Open `.env` and set your values:
```bash
SECRET_KEY=your-django-secret-key
GEMINI_API_KEY=your-gemini-api-key
QUIZLY_PIPELINE_MODE = "prod" # or "stub"
```
→ Pipeline mode explanation: [CreateQuiz – AI / Stub Pipeline](#createquiz--ai--stub-pipeline).

### Create a virtual environment
```bash
python -m venv env
.\env\Scripts\activate.ps1  # Windows
```

### Install dependencies
```bash
pip install -r requirements.txt
```

### Create migrations
```bash
python manage.py makemigrations
```

### Apply migrations
```bash
python manage.py migrate
```

### Start the development server
```bash
python manage.py runserver
```

API base URL: http://localhost:8000/api/  
Admin panel: http://localhost:8000/admin/

---
## Environment Variables

The project uses a `.env` file in the root directory.

Required variables:

| Variable | Description |
|----------|-------------|
| `SECRET_KEY` | Django secret key |
| `GEMINI_API_KEY` | API key for Gemini Flash |
| `QUIZLY_PIPELINE_MODE` | `stub` or `prod` (default: `stub`) |

---

## API Endpoints
All endpoints require authentication unless stated otherwise.  
Responses follow a consistent JSON structure and error codes are described in the Error Handling section.

### Authentication

The project uses JWT authentication via HttpOnly cookies.

Available endpoints:

| Method | Endpoint | Description |
|--------|----------|--------------|
| POST | `/api/register/` | Create a new user account with username, email, and password. |
| POST | `/api/login/` | Authenticate a user and return JWT tokens via HttpOnly cookies. |
| POST | `/api/logout/` | Log the user out by clearing the authentication cookies. |
| POST | `/api/token/refresh/` | Issue a new access token using the refresh token stored in the cookie. |

### createQuiz – AI / Stub Pipeline

Endpoint for converting a YouTube URL into a quiz.

| Method | Endpoint | Description |
|--------|----------|--------------|
| POST | `/api/createQuiz/` | Generate a quiz from a YouTube URL using either the stub pipeline or the full AI pipeline. |

Example request:

```json
{ "url": "https://www.youtube.com/watch?v=XXXXXXXXXXX" }
```

Modes:

- `QUIZLY_PIPELINE_MODE=stub` – Fast development mode
Returns a predefined example quiz without running any external tools.
This mode skips downloading the video, audio extraction, transcription, and AI generation.
Useful for local development and quick end-to-end testing.

- `QUIZLY_PIPELINE_MODE=prod` – Full AI production mode
Runs the complete pipeline:
YouTube download → audio extraction (ffmpeg) → transcription (Whisper) → quiz generation (Gemini Flash).
Requires ffmpeg, yt_dlp, Whisper, and a valid Gemini API key.

### Quiz Endpoints

Users can only access their own quizzes.

| Method | Endpoint | Description |
|--------|----------|--------------|
| GET | `/api/quizzes/` | List all quizzes of the authenticated user |
| GET | `/api/quizzes/{id}/` | Retrieve a single quiz |
| PATCH | `/api/quizzes/{id}/` | Update only the `title` field |
| DELETE | `/api/quizzes/{id}/` | Delete a quiz including its questions |

---

## Tests & Coverage

```bash
coverage erase
coverage run manage.py test
coverage report
```
---
## Error Handling

The API returns clear error messages with appropriate HTTP status codes:

- `400 Bad Request` – invalid input (e.g., malformed YouTube URL)
- `401 Unauthorized` – missing or invalid authentication
- `403 Forbidden` – accessing someone else's quiz
- `404 Not Found` – quiz does not exist
- `500 Internal Server Error` – unexpected server error (uncaught exception during request processing)
- `502 Bad Gateway` – technical error during AI pipeline (audio extraction, Whisper, Gemini)

---

## Project Structure

```
quizly-backend/
├── auth_app/
│   ├── api/
│   │   ├── authentication.py
│   │   ├── serializer.py
│   │   ├── urls.py
│   │   ├── view.py
│   ├── tests/
└── core/
│   ├── settings.py
│   ├── urls.py
├── quizzes_app/
│   ├── api/
│   │   ├── permissions.py
│   │   ├── serializer.py
│   │   ├── urls.py
│   │   ├── view.py
│   ├── services/
│   │   ├── error.py
│   │   ├── persist_quiz.py
│   │   ├── quiz_pipeline_prod.py
│   │   ├── quiz_pipeline_stub.py
│   ├── tests/
│   ├── admin.py
│   ├── models.py
└── manage.py
```
