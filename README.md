# dotnet-iis-deployment
CI/CD pipeline for IIS-hosted .NET applications using Jenkins, SonarQube, Nexus Repository, and automated IIS deployment through Ansible.



The pipeline automates: 
- Source code checkout
- Application build and publish
- Static code analysis using SonarQube
- Artifact packaging and versioning
- Artifact upload to Nexus Repository
- Automated IIS deployment using Ansible
- Health check validation
- Rollback to previous stable artifact on deployment failure
  
- The deployment workflow was designed to support reusable deployments across Dev and QA environments using dynamic runtime variables passed from Jenkins to Ansible playbooks.
- In addition to CI/CD automation, SQL Server environments were configured using SQL Server Management Studio (SSMS) and SQL Server Configuration Manager.
- Database access and permissions were managed by assigning: - DB Owner privileges to the Data Team - Read, Write, and Execute permissions to Dev and QA teams



# Technologies Used | Technology | Purpose | 
|---|---| 
| Jenkins | CI/CD Pipeline Automation | 
| Ansible | IIS Deployment Automation | 
| SonarQube | Static Code Analysis | 
| Nexus Repository | Artifact Management | 
| IIS | Application Hosting | 
| WinRM | Windows Remote Management | 
| SSMS | SQL Server Administration | 
| SQL Server Configuration Manager | SQL Server Configuration |
| .NET 8 | Application Framework | 
| Groovy | Jenkins Pipeline Scripting |


# CI/CD Workflow Architecture 
Developer Commit 
↓ 
Git Repository 
↓ 
Jenkins Pipeline 
↓ Windows Build Agent 
├── Restore Dependencies 
├── Build Application 
├── SonarQube Analysis 
├── Package Artifact 
└── Upload Artifact to Nexus 
↓ 
Ubuntu Deployment Agent 
↓ 
Ansible Playbook Execution 
↓ 
Windows IIS Server 
↓ 
Application Deployment 
↓ 
Health Check Validation


# CI/CD Pipeline Stages
## 1. Checkout Stage

The pipeline starts by pulling the latest application source code from the Git repository using the configured branch and credentials.

*Activities
- Workspace cleanup
- Source code checkout
- Branch verification

---

## 2. Build & Publish Stage

The .NET application is restored, compiled, and published in Release mode.

*Activities
- Dependency restoration using `dotnet restore`
- Application build and publish using `dotnet publish`
- Generation of deployable publish artifacts

---

## 3. SonarQube Static Analysis

Static code analysis is integrated into the pipeline using SonarQube to identify:
- Bugs
- Vulnerabilities
- Code smells
- Duplicate code blocks

*Activities
- Sonar scanner initialization
- Build analysis
- Code coverage collection
- Quality validation

> Quality gate failures occurred due to unavailable test cases, but SonarQube was actively used for static analysis and code quality monitoring.

---

## 4. Artifact Packaging

After successful build and analysis, the published application files are compressed into versioned ZIP artifacts.

*Activities
- ZIP packaging of published output
- Timestamp-based artifact versioning
- Build traceability support

---

## 5. Nexus Repository Upload

Packaged artifacts are uploaded to Nexus Repository for centralized artifact management and deployment consistency.

*Activities
- Artifact version management
- Centralized storage
- Deployment-ready artifact distribution

---

## 6. Automated IIS Deployment using Ansible

Deployment automation is executed from an Ubuntu-based Jenkins agent using reusable Ansible playbooks targeting Windows IIS servers through WinRM.

*Activities
- Dynamic runtime variable passing
- IIS website creation
- IIS application pool management
- Artifact deployment
- Automated restart handling

---

## 7. Health Check Validation

After deployment, automated health validation is performed to verify application accessibility and deployment success.

* Activities
- Endpoint verification
- HTTP status validation
- Deployment confirmation

---

## 8. Rollback Mechanism

If deployment validation fails, the playbook automatically restores the previously stable artifact version.

* Activities
- Automatic rollback execution
- Previous artifact restoration
- Failed artifact cleanup
- Service recovery


  # Pipeline Execution

![Jenkins Pipeline](screenshots/jenkins-View.png)

# SonarQube Analysis

![SonarQube Dashboard](screenshots/sonarqube-dashboard.png)

# Nexus Artifact Repository

![Nexus Repository](screenshots/nexus-artifact.png)
