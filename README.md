# GitHub Actions CI/CD Deployment to AWS EC2

This guide explains how to set up a CI/CD pipeline using GitHub Actions to automatically deploy your application to an AWS EC2 instance whenever code is pushed to a specific branch.

## Prerequisites

* A GitHub repository containing your application code.
* An AWS EC2 instance running and accessible via SSH.
* Docker and Docker Compose installed on the EC2 instance.
* A GitHub repository with Actions enabled.

---

## Step 1: Create the GitHub Actions Workflow Directory

Create the following directory structure in the root of your repository:

```text
.github/
└── workflows/
```

Inside the `workflows` folder, create a YAML file, for example:

```text
.github/workflows/deploy.yml
```

---

## Step 2: Configure the GitHub Actions Workflow

Add a workflow that triggers whenever code is pushed to a specific branch.

Example:

```yaml
name: Deploy Node.js Application to AWS EC2

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /home/ubuntu/GitHub-Actions-Learning
            git pull
            sudo docker compose up -d --build
```

---

## Step 3: Create and Configure an EC2 Instance

Launch an EC2 instance that will host your application.

Install the required software:

```bash
sudo apt update
sudo apt install -y git docker.io docker-compose-plugin
```

Enable and start Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Add the current user to the Docker group:

```bash
sudo usermod -aG docker ubuntu
```

Reconnect to the instance after running the above command.

---

## Step 4: Configure SSH Access

Generate an SSH key pair:

```bash
ssh-keygen -t rsa -b 4096
```

This generates:

```text
id_rsa      (Private Key)
id_rsa.pub  (Public Key)
```

### Add the Public Key to EC2

Copy the contents of `id_rsa.pub` and add it to:

```text
~/.ssh/authorized_keys
```

on the EC2 instance.

Example:

```bash
cat id_rsa.pub >> ~/.ssh/authorized_keys
```

---

## Step 5: Configure GitHub Secrets

Navigate to:

```text
Repository Settings
→ Secrets and Variables
→ Actions
→ New Repository Secret
```

Create the following secrets:

| Secret Name  | Description                          |
| ------------ | ------------------------------------ |
| SSH_HOST     | Public IP or DNS of the EC2 instance |
| SSH_USERNAME | SSH user (e.g., ubuntu)              |
| SSH_KEY      | Contents of the private SSH key      |

Example:

```text
SSH_HOST=54.xx.xx.xx
SSH_USERNAME=ubuntu
SSH_KEY=<contents of id_rsa>
```

---

## Step 6: Push Code to Trigger Deployment

Commit and push your changes:

```bash
git add .
git commit -m "Setup CI/CD pipeline"
git push origin master
```

GitHub Actions will automatically:

1. Trigger on push to the `master` branch.
2. Create a GitHub-hosted runner.
3. Checkout the repository code.
4. Connect to the EC2 instance via SSH.
5. Pull the latest code.
6. Rebuild Docker images.
7. Start or update the running containers.

---

## Deployment Flow

```text
Developer
    │
    ├── git push origin master
    │
    ▼
GitHub Repository
    │
    ▼
GitHub Actions Runner
    │
    ├── Checkout Code
    ├── SSH into EC2
    │
    ▼
AWS EC2 Instance
    ├── git pull
    ├── docker compose build
    └── docker compose up -d
```

---

## Verification

Check the GitHub Actions tab to verify that the workflow completed successfully.

On the EC2 instance, verify that the containers are running:

```bash
docker ps
```

View logs:

```bash
docker compose logs -f
```

Your CI/CD pipeline is now configured and will automatically deploy changes whenever code is pushed to the configured branch.
