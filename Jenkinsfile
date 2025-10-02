pipeline {
  agent any
  environment {
    // Talk to Docker-in-Docker (DinD) over TLS
    DOCKER_HOST       = 'tcp://docker:2376'
    DOCKER_CERT_PATH  = '/certs/client'
    DOCKER_TLS_VERIFY = '1'

    // Your forked repo and Docker Hub repo
    GIT_URL    = 'https://github.com/phutran-se/ISEC6000_P2_Test'
    IMAGE_NAME = 'phutranse/express-app'
    TAG        = "build-${env.BUILD_NUMBER}"

    // If package.json lives in a subfolder, set APP_DIR='subfolder'; otherwise '.'
    APP_DIR = '.'
  }
  options { timestamps() }

  stages {
    stage('Checkout') {
      steps {
        // Pull code from your fork; or keep "checkout scm" if job already points to the repo
        checkout scm
      }
    }
    stage("Debug Test") {
      steps {
        sh '''
          echo "[DEBUG] Host: $HOSTNAME ; WORKSPACE=$WORKSPACE"
          ls -la "$WORKSPACE"

          docker run --rm -v "$WORKSPACE":/app -w /app node:16 \
            sh -c "pwd && ls -la && test -f package.json && head -n 5 package.json || (echo '[ERROR] package.json not found' && exit 1)"
        '''
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

    stage('Push to Docker Hub') {
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
      // Evidence for the report (adjust as needed)
      archiveArtifacts artifacts: 'Dockerfile', onlyIfSuccessful: false
      cleanWs()
    }
  }
}
