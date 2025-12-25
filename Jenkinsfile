pipeline {
   agent {
        node {
            label 'AGENT-1'
        }
    }
    environment {
        appVersion= ''
        REGION= 'us-east-1'
        ACC_ID= '784585544641'
        PROJECT= 'roboshop'
        COMPONENT= 'catalogue'
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
     /*  parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')
        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password') 
    } */
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
                    docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                    docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                   """
                }
            }
}

        stage('Build') {
            steps {
                script {
                    sh """
                        echo "Hello bulid"
                        sleep 10
                        env
                        echo "Hello ${params.PERSON}"
                    """
                }
                
            }
        }
        stage('Test') {
            steps {
                script {
                    echo 'Testing..'
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