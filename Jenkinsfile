pipeline {
    agent {
        label 'Jenkins-Agent'
    }

    tools {
        jdk 'Java17'
        maven 'maven3'
    }

    environment {
        SONAR_SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "complete-prodcution-e2e-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "jall1985"
        DOCKER_PASS = 'docker'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    //JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs() // Clean the workspace before starting
            }
        }

        stage('Checkout from SCM') {
            steps {
                // Checkout the main branch from the specified GitHub repository
                git branch: 'main', 
                    url: 'https://github.com/jal119/complete-prodcution-e2e-.git', 
                    credentialsId: 'github'
            }
        }

        stage('Build Application') {
            steps {
                // Run Maven clean and package commands
                sh 'mvn clean package'
            }
        }

        stage('Test Application') {
            steps {
                // Execute tests using Maven
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "${env.SONAR_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=Register-app -Dsonar.java.binaries=. -Dsonar.projectKey=Register-app"
                }
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
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

