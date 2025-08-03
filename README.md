# 🚀 Next.js CI/CD Deployment to AWS EC2 with GitHub Actions & PM2

This guide sets up a fully automated CI/CD pipeline to deploy your **Next.js app** to an **AWS EC2 (Ubuntu)** instance using **GitHub Actions**, **PM2**, and **Appleboy SSH Action**.

---

## 📦 Tech Stack

- **Frontend Framework**: Next.js (React)
- **Deployment Host**: AWS EC2 (Ubuntu 20.04+)
- **CI/CD Tool**: GitHub Actions
- **Process Manager**: PM2
- **Remote Executor**: Appleboy SSH Action

---

## 📋 Prerequisites

Before setting up CI/CD, ensure you have:

1. ✅ AWS EC2 instance (Ubuntu 20.04 or later)
2. ✅ Node.js, npm, PM2 installed on EC2
3. ✅ SSH access via `.pem` key
4. ✅ GitHub repository with your Next.js app
5. ✅ GitHub Actions secret:
   - `PRIVATE_SSH_KEY` (paste your `.pem` private key content)

---

## 🖥️ EC2 Initial Setup Guide

SSH into your EC2 instance:

```bash
ssh -i "your-key.pem" ubuntu@your-ec2-ip
```
🔧 Install Node.js using NVM (Recommended)
We will follow the DigitalOcean Option 3 method (Full Guide):

Step 1: Install NVM
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
source ~/.bashrc
```
Step 2: Install Latest Node.js
```bash
nvm install node
```
✅ This will install the latest LTS version of Node.js and set it as default.

⚙️ Install PM2
PM2 is used to keep your Next.js app running after logout or reboot.
```bash
npm install -g pm2
pm2 startup ubuntu
```
📂 Clone Your Repository
```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git EC2-CICD-Nextjs
cd EC2-CICD-Nextjs

npm install
```
🔐 GitHub Secret Setup
```sql
GitHub → Settings → Secrets and variables → Actions → New repository secret
```
Add:

PRIVATE_SSH_KEY: Paste your private EC2 .pem key (no passphrase)(private key you created locally)

Add: yoor public key in EC2 .ssh/Authorized_key this location

⚙️ GitHub Actions CI/CD Workflow
Create .github/workflows/deploy.yml in your repo:
```yaml
name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Deploy to EC2
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: SSH & Deploy to EC2
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: YOUR_EC2_PUBLIC_IP
          username: ubuntu
          key: ${{ secrets.PRIVATE_SSH_KEY }}
          port: 22
          script: |
            cd ~/EC2-CICD-Nextjs || exit 1
            git pull origin main
            export NVM_DIR="$HOME/.nvm"
            [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
            nvm install node
            nvm use node
            npm install
            npm run build

            # Kill process on port 3000 if exists
            PID=$(sudo lsof -ti :3000)
            if [ -n "$PID" ]; then
              echo "Killing port 3000 (PID: $PID)..."
              sudo kill -9 $PID
            fi

            # Use PM2 to run app
            npm install -g pm2
            pm2 delete nextjs || true
            pm2 start npm --name "nextjs" -- run start
            pm2 save

```
🌀 Why PM2?
PM2 is a production-grade process manager for Node.js apps.

✅ Auto-restarts app on failure
✅ Keeps app running after SSH session closes
✅ Saves process list for reboot recovery
```bash
pm2 start npm --name "nextjs" -- run start
pm2 save
```
📚 References
## 📚 References

- 🧾 [DigitalOcean – Install Node.js on Ubuntu (Option 3)](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04#option-3-install-node-using-the-node-version-manager)
- 🧾 [Deploy Next.js to EC2 via GitHub Actions](https://www.interviewsolutionshub.com/blog/how-to-deploy-a-nextjs-application-on-aws-ec2-using-github-actions)
- 🧾 [PM2 Documentation](https://pm2.keymetrics.io/)
- 🧾 [Appleboy SSH GitHub Action](https://github.com/appleboy/ssh-action)


👨‍💻 Author & Contributions
This CI/CD guide was built to help developers deploy scalable and automated Next.js apps to AWS. Contributions are welcome!
