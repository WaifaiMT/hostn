# Deploying a React.js App from GitHub to VPS with Nginx and Auto-Updates

## Prerequisites
- A VPS running Ubuntu (or similar Linux distribution)
- A domain name pointed to your VPS
- A GitHub repository with your React.js app
- SSH access to your VPS

## Step-by-step Procedure

### 1. Set up your VPS

1.1. Update your system:
```bash
sudo apt update && sudo apt upgrade -y
```

1.2. Install Node.js and npm (for building the React app):
```bash
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs
```

1.3. Install Nginx:
```bash
sudo apt install nginx -y
```

1.4. Start and enable Nginx:
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 2. Set up your project directory

2.1. Create a directory for your app:
```bash
sudo mkdir -p /var/www/react-app
sudo chown -R $USER:$USER /var/www/react-app
```

### 3. Configure Nginx

3.1. Create a new Nginx configuration file:
```bash
sudo nano /etc/nginx/sites-available/react-app
```

3.2. Add the following configuration:
```nginx
server {
    listen 80;
    server_name yourdomain.com;
    root /var/www/react-app/build;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

3.3. Enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/react-app /etc/nginx/sites-enabled/
```

3.4. Test Nginx configuration:
```bash
sudo nginx -t
```

3.5. If the test is successful, reload Nginx:
```bash
sudo systemctl reload nginx
```

### 4. Set up GitHub Actions

4.1. In your GitHub repository, go to Settings > Secrets and add the following secrets:
- `HOST`: Your VPS IP address
- `USERNAME`: Your VPS SSH username
- `SSH_PRIVATE_KEY`: Your SSH private key

4.2. Create a `.github/workflows/deploy.yml` file in your repository:

```yaml
name: Deploy to VPS

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Install Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'
    - name: Install npm dependencies
      run: npm install
    - name: Run build
      run: npm run build
    - name: Deploy to VPS
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /var/www/react-app
          git pull origin main
          npm install
          npm run build
          sudo systemctl reload nginx
```

### 5. Initial manual deployment

5.1. On your local machine, push your code to GitHub:
```bash
git push origin main
```

5.2. SSH into your VPS and perform the first manual deployment:
```bash
cd /var/www/react-app
git clone https://github.com/yourusername/your-repo.git .
npm install
npm run build
```

### 6. Set up automatic updates

6.1. Create a post-receive hook in your VPS:
```bash
mkdir -p /var/repo/react-app.git
cd /var/repo/react-app.git
git init --bare
```

6.2. Create a post-receive hook:
```bash
nano hooks/post-receive
```

6.3. Add the following content:
```bash
#!/bin/bash
TARGET="/var/www/react-app"
GIT_DIR="/var/repo/react-app.git"
BRANCH="main"

while read oldrev newrev ref
do
    # only checking out the main (or any specified) branch
    if [ "$ref" = "refs/heads/$BRANCH" ];
    then
        echo "Ref $ref received. Deploying ${BRANCH} branch to production..."
        git --work-tree=$TARGET --git-dir=$GIT_DIR checkout -f $BRANCH
        cd $TARGET
        npm install
        npm run build
        sudo systemctl reload nginx
    else
        echo "Ref $ref received. Doing nothing: only the ${BRANCH} branch may be deployed on this server."
    fi
done
```

6.4. Make the hook executable:
```bash
chmod +x hooks/post-receive
```

6.5. On your local machine, add the VPS as a remote:
```bash
git remote add production ssh://username@your_server_ip/var/repo/react-app.git
```

Now, every time you push to the main branch on GitHub, the GitHub Action will trigger and deploy your app to the VPS. Additionally, you can manually trigger a deployment by pushing to the production remote:

```bash
git push production main
```

This setup provides both automated deployment through GitHub Actions and the ability to manually deploy when needed.
