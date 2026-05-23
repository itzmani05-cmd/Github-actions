# Automatic Deployment to AWS EC2 using GitHub Actions (Vite React + Nginx)

This guide explains how to automatically deploy your Vite React application to AWS EC2 whenever you push code to GitHub.

### Workflow

```text
VS Code
   ↓
Git Commit
   ↓
Git Push
   ↓
GitHub Actions
   ↓
SSH into EC2
   ↓
Pull latest code
   ↓
Build project
   ↓
Restart Nginx
   ↓
Website Updated Automatically
1. Launch EC2 Instance

Login to AWS Console.

Go to:

EC2 → Launch Instance

Choose:

Ubuntu Server (Latest LTS)

Instance Type:

t2.micro
Configure Security Group

Add inbound rules:

Type	Port	Source
SSH	22	My IP
HTTP	80	0.0.0.0/0
HTTPS	443	0.0.0.0/0

Create key pair and download:

mypassword.pem

Launch instance.

2. Connect EC2 using SSH (Windows CMD/PowerShell)

Move .pem file to .ssh folder:

mkdir $HOME\.ssh -Force
move .\mypassword.pem $HOME\.ssh\

Fix permission issue:

icacls $HOME\.ssh\mypassword.pem /inheritance:r
icacls $HOME\.ssh\mypassword.pem /grant:r "$($env:USERNAME):(R)"

Connect to EC2:

ssh -i "$HOME\.ssh\mypassword.pem" ubuntu@YOUR_PUBLIC_IP

Example:

ssh -i "$HOME\.ssh\mypassword.pem" ubuntu@43.xxx.xxx.xxx

Expected:

ubuntu@ip-xxx-xx-xx-xx:~$
3. Update Ubuntu
sudo apt update
sudo apt upgrade -y
4. Install Node.js and npm

Install latest LTS:

curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

Verify:

node -v
npm -v
5. Install Git
sudo apt install git -y

Verify:

git --version
6. Clone GitHub Repository

Clone repository:

git clone YOUR_GITHUB_REPO_URL

Example:

git clone https://github.com/username/project.git

Go inside project:

cd your-project-name

Check files:

ls

Expected:

package.json
src
vite.config.js
7. Install Dependencies
npm install
8. Build Project
npm run build

Check:

ls dist

Expected:

assets
index.html
9. Install Nginx
sudo apt install nginx -y

Check:

sudo systemctl status nginx

Enable auto start:

sudo systemctl enable nginx
10. Configure Nginx

Open config:

sudo nano /etc/nginx/sites-available/default

Replace everything with:

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /home/ubuntu/your-project-name/dist;
    index index.html;

    server_name _;

    location / {
        try_files $uri /index.html;
    }
}

Save:

CTRL + X
Y
ENTER
11. Fix Permissions
sudo chmod 755 /home/ubuntu
sudo chmod -R 755 /home/ubuntu/your-project-name
12. Test Nginx Configuration
sudo nginx -t

Expected:

syntax is ok
test is successful

Restart Nginx:

sudo systemctl restart nginx
13. Open Website

Open:

http://YOUR_PUBLIC_IP

Example:

http://43.xxx.xxx.xxx
14. Generate SSH Key for GitHub Actions

On Windows PowerShell:

ssh-keygen -t rsa -b 4096 -C "github-actions"

Press Enter for everything.

This creates:

C:\Users\YourName\.ssh\id_rsa
C:\Users\YourName\.ssh\id_rsa.pub
15. Add Public Key to EC2

Copy public key:

type $HOME\.ssh\id_rsa.pub

SSH into EC2:

ssh -i "$HOME\.ssh\mypassword.pem" ubuntu@YOUR_PUBLIC_IP

Open:

nano ~/.ssh/authorized_keys

Paste copied key.

Save:

CTRL + X
Y
ENTER

Fix permissions:

chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

Verify:

cat ~/.ssh/authorized_keys

Should end with:

github-actions
16. Add GitHub Secrets

Go to:

GitHub Repo
↓
Settings
↓
Secrets and variables
↓
Actions

Create these secrets.

EC2_HOST
43.xxx.xxx.xxx
EC2_USERNAME
ubuntu
EC2_SSH_KEY

Copy private key:

Get-Content $HOME\.ssh\id_rsa -Raw

Copy everything including:

-----BEGIN OPENSSH PRIVATE KEY-----

to:

-----END OPENSSH PRIVATE KEY-----

Paste into:

EC2_SSH_KEY
17. Create GitHub Actions Workflow

Create file:

.github/workflows/deploy.yml

Add:

name: Deploy to AWS EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Deploy React App
    runs-on: ubuntu-latest

    steps:
      - name: Deploy to EC2
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd ~/your-project-name

            echo "Fetching latest code..."
            git fetch origin

            echo "Resetting to latest GitHub main..."
            git reset --hard origin/main

            echo "Cleaning old files..."
            git clean -fd

            echo "Installing dependencies..."
            npm install

            echo "Building project..."
            npm run build

            echo "Restarting Nginx..."
            sudo systemctl restart nginx

            echo "Deployment Completed!"

Replace:

your-project-name

with actual folder name.

Example:

casfos-website
18. Push Code to GitHub

From VS Code terminal:

git add .
git commit -m "setup deployment"
git push origin main
19. Automatic Deployment

Now whenever you change code:

Example:

<h1>Hello AWS</h1>

Push:

git add .
git commit -m "new update"
git push origin main

GitHub Actions automatically:

SSH into EC2
↓
Pull latest code
↓
Build app
↓
Restart Nginx
↓
Website updated automatically

No manual SSH required.

20. Check Deployment Status

Go to:

GitHub
↓
Actions

Expected:

✅ Success
21. Troubleshooting
Check Nginx
sudo systemctl status nginx
Restart Nginx
sudo systemctl restart nginx
Check Logs
sudo tail -f /var/log/nginx/error.log
Verify Latest Code
cd ~/your-project-name
git log --oneline -5
Hard Refresh Browser
CTRL + SHIFT + R

or open Incognito mode.

Deployment Completed ✅

Now:

git push origin main

automatically updates your AWS EC2 website.
