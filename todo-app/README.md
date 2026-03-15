# DSO101 Assignment I - CI/CD Pipeline

**Student Name:** UgayNobu  
**Student ID:** 02240369  
**Course:** DSO101 - Continuous Integration and Continuous Deployment  

---

---

## Part A: Deploying a Pre-Built Docker Image to Docker Hub Registry

### Overview
In Part A, the backend and frontend Docker images were built locally and pushed to Docker Hub. The images were then deployed on Render.com using the "Existing Image from Docker Hub" option.

---

### Step 1: Login to Docker Hub

Logged into Docker Hub from the terminal using:
```bash
docker login
```

![Docker Login](todo-app/images/Screenshot_2026-03-15_at_6_52_12_PM.png)

---

### Step 2: Create Backend Dockerfile

Created `todo-app/backend/Dockerfile`:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
RUN npm install pg
COPY . .
EXPOSE 5000
CMD ["node", "server.postgres.js"]
```

---

### Step 3: Build and Push Backend Image

Built the backend Docker image with the student ID as the tag and pushed it to Docker Hub:
```bash
# Build for linux/amd64 platform (required by Render)
docker buildx build --platform linux/amd64 \
  -t ugaynobu/be-todo:02240369 \
  --push --provenance=false --sbom=false .
```

![Backend Build](todo-app/images/Screenshot_2026-03-15_at_8_02_42_PM.png)

---

### Step 4: Create Frontend Dockerfile

Created `todo-app/frontend/Dockerfile`:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

---

### Step 5: Build and Push Frontend Image
```bash
docker buildx build --platform linux/amd64 \
  -t ugaynobu/fe-todo:02240369 \
  --push --provenance=false --sbom=false .
```

![Frontend Build and Push](todo-app/images/Screenshot_2026-03-15_at_8_21_02_PM.png)

---

### Step 6: Verify Images on Docker Hub

Both images successfully pushed to Docker Hub at `https://hub.docker.com/u/ugaynobu`:

- `ugaynobu/be-todo:02240369`
- `ugaynobu/fe-todo:02240369`

![Docker Hub Repositories](todo-app/images/Screenshot_2026-03-15_at_6_54_16_PM.png)

---

### Step 7: Create PostgreSQL Database on Render

1. Logged into [render.com](https://render.com)
2. Clicked **New +** → **PostgreSQL**
3. Configured:
   - **Name:** `todo-db`
   - **Region:** Singapore
   - **Plan:** Free
4. Clicked **Create Database**

![Render PostgreSQL Created](todo-app/images/Screenshot_2026-03-15_at_8_01_38_PM.png)

---

### Step 8: Deploy Backend on Render

1. Clicked **New +** → **Web Service** → **Existing Image**
2. Entered image: `ugaynobu/be-todo:02240369`
3. Set region to **Singapore**, plan to **Free**
4. Added the following environment variables:

| Key | Value |
|-----|-------|
| `DB_HOST` | `dpg-d6rbkpogjchc73f8r4g0-a` |
| `DB_USER` | `todo_db_tx5b_user` |
| `DB_PASSWORD` | `****` |
| `DB_NAME` | `todo_db_tx5b` |
| `DB_PORT` | `5432` |
| `DB_SSL` | `true` |
| `PORT` | `5000` |

![Backend Environment Variables](todo-app/images/Screenshot_2026-03-15_at_8_27_48_PM.png)

5. Clicked **Deploy Web Service**

![Backend Live on Render](todo-app/images/Screenshot_2026-03-15_at_8_29_00_PM.png)

**Backend URL:** `https://be-todo-02240369.onrender.com`

Health check confirmed working:

![Backend Health Check](todo-app/images/Screenshot_2026-03-15_at_8_30_08_PM.png)

---

### Step 9: Deploy Frontend on Render

1. Clicked **New +** → **Web Service** → **Existing Image**
2. Entered image: `ugaynobu/fe-todo:02240369`
3. Added environment variable:

| Key | Value |
|-----|-------|
| `NEXT_PUBLIC_API_URL` | `https://be-todo-02240369.onrender.com` |

4. Clicked **Deploy Web Service**

![Frontend Deploying](todo-app/images/Screenshot_2026-03-15_at_8_31_32_PM.png)

**Frontend URL:** `https://fe-todo-t7y5.onrender.com`

---

### Part A Result

Both services successfully deployed and the To-Do List application is live:

- **Backend:** `https://be-todo-02240369.onrender.com`
- **Frontend:** `https://fe-todo-t7y5.onrender.com`

---

## Part B: Automated Image Build and Deployment

### Overview
In Part B, the deployment was configured to automatically build and deploy a new Docker image every time a new commit is pushed to the GitHub repository. This was achieved using Render Blueprints with a `render.yaml` file.

---

### Step 1: Create `.env.production` Files

**`backend/.env.production`:**
```
DB_HOST=dpg-d6rbkpogjchc73f8r4g0-a
DB_USER=todo_db_tx5b_user
DB_PASSWORD=****
DB_NAME=todo_db_tx5b
DB_PORT=5432
DB_SSL=true
PORT=5000
```

**`frontend/.env.production`:**
```
NEXT_PUBLIC_API_URL=https://be-todo.onrender.com
```

---

### Step 2: Create `render.yaml` Blueprint File

Created `todo-app/render.yaml` to define the multi-service deployment:
```yaml
services:
  - type: web
    name: be-todo-github
    runtime: docker
    branch: main
    dockerfilePath: ./todo-app/backend/Dockerfile
    dockerContext: ./todo-app/backend
    region: singapore
    plan: free
    envVars:
      - key: DB_HOST
        value: dpg-d6rbkpogjchc73f8r4g0-a
      - key: DB_USER
        value: todo_db_tx5b_user
      - key: DB_PASSWORD
        value: CuoopvLPgWQALVAZAHqSmDxwGGiInhBi
      - key: DB_NAME
        value: todo_db_tx5b
      - key: DB_PORT
        value: "5432"
      - key: DB_SSL
        value: "true"
      - key: PORT
        value: "5000"
  - type: web
    name: fe-todo-github
    runtime: docker
    branch: main
    dockerfilePath: ./todo-app/frontend/Dockerfile
    dockerContext: ./todo-app/frontend
    region: singapore
    plan: free
    envVars:
      - key: NEXT_PUBLIC_API_URL
        value: https://be-todo-github.onrender.com
```

---

### Step 3: Push to GitHub

Committed and pushed all files to GitHub:
```bash
git add .
git commit -m "Part B: Add render.yaml and env.production files"
git push origin main
```

![GitHub Repository with render.yaml](todo-app/images/Screenshot_2026-03-15_at_11_02_32_PM.png)

---

### Step 4: Create Render Blueprint

1. Logged into [render.com](https://render.com)
2. Clicked **New +** → **Blueprint**
3. Selected the GitHub repository: `UgayNobu/UgayNobu_02240369_DSO101_2026`
4. Configured:
   - **Blueprint Name:** `todo-app-blueprint`
   - **Branch:** `main`
   - **Blueprint Path:** `todo-app/render.yaml`

![Blueprint Configuration](todo-app/images/Screenshot_2026-03-15_at_11_07_20_PM.png)

---

### Step 5: Apply Blueprint and Deploy

Render detected the `render.yaml` file and automatically created two new services:

- `be-todo-github` — Backend built from GitHub Dockerfile
- `fe-todo-github` — Frontend built from GitHub Dockerfile

![Blueprint Sync Running](todo-app/images/Screenshot_2026-03-15_at_11_24_22_PM.png)

Both services were successfully deployed:

- **Backend (GitHub):** `https://be-todo-github.onrender.com`
- **Frontend (GitHub):** `https://fe-todo-github.onrender.com`

---

### Step 6: Auto-Deploy Verification

Every time a new commit is pushed to the `main` branch on GitHub, Render automatically triggers a new build and deployment. This was verified by pushing a test commit:
```bash
git add .
git commit -m "Part B: Test auto-deploy trigger"
git push origin main
```

Render automatically detected the new commit and started a new deployment without any manual intervention — confirming the CI/CD pipeline is working correctly.

---

### Part B Result

The automated CI/CD pipeline is fully configured:

| Feature | Status |
|---------|--------|
| GitHub → Render auto-deploy | ✅ Working |
| `render.yaml` Blueprint | ✅ Configured |
| Backend auto-build from Dockerfile | ✅ Working |
| Frontend auto-build from Dockerfile | ✅ Working |
| New deploy on every git push | ✅ Confirmed |

---

## Live URLs Summary

| Service | URL | Type |
|---------|-----|------|
| Backend (Part A) | `https://be-todo-02240369.onrender.com` | Docker Hub Image |
| Frontend (Part A) | `https://fe-todo-t7y5.onrender.com` | Docker Hub Image |
| Backend (Part B) | `https://be-todo-github.onrender.com` | GitHub Auto-Deploy |
| Frontend (Part B) | `https://fe-todo-github.onrender.com` | GitHub Auto-Deploy |

---

## References

- [Docker Documentation](https://docs.docker.com/)
- [Render Documentation](https://render.com/docs)
- [Render Blueprint Spec](https://render.com/docs/blueprint-spec)
- [Render Deploy from Image](https://render.com/docs/deploying-an-image)
- [Render Environment Variables](https://render.com/docs/configure-environment-variables)
