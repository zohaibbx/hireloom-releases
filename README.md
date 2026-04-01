# HireLoom

**AI-powered interview assistant that listens, understands, and helps you respond confidently.**

HireLoom is a macOS desktop app built with Flutter that provides real-time AI assistance during live interviews, screen-based assessment solving, a full mock interview simulator with voice interaction, resume review, and interview scheduling with Google Calendar integration.

---

## Features

### General Interview Mode
- Real-time system audio capture via ScreenCaptureKit
- Live transcription powered by Groq Whisper API
- AI-generated answers using LLaMA 3.3-70B
- Smart silence detection — knows when the speaker finishes
- PIP (Picture-in-Picture) overlay — always on top, draggable
- Stealth Mode toggle directly from PIP — hidden from screenshots & screen recordings
- CV-aware responses — AI tailors answers based on your uploaded resume
- Quick action chips: "What should I say?", "Follow-up questions"
- Manual text input for custom questions
- Session history with full conversation replay

### Screen Assist Mode
- AI reads your screen and assists in real-time
- Screen/window picker (like Zoom/Google Meet) — choose entire screen or a specific app window
- Live thumbnails of each window in the picker
- Auto-refreshing window list — detects new/closed windows every 2 seconds
- Captures screenshots every 3 seconds via ScreenCaptureKit
- Smart content change detection — skips duplicate screenshots to save API calls
- Multimodal AI analysis (Llama 4 Scout Vision) — reads MCQs, assessments, quizzes from screenshots
- Displays detected question, all options, highlighted best answer, and explanation
- Typewriter animation for AI explanations
- "Analyze Again" and "Explain More" quick actions
- Text input for follow-up questions about any detected question
- Stealth Mode toggle directly from PIP
- Works with HireVue, TestGorilla, online quizzes, and any screen-based assessment

### AI Mock Interview (Voice-First)
- Fully voice-driven interview simulator
- AI asks questions aloud via macOS text-to-speech
- Auto-starts recording after question finishes
- Auto-stops recording on 3-second silence detection
- Transcription via Groq Whisper, evaluation via LLaMA
- Choose type: Behavioral, Technical, System Design, Case Study
- Choose difficulty: Easy, Medium, Hard
- Set number of questions per session
- Real-time scoring & feedback after each answer (X/10)
- Timed sessions with animated summary scorecard
- Detailed reports generated automatically after each session
- Text input fallback available

### Resume Review
- Upload PDF resumes — parsed with Syncfusion
- AI analyzes resume against target role and job description
- ATS compatibility score, skills match, readability analysis
- Actionable feedback with success/warning/error/tip categories
- Overall score with detailed breakdown

### Interview Reports
- Automatic report generation for mock interviews and live sessions
- Detailed per-question breakdown with scores, answers, and AI feedback
- Overall performance score with animated score ring
- Filter reports by interview type and date range
- Paginated grid view with company, role, and score badges
- Long-press to delete reports from the list view
- Delete button in report detail view with confirmation dialog
- Reports synced to Supabase backend with local JSON fallback
- Download PDF report (placeholder)

### Interview Calendar & Scheduler
- Full month/week calendar view using table_calendar
- Schedule interviews with company, type, date/time, duration, notes & meeting link
- Color-coded event markers by interview type (up to 3 dots per day)
- Functional calendar widget on home sidebar with month navigation and date selection
- Selected day shows events; empty days show upcoming events
- Google Calendar sync — imports events with meeting links
- Quick stats: This Month, This Week, Upcoming counts
- Day events panel & upcoming events sidebar
- Add event dialog with full form (title, company, type, date, time, duration, link, notes)
- Local persistence via JSON + Google Calendar API integration

### Dashboard
- Sidebar navigation with expand/collapse animation
- Orbit layout for interview mode selection with hover effects
- 5 interview modes: General Interview, Mock Interview, Screen Assist, Resume Review, Coding Interview (coming soon)
- Session history with conversation replay
- Functional calendar widget with month navigation, date selection, and event dots
- Staggered entrance animations on page transitions

### Onboarding & Auth
- Animated splash screen with logo reveal
- 4-page swipeable onboarding with parallax effects
- Firebase Authentication (email/password + Google Sign-In)
- Profile management with photo upload

### Settings
- Resume/CV management (upload, replace, remove)
- Profile photo upload (local or Google avatar)
- Stealth Mode toggle
- Permission status for Screen Recording & Microphone
- Response length mode: Brief, Concise, Detailed
- Auto-send delay control
- TTS voice selection (6 OpenAI-style voices)
- App language selection (19 languages)

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Framework | Flutter (macOS) |
| Backend | FastAPI (Python) on Railway |
| Database | Supabase (PostgreSQL) |
| AI Models | LLaMA 3.3-70B, LLaMA 3.1-8B via Groq API |
| Vision AI | Llama 4 Scout 17B via Groq API |
| Transcription | Whisper Large v3 via Groq API |
| TTS | macOS native `say` command |
| Audio Capture | ScreenCaptureKit (native Swift) |
| Screen Capture | ScreenCaptureKit + CGWindowListCreateImage (native Swift) |
| Mic Recording | `record` package + Whisper STT |
| PDF Parsing | Syncfusion Flutter PDF |
| Auth | Firebase Auth + Google Sign-In |
| Calendar | table_calendar + Google Calendar API |
| Storage | Supabase + SharedPreferences + JSON file persistence |
| Window Management | window_manager |
| State Management | Riverpod + Provider |
| Design System | Poppins & Manrope fonts, Slate & Electric Blue theme |

---

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                   Flutter (macOS)                     │
│                                                      │
│  ┌─────────┐  ┌─────────────┐  ┌──────────────────┐ │
│  │ Screens  │  │  Services   │  │   Providers      │ │
│  │          │  │             │  │                  │ │
│  │ Home     │  │ AI Service  │  │ Theme Provider   │ │
│  │ PIP      │◄─┤ Audio Svc   │  │ Auth Provider    │ │
│  │ MCQ PIP  │  │ Screen Svc  │  │ Calendar Provider│ │
│  │ Mock     │  │ Report Svc  │  │ CV Provider      │ │
│  │ Calendar │  │ Auth Svc    │  │ History Provider │ │
│  │ Reports  │  │ CV Svc      │  └──────────────────┘ │
│  │ Resume   │  │ GCal Svc    │                       │
│  └─────────┘  └──────┬──────┘                       │
│                       │                              │
│  ┌────────────────────┴─────────────────────────┐   │
│  │        Native macOS (Swift)                   │   │
│  │  ScreenCaptureKit · CGWindowListCreateImage   │   │
│  │  MethodChannel: audio + screen                │   │
│  └───────────────────────────────────────────────┘   │
└──────────────────────────┬───────────────────────────┘
                           │ HTTPS
              ┌────────────┴────────────┐
              │   FastAPI Backend       │
              │   (Railway)             │
              │                         │
              │  /api/ai/chat           │
              │  /api/ai/transcribe     │
              │  /api/ai/vision         │
              │  /api/ai/analyze-resume │
              │  /api/mock-interview/*  │
              │  /api/reports/*         │
              │  /api/gcalendar/*       │
              └────────────┬────────────┘
                           │
              ┌────────────┴────────────┐
              │  Groq API    Supabase   │
              │  (LLaMA,     (PostgreSQL│
              │   Whisper,    + Auth)   │
              │   Vision)              │
              └─────────────────────────┘
```

---

## Project Structure

```
lib/
  main.dart                          # App entry point
  firebase_options.dart              # Firebase configuration
  
  constants/
    colors.dart                      # Slate & Electric Blue color palette
    fonts.dart                       # Poppins & Manrope typography system
    images.dart                      # Network image URLs + icon constants
    languages.dart                   # 19 languages with TTS voice mappings

  providers/
    ai_provider.dart                 # AI service state
    app_provider.dart                # Global app state
    auth_provider.dart               # Authentication state
    calendar_provider.dart           # Calendar events state
    cv_provider.dart                 # CV/resume state
    history_provider.dart            # Session history state
    theme_provider.dart              # Theme/dark mode state

  screens/
    auth/
      login_screen.dart              # Email/password + Google Sign-In
      signup_screen.dart             # Registration with validation
    calendar/
      calendar_screen.dart           # Full calendar with event management
    home/
      home_screen.dart               # Dashboard, sidebar, mode selection, settings
    mock/
      mock_interview_screen.dart     # Voice-first mock interview simulator
    onboarding/
      onboarding_screen.dart         # 4-page animated onboarding
    pip/
      pip_screen.dart                # PIP overlay for General Interview (audio)
      mcq_pip_screen.dart            # PIP overlay for Screen Assist (vision)
    reports/
      reports_screen.dart            # Interview reports list + detail view
    resume/
      resume_review_screen.dart      # AI-powered resume analysis
    splash/
      splash_screen.dart             # Animated splash with logo reveal
    voice_registration_screen.dart   # Speaker voice profile setup

  services/
    ai_service.dart                  # Groq API: chat, vision, questions, evaluation
    auth_service.dart                # Firebase Authentication wrapper
    cv_service.dart                  # PDF upload, parsing, storage
    deep_link_service.dart           # Deep link / app link handling
    gcalendar_service.dart           # Google Calendar OAuth + event sync
    history_service.dart             # Chat session history persistence
    interview_report_service.dart    # Report CRUD (Supabase + local JSON)
    mock_voice_service.dart          # TTS (macOS say) + mic recording + Whisper
    screen_capture_service.dart      # Screenshot polling + change detection
    system_audio_service.dart        # System audio capture + Whisper transcription
    voice_recorder_service.dart      # Mic recording for voice registration

macos/Runner/
  AppDelegate.swift                  # ScreenCaptureKit: audio capture, screenshot
                                     # capture, window listing, stealth mode
  MainFlutterWindow.swift            # Flutter window configuration
```

---

## Backend Structure

```
hireloom-backend/
  main.py                            # FastAPI app entry point
  requirements.txt                   # Python dependencies
  db/
    supabase.py                      # Supabase client factory
  routers/
    ai.py                            # /api/ai — chat, transcribe, vision, resume
    auth.py                          # /api/auth — authentication
    calendar.py                      # /api/calendar — calendar events
    gcalendar.py                     # /api/gcalendar — Google Calendar integration
    mock_interview.py                # /api/mock-interview — question gen, evaluation
    reports.py                       # /api/reports — CRUD for interview reports
    resume.py                        # /api/resume — resume storage
```

---

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/ai/chat` | LLM chat completion (Groq proxy) |
| POST | `/api/ai/transcribe` | Whisper audio transcription |
| POST | `/api/ai/vision` | Screenshot analysis with multimodal LLM |
| POST | `/api/ai/analyze-resume` | AI resume review and scoring |
| POST | `/api/mock-interview/generate-question` | Generate interview question |
| POST | `/api/mock-interview/evaluate-answer` | Evaluate answer with score |
| POST | `/api/mock-interview/save-report` | Save interview report |
| GET | `/api/reports/{user_id}` | Get all reports for user |
| DELETE | `/api/reports/{report_id}` | Delete a report |

---

## Getting Started

### Prerequisites
- macOS 13.0+
- Flutter 3.10+
- Xcode 15+
- Python 3.10+ (for backend)

### Frontend Setup
```bash
git clone https://github.com/your-username/hireloom.git
cd hireloom
flutter pub get
flutter run -d macos
```

### Backend Setup
```bash
cd hireloom-backend
pip install -r requirements.txt
cp .env.example .env  # Add your API keys
uvicorn main:app --reload
```

### Environment Variables (Backend)
```env
GROQ_API_KEY=your_groq_api_key
SUPABASE_URL=your_supabase_url
SUPABASE_SERVICE_KEY=your_supabase_service_key
ALLOWED_ORIGINS=*
```

### Permissions Required
- **Screen Recording** — for system audio capture and screenshot capture
- **Microphone** — for voice input in mock interviews

---

## How It Works

### General Interview Flow
```
Interview platform audio → ScreenCaptureKit → Whisper transcription
→ Silence detection → AI generates answer → PIP overlay displays response
```

### Screen Assist Flow
```
Window picker → Select screen/window → Screenshot every 3s
→ Change detection → Vision AI reads question + options
→ PIP shows best answer + explanation
```

### Mock Interview Flow
```
AI generates question → TTS reads aloud → User answers via mic
→ Whisper transcription → AI evaluates + scores → Next question
→ Session complete → Report saved
```

---

## Roadmap

- Google Calendar two-way sync
- Technical Coding Copilot (LeetCode, HackerRank)
- Story Studio (STAR Narrative Builder)
- Job Description Analyzer
- Performance Analytics Dashboard
- Voice & Tone Analysis
- Salary Negotiation Coach
- Windows & Linux support

---

## License

This project is proprietary. All rights reserved.
