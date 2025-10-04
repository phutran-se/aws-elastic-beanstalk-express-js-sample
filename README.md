> ⚠️ This is a fork of [AWS Elastic Beanstalk Node.js Sample App](https://github.com/aws-samples/aws-elastic-beanstalk-express-js-sample) for learning and testing purposes.

---
# AWS Elastic Beanstalk Node.js Sample App (Modified for Secure DevOps Assignment)

This repository is a fork of the **AWS Elastic Beanstalk Express sample**.  
It has been updated and extended for the **Secure DevOps Assignment (ISEC6000)** to demonstrate a secure CI/CD pipeline using **Jenkins** and **Docker-in-Docker (DinD)**.

---

## Overview

The main goal of this modified repository is to show how a simple Node.js web application can be built, tested, scanned, and deployed automatically through a Jenkins pipeline.  
All configuration for Jenkins and DinD is stored separately in the **Project2-Compose** repository.

---

## Changes from the Original AWS Sample

- Added a **Dockerfile** to containerize the Node.js application.  
- Added a **Jenkinsfile** defining a complete CI/CD pipeline with the following stages:
  1. Checkout source code from SCM  
  2. Install dependencies in Node 16 container  
  3. Run unit tests  
  4. Dependency scanning using **Snyk CLI** (fail on High/Critical)  
  5. Build Docker image  
  6. Push image to Docker Hub  
- Updated **package.json** to include a vulnerable dependency (`lodash@4.17.10`) used to demonstrate the Snyk security gate.  
- The pipeline connects to a Jenkins environment defined in `docker-compose.yml` from the **Project2-Compose** repository.

---

## How to Build and Run the Application

### Run Locally
You can build and run the app manually using Docker:
```bash
docker build -t node-sample .
docker run -p 8080:8080 node-sample
```
Then open your browser and go to:
```
http://localhost:8080
```

### Run via Jenkins Pipeline
The Jenkins pipeline defined in the **Jenkinsfile** automates the following:
- Installs dependencies and runs tests  
- Scans for vulnerabilities using **Snyk**  
- Builds and tags a Docker image  
- Pushes the image to Docker Hub  

To use this pipeline:
1. Configure Jenkins and Docker-in-Docker using the `docker-compose.yml` in the **Project2-Compose** repository.  
2. Add credentials in Jenkins:
   - `dockerhub` (username and password)
   - `snyk_token` (Secret text for Snyk authentication)
3. Create a pipeline job in Jenkins that reads this Jenkinsfile from SCM.  

---

## Security Features

- **Snyk Security Gate:**  
  The pipeline will fail automatically if High or Critical vulnerabilities are found in project dependencies.  
  This ensures that insecure code cannot proceed to the build and deployment stages.  

- **Least Privilege User:**  
  A non-admin user (`ci-user`) is used to trigger pipeline builds, following the principle of least privilege.

- **Credential Protection:**  
  Jenkins credentials are injected securely at runtime and masked in the console output.

---

## Related Repository

- **Project2-Compose:**  
  Contains the Docker Compose configuration, documentation, and Jenkins security notes.  
  Repository link: [https://github.com/phutran-se/Project2-Compose](https://github.com/phutran-se/Project2-Compose)

---

## License

This project remains licensed under the **MIT-0 License** from the original AWS sample.  
See the LICENSE file for details.

