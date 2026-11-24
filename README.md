 # CapstoneConnect — Full-stack app prototype

A runnable student teammate matcher prototype for the LPU capstone project. The repository contains an Expo + React Native mobile frontend, a Python FastAPI REST API, and a SQLite database that seeds sample data automatically.

## Features implemented

- Student profile cards with tech skills, CGPA, explicit past-project links, GitHub/portfolio links, roles, and contact email.
- Search and filters for student name, skills, and role.
- Project pitch board with project descriptions, skills, team owner contact, and open roles.
- Create/publish a project pitch from the app.
- Request to join a project with a message.
- Requests inbox with sent/incoming request views and accept/decline actions for incoming requests.
- SQLite persistence for new projects and join requests.
- FastAPI interactive docs at `/docs` and a `/api/health` check.

## Stack

- Mobile: Expo SDK 57, React Native 0.86, plain JavaScript (no UI or navigation library required).
- Backend: Python 3.10+ recommended, FastAPI, Uvicorn.
- Database: SQLite (built into Python).

An optional responsive HTML/CSS/JavaScript website is included in `web-prototype/`. It demonstrates student profiles with CGPA, skills, roles, past-project links and contact details; skill/role search; a project pitch board with open roles; and join requests with a message to the project owner. It runs independently with fictional demo data and does not persist requests or projects.

## Quick links

- [Standalone web prototype](web-prototype/index.html)
- [Mobile app source](mobile/)
- [Python backend source](backend/)
- [Backend API health check](http://127.0.0.1:8000/api/health) (available after starting the backend)
- [Interactive API docs](http://127.0.0.1:8000/docs) (available after starting the backend)

## Run the standalone web prototype

The web prototype does not need the Python backend or Node.js. Open `web-prototype/index.html` directly in a browser, or serve it locally from the project folder:

```powershell
cd web-prototype
py -m http.server 8080
```

Then open <http://127.0.0.1:8080>. This frontend-only demo uses sample data; projects and requests are not connected to the full-stack app or saved to the backend.

## 1. Start the Python backend

Open a terminal in `CapstoneConnect/backend`.

### Windows PowerShell

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
python run.py
```

If PowerShell blocks activation, run the venv Python directly instead:

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe run.py
```

### macOS/Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
python run.py
```

When the server starts, test these in a browser on your computer:

- Health: <http://127.0.0.1:8000/api/health>
- Interactive API docs: <http://127.0.0.1:8000/docs>
- Students: <http://127.0.0.1:8000/api/students>
- Projects: <http://127.0.0.1:8000/api/projects>

The database file `backend/capstoneconnect.db` is created and seeded on first start. It is ignored by Git. Delete it while the server is stopped if you want to reset the local prototype data.

## 2. Start the Expo mobile app

Use Node.js 22.13+ for the Expo SDK 57 toolchain. SDK 57 is selected for the current Expo Go workflow on physical Android devices. Open a second terminal in `CapstoneConnect/mobile`:

```bash
npm install
```

Create a local `.env` file by copying `.env.example`. The correct URL depends on where the app runs:

### Android Emulator

Keep this in `mobile/.env`:

```dotenv
EXPO_PUBLIC_API_URL=http://10.0.2.2:8000/api
```

`10.0.2.2` is the Android emulator's special route to the host computer. Do not use it when running Expo Go on a physical phone.

### Physical Android phone using Expo Go

1. Connect your laptop and phone to the same Wi-Fi network.
2. Find your laptop's private IPv4 address (Windows: run `ipconfig`; use the IPv4 address for your active Wi-Fi adapter).
3. Update `mobile/.env` like this, replacing the example IP:

   ```dotenv
   EXPO_PUBLIC_API_URL=http://192.168.1.25:8000/api
   ```

4. Keep the backend running. If Windows Firewall asks, allow Python on your **private** network. Do not expose the development server to the public internet.
5. Run Expo:

   ```bash
   npx expo start
   ```

6. Open Expo Go on the phone and scan the QR code. If the API cannot connect, check the URL, ensure both devices share Wi-Fi, and use the app's Retry button. Restart Expo after changing `.env`.

### Android emulator or USB device

- Emulator: `npm run android` (with Android Studio and an emulator installed).
- Physical phone: scan the Expo QR code in Expo Go, or enable USB debugging and use a suitable Android development workflow.

## 3. Build an Android APK

For a development APK / installable test build, use Expo Application Services (EAS). From `mobile`:

```bash
npx eas-cli@latest login
npx eas-cli@latest build:configure
```

The project already includes `mobile/eas.json` with a `preview` profile configured to output an APK. After logging in and configuring the EAS project, run:

```bash
npx eas-cli@latest build --platform android --profile preview
```

For a cloud build that can call your hosted backend, configure `EXPO_PUBLIC_API_URL` as an EAS environment variable (or set it in a local `mobile/.env` before running a build from your machine). EAS builds the APK in the cloud and provides a download page when complete. Configure `EXPO_PUBLIC_API_URL` to point to a deployed, HTTPS backend before sharing an APK with other people. A URL like `10.0.2.2` or a laptop's local Wi-Fi IP will only work in the appropriate local setup.

## Demo behaviour and known limits

- This is a full-stack **prototype**, not a production-ready service. It deliberately signs in as demo student ID `1` (Aarav Sharma); no account registration, password, authorization, or LPU login is implemented.
- Sample names, emails, CGPAs, profiles, and project URLs are fictional demo content. Replace them with consented data before a live launch.
- Contact email is shown to demonstrate the requested contact-detail workflow. Before a real release, add login, role-based authorization, and privacy/consent settings.
- Accept/decline updates are restricted by the API to the project owner ID passed by this prototype; without proper authentication this is **not secure authentication**.
- The app sends requests to the Python backend; request creation does not actually send an email, notification, or chat message.
- The IoT aspect is represented in skills/project ideas (for example, sensor dashboards). No physical ESP32/Arduino device is connected yet; that can be added later through an MQTT broker or a device-specific API.
- CORS is open for local development only. Tighten it before deployment, use HTTPS, and add authentication, authorization, input limits, abuse protection, and backups before production use.

## API endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/api/health` | Health check |
| `GET` | `/api/students?q=&skill=&role=` | Search/filter student profiles |
| `GET` | `/api/students/{id}` | One student profile |
| `GET` | `/api/projects?skill=` | Project pitch board/filter |
| `GET` | `/api/projects/{id}` | Project details and owner contact |
| `POST` | `/api/projects` | Publish a project idea |
| `GET` | `/api/requests?user_id=1` | Requests sent/received by the demo user |
| `POST` | `/api/requests` | Request to join a team |
| `PATCH` | `/api/requests/{id}?user_id=1` | Accept or decline an incoming request |

The user ID query parameter is only a prototype shortcut. Never use it as a substitute for authentication in a real app.
