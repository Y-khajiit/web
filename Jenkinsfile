pipeline {
    agent any

    environment {
        REMOTE_HOST = "192.168.0.90"      
        REMOTE_USER = "khajiit"      
        DEPLOY_PATH = "/var/www/html
"       
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Y-khajiit/web.git'
            }
        }

        stage('Test Code') {
            steps {
                script {
                    if (!fileExists('index.html')) {
                        error "Файл index.html відсутній!"
                    }
                }
            }
        }

        stage('Transfer Files via SSH') {
            steps {
                sh """
                    rsync -avz -e 'ssh -o StrictHostKeyChecking=no' ./ ${REMOTE_USER}@${REMOTE_HOST}:${DEPLOY_PATH}
                """
            }
        }

        stage('Restart Nginx') {
            steps {
                sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} 'sudo systemctl restart nginx'"
            }
        }

        stage('Validation') {
            steps {
                script {
                    def statusCode = sh(
                        script: "curl -o /dev/null -s -w '%{http_code}' http://${REMOTE_HOST}",
                        returnStdout: true
                    ).trim()

                    if (statusCode != "200") {
                        error "Сайт недоступний. Код відповіді: ${statusCode}"
                    }
                }
            }
        }
        stage('Validate HTML') {
            steps {
                sh '''
                    tidy -errors -quiet index.html > /dev/null 2> tidy_errors.txt || true
                    if [ -s tidy_errors.txt ]; then
                      echo "HTML має помилки:"
                      cat tidy_errors.txt
                      exit 1
                    fi
                '''
            }
        }

    }

    post {
        success {
            echo "✅ Деплоймент успішний!"
        }
        failure {
            echo "❌ Деплоймент не пройшов. Перевірте логи."
        }
    }
}
