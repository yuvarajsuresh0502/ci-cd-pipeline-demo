lone the repository to your local machine:

bash
git clone https://github.com/<your-username>/ci-cd-pipeline-demo.git
cd ci-cd-pipeline-demo


Create a simple Node.js application:

Create package.json:

json
{
  "name": "ci-cd-pipeline-demo",
  "version": "1.0.0",
  "description": "A simple CI/CD pipeline demo",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "jest": "^29.5.0",
    "supertest": "^6.3.3"
  }
}

Create app.js:

javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.send('Hello from our CI/CD Pipeline!');
});

app.get('/health', (req, res) => {
  res.status(200).json({ status: 'OK' });
});

module.exports = app;

if (require.main === module) {
  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
  });
}


Create a simple test file test/app.test.js:

javascript
const request = require('supertest');
const app = require('../app');

describe('GET /', () => {
  it('should return Hello message', async () => {
    const res = await request(app).get('/');
    expect(res.statusCode).toEqual(200);
    expect(res.text).toContain('Hello from our CI/CD Pipeline!');
  });
});

describe('GET /health', () => {
  it('should return health status', async () => {
    const res = await request(app).get('/health');
    expect(res.statusCode).toEqual(200);
    expect(res.body).toEqual({ status: 'OK' });
  });
});


Create a Dockerfile:

dockerfile
# Use official Node.js runtime as base image
FROM node:18-alpine

# Set working directory
WORKDIR /usr/src/app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Define health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# Run the application
CMD ["npm", "start"]


Create a health check script healthcheck.js:

javascript
const http = require('http');

const options = {
  host: 'localhost',
  port: 3000,
  path: '/health',
  timeout: 2000
};

const request = http.request(options, (res) => {
  console.log(`STATUS: ${res.statusCode}`);
  if (res.statusCode === 200) {
    process.exit(0);
  } else {
    process.exit(1);
  }
});

request.on('error', (err) => {
  console.log('ERROR', err);
  process.exit(1);
});

request.end();


Create docker-compose.yml:

yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    restart: unless-stopped


    Create .dockerignore:

text
node_modules
npm-debug.log
.git
.github
Dockerfile
docker-compose.yml
README.md


Create the directory structure for GitHub Actions:

bash
mkdir -p .github/workflows


Create .github/workflows/ci-cd.yml:

yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ secrets.DOCKERHUB_USERNAME }}/ci-cd-pipeline-demo:latest

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Deploy to local environment (simulated)
      run: |
        echo "Deployment would happen here"
        echo "Image pushed to: ${{ secrets.DOCKERHUB_USERNAME }}/ci-cd-pipeline-demo:latest"
        echo "In a real scenario, this would trigger deployment to your local environment"



        Configure Secrets in GitHub Repository
Go to your GitHub repository → Settings → Secrets and variables → Actions

Add the following secrets:

DOCKERHUB_USERNAME: Your Docker Hub username

DOCKERHUB_TOKEN: Your Docker Hub access token

To create a Docker Hub access token:

Log in to Docker Hub

Go to Account Settings → Security → New Access Token

Generate a token with read/write permissions



Test the application locally:

bash
npm install
npm startc




