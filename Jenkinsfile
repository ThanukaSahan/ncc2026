pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        DOCKER_IMAGE         = "yourdockerhubuser/yourapp:${env.BUILD_NUMBER}"
        SONARQUBE_NAME       = 'sonarqube'
        SONAR_TOKEN_ID       = 'sonar-token'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build (compile/package)') {
            agent {
                docker { image 'maven:3.9.9-eclipse-temurin-21-alpine' }
            }
            steps {
                // Setting -Dmaven.repo.local creates .m2 inside workspace where build user has permission
                sh 'mvn -B -DskipTests package -Dmaven.repo.local=.m2/repository'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def img = null
                    if (isUnix()) {
                        img = docker.build("${env.DOCKER_IMAGE}", "-f app/Dockerfile app")
                    } else {
                        img = docker.build("${env.DOCKER_IMAGE}", "-f app\\Dockerfile app")
                    }
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
                                sh 'mvn -B test sonar:sonar -Dsonar.login=${SONAR_TOKEN} -Dmaven.repo.local=.m2/repository'
                            } else {
                                bat 'mvn -B test sonar:sonar -Dsonar.login=%SONAR_TOKEN% -Dmaven.repo.local=.m2/repository'
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