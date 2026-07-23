# Month 4: CI/CD Pipelines — Comprehensive Study Material

> **Target Market:** GCC (Gulf Cooperation Council) — Heavy Azure DevOps usage  
> **Program:** DevOps Upskilling  
> **Last Updated:** July 2026

---

## Table of Contents

1. [CI/CD Fundamentals](#section-1-cicd-fundamentals)
2. [Azure DevOps (Primary — GCC Market Focus)](#section-2-azure-devops-primary--gcc-market-focus)
3. [GitLab CI/CD](#section-3-gitlab-cicd)
4. [Jenkins](#section-4-jenkins)
5. [GitHub Actions](#section-5-github-actions)
6. [CI/CD Best Practices](#section-6-cicd-best-practices)
7. [Lessons Learned / Pro Tips](#section-7-lessons-learned--pro-tips)
8. [Interview Questions](#section-8-interview-questions)
9. [Free Resources](#section-9-free-resources-with-links)

---

## Section 1: CI/CD Fundamentals

### What is CI (Continuous Integration)?

**Continuous Integration** is the practice of frequently merging developer code changes into a shared repository (multiple times per day), where automated builds and tests verify each integration.

**Key Principles:**
- Developers commit to mainline at least once per day
- Every commit triggers an automated build
- Build failures are fixed immediately (within 10 minutes)
- Tests run automatically on every integration

### What is CD?

**CD** has two meanings:

| Term | Definition | Automation Level |
|------|-----------|-----------------|
| **Continuous Delivery** | Code is always in a deployable state; deployment to production requires manual approval | Automated up to staging |
| **Continuous Deployment** | Every change that passes all tests is automatically deployed to production | Fully automated |

```
Continuous Integration:
  Code → Build → Unit Tests → Integration Tests

Continuous Delivery (adds):
  ... → Deploy to Staging → Manual Gate → Deploy to Prod

Continuous Deployment (adds):
  ... → Deploy to Staging → Auto Deploy to Prod
```

### Why CI/CD Matters

**Benefits:**
- **Faster time to market** — Release features in hours instead of months
- **Reduced risk** — Small, frequent changes are easier to debug
- **Higher quality** — Automated testing catches bugs early
- **Developer productivity** — Less time on manual processes
- **Consistent deployments** — Eliminate "works on my machine" issues
- **Faster feedback loops** — Know within minutes if code breaks something

**Key Metrics (DORA Metrics):**

| Metric | Elite | High | Medium | Low |
|--------|-------|------|--------|-----|
| Deployment Frequency | On-demand (multiple/day) | Weekly-Monthly | Monthly-Biannually | < once per 6 months |
| Lead Time for Changes | < 1 hour | 1 day - 1 week | 1 week - 1 month | > 1 month |
| Change Failure Rate | 0-15% | 16-30% | 16-30% | > 30% |
| Time to Restore Service | < 1 hour | < 1 day | 1 day - 1 week | > 1 week |

### Pipeline Anatomy

```
┌─────────────────────────────────────────────────────────────┐
│                        PIPELINE                              │
├─────────────────────────────────────────────────────────────┤
│  TRIGGER (push, PR, schedule, manual)                       │
├──────────┬──────────┬──────────────┬───────────────────────┤
│  STAGE 1 │  STAGE 2 │   STAGE 3    │      STAGE 4          │
│  Build   │  Test    │   Security   │      Deploy           │
├──────────┼──────────┼──────────────┼───────────────────────┤
│  Job 1   │  Job 1   │   Job 1      │      Job 1            │
│  ├─Step1 │  ├─Unit  │   ├─SAST     │      ├─Deploy Staging │
│  ├─Step2 │  ├─Integ │   ├─DAST     │      ├─Smoke Tests    │
│  └─Step3 │  └─E2E   │   └─SCA      │      └─Deploy Prod    │
└──────────┴──────────┴──────────────┴───────────────────────┘
```

**Components:**
- **Trigger** — What starts the pipeline (code push, PR, schedule, webhook)
- **Stage** — Logical grouping of jobs (Build, Test, Deploy)
- **Job** — Unit of work that runs on an agent/runner
- **Step** — Individual command or task within a job

### Deployment Strategies

#### 1. Rolling Update

Gradually replaces instances of the old version with the new version.

```
Time 0:  [v1] [v1] [v1] [v1] [v1]    ← All running v1
Time 1:  [v2] [v1] [v1] [v1] [v1]    ← 1 instance updated
Time 2:  [v2] [v2] [v1] [v1] [v1]    ← 2 instances updated
Time 3:  [v2] [v2] [v2] [v1] [v1]    ← 3 instances updated
Time 4:  [v2] [v2] [v2] [v2] [v1]    ← 4 instances updated
Time 5:  [v2] [v2] [v2] [v2] [v2]    ← All running v2
```

**Pros:** Zero downtime, gradual rollout, resource-efficient  
**Cons:** Slow rollback, mixed versions during deployment, complex session handling

#### 2. Blue-Green Deployment

Maintains two identical production environments; switch traffic atomically.

```
                    ┌─────────────┐
                    │   Router/   │
                    │   Load      │
                    │   Balancer  │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │                         │
    ┌─────────▼─────────┐   ┌─────────▼─────────┐
    │   BLUE (Active)    │   │   GREEN (Idle)     │
    │   Version 1.0      │   │   Version 2.0      │
    │   ██████████████   │   │   ░░░░░░░░░░░░░░  │
    └───────────────────┘   └───────────────────┘

After validation, switch traffic:

    ┌───────────────────┐   ┌───────────────────┐
    │   BLUE (Idle)      │   │   GREEN (Active)   │
    │   Version 1.0      │   │   Version 2.0      │
    │   ░░░░░░░░░░░░░░  │   │   ██████████████   │
    └───────────────────┘   └───────────────────┘
```

**Pros:** Instant rollback, zero downtime, full testing in production environment  
**Cons:** Double infrastructure cost, database migrations are complex

#### 3. Canary Deployment

Route a small percentage of traffic to the new version; gradually increase.

```
Phase 1 (5% canary):
    ┌──────────────────────────────────────────────┐
    │████████████████████████████████████████████░░│
    │           95% v1                         5%v2│
    └──────────────────────────────────────────────┘

Phase 2 (25% canary):
    ┌──────────────────────────────────────────────┐
    │██████████████████████████████████░░░░░░░░░░░░│
    │         75% v1              25% v2           │
    └──────────────────────────────────────────────┘

Phase 3 (50% canary):
    ┌──────────────────────────────────────────────┐
    │████████████████████████░░░░░░░░░░░░░░░░░░░░░│
    │      50% v1              50% v2              │
    └──────────────────────────────────────────────┘

Phase 4 (100%):
    ┌──────────────────────────────────────────────┐
    │░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
    │                100% v2                       │
    └──────────────────────────────────────────────┘
```

**Pros:** Low risk, real production validation, metrics-driven decisions  
**Cons:** Complex routing, monitoring overhead, slow full rollout

#### 4. A/B Testing

Route traffic based on user attributes (not random). Used for feature experimentation.

**Difference from Canary:** A/B testing is for measuring user behavior/business metrics. Canary is for measuring application health.

#### 5. Feature Flags

Decouple deployment from release. Code is deployed but features are toggled on/off.

```javascript
// Feature flag example
if (featureFlags.isEnabled('new-checkout-flow', user)) {
  renderNewCheckout();
} else {
  renderOldCheckout();
}
```

**Tools:** LaunchDarkly, Azure App Configuration, Unleash, Flagsmith

### Artifact Management

**What:** Versioned, immutable build outputs (binaries, containers, packages)  
**Why:** Build once, deploy many times; traceability; rollback capability

**Common Registries:**

| Type | Options |
|------|---------|
| Docker Images | Azure Container Registry, Docker Hub, Harbor |
| npm Packages | Azure Artifacts, GitHub Packages, Nexus |
| NuGet (.NET) | Azure Artifacts, NuGet.org |
| Maven/Gradle | Azure Artifacts, Nexus, Artifactory |
| Helm Charts | Azure Container Registry, ChartMuseum |
| Generic | Azure Artifacts Universal Packages |

### Pipeline Security Best Practices

1. **Never store secrets in code** — Use secret management (Key Vault, Vault)
2. **Least privilege** — Service connections have minimal permissions
3. **Signed artifacts** — Verify artifact integrity
4. **SAST/DAST scanning** — Automate security testing in pipelines
5. **Dependency scanning** — Check for vulnerable packages (SCA)
6. **Container scanning** — Scan images for CVEs
7. **Branch protection** — Require PR reviews, no direct pushes to main
8. **Audit trail** — Log all pipeline activities
9. **Network isolation** — Agents in private networks where possible
10. **Secret rotation** — Automate credential rotation

### Pipeline Performance Optimization

1. **Parallelism** — Run independent jobs concurrently
2. **Caching** — Cache dependencies (node_modules, .m2, NuGet)
3. **Incremental builds** — Only build what changed
4. **Optimized images** — Use slim/alpine base images for agents
5. **Artifact size** — Only publish what's needed
6. **Test splitting** — Distribute tests across parallel jobs
7. **Skip unnecessary stages** — Use conditions/rules
8. **Self-hosted agents** — Pre-warmed with tools, local caches

---

## Section 2: Azure DevOps (Primary — GCC Market Focus)

> ⭐ **GCC Market Note:** Azure DevOps is the dominant CI/CD platform in the GCC region (UAE, Saudi Arabia, Qatar, Bahrain, Kuwait, Oman). Most government projects, banking, oil & gas, and enterprise organizations mandate Azure DevOps. Master this section thoroughly.

### Azure DevOps Overview

Azure DevOps is a suite of development tools covering the entire SDLC:

| Service | Purpose |
|---------|---------|
| **Azure Repos** | Git repositories with branch policies, PRs |
| **Azure Boards** | Agile planning (Scrum, Kanban, CMMI) |
| **Azure Pipelines** | CI/CD automation (YAML or Classic) |
| **Azure Artifacts** | Package management (npm, NuGet, Maven, Python, Universal) |
| **Azure Test Plans** | Manual/exploratory testing, test case management |

**Azure DevOps Server** (on-premises) vs **Azure DevOps Services** (cloud) — both used in GCC.

### YAML Pipeline Structure — Complete Reference

#### Trigger Configuration

```yaml
# CI Trigger — runs on push to these branches
trigger:
  branches:
    include:
      - main
      - release/*
    exclude:
      - feature/experimental/*
  paths:
    include:
      - src/**
      - Dockerfile
    exclude:
      - docs/**
      - '*.md'
  tags:
    include:
      - v*

# PR Trigger — runs on pull request to these branches
pr:
  branches:
    include:
      - main
      - develop
  paths:
    include:
      - src/**
  drafts: false  # Don't run on draft PRs
```

#### Pool Configuration

```yaml
# Microsoft-hosted agents
pool:
  vmImage: 'ubuntu-latest'      # Linux
  # vmImage: 'windows-latest'   # Windows
  # vmImage: 'macos-latest'     # macOS

# Self-hosted agents
pool:
  name: 'MyPrivatePool'
  demands:
    - docker
    - Agent.OS -equals Linux
```

**Microsoft-hosted agent images:**
- `ubuntu-latest` (Ubuntu 22.04) — Most common
- `windows-latest` (Windows Server 2022)
- `macos-latest` (macOS 14)

#### Stages, Jobs, and Steps

```yaml
stages:
  - stage: Build
    displayName: 'Build Application'
    jobs:
      - job: BuildJob
        displayName: 'Compile and Package'
        pool:
          vmImage: 'ubuntu-latest'
        timeoutInMinutes: 30
        steps:
          - checkout: self
            clean: true

          - task: UseDotNet@2
            displayName: 'Install .NET SDK'
            inputs:
              packageType: 'sdk'
              version: '8.x'

          - script: |
              dotnet build --configuration Release
              dotnet test --no-build
            displayName: 'Build and Test'

          - task: PublishBuildArtifacts@1
            inputs:
              PathtoPublish: '$(Build.ArtifactStagingDirectory)'
              ArtifactName: 'drop'
```

#### Variables and Variable Groups

```yaml
# Inline variables
variables:
  buildConfiguration: 'Release'
  dockerRegistry: 'myregistry.azurecr.io'
  imageName: 'myapp'

# Variable groups (from Library)
variables:
  - group: 'Production-Secrets'
  - group: 'Shared-Config'
  - name: localVar
    value: 'some-value'

# Stage-level variables
stages:
  - stage: Deploy
    variables:
      environment: 'production'
      replicas: 3
```

**Variable types:**
- **Pipeline variables** — Defined in YAML
- **Variable groups** — Defined in Library, shared across pipelines
- **Secret variables** — Encrypted, not shown in logs
- **Predefined variables** — `$(Build.BuildId)`, `$(System.DefaultWorkingDirectory)`, etc.

**Key predefined variables:**

| Variable | Description |
|----------|-------------|
| `$(Build.BuildId)` | Unique build number |
| `$(Build.SourceBranch)` | Full branch ref (refs/heads/main) |
| `$(Build.SourceBranchName)` | Short branch name (main) |
| `$(Build.Repository.Name)` | Repository name |
| `$(System.DefaultWorkingDirectory)` | Agent working directory |
| `$(Pipeline.Workspace)` | Workspace path |
| `$(Build.ArtifactStagingDirectory)` | Artifact staging path |

#### Parameters (Runtime Inputs)

```yaml
parameters:
  - name: environment
    displayName: 'Target Environment'
    type: string
    default: 'staging'
    values:
      - development
      - staging
      - production

  - name: deployRegion
    displayName: 'Azure Region'
    type: string
    default: 'uaenorth'

  - name: runTests
    displayName: 'Run Integration Tests'
    type: boolean
    default: true

  - name: imageTag
    displayName: 'Docker Image Tag'
    type: string
    default: '$(Build.BuildId)'

stages:
  - stage: Deploy
    displayName: 'Deploy to ${{ parameters.environment }}'
    jobs:
      - job: DeployJob
        steps:
          - script: echo "Deploying to ${{ parameters.environment }} in ${{ parameters.deployRegion }}"
          - ${{ if eq(parameters.runTests, true) }}:
            - script: echo "Running integration tests..."
```

#### Conditions and Dependencies

```yaml
stages:
  - stage: Build
    jobs:
      - job: BuildApp
        steps:
          - script: echo "Building..."

  - stage: Test
    dependsOn: Build
    condition: succeeded()  # Only run if Build succeeded

  - stage: DeployStaging
    dependsOn: Test
    condition: |
      and(
        succeeded(),
        ne(variables['Build.Reason'], 'PullRequest')
      )

  - stage: DeployProduction
    dependsOn: DeployStaging
    condition: |
      and(
        succeeded(),
        eq(variables['Build.SourceBranch'], 'refs/heads/main')
      )
```

**Common conditions:**
- `succeeded()` — Previous stage/job succeeded
- `failed()` — Previous stage/job failed
- `always()` — Run regardless
- `canceled()` — Pipeline was canceled
- `eq(variables['Build.SourceBranch'], 'refs/heads/main')` — Branch check
- `contains(variables['Build.SourceBranch'], 'release')` — Branch pattern

#### Templates

**Step template** (`templates/build-steps.yml`):

```yaml
# templates/build-steps.yml
parameters:
  - name: buildConfiguration
    type: string
    default: 'Release'
  - name: projectPath
    type: string

steps:
  - task: UseDotNet@2
    inputs:
      packageType: 'sdk'
      version: '8.x'

  - script: dotnet restore ${{ parameters.projectPath }}
    displayName: 'Restore packages'

  - script: dotnet build ${{ parameters.projectPath }} --configuration ${{ parameters.buildConfiguration }}
    displayName: 'Build project'

  - script: dotnet test ${{ parameters.projectPath }} --no-build --configuration ${{ parameters.buildConfiguration }}
    displayName: 'Run tests'
```

**Job template** (`templates/deploy-job.yml`):

```yaml
# templates/deploy-job.yml
parameters:
  - name: environment
    type: string
  - name: azureSubscription
    type: string
  - name: appName
    type: string

jobs:
  - deployment: Deploy
    displayName: 'Deploy to ${{ parameters.environment }}'
    environment: ${{ parameters.environment }}
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
            - task: AzureWebApp@1
              inputs:
                azureSubscription: ${{ parameters.azureSubscription }}
                appType: 'webAppLinux'
                appName: ${{ parameters.appName }}
                package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

**Stage template** (`templates/deployment-stage.yml`):

```yaml
# templates/deployment-stage.yml
parameters:
  - name: stageName
    type: string
  - name: environment
    type: string
  - name: dependsOn
    type: string
    default: ''
  - name: azureSubscription
    type: string
  - name: resourceGroup
    type: string

stages:
  - stage: ${{ parameters.stageName }}
    dependsOn: ${{ parameters.dependsOn }}
    jobs:
      - template: deploy-job.yml
        parameters:
          environment: ${{ parameters.environment }}
          azureSubscription: ${{ parameters.azureSubscription }}
          appName: 'app-${{ parameters.environment }}'
```

**Using templates in main pipeline:**

```yaml
trigger:
  - main

stages:
  - stage: Build
    jobs:
      - job: BuildJob
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - template: templates/build-steps.yml
            parameters:
              buildConfiguration: 'Release'
              projectPath: 'src/MyApp.sln'

  - template: templates/deployment-stage.yml
    parameters:
      stageName: 'DeployStaging'
      environment: 'staging'
      dependsOn: 'Build'
      azureSubscription: 'Azure-Staging'
      resourceGroup: 'rg-staging'

  - template: templates/deployment-stage.yml
    parameters:
      stageName: 'DeployProd'
      environment: 'production'
      dependsOn: 'DeployStaging'
      azureSubscription: 'Azure-Production'
      resourceGroup: 'rg-production'
```

#### Extends Templates (Pipeline Security)

```yaml
# templates/restricted-pipeline.yml (controlled by platform team)
parameters:
  - name: buildSteps
    type: stepList
    default: []

stages:
  - stage: Build
    jobs:
      - job: Build
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - ${{ each step in parameters.buildSteps }}:
            - ${{ step }}

  - stage: SecurityScan
    dependsOn: Build
    jobs:
      - job: Scan
        steps:
          - task: CredScan@3
          - task: ContainerScan@0

# Main pipeline uses extends
extends:
  template: templates/restricted-pipeline.yml
  parameters:
    buildSteps:
      - script: npm install
      - script: npm run build
```


### Complete Azure DevOps Pipeline Examples

#### Example 1: Docker Build and Push Pipeline

```yaml
trigger:
  branches:
    include:
      - main
      - develop

variables:
  dockerRegistry: 'myacr.azurecr.io'
  imageName: 'myapp-api'
  tag: '$(Build.BuildId)'

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: Build
    displayName: 'Build and Push Docker Image'
    jobs:
      - job: DockerBuild
        steps:
          - task: Docker@2
            displayName: 'Login to ACR'
            inputs:
              command: login
              containerRegistry: 'ACR-ServiceConnection'

          - task: Docker@2
            displayName: 'Build Image'
            inputs:
              command: build
              repository: $(imageName)
              dockerfile: '$(Build.SourcesDirectory)/Dockerfile'
              buildContext: '$(Build.SourcesDirectory)'
              tags: |
                $(tag)
                latest
              arguments: '--build-arg BUILD_NUMBER=$(Build.BuildId)'

          - task: Docker@2
            displayName: 'Push Image'
            inputs:
              command: push
              repository: $(imageName)
              containerRegistry: 'ACR-ServiceConnection'
              tags: |
                $(tag)
                latest

          - task: trivy@1
            displayName: 'Scan Image for Vulnerabilities'
            inputs:
              image: '$(dockerRegistry)/$(imageName):$(tag)'
              severities: 'CRITICAL,HIGH'
```

#### Example 2: Multi-Stage Pipeline (Build → Test → Security → Deploy)

```yaml
trigger:
  branches:
    include:
      - main
      - release/*

parameters:
  - name: deployToProd
    displayName: 'Deploy to Production'
    type: boolean
    default: false

variables:
  - group: 'App-Settings'
  - name: buildConfiguration
    value: 'Release'

stages:
  # ===== BUILD STAGE =====
  - stage: Build
    displayName: '🔨 Build'
    jobs:
      - job: BuildApp
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: UseDotNet@2
            inputs:
              packageType: 'sdk'
              version: '8.x'

          - script: |
              dotnet restore
              dotnet build --configuration $(buildConfiguration) --no-restore
              dotnet publish --configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/app
            displayName: 'Build Application'

          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: '$(Build.ArtifactStagingDirectory)/app'
              artifact: 'app-package'

  # ===== TEST STAGE =====
  - stage: Test
    displayName: '🧪 Test'
    dependsOn: Build
    jobs:
      - job: UnitTests
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - script: dotnet test --configuration $(buildConfiguration) --collect:"XPlat Code Coverage" --results-directory $(Build.SourcesDirectory)/TestResults
            displayName: 'Run Unit Tests'

          - task: PublishCodeCoverageResults@2
            inputs:
              summaryFileLocation: '$(Build.SourcesDirectory)/TestResults/**/coverage.cobertura.xml'

      - job: IntegrationTests
        pool:
          vmImage: 'ubuntu-latest'
        services:
          postgres:
            image: postgres:15
            ports:
              - 5432:5432
        steps:
          - script: dotnet test --filter Category=Integration
            displayName: 'Run Integration Tests'
            env:
              ConnectionStrings__Default: 'Host=localhost;Database=testdb;Username=postgres;Password=postgres'

  # ===== SECURITY STAGE =====
  - stage: Security
    displayName: '🔒 Security Scan'
    dependsOn: Build
    jobs:
      - job: SecurityScan
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: UseDotNet@2
            inputs:
              packageType: 'sdk'
              version: '8.x'

          - script: |
              dotnet tool install --global security-scan
              security-scan $(Build.SourcesDirectory)/src/
            displayName: 'SAST Scan'

          - script: |
              pip install safety
              safety check --full-report
            displayName: 'Dependency Vulnerability Check'

  # ===== DEPLOY STAGING =====
  - stage: DeployStaging
    displayName: '🚀 Deploy Staging'
    dependsOn:
      - Test
      - Security
    condition: and(succeeded(), ne(variables['Build.Reason'], 'PullRequest'))
    jobs:
      - deployment: DeployToStaging
        environment: 'staging'
        pool:
          vmImage: 'ubuntu-latest'
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: 'app-package'

                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'Azure-Staging-SC'
                    appType: 'webAppLinux'
                    appName: 'myapp-staging'
                    package: '$(Pipeline.Workspace)/app-package/**/*.zip'

                - script: |
                    curl -f https://myapp-staging.azurewebsites.net/health || exit 1
                  displayName: 'Smoke Test'

  # ===== DEPLOY PRODUCTION =====
  - stage: DeployProduction
    displayName: '🏁 Deploy Production'
    dependsOn: DeployStaging
    condition: |
      and(
        succeeded(),
        eq(variables['Build.SourceBranch'], 'refs/heads/main'),
        eq('${{ parameters.deployToProd }}', 'true')
      )
    jobs:
      - deployment: DeployToProd
        environment: 'production'  # Has manual approval gate
        pool:
          vmImage: 'ubuntu-latest'
        strategy:
          rolling:
            maxParallel: 25%
            deploy:
              steps:
                - download: current
                  artifact: 'app-package'

                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'Azure-Production-SC'
                    appType: 'webAppLinux'
                    appName: 'myapp-production'
                    package: '$(Pipeline.Workspace)/app-package/**/*.zip'
                    deploymentMethod: 'zipDeploy'
                    slotName: 'staging'

                - task: AzureAppServiceManage@0
                  inputs:
                    azureSubscription: 'Azure-Production-SC'
                    Action: 'Swap Slots'
                    WebAppName: 'myapp-production'
                    SourceSlot: 'staging'
```

#### Example 3: Terraform Pipeline (Plan on PR, Apply on Merge)

```yaml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - infrastructure/**

pr:
  branches:
    include:
      - main
  paths:
    include:
      - infrastructure/**

variables:
  - group: 'Terraform-Backend'
  - name: workingDirectory
    value: '$(System.DefaultWorkingDirectory)/infrastructure'
  - name: tfVersion
    value: '1.7.0'

stages:
  - stage: Validate
    displayName: 'Terraform Validate'
    jobs:
      - job: Validate
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: TerraformInstaller@1
            inputs:
              terraformVersion: $(tfVersion)

          - task: TerraformTaskV4@4
            displayName: 'Terraform Init'
            inputs:
              provider: 'azurerm'
              command: 'init'
              workingDirectory: $(workingDirectory)
              backendServiceArm: 'Azure-Terraform-SC'
              backendAzureRmResourceGroupName: 'rg-terraform-state'
              backendAzureRmStorageAccountName: 'stterraformstate'
              backendAzureRmContainerName: 'tfstate'
              backendAzureRmKey: 'infrastructure.tfstate'

          - task: TerraformTaskV4@4
            displayName: 'Terraform Validate'
            inputs:
              provider: 'azurerm'
              command: 'validate'
              workingDirectory: $(workingDirectory)

          - script: |
              cd $(workingDirectory)
              terraform fmt -check -recursive
            displayName: 'Terraform Format Check'

  - stage: Plan
    displayName: 'Terraform Plan'
    dependsOn: Validate
    jobs:
      - job: Plan
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: TerraformInstaller@1
            inputs:
              terraformVersion: $(tfVersion)

          - task: TerraformTaskV4@4
            displayName: 'Terraform Init'
            inputs:
              provider: 'azurerm'
              command: 'init'
              workingDirectory: $(workingDirectory)
              backendServiceArm: 'Azure-Terraform-SC'
              backendAzureRmResourceGroupName: 'rg-terraform-state'
              backendAzureRmStorageAccountName: 'stterraformstate'
              backendAzureRmContainerName: 'tfstate'
              backendAzureRmKey: 'infrastructure.tfstate'

          - task: TerraformTaskV4@4
            displayName: 'Terraform Plan'
            inputs:
              provider: 'azurerm'
              command: 'plan'
              workingDirectory: $(workingDirectory)
              environmentServiceNameAzureRM: 'Azure-Terraform-SC'
              commandOptions: '-out=tfplan -input=false'

          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: '$(workingDirectory)/tfplan'
              artifact: 'tfplan'

  - stage: Apply
    displayName: 'Terraform Apply'
    dependsOn: Plan
    condition: |
      and(
        succeeded(),
        eq(variables['Build.SourceBranch'], 'refs/heads/main'),
        ne(variables['Build.Reason'], 'PullRequest')
      )
    jobs:
      - deployment: TerraformApply
        environment: 'infrastructure-production'
        pool:
          vmImage: 'ubuntu-latest'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: TerraformInstaller@1
                  inputs:
                    terraformVersion: $(tfVersion)

                - task: TerraformTaskV4@4
                  displayName: 'Terraform Init'
                  inputs:
                    provider: 'azurerm'
                    command: 'init'
                    workingDirectory: $(workingDirectory)
                    backendServiceArm: 'Azure-Terraform-SC'
                    backendAzureRmResourceGroupName: 'rg-terraform-state'
                    backendAzureRmStorageAccountName: 'stterraformstate'
                    backendAzureRmContainerName: 'tfstate'
                    backendAzureRmKey: 'infrastructure.tfstate'

                - download: current
                  artifact: 'tfplan'

                - task: TerraformTaskV4@4
                  displayName: 'Terraform Apply'
                  inputs:
                    provider: 'azurerm'
                    command: 'apply'
                    workingDirectory: $(workingDirectory)
                    environmentServiceNameAzureRM: 'Azure-Terraform-SC'
                    commandOptions: '$(Pipeline.Workspace)/tfplan/tfplan'
```

#### Example 4: Kubernetes Deployment Pipeline (AKS)

```yaml
trigger:
  branches:
    include:
      - main

variables:
  dockerRegistry: 'myacr.azurecr.io'
  imageName: 'myapp'
  tag: '$(Build.BuildId)'
  k8sNamespace: 'production'

stages:
  - stage: BuildAndPush
    displayName: 'Build & Push Image'
    jobs:
      - job: Build
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: Docker@2
            displayName: 'Build and Push'
            inputs:
              containerRegistry: 'ACR-Connection'
              repository: $(imageName)
              command: 'buildAndPush'
              Dockerfile: '**/Dockerfile'
              tags: |
                $(tag)
                latest

  - stage: DeployToAKS
    displayName: 'Deploy to AKS'
    dependsOn: BuildAndPush
    jobs:
      - deployment: DeployAKS
        environment: 'aks-production.$(k8sNamespace)'
        pool:
          vmImage: 'ubuntu-latest'
        strategy:
          canary:
            increments: [10, 20, 50]
            deploy:
              steps:
                - task: KubernetesManifest@1
                  displayName: 'Deploy to AKS'
                  inputs:
                    action: 'deploy'
                    connectionType: 'azureResourceManager'
                    azureSubscriptionConnection: 'Azure-AKS-SC'
                    azureResourceGroup: 'rg-aks'
                    kubernetesCluster: 'aks-production'
                    namespace: $(k8sNamespace)
                    manifests: |
                      k8s/deployment.yml
                      k8s/service.yml
                    containers: '$(dockerRegistry)/$(imageName):$(tag)'
                    imagePullSecrets: 'acr-secret'

            on:
              failure:
                steps:
                  - task: KubernetesManifest@1
                    displayName: 'Rollback'
                    inputs:
                      action: 'reject'
                      connectionType: 'azureResourceManager'
                      azureSubscriptionConnection: 'Azure-AKS-SC'
                      azureResourceGroup: 'rg-aks'
                      kubernetesCluster: 'aks-production'
                      namespace: $(k8sNamespace)
```

#### Example 5: .NET Application Pipeline

```yaml
trigger:
  - main
  - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'

stages:
  - stage: Build
    jobs:
      - job: Build
        steps:
          - task: UseDotNet@2
            inputs:
              packageType: 'sdk'
              version: '8.x'

          - task: DotNetCoreCLI@2
            displayName: 'Restore'
            inputs:
              command: 'restore'
              projects: $(solution)
              feedsToUse: 'select'
              vstsFeed: 'MyFeed'

          - task: DotNetCoreCLI@2
            displayName: 'Build'
            inputs:
              command: 'build'
              projects: $(solution)
              arguments: '--configuration $(buildConfiguration) --no-restore'

          - task: DotNetCoreCLI@2
            displayName: 'Test'
            inputs:
              command: 'test'
              projects: '**/*Tests/*.csproj'
              arguments: '--configuration $(buildConfiguration) --collect:"XPlat Code Coverage"'

          - task: DotNetCoreCLI@2
            displayName: 'Publish'
            inputs:
              command: 'publish'
              publishWebProjects: true
              arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
              zipAfterPublish: true

          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: '$(Build.ArtifactStagingDirectory)'
              artifact: 'webapp'
```

#### Example 6: Node.js Application Pipeline

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  nodeVersion: '20.x'

stages:
  - stage: Build
    jobs:
      - job: BuildAndTest
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: $(nodeVersion)

          - task: Cache@2
            displayName: 'Cache npm'
            inputs:
              key: 'npm | "$(Agent.OS)" | package-lock.json'
              restoreKeys: |
                npm | "$(Agent.OS)"
              path: '$(Pipeline.Workspace)/.npm'

          - script: npm ci --cache $(Pipeline.Workspace)/.npm
            displayName: 'Install Dependencies'

          - script: npm run lint
            displayName: 'Lint'

          - script: npm run test -- --coverage --ci
            displayName: 'Unit Tests'

          - task: PublishTestResults@2
            condition: succeededOrFailed()
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: '**/junit.xml'

          - task: PublishCodeCoverageResults@2
            inputs:
              summaryFileLocation: '$(System.DefaultWorkingDirectory)/coverage/cobertura-coverage.xml'

          - script: npm run build
            displayName: 'Build'

          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: 'dist'
              artifact: 'webapp-dist'
```


### Environments and Approvals

Environments represent deployment targets with gates and checks.

**Creating environments:**
- Azure DevOps → Pipelines → Environments → New Environment
- Types: Kubernetes, Virtual Machine, or None (abstract)

**Approval gates:**
1. **Manual approval** — Require specific users/groups to approve
2. **Branch control** — Only allow deployments from specific branches
3. **Business hours** — Restrict deployments to certain times
4. **Invoke REST API** — Call external service for validation
5. **Query Azure Monitor** — Check metrics before proceeding
6. **Evaluate artifact** — Verify artifact policies

```yaml
# Using environment with approval in pipeline
jobs:
  - deployment: DeployProd
    environment: 'production'  # Approvals configured in UI
    strategy:
      runOnce:
        deploy:
          steps:
            - script: echo "Deploying to production"
```

### Service Connections

Service connections authenticate pipelines to external services.

| Type | Use Case |
|------|----------|
| Azure Resource Manager | Deploy to Azure (App Service, AKS, etc.) |
| Docker Registry | Push/pull container images |
| Kubernetes | Deploy to K8s clusters |
| SSH | Deploy via SSH/SCP |
| GitHub | Access GitHub repos |
| Generic | REST API endpoints |

**Best practice:** Use Workload Identity Federation (passwordless) for Azure connections.

### Variable Groups and Azure Key Vault Integration

```yaml
# Link variable group to Azure Key Vault
# In Library → Variable Groups → Link secrets from Azure Key Vault

# In pipeline:
variables:
  - group: 'KeyVault-Secrets'  # Linked to Azure Key Vault

steps:
  - script: |
      echo "Using secret: $(my-secret-name)"
    displayName: 'Use Key Vault Secret'
    env:
      SECRET_VALUE: $(my-secret-name)  # Map to env var for security
```

### Self-Hosted Agents

**When to use:**
- Need access to private networks
- Specific software requirements
- Performance (persistent caches)
- Compliance (data residency in GCC)

**Setup on Linux:**
```bash
# Download agent
mkdir myagent && cd myagent
curl -O https://vstsagentpackage.azureedge.net/agent/3.x.x/vsts-agent-linux-x64-3.x.x.tar.gz
tar zxvf vsts-agent-linux-x64-3.x.x.tar.gz

# Configure
./config.sh --url https://dev.azure.com/yourorg --auth pat --token YOUR_PAT --pool MyPool --agent myagent-01

# Run as service
sudo ./svc.sh install
sudo ./svc.sh start
```

**Agent capabilities:** Custom capabilities can be added to match demands in pipelines.

### Matrix and Parallel Strategies

```yaml
jobs:
  - job: Test
    strategy:
      matrix:
        linux-node18:
          vmImage: 'ubuntu-latest'
          nodeVersion: '18.x'
        linux-node20:
          vmImage: 'ubuntu-latest'
          nodeVersion: '20.x'
        windows-node20:
          vmImage: 'windows-latest'
          nodeVersion: '20.x'
      maxParallel: 3
    pool:
      vmImage: $(vmImage)
    steps:
      - task: NodeTool@0
        inputs:
          versionSpec: $(nodeVersion)
      - script: npm test
```

### Deployment Jobs and Strategies

```yaml
# Rolling deployment
jobs:
  - deployment: RollingDeploy
    environment:
      name: 'production'
      resourceType: VirtualMachine
    strategy:
      rolling:
        maxParallel: 25%
        preDeploy:
          steps:
            - script: echo "Pre-deploy health check"
        deploy:
          steps:
            - script: echo "Deploying application"
        routeTraffic:
          steps:
            - script: echo "Routing traffic"
        postRouteTraffic:
          steps:
            - script: echo "Post-deploy validation"
        on:
          failure:
            steps:
              - script: echo "Rolling back!"
          success:
            steps:
              - script: echo "Deployment successful!"

# Canary deployment
jobs:
  - deployment: CanaryDeploy
    environment: 'production'
    strategy:
      canary:
        increments: [10, 20, 50, 100]
        deploy:
          steps:
            - script: echo "Deploying canary"
        routeTraffic:
          steps:
            - script: echo "Routing $(Strategy.Increment)% traffic"
        postRouteTraffic:
          steps:
            - script: echo "Monitoring canary metrics"
            - task: InvokeRestAPI@1
              inputs:
                serviceConnection: 'Monitoring-SC'
                method: 'GET'
                urlSuffix: '/api/canary/health'
```

### Azure DevOps CLI

```bash
# Install extension
az extension add --name azure-devops

# Login
az login
az devops configure --defaults organization=https://dev.azure.com/myorg project=MyProject

# Pipeline operations
az pipelines create --name "My Pipeline" --yaml-path /azure-pipelines.yml --repository myrepo --branch main
az pipelines run --name "My Pipeline" --branch main
az pipelines list --output table
az pipelines show --name "My Pipeline"

# Variable groups
az pipelines variable-group create --name "My-Vars" --variables key1=value1 key2=value2
az pipelines variable-group variable create --group-id 1 --name "secret" --value "hidden" --secret true

# Service connections
az devops service-endpoint list --output table
```

### Azure DevOps REST API

```bash
# List pipelines
curl -u :$PAT "https://dev.azure.com/{org}/{project}/_apis/pipelines?api-version=7.1"

# Queue a build
curl -X POST -u :$PAT \
  -H "Content-Type: application/json" \
  -d '{"definition": {"id": 1}, "sourceBranch": "refs/heads/main"}' \
  "https://dev.azure.com/{org}/{project}/_apis/build/builds?api-version=7.1"

# Get build status
curl -u :$PAT "https://dev.azure.com/{org}/{project}/_apis/build/builds/{buildId}?api-version=7.1"
```

### Organization Structure Best Practices (GCC)

```
Organization: company-name
├── Project: platform-infrastructure
│   ├── Repos: terraform-modules, ansible-playbooks
│   ├── Pipelines: infra-pipelines
│   └── Boards: Infrastructure backlog
├── Project: application-frontend
│   ├── Repos: web-app, mobile-app
│   ├── Pipelines: frontend-ci, frontend-cd
│   └── Boards: Frontend sprint board
├── Project: application-backend
│   ├── Repos: api-gateway, microservices
│   ├── Pipelines: backend-ci, backend-cd
│   └── Boards: Backend sprint board
└── Project: shared-templates
    ├── Repos: pipeline-templates, scripts
    └── Pipelines: template-validation
```

**GCC-specific considerations:**
- Data residency: Use Azure UAE North/Central regions
- Compliance: Enable audit logs, restrict access
- Agent pools: Self-hosted in UAE data centers for latency
- Security: Integrate with Azure AD (Entra ID) for SSO

---

## Section 3: GitLab CI/CD

### .gitlab-ci.yml Structure

```yaml
# Global configuration
default:
  image: node:20-alpine
  before_script:
    - echo "Global before script"
  after_script:
    - echo "Cleanup"
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure

# Define stages (execution order)
stages:
  - build
  - test
  - security
  - deploy

# Variables
variables:
  APP_NAME: "myapp"
  DOCKER_REGISTRY: "registry.gitlab.com"
```

### Stages, Jobs, and Rules

```yaml
stages:
  - build
  - test
  - deploy

build-app:
  stage: build
  image: node:20
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

# Rules (modern replacement for only/except)
test-app:
  stage: test
  script:
    - npm test
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: always
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: always
    - if: '$CI_COMMIT_TAG'
      when: never
    - when: manual  # Default: manual trigger

deploy-staging:
  stage: deploy
  script:
    - deploy-to-staging.sh
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: on_success
    - if: '$CI_COMMIT_BRANCH == "develop"'
      when: manual
      allow_failure: true

deploy-production:
  stage: deploy
  script:
    - deploy-to-prod.sh
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
  environment:
    name: production
    url: https://myapp.com
```

### Variables and Predefined CI Variables

| Variable | Description |
|----------|-------------|
| `$CI_COMMIT_SHA` | Full commit SHA |
| `$CI_COMMIT_SHORT_SHA` | Short commit SHA (8 chars) |
| `$CI_COMMIT_BRANCH` | Branch name |
| `$CI_COMMIT_TAG` | Tag name |
| `$CI_PIPELINE_SOURCE` | What triggered (push, merge_request_event, schedule) |
| `$CI_PROJECT_DIR` | Full path to project directory |
| `$CI_REGISTRY` | Container registry URL |
| `$CI_REGISTRY_IMAGE` | Registry image path |
| `$CI_ENVIRONMENT_NAME` | Environment name |

### Cache vs Artifacts

```yaml
# Cache: Shared between pipeline runs (speeds up builds)
build-job:
  cache:
    key:
      files:
        - package-lock.json
    paths:
      - node_modules/
    policy: pull-push  # pull, push, or pull-push

# Artifacts: Passed between jobs in same pipeline
build-job:
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    reports:
      junit: test-results.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    expire_in: 1 week
```

### Services (DinD, Databases)

```yaml
# Docker-in-Docker for building images
build-docker:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  variables:
    DOCKER_HOST: tcp://docker:2376
    DOCKER_TLS_CERTDIR: "/certs"
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

# Database service for testing
test-with-db:
  stage: test
  image: python:3.11
  services:
    - name: postgres:15
      alias: db
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: test
        POSTGRES_PASSWORD: test
  variables:
    DATABASE_URL: "postgresql://test:test@db:5432/testdb"
  script:
    - pip install -r requirements.txt
    - pytest
```

### Environments and Review Apps

```yaml
# Dynamic review environments for merge requests
deploy-review:
  stage: deploy
  script:
    - kubectl apply -f k8s/ --namespace review-$CI_MERGE_REQUEST_IID
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://$CI_COMMIT_REF_SLUG.review.myapp.com
    on_stop: stop-review
    auto_stop_in: 1 week
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'

stop-review:
  stage: deploy
  script:
    - kubectl delete namespace review-$CI_MERGE_REQUEST_IID
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: manual
```

### Includes and Templates

```yaml
# Include from various sources
include:
  # Local file in same repo
  - local: '/templates/build.yml'

  # From another project
  - project: 'devops/pipeline-templates'
    ref: main
    file: '/templates/docker-build.yml'

  # Remote URL
  - remote: 'https://example.com/templates/security.yml'

  # GitLab's built-in templates
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
```

### Parent-Child and Multi-Project Pipelines

```yaml
# Parent pipeline triggers child
trigger-deploy:
  stage: deploy
  trigger:
    include:
      - local: deploy/.gitlab-ci.yml
    strategy: depend  # Wait for child to complete

# Multi-project trigger
trigger-downstream:
  stage: deploy
  trigger:
    project: team/downstream-project
    branch: main
    strategy: depend
```

### GitLab Runners

| Runner Type | Scope | Use Case |
|-------------|-------|----------|
| Shared | Instance-wide | General purpose |
| Group | Group projects | Team-specific |
| Project | Single project | Specialized needs |

**Runner Executors:**
- **Docker** — Each job runs in a container (most common)
- **Shell** — Runs directly on runner host
- **Kubernetes** — Jobs run as pods in K8s cluster
- **Docker Machine** — Auto-scaling with cloud VMs

### DAG with `needs:`

```yaml
stages:
  - build
  - test
  - deploy

build-frontend:
  stage: build
  script: npm run build:frontend

build-backend:
  stage: build
  script: npm run build:backend

test-frontend:
  stage: test
  needs: [build-frontend]  # Starts immediately when build-frontend finishes
  script: npm run test:frontend

test-backend:
  stage: test
  needs: [build-backend]  # Doesn't wait for build-frontend
  script: npm run test:backend

deploy:
  stage: deploy
  needs: [test-frontend, test-backend]
  script: deploy.sh
```

### Complete GitLab Pipeline Examples

#### Docker Build Pipeline

```yaml
stages:
  - build
  - scan
  - push

variables:
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

build-image:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t $IMAGE_TAG .
    - docker save $IMAGE_TAG > image.tar
  artifacts:
    paths:
      - image.tar

scan-image:
  stage: scan
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  script:
    - trivy image --input image.tar --severity HIGH,CRITICAL --exit-code 1
  needs: [build-image]

push-image:
  stage: push
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker load < image.tar
    - docker push $IMAGE_TAG
    - docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest
  needs: [scan-image]
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

#### Terraform Pipeline (GitLab)

```yaml
stages:
  - validate
  - plan
  - apply

variables:
  TF_DIR: "infrastructure"

.terraform-base:
  image:
    name: hashicorp/terraform:1.7
    entrypoint: [""]
  before_script:
    - cd $TF_DIR
    - terraform init -backend-config="address=${TF_STATE_ADDRESS}"

validate:
  extends: .terraform-base
  stage: validate
  script:
    - terraform validate
    - terraform fmt -check

plan:
  extends: .terraform-base
  stage: plan
  script:
    - terraform plan -out=tfplan
  artifacts:
    paths:
      - $TF_DIR/tfplan
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'

apply:
  extends: .terraform-base
  stage: apply
  script:
    - terraform apply tfplan
  dependencies:
    - plan
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
  environment:
    name: production
```

---

## Section 4: Jenkins

### Jenkins Architecture

```
┌────────────────────────────────────────────────┐
│              Jenkins Controller                  │
│  ┌──────────┐ ┌────────┐ ┌─────────────────┐  │
│  │ Web UI   │ │ API    │ │ Job Scheduler   │  │
│  └──────────┘ └────────┘ └─────────────────┘  │
│  ┌──────────────────────────────────────────┐  │
│  │         Plugin Manager                    │  │
│  └──────────────────────────────────────────┘  │
└───────────────────┬────────────────────────────┘
                    │
        ┌───────────┼───────────┐
        │           │           │
   ┌────▼────┐ ┌───▼────┐ ┌───▼────┐
   │ Agent 1 │ │Agent 2 │ │Agent 3 │
   │ (Linux) │ │(Windows)│ │ (K8s)  │
   └─────────┘ └────────┘ └────────┘
```

**Components:**
- **Controller** — Orchestrates jobs, serves UI, manages plugins
- **Agents** — Execute build jobs (permanent or ephemeral)
- **Plugins** — Extend functionality (2000+ available)

### Declarative vs Scripted Pipelines

```groovy
// DECLARATIVE PIPELINE (Recommended)
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
    }
}

// SCRIPTED PIPELINE (More flexible, more complex)
node {
    stage('Build') {
        sh 'make build'
    }
}
```

### Jenkinsfile Deep Dive — Declarative Pipeline

```groovy
pipeline {
    // Where to run
    agent {
        docker {
            image 'node:20'
            args '-v /tmp:/tmp'
        }
    }

    // Environment variables
    environment {
        APP_NAME = 'myapp'
        DOCKER_REGISTRY = 'myregistry.azurecr.io'
        AZURE_CREDENTIALS = credentials('azure-sp')  // Binds username/password
        API_KEY = credentials('api-key')              // Binds secret text
    }

    // Parameters (runtime inputs)
    parameters {
        string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: 'Target environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run test suite')
        choice(name: 'REGION', choices: ['uaenorth', 'westeurope', 'eastus'], description: 'Azure region')
    }

    // Pipeline options
    options {
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        retry(2)
    }

    // Triggers
    triggers {
        pollSCM('H/5 * * * *')  // Poll every 5 minutes
        cron('H 2 * * 1-5')     // Nightly at 2 AM weekdays
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git log --oneline -5'
            }
        }

        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }

        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh 'npm audit --audit-level=high'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'acr-credentials') {
                        def image = docker.build("${APP_NAME}:${BUILD_NUMBER}")
                        image.push()
                        image.push('latest')
                    }
                }
            }
        }

        stage('Deploy Staging') {
            when {
                branch 'main'
            }
            steps {
                sh "kubectl apply -f k8s/staging/"
            }
        }

        stage('Approval') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?',
                      ok: 'Deploy',
                      submitter: 'admin,release-team'
            }
        }

        stage('Deploy Production') {
            when {
                branch 'main'
            }
            steps {
                sh "kubectl apply -f k8s/production/"
            }
        }
    }

    // Post-build actions
    post {
        always {
            junit '**/test-results/*.xml'
            archiveArtifacts artifacts: 'dist/**', fingerprint: true
            cleanWs()
        }
        success {
            slackSend(color: 'good', message: "Build ${BUILD_NUMBER} succeeded!")
        }
        failure {
            slackSend(color: 'danger', message: "Build ${BUILD_NUMBER} FAILED!")
            emailext(
                to: 'team@company.com',
                subject: "FAILED: ${JOB_NAME} #${BUILD_NUMBER}",
                body: "Check: ${BUILD_URL}"
            )
        }
        unstable {
            slackSend(color: 'warning', message: "Build ${BUILD_NUMBER} unstable")
        }
    }
}
```

### Shared Libraries

```groovy
// vars/buildDockerImage.groovy (in shared library repo)
def call(Map config = [:]) {
    def registry = config.registry ?: 'docker.io'
    def imageName = config.imageName
    def tag = config.tag ?: env.BUILD_NUMBER

    docker.withRegistry("https://${registry}", config.credentialsId) {
        def image = docker.build("${imageName}:${tag}")
        image.push()
        if (config.pushLatest) {
            image.push('latest')
        }
    }
}

// Using shared library in Jenkinsfile
@Library('my-shared-library') _

pipeline {
    agent any
    stages {
        stage('Build Image') {
            steps {
                buildDockerImage(
                    registry: 'myacr.azurecr.io',
                    imageName: 'myapp',
                    credentialsId: 'acr-creds',
                    pushLatest: true
                )
            }
        }
    }
}
```

### Jenkins + Kubernetes (Dynamic Agents)

```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:24-dind
    securityContext:
      privileged: true
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
      - sleep
    args:
      - infinity
  - name: node
    image: node:20
    command:
      - sleep
    args:
      - infinity
'''
        }
    }
    stages {
        stage('Build') {
            steps {
                container('node') {
                    sh 'npm ci && npm run build'
                }
            }
        }
        stage('Docker') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp .'
                }
            }
        }
        stage('Deploy') {
            steps {
                container('kubectl') {
                    sh 'kubectl apply -f k8s/'
                }
            }
        }
    }
}
```

### Common Jenkins Plugins

| Plugin | Purpose |
|--------|---------|
| Pipeline | Jenkinsfile support |
| Git | Source code management |
| Docker Pipeline | Docker integration |
| Kubernetes | Dynamic K8s agents |
| Blue Ocean | Modern UI |
| Credentials | Secret management |
| Slack Notification | Slack alerts |
| SonarQube Scanner | Code quality |
| OWASP Dependency-Check | Security scanning |
| Pipeline Utility Steps | File operations |

### Migration from Jenkins to Azure DevOps

| Jenkins | Azure DevOps |
|---------|-------------|
| Jenkinsfile | azure-pipelines.yml |
| `agent any` | `pool: vmImage: 'ubuntu-latest'` |
| `stage('Build')` | `- stage: Build` |
| `sh 'command'` | `- script: command` |
| `parallel { }` | Jobs run in parallel by default |
| `input` | Environment approvals |
| `credentials()` | Variable groups / Key Vault |
| Shared Libraries | Templates |
| `post { always { } }` | `condition: always()` |

---

## Section 5: GitHub Actions

### Workflow YAML Structure

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
    paths:
      - 'src/**'
      - 'Dockerfile'
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * 1-5'  # Weekdays at 2 AM UTC
  workflow_dispatch:  # Manual trigger
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm test
      - run: npm run build

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.myapp.com
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist

      - name: Deploy to Azure
        uses: azure/webapps-deploy@v3
        with:
          app-name: 'myapp-staging'
          publish-profile: ${{ secrets.AZURE_PUBLISH_PROFILE_STAGING }}
          package: .

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.com

    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist

      - name: Deploy to Azure
        uses: azure/webapps-deploy@v3
        with:
          app-name: 'myapp-production'
          publish-profile: ${{ secrets.AZURE_PUBLISH_PROFILE_PROD }}
          package: .
```

### Matrix Strategy

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20, 22]
        exclude:
          - os: windows-latest
            node-version: 18
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

### Reusable Workflows

```yaml
# .github/workflows/reusable-deploy.yml
name: Reusable Deploy
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      app-name:
        required: true
        type: string
    secrets:
      azure-credentials:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: azure/login@v2
        with:
          creds: ${{ secrets.azure-credentials }}
      - run: echo "Deploying to ${{ inputs.environment }}"

# Calling reusable workflow
# .github/workflows/main.yml
jobs:
  deploy-staging:
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: staging
      app-name: myapp-staging
    secrets:
      azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
```

### Docker Build and Push (GitHub Actions)

```yaml
name: Docker Build

on:
  push:
    branches: [main]

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:${{ github.sha }}
            ghcr.io/${{ github.repository }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## Section 6: CI/CD Best Practices

### Pipeline Design Patterns

1. **Trunk-based development** — Short-lived feature branches, frequent merges to main
2. **Pipeline as code** — All pipeline definitions in version control (YAML/Groovy)
3. **Single source of truth** — One pipeline file per service, no manual configurations
4. **Immutable artifacts** — Build once, promote same artifact across environments
5. **Environment parity** — Staging mirrors production as closely as possible

### Security in Pipelines

6. **Secrets never in code** — Use Key Vault, variable groups, or Vault
7. **Least privilege service connections** — Each connection has minimal RBAC
8. **Signed commits and images** — Verify source integrity
9. **SAST in every PR** — Static analysis blocks PRs with critical findings
10. **Dependency scanning** — Automated SCA on every build
11. **Container image scanning** — Scan for CVEs before pushing to registry
12. **Branch protection** — Require reviews, status checks, and signed commits
13. **Audit all pipeline changes** — Track who changed what, when

### Speed Optimization

14. **Cache aggressively** — node_modules, .m2, NuGet packages, Docker layers
15. **Parallelize tests** — Split test suites across multiple jobs
16. **Incremental builds** — Only rebuild changed modules
17. **Optimized Docker builds** — Multi-stage builds, layer ordering, .dockerignore
18. **Skip unchanged** — Use path filters to skip irrelevant pipelines
19. **Self-hosted agents** — Pre-warmed with tools and caches

### Testing Strategies

20. **Test pyramid** — Many unit tests, fewer integration, few E2E
21. **Fail fast** — Run fastest tests first, abort on failure
22. **Test in parallel** — Matrix strategies for multi-platform testing
23. **Flaky test quarantine** — Isolate unreliable tests, fix separately
24. **Contract testing** — Verify API contracts between microservices
25. **Chaos testing in pipeline** — Run reliability tests before production

---

## Section 7: Lessons Learned / Pro Tips

1. **Start simple, iterate** — Don't build a complex pipeline on day one. Start with build → test → deploy and add stages as needed.

2. **Template everything** — If you have more than 3 similar pipelines, create templates. In GCC enterprise projects, this saves weeks of maintenance.

3. **Version your templates** — Use tags/branches for templates so consuming pipelines don't break on template updates.

4. **Monitor pipeline metrics** — Track build times, failure rates, queue times. Set alerts for regressions.

5. **Use deployment slots/blue-green** — Never deploy directly to production. Always use a staging slot with swap.

6. **Gate production with multiple signals** — Approval + automated health check + business hours restriction.

7. **Keep pipelines under 15 minutes** — If longer, optimize caching, parallelism, or split into multiple pipelines.

8. **Don't use `latest` tags in production** — Always use specific version tags for reproducibility.

9. **Separate CI from CD** — Build/test pipeline triggers deploy pipeline. Clearer ownership and debugging.

10. **Use workload identity federation** — Avoid storing service principal secrets. Use OIDC-based authentication for Azure.

11. **Test your pipeline locally** — Use tools like `act` (GitHub Actions) or pipeline validation CLI before pushing.

12. **Document pipeline architecture** — Create diagrams showing pipeline flow, dependencies, and environments.

13. **Implement rollback automation** — Every deploy pipeline should have an automated rollback mechanism.

14. **Use semantic versioning in artifacts** — Tag builds with semver for clear release tracking.

15. **Rotate secrets before they expire** — Automate secret rotation and alert 30 days before expiry.

16. **Keep agents stateless** — Don't rely on agent state between runs. Each build should be self-contained.

17. **Use condition/rules to avoid waste** — Don't run deploy stages on PRs, don't run full test suites on docs changes.

18. **Implement pipeline notifications wisely** — Notify on failure (always), success only when fixing a broken build. Avoid notification fatigue.

---

## Section 8: Interview Questions

### CI/CD Concepts

**Q1: What is the difference between Continuous Delivery and Continuous Deployment?**

> **A:** Continuous Delivery means code is always in a deployable state and can be released to production at any time with a manual approval gate. Continuous Deployment goes further — every change that passes all automated tests is automatically deployed to production without human intervention. Most GCC enterprise organizations use Continuous Delivery because they require manual approvals for compliance.

**Q2: Explain the deployment strategies you've used and when to use each.**

> **A:** 
> - **Rolling:** Gradual replacement of instances. Use for stateless applications where brief mixed-version traffic is acceptable.
> - **Blue-Green:** Two identical environments with atomic traffic switch. Use when you need instant rollback and can afford double infrastructure.
> - **Canary:** Progressive traffic shifting (5% → 25% → 100%). Use for critical services where you want to validate with real traffic before full rollout.
> - **Feature Flags:** Deploy code but toggle features. Use when you want to decouple deployment from release, or do A/B testing.

**Q3: What are DORA metrics and why do they matter?**

> **A:** DORA (DevOps Research and Assessment) metrics measure software delivery performance:
> - Deployment Frequency — How often you deploy
> - Lead Time for Changes — Time from commit to production
> - Change Failure Rate — Percentage of deployments causing failures
> - Time to Restore Service — How quickly you recover from failures
> 
> They matter because they're empirically linked to organizational performance and provide objective measurement of DevOps maturity.

**Q4: How do you handle database migrations in a CI/CD pipeline?**

> **A:** 
> 1. Use migration tools (Flyway, Liquibase, EF Core Migrations)
> 2. Make migrations backward-compatible (additive changes first)
> 3. Run migrations as a separate pipeline step before application deployment
> 4. Use the "expand and contract" pattern: add new column → deploy app using both → remove old column
> 5. Always have a rollback migration script
> 6. Test migrations against a copy of production data

**Q5: How do you secure secrets in CI/CD pipelines?**

> **A:**
> - Never store secrets in code or pipeline YAML
> - Use platform secret stores (Azure Key Vault, HashiCorp Vault)
> - Use variable groups marked as secret (masked in logs)
> - Implement secret rotation policies
> - Use workload identity/OIDC instead of long-lived credentials
> - Audit access to secrets
> - Limit secret scope to specific stages/environments

### Azure DevOps Specific

**Q6: Explain the difference between a deployment job and a regular job in Azure DevOps.**

> **A:** A deployment job (`- deployment:`) targets an environment and supports deployment strategies (runOnce, rolling, canary). It provides lifecycle hooks (preDeploy, deploy, routeTraffic, postRouteTraffic, on.failure, on.success) and integrates with environment approvals and checks. Regular jobs (`- job:`) don't have these capabilities and are for build/test tasks.

**Q7: How do you implement multi-stage approvals in Azure DevOps?**

> **A:** Configure approvals on Environments:
> 1. Create environments (staging, production) in Pipelines → Environments
> 2. Add approval checks: specific approvers, timeout, approval policies
> 3. Add branch control: only allow main branch
> 4. Add business hours checks if needed
> 5. Use deployment jobs targeting these environments in the YAML pipeline
> The pipeline automatically pauses and waits for approval before proceeding.

**Q8: What are extends templates and why use them?**

> **A:** Extends templates enforce pipeline structure and security. The platform/DevOps team creates a base template that all pipelines must extend. This ensures:
> - Mandatory security scanning stages
> - Required approvals
> - Standardized logging and notifications
> - Compliance requirements (important in GCC banking/government)
> 
> Teams can only customize allowed parameters, not skip required stages.

**Q9: How do you integrate Azure Key Vault with Azure DevOps pipelines?**

> **A:** Two approaches:
> 1. **Variable Group linked to Key Vault:** Create a variable group in Library, link it to a Key Vault via service connection. Secrets are fetched at pipeline start.
> 2. **AzureKeyVault@2 task:** Fetch secrets at runtime in a specific step:
> ```yaml
> - task: AzureKeyVault@2
>   inputs:
>     azureSubscription: 'My-SC'
>     KeyVaultName: 'my-keyvault'
>     SecretsFilter: 'secret1,secret2'
> ```
> The variable group approach is simpler; the task approach gives more control over when secrets are fetched.

**Q10: Explain self-hosted agents vs Microsoft-hosted agents. When would you use each?**

> **A:** 
> - **Microsoft-hosted:** Fresh VM per job, maintained by Microsoft, no setup required. Use for standard builds without special requirements.
> - **Self-hosted:** You manage the infrastructure. Use when you need: access to private networks (common in GCC), specific software/hardware, persistent caches for faster builds, data residency compliance, or high-frequency builds (no queue wait).
> 
> In GCC, self-hosted agents are common because many organizations require data to stay within UAE/KSA regions and need access to private VNets.

**Q11: How do you implement a canary deployment strategy in Azure DevOps?**

> **A:** Use a deployment job with the canary strategy:
> ```yaml
> strategy:
>   canary:
>     increments: [10, 20, 50]
>     deploy:
>       steps: [deploy canary version]
>     routeTraffic:
>       steps: [shift traffic percentage]
>     postRouteTraffic:
>       steps: [validate metrics, health checks]
>     on:
>       failure:
>         steps: [rollback]
> ```
> This progressively shifts traffic (10% → 20% → 50% → 100%) with validation between each increment.

**Q12: How do you share pipeline logic across multiple projects in Azure DevOps?**

> **A:** Use a dedicated repository for pipeline templates:
> 1. Create a `pipeline-templates` repo in a shared project
> 2. Define step, job, and stage templates
> 3. Reference them from other repos using `resources.repositories`
> ```yaml
> resources:
>   repositories:
>     - repository: templates
>       type: git
>       name: SharedProject/pipeline-templates
>       ref: refs/tags/v1.0
> steps:
>   - template: build.yml@templates
> ```

### GitLab CI Specific

**Q13: Explain the difference between `cache` and `artifacts` in GitLab CI.**

> **A:** 
> - **Cache:** Persists between pipeline runs, stored on the runner. Used for dependencies (node_modules, pip packages). Not guaranteed to be available (may be evicted). Speeds up builds by avoiding re-downloads.
> - **Artifacts:** Passed between jobs in the SAME pipeline, stored on GitLab server. Used for build outputs (compiled code, test reports). Guaranteed availability within the pipeline. Has expiration settings.

**Q14: What is a DAG in GitLab CI and how does `needs:` work?**

> **A:** DAG (Directed Acyclic Graph) allows jobs to run as soon as their dependencies complete, regardless of stage ordering. Using `needs:`, a job in stage 3 can start as soon as its dependent job in stage 1 finishes, without waiting for all stage 2 jobs. This significantly reduces pipeline duration for independent workflows.

**Q15: How do parent-child pipelines work in GitLab?**

> **A:** A parent pipeline triggers child pipelines defined in separate YAML files. Benefits:
> - Break complex pipelines into manageable pieces
> - Different child pipelines for different components (frontend, backend, infra)
> - Children can be generated dynamically
> - `strategy: depend` makes parent wait for child completion

### Scenario-Based Questions

**Q16: Your pipeline takes 45 minutes. How would you reduce it to under 15 minutes?**

> **A:** 
> 1. **Analyze:** Identify the slowest stages (use pipeline analytics)
> 2. **Parallelize:** Run unit, integration, and lint tests in parallel jobs
> 3. **Cache:** Cache dependencies (npm ci with cache, Docker layer caching)
> 4. **Skip unnecessary work:** Path filters to skip unchanged services
> 5. **Self-hosted agents:** Eliminate queue wait, persistent caches
> 6. **Test splitting:** Distribute tests across parallel runners
> 7. **Incremental builds:** Only compile changed modules
> 8. **Optimize Docker builds:** Multi-stage, .dockerignore, layer ordering
> 9. **Remove flaky retries:** Fix flaky tests instead of retrying

**Q17: A deployment to production failed at 2 AM. Walk through your incident response.**

> **A:**
> 1. **Detect:** Monitoring alerts trigger PagerDuty
> 2. **Assess:** Check deployment pipeline logs, identify the failure point
> 3. **Rollback:** Execute automated rollback (swap slots, revert K8s deployment)
> 4. **Verify:** Confirm rollback restored service health
> 5. **Communicate:** Update status page, notify stakeholders
> 6. **Root cause:** Next day — analyze logs, identify fix
> 7. **Prevent:** Add missing test/gate that would have caught this
> 8. **Blameless post-mortem:** Document lessons learned

**Q18: How would you design a CI/CD pipeline for a microservices architecture with 20 services?**

> **A:**
> - **Monorepo approach:** Single repo with path-based triggers per service
> - **Multi-repo approach:** Each service has its own pipeline
> - **Shared templates:** Common build/deploy templates in a shared repo
> - **Service mesh routing:** Canary per service independently
> - **Contract testing:** Verify inter-service contracts in CI
> - **Dependency graph:** Auto-deploy downstream services when shared libraries change
> - **Environment management:** Ephemeral environments per PR for isolated testing

**Q19: You need to implement CI/CD for a banking application in the UAE with strict compliance requirements. What considerations apply?**

> **A:**
> - **Data residency:** All build agents and artifacts in UAE Azure regions
> - **Audit trail:** Complete logging of all pipeline activities
> - **Separation of duties:** Developers can't approve their own deployments
> - **4-eyes principle:** At least 2 approvers for production
> - **Change management:** Integration with ServiceNow/ITSM
> - **Security scanning:** Mandatory SAST, DAST, SCA in every pipeline
> - **Encrypted artifacts:** At-rest and in-transit encryption
> - **Private agents:** No internet-facing build infrastructure
> - **Branch policies:** Protected branches, signed commits
> - **Compliance gates:** Automated compliance checks before deploy

**Q20: Your team wants to migrate from Jenkins to Azure DevOps. What's your migration plan?**

> **A:**
> 1. **Inventory:** Document all Jenkins jobs, plugins, credentials, shared libraries
> 2. **Map concepts:** Jenkins agent → ADO pool, Jenkinsfile → YAML, shared libraries → templates
> 3. **Start with CI:** Migrate build/test pipelines first (lower risk)
> 4. **Parallel run:** Run both systems simultaneously during migration
> 5. **Migrate secrets:** Move credentials to Azure Key Vault / variable groups
> 6. **Convert pipelines:** Translate Groovy to YAML (no 1:1 mapping — redesign where needed)
> 7. **CD migration:** Move deployment pipelines last (higher risk)
> 8. **Decommission:** Remove Jenkins after full validation period (2-4 weeks)
> 9. **Training:** Upskill team on Azure DevOps YAML pipeline authoring

**Q21: How do you handle pipeline failures due to flaky tests?**

> **A:**
> - **Short-term:** Mark known flaky tests with `allow_failure: true` (GitLab) or `continueOnError: true` (ADO)
> - **Track flakiness:** Monitor test pass rates over time
> - **Quarantine:** Move consistently flaky tests to a separate non-blocking job
> - **Fix root cause:** Investigate timing issues, external dependencies, shared state
> - **Retry logic:** Limited retries for infrastructure-related failures (not test logic)
> - **Never ignore:** Flaky tests erode confidence in the pipeline — fix or remove them

**Q22: Explain how you implement GitOps with Azure DevOps.**

> **A:** GitOps uses Git as the single source of truth for infrastructure and application state:
> 1. Kubernetes manifests stored in Git
> 2. Pipeline updates manifests (image tags) via PR
> 3. ArgoCD/Flux watches the Git repo and syncs to cluster
> 4. Azure DevOps pipeline: Build → Push image → Update manifest repo → ArgoCD syncs
> 5. Rollback = Git revert on the manifest repo
> 6. Audit trail via Git history

**Q23: How do you handle multi-region deployments in a pipeline?**

> **A:**
> - Use matrix strategy to deploy to multiple regions in parallel
> - Implement progressive rollout: primary region first, then secondary
> - Health check each region before proceeding
> - Use traffic manager/front door for traffic shifting
> - Have region-specific variable groups for configuration
> - Implement automatic failover if one region's deployment fails

**Q24: What is pipeline observability and how do you implement it?**

> **A:** Pipeline observability provides insight into pipeline health and performance:
> - **Metrics:** Build duration, queue time, failure rate, test pass rate
> - **Logging:** Structured logs in each step, correlation IDs
> - **Tracing:** End-to-end trace from commit to deployment
> - **Dashboards:** Azure DevOps analytics, custom Power BI dashboards
> - **Alerts:** Notify on build time regression, increased failure rate
> - **Tools:** Azure DevOps Analytics, Grafana, Datadog CI Visibility

**Q25: Design a pipeline that supports hotfix deployments bypassing normal flow.**

> **A:**
> ```yaml
> trigger:
>   branches:
>     include:
>       - main
>       - hotfix/*
> 
> stages:
>   - stage: Build
>     # Always runs
>   - stage: Test
>     # For hotfix: only critical tests
>     jobs:
>       - job: CriticalTests
>         condition: contains(variables['Build.SourceBranch'], 'hotfix')
>       - job: FullTests
>         condition: not(contains(variables['Build.SourceBranch'], 'hotfix'))
>   - stage: DeployProd
>     # For hotfix: skip staging, direct to prod with approval
>     condition: |
>       and(succeeded(), 
>           or(
>             eq(variables['Build.SourceBranch'], 'refs/heads/main'),
>             contains(variables['Build.SourceBranch'], 'hotfix')
>           ))
> ```
> Hotfix branches skip staging and full test suites but still require approval. Document this as an exception process in your change management policy.

**Q26: How do you ensure consistency between environments?**

> **A:**
> - **Infrastructure as Code:** Same Terraform/ARM templates across environments
> - **Same artifact:** Promote the exact same Docker image/package (no rebuilding)
> - **Configuration separation:** Only configuration differs (via variable groups per environment)
> - **Environment-as-code:** Define environments and their configs in Git
> - **Drift detection:** Scheduled pipeline to detect configuration drift
> - **Immutable infrastructure:** Replace VMs/containers instead of updating in-place

---

## Section 9: Free Resources with Links

1. **Microsoft Learn — Azure DevOps Learning Paths**  
   https://learn.microsoft.com/en-us/training/browse/?products=azure-devops  
   Official Microsoft training modules covering all Azure DevOps services.

2. **Azure DevOps Labs (Hands-on)**  
   https://azuredevopslabs.com/  
   Step-by-step labs for Azure DevOps features including pipelines, repos, and boards.

3. **GitLab CI/CD Documentation**  
   https://docs.gitlab.com/ee/ci/  
   Comprehensive official documentation with tutorials and examples.

4. **Jenkins Pipeline Documentation**  
   https://www.jenkins.io/doc/book/pipeline/  
   Official Jenkins pipeline syntax and best practices guide.

5. **GitHub Actions Documentation**  
   https://docs.github.com/en/actions  
   Complete reference for GitHub Actions workflows, syntax, and marketplace.

6. **The DevOps Handbook (Summary Resources)**  
   https://itrevolution.com/product/the-devops-handbook-second-edition/  
   Core reference for CI/CD principles and organizational practices.

7. **DORA State of DevOps Reports**  
   https://dora.dev/research/  
   Annual reports with data-driven insights on DevOps performance metrics.

8. **Kubernetes Deployment Strategies (Blog)**  
   https://blog.container-solutions.com/kubernetes-deployment-strategies  
   Visual guide to Blue-Green, Canary, and Rolling deployments on K8s.

9. **Azure DevOps YAML Schema Reference**  
   https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/  
   Complete YAML schema documentation for Azure Pipelines.

10. **GitLab CI/CD Examples Repository**  
    https://gitlab.com/gitlab-org/gitlab/-/tree/master/lib/gitlab/ci/templates  
    Real-world GitLab CI templates for various languages and frameworks.

11. **Terraform Azure DevOps Provider**  
    https://registry.terraform.io/providers/microsoft/azuredevops/latest/docs  
    Manage Azure DevOps resources (projects, pipelines, repos) with Terraform.

12. **CI/CD Pipeline Security Best Practices (OWASP)**  
    https://owasp.org/www-project-devsecops-guideline/  
    OWASP guidelines for securing CI/CD pipelines and implementing DevSecOps.

13. **Azure Architecture Center — CI/CD Patterns**  
    https://learn.microsoft.com/en-us/azure/architecture/guide/devops/cicd-pipeline  
    Microsoft's reference architectures for CI/CD with Azure services.

14. **YouTube: Azure DevOps Tutorial for Beginners (freeCodeCamp)**  
    https://www.youtube.com/results?search_query=azure+devops+full+course  
    Free video courses covering Azure DevOps end-to-end.

---

## Quick Reference Card

| Platform | Config File | Key Command |
|----------|------------|-------------|
| Azure DevOps | `azure-pipelines.yml` | `az pipelines run` |
| GitLab CI | `.gitlab-ci.yml` | `gitlab-runner exec` |
| Jenkins | `Jenkinsfile` | `jenkins-cli build` |
| GitHub Actions | `.github/workflows/*.yml` | `gh workflow run` |

---

*End of Month 4 Study Material — CI/CD Pipelines*  
*Next: Month 5 — Monitoring, Logging & Observability*
