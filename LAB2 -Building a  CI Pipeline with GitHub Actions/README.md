# Building a Basic CI Pipeline with GitHub Actions

## 1. Project Overview

This project demonstrates how to configure a basic Continuous Integration (CI) pipeline for a Node.js task management application using Git, GitHub and GitHub Actions.

The objective was to understand how application code can be managed with Git, stored in a remote GitHub repository, and automatically validated whenever changes are pushed to the `main` branch.

The pipeline uses a GitHub-hosted Ubuntu runner to retrieve the repository code and verify that essential application files are present.

## 2. Project Objectives

- Install and configure Git on an Ubuntu server.
- Initialise a Git repository for an existing application.
- Track application changes using Git.
- Connect a local repository to GitHub.
- Understand branches, staging, commits and remote repositories.
- Configure GitHub Actions to automate basic checks.
- Trigger a CI workflow automatically when code is pushed to `main`.
- Understand the relationship between Git, GitHub and GitHub Actions.

## 3. Technologies Used

- Git
- GitHub
- GitHub Actions
- Ubuntu
- Node.js
- YAML
- AWS EC2 (existing application hosting environment)

## 4. Application Overview

The CI pipeline was configured for an existing task management application built with:

- Node.js
- Express.js
- PostgreSQL
- HTML, CSS and JavaScript
- Nginx

The application was previously deployed on an AWS EC2 Ubuntu instance.

The application source code was organised in the `/home/ubuntu/task-app` directory.

## 5. Implementation Steps

### Step 1: Installing Git

Git was installed on the Ubuntu EC2 server using the APT package manager.

Commands used:

```bash
sudo apt update
sudo apt install git -y
git --version
```

**Explanation:**

- `sudo apt update` refreshes the package information available to Ubuntu.
- `sudo apt install git -y` installs Git.
- `git --version` verifies that Git was successfully installed.

**Purpose:** Install the version control software required to manage the application's source code.

### Step 2: Navigating to the Application Directory

Before initialising Git, we navigated to the task application directory.

```bash
cd /home/ubuntu/task-app
```

**Explanation:**

- `cd` means change directory.
- `/home/ubuntu/task-app` is the location containing the application files.

**Purpose:** Ensure Git manages the task application rather than unrelated files elsewhere on the server.

### Step 3: Initialising the Git Repository

We initialised Git inside the application directory.

```bash
git init
```

**Explanation:**

- `git init` creates a new local Git repository in the current directory.
- Git creates a hidden `.git` directory to store repository metadata, commit history and configuration.

We then checked the repository status:

```bash
git status
```

**Explanation:**

- `git status` displays the current state of the repository.
- It identifies untracked files, modified files and staged changes.

**Purpose:** Start version control for the application and identify the files Git can see.

### Step 4: Configuring `.gitignore`

We created a `.gitignore` file to prevent sensitive and unnecessary files from being included in Git commits.

File contents:

```gitignore
.env
node_modules/
```

**Explanation:**

- `.env` is excluded because it may contain sensitive environment variables and database credentials.
- `node_modules/` is excluded because Node.js dependencies can be installed again using the package manager.

**Purpose:** Reduce the risk of exposing sensitive information and avoid committing unnecessary dependency files.

### Step 5: Renaming the Default Branch

We renamed the initial branch to `main`.

```bash
git branch -M main
```

**Explanation:**

- `git branch` manages Git branches.
- `-M` renames the current branch, even if a branch with the destination name already exists.
- `main` becomes the primary branch name.

**Purpose:** Use `main` as the primary development branch and align with the common GitHub convention.

### Step 6: Connecting the Local Repository to GitHub

We connected the local repository to the GitHub repository using a remote named `origin`.

```bash
git remote add origin https://github.com/emmanuel-aws-security/aws-task-app-cicd.git
```

**Explanation:**

- `git remote` manages remote repository connections.
- `add` creates a new remote connection.
- `origin` is the nickname assigned to the remote repository.
- The URL identifies the GitHub repository.

We verified the remote configuration using:

```bash
git remote -v
```

**Purpose:** Tell Git where the remote GitHub repository is located.

Adding a remote does not upload any files. It only establishes the connection information.

### Step 7: Staging Application Files

We staged the eligible application files for the next commit.

```bash
git add .
```

**Explanation:**

- `git add` stages changes for the next commit.
- `.` represents the current directory and its eligible contents.
- Files excluded by `.gitignore` are not added as new untracked files.

**Purpose:** Select the application changes we wanted to record in Git history.

### Step 8: Committing Application Files

We recorded the staged changes as commits.

Examples of the commit messages used:

```bash
git commit -m "Add task app backend"
```

```bash
git commit -m "Add task app frontend"
```

**Explanation:**

- `git commit` records the staged changes in the local repository.
- `-m` allows us to provide a descriptive commit message.

**Purpose:** Create identifiable snapshots of the application's development history.

The backend, frontend and workflow were recorded through separate commits.

### Step 9: Pushing Code to GitHub

We pushed the local `main` branch to the remote GitHub repository.

```bash
git push -u origin main
```

**Explanation:**

- `git push` sends committed changes to a remote repository.
- `origin` identifies the remote repository.
- `main` identifies the destination branch.
- `-u` establishes the upstream tracking relationship between the local and remote branches.

GitHub authentication was configured using a Personal Access Token with the necessary permissions.

An initial push failed because the token lacked the required workflow permission. The token permissions were corrected before the successful push.

**Purpose:** Transfer the committed application code from the EC2 server's local repository to GitHub.

### Step 10: Creating the GitHub Actions Workflow

We created the workflow directory and file inside the repository:

```text
.github/
└── workflows/
    └── ci.yml
```

The workflow file was named `ci.yml`.

**Purpose:** Give GitHub Actions instructions on when to run automation and what tasks to perform.

### Step 11: Configuring the CI Workflow

The following YAML configuration was used:

```yaml
name: Task App CI

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  check-app:
    runs-on: ubuntu-latest

    steps:
      - name: Get the repository code
        uses: actions/checkout@v4

      - name: Check required application files
        run: |
          test -f server.js
          test -f package.json
          test -f package-lock.json
          test -f index.html
          echo "All required application files are present."
```

**Explanation of the configuration:**

**Workflow name**

```yaml
name: Task App CI
```

Defines the workflow's display name in GitHub Actions.

**Workflow triggers**

```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:
```

- `push` triggers the workflow when changes are pushed.
- `branches: - main` restricts the automatic push trigger to the `main` branch.
- `workflow_dispatch` allows manual execution from the GitHub Actions interface.

**Jobs**

```yaml
jobs:
  check-app:
```

Defines a job named `check-app`.

**Runner environment**

```yaml
runs-on: ubuntu-latest
```

Tells GitHub Actions to execute the job on a GitHub-hosted Ubuntu runner.

This runner is separate from our EC2 application server.

**First step: Checking out the repository**

```yaml
- name: Get the repository code
  uses: actions/checkout@v4
```

Uses the official checkout action to retrieve the repository code so the runner can access the application files.

**Second step: Validating application files**

```yaml
- name: Check required application files
  run: |
    test -f server.js
    test -f package.json
    test -f package-lock.json
    test -f index.html
    echo "All required application files are present."
```

The `test -f` commands check whether the specified files exist as regular files.

If a required file is missing, its check returns a failure and the job stops.

If all checks pass, the final `echo` command prints the success message.

**Purpose:** Automatically verify that essential application files are present whenever the workflow runs.

### Step 12: Verifying the CI Pipeline

After pushing the workflow to GitHub, we opened the Actions tab and checked the workflow execution.

The workflow completed successfully and displayed a green check mark.

**Purpose:** Confirm that GitHub detected the configured trigger, executed the job and completed the required file validation successfully.

## 6. CI Pipeline Execution Flow

The pipeline follows this sequence:

1. Developer pushes committed code to GitHub.
2. GitHub detects a push to the `main` branch.
3. GitHub Actions triggers the configured workflow.
4. A GitHub-hosted Ubuntu runner is provisioned.
5. The repository code is checked out.
6. The required application files are validated.
7. The workflow reports a successful or failed result.

## 7. Screenshot Evidence

All screenshots are stored in the `evidence/` directory.

### Evidence 1: GitHub Repository

**Filename:** `01-github-repository.png`

**How to capture:**

1. Open the GitHub repository.
2. Select the Code tab.
3. Ensure the repository name, `main` branch and application files are visible.
4. Capture the page.

**What it proves:**

- The remote GitHub repository exists.
- Application source files have been pushed.
- The repository contains the application code under the `main` branch.

### Evidence 2: CI Workflow Configuration

**Filename:** `02-ci-workflow-file.png`

**How to capture:**

1. Open the repository's Code tab.
2. Navigate to `.github/workflows/ci.yml`.
3. Open the file.
4. Capture the YAML configuration.

**What it proves:**

- The GitHub Actions workflow has been created.
- The workflow is configured to trigger on pushes to `main`.
- The workflow uses an Ubuntu runner.
- The job contains repository checkout and file validation steps.

### Evidence 3: Successful Workflow Execution

**Filename:** `03-ci-successful-run.png`

**How to capture:**

1. Open the Actions tab.
2. Select `Task App CI`.
3. Open the successful workflow run.
4. Capture the page showing the green successful status.

**What it proves:**

- GitHub Actions detected and executed the workflow.
- The workflow completed successfully.
- The CI pipeline reached a successful final status.

### Evidence 4: CI Job Execution Steps

**Filename:** `04-ci-job-steps.png`

**How to capture:**

1. Open the successful workflow run.
2. Select the `check-app` job.
3. Locate the job's step list.
4. Capture the checkout and validation steps with their execution statuses.

**What it proves:**

- The `check-app` job was executed.
- The repository checkout step ran.
- The required application file validation step ran.
- The job completed successfully.

### Evidence 5: Application File Validation Output

**Filename:** `05-ci-validation-output.png`

**How to capture:**

1. Open the `check-app` job.
2. Expand `Check required application files`.
3. Locate the command output.
4. Capture the successful validation message.

Expected output:

```text
All required application files are present.
```

**What it proves:**

- The validation commands executed.
- The required application files were found.
- The validation step completed successfully.

### Evidence 6: Git Commit History

**Filename:** `06-git-commit-history.png`

**How to capture:**

1. Return to the repository's Code tab.
2. Open the commit history.
3. Capture the list of commits and their messages.

Look for commits such as:

- Add task app backend
- Add task app frontend
- Add first GitHub Actions CI workflow

**What it proves:**

- Git was used to track project development.
- Application changes were recorded as commits.
- The CI workflow was introduced through version-controlled changes.
- The repository maintains a traceable development history.

## 8. Evidence Directory Structure

The final repository documentation should follow this structure:

```text
aws-task-app-cicd/
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── evidence/
│   ├── 01-github-repository.png
│   ├── 02-ci-workflow-file.png
│   ├── 03-ci-successful-run.png
│   ├── 04-ci-job-steps.png
│   ├── 05-ci-validation-output.png
│   └── 06-git-commit-history.png
│
├── server.js
├── index.html
├── package.json
├── package-lock.json
└── README.md
```

## 9. Results

The initial CI workflow executed successfully and returned a successful status in GitHub Actions.

The pipeline confirmed that the required application files were present in the repository.

This demonstrated how GitHub Actions can automatically perform checks whenever changes are pushed to a configured branch.

## 10. What I Learned

Through this project, I gained practical experience with:

- Installing Git on an Ubuntu server.
- Initialising a Git repository inside an existing application directory.
- Using `.gitignore` to exclude sensitive and unnecessary files.
- Understanding local and remote Git repositories.
- Creating and managing branches.
- Staging changes and recording commits.
- Connecting Git to GitHub using remote configuration.
- Pushing committed code to a remote repository.
- Understanding GitHub Actions workflow triggers.
- Writing workflow configuration using YAML.
- Using GitHub-hosted runners to execute automated jobs.
- Understanding the separation between an application server and a CI runner.
- Using automated checks to validate repository contents.

## 11. Current Limitations

This project represents an initial CI implementation.

The current workflow checks for the presence of required application files. It does not yet:

- Install application dependencies and run automated tests.
- Validate database connectivity.
- Test the complete application runtime.
- Build and package the application.
- Deploy application changes to the EC2 server.

Continuous Deployment (CD) has not been configured as part of this project.

## 12. Future Improvements

- Extend CI to install application dependencies.
- Add automated application tests.
- Introduce more meaningful application validation.
- Implement a separate secure deployment workflow for the EC2 environment.

## 13. Conclusion

This project provided practical experience with Git version control, GitHub repository management and GitHub Actions.

It established a basic CI workflow that automatically validates essential application files whenever changes are pushed to the `main` branch.

The project provides a foundation for introducing more comprehensive automated testing and eventually implementing Continuous Deployment.
