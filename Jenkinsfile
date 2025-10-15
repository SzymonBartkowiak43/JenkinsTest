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

                        // Copy the JAR file
                        sh "scp -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${localJarPath} ${remoteUser}@${remoteHost}:${remotePath}/${jarName}"

                        // Create a systemd service file if it doesn't exist
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

                        // Enable and restart the service
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'systemctl daemon-reload && systemctl enable springapp && systemctl restart springapp'"

                        // Check service status
                        sh "ssh -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} 'systemctl status springapp'"
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