
pipeline {
    agent any
    stages {
        stage("copy files to ansible server") {
            steps {
                script {
                    echo "copying all files to ansible control node"
                    sshagent(['ansible-server-key']) {
                        sh "scp -o StrictHostKeyChecking=no ansible/* root@139.59.167.35:/root"

                        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                            sh "scp ${keyfile} root@139.59.167.35:/root/ssh-key.pem"
                        }
                    }
                }
            }
        }   
    }
}



