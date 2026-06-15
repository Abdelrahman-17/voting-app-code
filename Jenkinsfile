pipeline {
    agent any
    
    environment {
        DOCKER_HUB_USER = 'Abdelrahman-17'
    }
    
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
                // هنا ربطنا الـ Workspace بالـ /usr/src جوه الكونتينر، وخلينا السورس يقرأ من النقطة الحالية مع الفلترة
                sh "docker run --rm --network=host -v \$(pwd):/usr/src sonarsource/sonar-scanner-cli -Dsonar.projectKey=voting-app -Dsonar.sources=. -Dsonar.inclusions=apps/** -Dsonar.host.url=http://127.0.0.1:9000 -Dsonar.login=squ_1b4ab7516d37ba55ed68be0c647cde14b6c8727e"
            }
        }
        
        stage('Security Scan (Trivy FS)') {
            steps {
                echo 'Scanning Source Code Files via Trivy...'
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v \$(pwd):/root/ aquasec/trivy fs /root/"
            }
        }
        
        stage('Build Docker Images') {
            steps {
                echo 'Building Enterprise Docker Images...'
                // بندخل جوه الفولدر الأول ونعمل البيلد عشان نتفادى حوار الـ Workspace المتغير
                sh "cd apps/vote && docker build -t \${DOCKER_HUB_USER}/voting-app-vote:latest ."
                sh "cd apps/result && docker build -t \${DOCKER_HUB_USER}/voting-app-result:latest ."
            }
        }

        stage('Security Scan (Trivy Image)') {
            steps {
                echo 'Scanning Docker Images via Trivy...'
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_HUB_USER}/voting-app-vote:latest'
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${DOCKER_HUB_USER}/voting-app-result:latest'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin'
                    sh 'docker push ${DOCKER_HUB_USER}/voting-app-vote:latest'
                    sh 'docker push ${DOCKER_HUB_USER}/voting-app-result:latest'
                }
            }
        }

        stage('Continuous Deployment (CD)') {
            steps {
                echo 'Deploying Application Services...'
                sh "docker compose -f docker-compose.prod.yml pull"
                sh "docker compose -f docker-compose.prod.yml up -d"
            }
        }
    }
}
