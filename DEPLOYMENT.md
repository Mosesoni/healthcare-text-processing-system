# Deployment Guide: Healthcare Text Processing System

This document outlines the architecture, deployment process, and configuration instructions used to migrate the Healthcare Text Processing System from a local environment (XAMPP/MySQL) to a production-ready cloud environment.

---

## 🏗️ Architecture Overview

The system is split into two secure cloud layers:
1. **Frontend / Application Layer**: Hosted on **Render** (Platform-as-a-Service) using a Python Flask runtime powered by `gunicorn`.
2. **Database Layer**: A managed **MySQL** database hosted on **Aiven Cloud** using SSL encryption.

```
       ┌────────────────────────┐
       │      Client Browser    │
       └───────────┬────────────┘
                   │ HTTPS
                   ▼
       ┌────────────────────────┐
       │   Render Web Service   │ (Flask / Python / Gunicorn)
       └───────────┬────────────┘
                   │ MySQL over SSL (Port 12867)
                   ▼
       ┌────────────────────────┐
       │   Aiven Cloud MySQL    │ (Managed DB Instance)
       └────────────────────────┘
```

---

## 🔐 Configuration & Security

To prevent security credentials (passwords, hosts) from leaking onto public GitHub repositories, configuration is decoupled into:
- **Local Development**: Configured via a local file `db_config.local.json` (git-ignored/not uploaded).
- **Production (Render)**: Configured using **Environment Variables** injected via the Render Dashboard.

---

## 🛠️ Step-by-Step Deployment Instructions

### 1. Database Provisioning (Aiven MySQL)
- Create a free MySQL database instance on [Aiven.io](https://aiven.io/).
- Retrieve the connection details:
  - **Host**: `mysql-...aivencloud.com`
  - **Port**: `12867`
  - **User**: `avnadmin`
  - **Password**: `AVNS_...`
  - **Database**: `defaultdb`
- Download the **Aiven CA Certificate** (`ca.pem`) and save it in the root directory of your project. This cert is required to establish a secure SSL tunnel.

### 2. Local Config Setup
Create a file named `db_config.local.json` in the root of the project to allow the app to run locally:
```json
{
    "host": "YOUR_AIVEN_HOST",
    "user": "avnadmin",
    "password": "YOUR_AIVEN_PASSWORD",
    "database": "defaultdb",
    "port": 12867,
    "ssl_ca": "ca.pem"
}
```
*Note: Do not commit `db_config.local.json` to GitHub.*

### 3. Repository Preparation
Upload the following files/folders to your GitHub repository:
- `app.py` (Main Flask application code)
- `ca.pem` (Aiven SSL certificate)
- `requirements.txt` (List of dependencies)
- `static/` (CSS, JS, images)
- `templates/` (HTML templates)

**Do NOT upload:**
- `db_config.local.json` (Contains passwords)
- `__pycache__/` (Python compiled bytecode)
- `error.log` (Local session logs)

### 4. Deploying to Render
1. Register/Log in to [Render.com](https://render.com) using your **GitHub** account.
2. Click **New +** ➔ **Web Service**.
3. Select and connect your repository (e.g., `bi-directional-data-intelligence-web-app` or your renamed repository name).
4. Configure the settings:
   - **Runtime**: `Python`
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: `gunicorn app:app`
   - **Instance Type**: `Free`
5. Go to the **Environment** tab on Render and add the following keys:
   - `DB_HOST` ➔ Your Aiven host URL
   - `DB_USER` ➔ `avnadmin`
   - `DB_PASSWORD` ➔ Your Aiven password
   - `DB_NAME` ➔ `defaultdb`
   - `DB_PORT` ➔ `12867`
   - `DB_SSL_CA` ➔ `ca.pem`
6. Click **Deploy**. Render will install dependencies, initialize the database tables dynamically on first connection, and serve your application live.

---

## ⚡ Troubleshooting & Logs
- **Build Failures**: Check the Render deployment logs. Ensure all libraries in `requirements.txt` are version-compatible.
- **Database Connection Errors**: Confirm the Environment Variables match your Aiven credentials exactly and that `ca.pem` is committed to the root of the GitHub repository.
- **Application Logs**: Live runtime logs can be monitored in the Render dashboard under **Events** and **Logs**.
