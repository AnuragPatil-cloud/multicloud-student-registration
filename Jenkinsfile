pipeline {
  agent any
  options {
    timestamps()
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '10'))
  }
  environment {
    BACKEND_IMAGE = 'student-registration-backend'
    FRONTEND_IMAGE = 'student-registration-frontend'
    DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
    TRIVY_SEVERITY = 'HIGH,CRITICAL'
    VITE_API_URL = '/api'
  }
  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Backend Test') {
      steps { dir('backend') { sh './mvnw -B clean test || mvn -B clean test' } }
    }

    stage('SonarQube') {
      steps {
        dir('backend') {
          withSonarQubeEnv('SonarQube') {
            sh 'mvn -B sonar:sonar -Dsonar.projectKey=multicloud-student-registration'
          }
        }
      }
    }

    stage('Frontend Lint & Build') {
      steps { dir('frontend') { sh 'npm ci && npm run lint && npm run build' } }
    }

    stage('Prepare Tags') {
      steps {
        script {
          env.GIT_SHA = sh(script: 'git rev-parse --short=12 HEAD', returnStdout: true).trim()
          env.IMAGE_TAG = "${env.GIT_SHA}-${env.BUILD_NUMBER}"
        }
      }
    }

    stage('Build Images') {
      steps {
        sh '''
          docker build --pull -t ${BACKEND_IMAGE}:${IMAGE_TAG} ./backend
          docker build --pull --build-arg VITE_API_URL=${VITE_API_URL} -t ${FRONTEND_IMAGE}:${IMAGE_TAG} ./frontend
        '''
      }
    }

    stage('Trivy Security Gate') {
      steps {
        sh '''
          trivy image --scanners vuln,secret --severity ${TRIVY_SEVERITY} --exit-code 1 ${BACKEND_IMAGE}:${IMAGE_TAG}
          trivy image --scanners vuln,secret --severity ${TRIVY_SEVERITY} --exit-code 1 ${FRONTEND_IMAGE}:${IMAGE_TAG}
        '''
      }
    }

    stage('Push to Docker Hub') {
      when { branch 'main' }
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
          sh '''
            echo "$DOCKER_PASSWORD" | docker login --username "$DOCKER_USERNAME" --password-stdin
            docker tag ${BACKEND_IMAGE}:${IMAGE_TAG} ${DOCKER_USERNAME}/${BACKEND_IMAGE}:${IMAGE_TAG}
            docker tag ${FRONTEND_IMAGE}:${IMAGE_TAG} ${DOCKER_USERNAME}/${FRONTEND_IMAGE}:${IMAGE_TAG}
            docker push ${DOCKER_USERNAME}/${BACKEND_IMAGE}:${IMAGE_TAG}
            docker push ${DOCKER_USERNAME}/${FRONTEND_IMAGE}:${IMAGE_TAG}
            docker logout
          '''
        }
      }
    }

    stage('Update Multi-Cloud GitOps Values') {
      when { branch 'main' }
      steps {
        sh '''
          set -e
          for file in deploy/values/aws.yaml deploy/values/azure.yaml deploy/values/gcp.yaml; do
            sed -i "s/tag: ".*"/tag: \"${IMAGE_TAG}\"/g" "$file"
          done
          git config user.name "Jenkins CI"
          git config user.email "jenkins@localhost"
          git add deploy/values
          git commit -m "Release ${IMAGE_TAG}" || true
          git push origin HEAD:main
        '''
      }
    }
  }

  post {
    always { sh 'docker image prune -f || true' }
  }
}
