 **Azure DevOps Scenario-based Questions** — each with:

- Scenario
- Question
- Solution (explained step-by-step)
- Code / YAML Example if needed  

I'll give you a mix covering **Pipelines**, **Repos**, **Artifacts**, **Test Plans**, and **Boards**.

---

# 🚀 Azure DevOps Scenario-Based Questions, Solutions & Code

---

### 1️⃣ Scenario: CI/CD Pipeline for .NET Core App

**Scenario:**  
You are developing a .NET Core web application and want to automate the build and deployment to Azure App Service whenever code is pushed to `main` branch.

**Question:**  
How will you design a CI/CD pipeline in Azure DevOps?

---

**Solution:**

✅ Steps:

1. Create a **Pipeline** in Azure DevOps.  
2. Use a **Service Connection** to connect to Azure.  
3. Set a trigger for **push on `main`**.  
4. Add tasks:
   - Restore packages
   - Build
   - Publish
   - Deploy to Azure App Service

---

**Code: `.azure-pipelines.yml`**

```yaml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '7.x'

- script: dotnet restore
  displayName: 'Restore Packages'

- script: dotnet build --configuration Release
  displayName: 'Build Project'

- script: dotnet publish --configuration Release --output $(Build.ArtifactStagingDirectory)
  displayName: 'Publish Artifacts'

- task: AzureWebApp@1
  inputs:
    azureSubscription: '<Your-Service-Connection-Name>'
    appName: '<Your-App-Service-Name>'
    package: '$(Build.ArtifactStagingDirectory)'
```

---

### 2️⃣ Scenario: Approval Before Production Deployment

**Scenario:**  
You want deployments to production to happen **only after manual approval** by a manager.

**Question:**  
How can you enforce approval in Azure DevOps Pipeline?

---

**Solution:**

✅ Steps:

1. In the **Release Pipeline** → Stages → Add an **Approval** before the "Production" stage.
2. Assign the manager's name/email for approval.

✅ Alternative (in YAML):  
Use **Environments** + **Checks**.

---

**Code (YAML):**

```yaml
stages:
- stage: Production
  jobs:
  - deployment: DeployToProd
    environment: 'Production'  # <-- Environment that requires approval
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo Deploying to Production!
```

✅ In Azure DevOps:
- Go to **Environments > Production > Approvals and Checks**  
- Add **Manual Approval** → assign approvers.

---

### 3️⃣ Scenario: Pull Request Validation

**Scenario:**  
You want to ensure that **PRs (Pull Requests)** to the `main` branch pass **build** and **unit tests** before merging.

**Question:**  
How do you set this up?

---

**Solution:**

✅ Steps:

1. Create a **Pipeline** that builds and tests.  
2. Enable **Build Validation** in **Branch Policies** for `main`.

---

**Code: `.azure-pipelines.yml`**

```yaml
trigger: none
pr:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: DotNetCoreCLI@2
  inputs:
    command: 'build'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  inputs:
    command: 'test'
    projects: '**/*Tests/*.csproj'
```

✅ Also:
- Go to **Repos > Branches > main > Branch Policies**.
- Set **Build Validation** with the above pipeline.

---

### 4️⃣ Scenario: Reuse Common Templates Across Multiple Pipelines

**Scenario:**  
You have 10+ projects. You want a **common build template** used by all instead of repeating code.

**Question:**  
How will you achieve this?

---

**Solution:**

✅ Steps:

1. Create a **shared YAML template** in a repo.  
2. In pipelines, use `template` keyword to **import**.

---

**Code:**

**`templates/build.yml`**

```yaml
parameters:
  - name: projectPath
    type: string

steps:
- task: DotNetCoreCLI@2
  inputs:
    command: 'build'
    projects: '${{ parameters.projectPath }}'
```

**`azure-pipelines.yml`**

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- template: templates/build.yml
  parameters:
    projectPath: '**/*.csproj'
```

---

### 5️⃣ Scenario: Deploy to Multiple Environments (Dev → QA → Prod)

**Scenario:**  
You want to deploy an app automatically:
- First to **Dev** after build
- Then to **QA** after manual approval
- Finally to **Prod** after another approval

**Question:**  
How to design this multi-stage pipeline?

---

**Solution:**

✅ Steps:

1. Define **stages** for each environment.  
2. Use **manual approval** before QA and Prod.

---

**Code: `.azure-pipelines.yml`**

```yaml
trigger:
- main

stages:
- stage: Dev
  jobs:
  - job: DeployDev
    steps:
    - script: echo "Deploying to Dev"

- stage: QA
  dependsOn: Dev
  jobs:
  - job: DeployQA
    steps:
    - script: echo "Deploying to QA"

- stage: Prod
  dependsOn: QA
  jobs:
  - job: DeployProd
    steps:
    - script: echo "Deploying to Production"
```

✅ Then, configure **Pre-deployment Approvals** manually in the pipeline UI (for QA and Prod stages).

---

# ✨ Quick Topics Also Covered
| Topic | Short Tip |
|:----|:----|
| Secrets in pipeline | Use Azure Key Vault or pipeline secrets |
| Rollback if deployment fails | Enable automatic rollback with Deployment Groups |
| Artifact publishing | `publish: $(Build.ArtifactStagingDirectory)` |
| Parallel jobs | Use `dependsOn` and `matrix` strategy in YAML |
| Pipeline caching | Use `Cache@2` task for speeding builds |

---

# ✅ Summary
- Use **Triggers** smartly (push vs PR)
- Automate builds/tests/deployments
- Use **Environments** + **Checks** for approvals
- Reuse **Templates** for multiple pipelines
- Setup **multi-stage** flow for smooth releases

---
