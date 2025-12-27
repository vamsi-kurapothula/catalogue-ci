pipeline {
   agent {
        node {
            label 'AGENT-1'
        }
    }
    environment {
        REGION= 'us-east-1'
        ACC_ID= '784585544641'
        PROJECT= 'roboshop'
        COMPONENT= 'catalogue'
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    parameters {
       string(name: 'deploy', defaultValue: 'false', description: 'Toggle this value') 
       choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the environment')
    } 

// build
    
    stages {
       stage('Read package.json') {
            steps {
                script {
                    def packageJSON = readJSON file: 'package.json'
                     appVersion = packageJSON.version
                    echo "The project version is: ${appVersion}"

                }
            }
        }
        stage('install Dependencies') {
            steps {
                script {
                   sh """
                      npm install
                   """
                }
            }
        }
        stage('Docker Build') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${REGION}") {
                   sh """
                     aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                    docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                    docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                   """
                } 
            }
        }
        stage('Trigger Deploy') {
                when {
                    expression { params.deploy_to == 'dev' }
                }
                steps {
                    script {
                        build(
                            job: 'catalogue-cd',
                            parameters: [
                                string(name: 'appVersion', value: appVersion),
                                string(name: 'deploy_to', value: params.deploy_to)
                            ],
                            wait: false,       // CI will not wait for CD completion
                            propagate: false   // CD failure will not fail CI
                        )
                    }
                }
        }

    }

    post { 
        always { 
            echo 'I will always say Hello again!'
             cleanWs() 
        }
        success {
            echo 'hi this is success'
        }
        failure {
            echo 'hi, this is failure'
        }
    }
}