# React-FastAPI Finance Application

This is a full-stack finance application with React frontend and FastAPI backend, demonstrating both local development and containerized deployment.

---

## Running Locally (Development)

### Prerequisites
- Python 3.8+
- Node.js 18+
- npm

### Backend (FastAPI)

1. Open terminal and cd into FastAPI
```bash
   cd FastAPI
```

2. Install dependencies
```bash
   pip install -r requirements.txt
```

3. Run the server
```bash
   uvicorn main:app --reload
```

   Server runs on: `http://localhost:8000`

### Frontend (React)

Open new terminal (keep backend running)

1. Navigate to React app
```bash
   cd React/finance-app
```

2. Install dependencies
```bash
   npm install
   npm install axios
```

3. Start development server
```bash
   npm start
```

   App runs on: `http://localhost:3000`

**Note:** Both terminals should remain running simultaneously.

---

## Running with Docker

### Prerequisites
- Docker
- Docker Compose

### Steps

1. Clone the repository
```bash
   git clone https://github.com/Adewuumii/react-fastapi-app.git
   cd react-fastapi-app
```

2. Build and run with Docker Compose
```bash
   docker compose up -d
```

3. Access the application
   - Frontend: `http://localhost:3000`
   - Backend: `http://localhost:8000`

4. Stop the application
```bash
   docker compose down
```

### Architecture

- **Frontend:** Multi-stage build with Node.js (build) + Nginx (serve)
- **Backend:** Python 3.11-slim with FastAPI
- **Port Mapping:** React (3000:80), FastAPI (8000:8000)

---

## AWS Deployment

### EC2 Public IP 

3.252.238.153 

**Note:** The EC2 instance used for this project will be disabled after some time to avoid AWS costs.

- **Frontend (React):** http://3.252.238.153:3000
- **Backend (FastAPI):** http://3.252.238.153:8000/docs

---

###  ECR Public Images

**React Frontend:**
public.ecr.aws/j8j0w8u1/finance-app-react-public:latest

**FastAPI Backend:**
public.ecr.aws/j8j0w8u1/finance-app-fastapi-public:latest

---

### Deployment Steps

#### Step 1: Clone Repository
```bash
git clone https://github.com/Adewuumii/react-fastapi-app.git
cd react-fastapi-app
```

#### Step 2: Create Two Dockerfiles

See [`FastAPI/Dockerfile`](/FastAPI/Dockerfile)

Also [`React/finance-app/Dockerfile`](/React/finance-app/Dockerfile)

#### Step 3: Build Docker Images

```bash
# Build React with API URL
cd React/finance-app
docker build --build-arg REACT_APP_API_URL=http://<EC2_PUBLIC_IP>:8000 -t finance-app-react:latest .

# Build FastAPI
cd ../../FastAPI
docker build -t finance-app-fastapi:latest .
```

#### Step 4: Create ECR Repositories

1. Go to AWS Console → ECR
2. Create **4 repositories**:
   - Private: `finance-app-react-private`
   - Private: `finance-app-fastapi-private`
   - Public: `finance-app-react-public`
   - Public: `finance-app-fastapi-public`

#### Step 5: Authenticate with ECR

**Private ECR:**
```bash
aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.eu-west-1.amazonaws.com
```

**Public ECR:**
```bash
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws
```

#### Step 6: Tag and Push Images
```bash
# Tag for private ECR
docker tag finance-app-react:latest <ACCOUNT_ID>.dkr.ecr.eu-west-1.amazonaws.com/finance-app-react-private:latest
docker tag finance-app-fastapi:latest <ACCOUNT_ID>.dkr.ecr.eu-west-1.amazonaws.com/finance-app-fastapi-private:latest

# Push to private ECR
docker push <ACCOUNT_ID>.dkr.ecr.eu-west-1.amazonaws.com/finance-app-react-private:latest
docker push <ACCOUNT_ID>.dkr.ecr.eu-west-1.amazonaws.com/finance-app-fastapi-private:latest

# Tag for public ECR
docker tag finance-app-react:latest public.ecr.aws/<alias>/finance-app-react-public:latest
docker tag finance-app-fastapi:latest public.ecr.aws/<alias>/finance-app-fastapi-public:latest

# Push to public ECR
docker push public.ecr.aws/<alias>/finance-app-react-public:latest
docker push public.ecr.aws/<alias>/finance-app-fastapi-public:latest
```

#### Step 7: Launch EC2 Instance

1. Create Ubuntu 24.04 t2.micro instance
2. Configure Security Group:
   - Port 22 (SSH)
   - Port 3000 (React)
   - Port 8000 (FastAPI)

#### Step 8: Setup EC2 Environment

SSH into EC2:
```bash
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
```

Install Docker and set it up:
```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker ubuntu
sudo apt install docker-compose-v2 -y
```

Install AWS CLI:
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
```

Configure AWS:
```bash
aws configure
# Enter: Access Key, Secret Key, Region (eu-west-1), Output format (json)
```

Authenticate with ECR:
```bash
aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.eu-west-1.amazonaws.com
```

#### Step 9: Deploy with Docker Compose

Create `docker-compose.yml`.

See [`docker-compose.yml`](./docker-compose.yml) 

Copy to EC2 (from local):
```bash
scp -i your-key.pem docker-compose.yml ubuntu@<EC2_PUBLIC_IP>:~/
```

Run on EC2:
```bash
docker compose up -d
docker ps  
```

---
## Technologies Used

### Frontend
- React 18.2.0
- Axios
- React Scripts
- Nginx (production server)

### Backend
- Python 3.11
- FastAPI 0.110.0
- Uvicorn
- SQLAlchemy 2.0.29
- Pydantic 2.6.4

### DevOps
- Docker & Docker Compose
- AWS ECR (Elastic Container Registry)
- AWS EC2
- GitHub

---

## Project Structure
```
react-fastapi-app/
├── FastAPI/
│   ├── database.py
│   ├── main.py
│   ├── models.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .dockerignore
├── React/
│   └── finance-app/
│       ├── src/
│       ├── public/
│       ├── package.json
│       ├── Dockerfile
│       └── .dockerignore
├── docker-compose.yml
└── README.md
```
