pipeline {
    agent any

    tools { 
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    environment {
        IMAGE_NAME = "mehdisafraoui/student-management"
        TAG = "latest"
    }

    stages {

        stage('GIT') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Mehdi-Safraoui/Safraoui_Mehdi_4TWIN8.git'
            }
        }

        stage('Build Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=student-management -Dsonar.host.url=http://192.168.33.10:9000'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${env.IMAGE_NAME}:${env.TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        dockerImage.push()
                    }
                }
            }
        }

        /* ===== NOUVEAU STAGE POUR L’ATELIER 3 ===== */
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                  kubectl apply -f k8s/mysql-deployment.yaml
                  kubectl apply -f k8s/spring-config.yaml
                  kubectl apply -f k8s/spring-deployment.yaml
                '''
            }
        }
    }
}
