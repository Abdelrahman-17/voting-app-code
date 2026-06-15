pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Fetch Code') {
            steps {
                echo "Fetching latest changes..."
            }
        }

        stage('SonarQube Code Check') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=voting-app \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://127.0.0.1:9000 \
                    -Dsonar.login=squ_1b4ab7516d37ba55ed68be0c647cde14b6c87272"
                }
            }
        }

        stage('Security Scan (Trivy FS)') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }

        stage('Build Docker Images') {
            steps {
                sh "docker build -t abdelrahmana890/voting-app-vote:latest ./apps/vote"
                sh "docker build -t abdelrahmana890/voting-app-result:latest ./apps/result"
            }
        }

        stage('Security Scan (Trivy Image)') {
            steps {
                sh "trivy image abdelrahmana890/voting-app-vote:latest"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                    sh "docker push abdelrahmana890/voting-app-vote:latest"
                    sh "docker push abdelrahmana890/voting-app-result:latest"
                }
            }
        }

        stage('Continuous Deployment (CD)') {
            steps {
                script {
                    // التعديل السحري: بندخل للفولدر وبنشغل علطول مع البناء المحلي للـ worker والـ vote، بدون pull يعطلنا
                    dir('apps') {
                        sh 'docker compose -f docker-compose.prod.yml up -d --build'
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished execution."
        }
        success {
            echo "Deployment complete! Application is live."
        }
        failure {
            echo "Pipeline failed. Check logs for issues."
        }
    }
}
