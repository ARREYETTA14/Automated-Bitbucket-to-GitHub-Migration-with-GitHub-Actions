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

# Step 3: Create the GitHub Actions Workflow
GitHub Actions is an automation tool that runs scripts. In this step, you will create a file in your GitHub repository to automate the migration process.

**3.1: Create the Workflow File**

- Go to your GitHub repository’s **Code** tab.

- Click **Add file** > **Create new file**.

- Name the file:

```bash
.github/workflows/migrate.yml
```
- Copy and paste this code into the file:

```yml
name: Migrate Bitbucket to GitHub

on:
  workflow_dispatch:

jobs:
  migrate:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Clone Bitbucket repository
      run: |
        echo "Cloning Bitbucket repository..."
        git clone https://${BITBUCKET_USERNAME}:${BITBUCKET_APP_PASSWORD}@bitbucket.org/${{ secrets.BITBUCKET_USERNAME }}/${{ secrets.BITBUCKET_REPO_NAME }}.git bitbucket-repo
      env:
        BITBUCKET_USERNAME: ${{ secrets.BITBUCKET_USERNAME }}
        BITBUCKET_APP_PASSWORD: ${{ secrets.BITBUCKET_APP_PASSWORD }}

    - name: Push to GitHub repository
      run: |
        echo "Pushing to GitHub repository..."
        cd bitbucket-repo
        git remote add github https://${{ secrets.GITHB_TOKEN }}@github.com/${{ secrets.GITHB_USERNAME }}/${{ secrets.GITHB_REPO_NAME }}.git
        git push github --all
        git push github --tags
      env:
        GITHUB_TOKEN: ${{ secrets.GITHB_TOKEN }}

```

**Note** 
- This code seems to be a little complex because the repo I was cloning had some data already so I had to add some lines to avoid issues with the repository not being merged or avoid ```forcefully``` pushing the data to the repo which will warrant cleaning all data in the present repo. In case your Repo is empty, this code will still work or you will not need a code as complex as mine. Something less complex will do the job.
- Lines ```14``` and ```29``` should be substituted with the actual ```Bitbucket repository URL``` and the ```Github repository URL``` respectively. For line ```29```. Make sure to substitute at the appropriate section

# Step 4: Run the Workflow
- Go to the **Actions** tab in your GitHub repository.
- Click **Migrate from Bitbucket to GitHub** in the list of workflows.
- Click **Run workflow** (on the right side of the page).
- The workflow will start. You can monitor its progress in the logs.

# Step 5: Verify the Migration
- Go to your GitHub repository’s Code tab.
  - Verify that all files, branches, and tags from Bitbucket are present.
- If something is missing, check the logs in the Actions tab for errors.






