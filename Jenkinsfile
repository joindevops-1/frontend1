pipeline {
    agent {
        label 'AGENT-1'
    }

    options {
        timeout(time: 10, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }

    environment { 
        appVersion = ''
        account_id = '315069654700'
        region = 'us-east-1'
        environment = 'dev'
        project = 'expense'
        component = 'frontend'
    }
    
    stages {
        stage('Read Version') {
            steps {
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version: $appVersion"
                }
            }
        }
         stage('Install Dependencies') {
            steps {
               sh """
                npm install
                ls -ltr
                echo "application version: $appVersion"
               """
            }
        }
        
        stage('Docker build'){
            
                steps{
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    sh """
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com

                        docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/expense/${environment}/frontend:${appVersion} .

                        docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/expense/${environment}/frontend:${appVersion}
                    """
                }
            }
        }
        stage('Deploy'){
            steps{
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    script{               
                        echo "${component} not installed yet, first time installation"
                        sh"""
                            aws eks update-kubeconfig --region ${region} --name ${project}-${environment}
                            cd helm
                            sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environment}.yaml
                            helm upgrade --install ${component} -n ${project} -f values-${environment}.yaml .
                        """
                    }
                }
            }
        }
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
        }
        success { 
            echo 'I will run when pipeline is success'
        }
        failure { 
            echo 'I will run when pipeline is failure'
        }
    }
}