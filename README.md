# CI-CD-pipeline



This document contains set of files for the **nodejs-demo-app** CI/CD task. 

## File tree
```
nodejs-demo-app/
├── .github/
│   └── workflows/
│       └── main.yml
├── .gitignore
├── Dockerfile
├── README.md
├── app.js
├── package.json
├── test.js
└── LICENSE
```

---

## .github/workflows/main.yml

```yaml
name: CI/CD - build, test & push Docker image

on:
  push:
    branches: [ main ]

env:
  IMAGE_NAME: ${{ secrets.DOCKERHUB_REPO }}

jobs:
  test:
    name: Start app & run tests
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Install dependencies
        run: npm ci

      - name: Start app in background
        run: |
          npm start &
          # give app a moment to start
          sleep 3

      - name: Run health-test
        run: npm test

  build-and-push:
    name: Build and push Docker image
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: |
            ${{ env.IMAGE_NAME }}:latest
            ${{ env.IMAGE_NAME }}:${{ github.sha }}

      - name: Log pushed image
        run: echo "Pushed: ${{ env.IMAGE_NAME }}:latest and ${{ env.IMAGE_NAME }}:${{ github.sha }}"
```

---

## Dockerfile

```dockerfile
# Use official Node.js LTS image
FROM node:18-alpine

# Create app directory
WORKDIR /usr/src/app

# Copy package.json & package-lock.json first for caching
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy app source
COPY . .

# Expose port
EXPOSE 3000

# Start app
CMD ["node", "app.js"]
```

---

## app.js

```javascript
const http = require('http');
const PORT = process.env.PORT || 3000;

const requestHandler = (req, res) => {
  if (req.url === '/health') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ status: 'ok' }));
    return;
  }

  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from nodejs-demo-app\n');
};

const server = http.createServer(requestHandler);

server.listen(PORT, () => {
  console.log(`Server listening on port ${PORT}`);
});

module.exports = server;
```

---

## package.json

```json
{
  "name": "nodejs-demo-app",
  "version": "1.0.0",
  "description": "Simple demo Node.js app for CI/CD pipeline",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "test": "node test.js"
  },
  "author": "",
  "license": "MIT",
  "dependencies": {}
}
```

> Note: `dependencies` is intentionally empty for a zero-dependency demo. If you add libraries (e.g., express), run `npm install --save express` and commit updated `package.json` and `package-lock.json`.

---

## test.js

```javascript
const http = require('http');

const options = {
  host: 'localhost',
  port: process.env.PORT || 3000,
  path: '/health',
  timeout: 3000
};

const req = http.request(options, (res) => {
  if (res.statusCode === 200) {
    console.log('Health OK');
    process.exit(0);
  } else {
    console.error('Health check failed with status', res.statusCode);
    process.exit(2);
  }
});

req.on('error', (err) => {
  console.error('Request error:', err.message);
  process.exit(2);
});

req.end();
```

---

## .gitignore

```
node_modules/
.env
.DS_Store
```

---

## README.md

````markdown
# nodejs-demo-app

Demo Node.js app to demonstrate a CI/CD pipeline using GitHub Actions and Docker Hub.

## What's included
- `.github/workflows/main.yml` — CI/CD workflow (test, build, push)
- `Dockerfile` — builds the app image
- `app.js`, `test.js`, `package.json` — demo Node app + health test
- `.gitignore`, `LICENSE`

## Quick start
1. Create a GitHub repository called `nodejs-demo-app` and push these files to the `main` branch.
2. Add repository secrets (Settings → Secrets → Actions):
   - `DOCKERHUB_USERNAME` — Docker Hub username
   - `DOCKERHUB_TOKEN` — Docker Hub access token or password
   - `DOCKERHUB_REPO` — full image name, e.g. `yourusername/nodejs-demo-app`
3. Push any change to `main` and watch the workflow under the **Actions** tab.

## Local test
```bash
# start the app
node app.js
# test health endpoint
curl http://localhost:3000/health
````

## Build & run locally with Docker

```bash
docker build -t yourusername/nodejs-demo-app:local .
docker run -p 3000:3000 yourusername/nodejs-demo-app:local
```

## Notes

* The workflow starts the app in the `test` job and runs `npm test` (the health-check script). If you add more tests (jest/mocha) update `package.json` accordingly.

````

---





