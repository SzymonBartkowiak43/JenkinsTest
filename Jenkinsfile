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

                       // ✅ Używamy potrójnych apostrofów ''' zamiast podwójnych cudzysłowów "
                       //    Jenkins nie będzie interpolował zmiennych i nie potraktuje ich jako sekrety
                       sh(script: '''
                           set -e
                           ssh -i $SSH_KEY_FILE -o StrictHostKeyChecking=no root@167.99.243.51 "mkdir -p /root/app"
                           scp -i $SSH_KEY_FILE -o StrictHostKeyChecking=no target/jenkins-spring-example-0.0.1-SNAPSHOT.jar root@167.99.243.51:/root/app/app.jar
                           ssh -i $SSH_KEY_FILE -o StrictHostKeyChecking=no root@167.99.243.51 '
                               echo "Stopping old process (if any)..."
                               pkill -f app.jar || true
                               sleep 5
                               echo "Ensuring port 8080 is free..."
                               while lsof -i :8080 >/dev/null 2>&1; do
                                 echo "Port still busy, waiting..."
                                 sleep 2
                               done
                               echo "Port 8080 is free!"
                               echo "Starting new app..."
                               nohup /usr/bin/java -jar /root/app/app.jar > /root/app/app.log 2>&1 &
                               sleep 10
                               echo "Testing app..."
                               code=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8080 || true)
                               if [ "$code" = "200" ]; then
                                   echo "✅ App is up and responding!"
                               else
                                   echo "❌ App not responding (HTTP $code)"
                                   exit 1
                               fi
                           '
                       ''', returnStatus: false)
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
