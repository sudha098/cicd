# End-to-End CI/CD: Jenkins Shared Library → Build Image → Push to GAR → Deploy to GKE via Spinnaker

This repository demonstrates **enterprise-grade CI/CD** using:

* Jenkins (Shared Library only, no Groovy in Jenkinsfile)
* Docker image build
* Google Artifact Registry (GAR)
* Spinnaker for Kubernetes deployment

---

## 1. Repository Structure

```
jenkins-shared-library/
├── vars/
│   └── cloudCICD.groovy
├── src/
│   └── com/company/utils/
│       └── docker.groovy
└── resources/

application-repo/
├── Dockerfile
├── Jenkinsfile
├── spinnaker/
│   └── pipeline.json
├── k8s/
│   └── deployment.yaml
└── app/
    └── main.py
```

---

## 2. Jenkins Shared Library

### `vars/cloudCICD.groovy`

```groovy
def call(Map config) {
  pipeline {
    agent any

    environment {
      IMAGE_NAME = config.imageName
      GAR_REPO   = config.garRepo
      PROJECT_ID = config.projectId
      REGION     = config.region
      TAG        = "${env.BUILD_NUMBER}"
    }

    stages {
      stage('Checkout') {
        steps { checkout scm }
      }

      stage('Build Image') {
        steps {
          sh "docker build -t ${IMAGE_NAME}:${TAG} ."
        }
      }

      stage('Push to GAR') {
        steps {
          withCredentials([file(credentialsId: config.gcpCreds, variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
            sh '''
              gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
              gcloud auth configure-docker ${REGION}-docker.pkg.dev
              docker tag ${IMAGE_NAME}:${TAG} ${REGION}-docker.pkg.dev/${PROJECT_ID}/${GAR_REPO}/${IMAGE_NAME}:${TAG}
              docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${GAR_REPO}/${IMAGE_NAME}:${TAG}
            '''
          }
        }
      }

      stage('Trigger Spinnaker') {
        steps {
          sh "curl -X POST ${config.spinnakerWebhook}"
        }
      }
    }
  }
}
```

---

## 3. Jenkinsfile (Library Only)

```groovy
@Library('company-shared-lib') _

cloudCICD(
  imageName: 'sample-app',
  garRepo: 'containers',
  projectId: 'my-gcp-project',
  region: 'asia-south1',
  gcpCreds: 'gcp-sa',
  spinnakerWebhook: 'http://spinnaker/api/webhooks/deploy'
)
```

---

## 4. Dockerfile

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY app/ .
RUN pip install flask
CMD ["python", "main.py"]
```

---

## 5. Kubernetes Deployment (Spinnaker Managed)

### `k8s/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: app
        image: asia-south1-docker.pkg.dev/my-gcp-project/containers/sample-app
        ports:
        - containerPort: 5000
```

---

## 6. Spinnaker Pipeline

### `spinnaker/pipeline.json`

```json
{
  "application": "sample-app",
  "name": "deploy-to-gke",
  "stages": [
    {
      "type": "deployManifest",
      "account": "gke-account",
      "cloudProvider": "kubernetes",
      "manifests": [
        "k8s/deployment.yaml"
      ]
    }
  ]
}
```

---

## 7. Interview Explanation (Say This)

> "Jenkins handles CI — build and push to GAR. Spinnaker handles CD with declarative Kubernetes deployments and rollback support. Jenkinsfiles stay minimal using shared libraries for standardization."

---

## 8. Why This Is Enterprise Grade

* No Groovy in Jenkinsfile
* Secure credentials
* Immutable Docker images
* Separation of CI and CD
* Spinnaker handles rollout & rollback

---

## 9. Possible Extensions

* Canary deployment in Spinnaker
* Artifact-based triggers
* Multi-environment pipelines
* Helm manifests

---

This setup is **bank / oil & gas / FAANG level** CI/CD.
