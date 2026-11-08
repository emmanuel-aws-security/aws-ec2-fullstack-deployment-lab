# AWS EC2 Release-Based Deployment Using Git, Nginx and systemd

## About This Lab

This lab demonstrates how to organise and deploy a Node.js application on an AWS EC2 instance using a release-based deployment structure.

Instead of deploying new application code directly over the existing live application, each release is stored in its own directory, identified by its Git commit ID. A symbolic link named `current` points to the release being used by the application.

The application is served through Nginx, while systemd manages the Node.js backend service. PostgreSQL provides the application's database.

The main purpose of this lab was to understand how application releases can be organised on a Linux server, how services use those releases, and how Nginx connects the frontend and backend.

## What This Lab Covers

- Connecting an EC2 instance to a GitHub repository using an SSH deploy key.
- Retrieving the latest repository changes using `git fetch`.
- Creating commit-specific release directories.
- Extracting application code into a release directory using `git archive`.
- Installing application dependencies using `npm ci`.
- Using a symbolic link to identify the active release.
- Configuring systemd to run the Node.js application from the active release.
- Configuring Nginx to serve the frontend and forward API requests to the backend.
- Troubleshooting Linux permissions and systemd service configuration.

## Application Architecture

The application uses the following architecture:

```text
                 Browser
                    |
                    v
               Nginx :80
                    |
          +---------+---------+
          |                   |
          v                   v
    Frontend Files       /api/ Requests
                              |
                              v
                       Node.js :3000
                              |
                              v
                         PostgreSQL
```

The frontend is served by Nginx from:

`/home/ubuntu/task-app/current`

API requests are forwarded by Nginx to the Node.js backend listening on:

`127.0.0.1:3000`

The backend communicates with PostgreSQL to store and retrieve application data.

## Release Directory Structure

The deployment directory was organised as follows:

```text
/home/ubuntu/task-app/
│
├── .env
│
├── releases/
│   └── <commit-id>/
│       ├── server.js
│       ├── index.html
│       ├── package.json
│       └── package-lock.json
│
└── current -> releases/<commit-id>
```

Each release has a separate directory named after its Git commit ID.

The `current` symbolic link points to the release that the application is currently using.

## Deployment Steps

### 1. Connect EC2 to GitHub

An SSH deploy key was generated on the EC2 instance and its public key was added to the GitHub repository's Deploy Keys.

This allowed the EC2 instance to retrieve repository code securely without using personal GitHub credentials.

### 2. Retrieve the Latest Repository Changes

The following command was used:

```bash
git fetch origin
```

This retrieved the latest changes from GitHub without directly modifying the existing working files on the EC2 instance.

### 3. Create a Release Directory

The latest commit ID was retrieved:

```bash
RELEASE_ID=$(git rev-parse origin/main)
```

A directory was created for that specific release:

```bash
mkdir -p "/home/ubuntu/task-app/releases/$RELEASE_ID"
```

### 4. Extract the Application Code

The code from the fetched commit was extracted into the new release directory:

```bash
git archive origin/main | tar -x -C "/home/ubuntu/task-app/releases/$RELEASE_ID"
```

This allowed the application code to be placed in a separate release directory without overwriting the existing working tree.

### 5. Install Application Dependencies

Inside the new release directory, dependencies were installed using:

```bash
npm ci
```

This installs dependencies based on the project's package-lock.json file.

### 6. Create the Current Symbolic Link

The `current` symbolic link was created to point to the new release:

```bash
cd /home/ubuntu/task-app

ln -s "releases/$RELEASE_ID" current
```

This established a consistent path for services and configurations to use, regardless of the actual release directory name.

### 7. Configure the systemd Service

The systemd service file was updated at:

`/etc/systemd/system/taskapp.service`

The service was configured to use the `current` directory as its working directory and application entry point.

The final service configuration was:

```ini
[Unit]
Description=Task App Node.js Backend
After=network.target postgresql.service

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/task-app/current
ExecStart=/usr/bin/node /home/ubuntu/task-app/current/server.js
Restart=always
RestartSec=5
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

The service configuration was reloaded and the application restarted:

```bash
sudo systemctl daemon-reload
sudo systemctl restart taskapp
```

### 8. Configure Nginx

The Nginx configuration was updated so that its frontend document root pointed to:

`/home/ubuntu/task-app/current`

The existing API reverse proxy configuration continued forwarding requests to:

`http://127.0.0.1:3000`

Nginx was then reloaded to apply the configuration:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### 9. Verify the Application

The frontend was checked through the browser.

The backend and database connection were tested using:

```bash
curl -i http://localhost/api/health
```

The expected healthy response was:

```json
{
  "status": "healthy",
  "database": "connected"
}
```

This confirmed that the backend was running and could successfully communicate with PostgreSQL through the health endpoint.

## Problems Encountered and How They Were Resolved

### 1. Nginx Permission Denied

After changing the Nginx document root to the application directory, the frontend initially returned an error.

The Nginx error log showed:

```text
Permission denied
```

The problem was caused by the permissions on `/home/ubuntu`, which prevented the Nginx worker process from traversing that directory.

It was resolved with:

```bash
sudo chmod o+x /home/ubuntu
```

This granted other users directory traversal permission without granting directory listing or file-reading permission by itself.

### 2. Backend Service Failed to Start

The backend returned a `502 Bad Gateway` response.

Checking the systemd journal revealed that the application was trying to load:

```text
server.jsRestart=always
```

The cause was an incorrectly edited systemd service file where the `Restart=always` setting had accidentally been joined to the `ExecStart` command.

The service file was corrected, then systemd was reloaded and the service restarted.

### 3. Backend Health Check Returned 502

The 502 response occurred because the Node.js backend service had failed to start.

Once the systemd configuration was corrected and the backend restarted successfully, the health endpoint worked again.

## Final Outcome

The application was successfully configured to use a release-based directory structure on AWS EC2.

The frontend was served through Nginx, the backend was managed by systemd, and the backend successfully connected to PostgreSQL.

The lab demonstrated how Git, Linux symbolic links, systemd and Nginx work together to support a more organised application deployment structure.

## Scope and Limitations

This lab established the release directory structure and configured the application to run through the `current` symbolic link.

It did not implement a fully automated deployment pipeline, automatic release switching, or a tested rollback procedure. Those are separate improvements that can be explored in a future lab.

## Technologies Used

- AWS EC2
- Ubuntu Linux
- Git and GitHub
- Node.js
- Express.js
- PostgreSQL
- Nginx
- systemd
- SSH
- Bash
