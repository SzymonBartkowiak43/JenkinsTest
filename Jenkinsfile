pipeline {
    agent any

    tools {
        maven "M3"
    }

    stages {
        stage('Build & Test') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('Deploy to Digital Ocean') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'digitalocean-ssh-key', keyFileVariable: 'SSH_KEY_FILE')]) {

                        def remoteUser = "root"
                        def remoteHost = "167.99.243.51"
                        def remotePath = "/opt/kebabbb" // <-- Ta ścieżka jest poprawna
                        def jarName = "app.jar"
                        def localJarPath = "target/jenkins-spring-example-0.0.1-SNAPSHOT.jar"

                        // ===================================================================
                        // KROK 0: STWORZENIE KATALOGU (TA LINIA ZOSTAŁA PRZYWRÓCONA)
                        // ===================================================================
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'mkdir -p ${remotePath}'"

                        // Krok 1: Skopiowanie pliku .jar
                        sh "scp -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${localJarPath} ${remoteUser}@${remoteHost}:${remotePath}/${jarName}"

                        // Krok 2: Zrestartowanie usługi 'kebabbb' na serwerze
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'systemctl daemon-reload && systemctl restart kebabbb'"
                    }
                }
            }
        }
    }

    post {
        always {
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