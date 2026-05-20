A full-featured, production-ready Ecommerce web application built with the MERN stack.
Deployed on AWS EC2 with a complete CI/CD pipeline using Jenkins, Nginx, and PM2.
🌐 Live Demo · 🔧 Jenkins Dashboard · 🐛 Report Bug
</div>

📸 Application Preview
Home PageAdmin DashboardPremium Quality Guaranteed — Hero Banner with Shop & Top Brands CTAKPI metrics: Users, Products, Orders, Revenue

🌐 Live at: http://13.233.77.198


✨ Features
👤 User Features

🔐 Authentication — Register & Login with JWT-based secure sessions
🛍️ Product Browsing — Browse all products with search and category filter
🔍 Search & Filter — Search by name, filter by category, sort by price
🛒 Shopping Cart — Add, remove, update quantity in real time
📦 Order Placement — Place orders with shipping address
📋 Order History — View all past orders with expandable item details

🔧 Admin Features

📊 Dashboard — View total users, products, orders & revenue
📦 Product Management — Add, edit, delete products with image upload
🧾 Order Management — View and update order status
👥 User Management — View all registered users

🛡️ Security Features

JWT Authentication with token expiry
Bcrypt password hashing
Role-based access control (User / Admin)
Protected routes on both frontend & backend
CORS configured for production


🧰 Tech Stack
LayerTechnologyFrontendReact.js 18, Vite, Tailwind CSSBackendNode.js, Express.jsDatabaseMongoDB Atlas (Mongoose ODM)AuthenticationJWT + BcryptImage StorageCloudinaryFile UploadMulter + StreamifierState ManagementReact Context APIRoutingReact Router v6HTTP ClientAxiosWeb ServerNginx (Reverse Proxy)Process ManagerPM2CI/CDJenkins PipelineCloudAWS EC2 (Mumbai — ap-south-1)

🏗️ Architecture Overview
                        ┌─────────────────────────────────────┐
                        │         AWS EC2 Instance            │
                        │      (c7i-flex.large, Mumbai)       │
                        │   Public IP: 13.233.77.198          │
                        │                                     │
   User Browser  ──────►│  Nginx (Port 80)                    │
                        │    │                                │
                        │    ├── Serves React Build (static)  │
                        │    └── Proxy → Node.js :5000        │
                        │              │                      │
                        │           PM2 Process               │
                        │     (ecommerce-backend)             │
                        │              │                      │
                        │              └──► MongoDB Atlas      │
                        │                  (Cloud DB)         │
                        │                                     │
   GitHub Push  ───────►│  Jenkins (Port 8080)                │
   (Webhook)            │  CI/CD Auto Deploy Pipeline         │
                        └─────────────────────────────────────┘

🚀 CI/CD Pipeline (Jenkins + GitHub Webhook)
This project uses a fully automated CI/CD pipeline. Every git push to the main branch automatically triggers a Jenkins build that deploys the latest code to the AWS EC2 server — zero manual deployment needed.
🔄 Pipeline Flow
Developer pushes code
        │
        ▼
  GitHub (main branch)
        │
        │  Webhook POST trigger
        ▼
  Jenkins Server (43.204.35.151:8080)
        │
        ▼
  ┌─────────────────────────────────────────────────┐
  │              Jenkins Pipeline Stages            │
  │                                                 │
  │  Stage 1: Clone Repository          (~1s)  ✅  │
  │  Stage 2: Install Backend Deps      (~2s)  ✅  │
  │  Stage 3: Install Frontend Deps     (~1s)  ✅  │
  │  Stage 4: Build Frontend            (~3s)  ✅  │
  │  Stage 5: Deploy Frontend           (~616ms) ✅ │
  │  Stage 6: Restart Backend           (~1s)  ✅  │
  └─────────────────────────────────────────────────┘
        │
        ▼
  Live App Updated at http://13.233.77.198
  Total Pipeline Time: ~10 seconds ⚡
📄 Jenkinsfile
groovypipeline {
    agent any
    environment {
        APP_DIR = "/home/ubuntu/react-node-ecommerce"
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Adarsh7307/react-node-ecommerce.git'
            }
        }
        stage('Install Backend Dependencies') {
            steps {
                dir('backend') {
                    sh 'npm install'
                }
            }
        }
        stage('Install Frontend Dependencies') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                }
            }
        }
        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm run build'
                }
            }
        }
        stage('Deploy Frontend') {
            steps {
                sh 'sudo rm -rf /var/www/html/*'
                sh 'sudo cp -r frontend/dist/* /var/www/html/'
            }
        }
        stage('Restart Backend') {
            steps {
                dir('backend') {
                    sh '''
                    pm2 restart ecommerce-backend || \
                    pm2 start src/index.js --name ecommerce-backend
                    '''
                }
            }
        }
    }
}
✅ Pipeline Stage Details
StageWhat It DoesTimeClone RepositoryPulls latest code from GitHub main branch~1sInstall Backend DependenciesRuns npm install in /backend folder~2sInstall Frontend DependenciesRuns npm install in /frontend folder~1sBuild FrontendRuns npm run build — Vite compiles React to static files~3sDeploy FrontendCopies dist/ build output to /var/www/html/ (Nginx root)~616msRestart BackendRestarts PM2 process ecommerce-backend (or starts fresh)~1s

☁️ AWS Infrastructure
EC2 Instance Details
PropertyValueInstance IDi-03be927a8bc277128Instance Namemy-pm2-deploymentInstance Typec7i-flex.largeRegionAsia Pacific (Mumbai) — ap-south-1Public IPv413.233.77.198Private IPv4172.31.45.124OSUbuntu (Linux)State🟢 Running
🔒 Security Group — Inbound Rules
TypeProtocolPortPurposeSSHTCP22Remote server accessHTTPTCP80Public web traffic (Nginx)HTTPSTCP443Secure web trafficCustom TCPTCP8080Jenkins dashboard access

🌐 Nginx Configuration
Nginx serves as both the static file server for the React frontend and a reverse proxy for the Node.js backend API.
nginx# /etc/nginx/sites-available/default

server {
    listen 80;
    server_name 13.233.77.198;

    # Serve React build (frontend)
    root /var/www/html;
    index index.html;

    # SPA routing fix — serve index.html for all routes
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Proxy API requests to Node.js backend
    location /api/ {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
Check Nginx status:
bashsystemctl status nginx
# ● nginx.service - A high performance web server
#    Active: active (running) since May 16 16:23:42 UTC

⚙️ PM2 — Backend Process Manager
PM2 keeps the Node.js backend alive permanently — it auto-restarts if the process crashes.
bash# Start backend for the first time
pm2 start src/index.js --name ecommerce-backend

# Useful PM2 commands
pm2 status                        # View all running processes
pm2 logs ecommerce-backend        # View live backend logs
pm2 restart ecommerce-backend     # Restart the backend
pm2 stop ecommerce-backend        # Stop the backend
pm2 startup                       # Auto-start PM2 on server reboot
pm2 save                          # Save current process list
PM2 Process Status:
NameStatusCPUMemoryUptimeecommerce-backend🟢 online0%~30.8 MBstable

🔗 GitHub Webhook Setup
The GitHub Webhook connects your repository to Jenkins for automatic deployments.
How it Works

You push code → GitHub fires a POST request to Jenkins
Jenkins receives the trigger → starts the pipeline automatically
App is deployed within ~10 seconds ⚡

Configured Webhooks
URLEventsStatushttp://54.87.35.231:8080/github-we...All events❌ Failed (old server)http://43.204.35.151:8080/github-w...All events✅ Last delivery successful

The second webhook (43.204.35.151) is the active Jenkins server — last delivery was successful.


📁 Project Structure
react-node-ecommerce/
│
├── 📂 frontend/                  # React.js Frontend (Vite)
│   ├── src/
│   │   ├── components/           # Reusable UI components
│   │   │   ├── Navbar.jsx
│   │   │   ├── ProductCard.jsx
│   │   │   ├── PrivateRoute.jsx
│   │   │   └── AdminRoute.jsx
│   │   ├── pages/                # Page-level components
│   │   │   ├── Home.jsx
│   │   │   ├── Shop.jsx
│   │   │   ├── Cart.jsx
│   │   │   ├── Orders.jsx
│   │   │   ├── Login.jsx
│   │   │   ├── Register.jsx
│   │   │   └── admin/
│   │   │       ├── Dashboard.jsx
│   │   │       ├── Products.jsx
│   │   │       ├── Orders.jsx
│   │   │       └── Users.jsx
│   │   ├── context/
│   │   │   ├── AuthContext.jsx
│   │   │   └── CartContext.jsx
│   │   ├── api/
│   │   │   └── axios.js
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── public/
│   └── package.json
│
├── 📂 backend/                   # Node.js + Express API
│   ├── src/
│   │   ├── config/
│   │   │   ├── db.js
│   │   │   └── cloudinary.js
│   │   ├── controllers/
│   │   ├── middleware/
│   │   │   └── authMiddleware.js
│   │   ├── models/
│   │   │   ├── User.js
│   │   │   ├── Product.js
│   │   │   └── Order.js
│   │   ├── routes/
│   │   └── app.js
│   ├── index.js
│   └── package.json
│
├── Jenkinsfile                   # ← CI/CD pipeline definition
└── README.md

🛠️ Local Development Setup
Prerequisites

Node.js v18+, npm v9+, Git
MongoDB Atlas account
Cloudinary account

1. Clone the Repository
bashgit clone https://github.com/Adarsh7307/react-node-ecommerce.git
cd react-node-ecommerce
2. Backend Setup
bashcd backend
npm install
Create backend/.env:
envPORT=5000
MONGO_URI=mongodb+srv://<user>:<pass>@cluster0.mongodb.net/mern-shop
JWT_SECRET=your_jwt_secret
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
CLIENT_URL=http://localhost:5173
bashnpm run dev   # Starts at http://localhost:5000
3. Frontend Setup
bashcd ../frontend
npm install
Create frontend/.env:
envVITE_API_URL=http://localhost:5000
bashnpm run dev   # Starts at http://localhost:5173

🔌 API Endpoints
Auth — /api/auth
MethodEndpointAuthDescriptionPOST/register❌Register new userPOST/login❌Login, get JWTGET/profile✅Get current user
Products — /api/products
MethodEndpointAuthDescriptionGET/❌Get all productsGET/:id❌Get single productPOST/🔒 AdminAdd product + imagePUT/:id🔒 AdminUpdate productDELETE/:id🔒 AdminDelete product
Orders — /api/orders
MethodEndpointAuthDescriptionPOST/✅Place orderGET/✅Get my ordersPUT/:id🔒 AdminUpdate status
Admin — /api/admin
MethodEndpointAuthDescriptionGET/dashboard🔒 AdminKPI metricsGET/users🔒 AdminAll usersGET/orders🔒 AdminAll orders

🔐 Environment Variables
VariableWhereDescriptionMONGO_URIBackendMongoDB Atlas connection stringJWT_SECRETBackendJWT signing secretCLOUDINARY_CLOUD_NAMEBackendCloudinary cloud nameCLOUDINARY_API_KEYBackendCloudinary API keyCLOUDINARY_API_SECRETBackendCloudinary API secretCLIENT_URLBackendFrontend URL (CORS)PORTBackendServer port (default 5000)VITE_API_URLFrontendBackend API base URL

⚠️ Never commit .env files! Already in .gitignore.


📊 Live Production Stats
MetricValue👤 Registered Users13📦 Products Listed10🧾 Total Orders18💰 Total Revenue₹53,58,458🚀 Jenkins Builds#2 (latest SUCCESS)⚡ Deploy Time~10 seconds

👥 Team
NameRoleRoll NoMuhammad Siddique ShaikhFull Stack Developer2204190100027Adarsh TiwariBackend + DevOps2204190100004Subham KumarFrontend Developer2204190100050
College: Prabhat Engineering College, Kanpur Dehat
University: AKTU, Lucknow | Session: 2025-26 | B.Tech CSE
Guided By: Dr. Nirvikar Katiyar & Mr. Mohd. Azhar Naushad

🛠️ Troubleshooting
ProblemFixJenkins build fails at "Deploy Frontend"Run echo "jenkins ALL=(ALL) NOPASSWD: /bin/rm, /bin/cp" >> /etc/sudoers on EC2Backend not startingSSH into EC2 → pm2 logs ecommerce-backend to see errorNginx showing 502 Bad GatewayBackend is down → pm2 restart ecommerce-backendWebhook not triggeringCheck Jenkins URL in GitHub → Settings → WebhooksPage refresh returns 404Nginx config missing try_files $uri /index.html


