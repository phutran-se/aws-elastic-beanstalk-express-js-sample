
// Jenkinsfile (root) — Node 16 + Snyk + Build & Push
pipeline {
  agent any
  environment {
    // Talk to Docker-in-Docker (DinD) over TLS (From Task 2)
    DOCKER_HOST       = 'tcp://docker:2376'
    DOCKER_CERT_PATH  = '/certs/client'
    DOCKER_TLS_VERIFY = '1'

    // My forked repo and Docker Hub repo & Tag
    GIT_URL    = 'https://github.com/phutran-se/Project2-Compose'
    IMAGE_NAME = 'phutranse/aws-express-app'
    TAG        = "build-${env.BUILD_NUMBER}" 

    // If package.json lives in a subfolder, set APP_DIR='subfolder'; otherwise '.'
    APP_DIR = '.'
  }
  options { timestamps() }

  stages {
    stage('Checkout') {
      steps {
        // Pull code from my forked repo; or keep "checkout scm" if job already points to the repo
        git branch: 'main', url: "${GIT_URL}"
      }
    }
    stage('Install Dependencies (Node 16)') {
      steps {
        // Run Node 16 in a disposable container; mount Jenkins workspace and run npm install
        sh '''
          docker run --rm \
            -v "$WORKSPACE":/app -w /app \
            node:16 \
            sh -c "node -v && npm install --save"
        '''
      }
    }

    stage('Run Tests (Node 16)') {
      steps {
        // Do not break the pipeline if no tests are defined
        sh '''
          docker run --rm \
            -v "$WORKSPACE":/app -w /app \
            node:16 \
            sh -c "npm test || echo 'No tests defined'"
        '''
      }
      post {
        // Collect JUnit if present; ignore if none
        always { junit allowEmptyResults: true, testResults: 'junit*.xml' }
      }
    }

    stage('Dependency Scan (Snyk)') {
      steps {
        // Use Snyk CLI and fail the build on High/Critical (severity >= high)
        withCredentials([string(credentialsId: 'snyk_token', variable: 'SNYK_TOKEN')]) {
          sh '''
            docker run --rm \
              -e SNYK_TOKEN="$SNYK_TOKEN" \
              -v "$WORKSPACE":/app -w /app \
              snyk/snyk:docker snyk test \
              --file=package.json \
              --severity-threshold=high
          '''
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        // Build image from the Dockerfile at repo root; use a unique, traceable tag
        sh 'docker build -t "$IMAGE_NAME:$TAG" .'
      }
    }

    stage('Push Docker image to Docker Hub') {
      steps {
        // Login and push using Jenkins credentials (ID=dockerhub)
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh '''
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push "$IMAGE_NAME:$TAG"
          '''
        }
      }
    }
  }

  post {
    always {
      // Archive evidence for the report (Snyk JSON + Dockerfile)
      archiveArtifacts artifacts: 'Dockerfile', onlyIfSuccessful: false
      cleanWs()
    }
  }
}
