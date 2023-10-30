pipeline {
    agent any
    
    stages {
        stage ('Checkout'){
          steps {
              git branch: 'main', url: 'https://github.com/vladislav-polynskii/sf-web'
          }
        }
        
        stage('Build') {
            steps {
                sh 'docker run -d -p 9889:80 --name nginx nginx'
                sh 'docker cp ${WORKSPACE}/index.html nginx:/usr/share/nginx/html'
            }
        }
        
        stage('Test') {
            steps {
                script {
                    def httpResponse = sh(returnStdout: true, script: 'curl -sL -o /dev/null -w "%{http_code}" http://localhost:9889/')
                    if (httpResponse.trim() != '200') {
                        error 'HTTP response code is not 200'
                    }
                    
                    def gitFile = sh(returnStdout: true, script: 'md5sum ${WORKSPACE}/index.html | cut -d " " -f 1')
                    def urlFile = sh(returnStdout: true, script: 'curl -s http://localhost:9889/index.html | md5sum | cut -d " " -f 1')
                    if (gitFile.trim() != urlFile.trim()) {
                        error 'Hashes do not match'
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
            sh 'docker rm -f nginx'
        }
        
        success {
            echo 'CI passed successfully!'
            withCredentials([string(credentialsId: 'botSecret', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
            sh  ("""
            curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : CI passed successfully'
            """)
            }
        }
        
        failure {
            echo 'CI failed!'
            withCredentials([string(credentialsId: 'botSecret', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
            sh  ("""
            curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : CI failed'
            """)
            }
        }
    }
}
