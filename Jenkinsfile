pipeline {
    agent any

    tools {
        maven "M3"
    }

    stages {

        stage('Build & Test') {
            steps {
                sh "mvn clean package -DskipTests=false"
            }
        }

        stage('Deploy to Digital Ocean') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'digitalocean-ssh-key', keyFileVariable: 'SSH_KEY_FILE')]) {

                        def remoteUser = "root"
                        def remoteHost = "167.99.243.51"
                        def remotePath = "/root/app"
                        def jarName = "app.jar"
                        def localJarPath = "target/jenkins-spring-example-0.0.1-SNAPSHOT.jar"
                        def javaPath = "/usr/bin/java"

                        // 🔹 1. Stworzenie katalogu na serwerze
                        sh """
                        ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} '
                          mkdir -p ${remotePath}
                        '
                        """

                        // 🔹 2. Skopiowanie nowego JARa
                        sh """
                        scp -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${localJarPath} ${remoteUser}@${remoteHost}:${remotePath}/${jarName}
                        """

                        // 🔹 3. Zabicie starego procesu i odczekanie aż port się zwolni
                        sh """
                        ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} '
                          echo "Stopping old process (if any)..."
                          pkill -f ${jarName} || true
                          sleep 5
                          echo "Ensuring port 8080 is free..."
                          while lsof -i :8080 >/dev/null 2>&1; do
                            echo "Port still busy, waiting..."
                            sleep 2
                          done
                          echo "Port 8080 is free!"
                        '
                        """

                        // 🔹 4. Uruchomienie nowej aplikacji
                        sh """
                        ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} '
                          echo "Starting new app..."
                          nohup ${javaPath} -jar ${remotePath}/${jarName} > ${remotePath}/app.log 2>&1 &
                          sleep 10
                        '
                        """

                        // 🔹 5. Sprawdzenie logów i dostępności aplikacji
                        sh """
                        ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} '
                          echo "Last 10 lines of app.log:"
                          tail -n 10 ${remotePath}/app.log
                          echo "Testing endpoint:"
                          curl -s -o /dev/null -w "%{http_code}" http://localhost:8080 | grep 200 >/dev/null && echo "✅ Application is up!" || (echo "❌ App failed to respond" && exit 1)
                        '
                        """
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
            echo '✅ Pipeline zakończony sukcesem – aplikacja działa na serwerze!'
        }
        failure {
            echo '❌ Pipeline zakończył się błędem – sprawdź logi z app.log.'
        }
    }
}
