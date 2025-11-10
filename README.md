# nodejs-demo-app — CI/CD Demo Project

This README is designed to **document your project clearly** for GitHub and reviewers.

---

## 🚀 Project Overview

This project demonstrates a **Node.js web app** automated with **CI/CD using GitHub Actions** and **Docker Hub**. Every push to the `main` branch automatically tests the code, builds a Docker image, and pushes it to Docker Hub.

---

## 📁 Project Structure

```
nodejs-demo-app/
├── .github/workflows/main.yml   # CI/CD pipeline workflow
├── Dockerfile                  # Docker build instructions
├── app.js                      # Node.js app
├── test.js                     # Health check test
├── package.json                # Node metadata & scripts
└── README.md                   # Project documentation
```

---

## ⚙️ How the CI/CD Pipeline Works

1. **Trigger:** On push to `main` branch.
2. **Test Job:** Installs dependencies and runs `npm test` to validate the app.
3. **Build & Push Job:** If tests pass, builds a Docker image and pushes it to Docker Hub.
4. **Secrets:** Docker credentials are securely stored in GitHub Secrets.

---

## 🛠 Setup Instructions

1. Clone this repository.
2. Add repository secrets in GitHub (`Settings → Secrets → Actions`):

   * `DOCKERHUB_USERNAME` — your Docker Hub username
   * `DOCKERHUB_TOKEN` — Docker Hub password or token
   * `DOCKERHUB_REPO` — full image name (e.g., `yourusername/nodejs-demo-app`)
3. Push code to `main` → the pipeline runs automatically.

---

## 💡 Example Code Snippets

### app.js

```javascript
const http = require('http');
const PORT = process.env.PORT || 3000;
const server = http.createServer((req, res) => {
  res.end('Hello from CI/CD pipeline!');
});
server.listen(PORT);
```

### main.yml (Workflow snippet)

```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [ main ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install && npm test
      - uses: docker/build-push-action@v5
```

---

## ✅ Key Concepts

* **CI/CD:** Automates testing, building, and deployment.
* **Docker:** Packages the app with its dependencies to ensure consistency.
* **GitHub Actions:** Runs automated workflows using GitHub-hosted runners.
* **Secrets:** Securely store credentials (DockerHub username/token).
* **Testing:** Validates app functionality before building and deploying.

---

## 🧠 Interview Prep Q&A

**1. What is CI/CD?** Continuous Integration tests code automatically; Continuous Deployment delivers validated code to production.

**2. How do GitHub Actions work?** Workflows trigger on events (push, PR) and run jobs with steps on runners.

**3. What are runners?** Machines (VMs) that execute workflow jobs.

**4. Difference between jobs and steps?** Jobs are groups of steps, steps are individual commands/actions.

**5. How to secure secrets?** Store sensitive credentials in GitHub Secrets and reference them via `${{ secrets.NAME }}`.

**6. How to handle deployment errors?** Check logs, retry or rollback, alert team, fix the underlying issue.

**7. Explain Docker build-push workflow.** Build an image from Dockerfile, tag it, and push to a registry (Docker Hub).

**8. How can you test a CI/CD pipeline locally?** Use tools like `act` or manually run steps locally (tests, docker build, docker run).

---

## 📌 Notes

* For production apps, consider adding unit tests (Jest), linting, caching dependencies, and environment-specific secrets.
* This project is a beginner-friendly example of a complete CI/CD workflow.

---


