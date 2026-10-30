# Building a Basic CI Pipeline with GitHub Actions

## Project Overview

This project demonstrates how to set up a basic Continuous Integration (CI) pipeline for a Node.js task management application using Git, GitHub, and GitHub Actions.

The objective was to understand how application code can be managed with Git, stored in a GitHub repository, and automatically checked whenever changes are pushed to the main branch.

The pipeline uses a GitHub-hosted Ubuntu runner to retrieve the repository code and verify that essential application files are present.

## Project Objectives

- Understand Git version control and repository management.
- Connect a local Git repository to GitHub.
- Use branches, commits, and remote repositories.
- Configure GitHub Actions to automate basic checks.
- Trigger a workflow automatically when code is pushed to the main branch.
- Understand the role of CI in a software development workflow.

## Technologies Used

- Git
- GitHub
- GitHub Actions
- Ubuntu
- Node.js
- YAML

## Application Overview

The CI pipeline was configured for an existing task management application built with:

- Node.js
- Express.js
- PostgreSQL
- HTML, CSS and JavaScript
- Nginx

The application itself was previously deployed on an AWS EC2 Ubuntu instance.

## Implementation Steps

### 1. Installed Git

Installed Git on the Ubuntu EC2 server using the APT package manager and verified the installation.

### 2. Initialised the Git Repository

Navigated to the task application directory and ran `git init` to initialise a local Git repository.

### 3. Configured .gitignore

Created a `.gitignore` file to prevent sensitive and unnecessary files from being committed.

Excluded:
- `.env`
- `node_modules/`

### 4. Configured the Main Branch

Renamed the initial branch to `main` using Git branch management commands.

### 5. Connected the Local Repository to GitHub

Added the GitHub repository as a remote named `origin`, allowing Git to communicate with the remote repository.

### 6. Staged and Committed Application Files

Used `git add` to stage eligible application files and `git commit` to record changes in the local Git history.

### 7. Pushed Code to GitHub

Pushed the committed code to the remote `main` branch.

GitHub authentication was configured using a Personal Access Token with the necessary repository and workflow permissions.

### 8. Created the GitHub Actions Workflow

Created the workflow file:

`.github/workflows/ci.yml`

The workflow was configured to run when code is pushed to the `main` branch. It also supports manual execution through `workflow_dispatch`.

### 9. Configured the CI Job

The workflow uses a GitHub-hosted Ubuntu runner.

It performs two main steps:

1. Checks out the repository using `actions/checkout@v4`.
2. Verifies that the following application files exist:
   - `server.js`
   - `package.json`
   - `package-lock.json`
   - `index.html`

The workflow prints a success message when all required files are present.

## CI Workflow

The pipeline follows this process:

1. Code is pushed to GitHub.
2. GitHub detects the push to `main`.
3. GitHub Actions starts a workflow run.
4. A GitHub-hosted Ubuntu runner is provisioned.
5. The repository code is checked out.
6. Required application files are validated.
7. The workflow reports success or failure.

## Results

The initial CI workflow executed successfully and returned a successful status in GitHub Actions.

The pipeline confirmed that the required application files were present in the repository.

## What I Learned

- Git is a version control system used to track changes and maintain project history.
- GitHub hosts remote Git repositories.
- Git branches allow developers to work on separate lines of development.
- Git commits record snapshots of selected changes.
- Git push transfers committed changes to a remote repository.
- GitHub Actions automates tasks based on repository events.
- A workflow defines when automation runs and what tasks it performs.
- A GitHub-hosted runner provides an environment for executing CI jobs.
- CI can help identify problems before changes progress further through a development workflow.
- The application server and the GitHub Actions runner are separate environments.

## Current Limitations

This is an initial CI implementation.

The workflow currently checks for the presence of required application files. It does not yet:

- Install dependencies and run automated application tests.
- Validate database connectivity.
- Test the complete application runtime.
- Deploy application changes to the EC2 server.

Continuous Deployment (CD) has not been configured as part of this project stage.

## Future Improvements

- Extend CI to install dependencies.
- Add automated tests.
- Introduce application validation and build checks where appropriate.
- Implement a separate secure deployment workflow for the EC2 environment.

## Conclusion

This project provided practical experience with Git version control, GitHub repository management, and GitHub Actions.

It established a basic CI workflow that automatically validates essential application files whenever changes are pushed to the main branch.
