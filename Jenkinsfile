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
                // ===================================================================
                // DODAJEMY BLOK 'script', ABY MÓC UŻYWAĆ ZMIENNYCH
                // ===================================================================
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'digitalocean-ssh-key', keyFileVariable: 'SSH_KEY_FILE')]) {

                        def remoteUser = "root"
                        def remoteHost = "167.99.243.51"
                        def remotePath = "/root/app"
                        def jarName = "app.jar"
                        // Poprawka ścieżki do JARa, żeby była bardziej niezawodna
                        def localJarPath = "target/jenkins-spring-example-0.0.1-SNAPSHOT.jar"
                        def javaPath = "/usr/bin/java"

                        // Krok 1: Stworzenie katalogu na serwerze
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'mkdir -p ${remotePath}'"

                        // Krok 2: Skopiowanie pliku .jar na serwer
                        sh "scp -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${localJarPath} ${remoteUser}@${remoteHost}:${remotePath}/${jarName}"

                        // Krok 3: Zatrzymanie starego procesu i uruchomienie nowego
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'pkill -f ${jarName} || true && nohup ${javaPath} -jar ${remotePath}/${jarName} > ${remotePath}/app.log 2>&1 &'"
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