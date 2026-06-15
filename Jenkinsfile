pipeline {
    agent any
    
    environment {
        DOCKER_HUB_USER = 'Abdelrahman-17'
    }
    
    stages {
        stage('Fetch Code') {
            steps {
                echo 'Fetching Latest Code from GitHub...'
                // التأكد من أن الكود نزل بالكامل في المسار الحالي
                checkout scm
            }
        }
        
        stage('SonarQube Code Check') {
            steps {
                echo 'Running Static Code Analysis...'
                // تعديل المسار لـ ${WORKSPACE} صراحة عشان السونار يقرأ الكود الفعلي مش فولدر فاضي
                sh "docker run --rm --network=host -v ${WORKSPACE}:/usr/src sonarsource/sonar-scanner-cli -Dsonar.projectKey=voting-app -Dsonar.sources=/usr/src -Dsonar.host.url=http://127.0.0.1:9000 -Dsonar.login=squ_1b4ab7516d37ba55ed68be0c647cde14b6c8727e"
            }
        }
        
        stage('Security Scan (Trivy FS)') {
            steps {
                echo 'Scanning Source Code Files via Trivy...'
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v ${WORKSPACE}:/root/ aquasec/trivy fs /root/"
            }
        }
        
        stage('Build Docker Images') {
            steps {
                echo 'Building Enterprise Docker Images...'
                // غيرنا المسار لـ ${WORKSPACE} عشان يقفش الـ Dockerfile بالملي
                sh "docker build -t ${DOCKER_HUB_USER}/voting-app-vote:latest ${WORKSPACE}/apps/vote"
                sh "docker build -t ${DOCKER_HUB_USER}/voting-app-result:latest ${WORKSPACE}/apps/result"
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
                // تشغيل الـ compose من الفولدر الصح
                sh "docker compose -f ${WORKSPACE}/docker-compose.prod.yml pull"
                sh "docker compose -f ${WORKSPACE}/docker-compose.prod.yml up -d"
            }
        }
    }
}
