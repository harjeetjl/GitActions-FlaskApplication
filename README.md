
# Automated Deployment of Flask Applications Using GitHub Actions and EC2

This project showcases a robust continuous integration and delivery (CI/CD) pipeline for a Flask-based web application. It utilizes **GitHub Actions** to manage automated testing and deployment to **Amazon EC2** instances, supporting both staging and production environments.

---

## Repository Structure

- `app.py` – Primary Flask application logic  
- `test_app.py` – Unit tests written using `pytest`  
- `requirements.txt` – List of required Python packages  
- `.github/workflows/ci-cd.yml` – CI/CD pipeline configuration  
- `.env` – Populated dynamically during deployment with sensitive configuration  

---

## Pipeline Design

### Deployment Triggers

| Branch/Event        | Target Environment |
|---------------------|--------------------|
| Push to `staging`   | Staging            |
| GitHub Release Tag  | Production         |

### Workflow Stages

1. **Code Checkout** – Fetches the repository source code  
2. **Python Setup** – Prepares the Python runtime  
3. **Dependency Installation** – Installs dependencies from `requirements.txt`  
4. **Test Execution** – Runs test suite with `pytest`  
5. **SSH Key Handling** – Reconstructs private key from base64 for secure server access  
6. **Remote Deployment** – Executes deployment commands on EC2 server over SSH  

---

## Environment Configuration

During deployment, the pipeline provisions a `.env` file on the remote server, containing sensitive data such as database credentials. This is powered by GitHub Secrets.

**Example `.env` file:**
```dotenv
MONGO_URI=mongodb+srv://user:password@cluster.mongodb.net/database
```

**Required line in `app.py`:**
```python
from dotenv import load_dotenv
load_dotenv()
```

---

## Required GitHub Secrets

These must be defined in the GitHub repository under **Settings > Secrets and Variables > Actions**.

| Secret Key           | Description                            |
|----------------------|----------------------------------------|
| `MONGO_URI`          | MongoDB connection string              |
| `EC2_USER`           | Username for SSH access                |
| `EC2_KEY`            | Base64-encoded SSH private key         |
| `EC2_HOST_STAGING`   | Public IP of the staging EC2 instance  |
| `EC2_HOST_PRODUCTION`| Public IP of the production EC2 instance |

---

## Running Flask as a systemd Service on EC2

To ensure the Flask app runs persistently on EC2, configure it as a systemd service. Example service file (`/etc/systemd/system/flaskapp.service`):

```ini
[Unit]
Description=Flask Web Application
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/home/ec2-user/GitActions-FlaskApplication
EnvironmentFile=/home/ec2-user/GitActions-FlaskApplication/.env
ExecStart=/home/ec2-user/GitActions-FlaskApplication/venv/bin/python app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

**To apply and launch the service:**
```bash
sudo systemctl daemon-reexec
sudo systemctl enable flaskapp
sudo systemctl start flaskapp
```

---

**Screenshots:**
1. Github Actions

2. Logs
