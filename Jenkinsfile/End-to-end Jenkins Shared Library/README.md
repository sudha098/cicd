

## 🔥 What this solution covers (end-to-end)

### CI (Jenkins – Shared Library only)

✔ No Groovy in Jenkinsfile
✔ Centralized pipeline logic
✔ Docker image build
✔ Push to **Google Artifact Registry (GAR)**

### CD (Spinnaker)

✔ Triggered from Jenkins
✔ Kubernetes manifest deployment
✔ Rollback-ready (Spinnaker native)

### Platform

✔ GKE
✔ GAR
✔ Secure credentials
✔ Immutable images

---

## 🧠 How to explain this in interviews (VERY IMPORTANT)

Memorize this 👇

> “We separate CI and CD responsibilities. Jenkins handles build and artifact creation using shared libraries, while Spinnaker handles Kubernetes deployments with rollback and promotion strategies.”

If they ask **WHY** Spinnaker:

> “Spinnaker provides native Kubernetes deployment strategies like rolling, blue-green, and canary which Jenkins is not designed for.”

---

## 🎯 What makes this senior-level

* Jenkinsfile is **declarative only**
* Shared library is **config-driven**
* CI/CD separation
* Artifact-based promotion
* Cloud-native GAR usage

