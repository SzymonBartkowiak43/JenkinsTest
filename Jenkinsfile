pipeline {
    agent any

    tools {
        maven "M3"
    }

    stages {
        stage('Build') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('Test') {
             steps {
                 sh "mvn test"
             }
        }
        stage('Deploy to Digital Ocean') {
            steps {

                withCredentials([sshUserPrivateKey(credentialsId: 'digitalocean-ssh-key', keyFileVariable: 'SSH_KEY_FILE')]) {

                    sh "mvn deploy"
                }
            }
        }
    }

    post {
        always {
            // Sprzątanie artefaktów po budowaniu
            cleanWs()
        }
        success {
            echo 'Pipeline zakończony sukcesem!'
        }
        failure {
            echo 'Pipeline zakończył się błędem.'
        }
    }
}