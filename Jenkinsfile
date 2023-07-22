pipeline {
    environment {
        registryCredential = 'dockerhub'
        newApp = ''
        newWeb = ''
    }
    agent { label 'master'}
    stages {
        stage('Prepare') {
            steps {
                sh 'cp .env.example .env'
                sh "sed -i 's/DB_HOST.*/DB_HOST=database/g' .env.testing"
                sh "sed -i 's/DB_USERNAME.*/DB_USERNAME=homestead/g' .env.testing"
                sh "sed -i 's/DB_HOST.*/DB_HOST=database/g' .env"
            }
        }
        stage('Build'){
            steps{
                newApp = docker.build("danilomarcus/forum-app:$BUILD_NUMBER")
                newWeb = docker.build("danilomarcus/forum-web:$BUILD_NUMBER","-f DockerFile_Nginx .")
            }
        }
        stage('Test'){
            steps{
                sh "sed -i 's/forum-app.*/forum-app:$BUILD_NUMBER/g' docker-compose.yml"
                sh "sed -i 's/forum-web.*/forum-web:$BUILD_NUMBER/g' docker-compose.yml"
                sh "docker compose up -d"
                sh "echo 'teste executado'"
            }
        }        
        stage('Push'){
            steps{
                script {
                    docker.withRegistry('',dockerhub)
                    newApp.push()
                    newWeb.push()
                }
            }
        }      
    }
    post {
        failure {
            emailext(
                subject: "Job '${env.JOB_NAME} ${env.BUILD_NUMBER}'",
                body: """<p>A Build Falhou <a href="${env.BUILD_URL}">${env.JOB_NAME}</a></p>""",
                recipientProviders: [[$class: 'DevelopersRecipientProvider'],
                [$class: 'RequesterRecipientProvider']]
            )
        }
        success {
            emailext(
                subject: "Job '${env.JOB_NAME} ${env.BUILD_NUMBER}'",
                body: "Sucesso no Job '${env.JOB_NAME} ${env.BUILD_NUMBER}'",
                to: "danpayne21@gmail.com"
            )
        }
    }
}
