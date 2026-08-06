# Deploying a Todo App to AWS EC2 with GitHub Actions and Docker

This guide provides step-by-step instructions to automate the deployment of a Todo App using GitHub Actions, Docker, and AWS EC2.

## Prerequisites
- AWS account with access to EC2.
- GitHub account and a repository for the Todo App.
- SSH key pair for secure access to your EC2 instance.

---

## **Step 1: Clone the Todo App Repository**
Clone your Todo App to your local machine:

```bash
git clone <your-todo-app-repo-url>
cd todo-list-app
```

---

## **Step 2: Push the App to Your GitHub Repository**

1. Initialize a local repository if not already done:
   ```bash
   git init
   ```

2. Add your remote GitHub repository:
   ```bash
   git remote add origin git@github.com:<your-username>/<your-repo>.git
   ```

3. Add and commit the code:
   ```bash
   git add .
   git commit -m "Initial commit for Todo App"
   ```

4. Push the code to GitHub:
   ```bash
   git branch -M master
   git push -u origin master
   ```

---

## **Step 3: Create an AWS EC2 Instance**

1. **Go to the AWS Management Console** and launch a new EC2 instance.
2. Select **Ubuntu 20.04** or later as the OS.
3. Download the key pair (`ec2-key.pem`) when prompted.
4. SSH into the EC2 instance:
   ```bash
   ssh -i /path/to/ec2-key.pem ubuntu@<your-ec2-public-ip>
   ```

---

## **Step 4: Install Docker on EC2**

Run the following commands to install Docker:

```bash
sudo apt update
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu
```

---

## **Step 5: Generate and Add SSH Key**

### **Step A: On Your Local Machine**

1. Generate a new SSH key if not already created:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
   ```

2. Copy the public key:
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

### **Step B: On Your EC2 Instance**

1. Add the public key to `~/.ssh/authorized_keys`:
   ```bash
   nano ~/.ssh/authorized_keys
   ```
   Paste the public key here.

2. Set proper permissions:
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

---

## **Step 6: Set Up GitHub Actions Secrets**

1. **Go to your GitHub repo -> Settings -> Secrets and Variables -> Actions.**
2. Add a new secret named `EC2_SSH_KEY`.
3. Paste the contents of your private SSH key (`~/.ssh/id_rsa`) from your local machine.

---

## **Step 7: Create the GitHub Actions Workflow**
Create a `.github/workflows/deploy.yml` file with the following content:

```yaml
name: Deploy to AWS EC2

on:
  push:
    branches:
      - master  # Trigger deployment when changes are pushed to the 'master' branch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.EC2_SSH_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          echo "StrictHostKeyChecking no" >> ~/.ssh/config

      - name: Deploy to EC2
        run: |
          ssh ubuntu@<your-ec2-public-ip> << 'EOF'
            if [ ! -d "/home/ubuntu/docker-todo" ]; then
              git clone git@github.com:amitslog/docker-todo.git /home/ubuntu/docker-todo
            else
              cd /home/ubuntu/docker-todo
              git pull origin master
            fi

            cd /home/ubuntu/docker-todo
            docker build -t todo-list-app .

            docker stop todo-list-app || true
            docker rm todo-list-app || true

            docker run -d -p 80:3000 --name todo-list-app todo-list-app
          EOF
```

---

## **Step 8: Grant Docker Permissions on EC2**
Allow the `ubuntu` user to run Docker commands without `sudo`:

```bash
sudo usermod -aG docker ubuntu
```

Re-login or reboot for changes to take effect.

---

## **Step 9: Trigger Deployment**
Push changes to the `master` branch to trigger GitHub Actions:

```bash
git add .
git commit -m "Update code"
git push origin master
```

---

## **Step 10: Verify Deployment**
Visit your EC2 public IP in the browser:

```plaintext
http://<your-ec2-public-ip>
```

---

## **Troubleshooting**

- **Permission Issues:** Ensure SSH keys and Docker permissions are correctly configured.
- **Workflow Logs:** Check GitHub Actions logs for errors.
- **Docker Issues:** Verify Docker installation and configuration.

---

## **Summary**
You have set up a full CI/CD pipeline to deploy a **Todo App** to AWS EC2 using **GitHub Actions** and **Docker**. This process automates code updates, Docker image builds, and container restarts whenever changes are pushed to your GitHub repository.

