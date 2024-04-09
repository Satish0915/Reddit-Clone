pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'Sonar-scanner'
        APP_NAME = "reddit-clone-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "satish003"
        DOCKER_PASS = 'Dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"

    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', credentialsId: 'Github', url: 'https://github.com/Satish0915/Reddit-Clone.git'
               // git branch: 'main', url: 'https://github.com/Satish0915/Reddit-Clone.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('SonarQube-server') {
                    sh '''$SCANNER_HOME/bin/Sonar-scanner -Dsonar.projectName=Reddit-Clone-CI \
                    -Dsonar.projectKey=Reddit-Clone-CI'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonrQube-Token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
             }
         }
        stage("Build & Push Docker Image") {
             steps {
                 script {
                     docker.withRegistry('',DOCKER_PASS) {
                         docker_image = docker.build "${IMAGE_NAME}"
                     }
                     docker.withRegistry('',DOCKER_PASS) {
                         docker_image.push("${IMAGE_TAG}")
                         docker_image.push('latest')
                     }
                 }
             }
         }
    }
}
