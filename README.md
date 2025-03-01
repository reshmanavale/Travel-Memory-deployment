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
<img width="881" alt="installed nodejs on backend" src="https://github.com/user-attachments/assets/75fcd43e-bfaa-49e5-bcd4-61dddd42f836" />

## 2. Backend Configuration

### Clone Repository and Setup Backend

```sh
git clone <your-repo-url>
cd TravelMemory/backend
```
<img width="696" alt="git clone to frontend server" src="https://github.com/user-attachments/assets/36a17943-2a6f-4aab-9495-736615823faa" />

### Install Dependencies

```sh
npm install
```
<img width="905" alt="installed npm on backend server" src="https://github.com/user-attachments/assets/86b7f672-faa4-4c50-914c-a58fe9cbb3d6" />


### Configure Environment Variables

Create a `.env` file and add:
```sh
PORT=3001
MONGO_URI=mongodb+srv://<your-mongo-uri>
```
<img width="779" alt="sercurity_group" src="https://github.com/user-attachments/assets/f97b3ab4-f008-46d7-a609-0b39c8ee0845" />

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
<img width="659" alt="configured_reverce_proxy" src="https://github.com/user-attachments/assets/7592ad8f-978b-46df-82e5-67b76ab9c04d" />

<img width="799" alt="connecting to backend server" src="https://github.com/user-attachments/assets/d5113fd2-aad3-4bf6-9a89-d893d306c798" />


<img width="520" alt="backend_server_running" src="https://github.com/user-attachments/assets/92806eef-1890-4048-9de5-d009662e9b33" />

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
<img width="663" alt="added_ip_to_frontend" src="https://github.com/user-attachments/assets/29207cee-91e9-4563-90cb-ed68eeebef9d" />

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
<img width="456" alt="successfully_started_server" src="https://github.com/user-attachments/assets/163cd6b1-651f-4a6b-9862-dfa9761566a2" />

<img width="599" alt="frontend server after nginx" src="https://github.com/user-attachments/assets/b77380d3-aa7f-476c-83dc-7d973867fa37" />

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

   <img width="938" alt="image" src="https://github.com/user-attachments/assets/6503009c-4f0d-4d10-9762-6ce951db8320" />


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

