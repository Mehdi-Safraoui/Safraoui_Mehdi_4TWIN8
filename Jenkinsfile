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
        script {
            try {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                      mvn sonar:sonar \
                      -Dsonar.projectKey=student-management \
                      -Dsonar.host.url=http://192.168.33.10:9000 || true
                    '''
                }
            } catch (e) {
                echo "SonarQube unreachable, continuing pipeline..."
            }
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

        stage('Deploy to Kubernetes') {
    steps {
        script {
            sh '''
              if command -v kubectl >/dev/null 2>&1; then
                kubectl apply -f k8s/mysql-deployment.yaml
                kubectl apply -f k8s/spring-deployment.yaml
              else
                echo "kubectl not found on Jenkins agent – skipping Kubernetes deployment"
              fi
            '''
        }
    }
}

    }
}
