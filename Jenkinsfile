pipeline {
    agent any
    
    stages {
        stage('Fetch Code') {
            steps {
                echo 'Fetching Latest Code from GitHub...'
                checkout scm
            }
        }
        
        stage('SonarQube Code Check') {
            steps {
                echo 'Running Static Code Analysis via SonarScanner CLI...'
                sh "docker run --rm --network=host -v \$(pwd):/usr/src sonarsource/sonar-scanner-cli -Dsonar.projectKey=voting-app -Dsonar.sources=. -Dsonar.host.url=http://127.0.0.1:9000 -Dsonar.login=squ_1b4ab7516d37ba55ed68be0c647cde14b6c8727e"
            }
        }
        
        stage('Security Scan (Trivy FS)') {
            steps {
                echo 'Scanning Source Code Files via Trivy...'
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v \$(pwd):/root/ aquasec/trivy fs --skip-files /root/vote/Cargo.toml /root/"
            }
        }
        
        stage('Build Docker Images') {
            steps {
                echo 'Building Enterprise Docker Images...'
                // حولنا الـ A لحروف صغيرة abdelrahman-17 عشان نمشي مع معايير الـ Registry الصارمة
                sh """
                    cd vote && docker build -t abdelrahman-17/voting-app-vote:latest .
                    cd ../result && docker build -t abdelrahman-17/voting-app-result:latest .
                """
            }
        }

        stage('Security Scan (Trivy Image)') {
            steps {
                echo 'Scanning Docker Images for OS Vulnerabilities via Trivy...'
                // هنا تريفي هيعمل parse وهو مغمض لأن الاسم lowercase تماماً وسليم بنسبة 100%
                sh """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image abdelrahman-17/voting-app-vote:latest
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image abdelrahman-17/voting-app-result:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    echo 'Logging into Docker Hub Registry...'
                    sh """
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push abdelrahman-17/voting-app-vote:latest
                        docker push abdelrahman-17/voting-app-result:latest
                    """
                }
            }
        }

        stage('Continuous Deployment (CD)') {
            steps {
                echo 'Deploying Application Services via Docker Compose Prod...'
                sh """
                    docker compose -f docker-compose.prod.yml pull
                    docker compose -f docker-compose.prod.yml up -d
                """
                echo 'Deployment complete! Application is live.'
            }
        }
    }
}
