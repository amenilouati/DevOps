pipeline {
    agent any
    tools {
        maven 'M2_HOME'
        jdk 'JAVA_HOME'
    }
    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }
        stage('Test Backend') {
            steps {
                dir('backend') {
                    sh 'mvn test'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                dir('backend') {
                    withSonarQubeEnv('SonarQubeLocal') {
                        sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=Devops -Dsonar.projectName=Devops'
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        docker build -t amenilouati/amenilouati-gestionprojets-backend:latest ./backend
                        docker build -t amenilouati/amenilouati-gestionprojets-frontend:latest ./frontend
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push amenilouati/amenilouati-gestionprojets-backend:latest
                        docker push amenilouati/amenilouati-gestionprojets-frontend:latest
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker compose -p devops-appgestiondesprojets down'
                sh 'docker compose -p devops-appgestiondesprojets up -d --build'
            }
        }
    }
}