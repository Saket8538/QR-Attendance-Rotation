# Smart Attendance Management System with real-time sync and QR rotation

A comprehensive web-based attendance management system built with Python Flask, featuring real-time QR code scanning, dynamic QR rotation, cloud synchronization, and real-time updates via WebSocket.

## 🚀 Quick Start

### Prerequisites
- Python 3.8+
- pip (Python package manager)

### Installation

1. **Navigate to the project directory:**
   ```bash
   cd python_attendance_system
   ```

2. **Create and activate virtual environment:**
   ```bash
   python -m venv venv
   
   # Windows
   venv\Scripts\activate
   
   # macOS/Linux
   source venv/bin/activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Run the application:**
   ```bash
   python run.py
   ```

5. **Open in browser:**
   - Main page: http://localhost:5000
   - Student login: http://localhost:5000/auth/student/login
   - Professor login: http://localhost:5000/auth/professor/login

### Test Credentials
| Role | Email | Password |
|------|-------|----------|
| Professor | professor@acem.ac.in | password123 |
| Student | student@acem.ac.in | password123 |

## ✨ Features

### For Professors
- ✅ Create and manage classes
- ✅ Start/stop attendance sessions
- ✅ Generate dynamic QR codes (rotate every 30 seconds)
- ✅ View real-time attendance
- ✅ Enroll students in classes
- ✅ Update attendance status
- ✅ View analytics and reports
- ✅ Complete sessions (auto-mark absent)
- ✅ Cancel sessions

### For Students
- ✅ View enrolled classes
- ✅ Scan QR codes to mark attendance
- ✅ View attendance history
- ✅ Receive notifications
- ✅ Track attendance rate per class
- ✅ Update profile

### Security Features
- 🔐 HMAC-SHA256 signed QR codes
- 🔐 30-second QR expiration
- 🔐 Nonce to prevent replay attacks
- 🔐 Device fingerprinting
- 🔐 IP address logging
- 🔐 Email domain restriction (@acem.ac.in)

## 📁 Project Structure

```
python_attendance_system/
├── app/
│   ├── __init__.py           # Flask app factory
│   ├── models.py             # SQLAlchemy database models
│   ├── routes/
│   │   ├── auth.py           # Authentication routes
│   │   ├── main.py           # Home and common pages
│   │   ├── student.py        # Student dashboard and features
│   │   ├── professor.py      # Professor dashboard and features
│   │   └── api.py            # API endpoints
│   ├── templates/            # Jinja2 HTML templates
│   │   ├── base.html
│   │   ├── auth/
│   │   ├── main/
│   │   ├── student/
│   │   └── professor/
│   └── utils/
│       ├── qr_generator.py   # QR code generation & validation
│       ├── session_generator.py
│       └── seed_data.py      # Initial database data
├── config.py                 # Configuration settings
├── run.py                    # Application entry point
├── requirements.txt          # Python dependencies
└── FEATURES_DOCUMENTATION.md # Complete feature documentation
```

## 🔧 Configuration

Edit `config.py` to modify:

```python
# QR Code settings
QR_EXPIRY_SECONDS = 30  # QR codes expire in 30 seconds
QR_ROTATION_INTERVAL = 30

# Email domain restriction
ALLOWED_EMAIL_DOMAIN = '@acem.ac.in'

# Attendance settings
LATE_THRESHOLD_MINUTES = 5  # Students marked late after 5 minutes
```

## 🎯 How It Works

### QR Code Attendance Flow:

1. **Professor starts session** → Dynamic QR code generated
2. **QR code displayed** → Rotates every 30 seconds
3. **Student scans QR** → Using phone camera or web scanner
4. **System validates** → Checks signature, expiration, enrollment
5. **Attendance recorded** → Present or Late (based on time)
6. **Professor ends session** → Absent students auto-marked

### QR Code Security:

```json
{
  "sessionId": "uuid-string",
  "timestamp": 1736697600000,
  "nonce": "random-16-byte-hex",
  "signature": "hmac-sha256-hex",
  "expiresAt": "2026-01-12T12:00:30Z"
}
```

## 📱 Supported Browsers

- Chrome (recommended)
- Firefox
- Safari
- Edge

Camera access required for QR scanning.

## 🤝 API Endpoints

### Authentication
- `POST /auth/login` - General login
- `POST /auth/student/register` - Student registration
- `POST /auth/professor/register` - Professor registration
- `GET /auth/logout` - Logout

### Student
- `GET /student/dashboard` - Dashboard
- `GET /student/classes` - Class list
- `GET /student/scan` - QR scanner
- `POST /student/scan/process` - Process scan
- `GET /student/attendance` - Attendance history

### Professor
- `GET /professor/dashboard` - Dashboard
- `GET /professor/classes` - Class list
- `POST /professor/classes/create` - Create class
- `GET /professor/sessions` - All sessions
- `POST /professor/sessions/<id>/activate` - Start session
- `POST /professor/sessions/<id>/complete` - End session

### API
- `GET /api/health` - System health check
- `GET /api/sessions/<id>/qr` - Get QR code
- `GET /api/sessions/<id>/attendance` - Get attendance

## 📄 Documentation

For complete feature documentation, see [FEATURES_DOCUMENTATION.md](./FEATURES_DOCUMENTATION.md)

## 🛠️ Development

### Database
SQLite database stored in `instance/attendance.db`

### Real-time Updates
Flask-SocketIO for WebSocket connections

### Templates
Jinja2 templates with Tailwind CSS styling

## 📝 License

This project is for educational purposes.

---

## 🚀 Future Advancements & Roadmap

This section outlines every meaningful advancement that can be made to this project, organized by category. Each improvement is described in detail so any developer can pick it up and build on the existing foundation.

---

### 1. 🗄️ Database & Storage Upgrades

#### 1.1 Migrate from SQLite to PostgreSQL / MySQL
**Current state:** The system uses SQLite (`instance/attendance.db`) which stores data in a single file. SQLite is not designed for concurrent writes, multi-server deployments, or large datasets.

**Advancement:** Replace SQLite with a production-grade relational database (PostgreSQL recommended). The `config.py` already reads `DATABASE_URL` from an environment variable, so only the environment variable and the production server setup need to change. Benefits include:
- Concurrent read/write support for hundreds of simultaneous users.
- Full-text search for student names, class codes, etc.
- Row-level locking so attendance records from multiple students don't clash.
- Native JSON column type for fields like `days_of_week`.

**Steps:**
1. Set `DATABASE_URL=postgresql://user:pass@host/dbname` in `.env`.
2. Run `flask db upgrade` (after adding Flask-Migrate).
3. Update `requirements.txt` to include `psycopg2-binary`.

---

#### 1.2 Replace In-Memory QR Token Store with Redis
**Current state:** `QRCodeGenerator._active_tokens` is a plain Python dictionary stored in the web-server process memory. This means:
- All tokens are lost on server restart.
- Multiple worker processes (Gunicorn with 4 workers) each have separate dictionaries — tokens generated by worker A cannot be validated by worker B.

**Advancement:** Store tokens in Redis with a TTL (time-to-live) equal to `QR_EXPIRY_SECONDS + QR_GRACE_PERIOD_SECONDS`. Redis is already a standard dependency for Flask-SocketIO in production.

**Steps:**
1. Add `redis` and `flask-caching` to `requirements.txt`.
2. In `qr_generator.py`, replace `cls._active_tokens[token] = {...}` with `redis_client.setex(token, ttl, json.dumps(data))`.
3. Replace `cls._active_tokens[token]` lookups with `redis_client.get(token)`.
4. Remove `_cleanup_expired_tokens()` — Redis handles TTL automatically.

---

#### 1.3 Add Database Migration Support (Flask-Migrate / Alembic)
**Current state:** The database schema is created with `db.create_all()` at startup. Any schema change (adding a column, renaming a table) requires manually dropping and recreating the database, losing all data.

**Advancement:** Integrate Flask-Migrate (which wraps Alembic) so every schema change is versioned:
- `flask db init` — create migration folder.
- `flask db migrate -m "add face_id column to students"` — auto-generate migration script.
- `flask db upgrade` — apply migration without data loss.

This is essential for any production deployment.

---

### 2. 🔐 Security Enhancements

#### 2.1 Strengthen QR Code Signing (Full HMAC-SHA256)
**Current state:** The simplified token-based system stores tokens in memory and validates by checking if the token exists in `_active_tokens`. The legacy HMAC path exists but is not the primary path.

**Advancement:** Make HMAC-SHA256 signing the default for every QR code (the `_validate_legacy_qr` path already has this code). This means the server does not need to store tokens at all — validity is proven mathematically. Include:
- `sessionId`, `timestamp`, `nonce` (random 16-byte hex) in the signed payload.
- Store used nonces in Redis to prevent replay attacks — once a nonce is used, reject it even if the QR has not yet expired.

---

#### 2.2 Multi-Factor Authentication (MFA / TOTP)
**Current state:** Login is password-only.

**Advancement:** Add optional TOTP (Google Authenticator / Authy) for professor accounts, since they can start sessions and mark students absent. Use `pyotp` library:
- Professor enables MFA from their profile page.
- On login, after password check, show a "Enter 6-digit code" form.
- Validate TOTP and only then set the Flask-Login session.

---

#### 2.3 Rate Limiting & Brute-Force Protection
**Current state:** There is no rate limiting on login endpoints or QR scan endpoints.

**Advancement:** Use `Flask-Limiter` to add per-IP rate limits:
- Login: maximum 10 attempts per minute per IP.
- QR scan: maximum 5 scans per minute per student session (prevent automated scan bots).
- Registration: maximum 3 registrations per hour per IP.

---

#### 2.4 HTTPS Enforcement & Secure Cookies
**Current state:** `SESSION_COOKIE_SECURE = False` in `config.py` — cookies are transmitted over plain HTTP.

**Advancement:** In `ProductionConfig`:
- Set `SESSION_COOKIE_SECURE = True` and `REMEMBER_COOKIE_SECURE = True`.
- Set `SESSION_COOKIE_SAMESITE = 'Strict'`.
- Add HTTP-to-HTTPS redirect middleware.
- Document how to obtain a Let's Encrypt TLS certificate.

---

#### 2.5 Content Security Policy & Security Headers
**Advancement:** Add `Flask-Talisman` to set standard security headers on every response:
- `Strict-Transport-Security` (HSTS).
- `Content-Security-Policy` to prevent XSS.
- `X-Frame-Options: DENY` to prevent clickjacking.
- `X-Content-Type-Options: nosniff`.

---

### 3. 📱 Mobile Application

#### 3.1 React Native / Flutter Mobile App for Students
**Current state:** Students access the system through a mobile web browser. Camera access depends on browser permissions, and there is no push notification capability.

**Advancement:** Build a dedicated mobile app that:
- Opens the camera natively for faster, more reliable QR scanning (using `expo-barcode-scanner` or `flutter_barcode_scanner`).
- Receives push notifications (Firebase Cloud Messaging) when a professor starts a session.
- Shows attendance history with charts in a native UI.
- Works offline and syncs when back online.
- Stores the student's JWT token securely in the device keychain.

---

#### 3.2 Progressive Web App (PWA) Enhancements
**Current state:** The web app has no service worker or offline capability.

**Advancement:** Add a service worker and Web App Manifest so the existing web app can be "installed" on Android/iOS home screens and:
- Cache static assets for offline viewing of past attendance records.
- Show a "You are offline — QR scanning is unavailable" banner when the network is lost.
- Receive Web Push notifications via the Push API.

---

### 4. 📊 Analytics & Reporting Improvements

#### 4.1 Advanced Analytics Dashboard
**Current state:** The professor dashboard shows basic attendance counts. The `reports.py` util generates Excel exports.

**Advancement:** Add a rich analytics view using Chart.js or Recharts:
- **Attendance trend chart** — line graph showing attendance rate per session over the semester.
- **Student comparison chart** — bar chart comparing individual student rates.
- **Heatmap** — which days / times have the lowest attendance.
- **Prediction** — flag students who are at risk of falling below the 75% threshold (configurable via `LOW_ATTENDANCE_THRESHOLD` in `config.py`) before it happens.

---

#### 4.2 Automated Low-Attendance Email Alerts
**Current state:** `email_service.py` and `ENABLE_EMAIL_NOTIFICATIONS` exist in config but notifications are disabled by default (`False`).

**Advancement:** Enable and extend the email service:
- Send an automatic email to a student when their attendance drops below `LOW_ATTENDANCE_THRESHOLD`.
- Send a weekly summary email to professors listing students at risk.
- Send a reminder email to students 30 minutes before each class.
- Use an async task queue (Celery + Redis) so emails are sent in the background and do not block web requests.

---

#### 4.3 PDF Report Generation
**Current state:** Only Excel (`.xlsx`) export via `openpyxl` is implemented.

**Advancement:** Add PDF export using `reportlab` or `WeasyPrint`:
- Per-student report card showing all classes, attendance percentage, and a summary.
- Per-class report showing all students and their status for every session.
- Institution-stamped letterhead style for official use.

---

#### 4.4 Department-Level Admin Dashboard
**Current state:** There is an `admin` role in the `User` model but no admin-specific routes or dashboard.

**Advancement:** Implement the admin dashboard that:
- Shows attendance statistics for all departments.
- Can activate/deactivate user accounts.
- Can create/update/delete departments, courses, academic periods, and classroom locations without touching the database directly.
- Shows system health (active WebSocket connections, Redis memory usage, database size).

---

### 5. 📡 Real-Time & WebSocket Enhancements

#### 5.1 Live Attendance Feed on Professor Dashboard
**Current state:** Flask-SocketIO is installed and `events.py` exists, providing the WebSocket infrastructure.

**Advancement:** Use SocketIO rooms (one room per session ID) to:
- Push a real-time event to the professor's browser every time a student scans in — the student's name and status appear in a live feed without page refresh.
- Update the attendance count badge in real time.
- Animate newly scanned students appearing in the list.

---

#### 5.2 Student Scan Confirmation & Live QR Timer
**Current state:** The QR code is displayed as a static image; it rotates on a timer but there is no visual countdown.

**Advancement:**
- Add a circular countdown timer around the QR image showing seconds remaining before rotation.
- When a student scans successfully, the professor's screen immediately highlights that student as "Present" via SocketIO.
- Student receives an in-app success notification with a green checkmark instead of a page reload.

---

#### 5.3 Session State Sync Across Multiple Tabs
**Advancement:** Use SocketIO to broadcast session state changes (started, paused, ended) to all open browser tabs. If a professor ends a session from their phone, their laptop tab updates automatically without requiring a refresh.

---

### 6. 📍 Geolocation Enforcement

#### 6.1 Enforce Location Verification on Scan
**Current state:** The `AttendanceRecord` model has `latitude`, `longitude`, `location_verified`, and `location_distance` columns, and `ENABLE_GEOLOCATION` / `LOCATION_VERIFICATION_RADIUS` are in `config.py`. However, enforcement during scan is optional.

**Advancement:** Make geolocation verification mandatory (configurable per class instance):
- When a student scans, the browser requests their GPS coordinates.
- The server computes the Haversine distance between the student's location and the classroom's registered coordinates.
- If the distance exceeds `CLASSROOM_RADIUS` meters, reject the scan with an "Outside classroom range" error.
- Store the distance in `location_distance` for audit purposes.

---

#### 6.2 Per-Classroom GPS Coordinates Management
**Current state:** All classroom locations are derived from a single `CLASSROOM_LAT`/`CLASSROOM_LNG` environment variable.

**Advancement:** Add a `ClassroomLocation` model and an admin UI to:
- Register each room by name with its GPS coordinates and allowed radius.
- Auto-link a `ClassInstance` to its room's coordinates when the `room_location` field is set.
- Show a map (using Leaflet.js) in the admin panel where each classroom pin can be dragged to its real position.

---

### 7. 🎓 Academic Feature Enhancements

#### 7.1 Student Self-Enrollment with Class Codes
**Current state:** Students are enrolled manually by professors (`enrollment_method = 'manual'`). The `Enrollment` model supports `'self_enrollment'` but there is no UI for it.

**Advancement:** Allow students to join a class themselves:
- Professor shares a short join code (e.g., `CSC105-ABC123`, the `class_code` field already exists).
- Student enters the code on their dashboard and is immediately enrolled (if enrollment deadline has not passed and the class is not full).
- Professor sees a real-time notification when a student self-enrolls.

---

#### 7.2 Configurable Attendance Policies per Class
**Current state:** `LATE_THRESHOLD_MINUTES` and `GRACE_PERIOD_MINUTES` are global settings in `config.py`, applying identically to every class.

**Advancement:** Add per-`ClassInstance` attendance policy fields:
- `late_threshold_minutes` — override the global setting.
- `min_attendance_percent` — the minimum percentage for the class (used in analytics and alerts).
- `allow_excused_absences` — whether professors can mark students as "Excused" rather than "Absent".
- `lock_attendance_after_minutes` — automatically close attendance after N minutes so late students cannot scan.

---

#### 7.3 Excuse / Leave Request Workflow
**Current state:** A professor can manually change an attendance record's status, but there is no student-initiated workflow.

**Advancement:**
- Student submits a leave request with a reason and optional document upload (medical certificate, etc.).
- Professor approves/rejects from their dashboard.
- On approval, the attendance record is updated to `'excused'` automatically.
- Both parties receive email/in-app notifications at each step.

---

#### 7.4 Bulk Attendance Import via CSV / Excel
**Current state:** `bulk_enrollment.py` handles bulk student enrollment from a file. A similar pattern can be applied to attendance.

**Advancement:** Allow professors to upload a spreadsheet to mark attendance for a session retroactively — useful for field trips or external events. The importer should:
- Validate that every student ID in the file is enrolled in the class.
- Report errors row by row without failing the entire upload.
- Preview the import before committing.

---

### 8. 🤖 AI / Machine Learning Integrations

#### 8.1 Attendance Pattern Anomaly Detection
**Advancement:** Train a simple model (or use rule-based heuristics) to flag unusual patterns:
- A student who always attended suddenly stops — trigger an automated welfare check notification to the advisor.
- A cluster of students absent on the same day but not in other classes — possibly a legitimate event (holiday, trip) — flag for professor review.

---

#### 8.2 Face Recognition as Secondary Verification
**Advancement:** Optionally require face verification after QR scan to prevent proxy attendance (one student scanning for another):
- On first enrollment, student uploads a photo or takes one via webcam.
- At scan time, the camera captures the student's face and compares it against the stored embedding (using `face_recognition` or `DeepFace` library).
- If match confidence is below threshold, mark the scan as "Pending Verification" and notify the professor.

---

#### 8.3 Predictive Attendance Scheduling
**Advancement:** Use historical attendance data to predict which students are likely to miss a given class, allowing proactive outreach before the session rather than after.

---

### 9. 🌐 Multi-Tenancy & Scalability

#### 9.1 Multi-Institution Support
**Current state:** The `ALLOWED_EMAIL_DOMAIN = '@acem.ac.in'` setting is hardcoded in `config.py` for a single institution.

**Advancement:** Support multiple institutions on the same deployment:
- Each institution has its own allowed email domain(s), branding (logo, colours), and admin.
- A super-admin can onboard new institutions without code changes.
- Data is isolated per institution (either by schema or by a `institution_id` foreign key on every table).

---

#### 9.2 Horizontal Scaling with Docker & Kubernetes
**Current state:** The app runs as a single process with `eventlet`. Scaling requires manual configuration.

**Advancement:**
- Add a `Dockerfile` and `docker-compose.yml` for local development (app + PostgreSQL + Redis).
- Add Kubernetes manifests (`Deployment`, `Service`, `Ingress`) for production.
- Use Redis as the Flask-SocketIO message queue (`message_queue='redis://...'`) so SocketIO events propagate across multiple pods.
- Add a health check endpoint (`/api/health` already exists) and liveness/readiness probes.

---

#### 9.3 API Versioning & REST API for Third-Party Integrations
**Current state:** API endpoints are under `/api/` with no versioning.

**Advancement:**
- Version the API: `/api/v1/sessions`, `/api/v2/sessions`, etc.
- Add full OpenAPI (Swagger) documentation using `flask-restx` or `flasgger`.
- Issue API keys so the institution's existing student information system (SIS) can push enrollment updates automatically rather than manually importing CSVs.
- Add a webhook system so external systems are notified when attendance is recorded.

---

### 10. 🧪 Testing & CI/CD

#### 10.1 Automated Test Suite
**Current state:** No automated tests exist in the repository.

**Advancement:** Add comprehensive tests with `pytest` and `pytest-flask`:
- **Unit tests** — QR generation, HMAC validation, Haversine distance calculation, attendance status logic.
- **Integration tests** — Full scan flow: professor starts session → QR generated → student scans → attendance recorded.
- **Security tests** — Expired QR rejected, replayed nonce rejected, wrong session ID rejected.
- **Load tests** — Use `locust` to simulate 200 students scanning simultaneously and verify no race conditions.

---

#### 10.2 GitHub Actions CI/CD Pipeline
**Advancement:** Add `.github/workflows/ci.yml` to:
- Run `pytest` on every pull request.
- Run `flake8` / `black` for code style.
- Build a Docker image and push to GitHub Container Registry on merge to `main`.
- Automatically deploy to the staging environment and run smoke tests.

---

### 11. 🖥️ UI / UX Improvements

#### 11.1 Dark Mode
**Advancement:** Add a dark mode toggle stored in `localStorage`. Extend the Tailwind CSS configuration with a `dark:` variant applied to all components.

#### 11.2 Accessibility (WCAG 2.1 AA)
**Advancement:** Audit and fix colour contrast ratios, add ARIA labels to all interactive elements, ensure full keyboard navigation, and add screen-reader announcements for live attendance updates.

#### 11.3 Internationalisation (i18n)
**Advancement:** Use Flask-Babel to extract all user-facing strings into `.po` translation files so the application can be deployed in multiple languages without code changes.

#### 11.4 Configurable Branding / White-Label
**Advancement:** Store institution name, logo URL, and primary colour in the database (per institution in the multi-tenancy model). Inject these into the Jinja2 base template so each institution sees their own branding.

---

### Quick Reference: Advancement Priority Matrix

| # | Advancement | Impact | Effort | Priority |
|---|-------------|--------|--------|----------|
| 1.2 | Redis token store | 🔴 High | 🟡 Low | **P0** |
| 1.3 | Flask-Migrate | 🔴 High | 🟡 Low | **P0** |
| 2.3 | Rate limiting | 🔴 High | 🟡 Low | **P0** |
| 10.1 | Test suite | 🔴 High | 🟡 Low | **P0** |
| 1.1 | PostgreSQL | 🔴 High | 🟡 Medium | **P1** |
| 2.4 | HTTPS & secure cookies | 🔴 High | 🟡 Low | **P1** |
| 5.1 | Live attendance feed | 🟡 Medium | 🟡 Medium | **P1** |
| 6.1 | Geolocation enforcement | 🟡 Medium | 🟡 Medium | **P1** |
| 4.2 | Email alerts | 🟡 Medium | 🟡 Low | **P1** |
| 7.1 | Self-enrollment | 🟡 Medium | 🟡 Medium | **P2** |
| 3.1 | Mobile app | 🔴 High | 🔴 High | **P2** |
| 8.2 | Face recognition | 🟡 Medium | 🔴 High | **P3** |
| 9.1 | Multi-tenancy | 🟡 Medium | 🔴 High | **P3** |

---

**Smart Attendance Management System** - Making attendance tracking modern, secure, and efficient.
