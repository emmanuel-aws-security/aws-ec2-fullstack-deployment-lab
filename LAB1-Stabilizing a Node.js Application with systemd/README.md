# Lab 1 — Stabilizing a Node.js Application with systemd

## Objective

Configure a Node.js task management application on AWS EC2 to run as a managed Linux service, restart automatically after a crash, and be enabled to start when the server boots.

## Business Problem

When an application is started manually in a terminal, it may stop running if the process crashes or the terminal session ends. This can cause downtime and interrupt users.

This lab addresses that problem by using `systemd` to manage the application's lifecycle and recover automatically from unexpected failures.

## Environment

- **Cloud Provider:** AWS
- **Compute:** Amazon EC2
- **Operating System:** Ubuntu
- **Application Runtime:** Node.js
- **Database:** PostgreSQL
- **Web Server / Reverse Proxy:** Nginx
- **Service Manager:** systemd

## Architecture

```text
User / Browser
      |
      v
    Nginx
      |
      v
Node.js Backend
127.0.0.1:3000
      |
      v
  PostgreSQL
```

systemd manages the Node.js backend and monitors its process.

## Implementation

### 1. Created the systemd service

Created the service configuration at:

`/etc/systemd/system/taskapp.service`

```ini
[Unit]
Description=Task App Node.js Backend
After=network.target postgresql.service

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/task-app
ExecStart=/usr/bin/node /home/ubuntu/task-app/server.js
Restart=always
RestartSec=5
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

### 2. Reloaded systemd and enabled the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable taskapp
```

- `daemon-reload` loads the new service configuration.
- `enable` configures the service to start automatically during boot.

### 3. Started and verified the service

```bash
sudo systemctl start taskapp
sudo systemctl status taskapp
```

Confirmed that the service reached `active (running)`.

### 4. Verified backend and database connectivity

```bash
curl http://127.0.0.1:3000/api/health
```

**Result:**

```json
{"status":"healthy","database":"connected"}
```

This confirmed that the backend was responding and could connect to PostgreSQL.

### 5. Verified the Nginx reverse proxy

```bash
curl http://127.0.0.1/api/health
```

**Result:**

```json
{"status":"healthy","database":"connected"}
```

This confirmed that the health endpoint was accessible through Nginx.

### 6. Tested automatic crash recovery

Deliberately terminated the service process:

```bash
sudo systemctl kill --signal=SIGKILL taskapp
```

Then checked the service:

```bash
sudo systemctl status taskapp
```

The service logs showed that the main process had been killed, systemd scheduled a restart, and the application started again.

The service returned to `active (running)`, confirming that `Restart=always` was working.

Finally, checked the backend health endpoint again and confirmed that the application remained healthy with the database connected.

### 7. Verified boot-time configuration

```bash
sudo systemctl is-enabled taskapp
```

**Expected result:**

```text
enabled
```

This confirms that the service is configured to start automatically when the server boots. A separate reboot test is still needed to verify boot-time startup in practice.

## Evidence and Screenshots

The following screenshots document the lab:

| Screenshot | Evidence |
|---|---|
| `01-initial-service-status.png` | Initial check before the service was configured |
| `02-systemd-service-enabled.png` | Service enabled for boot |
| `03-taskapp-systemd-running.png` | Service running under systemd |
| `04-backend-health-database-connected.png` | Backend and database health check |
| `05-nginx-reverse-proxy-health-check.png` | Health check through Nginx |
| `06-systemd-automatic-restart.png` | Automatic recovery after the process was killed |

## Results

- The Node.js backend runs as a managed systemd service.
- The service is enabled for automatic startup at boot.
- The backend health endpoint reports a healthy application and connected database.
- Nginx successfully forwards requests to the backend.
- systemd automatically restarts the application after an unexpected process termination.
- The backend remains healthy after recovery.

## Business Value

This configuration reduces the need for manual intervention when the application process crashes. It improves service reliability and provides a foundation for more dependable deployments.

For a business owner, this means fewer interruptions caused by a stopped backend and less time spent manually restarting the application.

**Note:** Automatic process recovery does not guarantee zero downtime or protect against every possible server, database, or infrastructure failure.

## Skills Demonstrated

- AWS EC2 administration
- Linux service management
- systemd configuration
- Node.js process management
- PostgreSQL connectivity verification
- Nginx reverse-proxy testing
- Application health checks
- Failure simulation and recovery validation
- Technical documentation

## Next Steps

- Verify startup after an actual EC2 reboot.
- Automate application deployments using GitHub Actions.
- Provision infrastructure using Terraform.
- Containerize the application using Docker and Docker Compose.
- Perform controlled troubleshooting and failure-testing exercises.
