# Practical 6 — Dockerize the Student Result Prediction Application

This repository continues the Student Result Prediction ML application and implements Practical 6.

## What this practical adds

- A Dockerfile that packages Python 3.11, the application, trained model, metrics, and dependencies.
- A .dockerignore file that excludes tests, Git metadata, caches, and training-only files from the Docker build context.
- An updated GitHub Actions continuous-delivery workflow that includes both Docker configuration files in the `student-result-ml-release` artifact.

**Important:** As specified in Practical 6, GitHub Actions prepares the Docker-ready release candidate. Docker image building and container/API verification are performed manually on the local system. Automated Docker build and validation in GitHub Actions are introduced in Practical 7.

## Repository files

- `train_model.py` — trains the student result model.
- `quality_gate.py` — requires at least 0.85 model accuracy.
- `test_ml_pipeline.py` — seven ML pipeline tests.
- `app.py` — Flask health and prediction endpoints.
- `test_app.py` — four API tests.
- `Dockerfile` — builds the Docker image.
- `.dockerignore` — excludes unnecessary build-context files.
- `.github/workflows/ml-ci.yml` — trains, tests, and uploads the Docker-ready release candidate.

## GitHub Actions release artifact

The workflow uploads `student-result-ml-release` with:

```text
app.py
student_result_model.pkl
metrics.json
requirements.txt
Dockerfile
.dockerignore
DEPLOYMENT.txt
```

## Manual Docker steps (Windows PowerShell)

1. Download and extract `student-result-ml-release` from the latest successful GitHub Actions run.
2. Open PowerShell in the extracted folder.
3. Confirm Docker Desktop is running.
4. Build the image:

```powershell
docker build -t student-result-ml:1.0 .
```

5. Verify the image:

```powershell
docker images
```

6. Run the container:

```powershell
docker run -d --name student-result-ml-container -p 5000:5000 student-result-ml:1.0
```

7. Check the running container:

```powershell
docker ps
```

8. Test the health endpoint in a browser: http://localhost:5000/

Expected response:

```json
{
  "service": "student-result-prediction",
  "status": "ok"
}
```

9. Test a high-performance student (expected PASS):

```powershell
$body = @{
  attendance = 90
  internal_marks = 85
  assignment_marks = 88
  previous_score = 80
} | ConvertTo-Json

Invoke-RestMethod -Uri http://localhost:5000/predict -Method Post -ContentType "application/json" -Body $body
```

10. Test a low-performance student (expected FAIL):

```powershell
$body = @{
  attendance = 55
  internal_marks = 30
  assignment_marks = 40
  previous_score = 35
} | ConvertTo-Json

Invoke-RestMethod -Uri http://localhost:5000/predict -Method Post -ContentType "application/json" -Body $body
```

11. Inspect logs:

```powershell
docker logs student-result-ml-container
```

12. Stop and remove the container when finished:

```powershell
docker stop student-result-ml-container
docker rm student-result-ml-container
```

## Note

The Docker image and container steps above must be run on a computer with Docker Desktop or Docker Engine installed and running. They cannot be truthfully marked complete until the local Docker commands and API checks have been performed.
