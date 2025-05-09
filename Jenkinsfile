pipeline {
    agent any
    
    tools {
        nodejs "nodejs"
    }


    stages {
        stage('Install Packages') {
            steps {
                script {
                    sh 'yarn install'
                }
            }
        }

        stage('Run the App') {
            steps {
                script {
                    sh '''
                    npm install -g pm2
                    yarn start:pm2
                    sleep 5
                    '''
                }
            }
        }


        stage('Test the app') {
            steps {
                script {
                    sh 'curl http://localhost:3000/health'
                }
            }
        }

        stage('Stop the App') {
            steps {
                script {
                    sh 'pm2 stop todos-app'
                }
            }
        }
        
        stage('Add Host to known_hosts') {
            steps {
                sh '''
                mkdir -p /var/lib/jenkins/.ssh
                ssh-keyscan -H 13.53.172.221 >> /var/lib/jenkins/.ssh/known_hosts
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'AWS_INSTANCE_SSH', keyFileVariable: 'DEPLOY_SSH_KEY')]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no -i $DEPLOY_SSH_KEY ubuntu@13.53.172.221 << 'EOF'
                    cd /home/ubuntu/todos-app
                    curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
                    sudo apt-get install -y nodejs
                    sudo npm install -g pm2
                    yarn install
                    pm2 start --interpreter babel-node src/app.js --name todos-app
                    EOF
                    '''

                }
            }
        }

    }
}
