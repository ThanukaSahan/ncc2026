pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'   // create this credential in Jenkins (username/password or token)
        DOCKER_IMAGE = "yourdockerhubuser/yourapp:${env.BUILD_NUMBER}"
        SONARQUBE_NAME = 'sonarqube'                // SonarQube server configured in Jenkins
        SONAR_TOKEN_ID = 'sonar-token'              // Sonar token stored in Jenkins credentials (string)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build (compile/package)') {
            agent {
                docker { image 'maven:3.9.9' }
            }
            steps {
                sh 'mvn -B -DskipTests package'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // build Docker image from app/Dockerfile
                    def img = null
                    if (isUnix()) {
                        img = docker.build("${env.DOCKER_IMAGE}", "-f app/Dockerfile app")
                    } else {
                        img = docker.build("${env.DOCKER_IMAGE}", "-f app\\Dockerfile app")
                    }
                    // store image name for later stages
                    env.BUILT_IMAGE = env.DOCKER_IMAGE
                }
            }
        }

        stage('SonarQube + Tests') {
            steps {
                script {
                    withCredentials([string(credentialsId: env.SONAR_TOKEN_ID, variable: 'SONAR_TOKEN')]) {
                        withSonarQubeEnv(env.SONARQUBE_NAME) {
                            if (isUnix()) {
                                sh 'mvn -B test sonar:sonar -Dsonar.login=${SONAR_TOKEN}'
                            } else {
                                bat 'mvn -B test sonar:sonar -Dsonar.login=%SONAR_TOKEN%'
                            }
                        }
                    }
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', env.DOCKERHUB_CREDENTIALS) {
                        def toPush = docker.image(env.BUILT_IMAGE)
                        toPush.push()
                        // optionally push 'latest' tag
                        toPush.push('latest')
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}