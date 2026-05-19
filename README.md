# Movie Sentiment Analysis

## About This Project

I built this project as my capstone to learn and implement a complete MLOps workflow from scratch. The goal was to go beyond just training a model and actually understand what it takes to build, track, version, deploy, and monitor a machine learning system in production.

The project classifies movie reviews as positive or negative using a machine learning pipeline that I designed end-to-end — from raw data ingestion all the way to a live Flask API running on AWS EKS, monitored with Prometheus and Grafana.

---

## What I Built

I structured the project as a full MLOps pipeline:

Raw Data → Data Ingestion → Preprocessing → Feature Engineering → Model Training → Model Evaluation → Model Registry → Flask API → Docker → ECR → EKS (Kubernetes) → Monitoring (Prometheus + Grafana)

---

## Tech Stack

| Category | Tools I Used |
|---|---|
| Language | Python 3.10 |
| ML and NLP | Scikit-learn, NLTK |
| Experiment Tracking | MLflow, DagsHub |
| Data Versioning | DVC with AWS S3 as remote storage |
| Web Framework | Flask |
| Containerization | Docker |
| CI/CD | GitHub Actions |
| Cloud | AWS (ECR, EKS, S3, IAM) |
| Orchestration | Kubernetes via eksctl and kubectl |
| Monitoring | Prometheus, Grafana |
| Testing | pytest, tox |

---

## Project Structure

```
.github/workflows/      <- CI/CD pipeline I wrote using GitHub Actions
flask_app/              <- Flask web app for serving predictions
notebooks/              <- Jupyter notebooks I used for EDA and experiments
src/
    data/               <- Scripts for data ingestion
    features/           <- Feature engineering scripts
    models/             <- Model training, evaluation, and registration scripts
    visualization/      <- Visualization scripts
tests/                  <- Unit and integration tests
scripts/                <- Helper scripts used in CI
logs/                   <- Pipeline and application logs
reports/                <- Generated reports and figures
mlruns/                 <- MLflow experiment artifacts
dvc.yaml                <- DVC pipeline definition
dvc.lock                <- DVC pipeline lock file
params.yaml             <- Hyperparameters for the pipeline
deployment.yaml         <- Kubernetes deployment manifest
Dockerfile              <- Docker image definition
requirements.txt        <- Python dependencies
Makefile                <- Shortcut commands
```

---

## Pipeline Stages

I defined the full ML pipeline in `dvc.yaml`. Running `dvc repro` executes all stages in order:

1. Data Ingestion — Load and store raw movie review data
2. Data Preprocessing — Clean text, remove noise, handle nulls
3. Feature Engineering — Vectorize text using TF-IDF
4. Model Building — Train the classifier using hyperparameters from `params.yaml`
5. Model Evaluation — Compute and log metrics to MLflow on DagsHub
6. Model Registration — Register the best model to the MLflow Model Registry

---

## How to Run This Project

### Prerequisites

- Python 3.10
- Conda
- Docker Desktop
- AWS CLI configured
- DVC

### Step 1 - Clone the Repository

```bash
git clone https://github.com/Atulyaverma70/Movie-Sentiment-Analysis.git
cd Movie-Sentiment-Analysis
```

### Step 2 - Set Up the Virtual Environment

```bash
conda create -n atlas python=3.10
conda activate atlas
pip install -r requirements.txt
pip install -e .
```

### Step 3 - Set Environment Variables

```bash
export DAGSHUB_TOKEN=your_dagshub_token_here
```

### Step 4 - Run the DVC Pipeline

```bash
dvc repro
```

### Step 5 - Run the Flask App Locally

```bash
cd flask_app
python app.py
```

Then open `http://localhost:5000` in your browser.

---

## Docker

I containerized the Flask app so it can run anywhere without environment issues.

Build the image:

```bash
docker build -t capstone-app:latest .
```

Run the container:

```bash
docker run -p 8888:5000 -e DAGSHUB_TOKEN=your_token capstone-app:latest
```

Then open `http://localhost:8888`.

---

## Cloud Deployment on AWS EKS

I deployed the app to a Kubernetes cluster on AWS EKS. The CI/CD pipeline handles building and pushing the Docker image to ECR automatically on every push to main.

To create the EKS cluster manually:

```bash
eksctl create cluster \
  --name flask-app-cluster \
  --region us-east-1 \
  --nodegroup-name flask-app-nodes \
  --node-type t3.small \
  --nodes 1 --nodes-min 1 --nodes-max 1 \
  --managed
```

Deploy the app:

```bash
kubectl apply -f deployment.yaml
```

Get the external IP:

```bash
kubectl get svc
```

The app is accessible at `<EXTERNAL-IP>:5000`.

---

## CI/CD Pipeline

I set up a GitHub Actions workflow in `.github/workflows/ci.yaml` that runs on every push and does the following:

- Runs all tests
- Builds the Docker image
- Pushes the image to AWS ECR
- Deploys the updated image to EKS

The following secrets need to be added to the GitHub repository settings:

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
AWS_ACCOUNT_ID
ECR_REPOSITORY
DAGSHUB_TOKEN
```

---

## Experiment Tracking

I used MLflow hosted on DagsHub to track every experiment I ran. Each run logs accuracy, precision, recall, and F1-score so I could compare models and pick the best one for registration.

To view the MLflow UI, go to your DagsHub repository and click "Go to MLflow UI".

---

## Monitoring

After deployment, I set up Prometheus to scrape metrics from the Flask app and Grafana to visualize them on a dashboard. Both run on separate Ubuntu EC2 instances (t3.medium).

| Tool | Purpose | Port |
|---|---|---|
| Prometheus | Scrapes metrics from the Flask app | 9090 |
| Grafana | Displays dashboards from Prometheus data | 3000 |

---

## Running Tests

```bash
pytest tests/
```

Or using tox:

```bash
tox
```

---

## AWS Cleanup

When done, I clean up all resources to avoid extra charges:

```bash
kubectl delete deployment flask-app
kubectl delete service flask-app-service
kubectl delete secret capstone-secret
eksctl delete cluster --name flask-app-cluster --region us-east-1
```

Also delete ECR images, S3 data, and verify all CloudFormation stacks are removed from the AWS Console.

---

## Acknowledgements

I used the Cookiecutter Data Science template for the initial project structure. Experiment tracking is powered by DagsHub and MLflow.
