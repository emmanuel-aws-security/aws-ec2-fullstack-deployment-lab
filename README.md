# aws-ec2-fullstack-deployment-lab
Deployed a full-stack task management application on AWS EC2 using Node.js, Express, PostgreSQL, and Nginx. Configured the server, database, and reverse proxy to support creating, viewing, completing, and deleting tasks through a web interface.


# Cloud Lab: Deploying a Full-Stack Task Management Application on AWS EC2

## 🎯 Objective

The goal of this lab was to deploy a working web application on an AWS EC2 instance, configure communication between the frontend, backend, and database, and troubleshoot deployment issues.

This project demonstrates how to deploy a web application on a Linux server and verify that its components work together.

## 🏗️ Architecture

The application uses the following architecture:

```text
                User's Browser
                      |
                      v
                Nginx (Port 80)
                 /          \
                v            v
         Frontend Files    API Requests
         (HTML/CSS/JS)          |
                               v
                      Node.js + Express
                         (Port 3000)
                               |
                               v
                         PostgreSQL
                         (Port 5432)
```

**How it works:**

- **Nginx:** Serves the frontend and forwards `/api/` requests to the backend.
- **Frontend:** Provides a task management interface for creating, viewing, completing, and deleting tasks.
- **Node.js and Express:** Processes API requests and handles application logic.
- **PostgreSQL:** Stores task data persistently.

The backend listens on `127.0.0.1:3000`, and PostgreSQL remains accessible locally rather than being exposed publicly.

## 🛠️ Technologies Used

- AWS EC2
- Ubuntu Linux
- Nginx
- Node.js
- Express.js
- PostgreSQL
- HTML, CSS, and JavaScript
- Linux command line
- SSH and `curl`

## 🚀 Deployment Steps

### 1. Launch the EC2 Instance

- Launched an EC2 instance running Ubuntu.
- Connected to the instance over SSH using a private key.
- Configured the security group to allow SSH access from my IP and HTTP traffic on port 80.

### 2. Update the Server

Updated the package index and installed available system updates.

```bash
sudo apt update
sudo apt upgrade
```

### 3. Install and Configure Nginx

Installed Nginx and verified that its service was running.

```bash
sudo apt install nginx -y
sudo systemctl status nginx
```

Tested the web server by accessing the EC2 public IP in a browser.

### 4. Install Node.js and npm

Installed Node.js and npm to run the backend application.

```bash
sudo apt install nodejs npm -y

node --version
npm --version
```

### 5. Create the Backend Application

Created a project directory and initialized a Node.js application.

```bash
mkdir ~/task-app
cd ~/task-app

npm init -y
npm install express pg dotenv
```

The dependencies provide:

- `express` for the API.
- `pg` for connecting to PostgreSQL.
- `dotenv` for loading environment variables.

### 6. Install and Configure PostgreSQL

Installed PostgreSQL and verified that the database service was available.

```bash
sudo apt install postgresql postgresql-contrib -y
sudo systemctl status postgresql
```

Created a database named `taskapp` and a dedicated application user.

Created a `tasks` table with the following fields:

- `id` — unique task identifier.
- `title` — task title.
- `description` — optional task description.
- `completed` — completion status.
- `created_at` — task creation timestamp.

Granted the application user the required database and table permissions.

### 7. Configure Environment Variables

Created a `.env` file containing the database connection settings.

```dotenv
DB_HOST=127.0.0.1
DB_PORT=5432
DB_NAME=taskapp
DB_USER=taskapp_user
DB_PASSWORD=YOUR_DATABASE_PASSWORD
```

Restricted access to the environment file and added `.env` and `node_modules/` to `.gitignore`.

**Security note:** Never commit real passwords, private keys, or other secrets to GitHub.

### 8. Build the Backend API

Created `server.js` using Express and the PostgreSQL connection pool.

Implemented the following endpoints:

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/health` | Checks database connectivity |
| GET | `/api/tasks` | Retrieves tasks |
| POST | `/api/tasks` | Creates a task |
| PATCH | `/api/tasks/:id` | Updates completion status |
| DELETE | `/api/tasks/:id` | Deletes a task |

The API includes basic input validation and error handling.

### 9. Configure Nginx as a Reverse Proxy

Updated the Nginx site configuration to forward API requests to the Node.js backend.

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

Validated and reloaded the Nginx configuration.

```bash
sudo nginx -t
sudo systemctl reload nginx
```

This allowed the frontend and API to be accessed through the same public HTTP endpoint.

### 10. Test the Backend API

Started the backend application.

```bash
cd ~/task-app
node server.js
```

Checked the API health endpoint:

```bash
curl http://127.0.0.1:3000/api/health
```

Tested task creation:

```bash
curl -X POST http://127.0.0.1:3000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"My first deployed task","description":"Testing my EC2 application"}'
```

Retrieved the saved tasks:

```bash
curl http://127.0.0.1:3000/api/tasks
```

Verified the API through Nginx:

```bash
curl http://127.0.0.1/api/tasks
```

Confirmed that the created task was returned successfully.

### 11. Deploy the Frontend

Created the frontend at:

```text
/var/www/html/index.html
```

Implemented a responsive task management interface using HTML, CSS, and JavaScript.

The frontend communicates with the backend through the `/api/` endpoints and displays tasks retrieved from PostgreSQL.

Opened the EC2 public IP in a browser and verified that the application loaded and displayed the saved task.

## 🔧 Troubleshooting Experience

### Issue: Task API returned `Cannot GET /api/tasks`

While testing the backend, the task endpoint returned an Express `404` response even though the route existed in the saved `server.js` file.

**Investigation performed:**

1. Checked the saved backend source code.
2. Confirmed that the task route was present.
3. Inspected running Node.js processes using `ps`.
4. Checked which process was listening on port 3000 using `ss`.
5. Compared the live health endpoint response with the expected response from the saved source code.

**Root cause:** An outdated Node.js process was still serving the application.

**Resolution:** Stopped the old process, confirmed that port 3000 was free, and restarted the backend using the updated source code.

**Verification:** The task endpoint subsequently returned the expected JSON response, and task creation and retrieval worked successfully.

This demonstrated the importance of checking running processes, comparing live responses with application code, and verifying the fix instead of repeatedly changing configuration.

## ✅ Results

- Deployed a full-stack web application on AWS EC2.
- Configured Nginx to serve the frontend and proxy API requests.
- Connected an Express backend to PostgreSQL.
- Implemented and tested CRUD operations for tasks.
- Verified API connectivity both directly and through Nginx.
- Investigated and resolved a stale backend process.
- Confirmed that the application worked through a browser.

## 💼 Practical Business Value

This lab demonstrates the foundational skills required to deploy and troubleshoot web applications on cloud infrastructure.

The same troubleshooting principles can help investigate common production problems, including unavailable backend services, failed database connections, reverse-proxy errors, and outdated application processes.

## 🔮 Future Improvements

- Configure `systemd` to manage the backend process.
- Add HTTPS using a TLS certificate.
- Containerize the application using Docker Compose.
- Automate infrastructure provisioning with Terraform.
- Build a CI/CD pipeline for repeatable deployments.
- Introduce monitoring, centralized logs, and deliberate failure scenarios.

## 📚 Key Lessons Learned

- Understand how requests flow between the frontend, reverse proxy, backend, and database.
- Verify each component independently before testing the complete application.
- Use logs, process inspection, and HTTP requests to investigate failures.
- Keep application secrets out of source control.
- Prefer repeatable and automated deployment procedures over manual configuration as systems grow.
