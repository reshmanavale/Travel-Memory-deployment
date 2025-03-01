# TravelMemory Deployment Guide

This guide provides step-by-step instructions to deploy and scale the TravelMemory application on AWS EC2 instances with Nginx, MongoDB, and a Load Balancer.

## 1. Install Node.js

```sh
sudo apt update
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
node -v  # Verify installation
npm -v   # Verify npm installation
```

## 2. Backend Configuration

### Clone Repository and Setup Backend

```sh
git clone <your-repo-url>
cd TravelMemory/backend
```

### Install Dependencies

```sh
npm install
```

### Configure Environment Variables

Create a `.env` file and add:
```sh
PORT=3001
MONGO_URI=mongodb+srv://<your-mongo-uri>
```

### Start Backend Server
```sh
node index.js
```

### Set Up Nginx Reverse Proxy

1. Install Nginx:
```sh
sudo apt install -y nginx
```

2. Configure Nginx:
```sh
sudo nano /etc/nginx/sites-available/backend
```
Add the following configuration:
```nginx
server {
    listen 80;
    server_name <your-backend-public-ip>;

    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

3. Enable Configuration & Restart Nginx:
```sh
sudo ln -s /etc/nginx/sites-available/backend /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

## 3. Frontend Configuration

### Setup Frontend

```sh
cd TravelMemory/frontend
npm install
```

### Update API URL in Frontend

Edit `src/config/urls.js` and replace `API_BASE_URL` with the backend URL:
```js
export const API_BASE_URL = 'http://<your-backend-public-ip>/api';
```

### Build and Serve Frontend
```sh
npm run build
sudo npm install -g serve
serve -s build -l 3000
```

### Configure Nginx for Frontend

1. Create Nginx Config:
```sh
sudo nano /etc/nginx/sites-available/frontend
```
Add the following configuration:
```nginx
server {
    listen 80;
    server_name <your-frontend-public-ip>;

    root /home/ubuntu/TravelMemory/frontend/build;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```

2. Enable Configuration & Restart Nginx:
```sh
sudo ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

## 4. Scaling the Application

### Create an AMI for Backend and Frontend
1. Go to **EC2 Dashboard** → **Instances**.
2. Select the **Backend EC2 instance** → Click **Actions** → **Create Image** (AMI).
3. Select the **Frontend EC2 instance** → Repeat the process.

### Create Target Groups
1. Go to **EC2 Dashboard** → **Target Groups**.
2. Click **Create Target Group**.
3. Choose **Instance** and specify:
   - **Backend Target Group**: Port 3001
   - **Frontend Target Group**: Port 80
4. Register backend and frontend instances.

### Create an Application Load Balancer
1. Go to **Load Balancers** → Click **Create Load Balancer**.
2. Choose **Application Load Balancer**.
3. Add **Listeners**:
   - Port 80 → **Frontend Target Group**
   - Port 3001 → **Backend Target Group**
4. Assign **Instances** to respective target groups.
5. Create and test the load balancer.

## 5. Verify Deployment
- Visit `<Load Balancer DNS>` to check the frontend.
- API should be accessible via `<Load Balancer DNS>/api/trip`.

## 6. Push to GitHub
```sh
git add .
git commit -m "Added deployment guide"
git push origin main
```

Your TravelMemory app is now fully deployed and scalable! 🚀

