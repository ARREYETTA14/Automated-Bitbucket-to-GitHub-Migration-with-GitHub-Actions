# Automated-Bitbucket-to-GitHub-Migration-with-GitHub-Actions
Managing repositories across different version control platforms can be time-consuming, especially when transitioning from Bitbucket to GitHub. This project provides a fully automated solution using GitHub Actions to seamlessly clone a Bitbucket repository and push it to GitHub while preserving the entire commit history, branches, and tags

# 1. Prepare Your Bitbucket Repository

**Content of Bitbucket Repository**

![Image](https://github.com/user-attachments/assets/908ba8ca-0ec4-45e5-a251-984bd9743f3d)

**1.1: Find Your Bitbucket Username**
- Login to your Bitbucket account, go to the repo you wish to  migrate
- Copy the **repo name**
- In the top-right corner, click the **setting** icon > **personal bitbucket** setting
- Copy your username under **Username** in the settings and store it.

![Image](https://github.com/user-attachments/assets/299475ba-c584-43f4-8edd-9b588c086fde)

**1.2: Generate a Bitbucket App Password**
- While still at the setting section of bitbucket, On the left sidebar, scroll down to **Access management** and click **App passwords**.

![Image](https://github.com/user-attachments/assets/afb15ec7-7178-4a16-a8dd-6a4187760a86)

- Click **Create app password**:
  - Give it a label (e.g., ```GitHubMigration```).
  - Select these permissions:
    - **Account**: write.
    - **Workspace Membership**: write.
    - **Repositories**: admin.
    - **Pull requests**: write.
- Click **Create**.

![Image](https://github.com/user-attachments/assets/bdc20e46-30f9-41fa-9d86-83b818422909)

- **Copy the app password** that appears. **You won’t see it again** once you leave the page.

![Image](https://github.com/user-attachments/assets/40cf29ff-554a-4d6c-ba55-f91b01264616)

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

![Image](https://github.com/user-attachments/assets/1bd61dea-7ff7-49b6-b51f-0bb15153f0a4)

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

  - **Name**: GITHUB_USERNAME
    **Value**: Your Github username.
     
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
  workflow_dispatch:  # Allows manual triggering of the workflow

jobs:
  migrate:
    runs-on: ubuntu-latest  # Specifies the runner environment

    steps:
    - name: Checkout Bitbucket Repository
      run: |
        echo "Cloning Bitbucket repository..."
        git clone https://arreyettaaugustine@bitbucket.org/bitbucket-migration-github/migration-repo.git bitbucket-repo  # Clone the Bitbucket repository into a directory named 'bitbucket-repo'
      env:
        BITBUCKET_USERNAME: ${{ secrets.BITBUCKET_USERNAME }}  # Uses Bitbucket username from GitHub secrets
        BITBUCKET_APP_PASSWORD: ${{ secrets.BITBUCKET_APP_PASSWORD }}  # Uses Bitbucket app password from GitHub secrets

    - name: Push to GitHub Repository (Merge Approach)
      run: |
        cd bitbucket-repo || (echo "Error: Cloned directory 'bitbucket-repo' not found" && exit 1)  # Navigate to the cloned repository, exit if not found

        echo "Setting up Git identity..."
        git config --global user.email "github-actions@github.com"  # Set email for Git commits
        git config --global user.name "GitHub Actions"  # Set name for Git commits

        echo "Setting up GitHub remote..."
        git remote remove github || true  # Remove existing 'github' remote if it exists (ignoring errors)
        git remote add github https://${{ secrets.GITHB_TOKEN }}@github.com/ARREYETTA14/Automated-Bitbucket-to-GitHub-Migration-with-GitHub-Actions.git  # Add GitHub remote with authentication

        echo "Fetching latest changes from GitHub..."
        git fetch github  # Fetch latest changes from GitHub

        echo "Checking out the main branch..."
        git checkout main || git checkout -b main  # Switch to the main branch, or create it if it doesn't exist

        echo "Merging GitHub changes with Bitbucket repo..."
        git merge github/main --allow-unrelated-histories -m "Merging GitHub and Bitbucket histories" || (echo "Merge conflict detected. Resolving automatically..." && git merge --abort && exit 1)  # Merge GitHub's main branch with Bitbucket's, handle conflicts if any

        echo "Pushing merged changes to GitHub..."
        git push github main || (echo "Failed to push changes" && exit 1)  # Push the merged main branch to GitHub

        echo "Pushing all branches to GitHub..."
        git push github --all || (echo "Failed to push branches" && exit 1)  # Push all branches to GitHub

        echo "Pushing all tags to GitHub..."
        git push github --tags || (echo "Failed to push tags" && exit 1)  # Push all tags to GitHub
      env:
        GITHB_TOKEN: ${{ secrets.GITHB_TOKEN }}  # Uses GitHub token from GitHub secrets

```

**Note** 
- This code seems a little complex because the repo I was cloning already had some data. I had to add some lines to avoid issues with the repository not being merged or to avoid "forcefully" pushing the data to the repo, which would warrant cleaning all data in the present repo. In case your Repo is empty, this code will still work, or you will not need a code as complex as mine. Something less complex will do the job.
- Lines ```**14**``` and ```**29**``` should be substituted with the actual ```Bitbucket repository URL``` and the ```Github repository URL``` respectively. For line ```29```. Make sure to substitute at the appropriate section. Copy your **Github URL** from **github.com** and subtitude the section of the line from **github.com**.

# Step 4: Run the Workflow
- Go to the **Actions** tab in your GitHub repository.
- Click **Migrate from Bitbucket to GitHub** in the list of workflows.
- Click **Run workflow** (on the right side of the page).
- The workflow will start. You can monitor its progress in the logs.
- A successful run will show as thus:

![Image](https://github.com/user-attachments/assets/4ea47482-cbf0-4de5-9600-7842cb3af700)

# Step 5: Verify the Migration
- Go to your GitHub repository’s Code tab.
  - Verify that all files, branches, and tags from Bitbucket are present in the Github Repository. The ```.gitgnore``` and the ```Testcode``` files.

![Image](https://github.com/user-attachments/assets/31920f69-dc61-4cd3-9a81-e09ac752fdc1)

- If something is missing, check the logs in the Actions tab for errors.






