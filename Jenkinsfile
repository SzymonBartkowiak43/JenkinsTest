pipeline {
    agent any

    tools {
        maven "M3"
    }

    stages {
        stage('Build & Test') {
            steps {
                // Łączymy budowanie i testowanie w jeden krok, tak jak robi to Maven
                sh "mvn clean package"
            }
        }
        stage('Deploy to Digital Ocean') {
            steps {
                // Używamy poświadczeń tak jak wcześniej
                withCredentials([sshUserPrivateKey(credentialsId: 'digitalocean-ssh-key', keyFileVariable: 'SSH_KEY_FILE')]) {

                    // Definiujemy zmienne, żeby było czytelniej
                    def remoteUser = "root"
                    def remoteHost = "167.99.243.51"
                    def remotePath = "/root/app"
                    def jarName = "app.jar"
                    def localJarPath = "target/jenkins-spring-example-0.0.1-SNAPSHOT.jar"
                    def javaPath = "/usr/bin/java" // Pełna ścieżka, której się nauczyliśmy

                    // Krok 1: Stworzenie katalogu na serwerze (jeśli nie istnieje)
                    sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'mkdir -p ${remotePath}'"

                    // Krok 2: Skopiowanie pliku .jar na serwer
                    sh "scp -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${localJarPath} ${remoteUser}@${remoteHost}:${remotePath}/${jarName}"

                    // Krok 3: Zatrzymanie starego procesu i uruchomienie nowego
                    sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'pkill -f ${jarName} || true && nohup ${javaPath} -jar ${remotePath}/${jarName} > ${remotePath}/app.log 2>&1 &'"
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