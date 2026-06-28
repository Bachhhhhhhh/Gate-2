# Inspectra MVP - Setup, Environment, and Sample Queries

Inspectra is an MVP for automated surface-defect inspection. It provides a
deployed web application where users can log in, upload inspection images, run a
VLM defect check, and view user-owned reports.

Production URLs:

- Web UI: https://c2-app-089-production.up.railway.app
- API: https://inspectra-api-production.up.railway.app
- API health: https://inspectra-api-production.up.railway.app/api/health

Demo accounts:

- Admin: `admin@inspectra.ai` / `admin123`
- Inspector: `inspector@inspectra.ai` / `inspect123`
- Operator: `operator@inspectra.ai` / `operator123`

## Product Scope

- Surface inspection images are processed by a VLM.
- Reports are persisted and associated with the logged-in user.
- Admin users can inspect all reports.
- Normal users only see their own uploaded reports.
- Camera input is used for inspection capture only, not robot navigation.
- Robot waypoints must come from deterministic CAD/floorplan planning and a
  safety gate.

## Local Setup

Prerequisites:

- Python 3.10+
- Node.js 20+
- npm
- Optional: Docker Desktop

Run the app locally:

```powershell
cd "<repo>\deploy_app"
npm install
python -m pip install -r requirements.txt
npm run dev
```

Local URLs:

- Web: http://localhost:3000
- API: http://localhost:8000
- API docs: http://localhost:8000/docs

Docker alternative:

```powershell
cd "<repo>\deploy_app"
docker compose up --build
```

## Environment Variables

Backend variables:

```text
INSPECTRA_SECRET_KEY=replace-with-a-long-random-secret
INSPECTRA_CORS_ORIGINS=http://localhost:3000,https://c2-app-089-production.up.railway.app
INSPECTRA_DATABASE_PATH=backend/data/inspectra.db
INSPECTRA_UPLOAD_ROOT=backend/uploads
NOVITA_API_KEY=<your-novita-api-key>
NOVITA_BASE_URL=https://api.novita.ai/openai/v1
NOVITA_MODEL=qwen/qwen3-vl-235b-a22b-instruct
```

Railway backend variables:

```text
INSPECTRA_SECRET_KEY=<random-long-secret>
INSPECTRA_CORS_ORIGINS=https://c2-app-089-production.up.railway.app,http://localhost:3000
INSPECTRA_DATABASE_PATH=/data/inspectra.db
INSPECTRA_UPLOAD_ROOT=/data/uploads
NOVITA_API_KEY=<your-novita-api-key>
NOVITA_BASE_URL=https://api.novita.ai/openai/v1
NOVITA_MODEL=qwen/qwen3-vl-235b-a22b-instruct
```

Frontend variable:

```text
NEXT_PUBLIC_API_URL=http://localhost:8000/api
```

Railway frontend variable:

```text
NEXT_PUBLIC_API_URL=https://inspectra-api-production.up.railway.app/api
```

Robot upper-planner variables are optional and must not be treated as direct
robot control:

```text
QWEN_PLANNER_MODEL=Qwen/Qwen3.6-35B-A3B
```

## Sample API Queries

Health check:

```powershell
Invoke-RestMethod https://inspectra-api-production.up.railway.app/api/health
```

Login:

```powershell
$login = Invoke-RestMethod `
  -Method Post `
  -Uri https://inspectra-api-production.up.railway.app/api/auth/login `
  -ContentType "application/json" `
  -Body '{"email":"inspector@inspectra.ai","password":"inspect123"}'

$token = $login.access_token
```

List reports for the logged-in user:

```powershell
Invoke-RestMethod `
  -Uri https://inspectra-api-production.up.railway.app/api/reports `
  -Headers @{ Authorization = "Bearer $token" }
```

Upload a local image, run VLM, and save a user-owned report:

```powershell
$imagePath = "<path-to-surface-image>.jpg"

curl.exe -X POST "https://inspectra-api-production.up.railway.app/api/vlm-inspections" `
  -H "Authorization: Bearer $token" `
  -F "product=Manual surface sample" `
  -F "robot=Web upload" `
  -F "standard=VLM Surface Inspection PoC" `
  -F "detail=low" `
  -F "image=@$imagePath;type=image/jpeg"
```

Run ESP32-CAM capture, local VLM inspection, and upload report:

```powershell
cd "<repo>\surface_inspection_product\ros_robot_control"
$env:PYTHONPATH = ".\ros_ws\src\surface_inspection_robot"

python -m surface_inspection_robot.cli esp32-inspect `
  --camera-url http://<ESP32_CAMERA_IP> `
  --capture-dir .\captures `
  --runs-dir ..\vlm_surface_inspection\runs `
  --provider novita `
  --detail low `
  --grid 2 `
  --timeout 30 `
  --website-url https://inspectra-api-production.up.railway.app `
  --website-email inspector@inspectra.ai `
  --website-password inspect123 `
  --product "ESP32-CAM wall demo"
```

## Expected User Flow

1. Open the deployed web UI.
2. Log in with the inspector account.
3. Go to `Tao luot kiem tra`.
4. Upload a surface image.
5. Run VLM inspection.
6. Open the generated report detail page.
7. Confirm that the report belongs to the logged-in user and includes source
   image, segmentation/overlay artifact, result JSON, defect count, confidence,
   and reviewer note.
