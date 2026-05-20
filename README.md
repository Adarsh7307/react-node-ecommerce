🛒 MERN Shop — Full Stack Ecommerce App with CI/CD Pipeline
A full-stack ecommerce application built with the MERN stack (MongoDB, Express, React, Node.js), deployed on AWS EC2 using Jenkins CI/CD, PM2, and Nginx.

Every git push to main automatically builds and deploys the app via GitHub Webhooks + Jenkins Pipeline.


🚀 Live Demo
Frontend: http://13.233.77.198

🏗️ Architecture Overview
GitHub (push) → Webhook → Jenkins (CI/CD) → Build & Deploy
                                               ↓
                                         Nginx (port 80) → React Frontend (dist/)
                                         PM2             → Node.js Backend
Infrastructure:

Cloud: AWS EC2 — c7i-flex.large — Asia Pacific (Mumbai) ap-south-1
Web Server: Nginx — serves React build files, reverse proxy for backend
Process Manager: PM2 — keeps Node.js backend alive, auto-restarts on crash
CI/CD: Jenkins — running on port 8080
Trigger: GitHub Webhook — fires on every push to main


🧰 Tech Stack
LayerTechnologyFrontendReact + ViteBackendNode.js + ExpressDatabaseMongoDBWeb ServerNginxProcess ManagerPM2CI/CDJenkinsCloudAWS EC2 (Ubuntu)

⚙️ Jenkins Pipeline
The Jenkins pipeline (Jenkinsfile) has 6 stages:
groovypipeline {
  agent any
  environment {
    APP_DIR = "/home/ubuntu/react-node-ecommerce"
  }
  stages {
    stage('Clone Repository') {
      steps {
        git branch: 'main', url: 'https://github.com/Adarsh7307/react-node-ecommerce.git'
      }
    }
    stage('Install Backend Dependencies') {
      steps {
        dir('backend') { sh 'npm install' }
      }
    }
    stage('Install Frontend Dependencies') {
      steps {
        dir('frontend') { sh 'npm install' }
      }
    }
    stage('Build Frontend') {
      steps {
        dir('frontend') { sh 'npm run build' }
      }
    }
    stage('Deploy Frontend') {
      steps {
        sh 'sudo rm -rf /var/www/html/'
        sh 'sudo cp -r frontend/dist/ /var/www/html/'
      }
    }
    stage('Restart Backend') {
      steps {
        dir('backend') {
          sh 'pm2 restart ecommerce-backend || pm2 start src/index.js --name ecommerce-backend'
        }
      }
    }
  }
}
Pipeline Stage View
StageBuild #1Build #2Clone Repository✅ 1s✅ 757msInstall Backend Dependencies✅ 4s✅ 2sInstall Frontend Dependencies✅ 5s✅ 1sBuild Frontend✅ 3s✅ 3sDeploy Frontend❌ 507ms (failed)✅ 616msRestart Backend❌ skipped✅ 1s

Build #1 failed — sudo permission was not granted to Jenkins user for rm -rf /var/www/html/.
Fix: Added Jenkins user to sudoers with NOPASSWD for required commands.
Build #2 — All 6 stages passed ✅, app went live!


🔧 Server Setup
AWS EC2 Security Group — Inbound Rules
TypeProtocolPortSSHTCP22HTTPTCP80HTTPSTCP443Custom TCP (Jenkins)TCP8080
Nginx Config
Nginx serves the React production build (/var/www/html/) on port 80.
bash# Check nginx status
systemctl status nginx
PM2 — Backend Process Management
PM2 keeps the Express backend running as a daemon (ecommerce-backend).
bash# Check running processes
pm2 list

# Restart backend manually
pm2 restart ecommerce-backend

# View logs
pm2 logs ecommerce-backend

🔗 GitHub Webhook Setup

Go to your GitHub repo → Settings → Webhooks → Add webhook
Payload URL: http://<your-jenkins-ip>:8080/github-webhook/
Content type: application/json
Events: Just the push event (or all events)
Jenkins job → Configure → Build Triggers → ✅ GitHub hook trigger for GITScm polling

Now every git push to main will automatically trigger the Jenkins pipeline.

📁 Project Structure
react-node-ecommerce/
├── frontend/          # React + Vite app
│   ├── src/
│   ├── dist/          # Production build (generated)
│   └── package.json
├── backend/           # Express + Node.js API
│   ├── src/
│   │   └── index.js   # Entry point
│   └── package.json
└── Jenkinsfile        # CI/CD pipeline definition

🐛 Troubleshooting
Jenkins sudo: I'm afraid I can't do that error:
Jenkins user doesn't have sudo access by default. Fix:
bashsudo visudo
# Add this line:
jenkins ALL=(ALL) NOPASSWD: /bin/rm, /bin/cp, /usr/bin/npm
PM2 process not found on restart:
The pipeline uses || pm2 start ... fallback — if the process doesn't exist, it starts fresh.
Nginx not serving updated build:
Manually trigger a new Jenkins build or check if the Deploy Frontend stage ran successfully.

👤 Author
Adarsh — @Adarsh7307


