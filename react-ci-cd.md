# Deploying Node.js and React Application on VPS with CI/CD

## Prerequisites

- A Hostinger VPS
- GitHub repository for your project
- Node.js installed on both local machine and VPS
- PM2 for process management
- Nginx as a reverse proxy
- GitHub Actions for CI/CD

## Step 1: Local Project Structure Preparation

Ensure your project structure is organized:

```
your-project/
│
├── server/           # Node.js backend
│   ├── src/
│   ├── package.json
│   └── Dockerfile
│
├── client/           # React frontend
│   ├── src/
│   ├── package.json
│   └── Dockerfile
│
└── .github/
    └── workflows/
        └── deploy.yml
```

## Step 2: VPS Initial Setup

1. SSH into your Hostinger VPS

```bash
ssh root@your_vps_ip
```

2. Update system packages

```bash
sudo apt update && sudo apt upgrade -y
```

3. Install essential tools

```bash
sudo apt install -y nodejs npm nginx certbot python3-certbot-nginx docker.io docker-compose git
```

4. Create deployment user

```bash
sudo adduser deployer
sudo usermod -aG sudo,docker deployer
```

## Step 3: Configure Nginx as Reverse Proxy

Create Nginx configuration:

```nginx
# /etc/nginx/sites-available/your-app
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;  # React frontend
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /api {
        proxy_pass http://localhost:5000;  # Node.js backend
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/your-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## Step 4: SSL Configuration with Certbot

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

## Step 5: GitHub Actions Workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to VPS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"

      - name: Install server dependencies
        working-directory: ./server
        run: npm ci

      - name: Install client dependencies
        working-directory: ./client
        run: npm ci

      - name: Build client
        working-directory: ./client
        run: npm run build

      - name: Deploy to VPS
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USERNAME }}
          password: ${{ secrets.VPS_PASSWORD }}
          # Alternatively use SSH key
          # key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /path/to/your/app
            git pull origin main

            # Backend deployment
            cd server
            npm ci
            pm2 restart server-app || pm2 start npm --name "server-app" -- start

            # Frontend deployment
            cd ../client
            npm ci
            npm run build
            pm2 restart client-app || pm2 start npm --name "client-app" -- start

            # Restart Nginx if needed
            sudo nginx -t && sudo systemctl restart nginx
```

## Step 6: GitHub Secrets Configuration

In your GitHub repository:

1. Go to Settings > Secrets and Variables > Actions
2. Add these secrets:
   - `VPS_HOST`: Your VPS IP address
   - `VPS_USERNAME`: Deployment user
   - `VPS_PASSWORD`: Deployment user password
   - `VPS_SSH_KEY`: SSH key (optional, more secure)

## Step 7: PM2 Process Management

Install PM2 globally on VPS:

```bash
sudo npm install -g pm2
```

## Step 8: Docker Containers (Optional)

Create `Dockerfile` for both server and client:

Server Dockerfile:

```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 5000
CMD ["npm", "start"]
```

Client Dockerfile:

```dockerfile
FROM node:18 as build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Troubleshooting

- Check Nginx logs: `sudo tail -f /var/log/nginx/error.log`
- Check PM2 logs: `pm2 logs`
- Verify GitHub Actions workflow in repository Actions tab

## Notes

- Always keep your system and dependencies updated
- Use strong, unique passwords
- Consider using SSH key authentication instead of passwords
- Regularly backup your data

## Security Best Practices

1. Use environment variables for sensitive information
2. Implement proper CORS settings
3. Use HTTPS
4. Keep all software updated
5. Implement proper firewall rules
