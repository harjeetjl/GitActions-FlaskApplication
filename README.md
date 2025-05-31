
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
WorkingDirectory=/home/ec2-user/GithubActions_FlaskApp
EnvironmentFile=/home/ec2-user/GithubActions_FlaskApp/.env
ExecStart=/home/ec2-user/GithubActions_FlaskApp/venv/bin/python app.py
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
https://private-user-images.githubusercontent.com/84853581/448058426-2281bb55-6d8f-4d2a-ab70-885b59035b44.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDg2OTgyNjUsIm5iZiI6MTc0ODY5Nzk2NSwicGF0aCI6Ii84NDg1MzU4MS80NDgwNTg0MjYtMjI4MWJiNTUtNmQ4Zi00ZDJhLWFiNzAtODg1YjU5MDM1YjQ0LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA1MzElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwNTMxVDEzMjYwNVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQ2NDA4YzRhNDgwYmQyYTkyNWMxZDQxMjY4NGQ3ZDc3YjUwMWYwYzFkMTQ2MDMyOTBkMDM2OWY5MjE4N2NjNjUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.7Ad4hNSoQowQVTXG1oedjgxfHKwkL2BoWuzbEx3tC80
https://private-user-images.githubusercontent.com/84853581/448058426-2281bb55-6d8f-4d2a-ab70-885b59035b44.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDg2OTgyNjUsIm5iZiI6MTc0ODY5Nzk2NSwicGF0aCI6Ii84NDg1MzU4MS80NDgwNTg0MjYtMjI4MWJiNTUtNmQ4Zi00ZDJhLWFiNzAtODg1YjU5MDM1YjQ0LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA1MzElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwNTMxVDEzMjYwNVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQ2NDA4YzRhNDgwYmQyYTkyNWMxZDQxMjY4NGQ3ZDc3YjUwMWYwYzFkMTQ2MDMyOTBkMDM2OWY5MjE4N2NjNjUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.7Ad4hNSoQowQVTXG1oedjgxfHKwkL2BoWuzbEx3tC80
https://private-user-images.githubusercontent.com/84853581/448058426-2281bb55-6d8f-4d2a-ab70-885b59035b44.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDg2OTgyNjUsIm5iZiI6MTc0ODY5Nzk2NSwicGF0aCI6Ii84NDg1MzU4MS80NDgwNTg0MjYtMjI4MWJiNTUtNmQ4Zi00ZDJhLWFiNzAtODg1YjU5MDM1YjQ0LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA1MzElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwNTMxVDEzMjYwNVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQ2NDA4YzRhNDgwYmQyYTkyNWMxZDQxMjY4NGQ3ZDc3YjUwMWYwYzFkMTQ2MDMyOTBkMDM2OWY5MjE4N2NjNjUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.7Ad4hNSoQowQVTXG1oedjgxfHKwkL2BoWuzbEx3tC80
https://private-user-images.githubusercontent.com/84853581/448058426-2281bb55-6d8f-4d2a-ab70-885b59035b44.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDg2OTgyNjUsIm5iZiI6MTc0ODY5Nzk2NSwicGF0aCI6Ii84NDg1MzU4MS80NDgwNTg0MjYtMjI4MWJiNTUtNmQ4Zi00ZDJhLWFiNzAtODg1YjU5MDM1YjQ0LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA1MzElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwNTMxVDEzMjYwNVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQ2NDA4YzRhNDgwYmQyYTkyNWMxZDQxMjY4NGQ3ZDc3YjUwMWYwYzFkMTQ2MDMyOTBkMDM2OWY5MjE4N2NjNjUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.7Ad4hNSoQowQVTXG1oedjgxfHKwkL2BoWuzbEx3tC80
2. Logs
https://private-user-images.githubusercontent.com/84853581/448058426-2281bb55-6d8f-4d2a-ab70-885b59035b44.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDg2OTgyNjUsIm5iZiI6MTc0ODY5Nzk2NSwicGF0aCI6Ii84NDg1MzU4MS80NDgwNTg0MjYtMjI4MWJiNTUtNmQ4Zi00ZDJhLWFiNzAtODg1YjU5MDM1YjQ0LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA1MzElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwNTMxVDEzMjYwNVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQ2NDA4YzRhNDgwYmQyYTkyNWMxZDQxMjY4NGQ3ZDc3YjUwMWYwYzFkMTQ2MDMyOTBkMDM2OWY5MjE4N2NjNjUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.7Ad4hNSoQowQVTXG1oedjgxfHKwkL2BoWuzbEx3tC80
https://private-user-images.githubusercontent.com/84853581/448058426-2281bb55-6d8f-4d2a-ab70-885b59035b44.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDg2OTgyNjUsIm5iZiI6MTc0ODY5Nzk2NSwicGF0aCI6Ii84NDg1MzU4MS80NDgwNTg0MjYtMjI4MWJiNTUtNmQ4Zi00ZDJhLWFiNzAtODg1YjU5MDM1YjQ0LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA1MzElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwNTMxVDEzMjYwNVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQ2NDA4YzRhNDgwYmQyYTkyNWMxZDQxMjY4NGQ3ZDc3YjUwMWYwYzFkMTQ2MDMyOTBkMDM2OWY5MjE4N2NjNjUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.7Ad4hNSoQowQVTXG1oedjgxfHKwkL2BoWuzbEx3tC80
https://private-user-images.githubusercontent.com/84853581/448058426-2281bb55-6d8f-4d2a-ab70-885b59035b44.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDg2OTgyNjUsIm5iZiI6MTc0ODY5Nzk2NSwicGF0aCI6Ii84NDg1MzU4MS80NDgwNTg0MjYtMjI4MWJiNTUtNmQ4Zi00ZDJhLWFiNzAtODg1YjU5MDM1YjQ0LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA1MzElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwNTMxVDEzMjYwNVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQ2NDA4YzRhNDgwYmQyYTkyNWMxZDQxMjY4NGQ3ZDc3YjUwMWYwYzFkMTQ2MDMyOTBkMDM2OWY5MjE4N2NjNjUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.7Ad4hNSoQowQVTXG1oedjgxfHKwkL2BoWuzbEx3tC80