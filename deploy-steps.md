Here's the formatted text for a GitHub README file:

---
# youtube vide0
Code With Clinton
https://www.youtube.com/watch?v=tqRo922F9ao

# Deploying a Django Project to an AWS EC2 Instance

Deploying a Django project to an AWS EC2 instance involves several steps, including setting up the EC2 instance, preparing your Django project, configuring the server environment, and setting up a production server with Nginx and Gunicorn. Here's a step-by-step guide to help you through the process:

## Step 1: Set Up Your EC2 Instance

### Create an AWS EC2 Instance:
1. Log in to your AWS Management Console.
2. Go to the EC2 Dashboard and click on "Launch Instance."
3. Choose an Amazon Machine Image (AMI). For a basic setup, the Ubuntu Server is a good choice.
4. Select an instance type (e.g., t2.micro for free tier).
5. Configure instance details, add storage, and configure security groups (allow SSH, HTTP, and HTTPS).
6. Review and launch the instance. You will be prompted to create or use an existing key pair for SSH access.

### Connect to Your EC2 Instance:
Open your terminal and connect to your instance using SSH:

```bash
ssh -i /path/to/your-key.pem ubuntu@your-ec2-public-ip
```

## Step 2: Prepare Your Django Project

### Clone Your Project Repository:
Install Git if it's not already installed:

```bash
sudo apt update
sudo apt install git
```

Clone your Django project repository:

```bash
git clone https://github.com/yourusername/your-django-repo.git
cd your-django-repo
```

### Set Up a Virtual Environment:
Install Python and virtual environment tools:

```bash
sudo apt install python3-pip python3-venv
```

Create and activate a virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
```

### Install Dependencies:
Install your project's dependencies:

```bash
pip install -r requirements.txt
```

## Step 3: Configure the Server Environment

### Install and Configure PostgreSQL:
Install PostgreSQL:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

Set up a PostgreSQL database and user for your Django project:

```bash
sudo -u postgres psql
CREATE DATABASE yourdbname;
CREATE USER yourdbuser WITH PASSWORD 'yourpassword';
ALTER ROLE yourdbuser SET client_encoding TO 'utf8';
ALTER ROLE yourdbuser SET default_transaction_isolation TO 'read committed';
ALTER ROLE yourdbuser SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE yourdbname TO yourdbuser;
\q
```

Update your Django settings to use the PostgreSQL database.

### Run Database Migrations:

```bash
python manage.py migrate
```

## Step 4: Set Up Gunicorn and Nginx

### Install Gunicorn:

```bash
pip install gunicorn
```

### Create a Gunicorn Systemd Service File:
Create and edit the service file:

```bash
sudo nano /etc/systemd/system/gunicorn.service
```

Add the following configuration:

```ini
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/your-django-repo
ExecStart=/home/ubuntu/your-django-repo/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/ubuntu/your-django-repo.sock yourprojectname.wsgi:application

[Install]
WantedBy=multi-user.target
```

Reload systemd to apply the changes:

```bash
sudo systemctl daemon-reload
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
sudo systemctl status gunicorn
ls -l /home/ubuntu/django_cocker/gunicorn.sock
```

### Install and Configure Nginx:

Install Nginx:

```bash
sudo apt install nginx
```

Create an Nginx configuration file for your project:

```bash
sudo nano /etc/nginx/sites-available/yourproject
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name your_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/your-django-repo;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/your-django-repo.sock;
    }
}
```

Enable the Nginx configuration:

```bash
sudo ln -s /etc/nginx/sites-available/yourproject /etc/nginx/sites-enabled
sudo nginx -t

# allow permission
sudo systemctl restart nginx
sudo systemctl status nginx
```
#------------------------------
chmod o+rx /home/ubuntu
 chmod -R 755 /home/ubuntu/django_cocker
 sudo ufw allow 'Nginx Full'
 sudo systemctl restart nginx
 sudo systemctl restart gunicorn
#------------------------------


### Update Firewall Settings:
Allow Nginx through the firewall:

```bash
sudo ufw allow 'Nginx Full'
```

---

This guide provides a clear, step-by-step process for deploying a Django project to an AWS EC2 instance.





# CI CD pipeline


## ssh-keygen -t rsa -b 4096 -C "github-action-ec2"
### cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

cat ~/.ssh/id_rsa

Copy the entire content dont skip --- copy all  and go to your GitHub repository → Settings → Secrets → New repository secret

Name: SSH_PRIVATE_KEY

Paste the private key

Add another secret: EC2_PUBLIC_IP with value 44.220.61.135





## Github actions
create this folder and paster all code in this
# -----------------COpy all code donst skip any line--------------------------
.github/workflows/deploy.yml

-----------------------

name: Deploy Django App to EC2

# Trigger deployment on push to main branch
on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    # Step 1: Checkout the code from GitHub
    - name: Checkout code
      uses: actions/checkout@v3

    # Step 2: Deploy to EC2
    - name: Deploy to EC2
      env:
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
        EC2_PUBLIC_IP: ${{ secrets.EC2_PUBLIC_IP }}
      run: |
        # Save the private key
        echo "$SSH_PRIVATE_KEY" > key.pem
        chmod 600 key.pem

        # SSH into EC2 and deploy
        ssh -o StrictHostKeyChecking=no -i key.pem ubuntu@$EC2_PUBLIC_IP << 'EOF'
          cd /home/ubuntu/django_cocker

          # Reset any local changes and pull latest code
          git reset --hard
          git pull origin main

          # Activate the EC2 virtual environment
          source /home/ubuntu/django_cocker/venv/bin/activate

          # Install any new dependencies
          pip install -r requirements.txt

          # Collect static files
          python manage.py collectstatic --noinput

          # Restart services
          sudo systemctl restart gunicorn
          sudo systemctl restart nginx
        EOF
---------------------------------------------------------------------

























