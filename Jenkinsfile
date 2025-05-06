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
                ssh-keyscan -H 13.48.196.184 >> /var/lib/jenkins/.ssh/known_hosts
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'AWS_INSTANCE_SSH', keyFileVariable: 'DEPLOY_SSH_KEY')]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $DEPLOY_SSH_KEY ubuntu@13.48.196.184 << 'EOF'
                            if [ ! -d "todos-app" ]; then
                                git clone https://github.com/AhmadMazaal/todos-app.git todos-app
                                cd todos-app
                            else
                                cd todos-app
                                git pull
                            fi

                            yarn install
                    
                            if pm2 describe todos-app > /dev/null ; then
                                pm2 restart todos-app
                            else
                                yarn start:pm2
                            fi
                        EOF
                    '''
                }
            }
        }

    }
}
