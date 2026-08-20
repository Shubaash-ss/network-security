# Network Security — Phishing URL Detection (MLOps Pipeline)

An end-to-end MLOps pipeline that trains and serves a phishing-detection classification model, built following Krish Naik's Data Science Udemy course and extended with a full CI/CD deployment to AWS.

## 🔗 Live Demo

The API is currently deployed and live:

- **Swagger UI:** [http://13.48.178.11:8080/docs](http://13.48.178.11:8080/docs)
- **Predict endpoint:** `POST /predict` — upload a CSV of URL features, get back predictions as an HTML table
- **Train endpoint:** `GET /train` — triggers the training pipeline

> Note: This is deployed on a personal EC2 instance for demonstration purposes and may be stopped/restarted periodically.

## 🏗️ Architecture

```
MongoDB (data source)
    ↓
Data Ingestion → Data Validation → Data Transformation
    ↓
Model Training (scikit-learn) → MLflow/DagsHub (experiment tracking)
    ↓
FastAPI (serving layer)
    ↓
Docker → Amazon ECR (image registry)
    ↓
GitHub Actions (CI/CD) → Self-hosted runner on EC2
    ↓
Live container on EC2, port 8080
```

## 🛠️ Tech Stack

- **ML:** Python, scikit-learn, Pandas, NumPy
- **Experiment tracking:** MLflow, DagsHub
- **Data storage:** MongoDB (Atlas)
- **API:** FastAPI, Uvicorn
- **Containerization:** Docker
- **Cloud:** AWS (ECR, EC2, S3, IAM)
- **CI/CD:** GitHub Actions, self-hosted runner

## 🚀 Running Locally

```bash
git clone https://github.com/Shubaash-ss/network-security.git
cd network-security
python -m venv venv
venv\Scripts\activate  # Windows
pip install -r requirements.txt
```

Create a `.env` file in the project root with:
```
MONGO_DB_URL=<your MongoDB connection string>
MLFLOW_TRACKING_URI=<your DagsHub MLflow tracking URI>
MLFLOW_TRACKING_USERNAME=<your DagsHub username>
MLFLOW_TRACKING_PASSWORD=<your DagsHub token>
```

Run the app:
```bash
python app.py
```
The API will be available at `http://localhost:8000`.

> **Note:** This repo does not include trained model artifacts. Run `GET /train` after starting the app to generate a model before calling `/predict`.

## ⚙️ CI/CD Pipeline

Every push to `main` triggers a three-stage GitHub Actions workflow:

1. **Continuous Integration** — lint and test (currently placeholder checks)
2. **Continuous Delivery** — builds a Docker image and pushes it to Amazon ECR
3. **Continuous Deployment** — a self-hosted runner on EC2 pulls the latest image, stops the old container, and starts the new one

Required GitHub repository secrets:

| Secret | Purpose |
|---|---|
| `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` / `AWS_REGION` | AWS auth for ECR/EC2 |
| `AWS_ECR_LOG_IN_URI` | ECR registry URI |
| `ECR_REPOSITORY_NAME` | ECR repo name |
| `MONGO_DB_URL` | MongoDB connection string |
| `MLFLOW_TRACKING_URI` / `MLFLOW_TRACKING_USERNAME` / `MLFLOW_TRACKING_PASSWORD` | DagsHub MLflow credentials |

## 🐛 Deployment Notes & Lessons Learned

This project's deployment surfaced (and fixed) several real production issues, documented here as a reference:

- **Missing runtime secrets:** the container needs MongoDB and MLflow/DagsHub credentials passed explicitly via `docker run -e`, not just baked into the image — these aren't picked up from a local `.env` automatically.
- **Self-hosted runner setup:** the GitHub Actions runner must be registered and installed as a persistent service (`sudo ./svc.sh install && sudo ./svc.sh start`) on the EC2 instance, or deployment jobs will queue indefinitely.
- **Disk space on EC2:** the default EBS volume size can fill up quickly from repeated image pulls/builds; resizing the volume and running `docker system prune -af` periodically resolves `No space left on device` errors during `docker pull`.
- **Port mapping:** the container's Uvicorn server binds to port 8000 internally — the Docker port mapping must be `-p 8080:8000`, not `8080:8080`.
- **Security groups:** EC2 security groups block all inbound traffic by default except explicitly opened ports — port 8080 needs an explicit inbound rule for the API to be reachable externally.
- **Missing output directories:** the `/predict` endpoint writes results to a `prediction_output/` directory that isn't created automatically — the app now calls `os.makedirs('prediction_output', exist_ok=True)` before writing.

## 📌 Known Limitations

- Model artifacts are not committed to the repo, so a fresh clone/deploy requires running `/train` before `/predict` will work.
- CI stage currently uses placeholder lint/test steps rather than real automated tests.

## 👤 Author

**Shubaash S S**
- GitHub: [@Shubaash-ss](https://github.com/Shubaash-ss)
- LinkedIn: [shubaash-s-s](https://linkedin.com/in/shubaash-s-s-00796127a)