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
        stage('Deploy to Server') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'digitalocean-ssh-key', keyFileVariable: 'SSH_KEY_FILE')]) {
                        def remoteUser = "root"
                        def remoteHost = "167.99.243.51"
                        def remotePath = "/opt/springapp"
                        def jarName = "app.jar"
                        def localJarPath = "target/jenkins-spring-example-0.0.1-SNAPSHOT.jar"

                        // Create deployment directory
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'mkdir -p ${remotePath}'"

                        // Stop any existing Java processes using port 8080
                        sh """
                            ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} '
                                systemctl stop springapp || true
                                systemctl stop kebabbb || true
                                fuser -k 8080/tcp || true
                                sleep 2
                            '
                        """

                        // Copy the JAR file
                        sh "scp -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${localJarPath} ${remoteUser}@${remoteHost}:${remotePath}/${jarName}"

                        // Create the systemd service file
                        sh """
                            ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'cat > /etc/systemd/system/springapp.service << EOF
[Unit]
Description=Spring Boot Application
After=network.target

[Service]
User=root
ExecStart=/usr/bin/java -jar ${remotePath}/${jarName}
SuccessExitStatus=143
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF'
                        """

                        // Reload systemd, enable and start service (as separate commands)
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'systemctl daemon-reload'"
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'systemctl enable springapp'"
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'systemctl restart springapp'"

                        // Wait for application to start
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'sleep 10'"

                        // Check service status
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'systemctl status springapp'"

                        // Verify application response
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'curl -s localhost:8080'"
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
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}