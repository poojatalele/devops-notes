# CI/CD, Testing, and Deployment

## Understanding CI/CD

**CI/CD** represents the combined practices of **Continuous Integration** and either **Continuous Delivery** or **Continuous Deployment**. These practices bridge the gap between development and operation activities by enforcing automation in building, testing, and deploying applications.

---

## Continuous Integration (CI)

### What is CI?
Continuous Integration is the development practice where developers merge their changes back to the main branch as often as possible. These changes are validated by creating a build and running automated tests against the build.

### Key Objectives
-   **Early Bug Detection:** Catch issues immediately after code is committed.
-   **Integration:** Ensure that new code integrates seamlessly with the existing codebase.
-   **Automation:** Remove manual steps in the build and test process.
-   **Quality Assurance:** Maintain a high standard of code quality continuously.

### Common CI Workflow
1.  **Commit:** Developer pushes code to the repository (Git).
2.  **Fetch & Install:** The CI server (e.g., Jenkins) pulls the latest code and installs necessary dependencies.
3.  **Build:** The application is compiled.
4.  **Static Analysis:** Code is scanned for style violations and bad patterns (Linting).
5.  **Unit Tests:** Automated tests run to verify small parts of the application.
6.  **Security Scans:** SAST tools check the source code for vulnerabilities.
7.  **Dependency Check:** SCA tools scan libraries for known security issues.
8.  **Package:** The application is containerized (e.g., Docker).
9.  **Store:** The resulting artifact is pushed to a registry (e.g., ECR, Artifactory).

---

## Continuous Delivery (CD)

### What is Continuous Delivery?
Continuous Delivery is the practice where code changes are automatically prepared for a release to production. It ensures that you can release new changes to your customers quickly in a sustainable way.
> **Note:** In Continuous *Delivery*, the deployment to production is manual but the process to get there is automated.

### The CD Process
1.  **Retrieve Artifact:** Get the build from the repository.
2.  **Deploy to Staging:** Deploy to an environment (SIT/UAT) that mimics production.
3.  **Integration Testing:** Verify that system modules work together.
4.  **Performance Testing:** Assess speed and stability under load.
5.  **Dynamic Security Testing:** Run DAST against the live staging environment.
6.  **Approval:** A manual gate for final approval before going live.

---

## Continuous Deployment

**Continuous Deployment** extends Continuous Delivery by automating the final release. Every change that passes all stages of your production pipeline is released to your customers automatically. There is **no explicit manual approval**.

### Benefits
-   **Speed:** Immediate value delivery to users.
-   **Accuracy:** Eliminates human error in deployment.
-   **Feedback:** Rapid loop from development to user feedback.

**Popular Tools:** Jenkins, GitLab CI, GitHub Actions, Spinnaker, ArgoCD.

---

## The CI/CD Pipeline Visualized

```mermaid
graph TD
    Code[Source Code (Git)] --> CI_Build[CI: Build & Test]
    CI_Build --> CI_Scan[CI: Security & Linting]
    CI_Scan --> Artifact[Push Artifact]
    Artifact --> CD_Staging[CD: Deploy to Staging]
    CD_Staging --> CD_Test[CD: Integration & Perf Tests]
    CD_Test --> CD_Prod[CD: Production Release]
```

---

## Testing in DevOps

| Test Category | Description | Tools |
| :--- | :--- | :--- |
| **Unit Testing** | Testing individual components in isolation (often using mocks). | JUnit, pytest |
| **Integration Testing** | Verifying communication between different modules. | Postman, REST Assured |
| **System Testing** | End-to-end validation of the complete system. | Selenium, Cypress |
| **Performance Testing** | Stress testing the system's capacity and speed. | JMeter, Locust |
| **Security (SAST)** | White-box testing of source code. | SonarQube |
| **Security (DAST)** | Black-box testing of the running application. | OWASP ZAP |
| **SCA** | Analyzing third-party libraries for risks. | Snyk, Black Duck |

### Mocking in Tests
When unit testing, "mocking" is used to simulate the behavior of complex, real objects (like a database or an external API). this allows the unit test to focus solely on the logic of the code being tested without external dependencies.
*   **Java:** Mockito
*   **Python:** unittest.mock
*   **JS:** Sinon.js

---

# Linux Essentials

## A Brief History
-   **1969:** UNIX is born at Bell Labs.
-   **1991:** Linus Torvalds releases the Linux kernel, an open-source alternative.
*   **Impact:** Today, Linux powers the vast majority of the world's servers and cloud infrastructure.

## What is Linux?
Linux is an open-source operating system kernel that sits between the hardware and the applications. It manages resources and provides the foundation for the OS.

### Kernel Responsibilities
-   **Process Management:** Scheduling tasks on the CPU.
-   **Memory Management:** allocating RAM to applications.
-   **I/O Management:** Talking to disks, networks, and peripherals.
-   **Security:** Enforcing permissions and user access.

### Architecture Types
-   **Monolithic:** The entire OS runs in kernel space (Linux).
-   **Microkernel:** Only essential services run in kernel space; others are in user space.

## Key Components
| Component | Function |
| :--- | :--- |
| **Shell** | The CLI prompt (Bash, Zsh) for user interaction. |
| **File System** | The directory structure starting from root `/`. |
| **Process Manager** | Handles running programs (PID, init/systemd). |
| **User Manager** | Handles accounts, groups, and permissions. |

---

# AWS Networking: IP Types

Understanding IP addresses is crucial for AWS networking.

| IP Type | Characteristics | Use Case |
| :--- | :--- | :--- |
| **Private IP** | Internal to the AWS network. Persists for the instance's life. | Inter-service communication within a VPC. |
| **Public IP** | Routable on the internet. Dynamic (changes on stop/start). | Temporary internet access for an instance. |
| **Elastic IP (EIP)** | Static public IP. Assigned to your account, not the instance. | Permanent internet address for a server you need to reach reliably. |
