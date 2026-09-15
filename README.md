<div align="center">

# Multi-Environment Java Application Deployment using AWS EC2

![AWS](https://img.shields.io/badge/AWS-EC2%20%7C%20AMI-orange)
![Java](https://img.shields.io/badge/Java-17-blue)
![Tomcat](https://img.shields.io/badge/Apache%20Tomcat-Application%20Server-yellow)
![Maven](https://img.shields.io/badge/Maven-Build-red)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-black)
![Linux](https://img.shields.io/badge/Linux-Linux-lightgrey)
</div>

## 📌 Project Overview

This project demonstrates an automated approach for deploying a Java web application across multiple environments using **AWS EC2, a Golden AMI, Apache Tomcat, Maven, and GitHub Actions**.

The main objective is to eliminate repetitive server configuration. Java and Apache Tomcat are installed and configured on a base EC2 instance. A **Golden AMI** is then created from this configured instance. Additional EC2 instances can be launched from the Golden AMI, so the required Java/Tomcat runtime is already available on every new server.

GitHub Actions is used to automate the application build and deployment process across different environments such as **DEV, TEST, PRE-PROD, and PROD**.

---

## 🎯 Objectives

- Automate Java application deployment.
- Standardize EC2 server configuration using a Golden AMI.
- Avoid manually installing Java and Tomcat on every new EC2 instance.
- Build Java applications using Maven.
- Package the application as a WAR file.
- Deploy the WAR file to Apache Tomcat.
- Support environment-specific deployments.
- Implement CI/CD using GitHub Actions.
- Reduce manual deployment effort and configuration drift.

---

## 🏗️ Architecture

```text
                         ┌──────────────────┐
                         │     Developer    │
                         └────────┬─────────┘
                                  │
                                  │ git push
                                  ▼
                         ┌──────────────────┐
                         │     GitHub       │
                         │   Repository     │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │ GitHub Actions   │
                         │     CI/CD        │
                         └────────┬─────────┘
                                  │
                         ┌────────▼─────────┐
                         │  Maven Build     │
                         │  Unit Tests      │
                         │  Generate WAR    │
                         └────────┬─────────┘
                                  │
                   ┌──────────────┼──────────────┐
                   │              │              │
                   ▼              ▼              ▼
                DEV ENV        TEST ENV      PRE-PROD ENV
                   │              │              │
                   └──────────────┼──────────────┘
                                  │
                           Production Approval
                                  │
                                  ▼
                             PROD ENV


                 ─────── AWS EC2 GOLDEN AMI ───────

                    Base EC2 / Golden Image Server
                                  │
                  ┌───────────────┼───────────────┐
                  │               │               │
                Java           Tomcat        Configuration
                  │               │               │
                  └───────────────┼───────────────┘
                                  │
                                  ▼
                         Create Golden AMI
                                  │
                                  ▼
                    ┌─────────────────────────┐
                    │       Golden AMI        │
                    │ Java + Tomcat + Config  │
                    └────────────┬────────────┘
                                 │
                 ┌───────────────┼───────────────┐
                 ▼               ▼               ▼
              EC2-01          EC2-02          EC2-03
              Tomcat          Tomcat          Tomcat
```

> **Note:** The architecture diagram represents the target project workflow. Only components actually implemented in the repository should be described as implemented.

---

## 🔑 Key Concept: Golden AMI

A Golden AMI is a pre-configured Amazon Machine Image used as a standard base image for launching EC2 instances.

### Without Golden AMI

```text
New EC2
   ↓
Install Java
   ↓
Install Tomcat
   ↓
Configure Tomcat
   ↓
Install Dependencies
   ↓
Deploy Application
```

This process has to be repeated for every server.

### With Golden AMI

```text
Base EC2
   ↓
Install Java
   ↓
Install Tomcat
   ↓
Configure Server
   ↓
Create Golden AMI
   ↓
Launch New EC2
   ↓
Java + Tomcat already available
```

### Benefits

- Faster EC2 provisioning
- Consistent server configuration
- Reduced manual work
- Reduced configuration drift
- Repeatable infrastructure
- Easier horizontal scaling

---

## 🛠️ Technologies Used

| Category | Technologies |
|---|---|
| Cloud | AWS |
| Compute | Amazon EC2 |
| Machine Image | AWS AMI |
| Application Server | Apache Tomcat |
| Programming | Java |
| Build Tool | Apache Maven |
| CI/CD | GitHub Actions |
| Version Control | Git, GitHub |
| Operating System | Linux |
| Application Artifact | WAR |
| Security | AWS IAM / GitHub Secrets |


---

# 🚀 Implementation Steps

## 1. Launch the Base EC2 Instance

Create an EC2 instance that will be used as the base server.

Example:

```text
Name: tomcat-golden-image
OS: Amazon Linux
Instance Type: t3.micro
```

Configure the required security group.

For a basic lab:

```text
SSH   22
HTTP  80
Tomcat 8080
```

> In a production environment, restrict inbound access to trusted sources and avoid exposing Tomcat directly to the internet when possible.

---

## 2. Install Maven

```
sudo dnf insall maven
```
---

## 3. Install Java

Example for Amazon Linux:

```bash
sudo dnf update -y
sudo dnf install java-17-amazon-corretto -y
```

Verify:

```bash
java -version
```

Expected result:

```text
openjdk version "17..."
```

---

## 4. Install Apache Tomcat

Download and install the required Tomcat version.

Example installation location:

```text
/opt/tomcat
```

Verify that Tomcat is running:

```bash
sudo systemctl status tomcat
```

Test the Tomcat server:

```text
http://<EC2_PUBLIC_IP>:8080
```

---

## 5. Configure Tomcat

Configure:

- Tomcat service
- Environment variables
- Required permissions
- Application deployment directory
- JVM settings if required
- Logging
- Security settings

Example:

```text
/opt/tomcat
├── bin
├── conf
├── logs
├── webapps
└── lib
```

---

## 6. Create the Golden AMI

After Java, Tomcat, dependencies, and server configuration are completed:

```text
EC2 Base Instance
       ↓
Verify Java
       ↓
Verify Tomcat
       ↓
Verify Configuration
       ↓
Create AMI
```

Suggested AMI naming convention:

```text
java-tomcat-golden-ami-v1
java-tomcat-golden-ami-v2
```

The AMI becomes the standard image for new EC2 instances.

---

## 7. Launch New EC2 Instances from the Golden AMI

Select the Golden AMI when launching a new EC2 instance.

Example:

```text
Golden AMI
    │
    ├── EC2-DEV
    ├── EC2-TEST
    ├── EC2-PREPROD
    └── EC2-PROD
```

The new instances inherit the software and configuration captured in the AMI.

---

# 🔄 CI/CD Workflow

The GitHub Actions pipeline follows this general flow:

```text
Developer
    ↓
Git Push
    ↓
GitHub Repository
    ↓
GitHub Actions
    ↓
Checkout Source Code
    ↓
Setup Java
    ↓
Maven Build
    ↓
Run Tests
    ↓
Generate WAR
    ↓
Deploy to DEV
    ↓
Deploy to TEST
    ↓
Approval
    ↓
Deploy to PRE-PROD
    ↓
Approval
    ↓
Deploy to PROD
```

---

## 📦 Maven Build

Build the application:

```bash
mvn clean package
```

The WAR file will normally be generated under:

```text
target/
```

Example:

```text
target/my-java-application.war
```

---

## 🚀 Tomcat WAR Deployment

Copy the WAR file into the Tomcat deployment directory:

```bash
cp target/my-java-application.war /opt/tomcat/webapps/
```

Tomcat will deploy the application.

Application URL:

```text
http://<EC2_PUBLIC_IP>:8080/train-ticket-reservation
```

---

# 🌎 Environment Strategy

The project uses separate deployment environments:

```text
DEV
 ↓
TEST
 ↓
PRE-PROD
 ↓
PROD
```

### DEV

Used for developer validation and initial deployment testing.

### TEST

Used for application testing and integration validation.

### PRE-PROD

Used to validate the application in an environment close to production.

### PROD

Used for the final production release.

Production deployments should be protected with an approval gate.

---

# 📊 Monitoring and Troubleshooting

Useful commands:

### Check Java

```bash
java -version
```

### Check Tomcat

```bash
sudo systemctl status tomcat
```

### Restart Tomcat

```bash
sudo systemctl restart tomcat
```

### Check Tomcat logs

```bash
sudo tail -f /opt/tomcat/logs/catalina.out
```

### Check listening ports

```bash
sudo ss -lntp
```

### Check running processes

```bash
ps -ef | grep tomcat
```

---

# 🧪 Validation

After launching an EC2 instance from the Golden AMI, validate:

```text
✓ EC2 instance launched successfully
✓ Java is installed
✓ Java version is correct
✓ Tomcat is installed
✓ Tomcat service is running
✓ Required configuration is available
✓ Application WAR is deployed
✓ Application is accessible
✓ GitHub Actions deployment succeeds
```

---

# 💡 Why Golden AMI?

Without a Golden AMI, every new server requires manual configuration.

With a Golden AMI:

```text
Golden AMI
    ↓
New EC2
    ↓
Pre-configured Java
    ↓
Pre-configured Tomcat
    ↓
Ready for Application Deployment
```

This improves:

- Provisioning speed
- Consistency
- Reliability
- Repeatability
- Operational efficiency

---

# 📈 Future Enhancements

The project can be extended with:

- Terraform for Infrastructure as Code
- EC2 Launch Templates
- Auto Scaling Groups
- Application Load Balancer
- Target Groups and health checks
- AWS Systems Manager
- Amazon CloudWatch monitoring
- GitHub Actions OIDC with AWS IAM
- Blue/Green deployment
- Rolling deployment
- Automated AMI creation using Packer
- Amazon ECR and container-based deployment
- AWS Secrets Manager
- Automated rollback

---

# 🎓 DevOps Concepts Demonstrated

This project demonstrates practical experience with:

- AWS EC2
- AMI and Golden AMI concepts
- Linux administration
- Java application deployment
- Apache Tomcat
- Maven
- WAR deployment
- Git and GitHub
- GitHub Actions
- CI/CD
- Environment-based deployments
- Infrastructure standardization
- Immutable infrastructure concepts
- AWS IAM and security best practices

---

# 👨‍💻 Project Learning Outcomes

Through this project, I gained hands-on experience in designing a repeatable application deployment process, creating standardized EC2 server images, automating Java application builds and deployments, and managing application releases across multiple environments.

---

## ⭐ Author

**Sai Somasekhar**

AWS DevOps Engineer | AWS | Linux | Terraform | Docker | Kubernetes | GitHub Actions | CI/CD

---

## 📜 Disclaimer

This project is created for learning, portfolio, and DevOps practice purposes. AWS resources may incur charges. Always review and clean up unused AWS resources after completing the lab.
