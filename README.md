# Automated-Bitbucket-to-GitHub-Migration-with-GitHub-Actions
Managing repositories across different version control platforms can be time-consuming, especially when transitioning from Bitbucket to GitHub. This project provides a fully automated solution using GitHub Actions to seamlessly clone a Bitbucket repository and push it to GitHub while preserving the entire commit history, branches, and tags

# 1. Prepare Your Bitbucket Repository

**1.1: Find Your Bitbucket Username**
- Login to your Bitbucket account, go to the repo you wish to  migrate
- Copy the **repo name**
- In the top-right corner, click the **setting** icon > **personal bitbucket** setting
- Copy your username under **Username** in the settings and store it.

<image>

**1.2: Generate a Bitbucket App Password**
- While still at the setting section of bitbucket, On the left sidebar, scroll down to **Access management** and click **App passwords**.
<image>
- Click **Create app password**:
  - Give it a label (e.g., ```GitHubMigration```).
  - Select these permissions:
    - **Repository**: Read.
    - **Account**: Read.
- Click **Create**.
- **Copy the app password** that appears. **You won’t see it again** once you leave the page.

# 2. Prepare Your GitHub Repository

**2.1: Create a GitHub Repository**
- Log in to GitHub.
- Click the + icon in the top-right corner and select **New repository**.
- Fill in the details:
  - Repository name: Enter a name for your repo (e.g., ```migrated-repo```).
  - Description: Optional.
  - Visibility: Choose **Public** or **Private**.
  - **Do NOT initialize the repository with a README**.
- Click **Create repository**.
- 

**2.2: Generate a GitHub Personal Access Token**
 Personal Access Token (PAT) allows GitHub Actions to authenticate with your GitHub repository.

- In GitHub, click your **profile picture** in the top-right corner.
- Select **Settings**.
- Scroll down to **Developer settings** in the left menu and click it.
- Click **Personal Access Tokens > Tokens (classic)**.
- Click **Generate new token** (or **Generate new token (classic)** if applicable):
  - **Note**: If GitHub suggests using  **fine-grained tokens**, select it.
- Configure your token:
  - **Expiration**: Set an appropriate duration (e.g., 90 days).
  - **Permissions**:
    - **Repo**: Full control of private repositories.
- Click **Generate token**.
- Copy the token. **You won’t see it again after you leave the page**.

**2.3: Add Secrets in GitHub**
Now, store the sensitive data securely in your GitHub repository as secrets.

- Go to your new GitHub repository.

- Click **Settings > Secrets and variables > Actions**.

- Click **New repository secret** and add the following:

  - **Name**: BITBUCKET_USERNAME
    **Value**: Your Bitbucket username.
  
  - **Name**: BITBUCKET_APP_PASSWORD
    **Value**: Your Bitbucket App Password.

  - **Name**: BITBUCKET_REPO_NAME
    **Value**: Your Bitbucket repo name.
    
  - **Name**: GITHUB_USERNAME
    **Value**: Your Github username.
    
  - **Name**: GITHUB_REPO_NAME
    **Value**: Your Github repo name.
  
  - **Name**: GITHUB_TOKEN
    **Value**: Your GitHub Personal Access Token.


