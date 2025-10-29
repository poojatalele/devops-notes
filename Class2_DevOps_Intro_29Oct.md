# Introduction to DevOps

## Understanding DevOps
DevOps represents a fusion of **Development (Dev)** and **Operations (Ops)**, encompassing a collection of practices, tools, and cultural philosophies.
Its primary objective is to accelerate the delivery of applications and services while enhancing efficiency and reliability.

> **Core Concept:**
> DevOps acts as a bridge between software development and IT operations, fostering a culture of collaboration to **build, test, and deploy software with greater speed and stability**.

## The Value of DevOps
- **Rapid Delivery:** Faster release cycles for both new features and updates.
- **Enhanced Reliability:** Minimized downtime and improved system stability.
- **Feedback Loops:** continuous feedback leads to ongoing improvement.
- **Automation:** Reduces manual intervention and human error.
- **Better Collaboration:** Breaks down silos between teams.
- **Resilience:** Faster recovery times from incidents.

## Key Principles
1.  **Unified Collaboration:** Seamless teamwork between Dev and Ops.
2.  **Automated Workflows:** Utilizing CI/CD pipelines to handle testing and deployment automatically.
3.  **Continuous Integration (CI):** Frequent code merges accompanied by automated testing.
4.  **Continuous Delivery (CD):** Deployment to production is automated and reliable.
5.  **Observability:** Continuous monitoring and real-time feedback to maintain system health.

---

# Software Development Life Cycle (SDLC)

## Phases of SDLC
1.  **Requirement Gathering:**
    - Identifying business needs and project scope.
    - Analyzing what needs to be built.

2.  **System Design:**
    - Architecting the system and defining technical specifications.

3.  **Implementation (Coding):**
    - Writing the actual code based on the design.
    - Adhering to coding standards.

4.  **Testing:**
    - Verifying software through unit, integration, and system tests.
    - Ensuring the product is free of bugs.

5.  **Deployment:**
    - releasing the application to live environments.
    - Implementing strategies like **Blue-Green** or **Canary** for zero-downtime updates.

6.  **Maintenance:**
    - Ongoing monitoring of performance.
    - Bug fixing and rolling out patches.

---

# DevOps in the Lifecycle

| SDLC Stage | DevOps Activity | Common Tools |
| :--- | :--- | :--- |
| **Plan** | Define objectives and scope. | Jira, Trello |
| **Code** | Version control and code management. | Git, GitHub, GitLab |
| **Build** | Compiling source code into artifacts. | Maven, Gradle, Jenkins |
| **Test** | Automated validation of functionality. | JUnit, Selenium |
| **Release** | Managing release artifacts. | Jenkins, Spinnaker |
| **Deploy** | pushing changes to production environments. | Kubernetes, Docker |
| **Operate** | Managing infrastructure and configuration. | Ansible, Chef, Puppet |
| **Monitor** | Observing metrics and logs. | Prometheus, Grafana, ELK |

---

# DevSecOps: Integrating Security

**DevSecOps** embeds security practices into every phase of the DevOps operation, rather than treating it as an afterthought. This approach, known as **Shift-Left Security**, catches vulnerabilities early to save time and resources.

## Key Security Practices
-   **SAST (Static Application Security Testing):**
    -   Analyzing source code for flaws before compilation.
    -   *Tools:* SonarQube, Checkmarx.

-   **DAST (Dynamic Application Security Testing):**
    -   Testing the application in a running state for security gaps.
    -   *Tools:* OWASP ZAP, Burp Suite.

-   **SCA (Software Composition Analysis):**
    -   Identifying vulnerabilities in open-source dependencies.
    -   *Tools:* JFrog Xray, Black Duck.

-   **Linting:**
    -   Enforcing coding standards and syntax correctness.
    -   *Tools:* ESLint, Pylint.

-   **Container Security:**
    -   Scanning Docker images to ensure they are secure before deployment.

---

# Infrastructure as Code (IaC)

IaC is the practice of managing and provisioning infrastructure through **machine-readable definition files**, rather than physical hardware configuration or interactive configuration tools.

## Common IaC Tools
-   **Terraform:** A cloud-agnostic tool for building, changing, and versioning infrastructure safely.
-   **AWS CloudFormation:** AWS's native service for modeling and setting up Amazon Web Services resources.
-   **Configuration Management:** Tools like **Ansible, Chef, and Puppet** that automate software provisioning and configuration integration.
-   **SDKs:** Libraries like **Boto3** (Python) for programmatic control of AWS services.

---

# AWS VPC (Virtual Private Cloud)

## Overview
A **VPC** is your own logically isolated network within the AWS cloud. It allows you to provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define.

## Core Components

| Component | Function |
| :--- | :--- |
| **VPC** | The foundational private network container. |
| **Subnet** | A subdivision of the VPC IP range. |
| **Public Subnet** | A subnet with direct access to the internet via an IGW. |
| **Private Subnet** | A subnet with no direct internet access, used for backend systems. |
| **Internet Gateway (IGW)** | The gateway connecting the VPC to the internet. |
| **NAT Gateway** | Allows instances in a private subnet to connect to the internet (e.g., for updates) but prevents the internet from initiating connections with those instances. |
| **Route Table** | A set of rules (routes) that determine where network traffic is directed. |
| **Security Group** | act as a virtual firewall for your **instance** to control inbound and outbound traffic. |
| **Network ACL** | An optional layer of security for your **subnet** that acts as a firewall for controlling traffic in and out of one or more subnets. |

> **Security Layers:**
> *   **Instance Level:** Controlled by Security Groups (Stateful).
> *   **Subnet Level:** Controlled by Network ACLs (Stateless).
> *   **Network Level:** Controlled by Route Tables and VPC isolation.

---

# Deployment & Artifacts

## Artifact Management
-   **Artifact Repositories:** Systems to store binary outputs of the build process (like `.jar` files or container images).
-   **Examples:** JFrog Artifactory, Sonatype Nexus.

## Deployment Strategies
-   **Blue-Green:** Maintains two identical environments. Traffic is switched from the old "Blue" version to the new "Green" version, allowing instant rollback.
-   **Canary:** Introducing the new version to a small subset of users before a full rollout to limit the impact of potential issues.
-   **Rolling:** Updating instances incrementally, ensuring that some instances are always up to serve traffic.
