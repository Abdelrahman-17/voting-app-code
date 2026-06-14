pipeline {
    agent any
    
    environment {
        DOCKER_HUB_USER = 'Abdelrahman-17'
    }
    
    stages {
        stage('Fetch Code') {
            steps {
                echo 'Fetching Latest Code from GitHub...'
            }
        }
        
        stage('SonarQube Code Check') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    echo 'Running Static Code Analysis via SonarScanner Container...'
                    // الأمر السحري اللي بيبعت الكود فعلياً لسيرفر سونار ويخليه يظهر عندك في المتصفح
                    sh "docker run --rm --network=enterprise-devops-platform_default -v \$(pwd):/usr/src sonarsource/sonar-scanner-cli -Dsonar.projectKey=voting-app -Dsonar.sources=. -Dsonar.host.url=http://sonarqube:9000 -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }
        
        stage('Security Scan (Trivy)') {
            steps {
                echo 'Scanning Source Code via Trivy Container...'
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/root/ aquasec/trivy fs /root/'
            }
        }
        
        stage('Build Docker Images') {
            steps {
                echo 'Building Enterprise Docker Images...'
                sh 'docker build -t ${DOCKER_HUB_USER}/voting-app-vote:latest ./vote'
                sh 'docker build -t ${DOCKER_HUB_USER}/voting-app-result:latest ./result'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    echo 'Logging into Docker Hub...'
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    
                    echo 'Pushing Images to Docker Hub Registry...'
                    sh 'docker push ${DOCKER_HUB_USER}/voting-app-vote:latest'
                    sh 'docker push ${DOCKER_HUB_USER}/voting-app-result:latest'
                }
            }
        }
    }
}
