# Inspectra MVP

Inspectra is an MVP for surface-defect inspection with a robot/camera capture
flow, VLM defect recognition, and user-owned inspection reports.

## Gate 2 Deliverables

- MVP demo video evidence: `MVP_DEMO.md`
- Architecture diagram and data flow: `ARCHITECTURE_DIAGRAM.md`
- Repository PR evidence: `PR_EVIDENCE.md`
- Setup instructions, environment variables, and sample queries: this file
- Evaluation evidence with at least 5 manual test cases: `EVAL_EVIDENCES.md`

## Product Scope

- Users can log in and manage inspection reports.
- A user can upload a surface image from the web app.
- The robot/camera flow can capture an image, run VLM inspection, and upload
  the result into the same report system.
- Reports are associated with the authenticated user.
- The VLM detects and describes visible surface defects such as cracks and
  peeling/spalling.
- Camera images are used for inspection capture only, not for robot navigation.
- Robot waypoints must come from deterministic CAD/floorplan planning and a
  safety gate.

## Local Setup

Prerequisites:

- Python 3.10+
- Node.js 20+
- npm
- Optional: Docker Desktop

Backend:

```powershell
cd "<project-repo>\deploy_app\backend"
python -m pip install -r requirements.txt
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Frontend:

```powershell
cd "<project-repo>\deploy_app\frontend"
npm install
npm run dev
```

Local URLs:

- Web: `http://localhost:3000`
- API: `http://localhost:8000`
- API docs: `http://localhost:8000/docs`

Docker alternative:

```powershell
cd "<project-repo>\deploy_app"
docker compose up --build
```

## Environment Variables

Backend variables:

```text
INSPECTRA_SECRET_KEY=replace-with-a-long-random-secret
INSPECTRA_CORS_ORIGINS=http://localhost:3000
INSPECTRA_DATABASE_PATH=backend/data/inspectra.db
INSPECTRA_UPLOAD_ROOT=backend/uploads
NOVITA_API_KEY=<your-novita-api-key>
NOVITA_BASE_URL=https://api.novita.ai/openai/v1
NOVITA_MODEL=qwen/qwen3-vl-235b-a22b-instruct
```

Frontend variable:

```text
NEXT_PUBLIC_API_URL=http://localhost:8000/api
```

Robot / local VLM variables:

```text
NOVITA_API_KEY=<your-novita-api-key>
NOVITA_BASE_URL=https://api.novita.ai/openai/v1
NOVITA_MODEL=qwen/qwen3-vl-235b-a22b-instruct
```

Optional upper-planner variable:

```text
QWEN_PLANNER_MODEL=Qwen/Qwen3.6-35B-A3B
```

The upper planner is only allowed to suggest high-level scan order or planner
parameters. It must not send direct motor commands or executable waypoints.

## Demo Accounts

Seed/demo accounts used by the MVP:

```text
admin@inspectra.ai / admin123
inspector@inspectra.ai / inspect123
operator@inspectra.ai / operator123
```

## Sample API Queries

Set the API base URL:

```powershell
$API_BASE_URL = "http://localhost:8000/api"
```

Health check:

```powershell
Invoke-RestMethod "$API_BASE_URL/health"
```

Login:

```powershell
$login = Invoke-RestMethod `
  -Method Post `
  -Uri "$API_BASE_URL/auth/login" `
  -ContentType "application/json" `
  -Body '{"email":"inspector@inspectra.ai","password":"inspect123"}'

$token = $login.access_token
```

List reports for the logged-in user:

```powershell
Invoke-RestMethod `
  -Uri "$API_BASE_URL/reports" `
  -Headers @{ Authorization = "Bearer $token" }
```

Upload a local image, run VLM, and save a user-owned report:

```powershell
$imagePath = "<path-to-surface-image>.jpg"

curl.exe -X POST "$API_BASE_URL/vlm-inspections" `
  -H "Authorization: Bearer $token" `
  -F "product=Manual surface sample" `
  -F "robot=Web upload" `
  -F "standard=VLM Surface Inspection PoC" `
  -F "detail=low" `
  -F "image=@$imagePath;type=image/jpeg"
```

Run ESP32-CAM capture, local VLM inspection, and upload the result to the web
backend:

```powershell
cd "<project-repo>\surface_inspection_product\ros_robot_control"
$env:PYTHONPATH = ".\ros_ws\src\surface_inspection_robot"

python -m surface_inspection_robot.cli esp32-inspect `
  --camera-url http://<ESP32_CAMERA_IP> `
  --capture-dir .\captures `
  --runs-dir ..\vlm_surface_inspection\runs `
  --provider novita `
  --detail low `
  --grid 2 `
  --timeout 30 `
  --website-url <BACKEND_API_ORIGIN> `
  --website-email inspector@inspectra.ai `
  --website-password inspect123 `
  --product "ESP32-CAM wall demo" `
  --robot "ESP32-CAM"
```

## Expected User Flow

1. Log in with the inspector account.
2. Create a new inspection.
3. Upload a surface image or send a robot/camera capture result.
4. Run VLM inspection.
5. Open the generated report detail page.
6. Review the source image, segmentation/overlay artifact, result JSON, defect
   count, confidence, and reviewer note.
7. Confirm the report after human review.
