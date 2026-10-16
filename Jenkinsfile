pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Clone latest code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Chandanponnappa/TodoSummaryAssistant-final.git'
                sh 'git log -1 --oneline'
            }
        }

        stage('Test backend') {
            steps {
                withCredentials([file(credentialsId: 'todo-env', variable: 'ENV_FILE')]) {
                    dir('Backend/todo-summary-assistant') {
                        sh '''
                            set -a
                            . "$ENV_FILE"
                            set +a
                            chmod +x mvnw
                            ./mvnw -B test
                        '''
                    }
                }
            }
            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'Backend/todo-summary-assistant/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Add .env') {
            steps {
                withCredentials([file(credentialsId: 'todo-env', variable: 'ENV_FILE')]) {
                    sh 'cp "$ENV_FILE" .env'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker rm -f todo-backend todo-frontend || true'
                sh 'docker compose up -d'
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    sleep 25
                    docker compose ps
                    curl -i -X OPTIONS http://13.201.168.199:8080/api/todos \
                      -H "Origin: http://13.201.168.199" \
                      -H "Access-Control-Request-Method: POST" | head -8
                '''
            }
        }
    }

    post {
        always  { sh 'rm -f .env' }
        failure { sh 'docker compose logs --tail 50 backend || true' }
        success { sh 'docker image prune -f' }
    }
}
