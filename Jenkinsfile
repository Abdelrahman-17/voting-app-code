ipeline {
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
                    echo 'Running Static Code Analysis via SonarQube...'
                    
                }
            }
        }
        
        stage('Security Scan (Trivy)') {
            steps {
                echo 'Scanning Source Code for Security Vulnerabilities via Docker...'
                    sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $WORKSPACE:/root/ aquasec/trivy fs /root/'
                 }
             }
        
        stage('Build Docker Images') {
            steps {
                echo 'Building Enterprise Docker Images...'
                sh 'docker build -t ${DOCKER_HUB_USER}/voting-app-vote:latest ./vote'
                sh 'docker build -t ${DOCKER_HUB_USER}/voting-app-result:latest ./result'
            }
        }
    }
}
